# UI-Local Shared State

Read this before reading or writing any state that needs to be shared across components that are not in a parent-child relationship.

## When to Use This

Some presentational state is shared across components that aren't in a parent-child relationship (e.g. a global theme toggle, a collapsed/expanded sidebar affecting multiple unrelated components). Routing every one of these through the Event Bus — meant for domain-level announcements — adds unnecessary weight to the logic layer.

For this narrow case only, the UI layer keeps one small, UI-local shared state store, owned and created by the Entry Point and handed down to components that need it, the same way a service reference is handed down.

## Fixed API

- **`get(key)`** — returns the current value for `key`, or `undefined` if unset.
- **`set(key, value)`** — sets the value for `key` and synchronously notifies subscribers of that key, **only if the new value is not shallow-equal to the current one** (`Object.is` for primitives; reference equality for objects/arrays — the store does not deep-compare). Same primitive value again → no notification. New reference for an object/array, even with identical contents → notification does fire, since the store cannot know two different references are "equal" without a deep comparison it deliberately does not perform. `value` must be a plain, JSON-serializable value (string, number, boolean, plain object/array) — never a DOM reference, a service reference, or a function.
- **`subscribe(key, callback)`** — registers `callback(newValue)` to be called whenever `set(key, ...)` runs (and passes the equality check above); returns an `unsubscribe` function. A component calls `unsubscribe` in `disconnectedCallback`, the same discipline as Event Bus subscriptions.

## Immutability Requirement

**Values stored in this store must be treated as immutable.** Because equality is by reference, a component must never mutate an object/array it got from `get()` in place and expect the change to be noticed or propagated — it must always call `set(key, newValue)` with a new object/array. Mutating in place silently violates the equality check above and leaves other subscribers unaware of the change.

## Rules

- Holds **only UI presentation state** (theme, layout toggles, panel open/closed) — never product/business data, which always belongs to a service.
- Not a replacement for the Event Bus or a service call; product plugins never read from or write to it.
- Exactly one instance per running UI, created once by the Entry Point; a component never creates its own instance.
