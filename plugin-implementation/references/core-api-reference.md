# Core API Reference (Pey Core, 0.11 group-based startup)

Scope: this file states only the startup mechanics that the `core/` docs do not spell out. Everything else lives there — read the matching core file first, not a summary here. Architecture-level rules follow the Bonyan architecture docs (the primary reference); the `core/` files below govern only Core-interaction mechanics. Where this file and the `core/` files disagree on Core behavior, the `core/` files win; report the contradiction per the Architecture Feedback Clause instead of picking one silently.

## Start With the Core Docs

| You need | Read this, not below |
|---|---|
| Entry function signatures, contexts, lifecycle order, host contract | `core/api.md` |
| Adapter selection and injection | `core/adapters.md` |
| Manifest validation, UI manifest rules | `core/manifest-schema.md` |
| Failure codes | `core/error-codes.md` |
| Core and lifecycle events | `core/events.md` |
| Worked prepare/activate examples, hooks, config | `core/plugin-authoring.md` |
| Architecture concepts (services, dependencies, Event Bus, errors) | `plugin-fundamentals.md` |

## Mechanics Specified Here

### Prepare Returns an Array

`prepare()` must return an array (even `[]` if no services), each entry `{ name, object }` with `name` matching a `provides` entry. Anything else fails preparation.

### Activate Receives Only Services

`activate()` receives only the `services` object — no config, events, or hooks. Capture `prepareContext` in a closure or module-level variable during `prepare()` if you need it later (e.g. to publish an event from `activate()`).

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
