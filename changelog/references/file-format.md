# CHANGELOG.md Format

```markdown
# تاریخچه تغییرات

تمام تغییرات این پروژه در همین فایل ثبت می‌شود.
قالب فایل بر پایه‌ی [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) و نسخه‌گذاری بر پایه‌ی [Semantic Versioning](https://semver.org/) است.

## [منتشرنشده]

### برطرف‌شده
- `get` برای کلیدی که فقط فاصله بود خطا می‌داد؛ اکنون `null` برمی‌گرداند.

## [1.1.0] - 1405-08-16

### افزوده‌شده
- متد `list()` برای دریافت کلیدهای ذخیره‌شده.

### دگرگون‌شده
- منطق اعتبارسنجی کلید در `validateKey` بازسازی شد.
- مستندات مرجع به‌روز شد.

## [1.0.0] - 1405-08-08

نخستین انتشار: ذخیره‌ی ترجیحات کاربر با `get(key)` و `set(key, value)`.

[منتشرنشده]: https://github.com/<org>/<repo>/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/<org>/<repo>/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/<org>/<repo>/releases/tag/v1.0.0
```

## Rules

- `## [منتشرنشده]` is always the first version heading; it may be empty.
- Version heading: `## [MAJOR.MINOR.PATCH] - YYYY-MM-DD`. The date is **Solar Hijri (Jalali)**, always `YYYY-MM-DD` with Latin digits and zero-padded month and day (for example `1405-08-18`), never Gregorian and never Persian digits or month names. No `v` inside the heading; the `v` belongs only to tags and compare URLs.
- Get today's Solar Hijri date with this command (Tehran time), never from memory or by converting by hand:

```
node -e "const p=new Intl.DateTimeFormat('en-US-u-ca-persian-nu-latn',{timeZone:'Asia/Tehran',year:'numeric',month:'2-digit',day:'2-digit'}).formatToParts(new Date()).reduce((a,x)=>(a[x.type]=x.value,a),{});console.log(p.year+'-'+p.month+'-'+p.day)"
```
- Newest version first. The first release may be one short paragraph instead of groups.
- Compare links at the bottom use the real URL from `git remote get-url origin`; never guess it.
- Group names, in this order, omitting empty ones: `افزوده‌شده`, `دگرگون‌شده`, `کنارگذاشته‌شده`, `برداشته‌شده`, `برطرف‌شده`, `امنیتی`.
- `کنارگذاشته‌شده` means the item still works but is no longer recommended and will be removed later. When it is actually removed, it moves to `برداشته‌شده`.
- Breaking change entry: `- **ناسازگار:** متد `remove(key)` به `delete(key)` تغییر نام یافت؛ استفاده‌کننده‌ها باید نام را به‌روز کنند.`

## Entry style

Good: one sentence, what changed.

- `- افزودن متد `list()` برای دریافت کلیدهای ذخیره‌شده.`
- `- بازسازی `validateKey` بدون تغییر رفتار.`

Bad:

- an explanation of why (`چون کاربران نیاز داشتند...`)
- a pointer to another document (`جزئیات در docs/reference.md`)
- a commit hash or issue number
- two tasks in one sentence
- English sentences
