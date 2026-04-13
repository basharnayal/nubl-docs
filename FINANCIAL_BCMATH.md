# الحسابات المالية و BCMath

هذا المستند يوثّق التعديلات المتعلقة بمعالجة المبالغ (الريال، خانتان عشريتان) باستخدام **BCMath** بدلاً من الاعتماد على عمليات **float** في المسارات الحساسة، ويضمّن **خطة تراجع** إن رغبت في إلغاء التغييرات.

---

## 1. المشكلة التي عالجتها التعديلات

- **العمليات المالية على `float`** في PHP قد تسبب أخطاء تقريب صغيرة تتراكم مع آلاف المعاملات.
- **ملحق `bcmath`** لم يكن مُعلَناً في `composer.json`، فلا ضمان أن كل بيئة تشغيل تملكه.
- **`ProviderPayoutLedgerService`** كان يستخدم `bcadd`/`bccomp` فقط إن وُجدت الدالة، وإلا يعود إلى مقارنة/جمع **float**.

---

## 2. ما تم تنفيذه بالضبط

### 2.1 `composer.json`

- إضافة **`"ext-bcmath": "*"`** ضمن `require` لضمان أن تثبيت المشروع يفشل إن لم يكن ملحق BCMath مفعّلاً في PHP.

### 2.2 ملف جديد: `app/Support/FinancialMath.php`

- كلاس **`App\Support\FinancialMath`** (نهائي ثابت):
  - **`normalize`**: تطبيع المبلغ إلى سلسلة عشرية بخانتين، مع `trim` ومعالجة السلسلة الفارغة؛ يعتمد على `bcadd(..., '0', 2)`.
  - **`add` / `sub` / `compare` / `min`**: `bcadd` / `bcsub` / `bccomp` بمقياس **2** خانتين.

### 2.3 `app/Observers/FundTransactionObserver.php`

- قراءة مبلغ المعاملة من القيمة الخام (`getRawOriginal('amount')` عند الحاجة) ثم **`FinancialMath::normalize`**.
- مقارنة الصفر عبر **`FinancialMath::compare`** بدلاً من `<= 0` على float.
- تمرير **`increment` / `decrement`** على عمود `balance` بقيمة نصية عشرية طبيعية، وليس تحويل float أولاً.
- سجل التدقيق ما زال يخزّن المبلغ كـ **`(float)`** للتوافق مع الحقول الموجودة.

### 2.4 `app/Http/Services/SystemWalletService.php`

- **`hasSufficientBalance`**: مقارنة رصيد المحفظة مع المبلغ عبر **`FinancialMath::compare`** بعد **`normalize`** للقيم من القيمة الخام/النموذج.
- **`transferToProviderForRequest`**: مبلغ `reserved_amount` يُطبَّع كنص عشري قبل الفحوصات والإنشاء؛ ما زال إنشاء `FundTransaction` يمرّر **`(float)`** للتوافق مع النموذج.

### 2.5 `app/Http/Services/AllocationService.php`

- **`availableCityFundAmount`**: يُرجع `(float)` من دالة داخلية **`availableCityFundAmountString()`** التي تجمع المتاح عبر **`add`/`sub`/`normalize`** على مبالغ الدفع والروابط.
- **`canCoverRequestAmount`**: مقارنة **`availableCityFundAmountString()`** مع **`normalize($amount)`**؛ **إزالة** هامش **`+0.001`** الذي كان يغطي ضجيج float.
- **`allocateToRequest`**: متبقي التخصيص وحسابات `available`/`min` عبر **`FinancialMath`**؛ فشل عند **`compare` المتبقي مع `'0.00'`** بدلاً من `> 0.001`.

### 2.6 `app/Models/Ewallet.php`

- **`getBalanceFromTransactions()`**: نوع الإرجاع أصبح **`string`**؛ يُحسب الفرق **`sum(IN) - sum(OUT)`** عبر **`FinancialMath::sub`** بعد **`normalize`**.
- استخدام **`FundTransaction::DIRECTION_IN` / `DIRECTION_OUT`** في استعلامات الاتجاه.

### 2.7 `app/Http/Services/ProviderPayoutLedgerService.php`

