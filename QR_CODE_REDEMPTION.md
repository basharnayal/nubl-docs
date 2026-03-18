# QR Code Redemption Flow

Documentation for the QR code generation and redemption process used when recipients redeem their approved orders at the provider.

---

## Overview

When a request is **APPROVED** (provider adopted) or **REDEEMABLE** (City Fund), the recipient receives a QR code (and manual code) to show the provider. The provider scans the QR or enters the code manually to redeem the order.

---

## When the QR Code Appears

The QR code is displayed on the **recipient request details page** (`recipient/requests/{id}`) when:

| Condition | Description |
|-----------|--------------|
| Status | `APPROVED` or `REDEEMABLE` |
| Redemption exists | `OrderRedemption` record with `status = PENDING` |
| Not expired | `redeem_expires_at` is in the future |

**Both statuses show the QR code:**
- **APPROVED** — Provider adopted (pays from own pocket, `PROVIDER_ADOPTION`)
- **REDEEMABLE** — Provider accepted with City Fund (`CITY_FUND`)

---

## Redemption Token Generation

### When tokens are created

1. **Provider adopts** (`ProviderRequestController` → action `adopt`):
   - Status → `APPROVED`, `funding_source` → `PROVIDER_ADOPTION`
   - `RedemptionService::generateForRequest()` is called immediately

2. **Provider accepts with City Fund** (`ProviderRequestController` → action `approve`):
   - Status → `REDEEMABLE`, `funding_source` → `CITY_FUND`
   - `RedemptionService::generateForRequest()` is called immediately

3. **Legacy / backfill** (`RecipientRequestController::show`):
   - When a recipient views an APPROVED or REDEEMABLE request that has no redemption yet (e.g. created before this feature)
   - Token is generated on first view

### Token format

- **Raw token:** 9-character alphanumeric (e.g. `ABC123XYZ`)
- **Stored:** Encrypted in `token_ciphertext`, hashed in `token_code` (SHA256) for lookup
- **TTL:** Configurable via `config('qr.ttl_minutes', 180)` — default 3 hours

---

## Redemption Flow

1. **Recipient** opens request details → sees QR code and manual code
2. **Provider** scans QR (or enters code manually) on `/provider/qr/scan`
3. **Backend** (`ProviderQrController::redeem`):
   - Validates token (hash lookup)
   - Ensures provider matches the request's provider
   - Checks status (PENDING, not REDEEMED, not EXPIRED)
   - Marks redemption as `REDEEMED`
   - For `CITY_FUND`: transfers from system wallet to provider
   - Redirects to proof upload page

---

## Files Reference

| File | Purpose |
|------|---------|
| `app/Http/Services/RedemptionService.php` | Generates redemption tokens for APPROVED and REDEEMABLE requests |
| `app/Http/Controllers/Provider/ProviderRequestController.php` | Calls `generateForRequest` on adopt and approve |
| `app/Http/Controllers/Provider/ProviderQrController.php` | Scan page and redeem API |
| `app/Http/Controllers/Recipient/RecipientRequestController.php` | Ensures redemption exists when showing request (backfill for legacy) |
| `resources/views/recipient/requests/show.blade.php` | Displays QR code for APPROVED and REDEEMABLE |
| `resources/views/provider/qr/scan.blade.php` | Provider scan page (camera + manual entry) |

---

## Change: QR Code for APPROVED Orders

**Previously:** QR code only appeared for `REDEEMABLE` (City Fund) requests.

**Now:** QR code appears for both `APPROVED` (provider adopted) and `REDEEMABLE` (City Fund).

This allows recipients to redeem adopted orders the same way as City Fund orders — by showing the QR code to the provider at pickup.
