---
name: bonyan-plugin-requirements
description: Use when writing or reviewing the requirements document of a Bonyan-architecture plugin, before any implementation plan or code exists. Triggers on requests like "write requirements for this plugin", "what should this plugin do", "document the requirements before we plan it", "is this plugin's requirements doc complete".
---

# Plugin Requirements (Bonyan)

`docs/requirements.md` lives inside the plugin's own folder, is written in **Persian**, and is the non-technical source of truth a plugin is built from. It answers *what* the plugin must do and *why*, for a real set of consumers — never *how*. The manifest and every technical artifact (service/method names, event names, error codes, config keys) are derived **from** this document in a later step (`plugin-implementation-plan/SKILL.md`); this document never invents them itself.

This is a review artifact for humans first, an AI second. Write it so a technical reader — human or model — can mechanically derive the manifest's services, events, error codes, and config keys from it without having to guess or invent a missing requirement. Precision of *behavior* is what makes that possible, not technical vocabulary.

## The what-vs-how test

Before putting any detail in this document, apply this test:

> If this detail were swapped for something else tomorrow (a different name, a different mechanism) while the real need of the consumer stayed exactly the same, is it still true that nothing in this document would have to change?

- **Yes, nothing would change** → it is an implementation detail (an event's literal name, a method's name, an error's literal code string, a config key's literal name, a specific npm package). Leave it out. Describe the underlying need instead: the trigger, the content that must become available, and who needs it.
- **No, a different choice there is a genuinely different requirements decision** → it belongs here (e.g. a new plugin-type's adapter must be able to open a connection, list/read/write/delete — losing any one of these capabilities changes what the feature can promise). Describe it as a capability, still without fixing exact method names/signatures.

This test is the general rule; the per-section guidance below is just it applied to the cases that come up most often. Apply it yourself to anything not explicitly covered.

## Required opening note

Every `requirements.md` starts with a short status block, matching existing precedent (see `pey.markdown` and `sync` plugins for real examples of the overall style, pre-dating this skill):

- Status, plugin version, Bonyan architecture version it targets, plugin type.
- What this document is based on (upstream product/consumer needs, related repos).
- The conflict rule: **in any conflict between this document and `manifest.json`, `manifest.json` wins, and this document must be corrected to match** — the manifest is the final contract, this document is not.

## Structure

Use these exact Persian section headings and identifier prefixes — never the English abbreviations (no `FR`, `NFR`, `UC`, `SEC`, `POL`, `UX`, `C`, `AC`). Every identifier is `<prefix>-<number>` in Persian, e.g. `نیاز-۳`.

### Fixed core — always present

| # | Persian heading | Prefix | Content |
|---|---|---|---|
| 1 | هدف و مسئله | — | One paragraph: why this plugin needs to exist, the problem it solves. |
| 2 | مصرف‌کنندگان و سناریوها | `سناریو` | One entry per real consumer and the real flow it drives; even a single-consumer plugin needs at least one. Ground every requirement below in one of these. |
| 3 | دامنه | — | در دامنه / بیرون دامنه (or نامقاصد): explicit in/out list to block scope creep. |
| 4 | نیازمندی‌های کارکردی | `نیاز` | Numbered, atomic, testable behaviors. A requirement that implies the consumer or another plugin must be told something, or that some behavior must be tunable, is written as trigger + content + audience (or as "باید قابل‌تنظیم باشد") — never as an event name, method name, or config key (see the test above). |
| 5 | نیازمندی‌های کیفی | `کیفیت` | Non-functional needs: performance, statelessness, scale limits, etc., in behavioral terms (a limit must exist and be adjustable — never the literal key that adjusts it). |
| 6 | امنیت | `امنیت` | What must never leak and under what condition; state explicitly even when the answer is "nothing sensitive applies here". |
| 7 | شرایط خطای متمایز | — | Each situation that must be reported as a *distinct, recognizable* failure (not merged with another, not silently swallowed), described by situation — never by an error code string. |
| 8 | وابستگی‌ها | — | What capability this plugin genuinely needs from the host/another plugin (required vs optional), and what kind of external capability it needs (e.g. "a CommonMark-compliant parser") — never the exact package name, unless the package itself is a real constraint (license, offline availability, bundle size); a real constraint like that goes in قیود, not here. |
| 9 | معیار پذیرش | `پذیرش` | Always required. Observable, end-to-end conditions that define "done" for this plugin, each traceable to one or more `نیاز`/`سناریو` entries. |
| 10 | یادداشت مرزی | — | Restate: this document fixes no technical name (no event, method, service, error code, or config key) — only behavior precise enough for the next phase to derive them; manifest.json wins on conflict. |

### Conditional — include only when actually applicable to this plugin

| Persian heading | Prefix | When to include |
|---|---|---|
| سیاست‌ها | `سیاست` | A non-negotiable rule or ordering spans multiple `نیاز` entries (e.g. the relative order operations must happen in), too structural to be just one functional requirement. |
| تعهد به میزبان | `تعهد` | This plugin's correctness genuinely depends on something only the host/UI can provide (input it must collect, a choice it must offer) — note this is the plugin's requirement *of* the host, not a UI design spec. |
| قیود | `قید` | A hard constraint beyond نیاز/کیفیت: platform restriction, licensing, "no new dependency", offline-only, etc. |
| قابلیت‌های لازم برای نوع تازه | `قابلیت` | This plugin defines a brand-new extensible plugin-type/adapter contract (rare). List the capabilities any such adapter must support (what it must be able to do), never exact method signatures — those are designed in `plugin-implementation-plan/SKILL.md`. |
| موکول‌شده | — | Something is knowingly deferred to a later version, to keep it out of scope without it being forgotten. |

Do not pad a conditional section with "N/A" — omit it entirely when it does not apply. Padding for the sake of a fixed shape defeats the point of a lean, honest document.

## Worked example (what vs how)

Wrong (technical, fixed in the requirements doc): "رویداد `sync:changed` پس از هر اجرا با شکل `{pushed, pulled, conflicts}` منتشر شود."

Right (behavioral): "بعد از هر اجرای همگام‌سازی، شمار موارد ارسال‌شده، دریافت‌شده و فهرست تعارض‌ها باید برای میزبان قابل دسترسی باشد." — the fact that this ends up as a Bonyan event, a return value, or something else is a decision for `plugin-implementation-plan/SKILL.md`, not this document.

## Boundary with other skills

- **`plugin-implementation-plan/SKILL.md`** owns everything technical this document deliberately excludes: the manifest draft, service/method names, event names, error codes, config keys, folder/file structure, and implementation steps. It is written *from* this document, never the reverse — if the plan needs to invent a requirement that is not here, stop and add it here first instead of inventing it silently in the plan.
- **`plugin-documentation/SKILL.md`** (technical reference, `docs/reference.md`) describes the plugin *as built*; this document describes the plugin *as needed*, before it exists. They will diverge over time as implementation details evolve — that divergence is expected and fine, since this document is never re-derived from the build.
- Manifest-vs-document conflicts are resolved per the opening note's conflict rule: manifest wins, this document gets corrected.

## File location

`docs/requirements.md`, inside the plugin's own folder — same place as `docs/reference.md` (`plugin-documentation/SKILL.md`). Written once up front; revised only when the actual requirement changes (not to chase an implementation detail that changed).

## Workflow position

For a **new plugin**, this is the first step: `plugin-requirements` → `plugin-implementation-plan` → `plugin-implementation` → `plugin-testing-standard` + `plugin-documentation`.
