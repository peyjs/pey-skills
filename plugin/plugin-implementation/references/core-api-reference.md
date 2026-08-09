# Core API Reference (Pey Core, 0.11 group-based startup)

## Entry Point Contract — Two-Phase

`index.js` must export **two** functions: `prepare(prepareContext)` and `activate(services)`. There is no single `setup()` anymore.

```js
export async function prepare(prepareContext) {
  // Assign lifecycle hooks HERE, not in activate() — see the mandatory rule below.
  prepareContext.onShutdown = async () => { /* cleanup */ };
  prepareContext.onDeactivated = () => { /* release resources acquired in prepare() */ };

  savedContext = prepareContext; // still worth keeping, e.g. to publish events later in activate()
  // Build service instances using prepareContext.config — no other plugin's
  // service is available yet. Must not call another application-group plugin.
  return [
    { name: 'vendor.this-plugin.service', object: theServiceInstance },
  ];
}
let savedContext; // for anything else you need from activate(), e.g. prepareContext.events

export async function activate(services) {
  // services is a plain object: { "vendor.other.service": <direct ref>, ... }
  // Wire dependencies into whatever prepare() built. Do NOT assign onShutdown/
  // onDeactivated here — see the mandatory rule below.
}
```

- **`prepare()` must return an array**, even if empty. Returning non-array throws `prepare() must return an array of service instances` and fails preparation.
- Each returned entry is `{ name, object }` — `name` must match a `provides` entry, or that specific registration is rejected (`SERVICE_NOT_DECLARED_IN_MANIFEST`), which fails this plugin's preparation.
- **`activate()` does not receive `config`, `events`, or the lifecycle hooks again** — only `services`. If your plugin needs those in `activate()` (e.g. to publish an event), capture the `prepareContext` object from `prepare()` in a closure/module-level variable, as shown above.
- **Mandatory: assign `onShutdown` and `onDeactivated` in `prepare()`, never in `activate()`.** `activate()` is only called for plugins that survive settlement and group commit. A plugin that is cascaded during settlement, fails during activation, or is skipped by an infrastructure-group short-circuit **never has `activate()` called at all** — hooks assigned there are simply lost, and resources acquired during `prepare()` (e.g. an open database connection) would leak. The Core calls `onDeactivated` (if assigned) for any plugin that successfully prepared, even if it never reached `activate()`, specifically so preparation-time resources can be released.
- Both phases are invoked on the same loaded entry module. `moduleLoader.loadEntry()` is called once per plugin, after declaration validation passes for the whole batch of manifests (not just this one).

## PrepareContext (what `prepare(prepareContext)` receives)

```js
{
  config: { /* final merged config, including adapter injection if applicable */ },
  events: { publish(eventType, data), subscribe(eventType, listener) },
  onShutdown: null,     // assign your own function here, in prepare() only
  onDeactivated: null,  // assign your own function here, in prepare() only
  reportCriticalError,  // call this; already bound to your plugin
}
```

No `services` field exists here — that's the entire point of the two-phase split. During `prepare()`, calling another application-group plugin's service is not possible (you have no reference to it yet) and not allowed even conceptually.

