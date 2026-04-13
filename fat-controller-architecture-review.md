# Fat Controller Architecture Review

**Project:** NUBL Laravel Application
**Date:** April 8, 2026
**Scope:** All 40+ app controllers reviewed for business logic that belongs in services/use-cases
**Methodology:** Line-by-line review of every controller method, cross-referenced against 20 existing services

---

## Executive Summary

Out of 40+ controllers, **7 controllers contain genuine fat-controller violations** that warrant refactoring. The codebase already demonstrates strong service patterns in many areas (PaymentService, RedemptionService, AllocationEngineService, ResubmitApplicationService), but the registration flows and several request-processing controllers have not yet been refactored to match.

**Findings by severity:**

| Severity | Count |
|----------|-------|
| High     | 4     |
| Medium   | 3     |

**Key patterns observed:**
- Registration controllers (Provider and Recipient) were never migrated to use `UserService` or a dedicated registration service
- Request workflow actions (adopt/approve/reject) are duplicated across `ProviderRequestController` and `AdminRequestController` without a shared workflow service
- Proof upload flow mixes file I/O, base64 decoding, DB transaction, status transition, and notifications in one controller method

---

## Findings

---

### FC-01: Provider Registration — Monolithic store() with DB::transaction, 5 model creates, file I/O, notifications, and OTP

- **Severity:** High
- **File:** `app/Http/Controllers/Auth/ProviderRegistrationController.php`
- **Method:** `store()` (lines 71–175, ~105 lines)
- **Category:** Fat Controller
- **Evidence:**

```php
// Lines 76-84: File uploads before transaction
$licensePath = $request->file('business_license')->store('provider_documents', 'local');
$idPath = $request->file('id_or_iqama')->store('provider_documents', 'local');
$profileLogoPath = null;
if ($request->hasFile('profile_logo')) {
    $profileLogoPath = $request->file('profile_logo')->store('provider-logos', 'public');
}

// Lines 87-151: Large DB::transaction with 5 model creates + side effects
$user = DB::transaction(function () use (...) {
    $user = User::create([...]);          // 1. User
    $user->assignRole('provider');         // 2. Role
    ProviderProfile::create([...]);        // 3. Profile
    ProviderOperatingInfo::create([...]);  // 4. Operating info
    ProviderFinancialInfo::create([...]);  // 5. Financial info
    ProviderDocuments::create([...]);      // 6. Documents
    event(new Registered($user));          // Event
    $this->notificationService->sendNewUserRegisteredToAdmins($user); // Notification
    Auth::login($user);                   // Auth
    if (config('app.phone_verification_enabled', true)) {
        $this->otpService->sendOtp($user); // OTP
    }
    return $user;
});

// Lines 158-164: Manual file cleanup on exception
catch (\Throwable $e) {
    Storage::disk('local')->delete($licensePath);
    Storage::disk('local')->delete($idPath);
    if ($profileLogoPath) {
        Storage::disk('public')->delete($profileLogoPath);
    }
    throw $e;
}
```

- **Existing service bypassed:** `UserService` exists and already creates users with type-specific profiles for admin flows, but is not used here. `ResubmitApplicationService` already handles the similar resubmission flow via a service — this registration flow never received the same treatment.
- **Recommended refactor:** Extract all business logic into a dedicated `ProviderRegistrationService::register(array $validated, array $files): User` that handles: file storage with cleanup, DB::transaction with all model creates, event dispatch, notification, OTP, and audit logging. Controller reduces to ~8 lines.
- **Suggested service name:** `ProviderRegistrationService`
- **Suggested method signature:** `register(array $validatedData, OperatingHours $hours, UploadedFile $license, UploadedFile $idDoc, ?UploadedFile $logo): User`
- **Refactor priority:** P1 — This is the single largest fat controller method. Testing registration logic currently requires HTTP tests; extracting it enables unit testing of business rules.

---

### FC-02: Recipient Registration — DB::transaction, base64 image handling, 3 model creates, file cleanup, notifications

- **Severity:** High
- **File:** `app/Http/Controllers/Auth/RegisteredUserController.php`
- **Method:** `storeRecipient()` (lines 88–168, ~80 lines)
- **Category:** Fat Controller
- **Evidence:**

