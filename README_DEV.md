# NUBL - Development Documentation

> **⚠️ IMPORTANT: This is the CENTRAL documentation file for all development-related documentation.**
> 
> **All developers and AI assistants MUST add any new documentation to this file, NOT create separate `.md` files.**
> 
> **Follow the same conventions and structure. Keep everything organized and in English.**

---

## Table of Contents

- [Git Commit Conventions](#git-commit-conventions)
- [Email Configuration](#email-configuration)
- [Email Verification](#email-verification)
- [Roles & Permissions](#roles--permissions)
  - [Setup](#setup)
  - [Available Roles](#available-roles)
  - [Using Roles in Routes](#using-roles-in-routes)
  - [Using Roles in Controllers](#using-roles-in-controllers)
  - [Using Roles in Views](#using-roles-in-views)
  - [Managing Roles Programmatically](#managing-roles-programmatically)
  - [Permissions (Future Reference)](#permissions-future-reference)
- [Lineone Components](#lineone-components)
  - [Available Components](#available-components)
  - [Lineone Modal](#lineone-modal)
  - [Lineone Alert](#lineone-alert)
  - [Lineone Card](#lineone-card)
  - [Lineone Button](#lineone-button)
  - [Usage Examples](#usage-examples)
- [Additional Documentation](#additional-documentation)

---

## Git Commit Conventions

Simple commit message rules for clean Git history and easy code reviews.

### Format

```
<type>(<scope>): <summary> [FR-#]
```

**Example:**
```
feat(auth): add RBAC middleware [FR-1.5]
```

**Rules:**
- Use present tense (add, fix, NOT added/fixed)
- Keep summary ≤ 72 characters
- One commit = one change
- Add `[FR-#]` or `[NFR-#]` if related to requirements

### Types (Remember: F F R D T C)

| Type | Use When |
|------|----------|
| **feat** | New feature |
| **fix** | Bug fix |
| **refactor** | Code restructure (no behavior change) |
| **docs** | Documentation only |
| **test** | Tests only |
| **chore** | Setup/config/dependencies |

### Examples

```
feat(donations): create donation form [FR-2.1]
fix(auth): prevent unauthorized access [FR-1.5]
refactor(services): extract donation logic
docs(readme): add Spatie setup guide
test(auth): add role tests [FR-1.5]
chore(deps): install spatie/laravel-permission
```

### Common Scopes

- `auth`, `roles`, `users`
- `donations`, `requests`, `qr`
- `routes`, `views`, `ui`
- `db`, `migrations`, `seeders`
- `config`, `core`

### Optional Body

For complex changes, add details:

```
feat(auth): implement RBAC [FR-1.5]

- Added Spatie roles
- Protected routes with middleware
- Added role-based redirects
```

### Rules

**DO:**
- ✅ Write clear messages
- ✅ Keep commits small
- ✅ Reference FR/NFR when relevant
- ✅ One logical change per commit

**DON'T:**
- ❌ Vague messages ("update stuff")
- ❌ Mix unrelated changes
- ❌ Skip requirement mapping

### Quick Reference

**Format:** `type(scope): summary [FR-#]`  
**Types:** feat, fix, refactor, docs, test, chore  
**Always:** Present tense, clear, one change

**That's it! Keep it simple and consistent.** 🚀

---

## Email Configuration

### Mailtrap Setup (Development)

Mailtrap is the recommended email service for local development because:
- ✅ Free for development/testing
- ✅ Beautiful web interface to view emails
- ✅ No local installation required
- ✅ Easy to set up

#### Setup Steps

1. **Create Mailtrap Account:**
   - Go to: https://mailtrap.io
   - Sign up for a free account
   - After logging in, navigate to: **Email Testing** → **Sandboxes** → **My Sandbox**

2. **Get SMTP Credentials:**
   - In your Sandbox, click on **SMTP** tab
   - You'll see:
     - Host: `sandbox.smtp.mailtrap.io`
     - Port: `2525` (recommended for Laravel)
     - Username: (your unique username)
     - Password: (your unique password - click eye icon to reveal)

3. **Update `.env` File:**
   ```env
   MAIL_MAILER=smtp
   MAIL_HOST=sandbox.smtp.mailtrap.io
   MAIL_PORT=2525
   MAIL_USERNAME=your-mailtrap-username
   MAIL_PASSWORD=your-mailtrap-password
   MAIL_ENCRYPTION=tls
   MAIL_FROM_ADDRESS="noreply@nubl.com"
   MAIL_FROM_NAME="${APP_NAME}"
   ```

4. **Clear Config Cache:**
   ```bash
   php artisan config:clear
   ```

5. **View Emails:**
   - After sending any email from the application, go back to Mailtrap Dashboard
   - Navigate to: **Email Testing** → **Sandboxes** → **My Sandbox**
   - All sent emails will appear there
   - You can open and read them directly

#### Testing Email Setup

After updating `.env`, test the email configuration:

```bash
php artisan tinker
```

Then in Tinker:
```php
Mail::raw('Test email', function ($message) {
    $message->to('test@example.com')
            ->subject('Test Email');
});
```

Check your Mailtrap inbox to see the test email.

---

## Email Verification

Email verification can be easily enabled or disabled via the `.env` file. This allows you to toggle email verification requirements without modifying code.

### Configuration

Add the following variable to your `.env` file:

```env
# Email Verification
# Set to true to require email verification, false to disable
EMAIL_VERIFICATION_ENABLED=true
```

### How It Works

- **When `EMAIL_VERIFICATION_ENABLED=true`** (default):
  - Users must verify their email before accessing protected routes
  - After registration/login, unverified users are redirected to the verification notice page
  - All dashboard and profile routes require email verification

- **When `EMAIL_VERIFICATION_ENABLED=false`**:
  - Email verification is completely bypassed
  - Users can access all routes immediately after registration/login
  - No verification emails are sent
  - All users are considered verified

### Usage

#### Enable Email Verification

```env
EMAIL_VERIFICATION_ENABLED=true
```

#### Disable Email Verification

```env
EMAIL_VERIFICATION_ENABLED=false
```

After changing the value, clear the config cache:

```bash
php artisan config:clear
```

### Technical Details

- **Middleware**: Custom `EnsureEmailVerified` middleware checks the config before enforcing verification
- **Routes**: All protected routes automatically adapt based on the config value
- **Controllers**: Registration and login controllers check the config before redirecting to verification
- **User Model**: Includes helper methods `emailVerificationRequired()` and `isEmailVerified()`

### Helper Methods

In your code, you can check if email verification is enabled:

```php
// Check if email verification is required
if (User::emailVerificationRequired()) {
    // Email verification is enabled
}

// Check if user's email is verified (or if verification is disabled)
if ($user->isEmailVerified()) {
    // User is verified or verification is disabled
}

// Or use config directly
if (config('app.email_verification_enabled', true)) {
    // Email verification is enabled
}
```

### Important Notes

- Changing this setting requires clearing config cache: `php artisan config:clear`
- When disabled, users can still have `email_verified_at` set, but it won't be checked
- The `MustVerifyEmail` interface remains in the User model, but verification checks are bypassed when disabled
- This setting affects all routes that use the `email.verified` middleware

---

## Roles & Permissions

This project uses **Spatie Laravel Permission** for role-based access control (RBAC).

### Setup

#### 1. Run Seeders

```bash
# Run all seeders (creates permissions, roles, and assigns permissions to roles)
php artisan db:seed

# Or run individually
php artisan db:seed --class=PermissionSeeder
php artisan db:seed --class=RoleSeeder
```

#### 2. Clear Cache

```bash
php artisan optimize:clear
```

### Available Roles

The project has 4 main roles:

- **`admin`** - System administrator (has access to everything)
- **`donor`** - Donor (Gracious Neighbor) - can make donations
- **`recipient`** - Recipient (Neighbor) - can request items
- **`provider`** - Provider (Local supermarkets/restaurants) - can fulfill requests

### Using Roles in Routes

Protect routes based on user roles:

```php
// Single role
Route::middleware(['auth', 'role:admin'])->prefix('admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
});

// Multiple roles (user needs ANY of these roles)
Route::middleware(['auth', 'role:donor|recipient'])->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});

// Role-specific routes
Route::middleware(['auth', 'role:donor'])->prefix('donor')->group(function () {
    Route::get('/dashboard', [DonorController::class, 'dashboard']);
});

Route::middleware(['auth', 'role:recipient'])->prefix('recipient')->group(function () {
    Route::get('/dashboard', [RecipientController::class, 'dashboard']);
});

Route::middleware(['auth', 'role:provider'])->prefix('provider')->group(function () {
    Route::get('/dashboard', [ProviderController::class, 'dashboard']);
});
```

### Using Roles in Controllers

Check user roles in controller methods:

```php
use Illuminate\Http\Request;

class DashboardController extends Controller
{
    public function index()
    {
        $user = auth()->user();
        
        // Method 1: Check if user has role
        if ($user->hasRole('admin')) {
            return redirect()->route('admin.dashboard');
        }
        
        // Method 2: Check multiple roles
        if ($user->hasAnyRole(['donor', 'recipient'])) {
            // User is either donor or recipient
        }
        
        // Method 3: Abort if user doesn't have role
        if (!$user->hasRole('admin')) {
            abort(403, 'Unauthorized - Admin access required');
        }
        
        // Your code here
    }
}
```

### Using Roles in Views

Show/hide content based on user roles:

```blade
{{-- Method 1: @role directive (recommended) --}}
@role('admin')
    <a href="{{ route('admin.dashboard') }}">Admin Dashboard</a>
@endrole

@role('donor')
    <a href="{{ route('donor.dashboard') }}">Donor Dashboard</a>
@endrole

{{-- Method 2: @hasrole directive --}}
@hasrole('admin')
    <div>Admin content</div>
@else
    <div>Regular user content</div>
@endhasrole

{{-- Method 3: Check multiple roles --}}
@hasanyrole('admin|donor')
    <div>Admin or Donor content</div>
@endhasanyrole

{{-- Method 4: Using hasRole() method --}}
@if(auth()->user()->hasRole('admin'))
    <button>Admin Button</button>
@endif

{{-- Method 5: Check if user has any of multiple roles --}}
@if(auth()->user()->hasAnyRole(['donor', 'recipient']))
    <p>You are a donor or recipient</p>
@endif
```

### Managing Roles Programmatically

#### Assign Role to User

```php
$user = User::find(1);
$user->assignRole('donor');

// Assign multiple roles
$user->assignRole(['donor', 'recipient']);
```

#### Remove Role from User

```php
$user->removeRole('donor');

// Remove multiple roles
$user->removeRole(['donor', 'recipient']);
```

#### Sync Roles (Replace all roles)

```php
// This will remove all existing roles and assign only 'admin'
$user->syncRoles(['admin']);
```

#### Check User Roles

```php
$user = User::find(1);

// Check if user has role
$user->hasRole('admin'); // true/false

// Check if user has any of the roles
$user->hasAnyRole(['admin', 'donor']); // true/false

// Check if user has all roles
$user->hasAllRoles(['admin', 'donor']); // true/false

// Get all user roles
$user->roles; // Collection of Role models
$user->roles->pluck('name'); // ['admin', 'donor']
```

### Complete Example: Role-Based Dashboard Redirect

#### In Middleware (RedirectByRole)

```php
public function handle(Request $request, Closure $next)
{
    if (!auth()->check()) {
        return $next($request);
    }
    
    $user = auth()->user();
    
    // Redirect based on role
    if ($user->hasRole('admin')) {
        return redirect()->route('admin.dashboard');
    }
    
    if ($user->hasRole('donor')) {
        return redirect()->route('donor.dashboard');
    }
    
    if ($user->hasRole('recipient')) {
        return redirect()->route('recipient.dashboard');
    }
    
    if ($user->hasRole('provider')) {
        return redirect()->route('provider.dashboard');
    }
    
    return $next($request);
}
```

#### In View (Navigation)

```blade
<nav>
    @role('admin')
        <a href="{{ route('admin.dashboard') }}">Admin</a>
    @endrole
    
    @role('donor')
        <a href="{{ route('donor.dashboard') }}">Donor</a>
    @endrole
    
    @role('recipient')
        <a href="{{ route('recipient.dashboard') }}">Recipient</a>
    @endrole
    
    @role('provider')
        <a href="{{ route('provider.dashboard') }}">Provider</a>
    @endrole
</nav>
```

### Quick Commands

```bash
# Open Tinker
php artisan tinker

# Assign role to user
$user = User::find(1);
$user->assignRole('admin');

# Check user role
$user->hasRole('admin'); // true/false

# Get all roles
$user->roles->pluck('name');

# Remove role
$user->removeRole('admin');
```

### Important Notes

- **Admin role** automatically has access to everything
- Roles are cached for performance
- Always clear cache after role changes: `php artisan optimize:clear`
- Users can have multiple roles
- Use `syncRoles()` to replace all roles at once

---

## Permissions

> **Full Arabic guide (architecture, lists, seeders, UI, audit):** [`docs/PERMISSIONS_AND_RBAC_AR.md`](PERMISSIONS_AND_RBAC_AR.md)

Permissions are **fully defined in code** and **seeded into the database**. The canonical list lives in **`app/Permissions/PermissionDefinitions.php`** (`admin()`, `donor()`, `recipient()`, `provider()`, `all()`). **`database/seeders/PermissionSeeder.php`** creates every permission row; **`database/seeders/RoleSeeder.php`** assigns them to roles (admin gets all permissions; other roles get the subsets from `PermissionDefinitions`).

### Permission hierarchy

```
User
  └── Role (admin, donor, recipient, provider, …)
       └── Permissions (string names, e.g. users.manage, qr.configure_ttl)
```

### Admin UI: roles & permissions

- **URL:** `/admin/roles` (also under **Users → Roles & permissions** in the sidebar).
- **Controller:** `App\Http\Controllers\Admin\RoleController`
- **Service:** `App\Http\Services\RoleManagementService` (create/update/delete roles, sync permissions, clear Spatie cache, **audit logging**).
- **Core roles** (`admin`, `donor`, `recipient`, `provider`) **cannot be deleted**; their **names** stay fixed; **permissions** can be edited. Custom roles can be created and deleted (if no users still use them).
- **Audit:** `AuditService` records `role.created`, `role.updated`, `role.deleted` with context (IDs, permission lists). See **`docs/AUDIT_LOG_GUIDE_AR.md`**.
- **UI labels:** `lang/en.json` and `lang/ar.json` use keys `rbac.permission.{permission_name}` for human-readable names in the checkbox grid.

### How enforcement works today

- **Routes:** Most areas use **`role:admin`** (or `role:donor`, etc.) via `EnsureRole` — see `routes/web.php` and `bootstrap/app.php` middleware aliases.
- **`permission` middleware** is registered (`EnsurePermission`) but **not** used on route groups by default; you can add `permission:some.permission` when you want route-level checks.
- **Fine-grained checks:** A few admin controllers call **`abort_unless($request->user()->can('…'), 403)`** in addition to the admin role:
  - `qr.configure_ttl` — `QrSettingsController` (QR TTL settings)
  - `allowances.configure` — `AllowanceSettingsController`
  - `reports.export_csv` — `SummaryReportController` (summary reports download)
- If you add a new `can('x.y')` check, add **`x.y`** to `PermissionDefinitions`, re-seed or create the permission in DB, and add **`rbac.permission.x.y`** in both JSON locales.

### Available permissions (canonical)

**Admin (`PermissionDefinitions::admin()`):**

- `accounts.approve`, `requests.review`, `requests.approve`, `requests.reject`
- `qr.configure_ttl`
- `users.create`, `users.read`, `users.update`, `users.delete`, `users.manage`, `users.assign.roles`, `users.deactivate`, `users.reactivate`
- `funds.create`, `funds.read`, `funds.update`, `funds.delete`
- `policies.create`, `policies.read`, `policies.update`, `policies.delete`
- `reports.export_csv`, `reports.export_pdf`
- `allowances.configure`
- `allocation.pause_global`, `allocation.pause_per_provider`
- `roles.manage`, `permissions.manage`

**Donor:**

- `donations.process`, `dashboard.donor.view_stats`

**Recipient:**

- `requests.submit`

**Provider:**

- `qr.redeem`, `fulfillment_proof.upload`, `requests.adopt`
- `provider.capacity.toggle`, `provider.pickup_notes_and_hours.update`

### Using permissions in code

```php
// Routes (optional — middleware alias registered in bootstrap/app.php)
Route::middleware(['auth', 'account.approved', 'permission:roles.manage'])->group(function () {
    // ...
});

// Controllers
if (! auth()->user()->can('roles.manage')) {
    abort(403);
}

// Blade
@can('roles.manage')
    <a href="{{ route('admin.roles.index') }}">Roles</a>
@endcan
```

---

## Lineone Components

Essential Lineone Blade components (Alpine.js + Tailwind) for consistent UI across the application.

### Available Components

- `lineone-modal` - Modal dialogs (Alpine.js)
- `lineone-alert` - Alert notifications
- `lineone-card` - Card components
- `lineone-button` - Buttons with Lineone styles

### Lineone Modal

```blade
{{-- Basic Modal (Alpine.js events) --}}
<x-lineone-modal id="example-modal" title="Modal Title">
    <p>Modal content goes here</p>
</x-lineone-modal>

{{-- Trigger: dispatch open-modal --}}
<button @click="$dispatch('open-modal', 'example-modal')">Open Modal</button>

{{-- Close: $dispatch('close-modal', 'example-modal') or Escape key --}}
```

**Props:**
- `id` (required) - Unique modal ID
- `title` - Modal title
- `size` - sm, md, lg, xl, 2xl, 3xl

### Lineone Alert

```blade
{{-- Basic Alert --}}
<x-lineone-alert type="info">
    This is an info alert
</x-lineone-alert>

{{-- Dismissible Alert --}}
<x-lineone-alert type="success" dismissible>
    Operation completed successfully!
</x-lineone-alert>

{{-- All Types --}}
<x-lineone-alert type="info">Info message</x-lineone-alert>
<x-lineone-alert type="success">Success message</x-lineone-alert>
<x-lineone-alert type="warning">Warning message</x-lineone-alert>
<x-lineone-alert type="danger">Error message</x-lineone-alert>
```

**Props:**
- `type` - info, success, warning, danger
- `dismissible` - Enable dismiss button (default: false)

### Lineone Card

```blade
{{-- Basic Card --}}
<x-lineone-card title="Card Title" subtitle="Card subtitle">
    <p>Card content</p>
</x-lineone-card>

{{-- Card with Footer --}}
<x-lineone-card title="Card Title" :footer="'<a href=\"#\">Learn More</a>'">
    <p>Card content</p>
</x-lineone-card>
```

**Props:**
- `title` - Card title
- `subtitle` - Card subtitle
- `footer` - Footer content (Blade slot or HTML)

### Lineone Button

```blade
{{-- Primary Button --}}
<x-lineone-button variant="primary">Click Me</x-lineone-button>

{{-- Different Variants --}}
<x-lineone-button variant="primary">Primary</x-lineone-button>
<x-lineone-button variant="secondary">Secondary</x-lineone-button>
<x-lineone-button variant="success">Success</x-lineone-button>
<x-lineone-button variant="danger">Danger</x-lineone-button>
<x-lineone-button variant="warning">Warning</x-lineone-button>
<x-lineone-button variant="info">Info</x-lineone-button>
<x-lineone-button variant="slate">Slate</x-lineone-button>

{{-- Outline Buttons --}}
<x-lineone-button variant="primary" outline>Outline Primary</x-lineone-button>

{{-- Link as Button --}}
<x-lineone-button href="/donate" variant="primary">Donate Now</x-lineone-button>

{{-- Sizes --}}
<x-lineone-button size="xs">Extra Small</x-lineone-button>
<x-lineone-button size="sm">Small</x-lineone-button>
<x-lineone-button size="md">Medium</x-lineone-button>
<x-lineone-button size="lg">Large</x-lineone-button>
```

**Props:**
- `type` - button, submit, reset
- `variant` - primary, secondary, success, danger, warning, info, slate
- `size` - xs, sm, md, lg
- `outline` - Outline style (default: false)
- `href` - Render as link instead of button

### Usage Examples

```blade
{{-- Modal with Alert --}}
<x-lineone-modal id="success-modal" title="Success">
    <x-lineone-alert type="success" dismissible>
        Your changes have been saved!
    </x-lineone-alert>
</x-lineone-modal>

{{-- Card with Button --}}
<x-lineone-card title="Donation" subtitle="Make a donation">
    <p>Help support our cause</p>
    <div class="mt-4">
        <x-lineone-button href="/donate" variant="primary">
            Donate Now
        </x-lineone-button>
    </div>
</x-lineone-card>
```

---

## Additional Documentation

> **📝 Add new documentation sections below this line.**
> 
> **Format: Use clear headings, code blocks, and organized structure.**
> **Language: English only.**
> **Conventions: Follow the same style as existing sections.**

---

## Provider bank payout (weekly settlement)

Internal provider earnings are credited at **QR redemption** (`SystemWalletService::transferToProviderForRequest`) as ledger **IN** with source **`PAYOUT`**. Weekly **external bank** payouts are a separate admin-reviewed process: generation creates `provider_payouts` + `provider_payout_items` without wallet movement; **admin confirmation** creates one **`FundTransaction` OUT** with source **`PROVIDER_BANK_PAYOUT`**. See **`docs/PROVIDER_BANK_PAYOUT.md`**.

---

## Provider Menu Management + Recipient Browsing (ECS-62)

### Routes

**All routes are defined in `routes/web.php`.**

**Provider (Prefix: `/provider`, Middleware: `auth`, `account.approved`, `role:provider`)**
- `GET /provider/menu-items` - List menu items
- `GET /provider/menu-items/create` - Show create form
- `POST /provider/menu-items` - Store new item
- `GET /provider/menu-items/{item}/edit` - Show edit form
- `PUT /provider/menu-items/{item}` - Update item
- `DELETE /provider/menu-items/{item}` - Deactivate item (soft delete behavior)
*(Note: `/provider/application` is accessible without approval)*

**Recipient (Prefix: `/recipient`, Middleware: `auth`, `account.approved`, `role:recipient`)**
- `GET /recipient/providers` - Browse providers
- `GET /recipient/providers/{provider}` - View provider menu

### Main Controllers

- `App\Http\Controllers\Provider\MenuItemController`: Handles CRUD for provider's menu items. Ensures providers can only manage their own items.
- `App\Http\Controllers\Recipient\ProviderMenuController`: Handles listing providers and showing their menus to recipients.

### Views

**Provider:**
- `resources/views/provider/menu-items/index.blade.php`
- `resources/views/provider/menu-items/create.blade.php`
- `resources/views/provider/menu-items/edit.blade.php`

**Recipient:**
- `resources/views/recipient/providers/index.blade.php`
- `resources/views/recipient/providers/show.blade.php`

### Application Logic & Assumptions

- **Menu Items:** Linked to `User` (provider) via `provider_id`.
- **Provider Profile:** `ProviderProfile` is used to display business information. It is linked to `User` via `user_id`.
- **Validation:** `StoreMenuItemRequest` and `UpdateMenuItemRequest` enforce validation rules.
- **Deactivation:** Deleting a menu item sets `is_active` to `0` (false) instead of deleting the record, preserving history.

### Manual Verification Steps

1.  **Provider - Manage Menu:**
    - Login as a **Provider**.
    - Navigate to `/provider/menu-items`.
    - Click "Add New Item". Fill form (Name, Price, Category) and submit.
    - Verify item appears in the list.
    - Click "Edit". Change price. Submit. Verify update.
    - Click "Deactivate". Verify item status changes to Inactive.

2.  **Recipient - Browse & View:**
    - Login as a **Recipient**.
    - Navigate to `/recipient/providers`.
    - You should see a list of active providers.
    - Click "View Menu & Order" on a provider.
    - You should see the provider's details and their **active** menu items.
    - Verify that inactive items (deactivated by provider) are NOT shown.

3.  **Access Control:**
    - As a Recipient, try to access `/provider/menu-items`. Expect **403 Forbidden**.
    - As a Provider, try to edit another provider's item ID. Expect **404 Not Found**.

### Test Data Notes

- Ensure `ProviderProfile` exists for providers to be visible in the recipient list.
- Required fields for `ProviderProfile`: `business_name_en`, `business_category` (array), `city`, `location`.

---

## Request Lifecycle V2 (ECS-63)

### Overview
The Request Lifecycle V2 introduces a multi-item request system with a complete approval workflow involving Recipients, Providers, and Admins. It replaces the single-item request model.

### Key Features
- **Multi-Item Requests:** Recipients can add multiple items from a single provider to a "cart" and submit them as one request.
- **Weekly Allowance Logic:**
    - Limit: 400 SAR per week.
    - Usage is calculated based on requests that are `REQUESTED`, `APPROVED`, `ADMIN_PENDING`, `ADMIN_APPROVED`, `REDEEMABLE`, or `FULFILLED`.
    - `APPROVED` (PROVIDER_ADOPTION), `REJECTED`, and `CANCELLED` requests do NOT count towards the limit.
- **Provider Workflow:**
    - **Adopt:** Provider funds the request (`PROVIDER_ADOPTION` source). Status -> `APPROVED`. CITY_FUND not affected.
    - **Approve (Accept):** Provider accepts with City Fund (`CITY_FUND` source). Status -> `REDEEMABLE`. **No immediate ledger transfer at accept time**; the City Fund → provider transfer is executed on successful QR redemption.
    - **Reject:** Provider rejects request with a reason code and note. Status -> `REJECTED`.
- **Admin Workflow:**
    - Admins review requests in `ADMIN_PENDING` status.
    - Can `Approve` (Status -> `ADMIN_APPROVED`) or `Reject` (Status -> `ADMIN_REJECTED`).

### Database Schema
- **`requests` table:** Header information.
    - `items`: HasMany `RequestItem`
    - `reserved_amount`: Total cost of the request.
    - `funding_source`: `CITY_FUND` or `PROVIDER_ADOPTION`.
    - `rejection_reason_code`, `rejection_reason_note`: For rejections.
- **`request_items` table:** Line items.
    - `request_id`: FK to `requests`.
    - `menu_item_id`: FK to `provider_menu_items`.
    - `quantity`, `price_snapshot`.

### Routes & Controllers
- **Recipient:**
    - `GET /recipient/providers/{provider}`: Menu with Cart UI.
    - `POST /recipient/requests`: Submit multi-item request (`RecipientRequestController@store`).
    - `GET /recipient/requests`: List requests.
    - `GET /recipient/requests/{id}`: View request details.
- **Provider:**
    - `GET /provider/requests`: List incoming requests.
    - `GET /provider/requests/{id}`: View and Act (Adopt/Approve/Reject) (`ProviderRequestController`).
- **Admin:**
    - `GET /admin/requests`: View request queue.
    - `PUT /admin/requests/{id}`: Approve/Reject (`AdminRequestController`).

### Testing
- `RecipientRequestSubmissionTest`: Verifies submission, allowance logic, and item validation.
- `ProviderRequestFlowTest`: Verifies provider actions (adopt, approve, reject) and access control.
- `AdminRequestFlowTest`: Verifies admin actions and queue visibility.

# ECS-63 – Recipient Request Submission

## 1. Overview
The Recipient Request Submission feature enables registered recipients to browse provider menus and submit food requests, subject to a strict weekly allowance of 400 SAR. This feature bridges the gap between recipient needs and provider availability, ensuring efficient resource allocation while maintaining rigorous financial controls.

## 2. Business Objective
- **Financial Control**: Enforce a hard limit on recipient spending (400 SAR/week).
- **Operational Efficiency**: Automate the request process, reducing manual coordination.
- **Recipient Empowerment**: Provide a self-service interface for recipients to choose their preferred meals within budget constraints.
- **Provider Compliance**: Ensure requests are only routed to active providers with operational capacity designated as "ON".

## 3. Functional Scope
- **Request Creation**: Recipients can browse active provider menus and submit requests.
- **Allowance Enforcement**: System validates request amount against remaining weekly allowance.
- **Capacity Check**: Requests are blocked if provider capacity is OFF (`daily_capacity <= 0`).
- **Validation**: Strict checks on menu item ownership, status, and provider activity.
- **Notifications**: (Out of scope for this specific task, but planned for future).

## 4. Technical Architecture
The feature is built on top of the existing Laravel monolith, leveraging:
- **Controllers**: `RecipientRequestController` handles the business logic.
- **FormRequests**: `StoreRecipientRequest` encapsulates validation rules.
- **Eloquent Models**: `Request`, `User` (Provider), `ProviderMenuItem`, `ProviderOperatingInfo`.
- **Database**: Relational integrity enforced via foreign keys.

## 5. Database Design
A new `requests` table was introduced:
- `id`: Primary Key.
- `recipient_id`: FK to `users` (Cascade on delete).
- `provider_id`: FK to `users` (Cascade on delete).
- `menu_item_id`: FK to `provider_menu_items` (Cascade on delete).
- `quantity`: Integer.
- `price_snapshot`: Decimal(10, 2) - *Crucial for historical accuracy*.
- `status`: Enum string ('REQUESTED', 'APPROVED', 'REDEEMABLE', 'FULFILLED', 'REJECTED').
- `request_type`: String (copied from menu item category).
- `timestamps`: `created_at`, `updated_at`.

## 6. Validation & Security Rules
- **Provider Active**: Must be `is_active=true` and `status='active'`.
- **Capacity**: `ProviderOperatingInfo.daily_capacity` must be > 0.
- **Item Ownership**: `menu_item.provider_id` must match `request.provider_id`.
- **Item Active**: `menu_item.is_active` must be true.
- **Quantity**: Must be integer >= 1. Max quantity per request enforced if set on item.

## 7. Weekly Allowance Algorithm (Detailed Explanation)
The allowance is calculated dynamically at the moment of request:
1.  **Time Window**: `Carbon::now()->startOfWeek(Carbon::SUNDAY)` to `endOfWeek(Carbon::SATURDAY)`.
2.  **Usage Calculation**:
    ```sql
    SELECT SUM(price_snapshot * quantity)
    FROM requests
    WHERE recipient_id = ?
    AND created_at BETWEEN ? AND ?
    AND status IN ('REDEEMABLE', 'FULFILLED')
    ```
    *Note: 'REQUESTED' status does NOT count towards allowance.*
3.  **Check**: If `(Current Usage + New Request Amount) > 400.00`, the request is rejected with a 422 error.

## 8. Edge Case Handling
- **Concurrency**: Database transactions (implicit in Laravel) ensure atomicity.
- **Zero Capacity**: Providers with 0 capacity are hidden/disabled in UI and blocked in backend.
- **Inactive Items**: Items deactivated by providers mid-selection will trigger validation errors on submit.
- **Timezone**: All dates are stored in UTC and processed using application time (configured to Riyadh time).

## 9. RBAC & Authorization
- **Middleware**: `auth`, `role:recipient`, `account.approved`.
- **Policy**: Only users with the `recipient` role can access the submission endpoints.
- **Manual Check**: `StoreRecipientRequest::authorize()` expressly checks `$user->hasRole('recipient')`.

## 10. UI/UX Design Decisions
- **Sticky Summary**: A persistent summary panel (sidebar/bottom sheet) keeps the allowance visibility high.
- **Real-time Feedback**: Frontend calculates projected usage to warn users *before* submission.
- **Capacity Badges**: Clear visual indicators of provider availability.
- **Quantity Steppers**: Intuitive controls for adjusting order size.

## 11. Performance Considerations
- **Indexing**: Foreign keys are indexed. `created_at` index recommended for allowance queries as table grows.
- **Eager Loading**: `ProviderMenuItem` relationships are eager loaded where appropriate.
- **Query Optimization**: Allowance calculation is a single aggregate query, highly efficient.

## 12. Testing Strategy
Automated Feature tests (`tests/Feature/Recipient/RecipientRequestSubmissionTest.php`) cover:
- **Happy Path**: Valid submission.
- **Allowance Rejection**: Boundary testing at 400 SAR limit.
- **Week Boundary**: Verifying past weeks don't affect current allowance.
- **Security**: Trying to submit as non-recipient.
- **Integrity**: Ownership and status checks.

## 13. Traceability to Functional Requirements (Correct IDs)

- **FR-5.1 – Beneficiary Submit Request**  
  Implemented in `RecipientRequestController@store`, which creates a new row in the `requests` table after successful validation.

- **FR-5.2 – Validate Request Inputs**  
  Enforced via `StoreRecipientRequest` FormRequest, including:
  - Provider active check  
  - Capacity validation (`daily_capacity > 0`)  
  - Menu item ownership verification  
  - Menu item active status check  
  - Quantity validation  

- **FR-6.1 – Weekly Allowance Check (400 SAR Limit)**  
  Implemented using a dynamic aggregate query:  
  `weeklyUsed = SUM(price_snapshot * quantity)`  
  Filtered by:
  - `recipient_id = current user`
  - `created_at` between Sunday–Saturday
  - `status IN ('REDEEMABLE', 'FULFILLED')`

- **FR-6.3 – Allowance Exceeded Message**  
  When `(weeklyUsed + newAmount) > 400`, the system rejects the request and returns the exact validation message:  
  > "You have exceeded your weekly allowance of 400 SAR."


## 14. Traceability to Non-Functional Requirements (Correct IDs)

- **NFR-1 – Performance Efficiency**  
  The allowance check uses a single optimized aggregate SQL query, ensuring minimal processing overhead and fast response times.

- **NFR-6 – Usability**  
  The UI provides:
  - Clear request summary panel  
  - Real-time allowance feedback  
  - Immediate validation error messages  

- **NFR-7 – Reliability / Integrity**  
  Data integrity is ensured through:
  - Foreign key constraints  
  - Transaction-safe request creation  
  - `price_snapshot` preservation for historical accuracy  

- **NFR-4 – Security**  
  Access control is enforced via:
  - Middleware: `auth` + `role:recipient`  
  - Authorization in `StoreRecipientRequest`  
  - Automatic `403 Forbidden` response for unauthorized users  


## 15. Risks & Mitigations
- **Risk**: Pending requests ('REQUESTED') not counting could allow "allowance stuffing" if approval is slow.
    - *Mitigation*: Business rule accepted. Monitoring required.
- **Risk**: Heavy read load on `requests` table for allowance checks.
    - *Mitigation*: Index on `(recipient_id, created_at, status)` will optimize this.

## 16. Future Improvements
- **Notifications**: Email/SMS to recipient on status change.
- **Real-time Updates**: WebSockets for immediate capacity updates.
- **Soft Deletes**: Implementing SoftDeletes for audit trails.