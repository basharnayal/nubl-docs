# Payment System — مرجع الاستخدام

مرجع شامل لنظام الدفع والتبرعات في Nubl.

---

## 1. نظرة عامة

| المكون | الوظيفة |
|--------|---------|
| **Payment** | سجل عملية الدفع عند بوابة الدفع (Gateway) |
| **FundTransaction** | حركة الرصيد الفعلية داخل النظام (Ledger) |
| **Ewallet** | المحفظة (صندوق المدينة أو المزود) |
| **RequestPaymentLink** | ربط التبرعات بالطلبات (Allocation) |

---

## 2. تدفق التبرع (Donation Flow)

```
الداعم يحدد المبلغ → يختار الدفع → يُحوّل لـ MyFatoorah → يدفع
                                    ↓
                    عند النجاح: payment SUCCEEDED + fund_transaction IN + redirect success
                    عند الفشل:  payment FAILED + redirect failed (بدون إدخال أموال)
```

---

## 3. المسارات (Routes)

### صفحات الداعم (تحت auth + role:donor)

| المسار | الاسم | الوظيفة |
|-------|------|---------|
| `GET /donor/donations/new` | `donor.donations.new` | نموذج إدخال المبلغ |
| `POST /donor/payments/initiate` | `donor.payments.initiate` | إنشاء Payment والتحويل لـ MyFatoorah |
| `GET /donor/payments/success` | `donor.payments.success` | صفحة نجاح الدفع |
| `GET /donor/payments/failed` | `donor.payments.failed` | صفحة فشل الدفع |
| `GET /donor/donations` | `donor.donations.index` | قائمة تبرعات الداعم وتأثيرها |

### Callback (بدون auth — MyFatoorah يوجّه هنا)

| المسار | الاسم | الوظيفة |
|-------|------|---------|
| `GET /payments/callback` | `payments.callback` | CallBackUrl بعد الدفع |
| `GET /payments/error` | `payments.error` | ErrorUrl عند الفشل |

---

## 4. الخدمات (Services)

### 4.1 PaymentService

**الملف:** `app/Http/Services/PaymentService.php`

| الدالة | الوظيفة |
|--------|---------|
| `initiateSponsorPayment($sponsorId, $amount, $idempotencyKey?)` | إنشاء Payment بـ status=INITIATED، Audit، إرجاع Payment |
| `redirectToGateway(Payment $payment)` | إنشاء فاتورة MyFatoorah، تحديث Payment، redirect لصفحة الدفع. **FR-3.4:** عند فشل API يُعاد للمستخدم مع رسالة إعادة المحاولة |
| `handleCallback(Request $request)` | معالجة callback، التحقق عبر API، تحديث Payment، إدخال أموال أو redirect فشل |
| `handleError(Request $request)` | معالجة ErrorUrl، تحديث Payment إلى FAILED إن وُجد |

**مثال استخدام:**

```php
// بدء التبرع
$payment = $paymentService->initiateSponsorPayment(auth()->id(), 100.00);
return $paymentService->redirectToGateway($payment);
```

### 4.2 MyFatoorahService

**الملف:** `app/Http/Services/MyFatoorahService.php`

| الدالة | الوظيفة |
|--------|---------|
| `createInvoice($amount, $customerReference, $callbackUrl, $errorUrl, $email?, $name?)` | إنشاء فاتورة، يرجع `['invoice_id', 'payment_url', 'raw_response']` |
| `getPaymentStatus($keyId, $keyType)` | التحقق من حالة الدفع، يرجع `['status', 'invoice_id', 'raw_response']` |

**ملاحظة:** التحقق النهائي يعتمد على `getPaymentStatus` وليس query params فقط.

### 4.3 SystemWalletService

**الملف:** `app/Http/Services/SystemWalletService.php`

| الدالة | الوظيفة |
|--------|---------|
| `addFundsFromDonation($amount, $donorId, $paymentId?)` | إضافة FundTransaction IN للمحفظة النظامية |
| `getSystemWallet()` | إرجاع المحفظة النظامية |
| `hasSufficientBalance($amount)` | التحقق من كفاية الرصيد |
| `transferToProviderForRequest(Request $request, ?int $orderRedemptionId)` | خصم من SYSTEM wallet، إضافة لمحفظة المزود (يُستدعى عند Redemption، وليس عند Approval) |

### 4.4 AllocationService

**الملف:** `app/Http/Services/AllocationService.php`

| الدالة | الوظيفة |
|--------|---------|
| `allocateToRequest($requestId, $amount)` | توزيع المبلغ على Payments (FIFO)، إنشاء request_payment_links |

**استراتيجية FIFO:** أقدم التبرعات الناجحة تُستهلك أولاً.

---

## 5. النماذج (Models)

### Payment

| الحقل | النوع | الوصف |
|-------|------|-------|
| sponsor_id | FK users | الداعم |
| gateway | varchar | MYFATOORAH |
| external_payment_id | varchar | InvoiceId من MyFatoorah |
| status | enum | INITIATED, PENDING, PROCESSING, SUCCEEDED, FAILED |
| amount | decimal | المبلغ |
| notes | json | raw response للتدقيق |
| idempotency_key | uuid | لتجنب التكرار |

