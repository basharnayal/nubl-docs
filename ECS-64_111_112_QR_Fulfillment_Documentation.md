# Engineering Documentation: Request Fulfillment QR Lifecycle
**Milestone 3 (ECS-64, ECS-111, ECS-112)**
**Author:** Salem (Senior Software Engineer)
**Date:** March 2026

## Executive Summary
This document provides a comprehensive architectural and technical overview of the complete end-to-end QR code redemption lifecycle implemented within the NUBL platform. The deployment encompasses three major Epics: 
1. **ECS-64 (QR Generation)**
2. **ECS-111 (Provider QR Redemption & Scanning)**
3. **ECS-112 (Proof of Fulfillment)**

This feature transitions approved donor requests to actual on-the-ground fulfillment by generating secure, cryptographic QR codes for Recipients, scanning and validating those codes by certified Providers, transferring reserved donor funds atomically, and enforcing photo/file proof before finalizing the system state.

---

## 1. Architectural Design & Data Modeling
To support robust and auditable physical fulfillment, the database schema was expanded with strict relational integrity.

### 1.1 New Database Tables
- **`order_redemptions`**: Tracks the lifecycle of the generated token.
  - Columns: `id`, `request_id` (foreign key), `provider_id` (foreign key), `token_code` (hash), `short_code_hash` (hash), `token_ciphertext` (encrypted base64), `short_code_ciphertext` (encrypted base64), `redeem_expires_at`, `status` (`PENDING`, `REDEEMED`, `EXPIRED`).
- **`order_proofs`**: Tracks the final verification evidence.
  - Columns: `id`, `order_redemption_id` (foreign key constraint), `proof_url` (physical path), `is_provider_donation` (boolean), `fulfilled_at` (timestamp).

### 1.2 Dual-Token Cryptographic Strategy (Security vs. UX)
A major design decision was implementing a **dual-token system** to balance high security with fallback usability.
- **Core Token (32-byte cryptographic random string):** Encoded raw into the QR code payload. Virtually impossible to brute-force.
- **Short Alias (9-character alphanumeric):** A user-friendly, visually distinct code (e.g., `4R9-X2M-Q1P`) displayed beneath the QR on the Recipient's screen.

**Storage Security:** Both the core token and the 9-character alias are **hashed** using standard fast hashing (SHA-256 equivalent via `hash()`) before being stored in the database (`token_code` and `short_code_hash`). The raw strings are never stored in plaintext. To display the aliased 9-character code to the user, the raw string is symmetrically encrypted (`encrypt()`) and stored in `short_code_ciphertext`, which is safely decrypted at runtime solely for presentation in the Recipient View.

---

## 2. ECS-64: QR Token Generation & Assignment
The token generation sequence is tightly integrated into the Request approval lifecycle.

### 2.1 Backend Implementation: `RedemptionService`
The `RedemptionService::generateForRequest(Request $request)` method serves as the singular entry point for token creation. It is called immediately when a Request transitions to the `REDEEMABLE` state (after donor funds have been secured and successfully designated).
- The service enforces idempotency: It checks if an active `order_redemptions` row already exists before generating to prevent accidental overwrites.
- It dynamically generates the token, hashes it, encrypts the UI-facing payloads, and seeds the `order_redemptions` table with a standard 30-day expiration window (`redeem_expires_at = now()->addDays(30)`).

### 2.2 Recipient UI Integration
The Recipient Dashboard (`requests.show`) was updated to conditionally render the fulfillment UI.
- If `status == 'REDEEMABLE'`, the page decrypts `token_ciphertext` to generate the raw QR code using `simplesoftwareio/simple-qrcode`.
- It dynamically decrypts `short_code_ciphertext` to render the 9-character "Manual Code" directly beneath the QR image, allowing the recipient to verbally read the code to the provider if the camera scanner fails.

---

## 3. ECS-111: Provider QR Redemption & Scanning
This epic handles the physical interaction between the Provider and the Recipient at the storefront.

