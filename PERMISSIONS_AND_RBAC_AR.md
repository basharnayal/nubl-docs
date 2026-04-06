# الصلاحيات والأدوار (RBAC) في النظام

هذا المستند يشرح **بشكل دقيق** كيف تعمل صلاحيات Spatie في المشروع: البيانات، الملفات، التدفق، وواجهة الإدارة، وما الذي يحدث عند تغيير الأدوار أو الصلاحيات.

**حزمة أساسية:** [`spatie/laravel-permission`](https://github.com/spatie/laravel-permission)  
**نموذج المستخدم:** `App\Models\User` يستخدم الـ trait `Spatie\Permission\Traits\HasRoles`.

---

## 1. المفاهيم الأساسية

| المفهوم | المعنى |
|--------|--------|
| **صلاحية (Permission)** | سلسلة نصية فريدة ت describe إجراءً في النظام، مثل `users.read` أو `qr.configure_ttl`. تُخزَّن في جدول `permissions` (مع `guard_name` عادةً `web`). |
| **دور (Role)** | مجموعة من الصلاحيات؛ المستخدم يُربط بدور واحد أو أكثر عبر جداول الربط. الأدوار الأساسية الأربعة: `admin`, `donor`, `recipient`, `provider`. |
| **Guard** | في هذا المشروع الافتراضي هو `web` (مطابق لـ `config('auth.defaults.guard')`). |

**التسلسل الهرمي المنطقي:**

```
مستخدم (User)
  └── أدوار (Roles) — عبر model_has_roles
        └── صلاحيات (Permissions) — عبر role_has_permissions + صلاحيات مباشرة للمستخدم اختياريًا (نادر)
```

Spatie يوفّر: `hasRole()`, `hasAnyRole()`, `can()` (للصلاحيات)، `assignRole()`, `syncRoles()`, `syncPermissions()` على الدور، إلخ.

---

## 2. المصدر الوحيد لأسماء الصلاحيات في الكود

الملف **`app/Permissions/PermissionDefinitions.php`** هو المرجع:

- `admin()` — صلاحيات تُعرَض ضمن مجموعة «إدارة» في الواجهة وتُستخدم غالبًا للأدمن.
- `donor()`, `recipient()`, `provider()` — مجموعات الأدوار غير الإدارية.
- `all()` — اتحاد كل الصلاحيات المعرفة.
- `uiGroups()` — تجميعات لعرض شبكة الصلاحيات في لوحة الأدوار (عناوين عبر مفاتيح ترجمة `rbac.group.*`).

**أي صلاحية جديدة** يجب أن تُضاف هنا أولاً، ثم تُنشأ في قاعدة البيانات عبر الـ seeder أو واجهة الإدارة (حسب ما تختار الفريق)، ثم تُترجم في الواجهة إن لزم.

---

## 3. إنشاء البيانات الأولية (Seeders)

| الملف | الوظيفة |
|--------|---------|
| `database/seeders/PermissionSeeder.php` | يستدعي `PermissionDefinitions::all()` ويُنشئ كل صلاحية بـ `Permission::firstOrCreate`، ثم يفرغ كاش Spatie للصلاحيات. |
| `database/seeders/RoleSeeder.php` | يُنشئ الأدوار الأربعة؛ يعيّن للـ **admin** جميع الصلاحيات الموجودة في الجدول؛ ويعيّن لباقي الأدوار القوائم من `PermissionDefinitions::donor()` / `recipient()` / `provider()`. |

**ترتيب التشغيل المعتاد في `DatabaseSeeder`:** `PermissionSeeder` ثم `RoleSeeder` (وغيرها حسب المشروع).

بعد تغيير الأدوار/الصلاحيات يدويًا أو من الواجهة، يُستحسن:

```bash
php artisan optimize:clear
```

لأن Spatie يخزّن صلاحيات الأدوار في الكاش للأداء.

---

## 4. قائمة الصلاحيات المعرفة (مرجع)

### 4.1 صلاحيات مجموعة الإدارة (`PermissionDefinitions::admin()`)

- حسابات وطلبات: `accounts.approve`, `requests.review`, `requests.approve`, `requests.reject`
- إعدادات QR: `qr.configure_ttl`
- مستخدمون: `users.create`, `users.read`, `users.update`, `users.delete`, `users.manage`, `users.assign.roles`, `users.deactivate`, `users.reactivate`
- أموال وسياسات: `funds.create`, `funds.read`, `funds.update`, `funds.delete`, `policies.create`, `policies.read`, `policies.update`, `policies.delete`
- تقارير: `reports.export_csv`, `reports.export_pdf`
- إعانات: `allowances.configure`
- توزيع: `allocation.pause_global`, `allocation.pause_per_provider`
- RBAC: `roles.manage`, `permissions.manage`

### 4.2 متبرع (`donor()`)

- `donations.process`, `dashboard.donor.view_stats`

### 4.3 مستفيد (`recipient()`)

- `requests.submit`

### 4.4 مزوّد (`provider()`)

- `qr.redeem`, `fulfillment_proof.upload`, `requests.adopt`, `provider.capacity.toggle`, `provider.pickup_notes_and_hours.update`

### 4.5 صلاحيات إضافية في قاعدة البيانات

إن وُجدت صلاحيات في الجدول **غير** مذكورة في `PermissionDefinitions` (مثلاً أُضيفت يدويًا)، تظهر في واجهة تعديل الدور في مجموعة **«صلاحيات أخرى»** (`rbac.group.other_permissions`).

---

## 5. كيف يُفرّق النظام بين «دخول القسم» و«صلاحية دقيقة»؟

### 5.1 حماية المسارات بالدور (الأسلوب السائد)

في `routes/web.php` تُستخدم مجموعات مثل:

- `middleware(['auth', 'phone.verified', ...])` + `account.approved` + **`role:admin`** لمجموعة `/admin/*`
- `role:donor`, `role:recipient`, `role:provider` لمسارات كل فئة

الوسيط **`role`** (`App\Http\Middleware\EnsureRole`) يتحقق بـ:

```php
auth()->user()->hasRole($role)
```

إذا لم يكن للمستخدم الدور المطلوب → **403**. لا يُقرأ هنا اسم صلاحية معيّنة.

### 5.2 وسيط الصلاحية (اختياري على المسار)

في `bootstrap/app.php` مُعرَّف:

- `'permission' => \App\Http\Middleware\EnsurePermission::class`

يستدعي `auth()->user()->can($permission)`. يمكن استخدامه في المسارات مثل:

```php
Route::middleware(['auth', 'permission:roles.manage'])->...
```

**حاليًا** لا تُستخدم مجموعات المسارات الرئيسية لهذا الوسيط بشكل افتراضي؛ الاعتماد الأساسي على `role:*` في المسارات، والصلاحيات الدقيقة في بعض المتحكمات (أدناه).

### 5.3 فحوص داخل المتحكم (`can()`)

بعض صفحات الأدمن تتطلب **دور admin** للوصول للمسار، ثم تُضيف شرطًا على **صلاحية**:

| الصلاحية | المتحكم | السلوك |
|-----------|---------|--------|
| `qr.configure_ttl` | `QrSettingsController` | `abort_unless(...->can('qr.configure_ttl'), 403)` |
| `allowances.configure` | `AllowanceSettingsController` | نفس النمط |
| `reports.export_csv` | `SummaryReportController` | نفس النمط |

إذن: إزالة هذه الصلاحية من دور المدير في لوحة الأدوار **تمنع الصفحة** حتى لو بقي المستخدم `admin` من ناحية المسار.

### 5.4 الملخص

- **الدور** يحدد *أي لوحة* (أدمن، متبرع، …) يمكنك الدخول إليها عمومًا.
- **الصلاحية** (عبر `can()` أو وسيط `permission`) تحدد *إجراءات أو صفحات فرعية* داخل تلك اللوحة إذا رُبطت في الكود.

---

## 6. لوحة إدارة الأدوار والصلاحيات

| العنصر | التفاصيل |
|--------|-----------|
| **المسار** | `/admin/roles` (أسماء المسارات: `admin.roles.*`) |
| **القائمة** | ضمن **المستخدمون → الأدوار والصلاحيات** (`SidebarPanel` + `route_match` لـ `admin.roles.*`) |
| **المتحكم** | `App\Http\Controllers\Admin\RoleController` |
| **الخدمة** | `App\Http\Services\RoleManagementService` — إنشاء/تحديث/حذف، `syncPermissions`، مسح كاش Spatie، تسجيل تدقيق |
| **التحقق من الطلبات** | `StoreRoleRequest`, `UpdateRoleRequest` |

### 6.1 الأدوار المحمية (الأساسية)

الفئة **`App\Rbac\ProtectedRoles`**: الأسماء `admin`, `donor`, `recipient`, `provider`.

- **لا يُسمح بحذفها** من الواجهة.
- **اسم الدور** لا يُغيَّر من الواجهة (لتبقى متوافقة مع `role:` في المسارات والـ middleware).
- **تعديل الصلاحيات** مسموح (مع قيود منطقية، مثل عدم إزالة كل صلاحيات دور `admin` دفعة واحدة إن كان النظام يفرض ذلك في الخدمة).

### 6.2 أدوار مخصصة

يمكن إنشاء أدوار جديدة بأسماء تتبع نمط التحقق (أحرف صغيرة، أرقام، `_`, `-`). يُحذف الدور فقط إن لم يكن مرتبطًا بمستخدمين.

---

## 7. الترجمة في الواجهة (أسماء الصلاحيات للعرض)

في **`lang/en.json`** و **`lang/ar.json`**: مفاتيح بالشكل:

`rbac.permission.{اسم_الصلاحية}`

مثال: `rbac.permission.users.read` — القيمة العربية/الإنجليزية هي العنوان الظاهر في شبكة الاختيار؛ القيمة الفعلية المخزَّنة في النموذج تبقى اسم الصلاحية التقني.

عناوين المجموعات: `rbac.group.admin_permissions`, إلخ.

---

## 8. التدقيق (Audit)

عند إنشاء أو تحديث أو حذف دور عبر الخدمة، يُستدعى **`App\Http\Services\AuditService`**:

- الكيان: `role`
- الإجراءات: `created`, `updated`, `deleted`
- الخصائص في `activity_log` تتضمن معرفات وأسماء الصلاحيات قبل/بعد عند التحديث، إلخ.

التفاصيل العامة لـ Activity Log: **`docs/AUDIT_LOG_GUIDE_AR.md`**.

---

## 9. ملفات مرجعية سريعة

| الملف | الغرض |
|--------|--------|
| `app/Permissions/PermissionDefinitions.php` | تعريفات الصلاحيات |
| `app/Rbac/ProtectedRoles.php` | الأدوار غير القابلة للحذف |
| `app/Http/Services/RoleManagementService.php` | منطق الأعمال + تدقيق |
| `app/Http/Controllers/Admin/RoleController.php` | HTTP |
| `database/seeders/PermissionSeeder.php` / `RoleSeeder.php` | البذر الأولي |
| `bootstrap/app.php` | أسماء وسيطات `role` و `permission` |
| `app/Http/Middleware/EnsureRole.php` / `EnsurePermission.php` | تنفيذ الوسيطات |

---

## 10. قائمة تحقق عند إضافة صلاحية جديدة

1. إضافة الاسم في **`PermissionDefinitions`** (في المجموعة المناسبة: `admin()` أو غيرها).
2. تشغيل seeder للصلاحيات أو إنشاء الصلاحية في قاعدة البيانات وربطها بالأدوار المناسبة (أو عبر واجهة الأدوار).
3. إضافة **`rbac.permission.{الاسم}`** في `lang/en.json` و `lang/ar.json`.
4. إن استُخدمت في كود: استدعاء `can('الاسم')` أو وسيط `permission:الاسم` على المسار.
5. `php artisan optimize:clear` عند الحاجة.

---

## 11. استكشاف أخطاء شائعة

| المشكلة | سبب محتمل |
|---------|-----------|
| صفحة أدمن معيّنة 403 رغم أن المستخدم admin | متحكم يستخدم `can('صلاحية')` وصلاحية مُزالة من الدور في لوحة الأدوار. |
| تغييرات الأدوار لا تُلاحظ فورًا | كاش Spatie؛ نفّذ `optimize:clear`. |
| صلاحية تظهر في «أخرى» | غير موجودة في `PermissionDefinitions` لكنها موجودة في الجدول. |

---

## 12. توثيق إضافي في المشروع

- **`docs/README_DEV.md`** — قسم **Roles** و **Permissions** (مختصر بالإنجليزية).
- **`docs/AUDIT_LOG_GUIDE_AR.md`** — كيفية تسجيل الأحداث وقراءة السجل.

---

*آخر تحديث للمحتوى يتوافق مع بنية المشروع الحالية (Spatie Permission + لوحة الأدوار + تعريفات مركزية).*