```php
// Lines 92-93: Base64 image processing inline
$idPhotoPath = $this->storeBase64Image($validated['id_photo_base64'], 'recipient_id_photos');
$addressPhotoPath = $this->storeBase64Image($validated['address_confirmation_base64'], 'recipient_address_photos');

// Lines 100-145: DB::transaction with multiple creates
$user = DB::transaction(function () use (...) {
    $user = User::create([...]);
    $user->assignRole('recipient');
    if ($request->hasFile('profile_logo')) {
        $profileLogoPath = $request->file('profile_logo')->store('recipient-logos', 'public');
    }
    RecipientProfile::create([...]);
    RecipientKycDetails::create([...]);
    event(new Registered($user));
    $this->notificationService->sendNewUserRegisteredToAdmins($user);
    Auth::login($user);
    if (config('app.phone_verification_enabled', true)) {
        $this->otpService->sendOtp($user);
    }
    return $user;
});

// Lines 152-158: Same file cleanup pattern as FC-01
catch (\Throwable $e) {
    Storage::disk('local')->delete($idPhotoPath);
    Storage::disk('local')->delete($addressPhotoPath);
    if ($profileLogoPath) { Storage::disk('public')->delete($profileLogoPath); }
    throw $e;
}
```

- **Additional concern:** The `storeDonor()` method (lines 46–85) is less severe but still contains User::create + assignRole + event + notification + audit + Auth::login + OTP in a single controller method (~38 lines, no transaction wrapping).
- **Existing service bypassed:** `UserService::createUser()` handles this pattern for admin-created users but is not reused here. `storeBase64Image()` utility duplicates similar logic in `ResubmitApplicationService`.
- **Recommended refactor:** Create `RecipientRegistrationService::register()` and `DonorRegistrationService::register()` (or a unified `RegistrationService` with separate methods). Extract `storeBase64Image` to a shared `ImageStorageService`.
- **Suggested service name:** `RecipientRegistrationService`
- **Suggested method signature:** `register(array $validatedData, string $idPhotoBase64, string $addressBase64, ?UploadedFile $logo): User`
- **Refactor priority:** P1 — Same rationale as FC-01. The base64 image handling utility is also duplicated.

---

### FC-03: Provider Request Actions — Three distinct workflows in one DB::transaction with state transitions, wallet checks, redemption generation

- **Severity:** High
- **File:** `app/Http/Controllers/Provider/ProviderRequestController.php`
- **Method:** `update()` (lines 142–225, ~83 lines)
- **Category:** Fat Controller / Architecture
- **Evidence:**

```php
// Lines 154-222: Three distinct business workflows inside one transaction
$result = DB::transaction(function () use ($requestModel, $action, $validated) {
    if ($action === 'adopt') {
        $requestModel->update(['status' => 'APPROVED', 'funding_source' => 'PROVIDER_ADOPTION']);
        \App\Services\RedemptionService::generateForRequest($requestModel);
        $this->auditService->log('request', 'provider_credited_as_donor', [...]);
        $this->notificationService->sendRequestStatusChanged($requestModel->load('recipient'), 'APPROVED');
        $this->auditService->log('notification', 'sent', [...]);
    } elseif ($action === 'approve') {
        $amount = (float) $requestModel->reserved_amount;
        if (! $this->systemWalletService->hasSufficientBalance($amount)) {
            return back()->with('error', ...);
        }
        $requestModel->update(['status' => 'REDEEMABLE', 'funding_source' => 'CITY_FUND']);
        \App\Services\RedemptionService::generateForRequest($requestModel);
        $this->auditService->log('request', 'approved_city_fund', [...]);
        $this->notificationService->sendRequestStatusChanged(...);
        $this->auditService->log('notification', 'sent', [...]);
    } elseif ($action === 'reject') {
        $requestModel->update(['status' => 'REJECTED', ...]);
        $this->auditService->log('request', 'rejected_by_provider', [...]);
        $this->notificationService->sendRequestStatusChanged(...);
        $this->auditService->log('notification', 'sent', [...]);
    }
});
```