- **`events.publish`/`events.subscribe`** — same behavior as before: publish requires the event declared in `manifest.events`, source is auto-injected and unforgeable; subscribe accepts `"*"` as a wildcard (still a Pey Core implementation detail, not a Bonyan architecture guarantee).
- **`reportCriticalError`** — same shape as before: you build the complete `{code, message, source, type, timestamp, detail}` object yourself; the Core does not auto-enrich it here either (this hasn't changed). Effect is the same: `INACTIVE`, `onDeactivated` called, services deregistered (`core:service-unavailable` per service), `core:plugin-crashed` published.

## The `services` Argument to `activate(services)`

A plain object, exactly like the old `context.services`: keyed by exact service name string. Resolution order per dependency: checked first in the **private startup registry** (same group, enables required-dependency cycles within a group), then the **public Registry** (services already committed by a previously processed group — i.e. infrastructure services, when activating an application-group plugin). Required dependencies are guaranteed present here (settlement already verified it before `activate()` is ever called). Optional dependencies are present only if actually available.

Accessing an undeclared key is `undefined` — same as before, and still the only thing a plugin author actually observes for an undeclared-access attempt. The internal `SERVICE_NOT_DECLARED_IN_DEPENDENCIES` code still exists only inside the Registry, unreachable from plugin code directly.

## Two-Phase Group Startup — What Actually Happens, in Order

1. **Preparation** — every plugin in the current group (infrastructure first, then application) runs `prepare()`. An application-group plugin must not call another application-group plugin's service during this phase (it has none). Prepared services go into the **private startup registry**, invisible outside the Core.
2. **Settlement** — the Core checks every prepared plugin's `required` dependencies are actually available (in the startup registry for this group, or the public Registry from an earlier group). Any plugin whose required dependency isn't available is deactivated automatically, with reason `REQUIRED_DEPENDENCY_UNAVAILABLE` — and this **cascades**: anything that required a just-deactivated plugin's service is deactivated too, repeating until stable. **This is fully automatic and requires no code from you** — it only happens during this pre-activation settlement, not after plugins are running. **If your `onDeactivated` was assigned during `prepare()`, the Core calls it here too** — for any plugin that successfully prepared but is being deactivated before ever reaching `activate()` (cascaded at settlement, failed during activation, or skipped by an infrastructure-group short-circuit). This is exactly why hooks must be assigned in `prepare()`, not `activate()` — see below.
3. **Binding + Local Activation** — surviving plugins get `activate(services)` called, in configuration order. An application-group plugin still must not call another application-group plugin's service during this phase either (only after group commit).
4. **Group Commit** — prepared services are promoted from the private startup registry to the public Registry, plugins are marked `ACTIVE`, and `pluginname:ready` is published in configuration order — only now, for this group, as a whole.

**Two groups exist:** `infrastructure` (processed first) and `application` (Product + Feature + Platform + Transport, processed second, can see infrastructure's already-committed public services in addition to its own group's startup registry). A required-dependency cycle is only resolvable **within** one group (via the private startup registry) — a required cycle spanning across the infrastructure/application boundary is not supported and will simply fail settlement for at least one side.

**Infrastructure group failure at any phase is fatal** — stops the entire system startup (`core:startup-failed`, no `core:ready`). **Application group failures are not fatal** — the failing plugin (and its cascade of dependents) is deactivated; the rest of the group and the system continue.

## After Group Commit — Runtime Behavior Is Unchanged From Before

Once a plugin is `ACTIVE`, there is still **no automatic mechanism** that deactivates a dependent when its required service disappears later (e.g. because that service's plugin crashed via `reportCriticalError`). The Core only broadcasts `core:service-unavailable`. You must still subscribe to it yourself and self-report:

```js
savedContext.events.subscribe('core:service-unavailable', (event) => {
  if (REQUIRED_SERVICE_NAMES.includes(event.data.serviceName)) {
    savedContext.reportCriticalError({
      code: 'REQUIRED_DEPENDENCY_LOST',
      message: `Required service ${event.data.serviceName} is no longer available`,
      source: manifestName,
      type: 'critical',
      timestamp: new Date().toISOString(),
    });
  }
});
```

The automatic cascading described above only ever happens **before** a group commits — never afterward.

## Adapter Delivery

`context.config.adapter` is injected only for an infrastructure plugin that declares `adapterType`, based on `bootstrap.json` → `adapters[thatType]`. That configured value can take **two forms**:

**Simple form** — a plain string, the implementation identifier:
```js
// bootstrap.json: "adapters": { "storage": "IndexedDBAdapter" }
config.adapter // === "IndexedDBAdapter"
```

**Extended form** — an object with a required `implementation` field plus optional adapter-specific config:
```js
// bootstrap.json: "adapters": { "storage": { "implementation": "IndexedDBAdapter", "dbName": "my-db" } }
config.adapter        // === "IndexedDBAdapter"
config.adapterConfig  // === { "dbName": "my-db" }
```

Your plugin must handle both shapes being possible — check `config.adapter` for the identifier either way, and `config.adapterConfig` for any extra fields (absent entirely in the simple form). If the configured selection is an object *without* an `implementation` field, or is neither a string nor an object, this plugin's preparation fails with `ADAPTER_SELECTION_INVALID` before `prepare()` is even reached for that specific failure — but you still shouldn't assume the shape defensively-unchecked, since a malformed selection could theoretically vary by how the project's `bootstrap.json` is authored.

If `adapterType` is declared, `config.adapter` is guaranteed to be present by the time `prepare()` runs — `ADAPTER_NOT_CONFIGURED` (no matching adapter at all) is caught during declaration validation, before any entry point loads.

The architecture doesn't prescribe where the adapter's code physically lives — inside this plugin or a separate package it imports. Loading and instantiating it is this plugin's job either way.

## Manifest Validation — Unchanged Structural Rules

Required top-level fields: `name`, `version`, `architecture`, `description`, `type`, `dependencies`, `provides`, `events` — same as before, same `vendor.name` regex (`/^[a-z0-9-]+\.[a-z0-9-]+$/`), same five `type` values, same tolerance for unknown extra fields, same non-requirement of `config`.

**Two-stage validation, now explicit in the code:**
- **Stage 1 — declarations, across *all* manifests at once**, before any entry point loads: architecture compatibility, every `provides` service name has exactly one provider system-wide (not just per-plugin — a real duplicate-service check across the whole batch), every `required` dependency has *some* declared provider somewhere, and all `adapterType` rules.
- **Stage 2 — settlement**, per group, after preparation: actual availability, as described above.

**UI manifest validation now also rejects `adapterType`**, in addition to `type` and `provides` — if any of these three fields is present on a UI manifest, `INVALID_UI_MANIFEST`.

## Exact Error / Failure Codes

| Code | Meaning |
|---|---|
| `INCOMPATIBLE_ARCHITECTURE` | Declaration stage — architecture version mismatch |
| `DUPLICATE_PLUGIN_NAME` | Manifest-loading stage (before declaration validation) — a later plugin reused an already-loaded plugin's `name` |
| `DUPLICATE_SERVICE_PROVIDER` | Declaration stage — two manifests declare the same service name (system-fatal) |
| `MISSING_REQUIRED_DEPENDENCY_DECLARATION` | Declaration stage — no manifest anywhere declares a required service |
| `ADAPTER_TYPE_NOT_ALLOWED` | Declaration stage — non-infrastructure plugin declares `adapterType` |
| `DUPLICATE_ADAPTER_TYPE` | Declaration stage — two infrastructure plugins declare the same `adapterType` (system-fatal) |
| `ADAPTER_NOT_CONFIGURED` | Declaration stage — `adapterType` declared but not in `bootstrap.json` → `adapters` |
| `ADAPTER_SELECTION_INVALID` | Preparation phase — the configured adapter selection is malformed (not a string, or an object missing `implementation`) |
| `ENTRY_POINT_FAILED` | Preparation phase — `moduleLoader.loadEntry()` itself threw |
| `PREPARATION_FAILED` | Preparation phase — `prepare()` threw, or didn't return an array, or returned an undeclared/duplicate service name |
| `REQUIRED_DEPENDENCY_UNAVAILABLE` | Settlement phase — automatic, cascading, pre-commit only |
| `ACTIVATION_FAILED` | Binding+activation phase — `activate()` threw |
| `SERVICE_REGISTRATION_FAILED` | Commit phase — promoting a prepared service to the public Registry failed |
| `INVALID_UI_MANIFEST` | UI manifest has `type`, `provides`, or `adapterType` |
| `SERVICE_NOT_DECLARED_IN_MANIFEST` | Registry-level — tried to register a service not in this plugin's own `provides` |
| `SERVICE_NOT_DECLARED_IN_DEPENDENCIES` | Registry-level — internal only, not directly reachable from plugin code |
| `EVENT_NOT_DECLARED_IN_MANIFEST` | Tried to publish an event type not in this plugin's own `events` |

## Runtime Host Contract

Unchanged: `configSource.getConfig()`, `envSource.get(varName)`, `moduleLoader.loadManifest(ref)` + `moduleLoader.loadEntry(ref)`. Missing/malformed → `INVALID_HOST_SOURCE` at construction.

## Plugin Name Uniqueness (fixed)

Checked during manifest loading, before declaration validation: if a manifest's `name` matches one already loaded, the later entry fails with `DUPLICATE_PLUGIN_NAME` — fatal if that plugin is infrastructure (stops the whole system, same as any other fatal declaration failure), otherwise that specific plugin is skipped and the rest of loading continues. The first-loaded plugin with a given name keeps running; a later duplicate never gets its entry point loaded.