# Component Documentation

Fill this out after the component is implemented and tested — this is a reference document that ships alongside the component's code, not a planning artifact.

**Default: one file per component**, named `<component-name>.md`, placed next to the component per `ui-implementation/references/file-layout.md`. A regular component is usually small enough (a method or two, a couple of events, a little local state) that splitting it into a short `README.md` plus a long `reference.md` — the way a plugin's documentation does — only duplicates the Purpose/Dependencies sections for no real benefit.

**Exception — split into `README.md` + `reference.md`:** only when a single component's documentation would itself be large and hard to scan in one file — typically a page-level component with a substantial public API, several published/listened events, and non-trivial config. Follow `bonyan-ui-coding-standard`'s file-length reasoning here too: split when the content genuinely doesn't fit comfortably in one file, not by default. If split, `README.md` keeps Purpose + Dependencies + a link to `reference.md`; `reference.md` keeps everything else and must stand on its own regardless.

**There is no per-component manifest — the sources of authority are split, and this document explains the split rather than restating one file.** A component doesn't have its own `manifest.json` the way a plugin does. Instead:
- The **UI manifest** (`ui-implementation/references/architecture-context/ui-manifest.md`) is authoritative for which services/config *exist to be handed down* project-wide, and for which `ui:*` event *names* are allowed to exist at all.
- The component's own `connect(refs)` signature (see `ui-implementation/references/component-basics.md`) is authoritative for which of those this *specific* component actually receives.
- If this document and the component's actual code ever disagree on what the component *currently does*, the code wins — fix the mismatch (update this document to match). But for what a component *is allowed to receive or publish* — which services/config exist to be handed down, which `ui:*` event names may exist at all — the **UI manifest is the higher authority, not the code**: if the code uses a service or dispatches a `ui:*` event that isn't in the UI manifest, that is a defect in the code (or an out-of-date manifest), not something this document should quietly go along with. Fix it at the manifest level (or the code level, whichever is actually wrong) before documenting it as if it were normal.

## Purpose
What this component is responsible for — one or two sentences. State whether it's a regular component or a page-level component (see `ui-implementation/references/composition.md`).

## Dependencies
Everything received through this component's `connect(refs)` — nothing more, nothing less. Split by kind:
- **Services:** `vendor.name.service` — why it's needed. Note whether the UI manifest declares it `required` or `optional` for the UI as a whole (see `ui-implementation/references/architecture-context/ui-manifest.md`); a component may still choose to treat an optional-at-manifest-level service as required for its own function to work at all — say so explicitly if that's the case.
- **Config values:** name, expected type, what it controls. **Document only the specific value(s) this component's `connect(refs)` actually receives** (e.g. a single `pageSize` number), never the entire project configuration object even if a larger object happens to be passed through — if only part of a config object is used, name that part specifically, not the object as a whole.
- **Route params** (page components only): each expected param name and what it represents.

## Public API
Every method callable by a parent per `ui-implementation/references/composition.md`'s public-API rule (e.g. `open()`, `close()`, `scrollToItem(id)`). For each: a short usage example — not just a signature, but how a parent actually calls it. Do not document an internal/private method here — if it needs explaining for maintainers, that belongs in a code comment, not this public reference. If the component has no public API beyond `connect()`, write "None" rather than omitting the section.

## Events

**Published (`ui:*`):** every `ui:*` event this component dispatches — name, when it's triggered, and what `detail` it carries. Cross-check each one against the UI manifest's `events` section (`ui-implementation/references/architecture-context/ui-manifest.md`) — every event listed here must already be declared there; if it isn't, that's a defect, not a documentation gap.

**Listened to (domain events):** every domain event this component subscribes to via `subscriptions()` — name, and what the component does in response. Listening never requires a manifest declaration, so this list isn't cross-checked against anything; keep it clearly labeled "listens to," not mixed in with what this component publishes.

If the component neither publishes nor listens to anything, write "None" rather than omitting the section.

## Local State
Every piece of component-local state (see `ui-implementation/references/component-basics.md`'s Local State rule) — name and what it represents. This section exists partly as a self-check: if an entry here looks like it's describing product/business data rather than a presentational flag, that's a defect to fix in the component, not to document as-is (see `ui-implementation/references/decision-guide.md`).

## Config
Every config value this component receives via `connect()`: type, whether the component treats it as required for its own function, default/fallback behavior if absent. Write "None" if the component receives no config.

## Constraints
What must not be done with this component — explicit non-goals or misuse warnings (e.g. "do not mount this directly as a page component; it is a child of `app-accounts-list` and expects `connect()` to be called by that parent, not by the Entry Point's route registry").

## Related Decisions and Flows
If a product/UX decision recorded in `ui-documentation/references/decisions.md` specifically shaped this component, or this component is a step in a journey described in `ui-documentation/references/user-flows.md`, reference the decision by its ID (e.g. "see UI-004") or link to the relevant flow entry rather than re-explaining the reasoning here. This keeps the "why" in one place instead of scattered across every component that happens to touch it.
