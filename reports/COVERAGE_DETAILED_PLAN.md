# Coverage Detailed Master Plan

Generated at: 2026-04-21T17:51:47+00:00

## Baseline

- Line coverage: **85.13%** (5972/7015)
- Method coverage (Clover metric): **72.89%** (492/675)
- Method execution coverage (by executable line): **92.15%** (622/675)
- Files with line gaps: **92**
- Files with method gaps: **29**

## Priority Definition (Execution Meaning)

- **P0**: Critical blind spots (line < 50%) in core business paths. Must be fixed first before any lower priority.
- **P1**: High risk (line 50% to <75%) affecting user-facing flows, admin actions, and medium complexity domain logic.
- **P2**: Medium risk (line 75% to <90%) where edge branches and fallback logic are still missing.
- **P3**: Hardening (line 90% to <100%) to close residual branches and remaining method holes.

## Global Definition of Done (Per File)

- Line coverage target for modified file: **>=95%** (unless proven unreachable and documented).
- All uncovered methods for the file executed at least once in tests.
- At least one negative/guard branch test (validation, auth, or exception) for controllers/services.
- Full suite green + no flaky behavior in 2 consecutive focused reruns.
- Coverage reports regenerated and committed (`docs/reports/COVERAGE_GAPS.*`, `docs/reports/COVERAGE_EXECUTION_PLAN.*`).

## P0 Backlog and Test Mapping

| File | Line % | Uncovered Stmts | Method Exec % | Missing Methods | Test Type | Proposed Test File | Existing Test? |
|---|---:|---:|---:|---|---|---|---|
| `app/Http/Controllers/Admin/AdminFundTransactionController.php` | 18.29 | 67 | 50.00 | show(L31), resolveAuditEntries(L50), exportPdf(L123) | Feature | `tests/Feature/Admin/AdminFundTransactionControllerCoverageTest.php` | No |
| `app/Http/Controllers/Provider/ProviderWalletController.php` | 31.58 | 13 | 50.00 | index(L15) | Feature | `tests/Feature/Provider/ProviderWalletControllerCoverageTest.php` | No |
| `app/Jobs/ProcessRecipientAllowanceRetryJob.php` | 33.33 | 38 | 100.00 | - | Unit | `tests/Unit/Jobs/ProcessRecipientAllowanceRetryJobCoverageTest.php` | No |
| `app/Services/UserService.php` | 38.35 | 127 | 100.00 | - | Unit | `tests/Unit/Services/UserServiceCoverageTest.php` | No |
| `app/Jobs/ProcessRecipientFundRetryJob.php` | 38.78 | 30 | 100.00 | - | Unit | `tests/Unit/Jobs/ProcessRecipientFundRetryJobCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AdminPaymentController.php` | 38.81 | 41 | 80.00 | exportPdf(L96) | Feature | `tests/Feature/Admin/AdminPaymentControllerCoverageTest.php` | No |
| `app/Http/Controllers/Provider/MenuItemController.php` | 44.09 | 52 | 71.43 | index(L24), create(L59) | Feature | `tests/Feature/Provider/MenuItemControllerCoverageTest.php` | No |
| `app/Support/RecipientRequestStatusPresenter.php` | 45.75 | 83 | 50.00 | fulfilled(L128), rejected(L147), adminRejected(L166), cancelled(L186), adminApproved(L225) | Unit | `tests/Unit/Support/RecipientRequestStatusPresenterCoverageTest.php` | No |
| `app/Http/Controllers/Donor/DonationController.php` | 47.83 | 12 | 40.00 | index(L16), receipt(L27), create(L39) | Feature | `tests/Feature/Donor/DonationControllerCoverageTest.php` | No |
| `app/Http/Controllers/Admin/UserManagementController.php` | 48.33 | 31 | 72.73 | create(L49), edit(L71), rolesForUserForms(L97) | Feature | `tests/Feature/Admin/UserManagementControllerCoverageTest.php` | No |

