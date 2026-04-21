# Lineone Reference – مزايا برمجية مفيدة

مرجع للملفات الداعمة في قالب Lineone. استخدمه عند نسخ صفحة أو ميزة.

---

## هيكل المجلدات

```
resources/
├── lineone-pages/          # 132 صفحة blade (نسخ للاستخدام)
├── lineone-reference/      # مرجع الملفات الداعمة
│   ├── routes-web-full.php
│   ├── PagesController.php
│   ├── js-pages-index.js   # مرجع exports (charts, tables, etc.)
│   ├── js-store-snippet.js # مرجع Alpine store
│   ├── components/         # app-layout-sideblock, base-layout, sideblock
│   └── README.md
├── js/                     # موجود في NUBL
│   ├── components/         # usePopper, accordionItem
│   ├── directives/         # tooltip, inputMask
│   ├── magics/             # notification, clipboard
│   └── pages/              # apexchartDemo, tablesDemo, formValidationDemo, etc.
└── css/
    ├── components.css      # imports لـ 24 مكون CSS
    └── pages.css           # أنماط Todo, Kanban, Chat, etc.
```

---

## مزايا برمجية (JS)

| الميزة | الملف | الاستخدام |
|--------|------|-----------|
| **ApexCharts** | `pages/apexchartDemo.js` | `pages.charts.analyticsSalesOverview` وغيرها |
| **Tom Select** | `pages/tomselectDemo.js` | `pages.tomSelect` |
| **Grid.js** | `pages/tablesDemo.js` | `pages.tables` |
| **Form Validation** | `pages/formValidationDemo.js` | `pages.formValidation` (Iodine) |
| **Credit Card** | `pages/initCreditCard.js` | `pages.initCreditCard()` |
| **usePopper** | `components/usePopper.js` | `x-data="usePopper({ placement: 'bottom-end' })"` |
| **Tooltip** | `directives/tooltip.js` | `x-tooltip="'text'"` |
| **Input Mask** | `directives/inputMask.js` | `x-input-mask` |
| **Notification** | `magics/notification.js` | `$notification()` |
| **Clipboard** | `magics/clipboard.js` | `$clipboard()` |
| **Store** | `store.js` | `$store.global` (dark mode, sidebar, etc.) |

---

## مزايا برمجية (CSS)

| الملف | المحتوى |
|------|---------|
| `components/apexcharts.css` | تنسيقات ApexCharts |
| `components/avatar.css` | Avatar, mask |
| `components/badge.css` | Badge |
| `components/button.css` | أزرار |
| `components/card.css` | بطاقات |
| `components/filepond.css` | رفع الملفات |
| `components/flatpickr.css` | Datepicker |
| `components/form.css` | حقول النماذج |
| `components/mask.css` | أقنعة الصور |
| `components/notification.css` | Toast |
| `components/popper.css` | Dropdown/Popover |
| `components/progress.css` | شريط التقدم |
| `components/quill.css` | محرر النص |
| `components/simplebar.css` | شريط التمرير |
| `components/steps.css` | خطوات |
| `components/swiper.css` | Carousel |
| `components/table.css` | جداول |
| `components/timeline.css` | Timeline |
| `components/tom-select.css` | Tom Select |
| `components/tooltip.css` | Tooltip |
| `pages.css` | Todo, Kanban, Chat, Analytics |

---

## Routes & Controller

- **routes-web-full.php** – جميع routes الخاصة بصفحات Lineone
- **PagesController** – مرجع لربط الـ routes بالـ views

لإضافة صفحة: انسخ الـ route من `routes-web-full.php` وعدّل الـ view path إلى `lineone-pages.xxx`.

---

## Dependencies (package.json)

```
Alpine.js, @alpinejs/persist, collapse, intersect
ApexCharts, dayjs, Gridjs, highlight.js, Quill, flatpickr, Tom Select
FilePond, @caneara/iodine, SimpleBar, Sortable, Swiper
@fortawesome/fontawesome-free, @popperjs/core, tippy.js, toastify-js
```

---

## استخدام صفحة

1. انسخ من `lineone-pages/` إلى المجلد المناسب
2. عدّل: استبدل `<main>` بـ `<div>`، أزل `px-[var(--margin-x)]` إن لزم
3. أضف route في `web.php` يشير إلى الـ view الجديد
