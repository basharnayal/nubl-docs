# Docker — تشغيل بيئة التطوير

> **⚠️ أوقف Laragon (أو أي MySQL/Redis محلي) قبل التشغيل** — Docker يوفّر كل شيء، ولا حاجة لخدمات خارجية. تشغيلهما معاً يسبب تعارض في المنافذ.

## التشغيل السريع

```bash
docker compose up -d --build
```

| الخدمة | العنوان | الوصف |
|--------|---------|-------|
| App | http://localhost:8000 | Laravel (يُعدّ نفسه تلقائياً) |
| MySQL | `localhost:3306` | قاعدة البيانات |
| Redis | `localhost:6379` | Cache / Session / Queue |

أول تشغيل يأخذ ~2 دقيقة (composer install + npm install + build + migrations).
بعدها التشغيل فوري.

## الأوامر الأساسية

```bash
# تشغيل
docker compose up -d --build

# إيقاف
docker compose down

# إيقاف مع حذف البيانات (reset كامل)
docker compose down -v

# مشاهدة اللوقات
docker compose logs app --tail 50 -f

# تشغيل أوامر artisan
docker compose exec app php artisan migrate
docker compose exec app php artisan tinker

# تشغيل الاختبارات
docker compose exec app php artisan test

# تشغيل Vite dev server (اختياري — لتطوير الواجهة)
docker compose --profile dev up vite
```

## البنية

```
docker/
  Dockerfile           # PHP 8.2 + phpredis + Node 20 + Composer
  entrypoint.sh        # إعداد تلقائي (deps, key, migrations)
  queue-entrypoint.sh  # تشغيل queue worker
docker-compose.yml     # 5 خدمات: app, queue, vite, mysql, redis
.dockerignore
```

---

## قرار Redis

### ما الذي تغيّر؟

Cache و Session و Queue كانت تعمل على `database` driver. تم نقلها إلى **Redis**:

```
CACHE_STORE=redis
SESSION_DRIVER=redis
QUEUE_CONNECTION=redis
```

### لماذا؟

- **Cache**: بدل ما كل cache read/write يضرب قاعدة البيانات، Redis يخدمها من الذاكرة (أسرع بـ 10-100x)
- **Session**: كل request كان يقرأ/يكتب session في DB — الآن في الذاكرة
- **Queue**: معالجة الـ Jobs أسرع ولا تنافس queries التطبيق على DB

### كيف يعمل؟

- **داخل Docker**: الـ `docker-compose.yml` يضبط كل شيء تلقائياً. لا تحتاج تعديل `.env`
- **بدون Docker (محلي)**: شغّل Redis عبر Docker فقط: `docker run -d -p 6379:6379 redis:7-alpine`، ثم عدّل `.env` كما في القسم التالي

### Redis Client

نستخدم `predis` (مكتبة PHP خالصة) بدلاً من `phpredis` extension:
- لا يحتاج تثبيت extension على الجهاز
- يعمل على أي بيئة PHP فوراً
- الـ Docker image فيها `phpredis` أيضاً لو أردت التبديل

---

## ملف `.env` الأساسي

انسخ هذا الملف إلى `.env` في جذر المشروع.
**إذا تستخدم Docker**: لا تحتاج تعديل شيء — `docker-compose.yml` يتجاوز قيم DB/Redis تلقائياً.
**إذا تشتغل محلي**: عدّل قيم `DB_*` و `REDIS_HOST` حسب بيئتك.