## P1 Backlog and Test Mapping

| File | Line % | Uncovered Stmts | Method Exec % | Missing Methods | Test Type | Proposed Test File | Existing Test? |
|---|---:|---:|---:|---|---|---|---|
| `app/Services/SmsService.php` | 50.00 | 11 | 100.00 | - | Unit | `tests/Unit/Services/SmsServiceCoverageTest.php` | No |
| `app/Models/RequestPaymentLink.php` | 50.00 | 1 | 50.00 | payment(L21) | Unit | `tests/Unit/Models/RequestPaymentLinkCoverageTest.php` | No |
| `app/Http/Controllers/Recipient/ProviderMenuController.php` | 50.00 | 21 | 100.00 | - | Feature | `tests/Feature/Recipient/ProviderMenuControllerCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AdminAllocationController.php` | 56.25 | 7 | 83.33 | status(L24) | Feature | `tests/Feature/Admin/AdminAllocationControllerCoverageTest.php` | No |
| `app/Http/Controllers/Provider/ProviderProofController.php` | 57.58 | 28 | 66.67 | index(L21) | Feature | `tests/Feature/Provider/ProviderProofControllerCoverageTest.php` | No |
| `app/Http/Requests/Auth/LoginRequest.php` | 65.22 | 8 | 100.00 | - | Unit | `tests/Unit/Http/Requests/LoginRequestCoverageTest.php` | No |
| `app/Http/Controllers/Admin/ProviderPayoutController.php` | 65.31 | 34 | 62.50 | index(L23), show(L50), receiptFile(L89) | Feature | `tests/Feature/Admin/ProviderPayoutControllerCoverageTest.php` | No |
| `app/Models/FundTransaction.php` | 66.67 | 2 | 66.67 | orderRedemption(L69), providerPayout(L74) | Unit | `tests/Unit/Models/FundTransactionCoverageTest.php` | No |
| `app/Models/OrderRedemption.php` | 66.67 | 1 | 66.67 | provider(L35) | Unit | `tests/Unit/Models/OrderRedemptionCoverageTest.php` | No |
| `app/Http/Controllers/Auth/EmailVerificationPromptController.php` | 66.67 | 1 | 100.00 | - | Feature | `tests/Feature/Auth/EmailVerificationPromptControllerCoverageTest.php` | No |
| `app/Http/Middleware/EnsurePhoneVerified.php` | 66.67 | 3 | 100.00 | - | Feature | `tests/Feature/Security/EnsurePhoneVerifiedCoverageTest.php` | No |
| `app/Http/Controllers/ProfileController.php` | 68.64 | 37 | 75.00 | uploadPhoto(L91), syncProfilePhoto(L106) | Feature | `tests/Feature/ProfileControllerCoverageTest.php` | No |
| `app/Http/Controllers/Provider/ProviderProfileController.php` | 69.23 | 16 | 75.00 | edit(L22) | Feature | `tests/Feature/Provider/ProviderProfileControllerCoverageTest.php` | No |
| `app/Http/Controllers/Admin/MaintenanceSettingsController.php` | 71.43 | 14 | 100.00 | - | Feature | `tests/Feature/Admin/MaintenanceSettingsControllerCoverageTest.php` | No |
| `app/Models/ProviderProfile.php` | 73.68 | 5 | 83.33 | menuItems(L58) | Unit | `tests/Unit/Models/ProviderProfileCoverageTest.php` | No |
| `app/Services/PaymentService.php` | 74.02 | 66 | 88.89 | findPaymentByExternalIds(L414) | Unit | `tests/Unit/Services/PaymentServiceCoverageTest.php` | No |

## P2 Backlog and Test Mapping

