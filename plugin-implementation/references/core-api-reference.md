# Core API Reference (Pey Core, 0.11 group-based startup)

Scope: this file states only the startup mechanics that the `core/` docs do not spell out. Everything else lives there — read the matching core file first, not a summary here. The authoritative Core sources are `core/api.md`, `core/error-codes.md`, and `core/events.md` — where this file and those disagree on Core behavior, the `core/` files win; report the contradiction per the Architecture Feedback Clause instead of picking one silently.

## Start With the Core Docs

| You need | Read this, not below |
|---|---|
| Entry function signatures, contexts, lifecycle order, host contract | `core/api.md` |
| Adapter selection forms and injection | `core/adapters.md` |
| Manifest validation, UI manifest rules | `core/manifest-schema.md` |
| Failure codes | `core/error-codes.md` |
| Core and lifecycle events | `core/events.md` |
| Worked prepare/activate examples, hooks, config | `core/plugin-authoring.md` |
| Architecture concepts (services, dependencies, Event Bus, errors) | `plugin-fundamentals.md` |

## Mechanics Specified Here

### Entry Loading

`moduleLoader.loadEntry()` is called once per plugin, and only after declaration validation passes for the whole batch of manifests — never for a plugin whose manifest failed validation.

### Prepare Returns an Array

`prepare()` must return an array (even `[]` if no services), each entry `{ name, object }` with `name` matching a `provides` entry. Anything else fails preparation.

### Activate Receives Only Services

`activate()` receives only the `services` object — no config, events, or hooks. Capture `prepareContext` in a closure or module-level variable during `prepare()` if you need it later (e.g. to publish an event from `activate()`).

### Settlement Cascades Automatically — Once

Before group commit, a missing required dependency deactivates dependents automatically with `REQUIRED_DEPENDENCY_UNAVAILABLE`, cascading to their own dependents — no plugin code needed. This never happens after commit. Stage 1 additionally requires every provided service name to have exactly one provider system-wide.

### Duplicate Plugin Names

A later duplicate name is skipped (system-fatal if that plugin is infrastructure); the first-loaded plugin keeps running.

### Runtime Dependency Loss

Once active, a disappearing required service is the plugin's own job to detect — rule in `plugin-fundamentals.md` (Dependencies). The subscription pattern:

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

`REQUIRED_DEPENDENCY_LOST` is an author-defined code, not a Core code.

### Failure Codes Beyond the Catalog

`core/error-codes.md` is authoritative; the codes below are not listed there. `ENTRY_POINT_FAILED` here means specifically `loadEntry()` itself throwing (preparation/activation throws are the next two rows).

| Code | Meaning |
|---|---|
| `ADAPTER_SELECTION_INVALID` | Preparation phase — the configured adapter selection is malformed (not a string, or an object missing `implementation`) |
| `PREPARATION_FAILED` | Preparation phase — `prepare()` threw, or didn't return an array, or returned an undeclared/duplicate service name |
| `ACTIVATION_FAILED` | Binding+activation phase — `activate()` threw |
| `DUPLICATE_SERVICE_PROVIDER` | Declaration stage — two manifests declare the same service name (system-fatal) |
