# Service Contract (Caller's-Eye View)

Self-contained summary of what a "service" is and how calling one works, from the perspective of a component or the UI Entry Point calling it — not from the perspective of a plugin implementing one.

## What a Service Is

A service is a capability one plugin provides that others can call directly, expecting a response — the only formal request-response mechanism in Bonyan (as opposed to the Event Bus, which is one-way and expects no response). Once the Core has bound a reference at startup, the caller holds a direct reference and calls it like any object method; the Core plays no role in the call itself.

## The Primary Path Rule

For reading data or sending a user request, the primary path is always a direct service call. The Event Bus is only for coordination/notification after an operation completes (a plugin publishing a domain event once its own service call succeeds) — never the mechanism used to execute the operation itself. This is why `forms-and-i18n.md` says a form's submit handler calls a service method directly, and why a UI action event is never a substitute for that call.

## What a Component Actually Holds

A component never asks a registry or the Core for a service. It only ever holds whatever direct reference the UI Entry Point decided to hand it (see `core-boundaries.md`). This reference behaves like a plain object with methods — call it, get a return value or a thrown/returned standard error (see `error-structure.md`), nothing more exotic.

**A component must never stash a service reference in a global, a module-level variable, or any other singleton-like location "for convenience."** The only correct place a component holds a service reference is as a private instance field set in `onConnect(refs)` from the `refs` the base class handed it (see `component-basics.md`). Storing it globally would let a component reach it after that component's own reference-passing step was bypassed, silently breaking the ownership boundary in `core-boundaries.md` even though the call would still technically work — this is exactly the pattern `forbidden-patterns.md` rules out.

## Service Boundary (One Service vs. Several)

If several methods relate to the same entity/concern, they're one service (e.g. `getAll`/`getById`/`create`/`update`/`delete` on `myapp.tasks.service`). If a capability can be used independently, it's a separate service. This matters for the UI mainly when deciding what to ask the Entry Point to add to the UI manifest's `dependencies` (see `ui-manifest.md`) — ask for the specific service that matches the concern, not a broader one that happens to also contain it.

## Optional vs. Required (UI Manifest Angle)

A service the UI cannot function without belongs in `dependencies.required`. A service that only adds capability if present (e.g. a logging service used by the component-error boundary in `component-basics.md`) belongs in `dependencies.optional` — code must degrade gracefully, not assume it's there.

## Stale References After Deactivation

If a service's providing plugin is deactivated at runtime, the Core publishes `core:service-unavailable` and removes it from the Registry — but a caller that already holds a direct reference (as the UI does, once handed one) still holds that same reference. Calling it after this point must not produce undefined behavior: a well-behaved service detects its own `unavailable` state and returns a standard Bonyan error (see `error-structure.md`) instead of doing something undefined. From the UI side, this means: always handle the possibility that a service call returns an error, even for services that are normally always available.
