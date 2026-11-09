---
name: bonyan-changelog
description: Use when a task on any Bonyan-architecture project (plugin, UI, host app) is finished and CHANGELOG.md must record it, or when asked to prepare a release, decide the next version number, or update the changelog. Triggers on requests like "update the changelog", "release 1.2.0", "what version should this be".
---

# Changelog (Bonyan)

`CHANGELOG.md` lives in the project root, is written in **Persian**, and follows Keep a Changelog with `MAJOR.MINOR.PATCH` versions (rules in `plugin-implementation/references/versioning.md`; they apply to every kind of project). It is a mirror of the work done: what changed, one entry per finished task. It is not a release note for end users.

## After every finished task: record it

1. Open `CHANGELOG.md` (create it from `references/file-format.md` if missing) and add an entry under `## [منتشرنشده]` in the right group. Do not change the version number; that happens at release.
2. One task = one entry, including internal work (refactor, tests, docs). Do not merge different tasks and do not skip a task because it looks small.
3. The entry is **one short sentence** saying only what changed. No reasons, no links or references to other documents, no commit hashes. Technical names (methods, events, config keys, files) stay English in backticks; the sentence is Persian.
4. **Do not record work on something that was never released.** If the task fixes or adjusts something that exists only under `[منتشرنشده]`, edit that existing entry (or add nothing). Only a problem that exists in a released version gets a `برطرف‌شده` entry.
5. A change that forces users of the project to change their own code starts with `**ناسازگار:**` and says what they must change.
6. Never edit a released section, except to fix a factual error or typo.

Choose the group by the nature of the change:

| Group | Use for |
|---|---|
| افزوده‌شده | a new capability |
| دگرگون‌شده | changed behavior or structure, refactors, tests, docs |
| کنارگذاشته‌شده | still works, but no longer recommended and will be removed later |
| برداشته‌شده | something removed |
| برطرف‌شده | a bug that existed in a released version |
| امنیتی | a security fix |

Keep groups in this order and omit empty ones. Full template, headings, and links: `references/file-format.md`.

## Commit message

Writing a changelog entry is a file edit only. The commit message that follows is a separate step handled by `git-workflow/SKILL.md`.

## Releasing (only when asked)

Read `references/release.md` first: version choice, converting `[منتشرنشده]` into a version, syncing the version everywhere it is declared, and catching up on anything missed.

## Never

- Change the version number while recording a task.
- Write entries in English, or use Gregorian dates (dates are always Solar Hijri, see `references/file-format.md`).
- Add explanations, document references, or several tasks to one entry.
- Record work on something that was never released.
- Commit, tag, or push (see `git-workflow/SKILL.md`).
