# Composition

Read this before wiring components together, deciding who creates/destroys a component, reacting to routing, or adding a lazily-loaded component.

## Parent–Child Communication

- **Parent → child, setting data in:** properties or attributes set by the parent on the child element.
- **Parent → child, triggering an action:** a parent may call a method on a child **only if that method is part of the child's intentionally exposed public API** — a method the child's own definition documents as meant for outside callers (e.g. `modal.open()`, `list.scrollToItem(id)`). Never reach into a child's shadow root, private fields, or any undocumented method.
- **Child → parent:** only through native DOM events (`CustomEvent`, via `dispatchEvent`) that bubble out of the shadow root. A child never calls a parent method and never reaches upward through the DOM tree.
- **Siblings never talk to each other directly.** Coordination between components without a direct parent-child relationship happens only through a service call, the Event Bus, or the UI-local shared state store (`shared-state.md`) — never through a reference one component holds to another.

## Component Ownership and Lifecycle

Components are never self-instantiating at the top level:

- The **UI Entry Point** creates and mounts the top-level shell once at startup, and owns a **route registry**: a fixed mapping from route pattern to a loader for the page-level component responsible for it (see Lazy Loading below).
- The **Router**, on `router:changed`, determines the current route and calls one fixed method on the Entry Point (e.g. `entryPoint.mountForRoute(route)`) — the Router never imports, constructs, or directly mounts a page component itself. The Entry Point resolves the loader from its route registry, obtains the component (importing it if necessary), mounts it, and unmounts whatever page component was previously mounted.
- Below the page level, a component may create child components it directly contains as part of its own template — normal composition, not a violation — but it must never create another *page-level* component; that remains the Entry Point's responsibility, invoked by the Router.

This keeps "who decides mounting should happen" (Router, based on route) separate from "who actually performs loading and mounting" (Entry Point, exclusively).

## Routing

Routing is not implemented inside the UI layer. It's provided by a dedicated infrastructure plugin (`RouterService`).

UI components:
- Read the current route by calling `RouterService`.
- Request navigation by calling `RouterService`, never by manipulating browser history directly.
- Listen to `router:changed` to know when to re-render. Mounting/unmounting page-level components in response is the Entry Point's job (see above) — not something an arbitrary component does on its own.

No component contains `pushState`, `popstate`, or any direct History API usage.

## Lazy Loading

Because the product is expected to grow substantially over a long lifetime, not every component needs to load at startup. Dynamic `import()` is explicitly permitted and encouraged for page-level or rarely used components.

Concretely: the Entry Point's route registry maps a route to a **loader** — a small function that performs the dynamic `import()` and returns the component's class/definition. The Entry Point is the only part of the system that calls this loader; the Router only ever tells the Entry Point which route is now active.

This changes nothing else: a lazily loaded component still extends the shared base class, still receives its references the same way, and is still mounted/unmounted only by the Entry Point.

## Module Structure and Loading

- Each component is one self-contained ES Module file (class, template method, styles together), loaded via native `import` (statically at startup, or dynamically per Lazy Loading above).
- No bundling step. Files are served and imported as-is.
- The UI entry point (read by the Core via `ModuleLoader`) imports and registers all components needed at startup, and is the sole owner of any dynamic import used for lazy-loaded ones.
- Folder layout and naming are fixed, not left to individual judgment — see `file-layout.md` for the exact structure.
