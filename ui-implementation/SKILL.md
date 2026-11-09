---
name: bonyan-ui
description: Use this skill whenever creating, editing, reviewing, or reasoning about a UI component, the UI Entry Point, or the UI manifest in a project built on the Bonyan architecture. This applies even if the user just says "add a button", "make a form", "build a settings page", "fix this component", "set up the UI entry point", or mentions Web Components, Shadow DOM, or a .js file inside the project's ui/ folder — any of these should trigger this skill. Always consult this skill before writing or modifying UI code in a Bonyan project, not just when the user explicitly mentions "Bonyan" or "conventions".
---

# Bonyan UI Implementation

This skill makes every UI component in this project consistent, regardless of which session or model wrote it. It is the mandatory reference for implementing the UI layer of a Bonyan-architecture application — no framework, no build tool, native Web Components only. **`pey.webui` is the only allowed starting point: no UI is ever built from scratch — every task composes the kit and adds only the product's own components on top.**

**This skill governs the UI Entry Point exactly as much as it governs individual components.** The Entry Point is the single most important file in the UI layer — it is the only thing the Core talks to, the only thing that reads the UI manifest, and the only thing that hands references down to every component. The Entry Point comes from the kit's template (`templates/application-entry-point.js` → `ui/index.js`); adapting it (pages, routes, dependencies, events, translations) must follow this skill the same way a component-level task does.

Read this file first. It gives the non-negotiable core rules directly. For anything beyond the core rules — forms, i18n, styling, testing, the shared state store, architecture background — go to the matching source in the tables below **before** writing code in that area. Do not guess or rely on memory of a previous read; re-open the relevant file each time that topic comes up, even within the same conversation.

## Core Rules (always apply, no exceptions)

1. **`pey.webui` is the only allowed starting point.** Every UI task starts from the `pey.webui` kit — never build UI infrastructure from scratch. **Before creating any infrastructure file, check whether `pey.webui` already exports that capability; if it does, importing/composing it is mandatory.** The kit ships the component base class, the shell, routing, i18n, the shared-state store, design tokens, and its own docs; the application only adds its own product components and pages on top.
2. **Never modify the `pey.webui` package.** Code inside `pey.webui`, whether in the package source or in its `docs/`, is never edited by this skill. The application's own `ui/index.js` and `ui/manifest.json` are app-owned and are routinely adapted.
3. **Report kit gaps, don't patch them.** If something in the kit is missing, buggy, or differs from the project's needs, state it explicitly to the user instead of patching or working around kit code, so it can be fixed by the `pey.webui` maintainer in a new version.
4. **Ownership boundary.** Only the UI Entry Point talks to the Core (services, Event Bus, config). A component never fetches or declares its own service dependency — it only receives what the Entry Point hands it.
5. **One shared base class.** Every component extends `PeyElement` (`pey.webui/base/pey-element`). Never reimplement lifecycle, rendering, event subscription/cleanup, or error handling per component.
6. **Every component uses the Shadow DOM that `PeyElement` creates** — a subclass never calls `attachShadow()` itself. Shadow DOM is mandatory, with no exceptions, even for trivial components.
7. **Render:** full re-render of the shadow root from one template method is the default. Preserving focus/scroll/input-in-progress across re-render uses the base class's documented exception mechanism — never a component's own ad hoc diffing.
8. **Event handling is declarative on `PeyElement`.** A component implements `eventTypes()` and `handleEvent()`; the base class registers the delegated listeners once and removes them on disconnect. A component never writes `connectedCallback`/`disconnectedCallback` (or any render-path code) to manage listeners.
9. **Only transient, presentational local state** lives in a component (open/closed, active tab). Product/business data never does — always fetched from a service.
10. **Parent → child:** properties/attributes, or a documented public method. **Child → parent:** DOM `CustomEvent`, bubbled. **Siblings:** never reference each other directly.
11. **Business logic never enters a component.** No business validation, no calculation, no product decisions — those belong to plugins. (Syntactic client-side form constraints — `required`, `minlength`, `pattern` — are not business validation and are allowed; see `references/forms-and-i18n.md`.)
12. **Routing goes through `RouterService`** (an infrastructure plugin), never `pushState`/`popstate` directly. The Router tells the Entry Point which route is active; page mounting/unmounting and lazy `import()` are handled by the kit's page host (`pey.webui/routing/`).
13. **No build tools.** Native ES Modules, native CSS, native `<template>`. What's written in the file is what runs in the browser.
14. **The Entry Point has its own lifecycle contract with the Core.** `core:shutdown` is **notification only** — the Entry Point's cleanup runs in the synchronous `context.onShutdown` it registers, and completes before any plugin shuts down. The kit implements this in `createApplicationSetup()`; see `pey.webui/docs/entry-point.md` for the contract and `references/architecture-context/core-boundaries.md` for the rationale.

If a task only touches these points, you can proceed without opening a reference file. Anything more specific — the exact shape of something, an edge case, a convention for a domain not covered above — means open the matching file below.

## Sources — UI Topics (Primary In The Kit, Background In `references/`)