| File | Line % | Uncovered Stmts | Method Exec % | Missing Methods | Test Type | Proposed Test File | Existing Test? |
|---|---:|---:|---:|---|---|---|---|
| `app/Models/RecipientKycDetails.php` | 75.00 | 1 | 50.00 | user(L42) | Unit | `tests/Unit/Models/RecipientKycDetailsCoverageTest.php` | No |
| `app/Models/ProviderMenuItem.php` | 75.00 | 4 | 100.00 | - | Unit | `tests/Unit/Models/ProviderMenuItemCoverageTest.php` | No |
| `app/Providers/RateLimiterServiceProvider.php` | 75.32 | 19 | 100.00 | - | Unit | `tests/Feature/RateLimiting/RateLimiterServiceProviderCoverageTest.php` | No |
| `app/Support/RequestTypeLabel.php` | 76.19 | 5 | 100.00 | - | Unit | `tests/Unit/Support/RequestTypeLabelCoverageTest.php` | No |
| `app/Http/Controllers/Admin/FinancialReportController.php` | 76.92 | 6 | 100.00 | - | Feature | `tests/Feature/Admin/FinancialReportControllerCoverageTest.php` | No |
| `app/Http/Controllers/Auth/ProviderRegistrationController.php` | 77.78 | 22 | 75.00 | showApplication(L54) | Feature | `tests/Feature/Auth/ProviderRegistrationControllerCoverageTest.php` | No |
| `app/Http/Controllers/Auth/PhoneVerificationController.php` | 78.95 | 8 | 100.00 | - | Feature | `tests/Feature/Auth/PhoneVerificationControllerCoverageTest.php` | No |
| `app/Http/Controllers/NotificationController.php` | 79.41 | 7 | 50.00 | markAsRead(L37), markAllAsRead(L50) | Feature | `tests/Feature/NotificationControllerCoverageTest.php` | No |
| `app/Observers/UserObserver.php` | 80.00 | 2 | 100.00 | - | Unit | `tests/Unit/Observers/UserObserverCoverageTest.php` | No |
| `app/Rules/SaudiPhoneUnique.php` | 80.00 | 2 | 100.00 | - | Unit | `tests/Unit/Rules/SaudiPhoneUniqueCoverageTest.php` | No |
| `app/Models/ProviderOperatingInfo.php` | 80.00 | 1 | 50.00 | user(L30) | Unit | `tests/Unit/Models/ProviderOperatingInfoCoverageTest.php` | No |
| `app/Models/ProviderPayout.php` | 80.00 | 4 | 55.56 | confirmedBy(L82), rejectedBy(L87), cancelledBy(L92), fundTransactionOut(L97) | Unit | `tests/Unit/Models/ProviderPayoutCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AdminMenuController.php` | 80.77 | 10 | 100.00 | - | Feature | `tests/Feature/Admin/AdminMenuControllerCoverageTest.php` | No |
| `app/Models/User.php` | 80.95 | 8 | 77.27 | providerPayouts(L100), scopeActive(L177), scopeInactive(L185), emailVerificationRequired(L241), isEmailVerified(L249) | Unit | `tests/Unit/Models/UserCoverageTest.php` | No |
| `app/Notifications/AccountStatusUpdatedNotification.php` | 82.86 | 6 | 100.00 | - | Unit | `tests/Unit/Notifications/AccountStatusUpdatedNotificationCoverageTest.php` | No |
| `app/Http/Controllers/Auth/VerifyEmailController.php` | 83.33 | 2 | 100.00 | - | Feature | `tests/Feature/Auth/VerifyEmailControllerCoverageTest.php` | No |
| `app/Services/OtpService.php` | 83.87 | 15 | 100.00 | - | Unit | `tests/Unit/Services/OtpServiceCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AccountApprovalController.php` | 84.71 | 13 | 100.00 | - | Feature | `tests/Feature/Admin/AccountApprovalControllerCoverageTest.php` | No |
| `app/Services/ResubmitApplicationService.php` | 84.76 | 25 | 100.00 | - | Unit | `tests/Unit/Services/ResubmitApplicationServiceCoverageTest.php` | No |
| `app/Services/RedemptionService.php` | 84.93 | 11 | 100.00 | - | Unit | `tests/Unit/Services/RedemptionServiceCoverageTest.php` | No |
| `app/Http/Middleware/RedirectByRole.php` | 85.71 | 2 | 100.00 | - | Feature | `tests/Feature/Security/RedirectByRoleCoverageTest.php` | No |
| `app/Http/Middleware/EnsureEmailVerified.php` | 85.71 | 1 | 100.00 | - | Feature | `tests/Feature/Security/EnsureEmailVerifiedCoverageTest.php` | No |
| `app/Http/Requests/Recipient/StoreRecipientRequest.php` | 86.36 | 6 | 100.00 | - | Unit | `tests/Unit/Http/Requests/StoreRecipientRequestCoverageTest.php` | No |
| `app/Console/Commands/GenerateWeeklyProviderPayoutsCommand.php` | 87.50 | 1 | 100.00 | - | Feature | `tests/Feature/Admin/GenerateWeeklyProviderPayoutsCommandCommandCoverageTest.php` | No |
| `app/Helpers/PhoneHelper.php` | 88.24 | 4 | 100.00 | - | Unit | `tests/Unit/Helpers/PhoneHelperCoverageTest.php` | No |
| `app/Http/Controllers/Auth/OtpLoginController.php` | 88.89 | 5 | 100.00 | - | Feature | `tests/Feature/Auth/OtpLoginControllerCoverageTest.php` | No |
| `app/Http/Requests/Auth/UpdateResubmitApplicationRequest.php` | 89.29 | 9 | 100.00 | - | Unit | `tests/Unit/Http/Requests/UpdateResubmitApplicationRequestCoverageTest.php` | No |
| `app/Notifications/NewUserRegisteredNotification.php` | 89.47 | 2 | 100.00 | - | Unit | `tests/Unit/Notifications/NewUserRegisteredNotificationCoverageTest.php` | No |
| `app/Notifications/AccountApprovalPendingNotification.php` | 89.47 | 2 | 100.00 | - | Unit | `tests/Unit/Notifications/AccountApprovalPendingNotificationCoverageTest.php` | No |
| `app/Services/NotificationService.php` | 89.66 | 3 | 100.00 | - | Unit | `tests/Unit/Services/NotificationServiceCoverageTest.php` | No |

