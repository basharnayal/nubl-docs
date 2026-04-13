# Centralized HTTP rate limiting

## Summary

The app uses **named rate limiters** registered with `RateLimiter::for()` in `AppServiceProvider::configureRateLimiting()`. Routes apply them via the standard `throttle:{name}` middleware, grouped in `routes/auth.php` and `routes/web.php` so limits stay DRY and tunable.

A single flag **`RATE_LIMITING_ENABLED`** (see `config/rate_limiting.php`) disables **all** HTTP throttling at once by making every named limiter return `Limit::none()`.

**No Redis-specific throttle middleware** is used; limits use the default cache store (e.g. database or file as configured).

## Architecture

1. **`config/rate_limiting.php`**  
   - `enabled` from `RATE_LIMITING_ENABLED`  
   - Per-limiter numeric settings (optional env overrides documented below).

2. **`AppServiceProvider::configureRateLimiting()`**  
   - Defines limiters: `registration`, `login`, `otp`, `password_reset`, `verification`, `sensitive_auth`, `payments_gateway`, `donor_payments`, `notifications`, `profile_photo`, `application_resubmit`.  
   - If `config('rate_limiting.enabled')` is false, each closure returns `Illuminate\Cache\RateLimiting\Limit::none()`.

3. **Routes**  
   - `routes/auth.php`: nested `Route::middleware('throttle:{name}')->group(...)` for guest and authenticated flows.  
   - `routes/web.php`: `throttle:{name}` on payment callbacks, donor initiate, notifications group, profile photo upload, provider registration GET, application resubmit POST.

4. **Tests**  
   - `phpunit.xml` sets `RATE_LIMITING_ENABLED=false` so feature tests are not flaky.

## Files changed (when this feature was added)

| File | Role |
|------|------|
| `config/rate_limiting.php` | **New** — master switch and limiter parameters |
| `app/Providers/AppServiceProvider.php` | `configureRateLimiting()` + `RateLimiter::for` registrations |
| `routes/auth.php` | Grouped guest/auth routes under named throttle middleware |
| `routes/web.php` | Replaced raw `throttle:20,1` with `throttle:payments_gateway`; added other named throttles |
| `phpunit.xml` | `RATE_LIMITING_ENABLED=false` for the test suite |
| `.env.example` | Documents `RATE_LIMITING_ENABLED` |
| `docs/RATE_LIMITING.md` | This document |

## Recommended `.env` variables

| Variable | Default (via config) | Purpose |
|----------|----------------------|---------|
| `RATE_LIMITING_ENABLED` | `true` | Master switch for all route throttling |
| `RATE_LIMIT_REGISTRATION_DECAY_MINUTES` | `60` | Window for registration limiter |
| `RATE_LIMIT_REGISTRATION_MAX` | `5` | Max registration attempts per window per IP |
| `RATE_LIMIT_LOGIN_PER_MINUTE` | `10` | Login page GET/POST per IP per minute |
| `RATE_LIMIT_OTP_PER_MINUTE` | `6` | OTP request/verify per IP per minute |
| `RATE_LIMIT_PASSWORD_RESET_DECAY_MINUTES` | `60` | Window for forgot/reset password |
| `RATE_LIMIT_PASSWORD_RESET_MAX` | `5` | Max attempts per window per IP |
| `RATE_LIMIT_VERIFICATION_PER_MINUTE` | `6` | Phone/email verification actions per user (or IP if guest) per minute |
| `RATE_LIMIT_SENSITIVE_AUTH_PER_MINUTE` | `10` | Confirm password + password update per user per minute |
| `RATE_LIMIT_PAYMENTS_GATEWAY_PER_MINUTE` | `20` | Public payment callback/error routes per IP per minute |
| `RATE_LIMIT_DONOR_PAYMENTS_PER_MINUTE` | `15` | Donor payment initiation per user per minute |
| `RATE_LIMIT_NOTIFICATIONS_PER_MINUTE` | `120` | Notification index/read endpoints per user per minute |
| `RATE_LIMIT_PROFILE_PHOTO_PER_MINUTE` | `20` | Profile photo upload per user per minute |
| `RATE_LIMIT_APPLICATION_RESUBMIT_DECAY_MINUTES` | `60` | Window for resubmit POST |
| `RATE_LIMIT_APPLICATION_RESUBMIT_MAX` | `10` | Max resubmit POSTs per window per user |

Only `RATE_LIMITING_ENABLED` is required in `.env`; the rest are optional overrides.

## Routes intentionally left without throttling

- **`/`**, **`/locale/{locale}`**, **`/dashboard`**, role dashboards, most **GET** CRUD/admin pages — normal browsing; abuse is lower priority than auth endpoints.  
- **`GET /approval-pending`**, **`GET /application/resubmit`**, **`GET /application/my-file/{type}`** — reads; throttle only on **POST** resubmit.  
- **`/test-roles`**, **`/make-me-admin`** — development helpers; remove or protect in production separately.  
- **`GET /verify-email`** (notice) — **not** in the verification throttle group; POST/resend and signed link are throttled.  
- **`POST /logout`** — low risk; no throttle.  
- **Provider QR redeem** — still uses **controller-level** `RateLimiter` in `ProviderQrController` (unchanged).  
- **`LoginRequest`** — still applies **email+IP** attempt limiting for failed **password** login (unchanged; complements `throttle:login`).

## How it works

1. **Request** hits a route with `throttle:registration` (or another name).  
2. **Throttle middleware** resolves the named limiter from `RateLimiter`.  
3. The registered closure returns a **`Limit`** (or **`Limit::none()`** when disabled).  
4. Laravel uses the **cache** store to track counters (keyed by `->by(...)` — IP, user id, or composite).  
5. When exceeded, the response is **429 Too Many Requests** with standard headers.

**Disabling globally:** set `RATE_LIMITING_ENABLED=false`. All limiters return `Limit::none()`, so every `throttle:{name}` middleware becomes a no-op for limiting (still registered, minimal overhead).

## References

- Laravel: [Rate limiting (routing)](https://laravel.com/docs/routing#rate-limiting)  
- Laravel: [RateLimiter facade](https://laravel.com/docs/rate-limiting)
