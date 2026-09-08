# Manifest Schema (Bonyan)

Defines every field of `manifest.json`. For a ready-to-fill JSON skeleton, use `manifest-template.json` alongside this file. This file explains *what each field means and why*; the template is *what to type*.

The manifest is the only source of truth about a plugin. Anything a plugin does that isn't declared here does not architecturally exist — never implement an undeclared capability, even if it seems convenient.

## 1. Identity

| Field | Meaning |
|---|---|
| `name` | Unique, `vendor.name` format. No two plugins in one system may share it — checked during manifest loading, before declaration validation; a duplicate fails with `DUPLICATE_PLUGIN_NAME` (fatal if the plugin is infrastructure, per the usual rule). |
| `version` | The plugin's own version, `MAJOR.MINOR.PATCH`. |
| `architecture` | The Bonyan architecture version this plugin targets, `MAJOR.MINOR`. Core compares this to its own version; incompatible → plugin does not start. |
| `description` | Short, for humans only — the Core does not read it. |

## 2. Type

One of five values: `infrastructure`, `product`, `feature`, `platform`, `transport`. Determines which `plugin-types/*.md` file applies — it does not change Core behavior except for the `adapterType` rule below.

**`adapterType`** (optional field, infrastructure only): declares which adapter type from `bootstrap.json` → `adapters` this plugin is responsible for (e.g. `storage`, `auth`). Only legal when `"type": "infrastructure"` — any other type declaring it fails validation. Selection forms and edge cases (duplicates, missing or malformed selection, orphaned entries) are specified in `core/adapters.md`; failure codes in `core/error-codes.md`.

## 3. Dependencies

```
dependencies: { required: [...service names], optional: [...service names] }
```

- **required:** the plugin cannot function without it; unavailable → the plugin is deactivated at startup, or must report a critical error if it disappears at runtime.
- **optional:** used if present; absence means reduced capability only, nothing to report.

A plugin may only call a service name that appears here. This list is exhaustive — no undeclared service is ever reachable, at startup or later (see `plugin-fundamentals.md` → Security Baseline).

**Before declaring a `required` dependency, check the target plugin's type.** If the service belongs to a `feature`-type plugin, a `required` dependency on it is in tension with Feature's own definition (the application is supposed to work without it) — this isn't explicitly forbidden anywhere in the architecture, but it's a real inconsistency worth surfacing. Stop and report it (see "Requirement vs. Architecture Conflict" in `plugin-implementation/SKILL.md`) rather than declaring it silently.

## 4. Provided Services (`provides`)

A list of services this plugin registers. Every entry needs:

| Field | Meaning |
|---|---|
| `name` | The service's registered name, `vendor.name.service` format. |
| `description` | What the service does. |
| `methods` | Every method the service exposes. Mandatory even though the Core doesn't enforce it — this is the contract a human or another model reads instead of the source code. |

Each method in `methods` needs: `name`, `description`, `params` (name + type per param), `returns` (return type), `errors` (possible error codes). If a service is only used internally and never called by another plugin, do not register it here — only declare what other plugins may actually call. (The Core treats `methods` as documentation-only and constrains no entry shape — see `core/manifest-schema.md` — so this object form is a skill-level contract rule for readability, not a Core validation requirement.)

Leave this array empty if the plugin provides no services.

## 5. Published Events (`events`)

A list of domain events this plugin may publish. Every entry needs:

| Field | Meaning |
|---|---|
| `name` | `pluginname:eventname` format (see `naming-cheatsheet.md`), past tense. |
| `description` | When this fires. |
| `data` | The identifying fields carried with it (name + type each). Empty if the event carries no data. |

An event must be declared here before the plugin may publish it. Listening to any event, by contrast, needs no declaration at all — that asymmetry is intentional (see `plugin-fundamentals.md`).

Leave this array empty if the plugin publishes no events.

## 6. Default Configuration (`config`)

Default values for anything this plugin lets the developer configure (e.g. a timeout, a page size). The project's `bootstrap.json` → `config` section overrides these at startup; project values always win. Leave empty if the plugin is not configurable.

Values containing `${ENV_VAR_NAME}` placeholders are resolved by the Core using EnvSource before this plugin receives its final config — the plugin never reads environment variables itself.

## Manifest Rules

- **One manifest per plugin**, named exactly `manifest.json`, at the plugin's root.
- **Fixed at runtime.** The manifest is read once at startup. A running plugin cannot add a new dependency, a new provided service, or a new event without a restart.
- **No hidden dependencies.** If the plugin needs a service, it must be listed in `dependencies` — needing something not listed is an architecture violation, not a minor omission.
- **Every version bump must be reflected here.** A changed plugin with an unchanged `version` cannot be trusted. See `versioning.md` for the MAJOR/MINOR/PATCH rules; applied to manifest changes specifically: adding a service, method, event, or optional dependency → MINOR. Removing or changing the signature of an existing service/method/event, or adding a new *required* dependency → MAJOR (it can break something that depends on this plugin). A fix with no contract change → PATCH.
- **Array order in `provides`, `events`, and `dependencies` carries no architectural meaning.** Elements are identified by their `name` field, not by position — reordering them is not a contract change and needs no version bump.
- **`provides` and `events` must be present as arrays even when empty (`[]`)** — the Core requires the keys to exist. `config` is not required at all and may be omitted entirely if the plugin has no configuration. Unknown/extra top-level fields are tolerated but shouldn't be added on purpose — anything not in this file isn't part of the contract.