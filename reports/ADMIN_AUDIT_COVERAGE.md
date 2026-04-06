# Admin area — audit coverage report

This document inventories every **admin** entry point (from `routes/web.php` under `admin.*`, plus admin-only routes outside the prefix where relevant), maps them to code, and records **audit log** coverage against **FR-7.2** (decision/action, admin identity, timestamp, contextual IDs).

**Mechanism:** `App\Http\Services\AuditService` wraps Spatie **activitylog**. The **causer** is the acting user (`causer_id`); **timestamp** is `activity_log.created_at`; **decision/context** live in JSON **properties** (plus `description` = `{entity}.{action}`).

---

## Inventory (state-changing and sensitive)

| Area / module | File / method | Route name (approx.) | Action | State / sensitivity | Audit | How | Gap |
|---------------|---------------|----------------------|--------|---------------------|-------|-----|-----|
| Dashboard | Closure `web.php` | `admin.dashboard` | View dashboard | Read-only | — | none | OK (excluded) |
| Pending accounts list | `AccountApprovalController::index` | `admin.users.pending` | List queue | Read-only | — | none | OK |
| Approve account | `AccountApprovalController::approve` | `admin.users.approve` | Approve recipient/provider registration | State-changing | Yes | `AuditService` → `account_approval.approved` | OK |
| Reject account | `AccountApprovalController::reject` | `admin.users.reject` | Reject with reason | State-changing | Yes | `AuditService` → `account_approval.rejected` | OK |
| Reject form / application views | `showRejectForm`, `showApplication`, `showProviderApplication`, `showRecipientApplication` | various | Display forms | Read-only | — | none | OK |
| Serve KYC file | `AccountApprovalController::serveFile` | `admin.users.file` | Download document | Sensitive read | — | none | **Excluded** (see below) |
| User list / show / create / edit views | `UserManagementController` | `admin.manage.users.*` | Display | Read-only | — | none | OK |
| Create user | `UserManagementController::store` → `UserService::createUser` | `admin.manage.users.store` | Create user + profile | State-changing | Yes | `AuditService` → `user.created` | OK |
| Update user | `UserManagementController::update` → `UserService::updateUser` | `admin.manage.users.update` | Update user + profile | State-changing | Yes | `AuditService` → `user.updated` | OK |
| Delete user | `UserManagementController::destroy` → `UserService::deleteUser` | `admin.manage.users.destroy` | Delete user | State-changing | Yes | `AuditService` → `user.deleted` (explicit admin id) | OK |
| Deactivate user | `UserManagementController::deactivate` → `UserService::deactivate` | `admin.manage.users.deactivate` | Deactivate | State-changing | Yes | `AuditService` → `user.deactivated` | OK |
| Reactivate user | `UserManagementController::reactivate` → `UserService::reactivate` | `admin.manage.users.reactivate` | Reactivate | State-changing | Yes | `AuditService` → `user.reactivated` | OK |
| Request queue | `AdminRequestController::index` | `admin.requests.index` | List `ADMIN_PENDING` | Read-only | — | none | OK |
| Approve / reject request | `AdminRequestController::update` | `admin.requests.update` | Approve or reject city-fund request | State-changing | Yes | `AuditService` → `request.admin_approved` / `request.admin_rejected` | OK |
| Financial overview | `FinancialOverviewController::index` → `AdminFinancialService::getOverview` | `admin.finances.overview` | Aggregates | Read-only | — | none | OK |
| Payments list / show | `AdminPaymentController::index`, `show` | `admin.finances.payments.*` | List / detail + related activity | Read-only | — | none | OK |
| Payments CSV export | `AdminPaymentController::export` | `admin.finances.payments.export` | Download all matching rows | Sensitive | Yes | `AuditService` → `finance.payments_exported` | **Fixed** (was Missing) |
| Fund transactions list / show | `AdminFundTransactionController::index`, `show` | `admin.finances.fund-transactions.*` | List / detail | Read-only | — | none | OK |
| Fund transactions CSV export | `AdminFundTransactionController::export` | `admin.finances.fund-transactions.export` | Download ledger extract | Sensitive | Yes | `AuditService` → `finance.fund_transactions_exported` | **Fixed** (was Missing) |
| Financial reports | `FinancialReportController::index` → `AdminFinancialService::getRangeSummary` | `admin.finances.reports.index` | Date-range summary | Read-only | — | none | OK |

---

## Routes outside `/admin` (admin role or dev only)

| File / method | Route | Notes | Audit |
|---------------|-------|-------|-------|
| Closure `web.php` | `test-roles` | Debug view; `role:admin` | Excluded (non-production helper) |
| Closure `web.php` | `make-me-admin` | Assigns `admin` role; **not** protected by `role:admin` | Excluded (dev-only; not an admin UI workflow) |

No **Jobs**, **Actions**, or **API** routes were found that implement additional admin-only mutations beyond the table above.

**Authorization:** `StoreUserRequest` / `UpdateUserRequest` use `$this->user()?->hasRole('admin')` for admin user management forms. No separate Policy classes gate these beyond route middleware.

---

## FR-7.2 alignment

| Requirement | Implementation |
|-------------|----------------|
| Admin’s decision / action | Stored as `description` (`entity.action`) and/or `properties.decision`, `properties.entity`, `properties.action`, plus context (e.g. `user_id`, `request_id`, `filters` for exports). |
| Admin ID | `activity_log.causer_id` (Spatie **causedBy**). `UserService::deleteUser` / `deactivate` / `reactivate` pass `$admin->id` explicitly. |
| Timestamp | `activity_log.created_at` (automatic). |
| Entity IDs / context | Present in `properties` per call (e.g. `request_id`, `rejection_reason_code`, `deleted_user_id`, export `filters`). |

---

## What was already covered (before this pass)

- Account approval / rejection (`AccountApprovalController`).
- User lifecycle CRUD + deactivate / reactivate (`UserService`).
- Admin approve / reject recipient **requests** (`AdminRequestController`).

## Gaps fixed in this pass

1. **Finance CSV exports** — Sensitive bulk export of payments and fund transactions had no audit trail. Added `finance.payments_exported` and `finance.fund_transactions_exported` with `decision` = `export`, `export_type`, and request `filters`.

2. **Tests** — Extended `AccountApprovalTest`, `UserManagementTest`, `AdminFinancialManagementTest`, and `AdminRequestFlowTest` to assert activities, `causer_id`, and timestamps where applicable.

## Intentionally excluded from audit logging

| Item | Reason |
|------|--------|
| Dashboard, lists, detail pages, reports (HTML), financial charts | Read-only; no state change. |
| `serveFile` (KYC / document download) | Read-only; would need a separate “document access log” product decision (volume, PII). |
| `test-roles`, `make-me-admin` | Development / bootstrap helpers, not production admin workflows. |
| Locale switch (`/locale/{locale}`) | Global; not admin-specific. |

---

## Test commands

```bash
php artisan test tests/Feature/Admin/
```

---

*Last updated: audit pass aligned with `docs/AUDIT_LOG_GUIDE_AR.md` and `App\Http\Services\AuditService`.*