## P3 Backlog and Test Mapping

| File | Line % | Uncovered Stmts | Method Exec % | Missing Methods | Test Type | Proposed Test File | Existing Test? |
|---|---:|---:|---:|---|---|---|---|
| `app/Http/Controllers/Recipient/RecipientRequestController.php` | 90.24 | 8 | 83.33 | cancelThrottle(L176) | Feature | `tests/Feature/Recipient/RecipientRequestControllerCoverageTest.php` | No |
| `app/Console/Commands/BackfillMenuItemCategories.php` | 90.63 | 3 | 100.00 | - | Feature | `tests/Feature/Admin/BackfillMenuItemCategoriesCommandCoverageTest.php` | No |
| `app/Services/ProviderPayoutLedgerService.php` | 90.91 | 2 | 85.71 | sumEligibleAmount(L44) | Unit | `tests/Unit/Services/ProviderPayoutLedgerServiceCoverageTest.php` | No |
| `app/Http/Controllers/Auth/RegisteredUserController.php` | 91.01 | 8 | 100.00 | - | Feature | `tests/Feature/Auth/RegisteredUserControllerCoverageTest.php` | No |
| `app/Support/FinancialMath.php` | 91.67 | 1 | 100.00 | - | Unit | `tests/Unit/Support/FinancialMathCoverageTest.php` | No |
| `app/Services/AllocationEngineService.php` | 92.11 | 3 | 77.78 | isGloballyPaused(L80), isProviderPaused(L85) | Unit | `tests/Unit/Services/AllocationEngineServiceCoverageTest.php` | No |
| `app/Notifications/NewPendingAdminRequestNotification.php` | 92.31 | 1 | 66.67 | via(L21) | Unit | `tests/Unit/Notifications/NewPendingAdminRequestNotificationCoverageTest.php` | No |
| `app/Console/Commands/VerifyActivityLogIntegrityCommand.php` | 92.31 | 2 | 100.00 | - | Feature | `tests/Feature/Admin/VerifyActivityLogIntegrityCommandCommandCoverageTest.php` | No |
| `app/Notifications/RequestStatusChangedNotification.php` | 92.59 | 4 | 100.00 | - | Unit | `tests/Unit/Notifications/RequestStatusChangedNotificationCoverageTest.php` | No |
| `app/Http/Requests/Provider/UpdateMenuItemRequest.php` | 92.59 | 2 | 100.00 | - | Unit | `tests/Unit/Http/Requests/UpdateMenuItemRequestCoverageTest.php` | No |
| `app/Http/Middleware/EnsureAccountApproved.php` | 93.33 | 1 | 100.00 | - | Feature | `tests/Feature/Security/EnsureAccountApprovedCoverageTest.php` | No |
| `app/Http/Controllers/Provider/ProviderQrController.php` | 93.33 | 1 | 66.67 | scan(L16) | Feature | `tests/Feature/Provider/ProviderQrControllerCoverageTest.php` | No |
| `app/Http/Controllers/Auth/AuthenticatedSessionController.php` | 93.75 | 2 | 100.00 | - | Feature | `tests/Feature/Auth/AuthenticatedSessionControllerCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AuditLogController.php` | 93.94 | 2 | 100.00 | - | Feature | `tests/Feature/Admin/AuditLogControllerCoverageTest.php` | No |
| `app/Http/Controllers/Donor/DonorDashboardController.php` | 94.12 | 7 | 100.00 | - | Feature | `tests/Feature/Donor/DonorDashboardControllerCoverageTest.php` | No |
| `app/Notifications/DocumentsResubmittedForReviewNotification.php` | 94.74 | 1 | 100.00 | - | Unit | `tests/Unit/Notifications/DocumentsResubmittedForReviewNotificationCoverageTest.php` | No |
| `app/Observers/RequestObserver.php` | 95.24 | 1 | 100.00 | - | Unit | `tests/Unit/Observers/RequestObserverCoverageTest.php` | No |
| `app/Models/Request.php` | 95.24 | 1 | 90.91 | requestPaymentLinks(L64) | Unit | `tests/Unit/Models/RequestCoverageTest.php` | No |
| `app/Observers/FundTransactionObserver.php` | 95.45 | 1 | 100.00 | - | Unit | `tests/Unit/Observers/FundTransactionObserverCoverageTest.php` | No |
| `app/Services/ProviderPayoutConfirmationService.php` | 95.51 | 4 | 100.00 | - | Unit | `tests/Unit/Services/ProviderPayoutConfirmationServiceCoverageTest.php` | No |
| `app/Http/Requests/Provider/StoreMenuItemRequest.php` | 96.30 | 1 | 100.00 | - | Unit | `tests/Unit/Http/Requests/StoreMenuItemRequestCoverageTest.php` | No |
| `app/Services/ProviderPayoutGenerationService.php` | 96.39 | 3 | 100.00 | - | Unit | `tests/Unit/Services/ProviderPayoutGenerationServiceCoverageTest.php` | No |
| `app/Services/RecipientRequestSubmissionService.php` | 96.61 | 2 | 100.00 | - | Unit | `tests/Unit/Services/RecipientRequestSubmissionServiceCoverageTest.php` | No |
| `app/Services/AllocationService.php` | 96.84 | 3 | 100.00 | - | Unit | `tests/Unit/Services/AllocationServiceCoverageTest.php` | No |
| `app/Services/LandingPageStatsService.php` | 96.92 | 4 | 100.00 | - | Unit | `tests/Unit/Services/LandingPageStatsServiceCoverageTest.php` | No |
| `app/Support/ProviderDisplay.php` | 97.06 | 1 | 100.00 | - | Unit | `tests/Unit/Support/ProviderDisplayCoverageTest.php` | No |
| `app/Services/MyFatoorahService.php` | 97.30 | 1 | 100.00 | - | Unit | `tests/Unit/Services/MyFatoorahServiceCoverageTest.php` | No |
| `app/Http/Controllers/Auth/ResubmitApplicationController.php` | 97.30 | 1 | 100.00 | - | Feature | `tests/Feature/Auth/ResubmitApplicationControllerCoverageTest.php` | No |
| `app/Services/RecipientAllowanceService.php` | 97.47 | 2 | 100.00 | - | Unit | `tests/Unit/Services/RecipientAllowanceServiceCoverageTest.php` | No |
| `app/Http/Queries/Admin/AdminFundTransactionIndexQuery.php` | 97.56 | 1 | 100.00 | - | Unit | `tests/Unit/Http/Queries/AdminFundTransactionIndexQueryCoverageTest.php` | No |
| `app/Services/SystemWalletService.php` | 97.62 | 2 | 100.00 | - | Unit | `tests/Unit/Services/SystemWalletServiceCoverageTest.php` | No |
| `app/Services/Admin/AdminFinancialService.php` | 97.89 | 3 | 100.00 | - | Unit | `tests/Unit/Services/Admin/AdminFinancialServiceCoverageTest.php` | No |
| `app/Services/Admin/SummaryReportExportService.php` | 98.05 | 5 | 100.00 | - | Unit | `tests/Unit/Services/Admin/SummaryReportExportServiceCoverageTest.php` | No |
| `app/Http/Requests/Auth/StoreProviderRegistrationRequest.php` | 98.15 | 1 | 100.00 | - | Unit | `tests/Unit/Http/Requests/StoreProviderRegistrationRequestCoverageTest.php` | No |
| `app/Http/Controllers/Admin/AdminRequestController.php` | 98.39 | 1 | 100.00 | - | Feature | `tests/Feature/Admin/AdminRequestControllerCoverageTest.php` | No |
| `app/Http/Controllers/Provider/ProviderRequestController.php` | 98.64 | 2 | 100.00 | - | Feature | `tests/Feature/Provider/ProviderRequestControllerCoverageTest.php` | No |

