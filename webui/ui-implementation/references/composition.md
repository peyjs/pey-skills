# Composition

Read this before wiring components together, deciding who creates/destroys a component, reacting to routing, or adding a lazily-loaded component.

## Parent–Child Communication

- **Parent → child, setting data in:** properties or attributes set by the parent on the child element.
- **Parent → child, triggering an action:** a method call **only if it is part of the child's intentionally exposed public API** (e.g. `modal.open()`, `list.scrollToItem(id)`); never reach into a child's shadow root, private fields, or undocumented methods.
- **Child → parent:** only native DOM events (`CustomEvent`, via `dispatchEvent`) that bubble out of the shadow root; a child never calls a parent method or reaches upward.
- **Siblings never talk to each other directly** — coordination goes through a service call, the Event Bus, or the UI-local shared state store (`shared-state.md`).

## Who Creates, Mounts, And Unmounts What

Components are never self-instantiating at the top level. Only the **UI Entry Point** mounts page-level components (and creates the top-level shell at startup); a component never creates another page-level component. This behavior is implemented by the kit: see `pey.webui/docs/entry-point.md` and `pey.webui/routing/` (`page-host`, `page-loader`, `page-fallback`). The Router never mounts anything and never calls a method on the Entry Point — it only publishes `router:changed`.

## Routing

Routing is not implemented in the UI layer; it's provided by the `RouterService` infrastructure plugin. Components request navigation through `RouterService` — never by manipulating browser history directly. **Page mounting/unmounting in response to `router:changed` belongs to the kit's page host (`pey.webui/routing/page-host`), not to an arbitrary component.** A component subscribes to `router:changed` only when its own presentation is genuinely route-aware (e.g. highlighting the active nav item), and it does so deliberately — never as a blanket subscription. No component contains `pushState`, `popstate`, or any direct History API usage.

## Module Structure And Loading

- Each component is one self-contained ES Module file (class, template, styles together), loaded via native `import` — no bundling step.
- Folder layout and naming are fixed, not left to individual judgment — see `file-layout.md`.