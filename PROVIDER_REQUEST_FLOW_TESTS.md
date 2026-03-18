# Provider Request Flow Tests

This document describes the automated tests for the Provider Request Flow feature. The tests ensure that providers can adopt, approve (city fund), or reject incoming requests, and that wallet logic is enforced when approving with city fund.

---

## Test Location

**File:** `tests/Feature/Provider/ProviderRequestFlowTest.php`

---

## Provider Actions

| Action | Effect | Wallet impact |
|--------|--------|---------------|
| **Adopt (My Fund)** | Provider covers cost | No change to city fund or provider wallet |
| **Approve (City Fund)** | Request paid from city fund | Deducts from system wallet, credits provider wallet |
| **Reject** | Request rejected with reason | No wallet change |

See `docs/SYSTEM_WALLET_DONATION_FLOW.md` for full wallet flow details.

---

## Test Coverage

| Test | Description |
|------|-------------|
| **provider_can_view_incoming_requests** | Provider can view the requests index and see recipient name and amount. |
| **provider_can_adopt_request** | Adopt action updates status to `APPROVED`, funding_source to `PROVIDER_ADOPTION`. No wallet changes. CITY_FUND not affected. |
| **provider_can_approve_request** | Approve (accept) action updates status to `REDEEMABLE`, funding_source to `CITY_FUND`. Deducts from city fund (transfers to provider wallet). |
| **provider_cannot_approve_when_city_fund_has_insufficient_balance** | When system wallet balance (10 SAR) is less than `reserved_amount` (50 SAR), approve is blocked. Request stays `REQUESTED`, session has error. |
| **provider_can_reject_request_with_reason** | Reject action updates status to `REJECTED` with `rejection_reason_code` and `rejection_reason_note`. |
| **provider_cannot_act_on_non_pending_request** | Actions (adopt, approve, reject) are only allowed when request status is `REQUESTED`. Non-pending requests return error. |
| **provider_cannot_view_others_requests** | Provider can only view requests assigned to them. Other providers get 404. |

---

## Setup Requirements

The test `setUp` creates:

- **Roles:** recipient, provider
- **Users:** recipient, provider
- **System wallet:** Ewallet with `owner_type = SYSTEM`, seeded via `FundTransaction` (IN, 100 SAR) + observer/sync (for approve test)
- **Provider profile:** Creates provider ewallet via `ProviderProfile::booted()`
- **Menu item:** ProviderMenuItem (50 SAR)
- **Request:** REQUESTED request with `reserved_amount` 50 SAR

---

## How to Run

```bash
php artisan test --filter=ProviderRequestFlowTest
```

---

## Prerequisites & Notes

- **Database:** Uses `RefreshDatabase` trait. Requires a configured database (e.g. SQLite in-memory in `.env.testing`).
- **System wallet:** Must exist with sufficient balance for approve tests. The insufficient-balance test drains it to 10 SAR before asserting.
- **Provider profile:** Provider must have a `ProviderProfile` (and thus an ewallet) for approve to succeed; `ProviderProfile::booted()` creates the ewallet on profile creation.
