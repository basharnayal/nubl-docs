# Playwright end-to-end tests

This project uses [Playwright](https://playwright.dev/) for browser tests under `tests/e2e/`. This guide covers setup, which config to use, and practical configuration tips.

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
- **Node**: If npm prints `EBADENGINE` warnings, consider upgrading Node to match the range requested by Vite / `package.json` engines (if any).

---

## Config files

| File | Purpose |
|------|---------|
| `playwright.config.js` | **Default.** Starts Laravel on `127.0.0.1:8001` **and** Vite (`npm run dev` on port 5173). Use for full-stack flows that need hot assets. |
| `playwright.php-only.config.js` | Starts **only** `php artisan serve` on `127.0.0.1:8001`. Use when you do not need the Vite dev server or want a lighter run. |

Both set:

- `testDir: './tests/e2e'`
- `baseURL: 'http://127.0.0.1:8001'`
- `PHONE_VERIFICATION_ENABLED` and `EMAIL_VERIFICATION_ENABLED` to `false` in the web server environment for more stable tests.

---

## npm scripts

Defined in `package.json`:

| Script | Command |
|--------|---------|
| `npm run test:e2e` | `playwright test` (uses default `playwright.config.js`) |
| `npm run test:e2e:ui` | Playwright UI mode |
| `npm run test:e2e:headed` | Runs with a visible browser |

---

## Choosing a config on the CLI

Default config:

```bash
npx playwright test
```

PHP-only config (example spec):

```bash
npx playwright test tests/e2e/admin-dashboard.spec.js --config=playwright.php-only.config.js
```

Add `--headed` to see the browser:

```bash
npx playwright test --config=playwright.php-only.config.js --headed
```

---

## Configuration tips

### `baseURL` and navigation

`baseURL` is set in `use.baseURL`. In tests, prefer relative URLs:

```js
await page.goto('/login');
```

If you change the Laravel or Vite port, update `baseURL` and `webServer.url` / `webServer.command` together so they stay consistent.

### `webServer` and `reuseExistingServer`

Both configs use `reuseExistingServer: true`. If you already have `php artisan serve` (and optionally Vite) running, Playwright will reuse them instead of failing on port conflicts. Set to `false` in CI if you need a guaranteed clean server each run.

### CI vs local (`retries`, `workers`)

The default `playwright.config.js` sets:

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

To run Firefox or WebKit, install them (`npx playwright install`) and add entries under `projects` in `playwright.config.js`, following [Playwright docs on projects](https://playwright.dev/docs/test-projects).

### Environment variables for the app under test

Verification flags are disabled in the `webServer.env` block for predictable E2E runs. Add other keys there if tests need specific feature flags or `.env`-style overrides **without** editing `.env` on disk.

---

## Troubleshooting

| Symptom | What to try |
|---------|-------------|
| `Cannot find package '@playwright/test'` | Run `npm install` in the project root. |
| Browser executable missing | Run `npx playwright install` or `npx playwright install chromium`. |
| Connection refused / wrong port | Match `baseURL` and server commands; ensure nothing else uses the same port or stop the conflicting process. |
| Tests pass locally but fail in CI | Ensure CI runs `npm ci` or `npm install`, `npx playwright install --with-deps` (Linux) if needed, and that `CI` env triggers the intended retries/workers. |

---

## Further reading

- [Playwright Test configuration](https://playwright.dev/docs/test-configuration)
- [Web server](https://playwright.dev/docs/test-webserver)
