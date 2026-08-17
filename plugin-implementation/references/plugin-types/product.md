# Plugin Type: Product

## Definition
Implements the core business logic of the application. Without these plugins the application cannot fulfill its primary purpose.

## Examples
Transactions, accounts, categories (in a finance application).

## Type-Specific Rules

- **No `adapterType`.** This field is exclusive to infrastructure plugins. A product plugin never receives an adapter directly from the Core — if it needs adapter-level capability, it must go through the service of an infrastructure plugin.
- **No direct infrastructure/adapter dependency.** This plugin only reaches storage, external technology, or any adapter-backed capability through an infrastructure plugin's service — never by holding or calling an adapter itself.
- **Domain events:** After a state-changing operation succeeds (create/update/delete/etc.) **and other parts of the system may plausibly need to react to it**, publish a domain event. A change with no domain-level effect — purely internal bookkeeping nothing else could ever listen for — does not require one; the point of an event is to inform, not to log every mutation. Publishing events is fully allowed for this type, exactly like any other (see `../plugin-fundamentals.md`).
- **Business logic ownership:** All business rules, calculations, and validation related to this plugin's domain belong here — never in the UI, never in the Core.

## Pre-Delivery Checklist
- [ ] Manifest has no `adapterType` field.
- [ ] No adapter or infrastructure-technology API is called directly — only infrastructure plugin services.
- [ ] Every service method that changes state and has plausible external relevance publishes a corresponding domain event on success.
- [ ] Every dependency this plugin actually calls is declared in `dependencies` (required or optional, matching real necessity).
- [ ] No business rule or validation logic has leaked into the UI layer or into an adapter.