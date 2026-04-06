# Recipient Request Submission Tests

This document describes the automated tests for the Recipient Request Submission feature. The tests ensure that critical business rules like weekly allowance, provider capacity, and ownership are enforced.

---

## Test Location

**File:** `tests/Feature/Recipient/RecipientRequestSubmissionTest.php`

---

## Test Coverage

| Test | Description |
|------|-------------|
| **recipient_can_submit_multi_item_request_successfully** | Happy path: recipient submits a valid multi-item request. Verifies redirection, database insertion, correct status (`REQUESTED`), reserved amount, and price snapshots. |
| **weekly_allowance_exceeded_blocks_creation** | Enforces the 400 SAR weekly allowance. Sets up existing requests totaling 300 SAR, attempts to add 120 SAR (420 > 400). Blocks creation and asserts `allowance` error. |
| **rejected_request_does_not_count_towards_allowance** | Verifies that `ADMIN_REJECTED` requests do not count toward the weekly limit. Recipient can submit new request even with a high-value rejected request on record. |
| **adopted_request_does_not_count_towards_allowance** | Verifies that `APPROVED` requests with `PROVIDER_ADOPTION` (provider pays) do not count toward the limit. Recipient can submit new request after an adopted request. |
| **cannot_request_menu_item_not_belonging_to_provider** | Prevents requesting a menu item that belongs to a different provider. Asserts `items.0.id` validation error. |

---

## Business Rules Verified

1. **Weekly allowance:** 400 SAR per recipient per week; REQUESTED, REDEEMABLE, and FULFILLED count (see `RecipientAllowanceService`).
2. **Status exclusions:** APPROVED (PROVIDER_ADOPTION) and REJECTED do NOT count toward allowance.
3. **Ownership:** Menu items must belong to the provider specified in the request.
4. **Price snapshot:** `request_items` store `price_snapshot` and `quantity` for audit.

---

## How to Run

```bash
php artisan test --filter=RecipientRequestSubmissionTest
```

---

## Prerequisites & Notes

- **Database:** Uses `RefreshDatabase` trait. Requires a configured database (e.g. SQLite in-memory in `.env.testing`).
- **Time handling:** Tests use `Carbon::setTestNow()` to freeze time for deterministic weekly allowance calculations (e.g. Wednesday to avoid week boundaries).
- **Factories:** Uses `User` factory; creates `ProviderOperatingInfo` and `ProviderMenuItem` manually (no factories for these).
