# دليل سجل التدقيق (Activity Log) - للمطورين

## ما هو سجل التدقيق؟

نستخدم **Spatie Laravel Activity Log** لتسجيل الأحداث المهمة في النظام (مثل: إنشاء مستخدم، الموافقة على حساب، إضافة عنصر قائمة، إلخ). كل حدث يُخزَّن في جدول `activity_log` مع معلومات من قام به ومتى.

---

## بعد سحب الكود (Git Pull) — ماذا تفعل؟

عندما تسحب التحديثات من الريبو، اتبع الخطوات التالية:

### 1. تحديث الحزم

```bash
composer install
```

### 2. تشغيل الـ Migrations (إن وجدت جديدة)

```bash
php artisan migrate
```

هذا ينشئ أو يحدّث جدول `activity_log` إذا لم يكن موجوداً.

### 3. لا حاجة لأي إعداد إضافي

الباكج يعمل تلقائياً بعد `composer install` و `migrate`. لا حاجة لنسخ ملفات config أو تشغيل أوامر publish إضافية.

---

## كيف أضيف تسجيل تدقيق في مكان جديد؟

### الطريقة الموصى بها: استخدام AuditService

1. **في الـ Controller أو الـ Service** — أضف `AuditService` في الـ constructor:

```php
use App\Http\Services\AuditService;

class MyController extends Controller
{
    public function __construct(
        private AuditService $auditService
    ) {}
}
```

2. **عند حدوث الحدث** — استدعِ `log`:

```php
$this->auditService->log('اسم_الكيان', 'اسم_الإجراء', [
    'id' => 123,
    'أي_بيانات_إضافية' => 'القيمة',
]);
```

### أمثلة عملية

**مثال 1: تسجيل إنشاء طلب**

```php
$this->auditService->log('order', 'created', [
    'order_id' => $order->id,
    'amount' => $order->total,
]);
```

**مثال 2: تسجيل حدث قام به مستخدم معيّن (مثل الأدمن)**

```php
$this->auditService->log('user', 'deactivated', [
    'user_id' => $user->id,
], auth()->id()); // المستخدم الحالي (الأدمن) هو من قام بالإجراء
```

**مثال 3: في الـ Service**

```php
class DonationService
{
    public function __construct(
        private AuditService $auditService
    ) {}

    public function confirmDonation($donation)
    {
        // ... منطق التأكيد ...

        $this->auditService->log('donation', 'confirmed', [
            'donation_id' => $donation->id,
            'amount' => $donation->amount,
        ], $donation->user_id);
    }
}
```

---

## أسماء الكيانات والإجراءات الحالية

| الكيان (entity) | الإجراءات (actions) | الموقع |
|-----------------|---------------------|--------|
| `user` | created, updated, deleted, deactivated, reactivated | UserService |
| `registration` | completed | التسجيل الذاتي (متبرع / مستفيد / مزوّد): `RegisteredUserController`, `ProviderRegistrationController` — البيانات: `user_id`, `membership_type`, `requires_approval` (بدون بريد أو كلمة مرور) |
| `application` | resubmitted | إعادة تقديم طلب بعد الرفض: `ResubmitApplicationController` — `user_id`, `membership_type` |
| `auth` | login, logout, password_reset_completed, phone_verified, email_verified | تسجيل الدخول (كلمة مرور أو OTP)، تسجيل الخروج، إعادة تعيين كلمة المرور، التحقق من الجوال/البريد — `AuthenticatedSessionController`, `OtpLoginController`, `NewPasswordController`, `PhoneVerificationController`, `VerifyEmailController` — عادةً `user_id` و`method` للدخول (`password` \| `otp`) |
| `account_approval` | approved, rejected | AccountApprovalController |
| `menu_item` | created, updated, deactivated | MenuItemController |
| `donation` | confirmed | DonationService |
| `request` | created | RecipientRequestController |
| `wallet` | donation_added, payout_to_provider | SystemWalletService |
| `role` | created, updated, deleted | RoleManagementService |

عند إضافة كيان جديد، استخدم أسماء واضحة بالإنجليزي مثل: `order`, `payment`, `notification`.

---

## معلمات الدالة `log`

| المعامل | النوع | الوصف |
|---------|-------|-------|
| `$entity` | string | نوع الكيان (مثل: user, order, menu_item) |
| `$action` | string | الإجراء (مثل: created, updated, deleted) |
| `$data` | array | بيانات إضافية (اختياري) |
| `$userId` | int\|null | من قام بالإجراء (اختياري، الافتراضي: المستخدم الحالي) |

---

## استعلام السجل (للأدمن لاحقاً)

يمكن قراءة السجل عبر الموديل:

```php
use Spatie\Activitylog\Models\Activity;

// آخر 50 حدث
$activities = Activity::latest()->take(50)->get();

// أحداث مستخدم معيّن
$userActivities = Activity::causedBy($user)->get();

// أحداث على كيان معيّن
$modelActivities = Activity::forSubject($donation)->get();
```

---

## ملخص سريع

1. **بعد السحب**: `composer install` ثم `php artisan migrate`
2. **للتسجيل**: حقن `AuditService` واستدعاء `log('entity', 'action', [...])`
3. **البيانات**: تُخزَّن في جدول `activity_log` تلقائياً
