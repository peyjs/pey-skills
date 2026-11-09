---
name: bonyan-introduction
description: Use when creating or updating docs/introduction.md of any Bonyan-architecture project (plugin, UI, host app): the plain-language document that explains what the project is and how it works, for any reader. Also when a finished task changed something a reader of that document would notice. Triggers on requests like "write the introduction", "explain this project for non-developers".
---

# Introduction Document (Bonyan)

Writes `docs/introduction.md`: a **Persian**, plain-language document that anyone can read (a product manager, a designer, a new teammate, a developer from another team) to understand what the project is, why it exists, and how it behaves, without reading code. It is short: about two printed pages at most.

It is not the technical reference and must not duplicate it:

| Document | Reader | Language | Content |
|---|---|---|---|
| `README.md` | anyone, first look | Persian | what is in the project, where to start (`readme/SKILL.md`) |
| `docs/introduction.md` (this skill) | everyone | Persian, plain | what, why, and logic in everyday words |
| `docs/reference.md` | developers/models using the project | exact | contract: methods, events, errors, config (`plugin-documentation/SKILL.md`) |

## No overlap with the reference

The topics may match (purpose, dependencies, rules, failure behavior), but the introduction describes **behavior in everyday words** and never repeats the contract: no method signatures, event names, error codes, config keys, file paths, or JSON. If the introduction and the code or `docs/reference.md` disagree, the code and the reference are right: fix the introduction, or report it if the reference itself looks wrong.

## Facts come from the project, never from memory

Every claim must be verifiable in the project: its docs, code, and tests. Never describe a capability it does not have, and never leave out a limitation it has. If the problem the project solves or its purpose is not written anywhere, ask the person instead of guessing.

## When to touch it

- **After a finished task**, only if the task changed something a reader of this document would notice (a new capability, a changed rule, a new dependency, a changed failure behavior). Fix just the affected parts. Internal refactors and tests do not change this document.
- **When asked** to create or rewrite it.

## Fixed structure

Title `# <project name> — معرفی`, then these eight headings in this order; a section that truly does not apply keeps its heading and says `ندارد`:

1. این پروژه چیست؟
2. چه مشکلی را حل می‌کند؟
3. چه می‌کند و چه نمی‌کند؟
4. چطور کار می‌کند؟
5. یک مثال واقعی
6. اگر چیزی خراب شود چه می‌شود؟
7. به چه چیزهایی وابسته است؟
8. واژه‌نامه

What each section contains, an example, and a checklist: `references/template.md`.

## Writing

- Test: someone with no programming background must follow sections 1 to 6 without help.
- Short sentences, active voice, one idea each. No code blocks or JSON anywhere. Technical identifiers appear only in the glossary.
- Define a technical term the first time it appears, or leave it to the glossary; at most one analogy, and only an accurate one.
- Persian digits and correct half-spaces (نیم‌فاصله); no marketing language.

## Never

- Copy contract details (signatures, events, errors, config, paths) from the reference.
- Promise behavior the project does not have, or hide a limitation.
- Write it in English or in a code-heavy style.
- Invent a purpose or problem that is not in the project; ask instead.
