# Introduction Template and Rules

Title: `# <project name> — معرفی`

## Sections

**۱. این پروژه چیست؟** Two or three sentences: its job, in words a non-programmer understands.

**۲. چه مشکلی را حل می‌کند؟** The situation without it and what goes wrong or gets tedious.

**۳. چه می‌کند و چه نمی‌کند؟** Two short lists. «می‌کند» lists its capabilities. «نمی‌کند» lists deliberate non-goals, items not done yet («هنوز انجام نمی‌دهد»), and misuse warnings and limits (for example: not meant for storing passwords).

**۴. چطور کار می‌کند؟** The logic as a short numbered story in the order things happen, including the rules it enforces, each as a plain sentence. This is the core of the document.

**۵. یک مثال واقعی.** One concrete scenario with invented but plausible values, told as a short story of what a user or the host application experiences. It must match the project's real behavior; check it against the code or tests.

**۶. اگر چیزی خراب شود چه می‌شود؟** What the user or host sees on failure, using the project's actual behavior, never a promise it does not make.

**۷. به چه چیزهایی وابسته است؟** Each dependency in one plain sentence: what the project needs from it and what changes without it. `ندارد` if none.

**۸. واژه‌نامه.** Every technical term used above with a one-line meaning. Only terms that actually appear.

## Example

````markdown
# pey.prefs — معرفی

## این پروژه چیست؟

این افزونه ترجیحات کاربر برنامه، مثل تم و زبان، را به‌خاطر می‌سپارد تا با باز شدن دوباره‌ی برنامه از دست نروند.

## چه مشکلی را حل می‌کند؟

هر برنامه برای نگه‌داشتن ترجیحات سبک کاربر به یک راه ساده نیاز دارد. اگر هر صفحه مستقیم سراغ ذخیره‌سازی برود، هر برنامه قواعد خودش را می‌سازد و خرابی ذخیره‌سازی به خود صفحه‌ها می‌رسد.

## چه می‌کند و چه نمی‌کند؟

می‌کند:
- یک ترجیح را با نامش می‌نویسد و می‌خواند.

نمی‌کند:
- معنای ترجیحات را نمی‌داند و درستی مقدارها را نمی‌سنجد؛ این با برنامه است.
- فهرست ترجیحات را نمی‌دهد و چند ترجیح را یک‌جا نمی‌خواند.
- برای نگه‌داری رمز یا اطلاعات حساس مناسب نیست.

## چطور کار می‌کند؟

۱. برنامه از افزونه می‌خواهد ترجیحی، مثلاً «تم تیره»، را نگه دارد.
۲. افزونه آن را در بخش ذخیره‌سازی برنامه می‌نویسد.
۳. دفعه‌ی بعد که برنامه باز می‌شود، همان ترجیح را می‌پرسد و صفحه با همان تم باز می‌شود.
۴. افزونه مقدار را همان‌طور که هست نگه می‌دارد و برمی‌گرداند و تفسیرش نمی‌کند.

## یک مثال واقعی

سارا تم تیره را انتخاب می‌کند. فردا که برنامه را باز می‌کند، صفحه از همان اول تیره است، بدون اینکه دوباره انتخاب کند.

## اگر چیزی خراب شود چه می‌شود؟

- اگر ترجیحی ذخیره نشده باشد یا خواندنش نشود، افزونه «چیزی پیدا نشد» می‌گوید و برنامه با حالت پیش‌فرض خودش ادامه می‌دهد.
- اگر نوشتن شکست بخورد، افزونه خبر می‌دهد که ذخیره نشد؛ برنامه از کار نمی‌افتد.
- اگر بخش ذخیره‌سازی وسط کار از دسترس بیرون برود، سیستم یک هشدار جدی ثبت می‌کند.

## به چه چیزهایی وابسته است؟

به بخش ذخیره‌سازی برنامه؛ تمام نوشتن و خواندن از آن می‌گذرد و بدون آن افزونه نمی‌تواند چیزی نگه دارد.

## واژه‌نامه

- **ترجیح:** تنظیم سبکی که کاربر انتخاب می‌کند، مثل تم یا زبان.
- **ذخیره‌سازی:** بخشی از برنامه که داده‌ها را نگه می‌دارد.
````

Good vs bad (section ۴):

- Good: «برنامه از افزونه می‌خواهد ترجیح کاربر را به‌خاطر بسپارد؛ افزونه آن را می‌نویسد و دفعه‌ی بعد برمی‌گرداند.»
- Bad: «`set(key, value)` رکورد را در collection روی `pey.storage.service` می‌نویسد و `true` برمی‌گرداند.» (jargon, contract detail, copy of the reference)

## Checklist

- [ ] Title and all eight headings in order; unused ones say `ندارد`
- [ ] Every claim is verifiable in the project; limits and non-goals stated honestly
- [ ] Sections 1 to 6 readable without programming knowledge
- [ ] No code, JSON, signatures, event names, error codes, config keys, or paths outside the glossary
- [ ] The example matches real behavior
- [ ] Glossary covers every technical term used
- [ ] No contradiction with `docs/reference.md` or the code
