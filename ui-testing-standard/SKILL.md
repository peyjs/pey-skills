---
name: bonyan-ui-testing-standard
description: Use when writing tests for a Bonyan-architecture UI component, the UI Entry Point, the shared base class, or the UI-local shared state store. Triggers on requests like "write tests for this component", "add test coverage for this page", "test the base class".
---

# UI Testing Standard (Bonyan)

Read this only when a task explicitly asks for tests. No external testing tool/framework is used — use Node's built-in `node:test` and `node:assert`, plus a minimal DOM (see Environment below). Consistent with Bonyan's no-framework-dependency principle and with `bonyan-ui`'s Technology Baseline: nothing beyond the JavaScript runtime and a standards-compliant DOM is added.

This document assumes familiarity with `bonyan-ui/references/testing.md`, which states the two required test surfaces (component, and the shared base class) and the minimum base-class coverage list — this document goes into the concrete structure and fakes. If the two ever disagree, `bonyan-ui/references/testing.md` wins; report the conflict rather than resolving it by picking one.

## Environment
Custom Elements and Shadow DOM require a DOM. **The one officially sanctioned environment for this project is Node.js's `node:test` running against `jsdom`** (added as a dev-only dependency, never shipped in any production bundle or loaded by application code — this doesn't conflict with `bonyan-ui`'s no-framework/no-build-tool rule, which governs runtime code, not dev tooling). This is a fixed, project-wide choice specifically so every test file, written by any session, targets the same environment — don't substitute a different DOM implementation or test runner without updating this file first.

Since the actual runtime for this product is a real browser engine in every target (the browser itself, and Capacitor's and Tauri's embedded webviews are also browser engines), running the exact same, unmodified test files directly in a real browser — via that browser's own native test execution rather than any bundler-based runner — is also permitted and encouraged as a periodic supplementary check, not a replacement for the Node/`jsdom` suite. Native browser execution is optional; the Node/`jsdom` suite above is what's required for every change.

## File & Structure
- Test file: `<component-file-name>.test.js`, placed under `tests/`, mirroring the `components/`/`pages/` folder structure from `bonyan-ui/references/file-layout.md` (e.g. `components/example-widget/example-widget.js` → `tests/components/example-widget/example-widget.test.js`). Never colocated next to the source file.
- One test file per component file, mirroring `bonyan-ui-coding-standard`'s one-component-per-file rule.
- Arrange–Act–Assert per test. Every test must run independently — no shared mutable state between tests, no reliance on execution order. In particular, never reuse one component instance across tests; create and `connect()` a fresh one per test.
- Test naming template: `should_<expected outcome>_when_<condition>` (e.g. `should_render_error_state_when_service_call_rejects`). Use this exact template for every test name, not free-form prose.
- Async tests use `async`/`await`, never raw `.then()` chains — consistent with `bonyan-ui-coding-standard`.

## What to Fake (not deep-mock)

Bonyan is built to be testable without real infrastructure. A component only ever receives what `connect()` gives it (see `bonyan-ui/references/component-basics.md`) — the fake surface follows directly from that:

- **Services** — a component never touches a Registry or the Core; it only receives a plain object via `connect(refs)`. The fake is exactly that: a plain object with the same method names the real service exposes, returning resolved/rejected promises as needed for the test — `{ accountsService: { getById: async () => ({...}) } }`. Don't build or mock a "Registry" or "Entry Point" abstraction; it doesn't match what the component actually touches.
- **Event Bus** — use a fake exposing the same `subscribe`/`publish`-shaped surface the base class uses internally, recording published events for assertion on `type`/`detail` (for `ui:*` `CustomEvent`s dispatched on the element) or `type`/`data` (for the base class's internal Event Bus subscription mechanism, if the base class exposes one directly to tests). `id`, `source`, `timestamp` are added by the Core, not this component's concern to test.
- **UI-local shared state store** — use a minimal in-memory object implementing exactly the fixed API in `bonyan-ui/references/shared-state.md` (`get`/`set`/`subscribe`), including its documented equality behavior, so a test can verify a component reacts to `set()` the same way it would with the real store.
- **Configuration** — pass plain literal values through `connect()`; no fake needed beyond that, since config reaches a component the same way a service reference does (see `bonyan-ui/references/architecture-context/ui-manifest.md`).
- **`RouterService`** — fake it strictly per whatever the router infrastructure plugin's own `manifest.json` currently declares in `provides` (see the project's router plugin manifest and `router-types.md`) — this file intentionally does not enumerate method names, since duplicating them here would drift out of sync if that manifest ever changes. Look up the manifest before writing the fake; do not assume behavior beyond what it documents.

