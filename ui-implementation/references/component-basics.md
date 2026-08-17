# Component Basics

Read this before writing or editing any individual UI component's class.

## Custom Elements + Shadow DOM

Every component is a Web Component extending `PeyElement`, registered with `customElements.define()`. Shadow DOM is mandatory — not optional, even for a trivial component — but **`PeyElement` creates it** (`attachShadow({ mode: 'open' })`); a subclass never calls `attachShadow()` itself. Purpose: guarantee CSS/DOM isolation between components written independently, at different times, by different sessions. This removes class-name collisions and leaking selectors as a category of bug.

Shadow DOM does not exempt a component from accessibility (see `styling-and-assets.md`).

## One Shared Base Class

All components extend a single shared base class; no component defines its own lifecycle pattern from scratch. The kit's `PeyElement` (`pey.webui/base/pey-element`) owns the connect-once envelope, render coalescing, subscription/listener management, the error boundary, and the preserve mechanism; a consumer extends it and implements the hooks its component needs — typically `onConnect(refs)`, `render()`, `subscriptions()`, `eventTypes()` with `handleEvent()`, plus `stylesheetHref()` / `stylesheetNonce()` when the component carries an external stylesheet. A component does no service lookup, no injection, no global access — see `architecture-context/core-boundaries.md` for why.

## Render Strategy

**Default:** full re-render of the shadow root's content from a template method on every state change. Default because it's the simplest strategy to predict and generate consistently across many independent sessions — not because it's the only permitted one.

**Named exception — input-holding elements:** live text input, text selection, scroll position, or an in-progress animation that must survive a re-render uses the base class's documented preserve mechanism — never a component's own ad hoc diffing. Opt in only for the specific elements that need it.

## Template Contract

Every component implements exactly one method for producing its content (`render()`), reading only `this`'s current state; only the base class's render mechanism consumes its output — never bypassed to mutate the shadow root from elsewhere (see `forbidden-patterns.md`).

## Event Binding Rule

**Default:** the base class registers one delegated listener per event type, once, on the shadow root — guaranteeing the listener count never grows with re-renders. Never attach a listener inside code that runs on every re-render.

## Local State

Only transient, presentational state lives in a component (dropdown open/closed, active tab, scroll position). Product/business data always comes from a service — never cached in a component.

## Error Handling

One component's bug must never take down the rest of the UI. The base class wraps the template method's execution and every event handler, logs failures, renders a minimal fixed text-free fallback markup, and publishes one fixed `ui:component-error` event (declared in the UI manifest's `events` section). The exact codes, fallback markup, and logging/forwarding rules are fixed in the kit — see `pey.webui/docs/decisions.md` (Component Base Decisions).