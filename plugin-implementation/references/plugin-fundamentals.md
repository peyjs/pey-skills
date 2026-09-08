# Plugin Fundamentals (Bonyan)

Core concepts every plugin implementation must respect. Manifest structure lives in `manifest-schema.md`. Type-specific rules live in `plugin-types/*.md`. This file is the shared baseline underneath both.

## What Is a Plugin

An independent unit with exactly one manifest and one entry point. It never knows another plugin directly — it asks the Registry for a service address and calls that service directly. It has no hidden dependencies: everything it needs is declared in its manifest, which is why a correctly-built plugin can move to another project unchanged.

## Entry Point Contract

Startup itself has two architectural phases, not one, and the entry point receives different things in each:
- **Preparation phase:** the plugin's own final merged configuration, and the ability to publish/listen on the Event Bus. No services yet.
- **Binding + local activation phase:** the services declared in `dependencies` (already resolved to direct references). No config/events/hooks handed over again here — see below for why that matters.

The plugin returns to the Core, at the end of preparation, the services it declared in `provides`, for registration.

**Group mechanics** (private startup registry, settlement, group commit, `pluginname:ready`) are Core behavior — see `core/api.md` (Behavior) and `core/plugin-authoring.md` (Entry Point). Note in particular: whatever a plugin needs to remember from the preparation phase (like its lifecycle hooks) has to be captured by the plugin itself, since the phases don't share an implicit context automatically. The exact exported function names and signatures for these two phases are a contract detail — see `core/api.md` and `core/plugin-authoring.md`, not this file. Note in particular: whatever a plugin needs to remember from the preparation phase (like its lifecycle hooks) has to be captured by the plugin itself, since the phases don't share an implicit context automatically.

This entry point is a two-way contract for the plugin's entire life, not just startup:
- The Core can tell the plugin to release resources at shutdown.
- The Core can tell the plugin it has been deactivated (e.g. a dependency failed).
- The plugin uses this same path to report a critical error to the Core.

**Never self-deactivate.** A plugin must never remove its own services or consider itself inactive on its own — it reports the situation to the Core (as a critical error) and lets the Core run the formal deactivation cycle.

**After deactivation notice:** the instant the Core notifies a plugin it's deactivated, that plugin is inactive from the architecture's perspective — regardless of how long actual cleanup takes. From that moment, every service it provides must detect this and return a standard Bonyan error instead of executing, rather than behaving unpredictably. Other plugins may still hold a direct reference to a now-stale service (the Core handed over the object itself, not just an address) — this is exactly why the service itself, not the caller, must guard against being called while unavailable.

## What Is a Service

A service is a capability one plugin provides that others call directly and expect a response from. It is the only formal request/response mechanism between plugins.

- **Registration:** only services declared in `provides` may be registered. The Registry stores the address; it is a discovery point only — it never mediates the actual call.
- **Access:** only services declared in `dependencies` may be called. The Core checks this once, at startup, and hands over direct references. After that, calls are direct and the Core is not involved.
- **Deregistration:** when a plugin is deactivated, its services are removed from the Registry and the Core publishes `core:service-unavailable` (with the service name and the providing plugin's name) so dependents can react.

**One service or several?** If methods belong to the same entity, they are one service (e.g. `getAll`/`create`/`update`/`delete` on tasks → one `tasks.service`). If a capability can be used independently of the rest, split it into its own service (e.g. a reminder system inside the same plugin → a separate `reminders.service`).

## Dependencies: Required vs Optional

- **Required:** the plugin cannot function without it. Unavailable at startup → the plugin is deactivated. Becomes unavailable at runtime → the plugin must report this to the Core as a critical error (it may not just degrade silently).
- **Optional:** the plugin uses it if present. Unavailable at startup or runtime → the plugin continues with reduced capability, no report needed.

**A required-dependency cycle between two plugins is fine, as long as both are in the same startup group** (both infrastructure, or both in the application group — Product/Feature/Platform/Transport together). This works because of the two-phase split: both plugins finish preparing (into the private startup registry) before either one needs the other's actual reference, so binding can resolve both directions. A required cycle spanning *across* the infrastructure/application boundary is not supported.

**Detecting a required dependency going unavailable at runtime — after your plugin is already active — is the plugin's own job; nothing does it automatically.** (This is different from startup: if a required dependency fails to become available *during* the startup group settlement, the Core itself cascades deactivation automatically, with reason `REQUIRED_DEPENDENCY_UNAVAILABLE` — no plugin action needed there.) But once a plugin is active, the Core only broadcasts `core:service-unavailable`; it does not track who depends on what and does not deactivate dependents for you after the fact. Subscribe to `core:service-unavailable` (during preparation, since that's when Event Bus access is available) and check `event.data.serviceName` against your own required dependencies; if it matches, call `reportCriticalError` yourself. See `core/plugin-authoring.md` (Lifecycle Hooks) and `core/api.md` for the exact call shapes and function names.

A plugin may only call what it declared in `dependencies` and only register what it declared in `provides`. Nothing else is reachable, by design — this is the security boundary, not just a convention (see Security Baseline below).

## Event Bus

Used only to announce that something already happened. The publisher does not wait for and does not know about listeners.

**Service vs Event Bus — the deciding question:** does the caller need a response? Yes → service call. No, just a notification → Event Bus. The primary path for actually executing an operation (save, delete, etc.) is always a direct service call; the Event Bus is for coordination/notification afterward, never for triggering the operation itself.

**What a plugin sends when publishing:**
```json
{ "type": "transactions:saved", "data": { "transactionId": "tx-456" } }
```
The Core adds `id`, `source` (from the manifest name — unforgeable), and `timestamp` before delivery — see `core/events.md` (Event Structure). `type` and `data` are the plugin's responsibility.

**`data` field rule:** identifying information only (ids the listener can use to call a service for the full picture). Never: full entity content, lists of entities, passwords, tokens, or personal data.

**Publishing** requires the event to be declared in the plugin's manifest (see `manifest-schema.md`). **Listening** requires no declaration at all — any plugin may listen to any event freely.

No order is guaranteed between multiple listeners of the same event — never rely on one listener running before another.

## Standard Error Structure

Every error uses the PeyError structure (`code`, `message`, `source`, `type`, `timestamp`, optional `detail`) with one of three types (`startup`, `operational`, `critical`) — fully specified in `core/error-codes.md` (Error Structure). What the catalog does not state, and what matters at authoring time, are the rules below:

### Rules that apply everywhere
- No raw, unstructured error ever crosses a plugin's (or adapter's) boundary — always convert to the structure above first.
- Never ignore an error silently; never hide it — log/announce it as the type dictates.
- No infinite retries — if retrying, use a fixed, configurable count defined inside the part itself.
- Structured error handling does not replace tests; they complement each other.

## Security Baseline

**The golden rule: anything not declared is forbidden.** Enforced by the Core, once, at startup — it validates the manifest, resolves only the declared `dependencies` to direct references, and rejects (and logs) any attempt to reach an undeclared service. There is no runtime access-control step after that; the boundary exists because the plugin simply never received a reference to anything it didn't declare.

- Declare the minimum you actually need — extra declared access is a real weakness, not a convenience.
- Sensitive data (passwords, tokens, encryption keys, personal data) moves **only** through a direct, declared service call — never through the Event Bus, never through an error structure (not even `detail`).
- Environment values resolved from `${ENV_VAR_NAME}` placeholders must never end up in a log, an error, or an event, and should be held only as long as needed to deliver them.