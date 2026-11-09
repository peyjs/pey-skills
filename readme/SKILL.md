---
name: bonyan-readme
description: Use when creating or updating README.md of any Bonyan-architecture project (plugin, UI, host app), or when a finished task added/removed a top-level file or folder or changed the project's dependencies. Triggers on requests like "write the README", "update the README", "fix the badges".
---

# README (Bonyan)

`README.md` sits in the project root, is written in **Persian**, and is the project's short front door: what it is, what is in it, what it depends on, how to start, the license, and where to read more. It is not documentation: no full API, error codes, config tables, or long examples; those live in `docs/reference.md`.

This skill is the only owner of `README.md`. The technical reference (`docs/reference.md`) belongs to `plugin-documentation/SKILL.md`.

## Facts come from the project, never from memory

Take every fact from the real project: the file tree, the declared dependencies, the declared version, the `LICENSE` file, and how the project is actually installed and used. If the README disagrees with them, fix the README. Never invent a file, link, badge, dependency, or install step. If you cannot tell from the project how it is installed or used, ask the person instead of guessing.

## When to touch it

- **After a finished task**, only if that task added or removed a top-level file or folder, or changed the declared dependencies or the way the project is installed or used: fix just the affected lines. Otherwise leave the README alone.
- **The version badge** changes only at release time (together with the changelog release, see `changelog/references/release.md`).
- **When asked** to create or rewrite it.

## Fixed structure

Title, four badges, then these sections in this order:

1. `## هدف`: one or two sentences on what the project is responsible for.
2. `## ساختار`: top-level files and folders only, one line each.
3. `## وابستگی‌ها`: `**الزامی:**` and `**اختیاری:**`; write `ندارد` for an empty kind or when the project has none.
4. `## نصب و استفاده`: a few lines on how the project is added and used. Installation differs per project, so this skill gives no recipe: you derive the content from the project itself.
5. `## مشارکت`: **only if** `CONTRIBUTING.md` exists; a line pointing to it. Otherwise omit the section.
6. `## مجوز`: one sentence naming the license from the `LICENSE` file, linking to it. `ندارد` if there is no file.
7. `## سندهای مرتبط`: links to existing docs, the plain-language introduction first, then the technical reference.

Exact template, badges, per-part rules, and an example: `references/template.md`.

## Writing

- Short, plain sentences. No marketing language, no emoji.
- Correct half-spaces (نیم‌فاصله). Persian digits in prose; Latin digits in versions, badges, and code.
- Names, file names, and service ids stay English, in backticks.

## Never

- Put the full API, error codes, config tables, or long examples in the README.
- Invent how the project is installed or used.
- Link a file that does not exist.
- Change the version badge outside a release.
- List nested files, or files like `.gitignore`, `package.json`, `LICENSE`, `node_modules`.
