# Playwright End-to-End Tests

## نظرة سريعة

اختبارات **Playwright E2E** في المشروع تفتح المتصفح تلقائيًا، تنفّذ خطوات المستخدم، ثم تتحقق من الصفحات والنتائج.  
ملفات الاختبار موجودة داخل:

==================دليل مختصر للاستخدام==================
## تشغيل كل الاختبارات

### مع UI
```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --ui
```

### بدون UI
```bash
npm run test:e2e:php-only
```

### مع متصفح ظاهر
```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --headed
```

---

## تشغيل اختبار واحد فقط

### الصيغة العامة
```bash
npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js --headed
```

---

## أمثلة

### example
```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### landing page
```bash
npx playwright test tests/e2e/landing-page.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### register
```bash
npx playwright test tests/e2e/register.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### auth registration + login
```bash
npx playwright test tests/e2e/auth-registration-email-login.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### admin dashboard
```bash
npx playwright test tests/e2e/admin-dashboard.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### donor flow
```bash
npx playwright test tests/e2e/donor-flow.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### recipient request provider menu
```bash
npx playwright test tests/e2e/recipient-request-provider-menu.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

---

## الأفضل للاستخدام اليومي

- إذا تريد تشغيل بصري ومريح: استخدم `--ui`
- إذا تريد تشغيل سريع لكل الاختبارات: استخدم `npm run test:e2e:php-only`
- إذا تريد تجربة ملف واحد فقط: استخدم أمر الملف مع `--headed`

---

## عرض التقرير بعد التشغيل
```bash
npx playwright show-report
```
==================نهاية الدليل المختصر==================


```bash
tests/e2e/
```

وجميع الأوامر التالية تُنفّذ من **جذر المشروع**:

```bash
C:\Users\basha\SE\nubl
```

> **مهم:** لا يوجد ملف `playwright.config.js` في جذر المشروع، لذلك عند استخدام Playwright مباشرة يجب تحديد مسار الـ config صراحة.

---

## المتطلبات قبل التشغيل

نفّذ هذه الأوامر **مرة واحدة فقط** بعد استنساخ المشروع أو عند إعداد البيئة لأول مرة:

```bash
npm install
npx playwright install chromium
```

يمكنك أيضًا تثبيت جميع المتصفحات المضمنة إذا احتجتها:

```bash
npx playwright install
```

> تأكد أيضًا أن ملف `.env` وقاعدة البيانات جاهزان إذا كان التطبيق يحتاجهما أثناء الاختبارات.

### ملاحظات مهمة قبل التشغيل

- `@playwright/test` موجود كـ devDependency، لذلك بدون `npm install` قد يظهر خطأ مثل:
  ```text
  Error [ERR_MODULE_NOT_FOUND]: Cannot find package '@playwright/test'
  ```

- إذا كانت ملفات المتصفح غير مثبتة، قد يظهر خطأ من نوع:
  ```text
  Executable doesn't exist
  ```

- في **Windows PowerShell** القديم، قد لا يعمل `&&` بين الأوامر. شغّل كل أمر في سطر مستقل أو استخدم `;`.

---

## Quick check — هل Playwright يعمل فعلاً؟

بعد تثبيت الحزم والمتصفح، جرّب أولًا اختبارًا بسيطًا جدًا للتأكد أن كل شيء يعمل.

### Laravel only (بدون Vite)

```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js
```

### Full stack (Laravel + Vite)

```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.config.js
```

إذا نجح `example.spec.js` فهذه علامة جيدة أن الإعداد الأساسي سليم قبل الانتقال لبقية الاختبارات.

---

## أوضاع التشغيل المتاحة

يوجد إعدادان رئيسيان لتشغيل الاختبارات:

### 1) Laravel فقط — PHP-only
هذا هو الخيار **الأخف والأكثر استقرارًا** في أغلب الحالات، لأنه لا يعتمد على Vite.

```bash
config/playwright/playwright.php-only.config.js
```

### 2) Laravel + Vite
هذا هو الوضع الكامل للتطبيق.

```bash
config/playwright/playwright.config.js
```

> **مهم:** تشغيل Laravel + Vite يحتاج **Node.js 20.19+** أو **22.12+** لأن المشروع يستخدم **Vite 7**.  
> إذا كانت نسخة Node أقدم، قد يتوقف `npm run dev` مباشرة أو يفشل Playwright في انتظار `webServer`.

---

## ما الذي يفعله كل Config؟

| الملف | الوظيفة |
|---|---|
| `config/playwright/playwright.config.js` | الإعداد الافتراضي الكامل. يشغّل Laravel على `127.0.0.1:8001` **ويشغّل Vite أيضًا** |
| `config/playwright/playwright.php-only.config.js` | يشغّل **Laravel فقط** على `127.0.0.1:8001` بدون Vite |

### أشياء مشتركة بين الإعدادين
كلا الملفين يضبطان أشياء مهمة مثل:
- `testDir = tests/e2e`
- `baseURL = http://127.0.0.1:8001`
- تعطيل:
  - `PHONE_VERIFICATION_ENABLED=false`
  - `EMAIL_VERIFICATION_ENABLED=false`

