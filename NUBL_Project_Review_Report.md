# NUBL — Project Review Report
### Neighborhood-Based Digital Food Assistance Platform
> Academic Presentation Report — April 2026

---

## 1) ما هو المشروع (Problem & Solution)

**المشكلة:** غياب منصة رقمية منظمة تربط المتبرعين بالمحتاجين ومقدمي الغذاء المحليين بشكل شفاف وكريم.

**الحل:** NUBL — منصة ويب تدير دورة التبرع الغذائي من الدفع حتى الاستلام.

- تربط 4 أطراف: **متبرع، مستفيد، مقدم خدمة (متجر/مطعم)، ومدير نظام**
- المتبرع يدفع إلكترونياً → الأموال تُخصَّص تلقائياً كمخصصات أسبوعية للمستفيدين
- المستفيد يتصفح قوائم المتاجر ويطلب احتياجاته ضمن حد أسبوعي محدد
- مقدم الخدمة يتحقق من الطلب عبر QR Code ويرفع إثبات التسليم
- المدير يراقب المالية، يوافق على الحسابات، ويولّد تقارير دورية

---

## 2) Stack & Architecture (High Level)

### Application Pattern
- **MVC Monolith** — Laravel 12 القياسي مع طبقة Services إضافية
- **Server-Side Rendering** — لا يوجد SPA؛ كل الصفحات Blade templates

### Frontend
- **Blade + Tailwind CSS v4 + Alpine.js** — صفحات مُرندرة من السيرفر
- **Lineone UI Kit** — مكونات جاهزة (buttons, modals, cards, alerts)
- **Vite 7** — بناء الأصول (CSS/JS bundling, HMR أثناء التطوير)
- **Alpine.js Plugins:** persist, collapse, intersect — لتفاعلات خفيفة بدون SPA

### Backend (Key Layers)
- **Controllers** — مقسمة حسب الدور: `Admin/`, `Donor/`, `Recipient/`, `Provider/`
- **Services (19 service)** — طبقة منطق الأعمال المنفصلة عن Controllers
- **Form Requests (20)** — تحقق من المدخلات (validation) منفصل عن Controllers
- **Models (23 model)** — Eloquent ORM مع علاقات واضحة
- **Jobs (3)** — معالجة خلفية: تخصيص الأموال وإعادة المحاولة
- **Notifications (12)** — إشعارات داخلية وبريدية لكل حدث مهم

### Database (الكيانات الرئيسية)
- `users` — المستخدمون (بجميع الأدوار)
- `ewallets` — المحافظ الإلكترونية (city fund + provider wallets)
- `fund_transactions` — سجل حركات الأموال
- `payments` — مدفوعات البوابة (MyFatoorah)
- `requests` + `request_items` — طلبات المستفيدين وبنودها
- `order_redemptions` + `order_proofs` — استلام الطلبات وإثباتاتها
- `provider_menu_items` + `menu_item_categories` — قوائم الطعام
- `provider_profiles`, `provider_financial_info`, `provider_documents` — بيانات المتاجر
- `recipient_profiles`, `recipient_kyc_details` — بيانات المستفيدين
- `provider_payouts` + `provider_payout_items` — تسويات مدفوعات المتاجر
- `activity_log` — سجل التدقيق (audit trail)
- `system_settings` — إعدادات النظام الديناميكية
- `summary_reports` — التقارير الدورية المُولَّدة

---

## 3) هيكلة المجلدات (Structure)

| المجلد | الوظيفة |
|--------|---------|
| `app/Http/Controllers/` | متحكمات مقسمة حسب الدور (Admin, Donor, Recipient, Provider) + Auth |
| `app/Http/Middleware/` | 8 middleware: تحقق الدور، حالة الحساب، التحقق من الهاتف/البريد، اللغة |
| `app/Http/Requests/` | 20 Form Request للتحقق من صحة المدخلات لكل عملية |
| `app/Models/` | 23 نموذج Eloquent يمثل كيانات قاعدة البيانات |
| `app/Services/` | 19 خدمة تحتوي منطق الأعمال (دفع، تخصيص، QR، SMS، إلخ) |
| `app/Support/` | 16 فئة مساعدة (UI helpers, enums-like, settings wrappers) |
| `app/Notifications/` | 12 إشعار لكل حدث مهم في النظام |
| `app/Jobs/` | 3 وظائف خلفية لمعالجة التخصيصات والمحاولات |
| `routes/web.php` | ملف Routes رئيسي واحد (~280 سطر) مع تجميع حسب الدور والصلاحية |
| `resources/views/` | قوالب Blade مقسمة: admin, donor, recipient, provider, auth, components |
| `database/migrations/` | 48 migration تصف تطور قاعدة البيانات بالكامل |
| `database/seeders/` | 12 seeder لبيانات تجريبية وأدوار وأصناف وإعدادات |
| `config/` | 21 ملف إعداد: permission, qr, provider, provider_payout, rate_limiting, إلخ |

---

## 4) الأمان والصلاحيات

