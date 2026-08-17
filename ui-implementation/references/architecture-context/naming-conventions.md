# Naming Conventions

Self-contained summary of `naming.md`'s rules, limited to what's relevant when writing UI code that references plugin/service/event names.

## Plugin and Service Names

Format: `vendor.name` for a plugin (e.g. `myfinance.transactions`), and `vendor.name.service` for the service it provides (e.g. `myfinance.transactions.service`). When a component is handed a service reference by the Entry Point, its manifest-facing name (in the UI manifest's `dependencies`, see `ui-manifest.md`) always follows this exact suffix pattern.

## Event Names

Two distinct kinds, and the UI must not blur them:

- **Domain events**, format `pluginname:eventname` (no vendor prefix, just the second segment), **past tense** — describing something that already happened, e.g. `transactions:saved`, `accounts:updated`. A component reacting to one of these (per `component-basics.md`/`composition.md`) always treats it as "this already happened," never as a request to do something.
- **UI action events**, always prefixed `ui:`, describing a lightweight UI-only change — e.g. `ui:filter-changed`, `ui:panel-opened`, `ui:component-error`. Every `ui:*` event a component might ever publish must be declared once in the UI manifest (see `ui-manifest.md`) — it is never invented ad hoc inside a component file without that declaration existing.

## General Rules That Apply to UI Code Too

- Lowercase, hyphens for multi-word names (`ui:view-mode-changed`, not `viewModeChanged` or `ui:viewModeChanged`).
- No abbreviations — `transactions`, not `txn`. Applies equally to any new event or reference name introduced while building UI components.
- A UI action event never replaces a direct service call for a core product operation (saving, deleting, etc.) — naming something `ui:` does not change the primary-path rule in `service-contract.md`; it only marks the event as lightweight/presentational in nature.
