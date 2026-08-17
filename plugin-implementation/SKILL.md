---
name: bonyan-plugin-implementation
description: Use when implementing, modifying, refactoring, or reviewing a Bonyan-architecture plugin (infrastructure, product, feature, platform, or transport type), its manifest.json, its index.js entry point, or an adapter it consumes. Triggers on requests like "add a plugin for X", "implement the Y service", "write the manifest for Z", "create/modify an adapter for W", "refactor plugin X".
---

# Bonyan Plugin Implementation

Bonyan has precise, non-obvious rules that differ from generic plugin-architecture conventions. Do not rely on general prior knowledge — follow only the referenced files. **Never guess**: not a plugin type, not an adapter/Core API signature, not a naming format, not a manifest field. If unclear, stop and ask, or report per the Architecture Feedback Clause below.

## General Principles
- Respect architecture boundaries: only declared dependencies are reachable; only infrastructure gets `adapterType`; etc.
- Minimum necessary dependencies and provided services — never add one "for convenience."
- The manifest is finalized before implementation code, and code implements exactly what it declares — never the reverse (no undeclared capability, nothing declared left unimplemented).

## Workflow
1. **Requirements first.** If a requirements document path is given in the task, read it before anything else on this list.
2. Read `references/plugin-fundamentals.md`.
3. **Plugin type is given in the task.** Read the matching `references/plugin-types/<type>.md`. If the task doesn't state a type, ask before proceeding — never guess. If something in the task clearly contradicts the stated type (e.g. type is "feature" but the task asks for `adapterType`), do not silently reassign it or silently comply — stop and report per "Requirement vs. Architecture Conflict" below.
4. **Modifying/refactoring an existing plugin?** Read its current `manifest.json` and code first. Preserve behavior not targeted by the task. If the change adds/removes/changes a provided service, method, event, or dependency — the plugin's contract has changed: read `references/versioning.md` and bump the version accordingly (MAJOR/MINOR/PATCH). If the change is purely internal with no contract change, the manifest (beyond `version`/`description` if relevant) stays untouched.
5. **Manifest changed or this is a new plugin?** Write/update it via `references/manifest-schema.md` + `references/manifest-template.json`, using `references/naming-cheatsheet.md` for every name, before writing implementation code. If nothing about the contract changed, skip this step.
6. **Plugin consumes an adapter?** Read `references/adapter-contract.md` and `references/core-api-reference.md` for the real Core-side wiring. Never guess a method name or signature for either — if it isn't in these files, stop and report.
7. **Before calling the work done, validate:**
    - [ ] Every dependency actually called is declared — and nothing more.
    - [ ] `adapterType` present only if type is infrastructure.
    - [ ] No sensitive data in any event payload or error structure.
    - [ ] Every error crossing the plugin boundary uses the standard structure.
    - [ ] Manifest and code agree exactly — no `provides` entry, method, or event exists in code without being declared, and nothing declared is left unimplemented.
    - [ ] `index.js`'s `prepare()` returns an array (even `[]` if no services) — never `undefined`/nothing; `activate()` receives only `services`, not a full context — per `references/core-api-reference.md`.
    - [ ] `onShutdown` and `onDeactivated` are assigned inside `prepare()`, never inside `activate()` — `activate()` may never run for a plugin that fails settlement/activation, silently losing hooks assigned there.
    - [ ] If this plugin has a `required` dependency, `activate()`/`prepare()` subscribes to `core:service-unavailable` (via the saved prepare-time context) and reports a critical error if that dependency's service name appears **after this plugin is already active** — this is not automatic at runtime (it *is* automatic before group commit, during settlement — see `references/core-api-reference.md`).
    - [ ] Naming follows `references/naming-cheatsheet.md` everywhere.

## Requirement vs. Architecture Conflict
If a requirement conflicts with the architecture, do not silently follow the requirement and do not silently modify the architecture to fit it. Stop, report the conflict, and wait — resolving it (including any deliberate architecture change) is a decision for the human/architecture owner, not something to decide mid-implementation.

## Architecture Feedback Clause
Follow the referenced files exactly. If something is unresolved or contradictory between them, do not invent an interpretation — stop, report the exact file/section/gap, and wait.