- **`addDecimal` / `compare` / `roundDecimal`**: تفويض إلى **`FinancialMath`** بدون fallback إلى float.

### 2.8 `app/Http/Services/Admin/AdminFinancialService.php`

- **`ledger_net_amount`** في **`getRangeSummary`**: طُرح من **`FinancialMath::sub`** بين نتيجتي `normalize` لـ `ledgerIn` و`ledgerOut` بدلاً من `round($ledgerIn - $ledgerOut, 2)` على float.

### 2.9 `app/Http/Controllers/Admin/FinancialOverviewController.php`

- **`unsuccessful_payments_amount`**: جمع **`pending_amount` + `failed_amount`** عبر **`FinancialMath::add`** بعد **`normalize`** لكل منهما.

---

## 3. ما لم يُغيّر عمداً (للعلم)

- تجميعات قراءة فقط في **`AdminFinancialService::getOverview`** وغيرها ما زالت تُحوّل نتائج **`sum()`** من SQL إلى **`(float)`** للعرض في كثير من الحقول؛ الـ **SUM** يُنفَّذ في قاعدة البيانات.
- لوحات المتبرع/المستلم/المزود والإشعارات والبوابات قد تستخدم **`(float)`** لعرض القيم أو طبقة API.
- حدود **Eloquent** أو أعمدة **`decimal`** قد تستقبل **`(float)`** عند الإنشاء في بعض المواضع.

---

## 4. خطة التراجع (Rollback)

إذا قررت **إلغاء كل التعديلات** والعودة إلى السلوك السابق في Git:

### 4.1 تراجع عبر Git (الأنظف)

إن كانت التغييرات في فرع أو commit واحد:

```bash
git log --oneline -5
git revert <commit-hash>
```

أو إعادة الفرع إلى حالة قبل الدمج:

```bash
git checkout main
git reset --hard <commit-before-الfinancial-bcmath>
git push --force-with-lease   # فقط إذا كان الفرع خاصاً ولا يشاركها آخرون بدون تنسيق
```

**تحذير:** `reset --hard` و`force push` يحذفان تاريخاً على الفرع؛ استخدمهما فقط عند فهم العواقب.

### 4.2 تراجع يدوي (ملفّات محددة)

1. **`composer.json`**: احذف السطر `"ext-bcmath": "*",` من `require`.
2. احذف الملف **`app/Support/FinancialMath.php`**.
3. أعد **`app/Observers/FundTransactionObserver.php`**, **`SystemWalletService.php`**, **`AllocationService.php`**, **`Ewallet.php`**, **`ProviderPayoutLedgerService.php`**, **`AdminFinancialService.php`**, **`FinancialOverviewController.php`** إلى نسخة **ما قبل** التعديل (من `git checkout -- <file>` أو نسخ من الفرع `main`).
4. نفّذ **`composer update`** (أو على الأقل **`composer validate`**) بعد تعديل `composer.json`.
5. شغّل **`php artisan test`** للتأكد.

### 4.3 بيانات قاعدة البيانات

- **لا** توجد ترحيلات (migrations) مرتبطة بهذا التغيير؛ **لا** يلزم تعديل جداول للتراجع عن الكود.
- أرصدة المحافظ والمعاملات المخزّنة **لم تُغيّر** تلقائياً بسبب هذا الـ PR؛ أي اختلاف لاحق يأتي من **عمليات جديدة** فقط بعد النشر.

### 4.4 بعد التراجع

- تأكد أن **`ProviderPayoutLedgerService`** يعود إلى السلوك السابق (مع أو بدون fallback float حسب النسخة المسترجعة).
- راقب سيناريوهات **`canCoverRequestAmount`** إن أعدت **هامش `0.001`**؛ السلوك عند الحدود قد يختلف قليلاً عن النسخة الحالية.

---

## 5. مراجع سريعة

| الملف | الدور |
|--------|--------|
| `app/Support/FinancialMath.php` | طبقة BCMath المركزية |
| `composer.json` | `ext-bcmath` |
| الملفات المذكورة في القسم 2 | استخدام `FinancialMath` أو استبدال منطق float |

---

*آخر تحديث للمستند: يتوافق مع فرع التطوير الذي يضم تعديلات BCMath المالية الموصوفة أعلاه.*
