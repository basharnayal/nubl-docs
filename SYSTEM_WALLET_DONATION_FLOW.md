# System Wallet – Donation & Provider Approval Flow

## Overview

The **system wallet** (Ewallet with `owner_type = SYSTEM`) is the city fund. It receives donations from donors and pays out to providers when they approve requests and redeem qr code with city fund.

## Balance Calculation

**Balance =** sum(amount) where `wallet_id = wallet.id` AND `direction = 'IN'`  
**minus** sum(amount) where `wallet_id = wallet.id` AND `direction = 'OUT'`

The `ewallets.balance` column is kept in sync by `FundTransactionObserver` — on each new `FundTransaction`, the wallet balance is incremented (IN) or decremented (OUT). Applies to all wallets (city fund and provider).

- **`Ewallet::syncBalance()`** — recalculates from transactions and updates the column
- **`php artisan wallets:sync-balances`** — syncs all wallet balances (useful for fixing existing data)

---

## Flows

### 1. Donor adds funds

1. Donor initiates donation → payment gateway (e.g. MyFatoorah)
2. Donor completes payment
3. System verifies payment (e.g. `DonationService::confirmDonation()`)
4. **→ Call `SystemWalletService::addFundsFromDonation()`** to credit the system wallet

### 2. Provider actions on a request

| Action | Effect on city fund | Effect on provider wallet |
|--------|---------------------|---------------------------|
| **Adopt (My Fund)** | No change | No change (provider covers cost themselves) |
| **Approve (City Fund)** | Deducts `reserved_amount` | Credits provider with `reserved_amount` |

When provider clicks **Accept (City Fund)**:
1. Check system wallet has sufficient balance (`ewallets.balance`, kept in sync by `FundTransactionObserver`)
2. Create `FundTransaction` records (OUT from system wallet, IN to provider wallet, source: PAYOUT) — observer updates both wallet balances
3. Update request status to REDEEMABLE, funding_source to CITY_FUND

---

## SystemWalletService

**Location:** `app/Services/SystemWalletService.php`

| Method | Description |
|--------|-------------|
| `addFundsFromDonation($amount, $donorId, $paymentId?)` | Creates a `FundTransaction` record (source: DONATION, direction: IN). `FundTransactionObserver` updates wallet balance. |
| `getSystemWallet()` | Returns the system's default Ewallet |
| `hasSufficientBalance($amount)` | Returns true if `ewallets.balance` ≥ amount (balance kept in sync by observer) |
| `transferToProviderForRequest($request, $orderRedemptionId = null)` | Creates FundTransaction OUT (system) and IN (provider) for a City Fund request redemption. Observer updates both wallet balances. |

---

## Integration

In `DonationService::confirmDonation()`, after payment verification and before/after `CityFund::increment()`:

```php
app(SystemWalletService::class)->addFundsFromDonation(
    $donation->amount,
    $donation->user_id,
    $donation->id, // or payment_id when payments table exists
);
```

The call is currently **commented out** in `DonationService`. Uncomment when ready to integrate with the wallet flow.

---

## Data Model

- **Ewallet** (`owner_type = SYSTEM`, `owner_id = null`) – holds pooled donation funds; `balance` kept in sync by `FundTransactionObserver`
- **FundTransaction** – audit trail; each record triggers observer to update wallet `balance`
- **FundTransactionObserver** (`app/Observers/FundTransactionObserver.php`) – on `created`: IN → increment balance, OUT → decrement balance

---

## Missing / TODO

- **`is_provider_donation`** — Indicates if the fulfillment was a provider donation (adoption) or from city fund, to know whether to deduct from city fund or not. Currently inferred from `funding_source` on the request; an explicit field may be needed.
