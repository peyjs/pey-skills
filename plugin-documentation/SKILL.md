---
name: bonyan-plugin-documentation
description: Use when writing or updating the technical reference (docs/reference.md) of a Bonyan-architecture plugin, or when a finished task changed the plugin's contract, behavior, config, or errors. Triggers on requests like "document this plugin", "write the reference", "update the plugin docs".
---

# Plugin Reference (Bonyan)

Writes `docs/reference.md`, the plugin's complete, self-contained technical reference, in **English**. Fill it out after the plugin is implemented and tested. It ships alongside the plugin's code and is read by other developers and models integrating the plugin, not by the Core. Keep every section factual and short.

This skill writes only `docs/reference.md`. The other documents have their own skills and are not duplicated here:
- `README.md`: `readme/SKILL.md`
- `docs/introduction.md` (plain-language, Persian): `introduction/SKILL.md`

**Self-contained.** Someone must be able to understand and use the plugin from this file alone, without reading any other document first.

**The manifest is always the authoritative contract.** If this file and the plugin's actual `manifest.json` ever disagree, the manifest wins: this document describes and explains the manifest, it never overrides it. If you find a mismatch, fix whichever one is actually wrong rather than documenting the disagreement.

## When to touch it

- **After the plugin is implemented and tested**: write the whole file.
- **After a finished task** that changed the plugin's contract (`provides`, `events`, `dependencies`), behavior, config, errors, or files: fix only the affected sections. Tasks that change none of these (refactors, tests) leave it alone.

## Sections

Use these headings in this order, in `docs/reference.md`.

### Purpose
What this plugin is responsible for: one or two sentences.

### Structure
Files and folders the plugin contains, including nested ones, one line each.

### Dependencies
Every service declared in the manifest's `dependencies`: nothing more, nothing less. Split by kind:
- **Required:** `vendor.other-plugin.service`: why it's needed.
- **Optional:** `vendor.other-plugin.service`: what capability is reduced without it.

### Public API
The manifest's `provides` is the formal contract; this section only explains how to use it, it does not restate or redefine it. For every method declared there, add a short usage example: not just a copy of `params`/`returns`, but how a caller actually calls it. Do not document a method here that isn't in the manifest.

### Events
Every event in the manifest's `events`: nothing more, nothing less: name, when it's triggered, and what `data` it carries. Do not document a published event that isn't declared there. Separately, you may note events from *other* plugins this one listens to (listening never requires a manifest declaration), clearly labeled "listens to" and not mixed in with what this plugin publishes.

### Errors Reference
Every error code this plugin can return, in which situation, and whether it's `operational` or `critical`.

### Config
Every config key: type, whether required, default value.

### Business Rules
Plain-language description of every business rule this plugin enforces: no code, just the rule (e.g. "a transaction cannot be created with a negative amount").

### Constraints
What must not be done with this plugin: explicit non-goals or misuse warnings (e.g. "do not call `create` from the UI directly without validating amount client-side first for UX, though the plugin re-validates regardless").

## Never

- Document a method, event, config key, or error the manifest and code do not have.
- Leave a disagreement between this file and the manifest in place.
- Copy README or introduction content into this file beyond Purpose, Structure, and Dependencies.