## Method-Gap Focus List (Unexecuted Methods)

| File | Uncovered Methods | Test Type |
|---|---|---|
| `app/Http/Controllers/Donor/DonationController.php` | index(L16), receipt(L27), create(L39) | Feature |
| `app/Models/RecipientKycDetails.php` | user(L42) | Unit |
| `app/Models/RequestPaymentLink.php` | payment(L21) | Unit |
| `app/Support/RecipientRequestStatusPresenter.php` | fulfilled(L128), rejected(L147), adminRejected(L166), cancelled(L186), adminApproved(L225) | Unit |
| `app/Models/ProviderOperatingInfo.php` | user(L30) | Unit |
| `app/Http/Controllers/Admin/AdminFundTransactionController.php` | show(L31), resolveAuditEntries(L50), exportPdf(L123) | Feature |
| `app/Http/Controllers/NotificationController.php` | markAsRead(L37), markAllAsRead(L50) | Feature |
| `app/Http/Controllers/Provider/ProviderWalletController.php` | index(L15) | Feature |
| `app/Models/ProviderPayout.php` | confirmedBy(L82), rejectedBy(L87), cancelledBy(L92), fundTransactionOut(L97) | Unit |
| `app/Http/Controllers/Admin/ProviderPayoutController.php` | index(L23), show(L50), receiptFile(L89) | Feature |
| `app/Models/OrderRedemption.php` | provider(L35) | Unit |
| `app/Notifications/NewPendingAdminRequestNotification.php` | via(L21) | Unit |
| `app/Models/FundTransaction.php` | orderRedemption(L69), providerPayout(L74) | Unit |
| `app/Http/Controllers/Provider/ProviderProofController.php` | index(L21) | Feature |
| `app/Http/Controllers/Provider/ProviderQrController.php` | scan(L16) | Feature |
| `app/Http/Controllers/Provider/MenuItemController.php` | index(L24), create(L59) | Feature |
| `app/Http/Controllers/Admin/UserManagementController.php` | create(L49), edit(L71), rolesForUserForms(L97) | Feature |
| `app/Http/Controllers/Provider/ProviderProfileController.php` | edit(L22) | Feature |
| `app/Http/Controllers/ProfileController.php` | uploadPhoto(L91), syncProfilePhoto(L106) | Feature |
| `app/Http/Controllers/Auth/ProviderRegistrationController.php` | showApplication(L54) | Feature |
| `app/Models/User.php` | providerPayouts(L100), scopeActive(L177), scopeInactive(L185), emailVerificationRequired(L241), isEmailVerified(L249) | Unit |
| `app/Services/AllocationEngineService.php` | isGloballyPaused(L80), isProviderPaused(L85) | Unit |
| `app/Http/Controllers/Admin/AdminPaymentController.php` | exportPdf(L96) | Feature |
| `app/Models/ProviderProfile.php` | menuItems(L58) | Unit |
| `app/Http/Controllers/Recipient/RecipientRequestController.php` | cancelThrottle(L176) | Feature |
| `app/Http/Controllers/Admin/AdminAllocationController.php` | status(L24) | Feature |
| `app/Services/ProviderPayoutLedgerService.php` | sumEligibleAmount(L44) | Unit |
| `app/Services/PaymentService.php` | findPaymentByExternalIds(L414) | Unit |
| `app/Models/Request.php` | requestPaymentLinks(L64) | Unit |

