# Forbidden Patterns

Read this as a final self-check before finishing any UI component. The following are never permitted, regardless of how minor or temporary the reason seems.

- **No `document.querySelector` / `document.getElementById` reaching outside a component's own shadow root.** A component only ever queries within its own `this.shadowRoot`.
- **No content mutation outside the render cycle.** All *content* changes (markup, text, data-driven attributes) go through the component's single template method and the base class's render mechanism (including the input-preservation exception) — never an ad hoc `element.innerHTML = ...` called from elsewhere in the component's code.
  - **Exception — transient visual-only changes:** direct, imperative calls that affect presentation but not data (toggling a CSS class to trigger a CSS transition, calling `.focus()`/`.scrollIntoView()`, updating a `transform`/`style` property during a drag or animation frame) are allowed outside the render cycle — re-running a full template render for every animation frame would itself be the wrong tool. The line: if it reflects a change in the component's actual state/props, it goes through render; if it's purely visual and reverts to whatever render would already produce, it may be imperative.
- **No global singletons for state or services.** No `window.myAppState`, no module-level mutable shared object standing in for a service. The only sanctioned shared object is the UI-local state store (`shared-state.md`), and only when handed down by the Entry Point.
- **No component-level service or dependency lookup.** A component never imports a service module directly or asks the Core/Registry for anything; it only uses what the Entry Point handed it.
- **No inline event handlers baked into template markup**, and no listener attachment placed inside code that runs on every re-render.
- **No product/business data cached in component-local state.**
- **No component instantiating another page-level component** — that's the Entry Point's job, invoked by the Router.
- **No direct History API usage** (`pushState`, `popstate`) — always through `RouterService`.
- **No raw error `message` shown to the user** — always `code` → translated string → display (see `forms-and-i18n.md`).
- **No mutating an object/array obtained from the shared state store in place** — always `set()` with a new reference (see `shared-state.md`).