### 3.1 Frontend Implementation: HTML5 Barcode Integration
The Provider UI (`scan.blade.php`) utilizes the `html5-qrcode` JavaScript library to interface natively with mobile and desktop cameras.
- **Camera Configuration:** The scanner is explicitly forced to use `facingMode: "environment"` to prioritize the rear camera on mobile devices.
- **Flood Prevention (Debouncing):** A complex state machine (`isRedeeming`, `lastToken`) was engineered to manage the continuous video feed. Upon successfully decoding a QR code, the camera feed is immediately paused (`html5QrcodeScanner.pause()`) to prevent rapid, identical POST requests that would trigger HTTP 429 (Too Many Requests) errors and database locking timeouts.
- **UX Resilience:** If a backend error occurs (e.g., Insufficient funds or Invalid token), the camera automatically resumes scanning after a 2000ms delay, explicitly retaining memory of the failed token so it will comfortably ignore the identical physical QR code still resting in frame until the device is moved to scan a new code.

### 3.2 Backend Implementation: `ProviderQrController`
The redemption endpoint is engineered for absolute atomic safety, given it handles direct financial disbursements.

#### 3.2.1 Route & Middleware
POST `provider/qr/redeem` is protected by `auth`, `EnsureRole:provider`, and `EnsureAccountApproved` to ensure only fully vetted business entities can access the lookup.

#### 3.2.2 Lookup Flexibility
The logic natively accepts *either* the payload from the QR code (32-bytes) *or* the 9-character manual entry from the fallback UI. The controller hashes the incoming payload and searches both `token_code` and `short_code_hash` simultaneously:
```php
$redemption = OrderRedemption::where(function ($query) use ($tokenHash) {
    $query->where('token_code', $tokenHash)
          ->orWhere('short_code_hash', $tokenHash);
})->lockForUpdate()->first();
```

#### 3.2.3 Atomic Transaction Pipeline
The entire execution is wrapped inside a `DB::transaction()` with pessimistic row locking (`lockForUpdate()`) to mathematically eliminate double-spend race conditions if two parallel requests hit the server milliseconds apart.
1. **Validation Checks:** Validates Provider ID matching, Status (`PENDING`), and Expiration timestamp. Returns strict mapped HTTP codes (`404 Not Found`, `403 Forbidden`, `409 Conflict`, `422 Unprocessable Entity`) preventing swallowed `500 Server Errors`.
2. **State Mutation:** Updates `OrderRedemption` to `REDEEMED`.
3. **Financial Transfer:** Calls `SystemWalletService::transferToProviderForRequest()` to locate the securely held donor `Payment` row via `AllocationService`, physically deduct the real value from the `SYSTEM` wallet, credit the Provider's `Ewallet` balance, and write the immutable dual `FundTransaction` ledger entries.
4. **Audit Logging:** Triggers `Spatie\Activitylog` for compliance and non-repudiation tracking.

---

## 4. ECS-112: Proof of Fulfillment & Finalization
The final milestone enforces evidence collection before formally concluding the Request. 

### 4.1 Hybrid Camera/Upload UI (`proof.blade.php`)
The frontend Proof Upload view was heavily upgraded from a basic file input to a **Dual-Mode AlpineJS Component**, directly migrating the proven architecture from the Recipient KYC Registration flow.
- **Option 1 (File Upload):** Native HTML `<input type="file">` for pre-existing gallery photos or PDFs.
- **Option 2 (Live Capture):** Direct API hooks into `navigator.mediaDevices.getUserMedia()` to spawn an inline `<video>` viewport, allowing the Provider to natively snap a real-time photograph of the receipt/hand-over within the web application, immediately drawing the frame to an HTML5 `<canvas>` and converting it to a raw Base64 Data URI string.
- **Frontend Validation:** AlpineJS intercepts form submission (`validateBeforeSubmit()`), guaranteeing that either a valid HTML5 file array exists *or* a Base64 string payload is present, physically blocking empty POST requests.

### 4.2 Backend Processing (`ProviderProofController`)
The storage controller safely proxies both payload types.
- **Validation Rules:** Verifies maximum sizes (5MB), allowed MIME constraints (`mimes:jpg,jpeg,png,webp,pdf`), and nullable Base64 strings.
- **Dynamic Binary Parsing:** If a Base64 payload is detected, the controller dynamically parses the Data URI header (`preg_match('/^data:image\/(\w+);base64,/')`), sanitizes the string, natively hashes it through `base64_decode()`, writes the generated binary to the protected `/private/proofs/{id}/` Laravel Storage disk, and assigns the generated file path universally.
- **Final DB Commit:** Inserts the final `OrderProof` row linking the generated file path, permanently mutates the core `Request` model lifecycle to `FULFILLED`, logs the action universally via `AuditService`, and finally frees the Provider to accept new orders.
