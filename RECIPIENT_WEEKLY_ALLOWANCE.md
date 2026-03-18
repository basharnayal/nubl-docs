# Recipient Weekly Allowance

## Changelog (This Commit)

This commit implements the recipient weekly allowance feature end-to-end:

- **RecipientAllowanceService** – New service class encapsulating weekly limit logic (400 SAR, Sunday–Saturday), with methods for `getWeeklyUsed`, `getRemainingLimit`, and `wouldExceedAllowance`
- **Dashboard** – Dynamic display of remaining allowance and weekly limit instead of hardcoded values
- **Request validation** – New requests are rejected with "Weekly allowance exceeded" when they would exceed the limit; REQUESTED, REDEEMABLE, and FULFILLED requests count toward usage
- **Provider menu / cart** – Cart sidebar shows live allowance usage; JavaScript disables submit when projected total exceeds limit; validation error displayed on rejection
- **FundTransaction model & migration** – New `fund_transactions` table for tracking wallet movements (donations, redemptions, refunds, etc.)
- **Seeders** – `RecipientSeeder`, `ProviderSeeder`, `ProviderMenuItemSeeder` for test data; `AllowanceTestDataSeeder` seeds REDEEMABLE + FULFILLED (135 SAR) + APPROVED (25 SAR, does not count); remaining limit = 265 SAR

---

## Overview

This document describes the implementation of the recipient weekly allowance feature. Recipients have a **400 SAR** weekly spending limit that resets at the end of each week. The system tracks how much has been used and prevents new requests that would exceed the limit.

---

## Business Rules

1. **Weekly limit:** 400 SAR per recipient per week
2. **Week definition:** Sunday 00:00 → Saturday 23:59:59
3. **Reset:** The limit implicitly resets at the start of each new week (no manual reset required)
4. **Usage calculation:** Sum of `price_snapshot × quantity` for all request items where the parent request has `status IN ('REQUESTED', 'REDEEMABLE', 'FULFILLED')` and was created within the current week. **remaining_limit = 400 - weekly_used**
5. **Validation:** When submitting a new request, if `weekly_used + new_request_total > 400`, the request is rejected with "Weekly allowance exceeded."

---

## Implementation

### 1. RecipientAllowanceService

**Location:** `app/Http/Services/RecipientAllowanceService.php`

A dedicated service class that encapsulates all weekly allowance logic:

| Method | Description |
|--------|-------------|
| `getCurrentWeekBounds()` | Returns `[weekStart, weekEnd]` (Sunday 00:00 – Saturday 23:59:59) |
| `getWeeklyUsed($recipientId)` | Sums `price_snapshot × quantity` from `request_items` joined with `requests` where `recipient_id` matches, `status IN ('REQUESTED','REDEEMABLE','FULFILLED')`, and `created_at` is within the current week |
| `getRemainingLimit($recipientId)` | Returns `max(0, 400 - weekly_used)` |
| `wouldExceedAllowance($recipientId, $amount)` | Returns `true` if `weekly_used + amount > 400` |

**Constant:** `WEEKLY_LIMIT = 400`

---

### 2. Recipient Dashboard

**Location:** `resources/views/recipient/dashboard.blade.php`

- **Before:** Hardcoded "280" for remaining limit and "400" for weekly limit
- **After:** Displays dynamic values:
  - `$remainingLimit` – calculated via `RecipientAllowanceService::getRemainingLimit()`
  - `$weeklyLimit` – from `RecipientAllowanceService::WEEKLY_LIMIT`

**Controller:** `RecipientController@dashboard` passes these values to the view.

---

### 3. Request Submission Validation

**Location:** `app/Http/Controllers/Recipient/RecipientRequestController.php`

- Uses `RecipientAllowanceService::wouldExceedAllowance($user->id, $totalAmount)` before creating a request
- If the check fails, returns `back()->withErrors(['allowance' => __('Weekly allowance exceeded.')])`
- Removed the previous logic that summed `reserved_amount` across multiple statuses (REQUESTED, APPROVED, etc.)

---

### 4. Provider Menu / Cart Page

**Location:** `app/Http/Controllers/Recipient/ProviderMenuController.php` and `resources/views/recipient/providers/show.blade.php`

- **Controller:** Uses `RecipientAllowanceService::getWeeklyUsed()` and `RecipientAllowanceService::WEEKLY_LIMIT` instead of querying requests directly
- **View:** Weekly allowance card shows used vs remaining based on the service
- **JavaScript:** Cart validation uses the dynamic `allowance` value to warn when the projected total would exceed the limit and disables the submit button
- **Error display:** Added `@error('allowance')` to show the validation message when a submission is rejected

---

## Data Model

- **`requests`** – has `recipient_id`, `status`, `created_at`
- **`request_items`** – has `request_id`, `price_snapshot`, `quantity`
- Only requests with `status IN ('REQUESTED', 'REDEEMABLE', 'FULFILLED')` count toward `weekly_used`

---

## Routes

| Route | Controller | Purpose |
|-------|------------|---------|
| `GET /recipient/dashboard` | `RecipientController@dashboard` | Shows remaining weekly limit |
| `POST /recipient/requests` | `RecipientRequestController@store` | Validates and creates request (enforces allowance) |
| `GET /recipient/providers/{provider}` | `ProviderMenuController@show` | Shows menu + allowance usage in cart sidebar |

---

## Files Modified/Created

| File | Action |
|------|--------|
| `app/Http/Services/RecipientAllowanceService.php` | **Created** |
| `app/Models/FundTransaction.php` | **Created** |
| `database/migrations/2026_02_27_000000_create_fund_transactions_table.php` | **Created** |
| `database/seeders/AllowanceTestDataSeeder.php` | **Created** |
| `database/seeders/ProviderMenuItemSeeder.php` | **Created** |
| `database/seeders/ProviderSeeder.php` | **Created** |
| `database/seeders/RecipientSeeder.php` | **Created** |
| `app/Http/Controllers/Recipient/RecipientController.php` | Modified – added `dashboard()` method |
| `app/Http/Controllers/Recipient/RecipientRequestController.php` | Modified – uses `RecipientAllowanceService` for validation |
| `app/Http/Controllers/Recipient/ProviderMenuController.php` | Modified – uses `RecipientAllowanceService` for `$weeklyUsed` |
| `app/Models/Ewallet.php` | Modified |
| `database/seeders/DatabaseSeeder.php` | Modified – added allowance, provider, recipient seeders |
| `routes/web.php` | Modified – dashboard route uses controller instead of closure |
| `resources/views/recipient/dashboard.blade.php` | Modified – dynamic remaining limit and weekly limit |
| `resources/views/recipient/providers/show.blade.php` | Modified – dynamic allowance, error display |

---

## Note on Statuses

**REQUESTED**, **REDEEMABLE**, and **FULFILLED** requests count toward the weekly limit. APPROVED (provider adopted) and REJECTED do not.
