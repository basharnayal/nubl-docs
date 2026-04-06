# Provider bank payout (weekly settlement)

## Purpose

Providers earn internal wallet credits when the city fund pays them at **QR redemption** (`SystemWalletService::transferToProviderForRequest`). Those credits are ledger rows: provider wallet **IN**, source **`PAYOUT`** (internal platform payout from the system wallet).

This feature adds a **second layer**: periodic **external bank transfers** to providers. The platform wallet balance is **not** changed at payout-request creation. Only a real **`FundTransaction` OUT** from the provider wallet (source **`PROVIDER_BANK_PAYOUT`**) is created when an **admin confirms** the bank transfer after uploading a receipt.

## Accounting rules

1. **`ewallets.balance`** is updated only via **`FundTransactionObserver`** on new ledger rows (same as the rest of the app).
2. Eligible earnings for settlement = provider wallet **IN** rows with source **`PAYOUT`** (same as internal credit from redemption) that are **not** linked to a payout in a **reserving** status (`PENDING_ADMIN_REVIEW`, `PROCESSING`, `TRANSFERRED`).
3. **`TRANSFERRED`** permanently ties those IN lines to a completed settlement. **`REJECTED`** / **`CANCELLED`** do not reserve lines; the same IN rows can appear in a later weekly run.
4. Minimum amount per generated request: **`config('provider_payout.min_weekly_payout_amount')`** (default **50 SAR**, override with `PROVIDER_PAYOUT_MIN_AMOUNT`).

## Schedule

- Command: `php artisan provider-payouts:generate-weekly`
- Scheduler: **Sunday 00:00** in **`config('app.timezone')`** (set `APP_TIMEZONE=Asia/Riyadh` for KSA wall time).
- Week label: the **Sun–Sat week that ended** just before the run (aligned with `WeeklyAllowanceSettings` Sun–Sat convention).

## Admin UI

- **Finance → Provider payouts** (`/admin/finances/provider-payouts`)
- List, filters, detail with linked ledger lines, receipt upload, confirm / reject / cancel.

## Demo seeder (local QA)

```bash
php artisan db:seed --class=ProviderPayoutDemoSeeder
```

Creates internal **PAYOUT** IN lines plus one **pending** and one **transferred** `provider_payouts` row for **user id 3** (or `provider@nubl.com`) so you can test the admin queue and the provider **My Wallet** page. Skips if rows with `meta.demo_seed = true` already exist.

## Key files

| Area | Location |
|------|----------|
| Migrations | `database/migrations/2026_04_06_120000_create_provider_payouts_table.php`, `2026_04_06_120001_create_provider_payout_items_table.php`, `2026_04_06_120002_add_provider_payout_id_to_fund_transactions.php`, `2026_04_06_120003_add_fund_transaction_out_id_to_provider_payouts.php` |
| Models | `App\Models\ProviderPayout`, `App\Models\ProviderPayoutItem` |
| Ledger eligibility | `App\Http\Services\ProviderPayoutLedgerService` |
| Generation | `App\Http\Services\ProviderPayoutGenerationService` |
| Confirm / reject / cancel | `App\Http\Services\ProviderPayoutConfirmationService` |
| Week boundaries | `App\Support\ProviderPayoutWeek` |
| Admin | `App\Http\Controllers\Admin\ProviderPayoutController` |
| Tests | `tests/Feature/Admin/ProviderPayoutFlowTest.php` |

## Doc consistency

Some older docs still describe wallet timing differently than the current code. **Source of truth** for when the system wallet pays the provider remains **`PAYMENT_REFERENCE.md`** and `SystemWalletService::transferToProviderForRequest` (at redemption). This payout doc describes only the **external bank settlement** on top of that ledger.

---

## Technical reference (implementation & engineering decisions)

This section is a **dense implementation guide** for maintainers and developers. It documents what was built, **why** key choices were made, and how subsystems interact.

### Problem statement

Providers accumulate **internal** wallet credits when the city fund pays them at QR redemption (`SystemWalletService::transferToProviderForRequest`). Those are normal ledger **IN** rows (`FundTransaction`, source **`PAYOUT`**). The product also needs **external** bank payouts: the business periodically settles those accumulated credits to the provider’s real bank account. That second step must:

- **Not** double-count or double-spend ledger lines.
- **Not** move money on the platform ledger until an admin has evidence of a real bank transfer (receipt + reference).
- Fit a **weekly** operational rhythm aligned with existing **Sun–Sat** week semantics (`WeeklyAllowanceSettings` / app timezone).

### High-level architecture

1. **Generation (scheduled job)** builds **`provider_payouts`** rows plus **`provider_payout_items`** rows that **reference** eligible **`fund_transactions`** (earning IN lines). No **`FundTransaction`** is created at this stage for the settlement itself; the provider wallet balance is **unchanged** here by design.
2. **Admin workflow** reviews the request, uploads a receipt, then **confirms**. Confirmation creates exactly one **`FundTransaction` OUT** from the provider wallet with source **`PROVIDER_BANK_PAYOUT`**, which flows through the existing **`FundTransactionObserver`** and updates **`ewallets.balance`** like any other ledger line.
3. **Rejection / cancellation** stops the request without creating an OUT; linked IN lines become eligible again in a later run because they are no longer “reserved” by a payout in a reserving status.