- **Key risks:** (1) Returning a redirect from inside `DB::transaction` is a bug — the transaction closure returns the redirect response but should return data, not HTTP responses. (2) Three separate business workflows with different side effects are interleaved. (3) Static call to `RedemptionService::generateForRequest` mixes static and injected service usage.
- **Existing service bypassed:** `RedemptionService` is partially used (static call), but no unified request-action service exists. The audit+notification pattern is duplicated across all three branches.
- **Recommended refactor:** Create `ProviderRequestActionService` with three dedicated methods. Each method encapsulates: status transition, redemption generation (if applicable), wallet check (if applicable), audit logging, and notifications.
- **Suggested service name:** `ProviderRequestActionService`
- **Suggested method signatures:**
  - `adoptRequest(RequestModel $request, User $provider): void`
  - `approveCityFund(RequestModel $request, User $provider): void`
  - `rejectRequest(RequestModel $request, User $provider, string $reasonCode, ?string $reasonNote): void`
- **Refactor priority:** P1 — The redirect-inside-transaction is a latent bug. The three workflows will grow in complexity as business rules evolve.

---

### FC-04: Proof Upload — File/base64 processing + DB::transaction + status transition + notifications in one method

- **Severity:** High
- **File:** `app/Http/Controllers/Provider/ProviderProofController.php`
- **Method:** `store()` (lines 38–124, ~86 lines)
- **Category:** Fat Controller
- **Evidence:**

```php
// Lines 52-76: File/Base64 processing with validation
if ($request->hasFile('proof_file')) {
    $path = $file->store("private/proofs/{$redemption->id}");
} elseif (! empty($request->input('proof_photo_base64'))) {
    $base64Data = $request->input('proof_photo_base64');
    if (preg_match('/^data:image\/(\w+);base64,/', $base64Data, $type)) {
        $data = substr($base64Data, strpos($base64Data, ',') + 1);
        $type = strtolower($type[1]);
        if (! in_array($type, ['jpg', 'jpeg', 'png', 'webp'])) {
            return back()->with('error', __('Invalid image format captured.'));
        }
        $data = base64_decode($data);
        // ...store to disk...
    }
}

// Lines 78-115: Transaction with proof creation + request status change + notifications
DB::beginTransaction();
$redemption->refresh();
if ($redemption->proof) { DB::rollBack(); return redirect(...); }
OrderProof::create([...]);
$requestModel->status = 'FULFILLED';
$requestModel->save();
$this->auditService->log('request', 'fulfillment_proof_uploaded', [...]);
$this->notificationService->sendRequestStatusChanged($requestModel->load('recipient'), 'FULFILLED');
$this->auditService->log('notification', 'sent', [...]);
DB::commit();
```

- **Key concerns:** (1) Base64 decoding logic is duplicated with `RegisteredUserController::storeBase64Image()`. (2) Manual `DB::beginTransaction()` / `commit()` / `rollBack()` instead of the closure pattern. (3) Status transition (REDEEMED→FULFILLED) is a domain event that should be in a service.
- **Existing service bypassed:** No `ProofService` or `FulfillmentService` exists. The base64 handling is duplicated.
- **Recommended refactor:** Create `ProofUploadService` with method `uploadAndFulfill(OrderRedemption $redemption, ?UploadedFile $file, ?string $base64): OrderProof`. Extract shared base64 handling to `ImageStorageService`.
- **Suggested service name:** `ProofUploadService`
- **Suggested method signature:** `uploadAndFulfill(OrderRedemption $redemption, ?UploadedFile $file, ?string $base64Data): OrderProof`
- **Refactor priority:** P1 — Fulfillment is a critical business event. Having it in a controller makes it impossible to trigger fulfillment from a different entry point (e.g., API, batch job).

---

### FC-05: Admin Request Approval — Duplicate approve/reject workflow with status transitions and notifications

- **Severity:** Medium
- **File:** `app/Http/Controllers/Admin/AdminRequestController.php`
- **Method:** `update()` (lines 28–76, ~48 lines)
- **Category:** Fat Controller / Architecture
- **Evidence:**

