# تجهيز الإنتاج (Production)

ملخص متطلبات وتشغيل المشروع على بيئة الإنتاج. راجع `.env.example` لقائمة المتغيرات.

## المتطلبات

| البند | ملاحظة |
|--------|--------|
| PHP | `^8.2` (انظر `composer.json`) |
| امتدادات PHP | عادة: `pdo_mysql`، `mbstring`، `openssl`، `tokenizer`، `xml`، `ctype`، `json`، `bcmath`، `fileinfo`، `gd` أو ما يطلبه المشروع |
| Composer | لتثبيت التبعيات |
| Node.js + npm | لبناء الأصول الأمامية (`vite`) |
| قاعدة بيانات | MySQL/MariaDB (الإعداد الافتراضي في المشروع) |
| خادم ويب | Nginx أو Apache + PHP-FPM |

## إعداد البيئة

1. انسخ `.env.example` إلى `.env` واضبط على الخادم:
   - `APP_ENV=production`
   - `APP_DEBUG=false`
   - `APP_KEY` (مولَّد)
   - `APP_URL` = عنوان الموقع الفعلي (HTTPS)
   - `APP_TIMEZONE` حسب المنطقة (مثلاً `Asia/Riyadh`)
   - `DB_*` لقاعدة البيانات
   - البريد، SMS، MyFatoorah، وغيرها حسب التشغيل الفعلي
2. جلسات وطوابير: المشروع يفترض غالباً `SESSION_DRIVER=database` و`QUEUE_CONNECTION=database` و`CACHE_STORE=database` — تأكد من وجود جداول الطابور/الكاش إن لزم (`php artisan queue:table` ثم ترحيل، أو استخدم Redis إن غيّرت الإعدادات).

## أوامر بعد الرفع (أو في سكربت النشر)

```bash
composer install --no-dev --optimize-autoloader
npm ci
npm run build
php artisan migrate --force
php artisan storage:link
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

- **لا** تشغّل `config:cache` أثناء التطوير المحلي إن كنت تعدّل `.env` باستمرار.
- إن فشل أمر من أوامر الـ cache، راجع الأخطاء ثم أعد المحاولة بعد إصلاح الصلاحيات.

## صلاحيات المجلدات

تأكد أن خادم الويب/PHP يستطيع الكتابة في:

- `storage/`
- `bootstrap/cache/`

## المجدول (Scheduler)

المشروع يعتمد على مهام مجدولة في `routes/console.php` (تقارير، دفعات مزوّدين، إلخ).

**خيار 1 — موصى به في الإنتاج:** إدخال Cron يشغّل Laravel كل دقيقة:

```cron
* * * * * cd /path/to/project && php artisan schedule:run >> /dev/null 2>&1
```

استبدل `/path/to/project` بمسار المشروع على الخادم.

**خيار 2:** عملية طويلة الأمد (مثلاً تحت systemd أو Supervisor):

```bash
php artisan schedule:work
```

يُشغَّل كخدمة دائمة وليس من الطرفية مؤقتاً.

## طابور المهام (Queue)

إذا كان `QUEUE_CONNECTION` يستخدم `database` (أو redis)، شغّل عامل الطابور باستمرار:

```bash
php artisan queue:work --sleep=3 --tries=3
```

أو عبر Supervisor/systemd بعدة عمال حسب الحمل.

## بعد النشر السريع

- افتح الموقع وتأكد من تسجيل الدخول والدفع/الإشعارات حسب ما تفعّله.
- راجع `storage/logs/laravel.log` عند أي خطأ.

## فحص سريع قبل الإطلاق

- [ ] `APP_DEBUG=false` و`APP_URL` صحيح
- [ ] Cron لـ `schedule:run` أو خدمة `schedule:work`
- [ ] عامل `queue:work` إن كنت تستخدم الطوابير
- [ ] بناء الواجهة: `npm run build`
- [ ] الترحيلات منفّذة: `migrate --force`
- [ ] `routes/console.php` يعكس جدولة الإنتاج (وليس إعدادات التجربة المحلية فقط)