This separation (request snapshot vs. real OUT) matches accounting: the **request** is a batch proposal; the **OUT** is the actual withdrawal after human verification.

### Data model

| Artifact | Role |
|----------|------|
| **`provider_payouts`** | One row per generated settlement **request** for a provider for a given **settlement week window** (`week_start_at`, `week_end_at`), plus `scheduled_at`, `amount`, `status`, snapshots (`snapshot_wallet_balance`, `snapshot_available_amount`), optional `fund_transaction_out_id` after confirmation, admin metadata. |
| **`provider_payout_items`** | Join table: which **`fund_transactions`** (earning IN lines) are bundled into this payout request, with per-line `amount` copy for audit. |
| **`fund_transactions.provider_payout_id`** (nullable FK) | Used on the **OUT** row created on confirmation (links the withdrawal line to the payout record). Internal **PAYOUT** IN lines are not required to store this FK for eligibility; eligibility is driven by **`provider_payout_items`** + payout **status**. |

**Engineering decision:** Keep **`provider_payout_items`** as the canonical link between a payout request and earning IN lines. That makes eligibility queries explicit (`NOT EXISTS` against items joined to payouts in reserving statuses) and preserves history if a line appears in a later attempt after reject/cancel.

### Settlement week boundaries (`App\Support\ProviderPayoutWeek`)

`ProviderPayoutWeek::settlementWeekBoundariesAt(Carbon $runAt)` computes the **Sun–Sat** window that **ended** before the run, in `config('app.timezone')`. When the job runs **Sunday 00:00**, the labeled week is the **previous** Sun–Sat block. This aligns the **label** with “the week we are settling,” consistent with the rest of the app’s weekly conventions.

**Important for tests:** Use a fixed **`Carbon::setTestNow()`** when asserting week boundaries and generation behavior (see `ProviderPayoutFlowTest`).

### Eligible earnings (`App\Http\Services\ProviderPayoutLedgerService`)

Eligible rows are **`FundTransaction`** rows where:

- `wallet_id` = provider’s ewallet
- `direction` = **`IN`**
- `source` = **`PAYOUT`** (same internal credit type as redemption-time city-fund → provider transfer)

**Exclusion rule:** Exclude any IN row that already appears in **`provider_payout_items`** for a **`provider_payouts`** row whose **`status`** is in **`ProviderPayout::RESERVING_STATUSES`** (`PENDING_ADMIN_REVIEW`, `PROCESSING`, `TRANSFERRED`). Implemented as a `whereNotExists` subquery.

**Engineering decision:** Use **reserving statuses** instead of “any historical link,” so **`REJECTED`** / **`CANCELLED`** payouts **release** those IN lines for a future weekly run. **`TRANSFERRED`** permanently consumes them for settlement purposes.

**Precision:** Amounts are summed/compared with **decimal-safe** helpers (`bcadd` / `bccomp` when available) to avoid float drift.

### Generation (`App\Http\Services\ProviderPayoutGenerationService`)

**Inputs:** `generateWeeklyAt(Carbon $runAt)` — optional **`--at=`** on the Artisan command for deterministic tests.

**Flow:**

1. **Distributed lock:** `Cache::lock('provider-payout-generate-weekly', 600)` — only one generation run at a time across processes (TTL 600s). If the lock is not acquired, returns an **empty array** (no partial work).
2. For each **active provider profile** (user active, `provider` role) with an **enabled** ewallet:
   - **`DB::transaction`** + **`lockForUpdate()`** on that **ewallet** row — serializes generation **per provider** so two concurrent workers cannot both pass “no blocking payout” checks for the same wallet.
   - **`hasBlockingPayoutForWeek`:** if a **`provider_payouts`** row already exists for this **provider** and **same** `week_start_at` / `week_end_at` with status in **`PENDING_ADMIN_REVIEW`**, **`PROCESSING`**, or **`TRANSFERRED`**, **skip** (prevents duplicate requests for the same settlement window while one is still active or completed).
   - Load **eligible** IN lines; enforce **minimum** total vs. **`config('provider_payout.min_weekly_payout_amount')`** (env **`PROVIDER_PAYOUT_MIN_AMOUNT`**).
   - **Create** `provider_payouts` + `provider_payout_items` rows; store eligible fund transaction IDs in **`meta`** for debugging/audit.
   - **Audit log** + **notify admins** (`ProviderPayoutPendingAdminNotification`) for each created payout id.

**Engineering decision:** **No** `FundTransaction` at generation avoids phantom balance moves and keeps “batch intent” separate from “money moved.”

### Anti-duplication (idempotency & safety layers)