```php
// Lines 44-73: Approve and reject branches with duplicate notification/audit patterns
if ($validated['action'] === 'approve') {
    $requestModel->update([
        'status' => 'ADMIN_APPROVED',
        'funding_source' => 'CITY_FUND',
    ]);
    $requestModel->load(['recipient', 'provider']);
    $this->notificationService->sendRequestStatusChanged($requestModel, 'ADMIN_APPROVED');
    $this->notificationService->sendRequestStatusChangedToProvider($requestModel, 'ADMIN_APPROVED');
    $this->auditService->log('notification', 'sent', [...]);
} elseif ($validated['action'] === 'reject') {
    $requestModel->update([
        'status' => 'ADMIN_REJECTED',
        'rejection_reason_code' => $validated['rejection_reason_code'],
        'rejection_reason_note' => $validated['rejection_reason_note'] ?? null,
    ]);
    $requestModel->load(['recipient', 'provider']);
    $this->notificationService->sendRequestStatusChanged($requestModel, 'ADMIN_REJECTED');
    $this->notificationService->sendRequestStatusChangedToProvider($requestModel, 'ADMIN_REJECTED');
    $this->auditService->log('notification', 'sent', [...]);
}
```

- **Key concern:** This is structurally identical to the provider-side approve/reject in FC-03. Both controllers independently implement request status transitions + notification dispatch. As the workflow grows (e.g., adding allocation steps), both controllers would need to change in sync.
- **Existing service bypassed:** No `AdminRequestApprovalService` exists. The notification+audit pattern is not abstracted.
- **Recommended refactor:** Create `AdminRequestApprovalService` (or extend a shared `RequestWorkflowService` with FC-03) to encapsulate admin approve/reject logic.
- **Suggested service name:** `AdminRequestApprovalService`
- **Suggested method signatures:**
  - `approve(RequestModel $request): void`
  - `reject(RequestModel $request, string $reasonCode, ?string $reasonNote): void`
- **Refactor priority:** P2 — Less complex than FC-03 but architecturally related. Consider creating a shared `RequestWorkflowService` that both admin and provider controllers delegate to.

---

### FC-06: Recipient Request Submission — Allowance checking, fund availability, retry queuing, cooldown management all in controller

- **Severity:** Medium
- **File:** `app/Http/Controllers/Recipient/RecipientRequestController.php`
- **Method:** `store()` (lines 33–114, ~81 lines)
- **Category:** Fat Controller / Maintainability
- **Evidence:**

```php
// Lines 41-47: Cooldown check
if (RecipientRequestSubmitCooldown::active($user->id)) {
    $remaining = RecipientRequestSubmitCooldown::secondsRemaining($user->id);
    return back()->with('error', ...)->withInput();
}

// Lines 53-76: Allowance check + retry queuing (24 lines)
if (RecipientAllowanceService::wouldExceedAllowance($user->id, $totalAmount)) {
    RecipientAllowanceRetryCache::storePayload($user->id, [...]);
    $delaySeconds = (int) config('recipient.allowance_retry_delay_seconds', 60);
    if (RecipientAllowanceRetryCache::tryScheduleJobLock($user->id, $delaySeconds + 10)) {
        ProcessRecipientAllowanceRetryJob::dispatch($user->id)
            ->delay(now()->addSeconds($delaySeconds));
    }
    $this->auditService->log('request', 'allowance_retry_queued', [...]);
    RecipientRequestSubmitCooldown::start($user->id, $delaySeconds);
    return back()->with('info', ...)->withInput();
}

// Lines 80-103: Fund availability check + retry queuing (same pattern repeated)
if (! $this->allocationService->canCoverRequestAmount($totalAmount)) {
    RecipientFundRetryCache::storePayload($user->id, [...]);
    $delaySeconds = (int) config('recipient.fund_retry_delay_seconds', 60);
    if (RecipientFundRetryCache::tryScheduleJobLock($user->id, $delaySeconds + 10)) {
        ProcessRecipientFundRetryJob::dispatch($user->id)
            ->delay(now()->addSeconds($delaySeconds));
    }
    $this->auditService->log('request', 'fund_retry_queued', [...]);
    RecipientRequestSubmitCooldown::start($user->id, $delaySeconds);
    return back()->with('info', ...)->withInput();
}
```