وهذا يجعل اختبارات E2E أكثر استقرارًا.

---

## التشغيل السريع الموصى به

### تشغيل جميع الاختبارات بواجهة تفاعلية (موصى به)

```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --ui
```

هذا يفتح واجهة Playwright بحيث تستطيع:
- مشاهدة جميع الاختبارات
- تشغيلها واحدًا واحدًا أو كلها
- متابعة كل خطوة
- رؤية لقطات الشاشة والتنقلات بسهولة

---

### تشغيل جميع الاختبارات بمتصفح ظاهر

```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --headed
```

هذا يشغّل الاختبارات بشكل عادي لكن مع ظهور المتصفح أمامك.

---

### تشغيل جميع الاختبارات بدون متصفح ظاهر

```bash
npm run test:e2e:php-only
```

---

## تشغيل اختبار واحد فقط

### الصيغة العامة

```bash
npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js
```

### مع متصفح ظاهر

```bash
npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js --headed
```

---

## أمثلة جاهزة

### smoke test
```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### الصفحة الرئيسية
```bash
npx playwright test tests/e2e/landing-page.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### تسجيل donor
```bash
npx playwright test tests/e2e/register.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### تسجيل + تسجيل دخول
```bash
npx playwright test tests/e2e/auth-registration-email-login.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### لوحة الأدمن
```bash
npx playwright test tests/e2e/admin-dashboard.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### رحلة المتبرع
```bash
npx playwright test tests/e2e/donor-flow.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

### طلب المستفيد من قائمة المزود
```bash
npx playwright test tests/e2e/recipient-request-provider-menu.spec.js --config=config/playwright/playwright.php-only.config.js --headed
```

---

## ترتيب الملفات المنطقي للتشغيل

| # | الملف | ما يغطيه |
|---|---|---|
| 1 | `tests/e2e/example.spec.js` | smoke test — التأكد أن السيرفر يعمل |
| 2 | `tests/e2e/landing-page.spec.js` | الصفحة الرئيسية |
| 3 | `tests/e2e/register.spec.js` | تسجيل donor |
| 4 | `tests/e2e/auth-registration-email-login.spec.js` | التسجيل + تسجيل الدخول للأدوار |
| 5 | `tests/e2e/admin-dashboard.spec.js` | لوحة تحكم الأدمن |
| 6 | `tests/e2e/donor-flow.spec.js` | رحلة المتبرع |
| 7 | `tests/e2e/recipient-request-provider-menu.spec.js` | طلب المستفيد من قائمة المزود |

---

## تشغيل جميع الاختبارات

### Laravel فقط (موصى به)
```bash
npm run test:e2e:php-only
```

### Laravel + Vite
```bash
npm run test:e2e
```

> هذا الوضع يحتاج Node **20.19+** أو **22.12+** لأنه يشغّل `npm run dev`.

### تشغيل كل الاختبارات بمتصفح ظاهر
```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --headed
```

---

## npm scripts المهمة

