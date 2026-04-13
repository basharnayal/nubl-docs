# Playwright end-to-end tests
npx playwright test tests/e2e/admin-dashboard.spec.js

## تشغيل Playwright والتعامل معه (عربي)

**ما هو؟** اختبارات تفتح المتصفح وتنقر وتتحقق من الصفحات تلقائياً. ملفات الاختبار في `tests/e2e/` بامتداد `.spec.js`.

**من جذر المشروع** (المجلد الذي فيه `package.json`):

1. **مرة واحدة (أو بعد استنساخ المشروع):**
   - `npm install`
   - `npx playwright install chromium`

2. **تشغيل كل الاختبارات:**
   - **مع Laravel + Vite (الافتراضي):** `npm run test:e2e`  
     يحتاج Node **20.19+** أو **22.12+** لأن الإعداد يشغّل `npm run dev`.
   - **Laravel فقط (أخف، بدون Vite):** `npm run test:e2e:php-only`

3. **تشغيل ملف واحد** (مثال):
   - `npx playwright test tests/e2e/example.spec.js`
   - مع Laravel فقط: أضف `--config=config/playwright/playwright.php-only.config.js`

4. **واجهة تفاعلية أو متصفح ظاهر:**
   - `npm run test:e2e:ui` — واجهة Playwright للتشغيل والخطوات.
   - `npm run test:e2e:headed` — تشغيل عادي لكن المتصفح مرئي.

5. **السيرفر:** عادة **لا تحتاج** تشغيل `php artisan serve` يدوياً؛ الإعداد يشغّله (أو يعيد استخدامه إن كان يعمل). تأكد أن `.env` وقاعدة البيانات جاهزة إن احتاجها التطبيق.

6. **بعد التشغيل:** إن وُجد تقرير HTML، يمكن فتحه أو تنفيذ `npx playwright show-report`.

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
- **Node (default E2E config only)**: This repo uses **Vite 7**, which requires **Node.js 20.19+ or 22.12+**. Below that, `npm run dev` may exit immediately. Use **`config/playwright/playwright.php-only.config.js`** (or `npm run test:e2e:php-only`) until you upgrade Node, or run a production build if your tests do not need the dev server. The default `config/playwright/playwright.config.js` marks Vite as ready using **`port: 5173`** (TCP), not an HTTP probe on `/`, because the Vite dev server often returns **404** for `GET /`, which Playwright would otherwise treat as “not ready” until timeout.
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

Playwright considers an HTTP `webServer` “ready” when the URL returns a status code **from 200 up to 403** (not 404 or 5xx). The Vite dev server commonly returns **404** for `GET /` in a Laravel + Vite setup, so probing `http://localhost:5173/` can hang until **timeout**. Our default config uses **`port: 5173`** for the Vite entry so Playwright waits for the **TCP port** to accept connections instead. See [Playwright web server](https://playwright.dev/docs/test-webserver).

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
| `Timed out waiting ... webServer` (default config) | **Node too old for Vite 7** (`npm run dev` exits): upgrade to **20.19+** or **22.12+**, or use `config/playwright/playwright.php-only.config.js` / `npm run test:e2e:php-only`. If Node is fine, ensure the Vite `webServer` does not rely on HTTP `GET /` (404 breaks Playwright’s readiness check); this repo uses **`port: 5173`** for Vite. |
| `[WebServer] Vite requires Node.js version 20.19+ or 22.12+` | Same as above: upgrade Node, or use the PHP-only Playwright config. |
| Connection refused / wrong port | Match `baseURL` and server commands; ensure nothing else uses the same port or stop the conflicting process. |
| Tests pass locally but fail in CI | Ensure CI runs `npm ci` or `npm install`, `npx playwright install --with-deps` (Linux) if needed, that the Node version satisfies Vite if you use the default config, and that `CI` env triggers the intended retries/workers. |

---

## Further reading

- [Playwright Test configuration](https://playwright.dev/docs/test-configuration)
- [Web server](https://playwright.dev/docs/test-webserver)