For every UI topic the primary, mandatory source is the kit itself — `pey.webui/docs/`, `pey.webui/templates/`, or a kit module. The `references/` file is optional background for understanding *why* the contract is the way it is; read it only when the kit doc doesn't settle the question.

| When you're about to... | Primary source (mandatory) | Background / why (optional) |
|---|---|---|
| Start any UI task: install/import the kit and copy the application templates | `pey.webui/README.md` (Installation); `pey.webui/templates/application-manifest.json` → `ui/manifest.json`, `pey.webui/templates/application-entry-point.js` → `ui/index.js` | `references/file-layout.md` |
| Decide where an application file goes or what to name it | `references/file-layout.md` | — |
| Write a component's class structure, template method, render/error-handling, event binding, or local state | `pey.webui/docs/public-api.md` (`PeyElement`); `pey.webui/docs/decisions.md` (Component Base Decisions) | `references/component-basics.md` |
| Wire up parent/child communication, decide who mounts/unmounts, react to routing, or add lazy-loaded/page components | `pey.webui/docs/entry-point.md`; `pey.webui/docs/public-api.md` (Page Loading And Routing) | `references/composition.md` |
| Build any `<form>`, handle submission/validation/error display, or add/localize user-facing text, dates, numbers, currency | `pey.webui/docs/public-api.md` (I18n And Formatting); `pey.webui/i18n/` | `references/forms-and-i18n.md` |
| Style a component, add an icon/font/image, or address keyboard/ARIA/focus accessibility | `pey.webui/docs/decisions.md` (CSS Attach Method, Icon Sprite Contract); `pey.webui/tokens.css` | `references/styling-and-assets.md` |
| Read or write the UI-local shared state store (theme, cross-component UI toggles) | `pey.webui/docs/public-api.md` (`createSharedState()`); `pey.webui/shared-state.js` | `references/shared-state.md` |

## Sources — Concepts Outside The Kit

These topics are kit-independent: plugin authoring, the general Core/Registry contract, and architectural rationale the kit does not ship. They live in `plugin-implementation/references/core/` and the untouched `references/` files.

| When you need... | Read this |
|---|---|
| Decide whether something is business data, UI state, or a notification | `references/decision-guide.md` — check this first when unsure |
| Confirm a planned action isn't disallowed | `references/forbidden-patterns.md` |
| Write or review a test | `references/testing.md`; `bonyan-ui-testing-standard` |
| Understand why the UI never touches the Core/Registry directly, what the Event Bus guarantees | `references/architecture-context/core-boundaries.md` |
| The generic UI manifest's exact structure and how the Core uses it | `references/architecture-context/ui-manifest.md` |
| How a service call looks/behaves, and the standard error object shape | `references/architecture-context/service-contract.md`; `references/architecture-context/error-structure.md` |
| Naming rules for plugins, services, and event types | `references/architecture-context/naming-conventions.md` |
| Plugin authoring or other Core-level contracts | `plugin-implementation/references/core/*.md`; `router-reference.md`; `pey-router-manifest.json` |

## Workflow for Any UI Task

1. **Start from the kit — never from scratch.** Ensure `pey.webui` is installed/imported per `pey.webui/README.md`. Only when starting a *new* application (not for routine component work) copy `templates/application-manifest.json` → `ui/manifest.json` and `templates/application-entry-point.js` → `ui/index.js`, adapting pages, routes, dependencies, events, and translations. Then build the product component/page per `references/file-layout.md` in `ui/components/` or `ui/pages/`.
2. Identify the primary source this task touches from the tables above and open it fully before writing code; don't rely on a summary from earlier in the conversation.
3. Implement the product component/page extending `PeyElement`, following the Core Rules and whatever the primary source specifies.
4. Before finishing, check the task against `references/forbidden-patterns.md`.
5. If the component includes a form, or displays a service-rejected error, or shows user-facing text/dates/numbers, confirm the **I18n And Formatting** source in `pey.webui/docs/public-api.md` was actually consulted — these are the most commonly skipped.
6. **Before considering the task done, check what needs registering** — a new `ui:*` event or service dependency belongs in `ui/manifest.json`; routes are registered by the route-owning application plugin via `pey.router.service.registerRoutes()` (never in the UI manifest), and the page-loader catalog in `ui/index.js` maps each route to its page component. This step is part of finishing the task, not an optional follow-up.
7. **Close the task** once the steps above pass: add a changelog entry per `changelog/SKILL.md`; update only what this task changed: `ui-documentation/SKILL.md` (components and UI Map), `readme/SKILL.md` (top-level files, dependencies, install/use), `introduction/SKILL.md` (behavior a reader would notice); then end the reply with the one-line commit message per `git-workflow/SKILL.md`. Run no git command unless asked.

## If a Rule Seems to Conflict With the Request

If the user's request seems to require breaking a Core Rule or something in a reference file (e.g. "just put the state in a global variable," "skip Shadow DOM for this one," "patch this inside `pey.webui`"), say so explicitly, explain the specific rule and why it exists, and propose the compliant way to achieve the same goal before doing anything else. Don't silently comply and don't silently ignore the request either.