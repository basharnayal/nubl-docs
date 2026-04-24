# Seed Data Reference

## Quick Start

```bash
# Base data only (roles, permissions, core users, providers, menu items)
php artisan migrate:fresh --seed

# Demo data (requires base data to exist first)
php artisan db:seed --class=DemoDataSeeder
```

All demo seeders are **idempotent** — running `DemoDataSeeder` multiple times will not create duplicates.

---

## Seeding Order

### `DatabaseSeeder` (base data — `php artisan db:seed`)

| Step | Seeder | Purpose |
|------|--------|---------|
| 1 | `PermissionSeeder` | All permissions from `PermissionDefinitions` |
| 2 | `RoleSeeder` | 4 roles: admin, donor, recipient, provider + permission sync |
| 3 | `AdminUserSeeder` | Core admin user (`admin@nubl.com`) |
| 4 | `DonorSeeder` | Core donor users (`donor@nubl.com`, `donor-seed@nubl.com`) |
| 5 | `EwalletSeeder` | System wallet + bootstrap fund transaction |
| 6 | `RecipientSeeder` | Core recipient (`recipient@nubl.com`) + profile + KYC |
| 7 | `ProviderSeeder` | 7 providers with full profiles, financial info, documents |
| 8 | `MenuItemCategorySeeder` | ~88 categories across 6 business types |
| 9 | `ProviderMenuItemSeeder` | ~80 menu items across providers |
| 10 | `AllowanceTestDataSeeder` | 4 requests for allowance testing |

### `DemoDataSeeder` (demo data — `php artisan db:seed --class=DemoDataSeeder`)

| Step | Seeder | Purpose |
|------|--------|---------|
| 1 | `SystemSettingsSeeder` | QR TTL, weekly allowance, allocation settings |
| 2 | `DemoUsersSeeder` | 1 admin + 15 donors + 20 recipients + 2 providers |
| 3 | `DemoPaymentSeeder` | 50 payments (all statuses) + 35 donation fund_transactions |
| 4 | `DemoRequestSeeder` | ~100 requests across all 6 statuses + items + payment links + 3 pending allocations |
| 5 | `DemoRedemptionSeeder` | Order redemptions (PENDING/REDEEMED/EXPIRED) + proofs + redemption fund_transactions |
| 6 | `DemoProviderPayoutSeeder` | Provider payouts in all 5 statuses + payout items |
| 7 | `DemoMiscSeeder` | Notifications + 6 summary reports + 25 activity log entries |
| 8 | `WalletBalanceSyncSeeder` | `syncBalance()` on every ewallet (must be last) |

---

## Known Test Accounts

All accounts use password: **`password`**

| Role | Email | Status | Notes |
|------|-------|--------|-------|
| Admin | `admin@nubl.com` | active | Primary admin |
| Admin | `admin2@nubl.com` | active | Secondary admin |
| Donor | `donor@nubl.com` | active | Primary donor |
| Donor | `donor-seed@nubl.com` | active | Bootstrap donor (city fund) |
| Donor | `demo-donor-01@nubl.com` … `demo-donor-15@nubl.com` | active | Bulk donors |
| Recipient | `recipient@nubl.com` | active | Primary recipient |
| Recipient | `demo-recipient-01@nubl.com` … `demo-recipient-15@nubl.com` | active | Bulk recipients |
| Recipient | `pending-recipient@nubl.com` | pending_approval | Pending approval demo |
| Recipient | `rejected-recipient@nubl.com` | rejected | Rejected demo |
| Provider | `provider@nubl.com` | active | Primary provider (Al-Rashid Kitchen) |
| Provider | `pending-provider@nubl.com` | pending_approval | Pending approval demo |
| Provider | `rejected-provider@nubl.com` | rejected | Rejected demo |

---

## Data Volume Summary

| Entity | Approximate Count | Notes |
|--------|-------------------|-------|
| Users (total) | ~50 | 2 admins, 17 donors, 21 recipients, 9+ providers |
| Permissions | ~30+ | From `PermissionDefinitions` |
| Roles | 4 | admin, donor, recipient, provider |
| System settings | 4 | QR TTL, allowance limit, allocation pause |
| Ewallets | ~10 | 1 SYSTEM + provider wallets |
| Payments | ~51 | 35 SUCCEEDED, 5 FAILED, 5 PENDING, 3 INITIATED, 2 PROCESSING |
| Fund transactions | ~110+ | Donations IN, redemptions IN/OUT, bank payouts OUT |
| Requests | ~104 | 15 REQUESTED, 10 APPROVED, 15 REDEEMABLE, 10 REJECTED, 5 CANCELLED, 45 FULFILLED + 4 from AllowanceTestData |
| Request items | ~200+ | 1–3 per request |
| Request payment links | ~60 | FIFO allocation for REDEEMABLE/FULFILLED |
| Order redemptions | ~70 | PENDING, REDEEMED, EXPIRED |
| Order proofs | ~45 | For FULFILLED requests |
| Provider payouts | ~8–12 | All 5 statuses |
| Provider payout items | ~15–25 | Linked earning transactions |
| Pending allocations | 3 | Global and provider-paused |
| Notifications | ~25 | Donation receipts, admin alerts, request status, payouts |
| Summary reports | 6 | 4 weekly + 2 monthly |
| Activity log | 25 | Across all log categories |

---

## Temporal Distribution

Data is spread across **8 weeks** for realistic dashboard charts:
- **Fulfilled requests:** weeks 1–7 (not current week)
- **Active requests (REQUESTED, REDEEMABLE, APPROVED):** weeks 0–3
- **Payments:** spread evenly across weeks 0–7
- **Rejected / cancelled:** spread throughout
- **User registrations:** staggered from week 7 to now

---

## Idempotency

Each demo seeder checks for existing data before inserting:
- `DemoUsersSeeder` — checks `demo-donor-01@nubl.com` existence
- `DemoPaymentSeeder` — checks `notes = 'demo_seed'`
- `DemoRequestSeeder` — checks `invoice_id LIKE 'DEMO-%'`
- `DemoRedemptionSeeder` — checks redemptions linked to DEMO requests
- `DemoProviderPayoutSeeder` — checks `reference_number LIKE 'DEMO-%'`
- `DemoMiscSeeder` — checks by notification type count and activity log `demo_seed` property

---

## Important: WithoutModelEvents

`DatabaseSeeder` uses `WithoutModelEvents` trait. This means:
- `FundTransactionObserver` does **not** fire → wallet balances are **not** auto-updated
- `ProviderProfile::booted()` does **not** fire → ewallets for new providers are created manually
- `Activity::booted()` hash computation does **not** fire → hashes are computed manually in `DemoMiscSeeder`
- **`WalletBalanceSyncSeeder` MUST run last** to recalculate all wallet balances from transactions

---

## Running Individual Seeders

```bash
# Run just the demo data seeders (requires base data to exist)
php artisan db:seed --class=DemoUsersSeeder
php artisan db:seed --class=DemoPaymentSeeder
php artisan db:seed --class=DemoRequestSeeder
php artisan db:seed --class=DemoRedemptionSeeder
php artisan db:seed --class=DemoProviderPayoutSeeder
php artisan db:seed --class=DemoMiscSeeder
php artisan db:seed --class=WalletBalanceSyncSeeder
```

> **Note:** When running individual seeders outside `DatabaseSeeder`, model events **will** fire. This is generally fine but may create duplicate activity log entries from `FundTransaction`'s `LogsActivity` trait.