## Docs-to-Tests Audit Matrix (`/docs`)

| Docs File | Audit Mapping (Feature/Code Area -> Required Tests) |
|---|---|
| `docs/ADMIN_FINANCE_MODULE.md` | Admin financial controllers + payout services | tests/Feature/Admin/AdminFinancialManagementTest.php, tests/Feature/Admin/ProviderPayoutFlowTest.php, tests/Unit/Services/Admin/SummaryReportExportServiceTest.php |
| `docs/AUDIT_LOG_GUIDE_AR.md` | Audit flows and integrity | tests/Feature/Admin/AuditLogTest.php, tests/Feature/Audit/AuditHashIntegrityTest.php, tests/Feature/Audit/VerifyActivityLogIntegrityCommandTest.php |
| `docs/CHANGELOG_REQUEST_AND_WALLET.md` | Request + wallet behavior deltas | tests/Feature/Provider/ProviderWalletReceiptTest.php, tests/Feature/Recipient/RecipientInterfaceTest.php, tests/Unit/Services/SystemWalletServiceTest.php |
| `docs/DEPLOYMENT.md` | Operational doc (no direct code target) | smoke only via full suite |
| `docs/ECS-64_111_112_QR_Fulfillment_Documentation.md` | QR fulfillment flow | tests/Feature/Provider/ProviderQrRedemptionTest.php, tests/Feature/Provider/ProviderRequestFlowTest.php |
| `docs/fat-controller-architecture-review.md` | Controller decomposition risks | enforce by controller coverage waves P0/P1 + service isolation tests |
| `docs/FINANCIAL_BCMATH.md` | Financial math precision | tests/Unit/Support/FinancialMathTest.php, tests/Unit/Donor/AllocationServiceTest.php |
| `docs/NOTIFICATIONS.md` | Notification triggers/channels | tests/Feature/Notifications/NotificationCatalogTest.php, tests/Unit/Notifications/P1NotificationsTest.php, tests/Unit/Notifications/P3NotificationsCoverageTest.php |
| `docs/NUBL_Project_Review_Report.md` | Project review/meta (no direct code target) | validate by regression run only |
| `docs/PAYMENT_REFERENCE.md` | Payment callbacks and references | tests/Feature/Donor/PaymentFlowTest.php, tests/Feature/Donor/PaymentCallbackControllerPagesTest.php, tests/Unit/Services/PaymentServiceCoverageTest.php |
| `docs/PERMISSIONS_AND_RBAC_AR.md` | RBAC and permissions | tests/Feature/Admin/RoleManagementTest.php, tests/Feature/Admin/RoleControllerCoverageTest.php, tests/Feature/Security/MiddlewareGuardsTest.php, tests/Unit/Permissions/PermissionDefinitionsTest.php |
| `docs/playwright.md` | E2E guidance (out of phpunit scope) | keep as optional browser checks after backend pass |
| `docs/PROVIDER_BANK_PAYOUT.md` | Provider bank payout lifecycle | tests/Feature/Admin/ProviderPayoutFlowTest.php, tests/Unit/Services/ProviderPayoutConfirmationServiceTest.php, tests/Unit/Services/ProviderPayoutGenerationServiceCoverageTest.php |
| `docs/PROVIDER_REQUEST_FLOW_TESTS.md` | Provider request lifecycle | tests/Feature/Provider/ProviderRequestFlowTest.php, tests/Feature/Recipient/RecipientRequestSubmissionTest.php |
| `docs/QR_CODE_REDEMPTION.md` | QR scan/redemption | tests/Feature/Provider/ProviderQrRedemptionTest.php, tests/Feature/Provider/ProviderQrControllerCoverageTest.php |
| `docs/RATE_LIMITING.md` | Rate-limiter behavior | tests/Feature/RateLimiting/EnabledRateLimitingTest.php, tests/Feature/RateLimiting/RateLimiterServiceProviderTest.php |
| `docs/README_DEV.md` | Developer onboarding/meta | no direct coverage target |
| `docs/RECIPIENT_WEEKLY_ALLOWANCE.md` | Weekly allowance + retries | tests/Unit/Recipient/RecipientAllowanceServiceTest.php, tests/Unit/Jobs/ProcessRecipientAllowanceRetryJobCoverageTest.php, tests/Unit/Jobs/ProcessRecipientFundRetryJobCoverageTest.php |
| `docs/reports/ADMIN_AUDIT_COVERAGE.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/COVERAGE_DETAILED_PLAN.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/COVERAGE_EXECUTION_PLAN.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/COVERAGE_GAPS.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/RBAC_AUDIT_REPORT.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/README_RECIPIENT_REQUEST_SUBMISSION_TESTS.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/RECIPIENT_REQUEST_SUBMISSION_TESTS.md` | Generated report artifact | keep synchronized after each wave |
| `docs/reports/VERIFICATION_REPORT.md` | Generated report artifact | keep synchronized after each wave |
| `docs/REQUEST_STATUSES.md` | Request status transitions | tests/Unit/Support/RequestTypeAndStatusPresenterTest.php, tests/Feature/Recipient/RecipientRequestSubmissionTest.php |
| `docs/SYSTEM_WALLET_DONATION_FLOW.md` | System wallet + donation allocation | tests/Feature/Donor/AllocationFifoTest.php, tests/Unit/Services/SystemWalletServiceTest.php, tests/Unit/Services/AllocationServiceTest.php |

## Execution Order and Control Gates

1. Execute all P0 files (feature + unit) and close all listed missing methods for those files.
2. Run focused subset for modified suites, then full `php artisan test --coverage` gate.
3. Move to P1 only when no P0 file remains below 90% line and no P0 missing method remains unexecuted.
4. Repeat same gate for P1, then proceed to P2, then P3.
5. After each wave, regenerate reports and refresh this plan to keep backlog authoritative.

## Recommended Commands (Per Wave)

```bash
# Full suite + coverage artifact
php artisan test --coverage-clover=storage/coverage/clover.xml --coverage-text

# Suggested focused runs while implementing each wave (examples)
php artisan test tests/Feature/Admin tests/Feature/Provider tests/Feature/Donor tests/Feature/Recipient
php artisan test tests/Unit/Services tests/Unit/Jobs tests/Unit/Models tests/Unit/Support
```
