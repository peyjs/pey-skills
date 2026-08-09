---
name: bonyan-testing-standard
description: Use when writing tests for a Bonyan-architecture plugin or the adapter it consumes. Triggers on requests like "write tests for X", "add test coverage for this plugin", "test this service".
---

# Testing Standard (Bonyan)

Read this only when a task explicitly asks for tests. No external testing tool/framework is used — use Node's built-in `node:test` and `node:assert` only. This is consistent with Bonyan's own no-framework-dependency principle; nothing beyond the JavaScript runtime is added.

## File & Structure
- Test file: `<subject-file-name>.test.js`, placed under `tests/`, mirroring the plugin's own source folder structure (e.g. `src/services/transactions-service.js` → `tests/services/transactions-service.test.js`). Never colocated next to the source file.
- One test file per source file, mirroring `coding-standard.md`'s one-responsibility-per-file rule — not one per service. A service split across multiple files (e.g. `transactions-service.js` + `transactions-validator.js`) gets one test file per source file, not one combined file for the whole service.
- Arrange–Act–Assert per test. Every test must run independently — no shared mutable state between tests, no reliance on execution order.
- Test naming template: `should_<expected outcome>_when_<condition>` (e.g. `should_reject_task_creation_when_limit_exceeded`). Use this exact template for every test name, not free-form prose.
- Async tests use `async`/`await`, never raw `.then()` chains — consistent with `coding-standard.md`.

## What to Fake (not deep-mock)

Bonyan is built to be testable without real infrastructure (see `architecture.md` → Testability Principles). Use that, don't work around it:

- **Adapters** — replace with an in-memory implementation exposing the exact same method set defined in `adapter-contract.md` for that kind. Never test against the real external technology (real IndexedDB, real network, real filesystem).
- **Other plugins' services** — a plugin never touches a Registry directly (see `core-api-reference.md`); it only ever receives a plain `services` object as the argument to `activate(services)`. So the fake is just that: a plain object with the service names your plugin declared as keys, and fake implementations as values — `{ 'vendor.other.service': { methodX: async () => {...} } }`. Don't build or mock a "Registry" abstraction; it doesn't match what your code actually touches.
- **Event Bus** — use a fake that records published events for assertion on `type`/`data` only. `id`, `source`, `timestamp` are added by the Core, not this plugin's concern to test.

## What Must Be Tested

- Every method of every service in `provides`: at least one success path and one operational-error path.
- Every business rule this plugin enforces has its own dedicated test — not only covered incidentally through a method test.
- Every declared event: published with correct `type` and only identifying `data` when the triggering operation succeeds; **not** published when it fails.
- Every `required` dependency becoming unavailable at runtime: assert the plugin reports a critical error to the Core — not that it silently continues.
- Every `optional` dependency absent: assert reduced-capability behavior, not a crash.
- Module initializes correctly with valid config, and fails clearly (a structured error, not a crash) with missing required config.
- Module releases all resources (subscriptions, listeners, timers) on shutdown — see `coding-standard.md` → Resource Management.

## Shared Contract Tests for Adapters
Because every adapter of a kind must expose identical methods, write one shared contract test suite per kind (e.g. "storage adapter contract"), asserting only against the base methods, and run that same suite against every implementation of that kind (in-memory, real-technology-backed, etc.). Do not write separate from-scratch tests per implementation for the same behavior.

**Where this suite physically lives is a project decision, not a Bonyan rule** — the architecture doesn't define where adapters live or how many plugins may implement the same kind. Placing it in a project-level shared test-utilities location (rather than duplicating it inside every plugin that happens to use a `storage` adapter) is a reasonable default, but confirm this with the project owner rather than assuming.

## Rules
- Never assert on internal/private implementation details — only on declared services, published events, and manifest-visible behavior.
- Never let a test touch real network, real filesystem, or real timers — use `node:test`'s built-in `t.mock.timers` for anything timer-based (setTimeout/setInterval cleanup, debounce logic, etc.). This is part of `node:test` itself, not an external package, so it doesn't violate the no-external-tool rule above.
- Multiple `assert` calls in one test are fine only if they check the same behavior from different angles; otherwise split the test.

## If a Plugin Resists This Structure
If testing a plugin this way is impossible without spinning up the real Core or a real adapter, that is a sign of a hidden dependency in the plugin, not a reason for heavier test scaffolding. Report it per the Architecture Feedback Clause in `SKILL.md` instead of working around it.

## Delivery Gate
All tests must pass before the code is considered delivered. A failing test is never left for "later" or reported as a known issue alongside delivered code.