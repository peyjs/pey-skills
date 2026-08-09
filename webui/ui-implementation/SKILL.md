---
name: bonyan-ui
description: Use this skill whenever creating, editing, reviewing, or reasoning about a UI component, the UI Entry Point, or the UI manifest in a project built on the Bonyan architecture. This applies even if the user just says "add a button", "make a form", "build a settings page", "fix this component", "set up the UI entry point", or mentions Web Components, Shadow DOM, or a .js file inside the project's ui/ folder — any of these should trigger this skill. Always consult this skill before writing or modifying UI code in a Bonyan project, not just when the user explicitly mentions "Bonyan" or "conventions".
---

# Bonyan UI Implementation

This skill makes every UI component in this project consistent, regardless of which session or model wrote it. It is the mandatory reference for implementing the UI layer of a Bonyan-architecture application — no framework, no build tool, native Web Components only.

**This skill governs the UI Entry Point exactly as much as it governs individual components.** The Entry Point is the single most important file in the UI layer — it is the only thing the Core talks to, the only thing that reads the UI manifest, and the only thing that hands references down to every component. Any task that touches the Entry Point (bootstrapping, mounting the shell, wiring the route registry, handling shutdown, distributing config) must follow this skill the same way a component-level task does — don't assume the rules below are component-only just because most of the reference files describe component behavior.

Read this file first. It gives the non-negotiable core rules directly. For anything beyond the core rules — forms, i18n, styling, testing, the shared state store, architecture background — go to the matching reference file listed below **before** writing code in that area. Do not guess or rely on memory of a previous read; re-open the relevant reference file each time that topic comes up, even within the same conversation.

## Core Rules (always apply, no exceptions)

1. **Ownership boundary.** Only the UI Entry Point talks to the Core (services, Event Bus, config). A component never fetches or declares its own service dependency — it only receives what the Entry Point hands it.
2. **One shared base class.** Every component extends it. Never reimplement lifecycle, rendering, event subscription/cleanup, or error handling per component.
3. **Shadow DOM is mandatory** on every component (`attachShadow({ mode: 'open' })`). No exceptions, even for trivial components.
4. **Render:** full re-render of the shadow root from one template method is the default. Preserving focus/scroll/input-in-progress across re-render uses the base class's documented exception mechanism — never a component's own ad hoc diffing.
5. **Events are delegated on the shadow root by default**, registered once in `connectedCallback`, removed in `disconnectedCallback`. Never attach a listener inside code that runs on every render.
6. **Only transient, presentational local state** lives in a component (open/closed, active tab). Product/business data never does — always fetched from a service.
7. **Parent → child:** properties/attributes, or a documented public method. **Child → parent:** DOM `CustomEvent`, bubbled. **Siblings:** never reference each other directly.
8. **Business logic never enters a component.** No validation, no calculation, no product decisions — those belong to plugins.
9. **Routing goes through `RouterService`** (an infrastructure plugin), never `pushState`/`popstate` directly. The Router tells the Entry Point which route is active; the Entry Point is the only thing that mounts/unmounts page components and performs lazy `import()`.
10. **No build tools.** Native ES Modules, native CSS, native `<template>`. What's written in the file is what runs in the browser.
11. **The Entry Point has its own lifecycle contract with the Core**, including shutdown: on `core:shutdown`, the Entry Point releases its resources and stops before any plugin shuts down. See `architecture-context/core-boundaries.md` for the exact sequence.

If a task only touches these eleven points, you can proceed without opening a reference file. Anything more specific — the exact shape of something, an edge case, a convention for a domain not covered above — means open the matching file below.

## Reference Files — Open When Relevant

| When you're about to... | Read this file |
|---|---|
| Decide whether something is business data, UI state, or a notification | `references/decision-guide.md` — check this first when unsure |
| Start a brand-new component or page component from scratch | `references/templates.md` |
| Write a component's class structure, template method, render/error-handling behavior, event binding, or local state | `references/component-basics.md` |
| Wire up parent/child communication, decide who mounts/unmounts a component, implement routing reactions, or add lazy-loaded/page-level components | `references/composition.md` |
| Build any `<form>`, handle submission/validation/error display, or add/localize user-facing text, dates, numbers, currency | `references/forms-and-i18n.md` |
| Style a component, add an icon/font/image, or address keyboard/ARIA/focus accessibility | `references/styling-and-assets.md` |
| Read or write the UI-local shared state store (theme, cross-component UI toggles) | `references/shared-state.md` |
| Decide where a new file goes, or what to name it | `references/file-layout.md` |
| Write or review any test (component or the shared base class) | `references/testing.md` |
| Double-check whether something you're about to do is disallowed | `references/forbidden-patterns.md` |
| Need to justify *why* a UI rule exists in terms of Core/Registry/Event Bus, need the UI manifest's exact shape, need a service-call/error-object shape, or need naming rules for events/services | `references/architecture-context/` (see its own index below) |

### `references/architecture-context/` index

| When you need... | Read this file |
|---|---|
| Why the UI never touches the Core/Registry directly, what the Event Bus guarantees | `core-boundaries.md` |
| The UI manifest's exact structure (`dependencies`, `events`) and how the Core uses it | `ui-manifest.md` |
| How a service call looks/behaves from the caller's side | `service-contract.md` |
| The standard error object shape (`code`, `message`, `type`, etc.) | `error-structure.md` |
| Naming rules for plugins, services, and event types | `naming-conventions.md` |

## Workflow for a New or Edited Component

1. Identify which reference files this task touches from the table above — usually `component-basics.md` plus one or two others.
2. Open and read them fully before writing code; don't rely on a summary from earlier in the conversation.
3. Implement, extending the shared base class, following the Core Rules and whatever the opened reference files specify.
4. Before finishing, check the task against `references/forbidden-patterns.md`.
5. If the component includes a form, or displays a service-rejected error, or shows user-facing text/dates/numbers, confirm `forms-and-i18n.md` was actually consulted — these are the most commonly skipped.
6. **Before considering the task done, check whether the UI Map needs updating** (added/removed/renamed a component or page, changed a route, added a shared-state key or design token, made a non-obvious UX call, touched a multi-page flow) — see `bonyan-ui-documentation`'s `references/ui-map/README.md` for which specific file that change belongs in. This step is part of finishing the task, not an optional follow-up; skipping it is how the UI Map goes stale.

## If a Rule Seems to Conflict With the Request

If the user's request seems to require breaking a Core Rule or something in a reference file (e.g. "just put the state in a global variable," "skip Shadow DOM for this one"), say so explicitly, explain the specific rule and why it exists, and propose the compliant way to achieve the same goal before doing anything else. Don't silently comply and don't silently ignore the request either.
