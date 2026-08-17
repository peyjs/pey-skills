# Naming Cheatsheet (Bonyan)

Normative reference. If something is not covered here, do not guess — report it per the Architecture Feedback Clause in `plugin-implementation/SKILL.md`.

## Format Table

| Thing | Format | Example |
|---|---|---|
| Plugin name | `vendor.name` | `myfinance.transactions` |
| Service name | `vendor.name.service` | `myfinance.transactions.service` |
| Domain event | `pluginname:eventname` (past tense, no vendor segment) | `transactions:saved`, `storage:ready` |
| Core event | `core:eventname` | `core:plugin-failed`, `core:ready` |
| UI action event | `ui:action-name` (noun form, no vendor segment) | `ui:filter-changed`, `ui:panel-opened` |
| Adapter type key (in `bootstrap.json` → `adapters`) | single lowercase word, matches `adapterType` in manifest | `storage`, `auth`, `notification` |
| File name | `kebab-case` (per general coding standard) | `transaction-service.js` |

## Plugin Name (`vendor.name`)

- **vendor**: the person, team, or organization that wrote the plugin. Prevents collisions across sources.
- **name**: short, descriptive, says what the plugin does.
- No two plugins **loaded into the same running Core instance** (i.e. listed in the same `bootstrap.json`) may share the same name — checked at startup. This is not global uniqueness across every Bonyan/Pey project; it's why the `vendor` segment exists — to avoid collisions when plugins from different sources end up loaded together.

## Service Name (`vendor.name.service`)

- Always the plugin's own name plus a literal `.service` suffix.
- A plugin registers a service under this name in `provides`; other plugins reference this exact string in `dependencies`.

### Service Method Names (inferred from the architecture's own example, not a directly stated rule)
The architecture's naming reference has no explicit rule for this; the pattern below is extracted from the one real example in the architecture's manifest reference (`getAll`, `getById`, `create`, `update`, `delete` on a transactions service) and general `plugin-coding-standard/SKILL.md` conventions:
- `camelCase`, verb-first: `getAll`, `create`, not `all` or `creation`.
- Don't repeat the entity/service name in the method — it's already namespaced. `transactions.service.getAll()`, not `.getAllTransactions()`.
- Boolean-returning methods follow the general `is`/`has`/`can` prefix rule.

## Event Names

Two distinct kinds. Do not mix them.

### Domain Event — `pluginname:eventname`
- Announces something that **already happened**. Written in the **past tense**.
- `pluginname` is the plugin's name **without the vendor segment**.
- Correct: `saved`, `deleted`, `updated`, `ready`. Incorrect: `save`, `delete`, `update`.
- Exception: events published by the Core always use the literal prefix `core:` (e.g. `core:plugin-failed`, `core:plugin-crashed`, `core:ready`, `core:ui-failed`, `core:startup-failed`, `core:service-unavailable`), regardless of which plugin's failure triggered them.

### UI Action Event — `ui:action-name`
- Always starts with the literal prefix `ui:`.
- Describes a lightweight UI-only state change — never a core product operation (saving, deleting, etc. always go through a direct service call, never this event type).
- Uses a noun/state form ending in `-changed`, `-opened`, or similar. Example: `ui:filter-changed`, `ui:view-mode-changed`.

### Data carried by any event
- Identifying information only (e.g. an id the listener can use to call the relevant service).
- Never: full entity payloads, lists of entities, passwords, tokens, or personal data.

## General Naming Rules

- **English only.**
- **Lowercase + hyphens** for multi-word segments (e.g. `budget-reports`, not `budgetReports` or `budget_reports`).
- **No abbreviations.** `transactions`, not `txn`.
- **Descriptive, not clever.** A name says what the thing is, not how it works internally.
- **Case-sensitivity matters** — names are compared as exact strings by the Core (uniqueness checks, Registry lookups, event routing).