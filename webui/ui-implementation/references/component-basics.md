# Component Basics

Read this before writing or editing any individual UI component's class.

## Custom Elements + Shadow DOM

Every component is a Web Component: a class extending `HTMLElement`, registered with `customElements.define()`, using Shadow DOM (`attachShadow({ mode: 'open' })`) — mandatory, not optional, even for a trivial component. Purpose: guarantee CSS/DOM isolation between components written independently, at different times, by different sessions. This removes class-name collisions and leaking selectors as a category of bug.

Shadow DOM does not exempt a component from accessibility (see `styling-and-assets.md`). Style/markup isolation and assistive-technology support are handled separately; neither is dropped for the other.

## One Shared Base Class

All components extend a single shared base class (e.g. `BonyanElement`). No component defines its own lifecycle pattern from scratch. The base class alone is responsible for:

- **Receiving references handed down by the Entry Point** (services, config) through **one fixed method the base class defines** (e.g. `connect(refs)`), called by whoever creates the component (the Entry Point, or a parent composing a child) right after creating it and before it's appended to the DOM — a method call, not constructor arguments, since the Custom Elements spec expects a no-argument constructor. **`connect()` is called exactly once per component instance, before it is ever connected to the DOM; it is not a general-purpose update method.** If a component's references need to change after that (e.g. a parent re-renders and produces a new child instance instead of reusing the old one), the fixed pattern is to discard the old instance and create/`connect()` a new one — never to call `connect()` a second time on an already-connected instance expecting it to behave like an update. The base class enforces this by throwing if `connect()` is called on an instance that has already been connected once. The component does no lookup, no injection, no global access — see `architecture-context/core-boundaries.md` for why.
- **Rendering** (see Render Strategy below).
- **Subscribing to Event Bus domain events on connect, unsubscribing on disconnect**, automatically.
- **Catching errors** from its own render and event-handling code (see Error Handling below).
- **Publishing `ui:*` events** through one fixed method.
- Standard lifecycle hooks (prepare / activate / teardown equivalents).

A new component implements only: its template method, the list of references it expects to be handed, and the events it listens/publishes. Everything else is inherited — never reimplemented per component.

## Render Strategy

**Default:** full re-render of the shadow root's content from a template method on every state change. Default because it's the simplest strategy to predict and generate consistently across many independent sessions — not because it's the only one permitted.

**Named exception — input-holding elements:** a component with live text input, text selection, scroll position, or an in-progress animation that must survive a re-render must not blow that state away each time. Use the base class's documented escape hatch: fields/regions marked to be preserved are diffed and left untouched (value, selection range, scroll offset, focus) while the rest of the shadow root is replaced normally. Opt in only for the specific elements that need it — this is not a general alternative rendering engine, and a component must never silently roll its own diffing outside this mechanism.

## Template Contract

Every component implements exactly one method for producing its content (e.g. `render()`), taking no external arguments (reads only `this`'s current state/props). What it returns is "a standard template output" — today an HTML string applied via `innerHTML`, defined at this abstraction level specifically so a future move to `<template>` + `content.cloneNode(true)` doesn't require renegotiating the rule. Fixed regardless of concrete format: exactly one such method per component, and only the base class's render mechanism consumes its output — never bypassed to mutate the shadow root from elsewhere (see `forbidden-patterns.md`).

## Event Binding Rule

**Default:** the base class registers one delegated listener per event type, once, on the shadow root, in `connectedCallback`, removed in `disconnectedCallback`. The handler inspects `event.target` (or a data attribute identifying the action). Default because it guarantees listener count never grows with re-renders.

**Allowed exception — small, static components:** a component with a small, fixed number of interactive elements that doesn't re-render its listener-bearing markup (or rarely does) may attach direct `addEventListener` calls in `connectedCallback` instead, provided it removes them in `disconnectedCallback`. Never allowed, in either approach: attaching a listener inside code that runs on every re-render — that's the actual failure mode being guarded against, not the presence of delegation specifically.

## Local State: What May Live Inside a Component

Only transient, presentational state — no meaning outside the component, not product data. Examples: dropdown open/closed, which tab is visually active, tooltip showing, scroll position.

Never: product/business data (even cached), the result of a business decision, or anything another part of the system needs to know about. That data is always fetched from a service when needed, never duplicated into component-local state.

## Error Handling (Component-Level Error Boundary)

One component's bug must never take down the rest of the UI. The base class wraps the template method's execution and every event handler invocation:

- If the template method throws: caught, logged (component tag name + error) to console, and a minimal fixed fallback markup is rendered in that shadow root (e.g. "this section failed to load") instead of a half-rendered/broken shadow root.
- If an event handler throws: caught, logged the same way, component keeps functioning for subsequent interactions rather than becoming permanently unresponsive.
- In both cases the base class publishes one fixed event, `ui:component-error`, carrying only the component's tag name and an error code/message — never full stack traces or sensitive data. This is a UI-layer concern (a component's own rendering/handler failure), not a plugin's critical error, and not a substitute for a plugin's own critical-error reporting to the Core — but like any Event Bus event it is publicly visible to any listener.
- `ui:component-error` is always declared in the UI manifest's `events` section (see `architecture-context/ui-manifest.md`) — not an ad hoc, undeclared event. Declared once per project; published centrally by the base class, never redeclared per component.
- **Console logging always happens; forwarding to a logging plugin is optional.** If the project has a logging infrastructure plugin (`LoggerService`, see `architecture-context/service-contract.md`) and the Entry Point declared it as an **optional** dependency and handed a reference to the base class, the base class also forwards the error there. If unavailable, console logging alone is sufficient — error handling must never depend on a logging plugin being present.
