---
name: bonyan-plugin-documentation
description: Use when writing or updating the reference documentation for a completed Bonyan-architecture plugin (the docs that ship alongside its code). Triggers on requests like "document this plugin", "write the README for X", "add plugin docs".
---

# Plugin Documentation Template

Fill this out after the plugin is implemented and tested — these are reference documents that ship alongside the plugin's code, not a planning artifact. Keep every section factual and short; this is read by other developers/models integrating this plugin later, not by the Core.

Two files, not one. Never put everything into a single `README.md`:

- **`reference.md`** — the complete, self-contained reference. Everything: Purpose, Structure, Dependencies, Public API, Events, Errors Reference, Config, Business Rules, Constraints. Someone should be able to understand and use this plugin from this file alone.
- **`README.md`** — a short, convenience entry point only: Purpose, Structure, Dependencies (the same short sections, duplicated from `reference.md`), and a link to `reference.md` for everything else. It exists for a quick first look, not as a dependency `reference.md` needs.

**The manifest is always the authoritative contract.** If either file and the plugin's actual `manifest.json` ever disagree, the manifest wins — these documents describe and explain the manifest, they never override or supersede it. If you find a mismatch while writing this, fix the mismatch (update whichever one is actually wrong) rather than documenting the disagreement as-is.

## README.md

### Purpose
What this plugin is responsible for — one or two sentences.

### Structure
Files and folders the plugin contains, one line each.

### Dependencies
Every service declared in the manifest's `dependencies` — nothing more, nothing less. Split by kind:
- **Required:** `vendor.other-plugin.service` — why it's needed.
- **Optional:** `vendor.other-plugin.service` — what capability is reduced without it.

### See Also
One line pointing to `reference.md` for the full API/events/errors/config/business-rules/constraints reference.

## reference.md

Self-contained — must make sense on its own, without requiring `README.md` to be read first.

### Purpose
What this plugin is responsible for — one or two sentences. (Same content as `README.md`'s Purpose; short enough that duplicating it costs nothing.)

### Structure
Files and folders the plugin contains, one line each. (Same content as `README.md`'s Structure.)

### Dependencies
Every service declared in the manifest's `dependencies` — nothing more, nothing less. Split by kind:
- **Required:** `vendor.other-plugin.service` — why it's needed.
- **Optional:** `vendor.other-plugin.service` — what capability is reduced without it.

(Same content as `README.md`'s Dependencies.)

### Public API
The manifest's `provides` is the formal contract; this section only explains how to use it, it does not restate or redefine it. For every method declared there, add a short usage example — not just a copy of `params`/`returns`, but how a caller actually calls it. Do not document a method here that isn't in the manifest.

### Events
Every event in the manifest's `events` — nothing more, nothing less: name, when it's triggered, and what `data` it carries. Do not document a published event that isn't declared there. Separately, you may also note events from *other* plugins this one listens to (listening never requires a manifest declaration, so this list isn't bound to any manifest section) — keep it clearly labeled as "listens to," not mixed in with what this plugin publishes.

### Errors Reference
Every error code this plugin can return, in which situation, and whether it's `operational` or `critical`.

### Config
Every config key: type, whether required, default value.

### Business Rules
Plain-language description of every business rule this plugin enforces — no code, just the rule (e.g. "a transaction cannot be created with a negative amount").

### Constraints
What must not be done with this plugin — explicit non-goals or misuse warnings (e.g. "do not call `create` from the UI directly without validating amount client-side first for UX, though the plugin re-validates regardless").