| Script | ماذا يشغّل |
|---|---|
| `npm run test:e2e` | `playwright test --config=config/playwright/playwright.config.js` |
| `npm run test:e2e:php-only` | `playwright test --config=config/playwright/playwright.php-only.config.js` |
| `npm run test:e2e:ui` | تشغيل Playwright في UI mode |
| `npm run test:e2e:headed` | تشغيل Playwright بمتصفح ظاهر |

---

## أوامر مفيدة للتصحيح والمتابعة

### واجهة Playwright التفاعلية
```bash
npm run test:e2e:ui
```

### عرض تقرير Playwright بعد التشغيل
```bash
npx playwright show-report
```

### تشغيل مع debug
```bash
npx playwright test --debug --config=config/playwright/playwright.php-only.config.js
```

---

## baseURL والتنقل داخل الاختبارات

المشروع يستخدم `baseURL` داخل إعدادات Playwright، لذلك يفضّل داخل الاختبارات استخدام روابط نسبية مثل:

```js
await page.goto('/login');
```

بدلًا من كتابة الرابط كاملًا.

> إذا غيّرت البورتات لاحقًا، حدّث معًا:
> - `baseURL`
> - `webServer.url`
> - `webServer.port`
> - `webServer.command`

حتى تبقى الإعدادات متناسقة.

---

## لماذا يستخدم Vite readiness عبر port وليس URL؟

هذه نقطة مهمة جدًا في هذا المشروع.

في Playwright، عندما تضع `webServer.url` فهو ينتظر أن يرجع السيرفر HTTP status من **200 إلى 403** حتى يعتبره جاهزًا.

لكن في مشاريع **Laravel + Vite** غالبًا خادم Vite يرجّع:

```text
404
```

على المسار `/`

وهذا يجعل Playwright يظن أن السيرفر **ليس جاهزًا** ويستمر في الانتظار حتى timeout.

لذلك هذا المشروع يستخدم **`port: 5173`** بدل انتظار URL مباشرة، حتى ينتظر فقط أن منفذ Vite أصبح جاهزًا لاستقبال الاتصال.

---

## webServer و reuseExistingServer

كلا الـ configs يستخدمان عادة:

```js
reuseExistingServer: true
```

وهذا يعني:
- إذا كان `php artisan serve` أو Vite يعمل أصلًا، فسيعيد Playwright استخدامه
- ولن يفشل مباشرة بسبب تعارض المنافذ

هذا مناسب جدًا محليًا.  
أما في CI، فقد تفضّل أحيانًا جعله `false` إذا أردت بيئة نظيفة تمامًا في كل تشغيل.

---

## CI / retries / workers

في الإعداد الافتراضي قد ترى منطقًا مثل:

```js
retries: process.env.CI ? 2 : 0
workers: process.env.CI ? 1 : undefined
```

المعنى:
- محليًا: بدون retries غالبًا
- في CI: يعيد المحاولة عند الفشل، ويقلل عدد الـ workers لتحسين الاستقرار

إذا كان نظام الـ CI عندك لا يضبط `CI=true` تلقائيًا، قد تحتاج تكييف هذا السلوك يدويًا.

---

## traces والتصحيح

في Playwright قد يكون مفعّلًا:

```js
trace: 'on-first-retry'
```

وهذا يعني أنه يسجل trace عند أول retry، وهو مفيد جدًا لفهم الاختبارات المتذبذبة.

للتصحيح المحلي يمكنك أيضًا استخدام:
- `--debug`
- `npm run test:e2e:ui`
- أو مؤقتًا تفعيل `trace: 'on'`

---

## HTML Report

عادة يكون:

```js
reporter: 'html'
```

مفعّلًا في إعدادات المشروع.

بعد انتهاء التشغيل يمكنك:
- فتح المسار الذي يطبعه Playwright
- أو تشغيل:

```bash
npx playwright show-report
```

---

## Environment variables الخاصة بالاختبارات

بعض الإعدادات في هذا المشروع تُمرّر من خلال `webServer.env` بدل تعديل `.env` نفسه، مثل تعطيل:
- التحقق من الهاتف
- التحقق من البريد

