# مهارت‌های چارچوب «پی» — پیاده‌سازی افزونه‌ها و رابط کاربری بر پایه معماری بنیان

[![AI](https://img.shields.io/badge/Built%20with-AI-blueviolet)](#)
[![license](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Bonyan Architecture](https://img.shields.io/badge/bonyan-0.11-56c3bf)](https://github.com/barnevis/bonyan)

این مخزن مجموعه‌ای از skillها برای پیاده‌سازی **افزونه‌ها (plugins)** و **لایه رابط کاربری (UI)** چارچوب **پی (Pey)** بر اساس معماری **بنیان (Bonyan)** است؛ مهارت‌هایی که کد را طبق اصول، قراردادها و استانداردهای این معماری هدایت می‌کنند.

این مهارت‌ها **برای مصرف توسط مدل‌های هوش مصنوعی** (عامل‌ها/مشاورهای کد) طراحی شده‌اند، نه صرفاً برای مطالعه توسط انسان. هر مهارت یک پوشه مستقل با فایل `SKILL.md` دارد که در آن توضیح داده می‌شود چه زمانی باید فراخوانی شود و چه قوانینی را اعمال می‌کند. درجه‌بندی جزئیات طوری است که مدل پس از فراخوانیِ مهارت بتواند مستقیم طبق آن اقدام کند؛ قواعدِ غیربدیهی به‌صراحت نوشته شده‌اند تا «به‌جای حدس زدن» به فایل‌های مرجع ارجاع داده شود.

## ساختار

هر مهارت **یک پوشه مستقل** است که با `SKILL.md` شروع می‌شود و در صورت نیاز `references/` خودش را دارد. برای تفکیک مهارت‌های رابط کاربری از مهارت‌های افزونه از پسوندهای نام پوشه استفاده شده است:

| الگوی پوشه | موضوع |
|---|---|
| `plugin-*` | مهارت‌های سمت افزونه‌ها (کد جاوااسکریپت، manifest، adapterها) |
| `ui-*` | مهارت‌های لایه رابط کاربری (Web Components بدون فریمورک و ابزار ساخت) |
| بدون پیشوند | مهارت‌های عمومی مشترک میان افزونه و UI (تاریخچه تغییرات، README، سند معرفی، گیت) |

## مهارت‌های افزونه‌ها (`plugin-*`)

| پوشه | نام مهارت | کاربرد |
|---|---|---|
| `plugin-coding-standard` | `bonyan-coding-standard` | استاندارد نوشتن یا بازبینی کد جاوااسکریپت: ماژول سیستم (ESM)، نام‌گذاری، سازمان‌دهی فایل‌ها، مدیریت منابع و خطا، الگوهای async و امنیت |
| `plugin-implementation-plan` | `bonyan-implementation-plan` | نوشتن طرح پیاده‌سازی یک افزونه **پیش از** نوشتن هر کدی — به عنوان مصنوع بازبینی |
| `plugin-documentation` | `bonyan-plugin-documentation` | نوشتن مرجع فنی یک افزونه در `docs/reference.md` (مرجع کامل و خودکفا) |
| `plugin-implementation` | `bonyan-plugin-implementation` | پیاده‌سازی، تغییر، بازسازی یا بازبینی یک افزونه، `manifest.json`، نقطه ورود `index.js` و adapterهای مصرفی آن |
| `plugin-testing-standard` | `bonyan-testing-standard` | نوشتن تست برای افزونه‌ها تنها با `node:test` و `node:assert` — بدون هیچ فریمورک خارجی |

### پوشه `plugin-implementation/references/`

مستندات مرجع موردنیاز هنگام پیاده‌سازی افزونه:

- `plugin-implementation/references/plugin-fundamentals.md` — مبانی و قواعد کلی معماری افزونه (ساختار خطای استاندارد، انواع خطا و…)
- `plugin-implementation/references/manifest-schema.md` و `plugin-implementation/references/manifest-template.json` — ساختار و قالب `manifest.json`
- `plugin-implementation/references/naming-cheatsheet.md` — قواعد نام‌گذاری افزونه‌ها، سرویس‌ها و رویدادها
- `plugin-implementation/references/adapter-contract.md` — قرارداد adapterها؛ و `plugin-implementation/references/core/` (سندهای زیر) — مرجع اصلی API و رفتار هسته (Core) برای پیاده‌سازی افزونه
- `plugin-implementation/references/plugin-types/` — انواع افزونه: `feature`، `infrastructure`، `platform`، `product`، `transport`
- `plugin-implementation/references/versioning.md` — قواعد افزایش نسخه بر اساس تغییر قرارداد

### پوشه `plugin-implementation/references/core/`

سندهای مرجع هسته — مرجع اصلی استفاده از هسته هنگام پیاده‌سازی افزونه؛ در تعارض با خلاصه‌های سطح مهارت، این پوشه ملاک است (محتوای آن دست‌نخورده و معتبر نگه داشته می‌شود):

- `plugin-implementation/references/core/api.md` — مرجع API هسته: ساخت `PeyCore`، `start`/`shutdown`، منابع Runtime Host، زمینه‌های `prepare`/`activate` و ماژول‌های داخلی
- `plugin-implementation/references/core/adapters.md` — اتصال adapterها: اعلام در `bootstrap.json`، مسیریابی `adapterType` و تحویل به افزونه زیرساختی
- `plugin-implementation/references/core/manifest-schema.md` — اسکیمای manifest افزونه و رابط کاربری + جدول قوانین اعتبارسنجی
- `plugin-implementation/references/core/plugin-authoring.md` — راهنمای نوشتن افزونه از دید هسته (ساختار، دو فاز، هوک‌ها، پیکربندی، رویدادها)
- `plugin-implementation/references/core/error-codes.md` — فهرست کدهای خطای هسته
- `plugin-implementation/references/core/events.md` — فهرست رویدادهای هسته (`core:*`) و رویدادهای چرخه حیات افزونه

## مهارت‌های رابط کاربری (`ui-*`)

| پوشه | نام مهارت | کاربرد |
|---|---|---|
| `ui-implementation` | `bonyan-ui` | مهارت اصلی لایه UI: شروع هر کار از کیت `pey.webui` (تنها نقطه شروع مجاز)، کامپوننت‌ها، UI Entry Point و manifest رابط کاربری — Web Components بومی، Shadow DOM اجباری، بدون فریمورک و ابزار ساخت |
| `ui-coding-standard` | `bonyan-ui-coding-standard` | استاندارد کد لایه UI: نام‌گذاری، نظم اعضای کلاس کامپوننت، مدیریت منابع و خطا درون کامپوننت، امنیت |
| `ui-documentation` | `bonyan-ui-documentation` | مستندسازی کامپوننت‌ها و «نقشه UI» (UI Map) کل پروژه |
| `ui-testing-standard` | `bonyan-ui-testing-standard` | تست UI با `node:test` + `jsdom` (و در صورت تمایل اجرای مکمل در مرورگر واقعی) |

### پوشه `ui-implementation/references/`

در این مهارت، **منبع اصلی و الزامی هر موضوع، خود کیت `pey.webui` است** (`pey.webui/docs/`، `pey.webui/templates/` یا ماژول‌های کیت). فایل‌های `references/` فقط پیش‌زمینه اختیاری برای فهم «چرا» هستند:

- `ui-implementation/references/component-basics.md` — ساختار کلاس کامپوننت، رندر، مدیریت خطا و رویداد، کلاس پایه مشترک
- `ui-implementation/references/composition.md` — ارتباط والد/فرزند، مونت/آن‌مونت، واکنش به مسیریابی و کامپوننت‌های صفحه
- `ui-implementation/references/forms-and-i18n.md` — فرم‌ها، اعتبارسنجی، نمایش خطا و متن‌های محلی‌شده
- `ui-implementation/references/styling-and-assets.md` — استایل، آیکون، فونت، دسترس‌پذیری و ARIA
- `ui-implementation/references/shared-state.md` — فروشگاه وضعیت مشترک محلی UI
- `ui-implementation/references/file-layout.md` — محل و نام فایل‌ها (`app-<name>` / `app-page-<name>`)
- `ui-implementation/references/decision-guide.md` — راهنمای تشخیص داده کسب‌وکار در برابر وضعیت UI
- `ui-implementation/references/forbidden-patterns.md` — الگوهای ممنوع
- `ui-implementation/references/testing.md` — نمای کلی سطوح تست UI
- `ui-implementation/references/architecture-context/` — چرا UI هرگز مستقیم به هسته دسترسی ندارد (`ui-implementation/references/architecture-context/core-boundaries.md`)، ساختار manifest رابط کاربری (`ui-implementation/references/architecture-context/ui-manifest.md`)، قرارداد سرویس (`ui-implementation/references/architecture-context/service-contract.md`)، ساختار خطای استاندارد (`ui-implementation/references/architecture-context/error-structure.md`) و قواعد نام‌گذاری (`ui-implementation/references/architecture-context/naming-conventions.md`)

> برخی منابع خارج از این مخزن هستند و به همین شکل ارجاع داده می‌شوند: کیت `pey.webui` (پکیج نصب‌شده)، و مفاهیم مستقل از کیت در `plugin-implementation/references/core/*.md`، `router-reference.md` و `pey-router-manifest.json` (نگاه کنید به جدول «Sources — Concepts Outside The Kit» در `ui-implementation/SKILL.md`).

### پوشه `ui-documentation/references/`

فایل‌های «نقشه UI» (UI Map) — یک نقشه واحد برای کل پروژه که به چند فایل کوچک تقسیم شده است:

- `ui-documentation/references/README.md` — فهرست و راهنمای انتخاب فایل مناسب
- `ui-documentation/references/component-index.md` — فهرست کامپوننت‌ها و جدول مسیرها
- `ui-documentation/references/entry-point.md` — رفتار نقطه ورود UI در راه‌اندازی/خاموشی
- `ui-documentation/references/shared-infrastructure.md` — خلاصه کلاس پایه، کلیدهای فروشگاه وضعیت و توکن‌های طراحی
- `ui-documentation/references/decisions.md` — چرایی تصمیم‌های محصول/UX
- `ui-documentation/references/user-flows.md` — جریان‌های کاربری چندصفحه‌ای
- `ui-documentation/references/component-documentation.md` — قالب مستندات یک کامپوننت/صفحه (به‌صورت هم‌مکان با خود کامپوننت)

## مهارت‌های عمومی

مشترک میان افزونه‌ها، رابط کاربری و برنامه‌ی میزبان.

| پوشه | نام مهارت | کاربرد |
|---|---|---|
| `changelog` | `bonyan-changelog` | ثبت هر کار انجام‌شده در `CHANGELOG.md` (فارسی، تاریخ خورشیدی) و آماده‌سازی انتشار و انتخاب نسخه |
| `readme` | `bonyan-readme` | نوشتن و به‌روزرسانی `README.md` کوتاه فارسی پروژه |
| `introduction` | `bonyan-introduction` | نوشتن `docs/introduction.md`: معرفی ساده‌ی پروژه و منطق آن برای همه |
| `git-workflow` | `bonyan-git-workflow` | پیام کامیت یک‌خطی پس از هر کار؛ کامیت، تگ و push تنها با درخواست صریح |

## جریان کاری کلی

1. برای **افزونه جدید**: از `plugin-implementation-plan` شروع کنید، سپس `plugin-implementation` با نوع افزونه، و در نهایت `plugin-testing-standard` و `plugin-documentation`.
2. برای **کد جاوااسکریپت** (هر جا): `plugin-coding-standard`.
3. برای **UI**: `ui-implementation` (با شروع از کیت `pey.webui`)، همراه `ui-coding-standard`، سپس `ui-testing-standard` و در پایان به‌روزرسانی `ui-documentation`.
4. پس از اتمام هر کار، پیش از تحویل، تست‌ها باید پاس شوند (Deliverable Gate).
5. سپس کار بسته می‌شود: ثبت ردیف در تاریخچه تغییرات (`changelog`)، به‌روزرسانی `readme`، `introduction` و مستندات فقط اگر کار چیزی از آن‌ها را عوض کرده، و در پایان یک پیام کامیت یک‌خطی (`git-workflow`) بدون اجرای هیچ دستور گیت.

## اصول کلیدی مشترک

- **بدون فریمورک و بدون ابزار ساخت** در کد اجرایی؛ فقط ES Modules بومی (و برای تست، `node:test` + `node:assert`).
- **UI از صفر ساخته نمی‌شود**: در لایه UI تنها نقطه شروع مجاز، کیت `pey.webui` است؛ کد داخل کیت هرگز ویرایش نمی‌شود و شکاف‌های کیت گزارش می‌شوند نه وصله.
- **هیچ حدسی زده نمی‌شود**: نوع افزونه، امضای API، نام‌گذاری یا فیلدهای manifest باید از فایل‌های مرجع استخراج شوند؛ در صورت ابهام، توقف و گزارش (Architecture Feedback Clause).
- **manifest قرارداد نهایی است**: کد فقط همان چیزی را پیاده می‌کند که `manifest.json` اعلام کرده است — نه بیشتر، نه کمتر.
- **مرز مالکیت**: در UI فقط Entry Point با هسته در ارتباط است؛ در افزونه‌ها فقط وابستگی‌های اعلام‌شده در دسترس‌اند.
- اگر الزامی با معماری در تعارض باشد، به‌جای حل خودسرانه تعارض، باید متوقف شد و گزارش داد.