- **المصادقة:** Laravel Sanctum + Breeze scaffolding (session-based)
- **التحقق الثنائي:** OTP عبر SMS (`OtpService` + `SmsService` + `EnsurePhoneVerified` middleware) — التحقق من البريد اختياري عبر `.env`
- **الأدوار (RBAC):** Spatie Laravel Permission — 4 أدوار: admin, donor, recipient, provider
- **Middleware مخصص:**
  - `EnsureRole` — يمنع الوصول لغير الدور المطلوب
  - `EnsureAccountApproved` — يحجب المستخدمين غير المعتمدين (pending/rejected)
  - `RedirectByRole` — يوجه كل دور للوحته تلقائياً
  - `EnsurePhoneVerified` — يشترط تحقق OTP قبل أي عملية
  - `EnsurePermission` — تحقق صلاحيات دقيقة (granular permissions)
  - `SetLocale` — تبديل اللغة (en/ar)
- **Rate Limiting:** حماية ضد الإساءة على: الدفع (`throttle:donor_payments`), callback البوابة (`throttle:payments_gateway`), التسجيل، رفع الصور، الإشعارات
- **تدقيق (Audit):** Spatie Activity Log + `AuditService` مخصص — كل عملية حساسة مسجلة مع SHA-256 hash
- **موافقة يدوية:** حسابات المستفيدين ومقدمي الخدمة تتطلب موافقة المدير قبل التفعيل

---

## 5) التكاملات الخارجية

| التكامل | الغرض | الموقع في الكود |
|---------|-------|-----------------|
| **MyFatoorah** | بوابة دفع إلكتروني (تبرعات) | `MyFatoorahService`, `PaymentService`, `PaymentCallbackController` |
| **Taqnyat SMS** | إرسال OTP + إشعارات SMS | `SmsService`, `OtpService` |
| **Simple QRCode** | توليد رموز QR لاستلام الطلبات | `ProviderQrController`, `RedemptionService` |
| **PHPSpreadsheet** | تصدير تقارير مالية (Excel) | `SummaryReportExportService`, `AdminFinancialService` |
| **Laravel Lang** | ترجمة الواجهة (عربي/إنجليزي) | `config/`, `SetLocale` middleware |
| **Mailtrap** | بريد تطويري (development) | `config/mail.php` |
| **Playwright** | اختبارات E2E | `package.json` scripts |

---

## 6) قرارات هندسة البرمجيات (Software Engineering Decisions)

1. **Laravel MVC + Service Layer** — فصل منطق الأعمال (19 service) عن Controllers لتسهيل الصيانة والاختبار
2. **Form Requests للتحقق** — نقل validation خارج Controllers إلى 20 request class مخصص → كود أنظف وقابل لإعادة الاستخدام
3. **Role-Based Controller Organization** — تقسيم Controllers حسب الدور (Admin/, Donor/, Provider/, Recipient/) بدل ملف واحد ضخم → فصل واضح للمسؤوليات
4. **Spatie Permission (RBAC)** — استخدام حزمة ناضجة ومجربة بدل بناء نظام صلاحيات من الصفر → وقت تطوير أقل وأمان أعلى
5. **Spatie Activity Log + SHA-256** — تدقيق شامل لكل عملية حساسة مع hash للنزاهة → شفافية ومساءلة
6. **Background Jobs (Queue)** — معالجة التخصيصات المالية وإعادة المحاولات بشكل غير متزامن → أداء أفضل وتجربة مستخدم أسرع
7. **Configuration-Driven Settings** — إعدادات ديناميكية (حد أسبوعي، TTL لـ QR، إيقاف التخصيص) عبر config + DB → مرونة بدون تعديل الكود
8. **Comprehensive Seeding (12 seeders)** — بيانات تجريبية جاهزة لكل كيان → تسهيل التطوير والعرض والاختبار
9. **Multi-Language Support (AR/EN)** — ترجمة كاملة عبر Laravel Lang → دعم مستخدمين عرب وغير عرب
10. **Custom Middleware Pipeline** — 8 middleware مخصصة تبني سلسلة أمان متعددة الطبقات (auth → phone → approval → role)
11. **Rate Limiting per Action** — حماية نقاط حساسة (دفع، تسجيل، callbacks) بمعدلات مختلفة → حماية ضد الإساءة

---

## 7) مخطط تدفق مبسّط

### رحلة التبرع → الاستلام (Core Flow)

```
المتبرع يدفع عبر MyFatoorah
    → الأموال تُضاف إلى City Fund (e-wallet)
        → النظام يخصص مبلغاً أسبوعياً لكل مستفيد مؤهل
            → المستفيد يتصفح قوائم المتاجر ويختار أصناف ويرسل طلباً
                → مقدم الخدمة يستلم إشعاراً ويقبل الطلب
                    → المستفيد يحصل على QR Code
                        → مقدم الخدمة يمسح QR ويسلّم الطلب ويرفع إثبات
                            → المدير يراقب كل شيء عبر لوحة مالية + تقارير
```

### رحلة التسجيل والموافقة

```
مستخدم جديد يسجل (recipient أو provider)
    → يتحقق من الهاتف عبر OTP
        → المدير يراجع الطلب (KYC / documents)
            → الموافقة أو الرفض (مع سبب)
                → في حال الرفض: إعادة تقديم متاحة
```

---

## Suggested Presentation Slides (English Titles)

1. **What is NUBL? — Problem, Users & Value Proposition**
2. **System Architecture — MVC + Service Layer + Blade SSR**
3. **Database Design — Core Entities & Relationships**
4. **Security & Access Control — RBAC, OTP, Approval Pipeline**
5. **External Integrations — Payments, SMS, QR, Reporting**
6. **Software Engineering Decisions — Why & How**
7. **Core User Flow — Donation to Delivery Journey**

---

*Report generated from codebase analysis — April 2026*
