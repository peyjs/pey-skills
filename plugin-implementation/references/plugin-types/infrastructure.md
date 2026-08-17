# Plugin Type: Infrastructure

## Definition
Provides base services that other plugins depend on.

## Examples
Storage, routing, state management, logging, internationalization.

## Type-Specific Rules

- **`adapterType` is optional, not required.** An infrastructure plugin that consumes no external/adapter-backed technology (e.g. a pure in-memory state-management plugin) is fully valid without declaring `adapterType` at all.
- **`adapterType` field:** Only plugins with `"type": "infrastructure"` may declare `adapterType` in their manifest. If a plugin of any other type declares it, validation fails.
- **Adapter handling:** If this plugin declares `adapterType`, it is solely responsible for preparing and using the adapter of that type — see `../adapter-contract.md` for the adapter's own rules, including the two possible selection forms (plain string, or an object with `implementation` + extra config). The Core only extracts the adapter's configured name (and optional extra config); it does not instantiate or validate the adapter itself. If the project's `bootstrap.json` selection for this type is malformed (neither a string nor an object with `implementation`), the Core fails this plugin before `prepare()` is ever called — nothing for you to handle in code, but don't assume `config.adapter` shape without reading `../adapter-contract.md`.
- **Error conversion:** Raw errors from the adapter must be converted to Bonyan's standard error structure before they leave this plugin's boundary.
- **Publishing events is fully allowed.** Nothing about this type restricts it — declare and publish domain events exactly like any other plugin type (see `../plugin-fundamentals.md`).

## Pre-Delivery Checklist
- [ ] If `adapterType` is declared, is `"type"` exactly `"infrastructure"`?
- [ ] Does the `adapterType` value match one of the recognized adapter kinds (see `../adapter-contract.md`), not an arbitrary string?
- [ ] Are all errors from the adapter converted to the standard Bonyan error structure before returning from the service?