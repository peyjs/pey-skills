# UI-Local Shared State

Read this before reading or writing any state that needs to be shared across components that are not in a parent-child relationship.

## When to Use This

For UI presentation state shared across unrelated components (theme toggle, collapsed/expanded sidebar) — routing every one of these through the Event Bus adds unnecessary weight. Product/business data never lives here; it always belongs to a service.

## The Store

One instance per running UI, created by the Entry Point and handed down like a service reference; a component never creates its own.

- **Immutability requirement:** stored values must be treated as immutable — equality is by reference, so always `set(key, newValue)` with a new object/array; mutating a value from `get()` in place silently leaves other subscribers unaware.

## Contract (Implemented By The Kit)

The kit's `createSharedState()` (`pey.webui/shared-state`) implements the API and its contract specifics — `get`/`set`/`subscribe`/`clear`, validation with stable `PEY_SHARED_STATE_*` codes, listener isolation, idempotent unsubscribe, and the depth limit. See `pey.webui/docs/public-api.md`, section **`createSharedState()`**, and `pey.webui/docs/decisions.md` (Shared State Decisions).