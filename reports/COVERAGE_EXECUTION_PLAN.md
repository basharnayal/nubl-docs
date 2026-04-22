# Coverage Execution Plan

Generated at: 2026-04-21T17:51:47+00:00

## Current Baseline

- Line coverage: **85.13%** (5972/7015)
- Method coverage (Clover metric): **72.89%** (492/675)
- Method execution coverage (method-line count>0): **92.15%** (622/675)
- Files with line gaps: **92**
- Files with unexecuted methods: **29**
- Priority split: P0=10, P1=16, P2=30, P3=36

## Line Gaps Backlog (All Files)

| Priority | File | Line % | Statements | Uncovered Stmts | Test Type | Focus |
|---|---|---:|---:|---:|---|---|
| P0 | `app\Http\Controllers\Admin\AdminFundTransactionController.php` | 18.29 | 15/82 | 67 | Feature | happy-path + authz + validation/error branches |
| P0 | `app\Http\Controllers\Provider\ProviderWalletController.php` | 31.58 | 6/19 | 13 | Feature | happy-path + authz + validation/error branches |
| P0 | `app\Jobs\ProcessRecipientAllowanceRetryJob.php` | 33.33 | 19/57 | 38 | Unit | handle() success/failure, retry/idempotency behavior |
| P0 | `app\Services\UserService.php` | 38.35 | 79/206 | 127 | Unit | success/failure branches + edge cases + side-effects |
| P0 | `app\Jobs\ProcessRecipientFundRetryJob.php` | 38.78 | 19/49 | 30 | Unit | handle() success/failure, retry/idempotency behavior |
| P0 | `app\Http\Controllers\Admin\AdminPaymentController.php` | 38.81 | 26/67 | 41 | Feature | happy-path + authz + validation/error branches |
| P0 | `app\Http\Controllers\Provider\MenuItemController.php` | 44.09 | 41/93 | 52 | Feature | happy-path + authz + validation/error branches |
| P0 | `app\Support\RecipientRequestStatusPresenter.php` | 45.75 | 70/153 | 83 | Unit | fallbacks and branch permutations |
| P0 | `app\Http\Controllers\Donor\DonationController.php` | 47.83 | 11/23 | 12 | Feature | happy-path + authz + validation/error branches |
| P0 | `app\Http\Controllers\Admin\UserManagementController.php` | 48.33 | 29/60 | 31 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Controllers\Recipient\ProviderMenuController.php` | 50.00 | 21/42 | 21 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Models\RequestPaymentLink.php` | 50.00 | 1/2 | 1 | Unit | relationships + casts + scopes/accessors |
| P1 | `app\Services\SmsService.php` | 50.00 | 11/22 | 11 | Unit | success/failure branches + edge cases + side-effects |
| P1 | `app\Http\Controllers\Admin\AdminAllocationController.php` | 56.25 | 9/16 | 7 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Controllers\Provider\ProviderProofController.php` | 57.58 | 38/66 | 28 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Requests\Auth\LoginRequest.php` | 65.22 | 15/23 | 8 | Unit | rules(), authorize(), conditional branches |
| P1 | `app\Http\Controllers\Admin\ProviderPayoutController.php` | 65.31 | 64/98 | 34 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Controllers\Auth\EmailVerificationPromptController.php` | 66.67 | 2/3 | 1 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Middleware\EnsurePhoneVerified.php` | 66.67 | 6/9 | 3 | Feature | allow/deny redirects and passthrough flow |
| P1 | `app\Models\FundTransaction.php` | 66.67 | 4/6 | 2 | Unit | relationships + casts + scopes/accessors |
| P1 | `app\Models\OrderRedemption.php` | 66.67 | 2/3 | 1 | Unit | relationships + casts + scopes/accessors |
| P1 | `app\Http\Controllers\ProfileController.php` | 68.64 | 81/118 | 37 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Controllers\Provider\ProviderProfileController.php` | 69.23 | 36/52 | 16 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Http\Controllers\Admin\MaintenanceSettingsController.php` | 71.43 | 35/49 | 14 | Feature | happy-path + authz + validation/error branches |
| P1 | `app\Models\ProviderProfile.php` | 73.68 | 14/19 | 5 | Unit | relationships + casts + scopes/accessors |
| P1 | `app\Services\PaymentService.php` | 74.02 | 188/254 | 66 | Unit | success/failure branches + edge cases + side-effects |
| P2 | `app\Models\ProviderMenuItem.php` | 75.00 | 12/16 | 4 | Unit | relationships + casts + scopes/accessors |
| P2 | `app\Models\RecipientKycDetails.php` | 75.00 | 3/4 | 1 | Unit | relationships + casts + scopes/accessors |
| P2 | `app\Providers\RateLimiterServiceProvider.php` | 75.32 | 58/77 | 19 | Unit | branch and edge coverage |
| P2 | `app\Support\RequestTypeLabel.php` | 76.19 | 16/21 | 5 | Unit | fallbacks and branch permutations |
| P2 | `app\Http\Controllers\Admin\FinancialReportController.php` | 76.92 | 20/26 | 6 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Http\Controllers\Auth\ProviderRegistrationController.php` | 77.78 | 77/99 | 22 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Http\Controllers\Auth\PhoneVerificationController.php` | 78.95 | 30/38 | 8 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Http\Controllers\NotificationController.php` | 79.41 | 27/34 | 7 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Models\ProviderOperatingInfo.php` | 80.00 | 4/5 | 1 | Unit | relationships + casts + scopes/accessors |
| P2 | `app\Models\ProviderPayout.php` | 80.00 | 16/20 | 4 | Unit | relationships + casts + scopes/accessors |
| P2 | `app\Observers\UserObserver.php` | 80.00 | 8/10 | 2 | Unit | branch and edge coverage |
| P2 | `app\Rules\SaudiPhoneUnique.php` | 80.00 | 8/10 | 2 | Unit | branch and edge coverage |
| P2 | `app\Http\Controllers\Admin\AdminMenuController.php` | 80.77 | 42/52 | 10 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Models\User.php` | 80.95 | 34/42 | 8 | Unit | relationships + casts + scopes/accessors |
| P2 | `app\Notifications\AccountStatusUpdatedNotification.php` | 82.86 | 29/35 | 6 | Unit | via()/toMail()/toArray() per event branch |
| P2 | `app\Http\Controllers\Auth\VerifyEmailController.php` | 83.33 | 10/12 | 2 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Services\OtpService.php` | 83.87 | 78/93 | 15 | Unit | success/failure branches + edge cases + side-effects |
| P2 | `app\Http\Controllers\Admin\AccountApprovalController.php` | 84.71 | 72/85 | 13 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Services\ResubmitApplicationService.php` | 84.76 | 139/164 | 25 | Unit | success/failure branches + edge cases + side-effects |
| P2 | `app\Services\RedemptionService.php` | 84.93 | 62/73 | 11 | Unit | success/failure branches + edge cases + side-effects |
| P2 | `app\Http\Middleware\EnsureEmailVerified.php` | 85.71 | 6/7 | 1 | Feature | allow/deny redirects and passthrough flow |
| P2 | `app\Http\Middleware\RedirectByRole.php` | 85.71 | 12/14 | 2 | Feature | allow/deny redirects and passthrough flow |
| P2 | `app\Http\Requests\Recipient\StoreRecipientRequest.php` | 86.36 | 38/44 | 6 | Unit | rules(), authorize(), conditional branches |
| P2 | `app\Console\Commands\GenerateWeeklyProviderPayoutsCommand.php` | 87.50 | 7/8 | 1 | Feature | artisan command outputs + branch conditions |
| P2 | `app\Helpers\PhoneHelper.php` | 88.24 | 30/34 | 4 | Unit | branch and edge coverage |
| P2 | `app\Http\Controllers\Auth\OtpLoginController.php` | 88.89 | 40/45 | 5 | Feature | happy-path + authz + validation/error branches |
| P2 | `app\Http\Requests\Auth\UpdateResubmitApplicationRequest.php` | 89.29 | 75/84 | 9 | Unit | rules(), authorize(), conditional branches |
| P2 | `app\Notifications\AccountApprovalPendingNotification.php` | 89.47 | 17/19 | 2 | Unit | via()/toMail()/toArray() per event branch |
| P2 | `app\Notifications\NewUserRegisteredNotification.php` | 89.47 | 17/19 | 2 | Unit | via()/toMail()/toArray() per event branch |
| P2 | `app\Services\NotificationService.php` | 89.66 | 26/29 | 3 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Http\Controllers\Recipient\RecipientRequestController.php` | 90.24 | 74/82 | 8 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Console\Commands\BackfillMenuItemCategories.php` | 90.62 | 29/32 | 3 | Feature | artisan command outputs + branch conditions |
| P3 | `app\Services\ProviderPayoutLedgerService.php` | 90.91 | 20/22 | 2 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Http\Controllers\Auth\RegisteredUserController.php` | 91.01 | 81/89 | 8 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Support\FinancialMath.php` | 91.67 | 11/12 | 1 | Unit | fallbacks and branch permutations |
| P3 | `app\Services\AllocationEngineService.php` | 92.11 | 35/38 | 3 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Console\Commands\VerifyActivityLogIntegrityCommand.php` | 92.31 | 24/26 | 2 | Feature | artisan command outputs + branch conditions |
| P3 | `app\Notifications\NewPendingAdminRequestNotification.php` | 92.31 | 12/13 | 1 | Unit | via()/toMail()/toArray() per event branch |
| P3 | `app\Http\Requests\Provider\UpdateMenuItemRequest.php` | 92.59 | 25/27 | 2 | Unit | rules(), authorize(), conditional branches |
| P3 | `app\Notifications\RequestStatusChangedNotification.php` | 92.59 | 50/54 | 4 | Unit | via()/toMail()/toArray() per event branch |
| P3 | `app\Http\Controllers\Provider\ProviderQrController.php` | 93.33 | 14/15 | 1 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Http\Middleware\EnsureAccountApproved.php` | 93.33 | 14/15 | 1 | Feature | allow/deny redirects and passthrough flow |
| P3 | `app\Http\Controllers\Auth\AuthenticatedSessionController.php` | 93.75 | 30/32 | 2 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Http\Controllers\Admin\AuditLogController.php` | 93.94 | 31/33 | 2 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Http\Controllers\Donor\DonorDashboardController.php` | 94.12 | 112/119 | 7 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Notifications\DocumentsResubmittedForReviewNotification.php` | 94.74 | 18/19 | 1 | Unit | via()/toMail()/toArray() per event branch |
| P3 | `app\Models\Request.php` | 95.24 | 20/21 | 1 | Unit | relationships + casts + scopes/accessors |
| P3 | `app\Observers\RequestObserver.php` | 95.24 | 20/21 | 1 | Unit | branch and edge coverage |
| P3 | `app\Observers\FundTransactionObserver.php` | 95.45 | 21/22 | 1 | Unit | branch and edge coverage |
| P3 | `app\Services\ProviderPayoutConfirmationService.php` | 95.51 | 85/89 | 4 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Http\Requests\Provider\StoreMenuItemRequest.php` | 96.30 | 26/27 | 1 | Unit | rules(), authorize(), conditional branches |
| P3 | `app\Services\ProviderPayoutGenerationService.php` | 96.39 | 80/83 | 3 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\RecipientRequestSubmissionService.php` | 96.61 | 57/59 | 2 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\AllocationService.php` | 96.84 | 92/95 | 3 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\LandingPageStatsService.php` | 96.92 | 126/130 | 4 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Support\ProviderDisplay.php` | 97.06 | 33/34 | 1 | Unit | fallbacks and branch permutations |
| P3 | `app\Http\Controllers\Auth\ResubmitApplicationController.php` | 97.30 | 36/37 | 1 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Services\MyFatoorahService.php` | 97.30 | 36/37 | 1 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\RecipientAllowanceService.php` | 97.47 | 77/79 | 2 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Http\Queries\Admin\AdminFundTransactionIndexQuery.php` | 97.56 | 40/41 | 1 | Unit | branch and edge coverage |
| P3 | `app\Services\SystemWalletService.php` | 97.62 | 82/84 | 2 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\Admin\AdminFinancialService.php` | 97.89 | 139/142 | 3 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Services\Admin\SummaryReportExportService.php` | 98.05 | 252/257 | 5 | Unit | success/failure branches + edge cases + side-effects |
| P3 | `app\Http\Requests\Auth\StoreProviderRegistrationRequest.php` | 98.15 | 53/54 | 1 | Unit | rules(), authorize(), conditional branches |
| P3 | `app\Http\Controllers\Admin\AdminRequestController.php` | 98.39 | 61/62 | 1 | Feature | happy-path + authz + validation/error branches |
| P3 | `app\Http\Controllers\Provider\ProviderRequestController.php` | 98.64 | 145/147 | 2 | Feature | happy-path + authz + validation/error branches |

