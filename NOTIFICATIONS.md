# نظام الإشعارات — Notifications

دليل استخدام نظام الإشعارات في NUBL.

---

## 1. نظرة عامة

| المكون | الوظيفة |
|--------|---------|
| **جدول notifications** | تخزين الإشعارات (Laravel Notifications) |
| **DonationReceiptNotification** | إشعار إيصال التبرع (قاعدة بيانات + بريد) |
| **NotificationController** | API لجلب الإشعارات وتحديد المقروء |
| **notificationPanel** | مكوّن Alpine.js لعرض الإشعارات في الهيدر |

### إشعارات الأدمن (قائمة المراجعة)

| النوع (`type`) | متى يُرسل | الرابط الافتراضي |
|----------------|-----------|------------------|
| `account_approval_pending` | تسجيل مستفيد أو مزوّد جديد بحالة `pending_approval` | `admin.users.application` لمقدّم الطلب |
| `documents_resubmitted_for_review` | انتقال الحالة من `rejected` إلى `pending_approval` (إعادة إرسال مستندات عبر `ResubmitApplicationController` أو أي تحديث يحقق هذا الانتقال) | نفس الرابط أعلاه |
| `provider_payout_pending_review` | إنشاء طلب تحويل أسبوعي جديد للمزوّد (`ProviderPayoutGenerationService`) | `admin.finances.provider-payouts.show` |

- **كل المستخدمين ذوي دور `admin`** يستلمون الإشعار (`User::role('admin')` ثم `notify` لكل منهم).
- الإرسال عند أول تقديم: `NotificationService::sendNewUserRegisteredToAdmins()` من مساري تسجيل المستفيد والمزوّد (`RegisteredUserController`، `ProviderRegistrationController`) عند اكتمال التسجيل بحالة `pending_approval`.
- إعادة التقديم بعد الرفض: `UserObserver` على `updated` عند انتقال الحالة من `rejected` إلى `pending_approval` → `sendDocumentsResubmittedForReviewToAdmins()`.

يُسجَّل الأنواع في `config/notifications.php` ويُستدعَى من `NotificationService` و`UserObserver` و`ProviderPayoutGenerationService` حسب الحالة.

### إشعار المزوّد (ليس للأدمن)

| النوع (`type`) | متى يُرسل | الرابط الافتراضي |
|----------------|-----------|------------------|
| `provider_payout_transferred` | تأكيد الأدمن لتحويل بنكي للمزوّد (`ProviderPayoutConfirmationService`) | لوحة المزوّد (`provider.dashboard`) |

---

## 2. إرسال إشعار للمستخدم

### الطريقة الأساسية

```php
use App\Notifications\DonationReceiptNotification;

$user = User::find(1);
$user->notify(new DonationReceiptNotification($payment));
```

### شروط النموذج (User)

- يجب أن يستخدم الـ User الـ trait `Notifiable` (موجود افتراضياً في `App\Models\User`)

---

## 3. إنشاء إشعار جديد

### الخطوة 1: إنشاء كلاس الإشعار

```bash
php artisan make:notification MyCustomNotification
```

### الخطوة 2: تعريف القنوات (Channels)

```php
public function via(object $notifiable): array
{
    $channels = ['database'];  // للتخزين في جدول notifications
    if (!empty($notifiable->email)) {
        $channels[] = 'mail';   // للإرسال بالبريد
    }
    return $channels;
}
```

### الخطوة 3: تعريف محتوى قاعدة البيانات (toArray)

```php
public function toArray(object $notifiable): array
{
    return [
        'type' => 'my_custom_type',      // مطلوب — يُستخدم في NotificationController
        'message' => __('Your message'),
        'subtitle' => __('Optional subtitle'),
        'url' => route('some.route'),
        // أي حقول إضافية حسب الحاجة
    ];
}
```

### الخطوة 4: تعريف البريد (toMail) — اختياري

```php
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject(__('Subject'))
        ->line(__('Line 1'))
        ->action(__('Action'), url('/path'));
}
```

### الخطوة 5: إضافة النوع في NotificationController

في `formatNotification()` أضف حالة جديدة في `match`:

```php
'my_custom_type' => [
    'icon' => 'success',        // success | warning | info | primary
    'icon_svg' => 'check-circle',  // check-circle | bell | clock | users
    'title' => $data['message'] ?? __('Default title'),
    'subtitle' => $data['subtitle'] ?? '',
    'url' => $data['url'] ?? '#',
],
```

---

## 4. المسارات (Routes)

| المسار | الطريقة | الوظيفة |
|--------|---------|---------|
| `GET /notifications` | GET | جلب الإشعارات (JSON) |
| `POST /notifications/{id}/read` | POST | تحديد إشعار كمقروء |
| `POST /notifications/read-all` | POST | تحديد الكل كمقروء |

**ملاحظة:** جميع المسارات تتطلب تسجيل الدخول (`auth` middleware).

---

## 5. API الاستجابة

### GET /notifications

```json
{
  "notifications": [
    {
      "id": "uuid",
      "type": "donation_receipt",
      "read_at": null,
      "created_at": "2026-03-02T12:00:00.000000Z",
      "title": "شكراً لك! تمت عملية التبرع بمبلغ 100.00 ريال بنجاح.",
      "subtitle": "تم إرسال الإيصال إلى بريدك الإلكتروني",
      "url": "http://localhost/donor/donations",
      "icon": "success",
      "icon_svg": "check-circle"
    }
  ],
  "unread_count": 1
}
```

---

## 6. الواجهة الأمامية (Frontend)

### آلية التحديث (Polling)

- يتم جلب الإشعارات عند فتح القائمة
- يتم التحديث تلقائياً كل **30 ثانية**
- المكوّن: `resources/js/components/notificationPanel.js`

### استخدام الـ API من JavaScript

```javascript
// جلب الإشعارات
const res = await fetch('/notifications', {
  headers: { Accept: 'application/json', 'X-Requested-With': 'XMLHttpRequest' },
  credentials: 'same-origin',
});
const { notifications, unread_count } = await res.json();

// تحديد كمقروء
await fetch(`/notifications/${id}/read`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]')?.content,
  },
  credentials: 'same-origin',
});
```

---

## 7. الأيقونات المتاحة

| icon_svg | اللون | الاستخدام |
|----------|-------|-----------|
| `check-circle` | success (أخضر) | نجاح، إكمال |
| `bell` | info (أزرق) | تنبيه عام |
| `clock` | warning (أصفر) | معلق، انتظار |
| `users` | primary | مستخدمون، مجموعات |

---

## 8. التنفيذ المتزامن vs قائمة الانتظار

- **DonationReceiptNotification** يعمل **مباشرة** (بدون Queue) حتى يظهر الإشعار فوراً
- إذا أردت استخدام `ShouldQueue` لإشعارات أخرى، يجب تشغيل:
  ```bash
  php artisan queue:work
  ```

---

## 9. الترجمات

أضف النصوص في `lang/ar.json`:

```json
"Your notification message": "رسالة الإشعار بالعربية"
```

---

## 10. الملفات المرجعية

| الملف | الوظيفة |
|-------|---------|
| `app/Notifications/DonationReceiptNotification.php` | إشعار إيصال التبرع |
| `app/Http/Controllers/NotificationController.php` | API الإشعارات |
| `resources/js/components/notificationPanel.js` | مكوّن العرض |
| `resources/views/components/app-partials/header.blade.php` | مكان عرض الإشعارات |
| `database/migrations/*_create_notifications_table.php` | جدول الإشعارات |