**العلاقات:** `sponsor()`, `fundTransactions()`, `requestPaymentLinks()`

### FundTransaction

| الحقل | الوصف |
|-------|-------|
| wallet_id | المحفظة |
| sponsor_id | الداعم (للتبرعات) |
| source | DONATION, REDEMPTION, REFUND, PAYOUT, ... |
| direction | IN أو OUT |
| amount | المبلغ |
| payment_id | ربط بالدفعة (للتبرعات) |
| request_id | ربط بالطلب (للخصم) |

### RequestPaymentLink

| الحقل | الوصف |
|-------|-------|
| payment_id | الدفعة |
| request_id | الطلب |
| amount | مبلغ المساهمة |

---

## 6. الإعدادات (Config)

**الملف:** `config/services.php`

```php
'myfatoorah' => [
    'api_key' => env('MYFATOORAH_API_KEY'),
    'country_code' => env('MYFATOORAH_COUNTRY_CODE', 'SAU'),
    'is_test' => env('MYFATOORAH_IS_TEST', true),
],
```

**الملف:** `.env`

```
MYFATOORAH_API_KEY=your_api_key
MYFATOORAH_COUNTRY_CODE=SAU
MYFATOORAH_IS_TEST=true
```

---

## 7. Idempotency (منع التكرار)

البوابة قد ترسل callback أكثر من مرة. التحقق يتم كالتالي:

1. إذا `payment.status === SUCCEEDED` → redirect success فوراً
2. إذا وُجد `FundTransaction` بنفس `payment_id` → redirect success فوراً
3. فقط بعد اجتياز الفحصين يتم إنشاء fund_transaction

---

## 8. مصدر الحقيقة للرصيد

- **`ewallets.balance`** هو المصدر الرسمي.
- التحديث يتم عبر **FundTransactionObserver** عند إنشاء كل FundTransaction:
  - IN → `$wallet->increment('balance', $amount)`
  - OUT → `$wallet->decrement('balance', $amount)`
- **لا يوجد تعديل يدوي** على ewallet.

---

## 9. Audit Events

| الحدث | الموقع |
|-------|--------|
| payment_initiated | PaymentService::initiateSponsorPayment |
| gateway_initiated | PaymentService::redirectToGateway |
| callback_received | PaymentService::handleCallback |
| payment_succeeded | PaymentService::handleCallback |
| payment_failed | PaymentService::handleCallback / handleError |
| wallet_donation_added | SystemWalletService::addFundsFromDonation |
| payout_to_provider | SystemWalletService::transferToProviderForRequest |
| allocation_created | AllocationService::allocateToRequest |

---

## 10. عرض تأثير الداعم

من `request_payment_links` يمكن عرض:

- لكل Payment: قائمة الطلبات التي ساهم بها + المبالغ
- أو عرض الطلبات فقط بدون مبالغ

**مثال استعلام:**

```php
$donorPaymentIds = Payment::where('sponsor_id', auth()->id())
    ->where('status', Payment::STATUS_SUCCEEDED)
    ->pluck('id');

$donorRequestsFunded = RequestPaymentLink::whereIn('payment_id', $donorPaymentIds)
    ->distinct('request_id')
    ->count('request_id');
```

---

## 11. تلبية الطلب (Provider Approve & Redemption)

**عند موافقة المزود (Approval):**
- التحقق من كفاية الرصيد (`hasSufficientBalance`)
- تحديث الطلب إلى REDEEMABLE و funding_source CITY_FUND
- إنشاء OrderRedemption (رمز QR للمستلم)
- **لا يحدث تحويل** في هذه المرحلة

**عند الاستلام (Redemption — مسح QR من المستلم عند المزود):**
1. `AllocationService::allocateToRequest` — توزيع المبلغ على Payments (FIFO)
2. `SystemWalletService::transferToProviderForRequest` — FundTransaction OUT من SYSTEM wallet و IN لمحفظة المزود (مع order_redemption_id)
3. Audit: payout_to_provider

---

## 12. أوامر مفيدة

```bash
# مزامنة رصيد المحافظ من fund_transactions
php artisan wallets:sync-balances
```

---

## 13. الملفات المرجعية

| الملف | الوظيفة |
|-------|---------|
| `app/Http/Services/PaymentService.php` | دورة حياة الدفع |
| `app/Http/Services/MyFatoorahService.php` | تكامل MyFatoorah v2 |
| `app/Http/Services/SystemWalletService.php` | عمليات صندوق المدينة |
| `app/Http/Services/AllocationService.php` | توزيع التبرعات على الطلبات |
| `app/Http/Controllers/Donor/DonationController.php` | واجهة الداعم |
| `app/Http/Controllers/PaymentCallbackController.php` | معالجة callback و error |
| `app/Observers/FundTransactionObserver.php` | تحديث رصيد المحفظة تلقائياً |