يمكنك إضافة مفاتيح أخرى هناك إذا احتجت:
- feature flags
- overrides خاصة بالاختبارات
- سلوك مختلف للبيئة

وذلك بدون تعديل `.env` على القرص.

---

## إضافة متصفحات أو Projects أخرى

إذا أردت مستقبلًا تشغيل الاختبارات على:
- Firefox
- WebKit

ثبّت المتصفحات:

```bash
npx playwright install
```

ثم أضف المشاريع المطلوبة داخل `projects` في ملف config.

---

## هل أحتاج تشغيل السيرفر يدويًا؟

في الغالب **لا**.  
إعدادات Playwright في المشروع تشغّل السيرفر تلقائيًا أو تعيد استخدامه إذا كان يعمل بالفعل.

لكن يجب التأكد من:
- وجود ملف `.env`
- جاهزية قاعدة البيانات
- عمل التطبيق محليًا إذا كانت الاختبارات تعتمد على بيانات أو جداول معينة

---

## Troubleshooting

| المشكلة | ماذا تفعل |
|---|---|
| `Cannot find package '@playwright/test'` | نفّذ `npm install` من جذر المشروع |
| Browser executable missing | نفّذ `npx playwright install` أو `npx playwright install chromium` |
| `Timed out waiting ... webServer` عند default config | غالبًا نسخة Node قديمة بالنسبة لـ Vite 7. حدّث Node إلى `20.19+` أو `22.12+` أو استخدم `php-only` config |
| `[WebServer] Vite requires Node.js version 20.19+ or 22.12+` | نفس الحل: حدّث Node أو استخدم `config/playwright/playwright.php-only.config.js` |
| السيرفر لا يستجيب أو المنفذ خطأ | تأكد من `baseURL` والبورتات ومن عدم وجود تعارض على نفس المنفذ |
| الاختبارات تعمل محليًا وتفشل في CI | تأكد من Node المناسب، تشغيل `npm install` أو `npm ci`، تثبيت Playwright browsers، وضبط retries/workers بشكل صحيح |
| `npm run test:e2e` يفشل بعد ترقية Node | راجع readiness الخاص بـ Vite وتأكد أن الانتظار ليس مبنيًا على `GET /` إذا كان Vite يعيد `404` |

---

## ملاحظات عملية مهمة

- إذا كنت تريد **أخف وأبسط تشغيل** فاستخدم:
  ```bash
  npm run test:e2e:php-only
  ```

- إذا كنت تريد **مراقبة الخطوات بصريًا** فاستخدم:
  ```bash
  npx playwright test --config=config/playwright/playwright.php-only.config.js --ui
  ```

- إذا كنت تريد اختبار ملف واحد فقط بسرعة مع ظهور المتصفح:
  ```bash
  npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js --headed
  ```

- لا تنسَ أن بعض الاختبارات تعتمد على:
  - قاعدة البيانات
  - بيانات seeded
  - أدوار وصلاحيات
  - جاهزية التطبيق محليًا

---

## أفضل طريقة استخدام يومية

للاستخدام اليومي، هذا هو الترتيب الأفضل غالبًا:

1. جرّب أولًا:
   ```bash
   npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js
   ```

2. ثم شغّل الواجهة التفاعلية:
   ```bash
   npx playwright test --config=config/playwright/playwright.php-only.config.js --ui
   ```

3. ابدأ بالملفات بهذا الترتيب:
   - `example.spec.js`
   - `landing-page.spec.js`
   - `register.spec.js`
   - `auth-registration-email-login.spec.js`
   - `admin-dashboard.spec.js`
   - `donor-flow.spec.js`
   - `recipient-request-provider-menu.spec.js`

4. عند الحاجة لتشغيل اختبار واحد فقط:
   ```bash
   npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js --headed
   ```

---

## ملخص سريع جدًا

### تثبيت أول مرة
```bash
npm install
npx playwright install chromium
```

### فحص سريع
```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js
```

### تشغيل الكل بواجهة
```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --ui
```

### تشغيل الكل سريعًا
```bash
npm run test:e2e:php-only
```