## Method Gaps Backlog (Unexecuted Methods)

| File | Method Exec % | Covered Methods | Missing Methods | Test Type |
|---|---:|---:|---|---|
| `app\Http\Controllers\Donor\DonationController.php` | 40.00 | 2/5 | index (L16), receipt (L27), create (L39) | Feature |
| `app\Http\Controllers\Admin\AdminFundTransactionController.php` | 50.00 | 3/6 | show (L31), resolveAuditEntries (L50), exportPdf (L123) | Feature |
| `app\Http\Controllers\NotificationController.php` | 50.00 | 2/4 | markAsRead (L37), markAllAsRead (L50) | Feature |
| `app\Http\Controllers\Provider\ProviderWalletController.php` | 50.00 | 1/2 | index (L15) | Feature |
| `app\Models\ProviderOperatingInfo.php` | 50.00 | 1/2 | user (L30) | Unit |
| `app\Models\RecipientKycDetails.php` | 50.00 | 1/2 | user (L42) | Unit |
| `app\Models\RequestPaymentLink.php` | 50.00 | 1/2 | payment (L21) | Unit |
| `app\Support\RecipientRequestStatusPresenter.php` | 50.00 | 5/10 | fulfilled (L128), rejected (L147), adminRejected (L166), cancelled (L186), adminApproved (L225) | Unit |
| `app\Models\ProviderPayout.php` | 55.56 | 5/9 | confirmedBy (L82), rejectedBy (L87), cancelledBy (L92), fundTransactionOut (L97) | Unit |
| `app\Http\Controllers\Admin\ProviderPayoutController.php` | 62.50 | 5/8 | index (L23), show (L50), receiptFile (L89) | Feature |
| `app\Http\Controllers\Provider\ProviderProofController.php` | 66.67 | 2/3 | index (L21) | Feature |
| `app\Http\Controllers\Provider\ProviderQrController.php` | 66.67 | 2/3 | scan (L16) | Feature |
| `app\Models\FundTransaction.php` | 66.67 | 4/6 | orderRedemption (L69), providerPayout (L74) | Unit |
| `app\Models\OrderRedemption.php` | 66.67 | 2/3 | provider (L35) | Unit |
| `app\Notifications\NewPendingAdminRequestNotification.php` | 66.67 | 2/3 | via (L21) | Unit |
| `app\Http\Controllers\Provider\MenuItemController.php` | 71.43 | 5/7 | index (L24), create (L59) | Feature |
| `app\Http\Controllers\Admin\UserManagementController.php` | 72.73 | 8/11 | create (L49), edit (L71), rolesForUserForms (L97) | Feature |
| `app\Http\Controllers\Auth\ProviderRegistrationController.php` | 75.00 | 3/4 | showApplication (L54) | Feature |
| `app\Http\Controllers\ProfileController.php` | 75.00 | 6/8 | uploadPhoto (L91), syncProfilePhoto (L106) | Feature |
| `app\Http\Controllers\Provider\ProviderProfileController.php` | 75.00 | 3/4 | edit (L22) | Feature |
| `app\Models\User.php` | 77.27 | 17/22 | providerPayouts (L100), scopeActive (L177), scopeInactive (L185), emailVerificationRequired (L241), isEmailVerified (L249) | Unit |
| `app\Services\AllocationEngineService.php` | 77.78 | 7/9 | isGloballyPaused (L80), isProviderPaused (L85) | Unit |
| `app\Http\Controllers\Admin\AdminPaymentController.php` | 80.00 | 4/5 | exportPdf (L96) | Feature |
| `app\Http\Controllers\Admin\AdminAllocationController.php` | 83.33 | 5/6 | status (L24) | Feature |
| `app\Http\Controllers\Recipient\RecipientRequestController.php` | 83.33 | 5/6 | cancelThrottle (L176) | Feature |
| `app\Models\ProviderProfile.php` | 83.33 | 5/6 | menuItems (L58) | Unit |
| `app\Services\ProviderPayoutLedgerService.php` | 85.71 | 6/7 | sumEligibleAmount (L44) | Unit |
| `app\Services\PaymentService.php` | 88.89 | 8/9 | findPaymentByExternalIds (L414) | Unit |
| `app\Models\Request.php` | 90.91 | 10/11 | requestPaymentLinks (L64) | Unit |

## Execution Waves (Detailed)

### Wave 1 (P0: <50% line coverage)
- Goal: close the highest-risk blind spots first (controller/service/job core behavior).
- Scope: files marked P0 in the table above.
- Definition of Done per file:
  - happy path covered
  - authorization/guard branch covered where applicable
  - failure/validation/error branch covered
  - method gap rows for that file resolved (if listed).

### Wave 2 (P1: 50%-74.99%)
- Goal: complete CRUD/report/export and medium-complexity domain paths.
- Scope: files marked P1.
- Definition of Done: line >=90% for each file, no unexecuted core methods.

### Wave 3 (P2: 75%-89.99%)
- Goal: close edge branches, fallback behavior, and guard rails.
- Scope: files marked P2.

### Wave 4 (P3: 90%-99.99% + method leftovers)
- Goal: near-100 hardening and branch parity on remaining conditionals.
- Scope: files marked P3 and remaining method-gap rows.

### Final Verification Gate
- Run full suite with coverage.
- Recompute this report and ensure:
  - line gap table empty
  - method gap table empty
  - no flaky tests in two consecutive runs.