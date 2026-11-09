# README Template

```markdown
# pey.prefs

[![AI](https://img.shields.io/badge/Built%20with-AI-blueviolet)](#)
[![version](https://img.shields.io/badge/version-1.0.0-blue.svg)](#)
[![license](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Bonyan Architecture](https://img.shields.io/badge/bonyan-0.11-56c3bf)](https://github.com/barnevis/bonyan)

## هدف

افزونه‌ی ویژگی (feature) برای ذخیره‌ی ترجیحات رابط کاربری برنامه‌ی میزبان (تم، زبان) روی سرویس زیرساخت ذخیره‌سازی.

## ساختار

- `docs/` — سندهای پروژه.
- `lib/` — منطق سرویس.
- `tests/` — آزمون‌های واحد.
- `index.js` — نقطه‌ی ورود.
- `manifest.json` — قرارداد افزونه.

## وابستگی‌ها

- **الزامی:** `pey.storage.service` — تمام ذخیره‌سازی از این سرویس می‌گذرد.
- **اختیاری:** ندارد.

## نصب و استفاده

<دو تا چهار خط: چگونه این پروژه به برنامه افزوده می‌شود و چگونه استفاده می‌شود. نویسنده محتوا را از خود پروژه درمی‌آورد؛ برای جزئیات کامل به مرجع پیوند می‌دهد.>

## مشارکت

برای مشارکت در پروژه، [CONTRIBUTING.md](CONTRIBUTING.md) را بخوانید.

## مجوز

این پروژه با مجوز MIT منتشر شده است؛ متن کامل در [LICENSE](LICENSE).

## سندهای مرتبط

- [معرفی](docs/introduction.md) — شناخت افزونه به زبان ساده.
- [مرجع](docs/reference.md) — API، رویدادها، خطاها و پیکربندی.
```

The `مشارکت` section above appears only when `CONTRIBUTING.md` exists. The angle-bracket text in `نصب و استفاده` is a placeholder for the writer; it never appears in a finished README.

## Per-part rules

**Title.** The project name, in Latin, as the only level-1 heading.

**Badges.** Exactly four, in this order:
1. AI (fixed).
2. Version: the number from wherever the project declares its version (`manifest.json`, `package.json`). Update only at release.
3. License: link to `LICENSE`; include only if that file exists. Text matches the file (for example `MIT`).
4. Bonyan Architecture: the architecture version the project declares. Read the field name in `plugin-implementation/references/manifest-schema.md`; never guess it. If the project declares none, omit this badge.

**هدف.** One or two sentences: what the project is responsible for, and its kind (feature, infrastructure, UI, host app). No marketing words.

**ساختار.** Only files and folders at the project root (`ls -A` or `git ls-files` at the top level), one line each: name in backticks, a dash, a short Persian description. Folders end with `/`. Skip: `.git`, `.gitignore`, `node_modules`, `package.json`, `package-lock.json`, `LICENSE`, and anything nested. Add a line when a top-level item is added; remove it when deleted.

**وابستگی‌ها.** Exactly the dependencies the project declares (for a plugin, its manifest `dependencies`), nothing more or less. Two bullets:
- `**الزامی:**` each service id in backticks and one clause on why it is needed.
- `**اختیاری:**` each service id and what is reduced without it.
Write `ندارد` for a kind with none, and for projects that declare no dependencies at all; keep the section in every README.

**نصب و استفاده.** Two to four lines, written from what the project really needs: how it is added to a host and the smallest way to use it. A very small code block is allowed when it is the clearest way, but never a full API walk-through; point to `docs/reference.md` for that. Everything stated here must be verifiable in the project (its files, scripts, and docs). If it cannot be determined, ask the person; do not leave the placeholder in the file.

**مشارکت.** Include only when `CONTRIBUTING.md` exists in the root. One line linking to it.

**مجوز.** One sentence: the license name as written in `LICENSE`, and a link to the file. If the project has no `LICENSE` file, write `ندارد`.

**سندهای مرتبط.** One line per existing doc, link first then a dash and a short reason. Order: introduction (`docs/introduction.md`), reference (`docs/reference.md`). Add other existing docs (for example `docs/requirements.md`) after them. Never link a file that does not exist; if no doc exists, write `ندارد`.

## Checklist

- [ ] Title, four badges, and the sections in this exact order (`مشارکت` only when `CONTRIBUTING.md` exists)
- [ ] Version and architecture badges match what the project declares
- [ ] Structure lists only existing top-level items
- [ ] Dependencies match the declared ones exactly
- [ ] `نصب و استفاده` is verifiable in the project and has no leftover placeholder
- [ ] `مجوز` matches the `LICENSE` file
- [ ] Every link points to an existing file
- [ ] Persian wording with correct half-spaces; identifiers in backticks
