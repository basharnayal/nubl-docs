# Time-Manipulation Mechanism — Integration Guide

## Files Provided

| File | Purpose |
|---|---|
| `app/Http/Middleware/ApplyMockedTime.php` | Runs on every request; rehydrates `Carbon::setTestNow()` from cache |
| `app/Http/Middleware/TestingEnvironmentOnly.php` | Guards the control endpoints; 404s in production |
| `app/Http/Controllers/Testing/TimeController.php` | The four control actions (show / set / advance / reset) |
| `routes/testing.php` | Route definitions for the control endpoints |

---

## Step 1 – Add the optional token to `.env`

```dotenv
# Leave empty to skip token checking (local dev only).
# In CI / staging, set this to a long random string and pass it as a header.
TESTING_TIME_TOKEN=replace-with-a-long-random-secret
```

Add the key to `config/app.php` so it is accessible via `config()`:

```php
// config/app.php  – inside the return array
'testing_time_token' => env('TESTING_TIME_TOKEN', ''),
```

---

## Step 2 – Register the global middleware

### Laravel 11 + (`bootstrap/app.php`)

Register `ApplyMockedTime` in the global stack. Do **not** wrap it in an environment check here — the middleware has its own production guard built in.

```php
use App\Http\Middleware\ApplyMockedTime;

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(ApplyMockedTime::class);
})
```

> **Important:** Do not use `app()->environment()` inside `withMiddleware` to conditionally append the middleware — it may be evaluated before the application environment is fully resolved, causing the middleware to be silently skipped.

### Laravel 10 and below (`app/Http/Kernel.php`)

Add `ApplyMockedTime` to the `$middleware` array (runs on every request):

```php
protected $middleware = [
    // ... existing entries ...
    \App\Http\Middleware\ApplyMockedTime::class,
];
```

---

## Step 3 – Load the testing routes

### Laravel 11 + (`bootstrap/app.php`)

Use the `using:` callback to take full control of route registration. You must register all route groups (web, api, etc.) yourself inside the callback — the shorthand `web:` and `api:` parameters cannot be combined with `using:`.

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    using: function () {
        Route::middleware('web')
             ->group(base_path('routes/web.php'));

        if (! app()->environment('production')) {
            Route::middleware('api')
                 ->group(base_path('routes/testing.php'));
        }
    },
)
```

### Laravel 10 and below (`app/Providers/RouteServiceProvider.php`)

Inside the `boot()` method, alongside your existing `Route::middleware(...)->group(...)` calls:

```php
if (! $this->app->environment('production')) {
    Route::middleware('api')
         ->group(base_path('routes/testing.php'));
}
```

---

## Step 4 – Verify in production

Deploy and confirm:

```bash
curl -I https://your-production-domain.com/_testing/time
# Expected: HTTP 404
```

---

## API Reference

All endpoints live under `/_testing/time`.  
Pass `X-Testing-Token: <your-token>` if `TESTING_TIME_TOKEN` is set.

### `GET /_testing/time`

Returns the current application time and whether a mock is active.

```json
{
  "message": "Time is currently mocked.",
  "is_mocked": true,
  "mocked_time": "2026-01-15T09:00:00+00:00",
  "real_time": "2026-04-22T00:28:33+00:00"
}
```

### `POST /_testing/time/set`

Sets the clock to an absolute datetime.

```json
{ "datetime": "2026-06-01 09:00:00" }
```

### `POST /_testing/time/advance` · `GET /_testing/time/advance`

Advances the clock by the given duration (combine units freely).

**POST** — send as JSON body:
```json
{ "days": 30 }
{ "hours": 3, "minutes": 30 }
{ "years": 1 }
```

**GET** — pass as query parameters (browser-friendly):
```
/_testing/time/advance?hours=5
/_testing/time/advance?days=1&hours=3&minutes=30
```

### `POST /_testing/time/reset` · `GET /_testing/time/reset`

Clears the mock; the app returns to the real system clock. Both `POST` and `GET` are accepted.

---

## Known behaviour: sessions and mocked time

Because `ApplyMockedTime` runs in the global middleware stack (before `StartSession`), advancing the clock by more than the session lifetime (default: 120 minutes) will cause authenticated users to be logged out on their next request.

**Workaround for testing:** log in again after advancing time, or advance time in smaller increments and refresh the session between steps.

This is expected — the session system correctly sees the mocked time as the current time.

---

## How it works end-to-end

```
Automation tool / browser
     │
     ▼
POST /_testing/time/advance  { "hours": 5 }
  — or —
GET  /_testing/time/advance?hours=5
     │
     ├─ TestingEnvironmentOnly middleware → 404 in production ✓
     ├─ TimeController::advance()
     │     └─ real_time captured via time() before Carbon is mocked
     │     └─ Cache::forever('testing:mocked_time', '<new-time>')
     │     └─ Carbon::setTestNow(...)  ← affects this request
     ▼
Next business request (e.g. GET /subscriptions/status)
     │
     ├─ ApplyMockedTime middleware (global)
     │     └─ Cache::get('testing:mocked_time')  → '<new-time>'
     │     └─ Carbon::setTestNow(Carbon::parse(...))
     │
     └─ Your application code sees Carbon::now() === mocked time ✓
```

> **Why `time()` for `real_time`?**  
> `Carbon::setTestNow()` overrides all `Carbon::now()` calls. To return the
> true wall-clock time in the API response, `real_time` is captured using
> PHP's native `time()` function, which Carbon does not mock.

> **Why the cache?**  
> PHP shares no memory between HTTP requests. Storing the mock in the cache
> (file, Redis, Memcached – whatever `CACHE_DRIVER` is set to) is the only
> way to make the time mock persist across the stateless request boundary.

---

## Teardown

Always call `POST /_testing/time/reset` (or `GET /_testing/time/reset`) in your test suite's after-all hook so the mock does not leak into unrelated tests.