- **Key concern:** The retry-queuing pattern is duplicated almost identically for allowance and fund checks. The controller orchestrates 6 different support classes (AllocationService, RecipientAllowanceService, RecipientAllowanceRetryCache, RecipientFundRetryCache, RecipientRequestSubmitCooldown, RecipientRequestSubmissionService). This makes the submission flow untestable without HTTP integration tests.
- **Existing service partially used:** `RecipientRequestSubmissionService::createRequest()` handles the final creation, but all pre-validation, retry queuing, and cooldown logic lives in the controller.
- **Recommended refactor:** Extend `RecipientRequestSubmissionService` (or create `RecipientRequestOrchestrator`) to encapsulate the full submission pipeline: cooldown check → allowance check (with retry) → fund check (with retry) → create. Return a result DTO that tells the controller which redirect/flash to use.
- **Suggested service name:** `RecipientRequestOrchestrator`
- **Suggested method signature:** `submit(User $recipient, int $providerId, array $items): SubmissionResult` (where `SubmissionResult` is a DTO with `status`, `message`, `redirectRoute`, etc.)
- **Refactor priority:** P2 — The retry logic is the most complex business rule in this controller and is impossible to unit test in its current location.

---

### FC-07: Account Approval — Approve/reject with inline notification dispatch and conditional logic

- **Severity:** Medium
- **File:** `app/Http/Controllers/Admin/AccountApprovalController.php`
- **Methods:** `approve()` (lines 34–56, ~22 lines), `reject()` (lines 67–96, ~29 lines)
- **Category:** Fat Controller
- **Evidence:**

```php
// approve() — lines 40-53
$user->update(['status' => User::STATUS_ACTIVE, 'rejection_reason' => null]);
$this->auditService->log('account_approval', 'approved', [...]);
if ($user->membership_type === User::MEMBERSHIP_RECIPIENT) {
    $this->notificationService->sendAccountStatusUpdated($user, true);
    $this->auditService->log('notification', 'sent', [...]);
}

// reject() — lines 77-93
$user->update(['status' => User::STATUS_REJECTED, 'rejection_reason' => $validated['rejection_reason']]);
$this->auditService->log('account_approval', 'rejected', [...]);
if ($user->membership_type === User::MEMBERSHIP_RECIPIENT) {
    $this->notificationService->sendAccountStatusUpdated($user, false, $validated['rejection_reason']);
    $this->auditService->log('notification', 'sent', [...]);
}
```

- **Key concern:** Each method is individually short, but the approve/reject pattern (status change + conditional notification by membership type + audit) is a domain workflow. As the approval process grows (e.g., adding wallet provisioning for recipients, welcome emails, onboarding tasks), this controller will bloat.
- **Existing service bypassed:** No `AccountApprovalService` exists. The conditional notification-by-type logic is a business rule, not HTTP routing.
- **Recommended refactor:** Create `AccountApprovalService` with `approve(User $user): void` and `reject(User $user, string $reason): void`. The service encapsulates the state transition, conditional notifications, and audit trail.
- **Suggested service name:** `AccountApprovalService`
- **Suggested method signatures:**
  - `approve(User $user): void`
  - `reject(User $user, string $reason): void`
- **Refactor priority:** P3 — Currently manageable but will become problematic as approval workflow complexity grows.

---

## Controllers Reviewed and Found Acceptable

The following controllers were reviewed and found to be properly thin or acceptably structured. They are not flagged:

| Controller | Reason |
|-----------|--------|
| `Admin\AdminAllocationController` | Excellent delegation to `AllocationEngineServiceInterface` |
| `Admin\AllowanceSettingsController` | Clean delegation to `RecipientAllowanceService` |
| `Admin\RoleController` | Clean delegation to `RoleManagementService` |
| `Admin\SummaryReportController` | Clean delegation to `SummaryReportExportService` |
| `Admin\UserManagementController` | Excellent delegation to `UserService` |
| `Admin\QrSettingsController` | Minimal, config-driven |
| `Admin\AdminFundTransactionController` | Minor inline query logic (acceptable) |
| `Admin\AdminPaymentController` | Minor inline query logic (acceptable) |
| `Admin\FinancialOverviewController` | Chart config logic is presentation-layer (borderline but acceptable) |
| `Admin\FinancialReportController` | Date range resolution is minor (acceptable) |
| `Auth\AuthenticatedSessionController` | Clean auth flow |
| `Auth\OtpLoginController` | Good delegation to `OtpService` |
| `Auth\PhoneVerificationController` | Good delegation to `OtpService` |
| `Auth\ResubmitApplicationController` | Excellent delegation to `ResubmitApplicationService` |
| `Auth\NewPasswordController` | Uses Laravel `Password::reset()` correctly |
| `Auth\ConfirmablePasswordController` | Minimal |
| `Auth\VerifyEmailController` | Minimal |
| `Donor\DonationController` | Clean delegation to `PaymentService` |
| `Donor\DonorDashboardController` | Aggregation queries (presentation-layer, acceptable) |
| `Provider\ProviderDashboardController` | Aggregation queries (presentation-layer, acceptable) |
| `Provider\MenuItemController` | Simple CRUD |
| `Provider\ProviderProfileController` | Clean with simple transaction |
| `Provider\ProviderQrController` | Excellent delegation to `RedemptionService` |
| `Provider\ProviderWalletController` | Simple data retrieval |
| `Recipient\RecipientController` | Dashboard aggregation (acceptable) |
| `Recipient\ProviderMenuController` | Simple filtering |
| `PaymentCallbackController` | Model thin controller — excellent pattern to follow |
| `ProfileController` | File handling in `syncProfilePhoto` is borderline but acceptable |
| `NotificationController` | Simple CRUD |
| `LanguageController` | Minimal |

---

## Recommended Refactoring Roadmap

### Phase 1 — Critical (P1)

| # | Finding | New Service | Estimated Effort |
|---|---------|-------------|-----------------|
| 1 | FC-01 | `ProviderRegistrationService` | 2–3 hours |
| 2 | FC-02 | `RecipientRegistrationService` + shared `ImageStorageService` | 2–3 hours |
| 3 | FC-03 | `ProviderRequestActionService` | 2–3 hours |
| 4 | FC-04 | `ProofUploadService` | 1–2 hours |

### Phase 2 — Important (P2)

| # | Finding | New Service | Estimated Effort |
|---|---------|-------------|-----------------|
| 5 | FC-05 | `AdminRequestApprovalService` | 1–2 hours |
| 6 | FC-06 | `RecipientRequestOrchestrator` (extend `RecipientRequestSubmissionService`) | 2–3 hours |

### Phase 3 — Improvement (P3)

| # | Finding | New Service | Estimated Effort |
|---|---------|-------------|-----------------|
| 7 | FC-07 | `AccountApprovalService` | 1 hour |

### Cross-cutting extractions

- **`ImageStorageService`** — Consolidate base64 image handling from `RegisteredUserController::storeBase64Image()`, `ProviderProofController::store()`, and `ResubmitApplicationService`. Single place for file type validation, decoding, storage, and cleanup.
- **Shared audit+notification pattern** — Consider a `RequestStatusTransitionService` that any controller/service calls when changing request status. This would centralize the "update status → audit → notify recipient → notify provider → audit notification" pattern that appears in FC-03, FC-04, FC-05, and FC-07.

---

## Architectural Notes

### Good patterns already in the codebase (use as templates):

1. **`PaymentCallbackController`** — 1-line methods delegating entirely to `PaymentService`. This is the gold standard.
2. **`ResubmitApplicationController`** — All complex logic in `ResubmitApplicationService`. Controller handles only routing and status checks.
3. **`AdminAllocationController`** — Pure delegation to `AllocationEngineServiceInterface` with interface-based injection.
4. **`UserManagementController`** — Clean CRUD with `UserService` handling all business logic and proper form request validation.

### Anti-patterns to avoid:

1. **Returning HTTP responses from inside `DB::transaction` closures** (FC-03, line 181) — The closure should return data; the controller wrapping it should produce the HTTP response.
2. **Static service calls mixed with dependency injection** (FC-03, `\App\Services\RedemptionService::generateForRequest()`) — Use injected instances for consistency and testability.
3. **Duplicate file cleanup in catch blocks** (FC-01, FC-02) — A service with proper file lifecycle management eliminates this.
