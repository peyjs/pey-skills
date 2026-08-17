---
name: bonyan-implementation-plan
description: Use when asked to write an implementation plan for a Bonyan-architecture plugin before any code is written. Triggers on requests like "write an implementation plan for X", "plan out this plugin before building it".
---

# Implementation Plan Template

Fill this out before writing any code, only when the task explicitly asks for a plan first. Keep every section short — this is a review artifact for humans and other models, not documentation. If a section doesn't apply, write "N/A" rather than padding it.

## 1. Purpose and Scope
- **What the Plugin Is** — one or two sentences.
- **What the Plugin Is Not** — explicit non-goals, to prevent scope creep.
- **Plugin Responsibilities** — bullet list, each one a single responsibility.
- **Design Principles (Non-Negotiable)** — only principles specific to this plugin beyond the general architecture (e.g. "never stores unencrypted card numbers"). Do not restate general Bonyan rules already in `plugin-fundamentals.md`.
- **Technology Constraints** — any real constraint (e.g. must run in browser only, must not exceed X ms per call).

## 2. Overview
One paragraph: what this plugin does end to end, in plain terms.

## 3. Key Data Structures
The entities **this plugin owns or shapes** — name and shape only (field names + types), no implementation. Not the full manifest `params`/`returns` detail — just enough to recognize the entity. Don't re-describe an entity that belongs to a dependency and merely passes through unchanged — reference it by name only; its shape is that other plugin's documentation to own.

## 4. Error Management
List only the error codes specific to this plugin (e.g. `TRANSACTION_NOT_FOUND`, `LIMIT_EXCEEDED`). Do not restate the standard error structure or the three error types — those are in `plugin-fundamentals.md`.

## 5. Security
Plugin-specific notes only: which fields/data are sensitive, which declared dependency is genuinely necessary and why. Do not restate the general security baseline.

## 6. Events Reference
| Event | Published when | Data |
|---|---|---|

## 7. Folder Structure and Files
Tree of files this plugin will have, each with its exposed method names only (no signatures — those live in the manifest draft, item 7a).

```
plugin-root/
  manifest.json
  index.js                    — methods: prepare, activate
  <service-name>-service.js   — methods: ...
  ...
```

**7a. Manifest Draft** — attach the actual draft `manifest.json` content here (or as a linked file), following `manifest-schema.md` / `manifest-template.json`. This is the primary review artifact — review it as a contract, not the prose above.

## 8. Documents to Be Written
List only: filename and one-line purpose (e.g. a README, a migration note). Not the content itself.

## 9. Tests Per Part
List only, no test code: which scenarios will be covered per file/service, referencing `testing-standard.md` categories (success path, error path, event published, dependency-unavailable path, etc.).

## 10. Implementation Steps
Ordered list of what gets built in what order, file by file (e.g. "1. manifest.json → 2. storage-adapter fake for tests → 3. transactions-service.js → 4. index.js wiring → 5. tests").

## 11. Architectural Ambiguities
Anything encountered while planning that the architecture doesn't clearly resolve or that conflicts between referenced files. Report per the Architecture Feedback Clause — do not resolve these yourself with an invented interpretation. Write "None" if there are none.