```env
APP_NAME=NUBL
APP_ENV=local
APP_KEY=base64:B/ejm3V72X7EG2Fje6IV3pwh8BqY7/DWfafC3SiUxq4=
APP_DEBUG=true


# Use the same host you open in the browser (http://127.0.0.1:8000 vs http://localhost:8000 are different sites for cookies).
APP_URL=http://localhost.com:8000
# ASSET_URL=http://nubl.com:8000
# Trust proxy headers (ngrok, Cloudflare Tunnel, Reflect, etc.). Use * only in dev; comma-separated IPs in production.
# TRUSTED_PROXIES=*
# enable this secure cookie for production and make app url https
#   SESSION_SECURE_COOKIE = true;

APP_LOCALE=en
APP_FALLBACK_LOCALE=en
APP_FAKER_LOCALE=en_US

APP_MAINTENANCE_DRIVER=file
# APP_MAINTENANCE_STORE=database
# PHP_CLI_SERVER_WORKERS=4

# DEBUGBAR_COLLECTORS_MAIL=true

BCRYPT_ROUNDS=12

LOG_CHANNEL=stack
LOG_STACK=single
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nubl
DB_USERNAME=root
DB_PASSWORD=

# MyFatoorah Payment Gateway (v2 API)Configuration
## SANDBOX Kuwait API KEY
MYFATOORAH_API_KEY=SK_KWT_vVZlnnAqu8jRByOWaRPNId4ShzEDNt256dvnjebuyzo52dXjAfRx2ixW5umjWSUx
MYFATOORAH_COUNTRY_CODE=SAU
MYFATOORAH_IS_TEST=true

## LIVE Saudi Arabia API KEY (production)
# MYFATOORAH_API_KEY= 5aTPjtQ_hI4GcGDo5iE3sn63GfKsg_21L7menpg-Ht-ksmlOQV9lXYrz5v1EDMShyYHEemLoCnu786WdDEl-5faXhW7EeLrbbtfmrp_P_X0yya3yf3eoHiMxmYnlu0f4iJAcAXdO-ZGYsuEJZdfetQSMr6g8suAOF4MKjL5I8fV7zLNk-7R_lBXCJbUyc7NkRwGWzz2NmGgRPqMkpl1azKu0f2je1HPjuR73496pZJz5OZ-uCzouTFLaTAl3bPhx79seZgp2dZGQ_cmGdvGGnuAqkfkZInvEld25gn-f1OtJLFyEpCEqKl6n1Rsoude4tjq5x2-tkqS2rbpiSA6JYXNWNYHWc7pg4ePu_HAhLx8yz0HCVbO43WQhqAX1mnDini0DbvHkcM_o1DGZZMWwihuwX-hp8otVsqY92RggLb8TMHLMSZPong-vHrjpbcd77xEfbRrB30h4mHo70IL4GdWZrrU2uQYh7bWSvTNtN4YBAOh7yNlSEEkRGmHLBqlk-O4onOGO1gyYceU7b0ssz1q2zxRb0Z0rLbmU34AAMClaxrpVi-FCdFmAMvYmkJA8ec8axfqv3aPO4qX2q6fy7cvL9HOzBKefz6_A9vLpcfeHP9PoDpby5PAYKssVem_vgbfWhmHjX8KqANmvRXC15dqx8LFlLCUG5kUlCusU7QcV1eEZGfAkxwIjHWpzZcvdAgMQRDhipA0fFX_PTB1QVTH0QcGNqSpW-y0RSgYWORrvSD4-
# MYFATOORAH_COUNTRY_CODE=SAU

## $$ FOR REVIEW $$ ##
# File Storage
#FILESYSTEM_DISK=public

# Phone Verification Taqnyat Configuration
PHONE_VERIFICATION_ENABLED=false
TAQNYAT_BEARER_TOKEN=
TAQNYAT_SENDER_NAME=Etmaam

SESSION_DRIVER=redis
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=
SESSION_SAME_SITE=lax

BROADCAST_CONNECTION=log
FILESYSTEM_DISK=local
QUEUE_CONNECTION=redis

CACHE_STORE=redis
# CACHE_PREFIX=

MEMCACHED_HOST=127.0.0.1

REDIS_CLIENT=predis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

# Email Verification
# Set to true to require email verification, false to disable
EMAIL_VERIFICATION_ENABLED=false

# Mail Configuration
# For Development: Use Mailtrap (https://mailtrap.io) - Free for testing
# Get credentials from: Mailtrap Dashboard → Email Testing → Inboxes → SMTP Settings
MAIL_MAILER=log
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=#
MAIL_PASSWORD=#
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@nubl.com"
MAIL_FROM_NAME="${APP_NAME}"


VITE_APP_NAME="${APP_NAME}"


# HTTP rate limiting (named limiters; see config/rate_limiting.php and docs/RATE_LIMITING.md)
RATE_LIMITING_ENABLED=false



```
