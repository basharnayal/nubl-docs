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

```php
use App\Http\Middleware\ApplyMockedTime;

->withMiddleware(function (Middleware $middleware) {
    $middleware->append(ApplyMockedTime::class);   // appends to the global stack
})
```

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

```php
->withRouting(function () {
    // ... your existing route loading ...

    if (! app()->environment('production')) {
        Route::middleware('api')          // or 'web' – whichever suits your app
             ->group(base_path('routes/testing.php'));
    }
})
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
  "is_mocked": true,
  "mocked_time": "2026-01-15T09:00:00+00:00",
  "real_time": "2026-04-21T14:32:00+00:00"
}
```

### `POST /_testing/time/set`

Sets the clock to an absolute datetime.

```json
{ "datetime": "2026-06-01 09:00:00" }
```

### `POST /_testing/time/advance`

Advances the clock by the given duration (combine units freely).

```json
{ "days": 30 }
{ "hours": 3, "minutes": 30 }
{ "years": 1 }
```

### `POST /_testing/time/reset`

Clears the mock; the app returns to the real system clock.

---

## How it works end-to-end

```
Automation tool
     │
     ▼
POST /_testing/time/set  { "datetime": "2026-06-01" }
     │
     ├─ TestingEnvironmentOnly middleware → 404 in production ✓
     ├─ TimeController::set()
     │     └─ Cache::forever('testing:mocked_time', '2026-06-01T00:00:00Z')
     │     └─ Carbon::setTestNow(...)  ← affects this request
     ▼
Next business request (e.g. GET /subscriptions/status)
     │
     ├─ ApplyMockedTime middleware
     │     └─ Cache::get('testing:mocked_time')  → '2026-06-01T00:00:00Z'
     │     └─ Carbon::setTestNow(Carbon::parse(...))
     │
     └─ Your application code sees Carbon::now() === 2026-06-01 ✓
```

> **Why the cache?**  
> PHP shares no memory between HTTP requests. Storing the mock in the cache
> (file, Redis, Memcached – whatever `CACHE_DRIVER` is set to) is the only
> way to make the time mock persist across the stateless request boundary.

---

## Teardown

Always call `POST /_testing/time/reset` in your test suite's after-all hook
so the mock does not leak into unrelated tests.
