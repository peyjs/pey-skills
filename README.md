# مهارت‌های معماری بنیان (Bonyan Skills)

این مخزن مجموعه‌ای از skillها برای توسعه بر پایه معماری بنیان است؛ مهارت‌هایی که پیاده‌سازی **افزونه‌ها (plugins)** و **لایه رابط کاربری (UI)** را طبق اصول، قراردادها و استانداردهای این معماری هدایت می‌کنند.

این مهارت‌ها **برای مصرف توسط مدل‌های هوش مصنوعی** (عامل‌ها/مشاورهای کد) طراحی شده‌اند، نه صرفاً برای مطالعه توسط انسان. هر skill یک فایل `SKILL.md` دارد که در آن توضیح داده می‌شود چه زمانی باید فراخوانی شود و چه قوانینی را اعمال می‌کند. درجه‌بندی جزئیات طوری است که مدل پس از فراخوانیِ skill بتواند مستقیم طبق آن اقدام کند؛ قواعدِ غیربدیهی به‌صراحت نوشته شده‌اند تا «به‌جای حدس زدن» به فایل‌های مرجع ارجاع داده شود.

## ساختار

این مخزن از دو بخش اصلی تشکیل شده است:

| پوشه | موضوع |
|---|---|
| `plugin/` | مهارت‌های سمت افزونه‌ها (کد جاوااسکریپت، manifest، adapterها) |
| `webui/` | مهارت‌های لایه رابط کاربری (Web Components بدون فریمورک و ابزار ساخت) |

## مهارت‌های افزونه‌ها (`plugin/`)

| مهارت | نام | کاربرد |
|---|---|---|
| `coding-standard` | `bonyan-coding-standard` | استاندارد نوشتن یا بازبینی کد جاوااسکریپت: ماژول سیستم (ESM)، نام‌گذاری، سازمان‌دهی فایل‌ها، مدیریت منابع و خطا، الگوهای async و امنیت |
| `implementation-plan` | `bonyan-implementation-plan` | نوشتن طرح پیاده‌سازی یک افزونه **پیش از** نوشتن هر کدی — به عنوان مصنوع بازبینی |
| `plugin-documentation` | `bonyan-plugin-documentation` | نوشتن مستندات مرجع یک افزونه تکمیل‌شده: `README.md` (ورودی کوتاه) و `reference.md` (مرجع کامل خودکفا) |
| `plugin-implementation` | `bonyan-plugin-implementation` | پیاده‌سازی، تغییر، بازسازی یا بازبینی یک افزونه، `manifest.json`، نقطه ورود `index.js` و adapterهای مصرفی آن |
| `testing-standard` | `bonyan-testing-standard` | نوشتن تست برای افزونه‌ها تنها با `node:test` و `node:assert` — بدون هیچ فریمورک خارجی |

### پوشه `plugin/plugin-implementation/references/`

مستندات مرجع موردنیاز هنگام پیاده‌سازی افزونه:

- `plugin-fundamentals.md` — مبانی و قواعد کلی معماری افزونه (ساختار خطای استاندارد، انواع خطا و…)
- `manifest-schema.md` و `manifest-template.json` — ساختار و قالب `manifest.json`
- `naming-cheatsheet.md` — قواعد نام‌گذاری افزونه‌ها، سرویس‌ها و رویدادها
- `adapter-contract.md` و `core-api-reference.md` — قرارداد adapterها و API واقعی هسته (Core)
- `plugin-types/` — انواع افزونه: `feature`، `infrastructure`، `platform`، `product`، `transport`
- `versioning.md` — قواعد افزایش نسخه بر اساس تغییر قرارداد

## مهارت‌های رابط کاربری (`webui/`)

| مهارت | نام | کاربرد |
|---|---|---|
| `ui-implementation` | `bonyan-ui` | مهارت اصلی لایه UI: کامپوننت‌ها، UI Entry Point و manifest رابط کاربری — Web Components بومی، Shadow DOM اجباری، بدون فریمورک و ابزار ساخت |
| `coding-standard` | `bonyan-ui-coding-standard` | استاندارد کد لایه UI: نام‌گذاری، نظم اعضای کلاس کامپوننت، مدیریت منابع و خطا درون کامپوننت، امنیت |
| `documentation` | `bonyan-ui-documentation` | مستندسازی کامپوننت‌ها و «نقشه UI» (UI Map) کل پروژه |
| `testing-standard` | `bonyan-ui-testing-standard` | تست UI با `node:test` + `jsdom` (و در صورت تمایل اجرای مکمل در مرورگر واقعی) |

