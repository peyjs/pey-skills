# Decision Guide

Check this first whenever it's unclear where a piece of data, a value, or a reaction belongs. Walk the questions in order — stop at the first "yes."

## Where does this data live?

1. **Is it business/product data** (an entity, a value another part of the system cares about, anything that would need to survive a page refresh or matter to more than presentation)?
   → It belongs to a **service**. Fetch it when needed; never cache it in a component. See `component-basics.md` (Local State) and `architecture-context/service-contract.md`.

2. **Is it purely visual/presentational, and scoped to one component** (a dropdown's open/closed state, which tab looks active, a tooltip's visibility)?
   → **Component-local state.** See `component-basics.md` (Local State).

3. **Is it purely visual/presentational, but shared across components that aren't parent/child** (theme, a sidebar's collapsed state affecting multiple unrelated components)?
   → **UI-local shared state store.** See `shared-state.md`.

4. **Is it a configuration value or feature flag** (a default page size, a feature toggle, an API base URL)?
   → **From the Entry Point, in `refs`** — the component receives it in `onConnect(refs)`, never from a service call and never from the shared state store. Configuration is read once by the Entry Point from `context.config` and handed down explicitly to whatever component needs it, the same way a service reference is handed down. See `architecture-context/ui-manifest.md` (UI Configuration).

## How should this reaction happen?

5. **Does the caller need a response** (data back, confirmation an operation succeeded, a value to use next)?
   → **Direct service call.** This is the primary path — see `architecture-context/service-contract.md`.

6. **Is this just letting other parts know something already happened, with no response expected** — and is it a **business-level fact** (a transaction was saved, an account was updated)?
   → That's a **domain event**, published by the *plugin* whose service handled the operation, not by the UI. A component only *listens* — see `component-basics.md` and `architecture-context/naming-conventions.md`.

7. **Is this just letting other parts know something already happened, with no response expected** — and is it a **lightweight UI-only fact** (a filter changed, a panel opened, a component's own render/handler failed)?
   → **`ui:*` event on the Event Bus**, declared in the UI manifest. See `architecture-context/ui-manifest.md`.

## Quick Cross-Check

If the answer to "does the caller need a response" is yes, it is never an event of any kind — always a direct service call, even if it feels like it "announces" something. Events (domain or `ui:*`) are only for the no-response-needed case.

If in doubt between component-local state (#2) and the shared store (#3), ask: would a sibling component, unrelated in the DOM tree, ever need to know this changed? If no, keep it local (#2) — don't reach for the shared store by default.