## What Must Be Tested

**Per component:**
- `render()` output for every distinct state the component can be in (e.g. loading/ready/error for a page component) — assert against the shadow root's content, not internal fields.
- Every public API method (see `bonyan-ui/references/composition.md`) — calling it produces the expected state change and re-render.
- Every entry in `subscriptions()` — simulate the relevant domain event through the Event Bus fake and assert the component reacts correctly (typically: re-fetches or re-renders), per `bonyan-ui/references/templates.md`'s pattern. **If a component's `subscriptions()` returns an empty object (it listens to nothing), this bullet is intentionally skipped — do not fabricate an artificial subscription or a placeholder test just to have something to assert here.**
- Every `ui:*` event the component can dispatch — assert it's dispatched with the correct `detail`, `bubbles: true`, `composed: true`, by attaching a listener in the test, not by inspecting internal state.
- Forms (if present): valid submission calls the service with correctly gathered data; a service-rejected error is displayed via the fixed `aria-describedby` convention, using the translated string, not the raw `message` — see `bonyan-ui/references/forms-and-i18n.md`.
- Cleanup: after `disconnectedCallback`, assert Event Bus subscriptions are removed (the fake Event Bus should confirm no further calls reach the component) **and, if the component subscribes to the UI-local shared state store (`shared-state.md`), assert that subscription's `unsubscribe()` was called too** — the fake store should confirm no further callback invocations reach the component after disconnect, the same way the Event Bus fake does — and any in-flight async work is aborted/guarded (see the `isConnected` pattern in `bonyan-ui/references/templates.md`).
- `connect()` called a second time on an already-connected instance throws, per `bonyan-ui/references/component-basics.md`'s call-once contract.

**For the shared base class** (in addition to `bonyan-ui/references/testing.md`'s minimum list, using a minimal throwaway subclass):
- A thrown error inside `render()` is caught, produces the documented fallback markup, and dispatches `ui:component-error` with only `componentTag`/`code` — never a stack trace.
- A thrown error inside an event handler is caught the same way, and the component remains responsive to further interaction afterward.
- The delegated listener is attached exactly once on `connectedCallback` regardless of how many times `requestRender()` is called afterward.
- Event Bus subscriptions declared via `subscriptions()` are registered on connect and removed on disconnect.
- If an optional logger reference was handed to the base class, errors are forwarded to it; if none was handed, no error is thrown for its absence (graceful degradation, per `bonyan-ui/references/component-basics.md`).

**For the UI-local shared state store** (see `bonyan-ui/references/shared-state.md`):
- `set()` with an unchanged primitive does not notify subscribers.
- `set()` with a new object/array reference (even if contents are equal) does notify subscribers.
- `unsubscribe()` actually stops further notifications.

**For the UI Entry Point:**
- On `core:shutdown`, resources are released and mounted components are properly unmounted before the method returns — see `bonyan-ui/references/architecture-context/core-boundaries.md`'s Entry Point Lifecycle section.
- `mountForRoute(route)` mounts the correct page component for a given route and unmounts the previously mounted one.

## Rules
- Never assert on internal/private implementation details — only on rendered shadow-root content, dispatched events, and public API behavior.
- Never let a test touch a real network, real Event Bus, real Core, or real timers — use `node:test`'s built-in `t.mock.timers` for anything timer-based. This is part of `node:test` itself, not an external package, so it doesn't violate the no-external-tool rule above.
- Multiple `assert` calls in one test are fine only if they check the same behavior from different angles; otherwise split the test.

## If a Component Resists This Structure
If testing a component this way is impossible without a real Core, a real service, or a real Event Bus, that's a sign the component is reaching outside what `connect()` gave it — a violation of `bonyan-ui/references/forbidden-patterns.md` — not a reason for heavier test scaffolding. Fix the component's dependency handling rather than working around it in the test.

## Delivery Gate
All tests must pass before UI code is considered delivered. A failing test is never left for "later" or reported as a known issue alongside delivered code.

**The tests themselves are also subject to `bonyan-ui` and `bonyan-ui-coding-standard`** — a test file is still a code file (naming, file organization, no ignored errors, etc.), and a test's fakes must still respect the ownership/injection boundaries in `bonyan-ui/references/core-boundaries.md` and `component-basics.md` (e.g. a test must not reach into a component's shadow root internals just because assertions would be easier that way — see `bonyan-ui/references/forbidden-patterns.md`). Passing tests is never achieved by loosening or working around an architectural rule in either the component or the test — if a rule seems to make a component hard to test, that's addressed by fixing the component's dependency handling (see "If a Component Resists This Structure" above), never by bypassing the rule in the test to force a pass.