### پوشه `webui/ui-implementation/references/`

- `component-basics.md` — ساختار کلاس کامپوننت، رندر، مدیریت خطا و رویداد، کلاس پایه مشترک
- `composition.md` — ارتباط والد/فرزند، مونت/آن‌مونت، واکنش به مسیریابی و کامپوننت‌های صفحه
- `templates.md` — قالب شروع یک کامپوننت یا کامپوننت صفحه جدید
- `forms-and-i18n.md` — فرم‌ها، اعتبارسنجی، نمایش خطا و متن‌های محلی‌شده
- `styling-and-assets.md` — استایل، آیکون، فونت، دسترس‌پذیری و ARIA
- `shared-state.md` — فروشگاه وضعیت مشترک محلی UI
- `file-layout.md` — محل و نام فایل‌ها (`app-<name>` / `app-page-<name>`)
- `decision-guide.md` — راهنمای تشخیص داده کسب‌وکار در برابر وضعیت UI
- `forbidden-patterns.md` — الگوهای ممنوع
- `testing.md` — نمای کلی سطوح تست UI
- `architecture-context/` — چرا UI هرگز مستقیم به هسته دسترسی ندارد (`core-boundaries.md`)، ساختار manifest رابط کاربری (`ui-manifest.md`)، قرارداد سرویس (`service-contract.md`)، ساختار خطای استاندارد (`error-structure.md`) و قواعد نام‌گذاری (`naming-conventions.md`)

### پوشه `webui/documentation/references/`

فایل‌های «نقشه UI» (UI Map) — یک نقشه واحد برای کل پروژه که به چند فایل کوچک تقسیم شده است:

- `README.md` — فهرست و راهنمای انتخاب فایل مناسب
- `component-index.md` — فهرست کامپوننت‌ها و جدول مسیرها
- `entry-point.md` — رفتار نقطه ورود UI در راه‌اندازی/خاموشی
- `shared-infrastructure.md` — خلاصه کلاس پایه، کلیدهای فروشگاه وضعیت و توکن‌های طراحی
- `decisions.md` — چرایی تصمیم‌های محصول/UX
- `user-flows.md` — جریان‌های کاربری چندصفحه‌ای

## جریان کاری کلی

1. برای **افزونه جدید**: از `implementation-plan` شروع کنید، سپس `plugin-implementation` با نوع افزونه، و در نهایت `testing-standard` و `plugin-documentation`.
2. برای **کد جاوااسکریپت** (هر جا): `coding-standard`.
3. برای **UI**: `ui-implementation`، همراه `ui-coding-standard`، سپس `ui-testing-standard` و در پایان به‌روزرسانی `ui-documentation`.
4. پس از اتمام هر کار، پیش از تحویل، تست‌ها باید پاس شوند (Deliverable Gate).

## اصول کلیدی مشترک

- **بدون فریمورک و بدون ابزار ساخت** در کد اجرایی؛ فقط ES Modules بومی (و برای تست، `node:test` + `node:assert`).
- **هیچ حدسی زده نمی‌شود**: نوع افزونه، امضای API، نام‌گذاری یا فیلدهای manifest باید از فایل‌های مرجع استخراج شوند؛ در صورت ابهام، توقف و گزارش (Architecture Feedback Clause).
- **manifest قرارداد نهایی است**: کد فقط همان چیزی را پیاده می‌کند که `manifest.json` اعلام کرده است — نه بیشتر، نه کمتر.
- **مرز مالکیت**: در UI فقط Entry Point با هسته در ارتباط است؛ در افزونه‌ها فقط وابستگی‌های اعلام‌شده در دسترس‌اند.
- اگر الزامی با معماری در تعارض باشد، به‌جای حل خودسرانه تعارض، باید متوقف شد و گزارش داد.