### تشغيل ملف واحد
```bash
npx playwright test tests/e2e/<اسم-الملف> --config=config/playwright/playwright.php-only.config.js --headed
```

### عرض التقرير
```bash
npx playwright show-report
```


---

This project uses [Playwright](https://playwright.dev/) for browser tests under `tests/e2e/`. This guide covers setup, which config to use, and practical configuration tips.

---

## Quick check: is it working?

From the **repository root**, after `npm install` and `npx playwright install chromium`:

**Laravel only (no Vite):**

```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.php-only.config.js
```

**Full stack (Laravel + Vite, default config):**

```bash
npx playwright test tests/e2e/example.spec.js --config=config/playwright/playwright.config.js
```

The default config needs **Node.js 20.19+ or 22.12+** so `npm run dev` (Vite 7) can start. If `npm run test:e2e` still fails with `Timed out waiting ... webServer` after upgrading Node, see [Vite `webServer` readiness](#vite-webserver-readiness-port-vs-url) and [Troubleshooting](#troubleshooting).

**Windows PowerShell:** older shells do not support `&&` between commands; run each command on its own line, or use `;` (e.g. `cd path; npm install`).

---

## First-time setup

Run these from the **repository root** (where `package.json` lives).

### 1. Install Node dependencies

`@playwright/test` is a devDependency. Without installing packages, the config cannot import `@playwright/test` and you may see:

`Error [ERR_MODULE_NOT_FOUND]: Cannot find package '@playwright/test'`

```bash
npm install
```

### 2. Install browser binaries

Playwright downloads Chromium (and optionally other browsers) outside `node_modules`. If browsers are missing, you will see errors like `Executable doesn't exist` under `%LOCALAPPDATA%\ms-playwright\` on Windows.

Install Chromium only (matches the default project in our configs):

```bash
npx playwright install chromium
```

Install all bundled browsers:

```bash
npx playwright install
```

### 3. Application prerequisites

- **PHP / Laravel**: Tests expect the app to be reachable at the configured `baseURL` (see below). The Playwright configs can start `php artisan serve` for you; you still need a valid `.env`, database if the app requires it, etc.
- **Node (default E2E config only)**: This repo uses **Vite 7**, which requires **Node.js 20.19+ or 22.12+**. Below that, `npm run dev` may exit immediately. Use **`config/playwright/playwright.php-only.config.js`** (or `npm run test:e2e:php-only`) until you upgrade Node, or run a production build if your tests do not need the dev server. The default `config/playwright/playwright.config.js` marks Vite as ready using **`port: 5173`** (TCP), not an HTTP probe on `/`, because the Vite dev server often returns **404** for `GET /`, which Playwright would otherwise treat as "not ready" until timeout.
- **Node**: If npm prints `EBADENGINE` warnings, upgrade Node to satisfy Vite as above.

---

## Config files

| File | Purpose |
|------|---------|
| `config/playwright/playwright.config.js` | **Default.** Starts Laravel on `127.0.0.1:8001` **and** Vite (`npm run dev`). Vite readiness uses **`port: 5173`** (see [below](#vite-webserver-readiness-port-vs-url)). |
| `config/playwright/playwright.php-only.config.js` | Starts **only** `php artisan serve` on `127.0.0.1:8001`. Use when you do not need the Vite dev server or want a lighter run. |

Both set:

- `testDir` → `tests/e2e` (resolved relative to the config file location)
- `baseURL: 'http://127.0.0.1:8001'`
- `PHONE_VERIFICATION_ENABLED` and `EMAIL_VERIFICATION_ENABLED` to `false` in the web server environment for more stable tests.

---

## npm scripts

Defined in `package.json`:

| Script | Command |
|--------|---------|
| `npm run test:e2e` | `playwright test --config=config/playwright/playwright.config.js` (needs Node version compatible with Vite 7) |
| `npm run test:e2e:php-only` | `playwright test --config=config/playwright/playwright.php-only.config.js` (Laravel only; works on older Node) |
| `npm run test:e2e:ui` | Playwright UI mode |
| `npm run test:e2e:headed` | Runs with a visible browser |

---

## Choosing a config on the CLI

Default config (explicit path — there is no `playwright.config.js` in the repo root):

```bash
npx playwright test --config=config/playwright/playwright.config.js
```

PHP-only config (example spec):

```bash
npx playwright test tests/e2e/admin-dashboard.spec.js --config=config/playwright/playwright.php-only.config.js
```

Add `--headed` to see the browser:

```bash
npx playwright test --config=config/playwright/playwright.php-only.config.js --headed
```

---

## Configuration tips

### `baseURL` and navigation

`baseURL` is set in `use.baseURL`. In tests, prefer relative URLs:

```js
await page.goto('/login');
```

If you change the Laravel or Vite port, update `baseURL` and `webServer.url` / `webServer.port` / `webServer.command` together so they stay consistent.

### Vite `webServer` readiness (port vs URL)

Playwright considers an HTTP `webServer` "ready" when the URL returns a status code **from 200 up to 403** (not 404 or 5xx). The Vite dev server commonly returns **404** for `GET /` in a Laravel + Vite setup, so probing `http://localhost:5173/` can hang until **timeout**. Our default config uses **`port: 5173`** for the Vite entry so Playwright waits for the **TCP port** to accept connections instead. See [Playwright web server](https://playwright.dev/docs/test-webserver).

### `webServer` and `reuseExistingServer`

Both configs use `reuseExistingServer: true`. If you already have `php artisan serve` (and optionally Vite) running, Playwright will reuse them instead of failing on port conflicts. Set to `false` in CI if you need a guaranteed clean server each run.

### CI vs local (`retries`, `workers`)

The default `config/playwright/playwright.config.js` sets:

- `retries: process.env.CI ? 2 : 0`
- `workers: process.env.CI ? 1 : undefined`

Many CI systems set `CI=true` automatically. Adjust if your pipeline uses a different convention.

### Traces and debugging

`trace: 'on-first-retry'` records a trace when a test fails and is retried—useful for flaky failures. For local debugging you can temporarily use `trace: 'on'` or run with `--debug` / UI mode (`npm run test:e2e:ui`).

### HTML report

`reporter: 'html'` is set in both configs. After a run, open the report path Playwright prints, or run:

```bash
npx playwright show-report
```

### Adding browsers or projects

To run Firefox or WebKit, install them (`npx playwright install`) and add entries under `projects` in `config/playwright/playwright.config.js`, following [Playwright docs on projects](https://playwright.dev/docs/test-projects).

### Environment variables for the app under test

Verification flags are disabled in the `webServer.env` block for predictable E2E runs. Add other keys there if tests need specific feature flags or `.env`-style overrides **without** editing `.env` on disk.

---

## Troubleshooting

| Symptom | What to try |
|---------|-------------|
| `Cannot find package '@playwright/test'` | Run `npm install` in the project root. |
| Browser executable missing | Run `npx playwright install` or `npx playwright install chromium`. |
| `Timed out waiting ... webServer` (default config) | **Node too old for Vite 7** (`npm run dev` exits): upgrade to **20.19+** or **22.12+**, or use `config/playwright/playwright.php-only.config.js` / `npm run test:e2e:php-only`. If Node is fine, ensure the Vite `webServer` does not rely on HTTP `GET /` (404 breaks Playwright's readiness check); this repo uses **`port: 5173`** for Vite. |
| `[WebServer] Vite requires Node.js version 20.19+ or 22.12+` | Same as above: upgrade Node, or use the PHP-only Playwright config. |
| Connection refused / wrong port | Match `baseURL` and server commands; ensure nothing else uses the same port or stop the conflicting process. |
| Tests pass locally but fail in CI | Ensure CI runs `npm ci` or `npm install`, `npx playwright install --with-deps` (Linux) if needed, that the Node version satisfies Vite if you use the default config, and that `CI` env triggers the intended retries/workers. |

---

## Further reading

- [Playwright Test configuration](https://playwright.dev/docs/test-configuration)
- [Web server](https://playwright.dev/docs/test-webserver)