| Layer | Mechanism |
|-------|-----------|
| Scheduler | `withoutOverlapping()` on the scheduled command reduces overlapping cron ticks. |
| Process-wide | Cache lock **`provider-payout-generate-weekly`** blocks concurrent command execution. |
| Per provider | Ewallet **`lockForUpdate`** inside a DB transaction orders conflicting work. |
| Business rule | **`hasBlockingPayoutForWeek`** blocks a second payout for the **same** week window while a blocking status exists. |
| Ledger | **`eligibleEarningInQuery`** excludes IN lines tied to **reserving** payouts via **`provider_payout_items`**. |
| Database | **Unique** `(provider_payout_id, fund_transaction_id)` on **`provider_payout_items`** — prevents duplicate inclusion of the same IN line **twice in one payout**. **Unique** `fund_transaction_out_id` on **`provider_payouts`** — ensures a single OUT transaction cannot back two payout records. |

**Note on “same week, after reject”:** If a payout is **`REJECTED`** or **`CANCELLED`**, it is **not** in reserving statuses, so **`hasBlockingPayoutForWeek`** does **not** block another request for the **same** `week_start_at` / `week_end_at`. That allows **retry in the same labeled week** if operations need it; the ledger still prevents double-use of IN lines while another payout is **active** (reserving). Product copy may emphasize “future run” for clarity; the code allows same-calendar-week retry after terminal non-reserving states.

### Confirmation / reject / cancel (`App\Http\Services\ProviderPayoutConfirmationService`)

- **`confirm`:** `lockForUpdate` on the **`provider_payouts`** row; require **`isConfirmable()`** (`PENDING_ADMIN_REVIEW` only); validate items vs. `amount`; `lockForUpdate` on **ewallet**; check **sufficient balance**; **`FundTransaction::create`** **OUT** with **`SOURCE_PROVIDER_BANK_PAYOUT`** and **`provider_payout_id`** set; set payout to **`TRANSFERRED`**, set **`fund_transaction_out_id`**; audit; **`ProviderPayoutTransferredNotification`** to provider.
- **`reject` / `cancel`:** `lockForUpdate` on payout; require confirmable; set **`REJECTED`** or **`CANCELLED`** with audit.

**Double-submit safety:** Second confirm hits non-confirmable state → exception / error path (covered by tests).

### Artisan command & schedule

- **Command:** `provider-payouts:generate-weekly` (`GenerateWeeklyProviderPayoutsCommand`), optional **`--at=`** ISO8601 for tests.
- **Production schedule:** **`routes/console.php`** — **`weeklyOn(Carbon::SUNDAY, '00:00')`** with **`timezone(config('app.timezone'))`**, **`withoutOverlapping()`**, log append.
- **Local experimentation:** The same file may include a **commented** alternate schedule (e.g. **`everyTwoMinutes()`**) so developers can swap schedules and run **`php artisan schedule:work`** without waiting a week. **Restore weekly schedule before production deploys.**

### Configuration

- **`config/provider_payout.php`** — minimum amount and related settings.
- **`APP_TIMEZONE`** / **`config('app.timezone')`** — wall-clock for the scheduler and `ProviderPayoutWeek`.

### Admin & provider surfaces

- **Admin:** `ProviderPayoutController` — list, show, receipt upload, confirm / reject / cancel; routes under **`admin.finances`** (see `routes/web.php`).
- **Provider:** wallet/history views for payout status and receipt download where implemented (`ProviderWalletController`).

### Notifications

- Admins: new pending payout request.
- Provider: payout marked transferred.

Centralized copy in **`lang/en.json`** / **`lang/ar.json`** and **`config/notifications.php`** as per project patterns.

### Database migrations (complete set)

Include, in order:

- `2026_04_06_120000_create_provider_payouts_table.php`
- `2026_04_06_120001_create_provider_payout_items_table.php`
- `2026_04_06_120002_add_provider_payout_id_to_fund_transactions.php`
- `2026_04_06_120003_add_fund_transaction_out_id_to_provider_payouts.php`
- `2026_04_06_120004_add_provider_payout_uniqueness_indexes.php` — uniqueness constraints described above.

### Automated tests (`tests/Feature/Admin/ProviderPayoutFlowTest.php`)

Covers: happy-path generation, minimum amount skip, **no duplicate generation** for same provider/same week, confirmation creates correct **OUT**, **cannot confirm twice**, reject then eligibility on a **later** run, exclusion of non-**PAYOUT** IN sources.

### Maintenance checklist

When changing this feature, verify:

1. **Eligibility** still excludes IN lines attached to **reserving** payouts only.
2. **Generation** still does not create balance-moving **`FundTransaction`** rows.
3. **Confirmation** still creates exactly one **OUT** per successful confirm and updates **`ewallets.balance`** only through the observer.
4. **Locks** (`cache`, `wallet`, payout `lockForUpdate`) remain coherent for concurrency.
5. **Migrations** for uniqueness are applied in fresh and upgrade paths.
6. **Schedule** in `routes/console.php` matches deployment intent (weekly vs. local dev).
