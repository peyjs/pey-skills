---
name: bonyan-integration-testing
description: Use when writing the system/integration/scenario test suite of a Bonyan-architecture host app — the UI-free tests that boot the real Core with the real set of plugins this app actually ships, as opposed to a single plugin's own isolated unit tests. Triggers on requests like "write integration tests for this app", "test that these plugins work together", "write a test for this user scenario/journey", "how do I test the whole system".
---

# Integration Testing (Bonyan)

`plugin-testing-standard/SKILL.md` deliberately fakes the Core, every other plugin's services, and the Event Bus so one plugin can be tested in isolation — and it says so explicitly: needing a real Core or a real adapter to test a plugin is a sign of a hidden dependency, not a reason for heavier scaffolding. That rule is correct for a single plugin, and it is exactly why isolated unit tests structurally cannot catch what only exists once several **real** plugins are assembled together in one host app:

- Two real plugins whose manifests don't actually agree (a declared dependency's shape doesn't match what the other's `provides` actually exposes).
- The real startup/shutdown order produced by this app's actual dependency graph.
- A real adapter named in this app's `bootstrap.json` genuinely satisfying what a plugin expects from it.
- A real cross-plugin business flow — one plugin's real output feeding another's real input, not a stand-in value.
- The real UI Entry Point receiving the real, bound `services` object the Core actually hands it.

This skill covers that suite, in two distinct categories (see "What must be tested" below): **wiring/contract** checks, and **scenario** checks — a real user/business journey carried out against real services. Both share the same environment and both are UI-free: this suite never renders a component or drives a browser. It lives only in a **host app**, under `src/tests/integration/` (see `project-structure/references/host-app-layout.md`). A plugin's own `tests/` never grows this kind of test — that belongs here instead.

## Precedent this follows

`pey-core/docs/testing.md` already prescribes the same shape for testing Core itself: fake only the three Runtime Host sources (`ConfigSource`, `EnvSource`, `ModuleLoader`), then exercise real flows end-to-end through the real `PeyCore` class. Apply that same idea one level up — to a host app's real, chosen set of plugins instead of Core's own internals.

## Environment

Boot the real `PeyCore` from the project's actual installed `pey-core` dependency (never hand-copied — see `project-structure/SKILL.md`'s Dependencies rule) together with the real plugins this app actually ships, configured the same way this app's real `bootstrap.json` configures them. Fake only:

- The three Runtime Host sources (`configSource`, `envSource`, `moduleLoader`) — in-memory equivalents, exactly like `pey-core/docs/testing.md`'s own fakes. The `moduleLoader` still loads each plugin's **real** `manifest.json` and `index.js` from disk/import — only the three sources are the fake part, not the plugins they load.
- Whatever is genuinely external I/O for this specific app (network, device APIs) — never a plugin, never the Core.
- The UI, with a minimal fake entry point whose only job is capturing the real `services` object Core binds to it, so the test can then call those real services exactly as the real UI Entry Point would. Never reimplement plugin behavior inside this fake UI.

A platform-specific adapter that cannot run under the test environment (e.g. a browser-only adapter under Node) is excluded from this suite with a one-line comment saying why and where its coverage lives instead (typically a separate real-browser end-to-end pass). Don't force a fake adapter in just to include it — that collapses the test back into a unit test and defeats the purpose.

## What must be tested

### Wiring and contract checks

These verify that the real pieces connect correctly — not that any particular business outcome is right.

- **Startup reaches the ready state** with this app's actual plugin set, and the relative order of readiness events matches the real declared dependency graph (a dependent's ready event after its real required dependency's).
- **The real UI Entry Point's service map**: every service the UI's manifest declares is present and callable on the object Core hands it.
- **Every optional-dependency variation this app's `bootstrap.json` actually supports** (a feature toggled on/off by including or excluding a plugin or adapter) is exercised at least once — not only the everything-present happy path.
- **A required dependency genuinely failing to activate** (make the fake `moduleLoader` refuse one real plugin) cascades to its real dependents the way this app's own manifests declare.
- **Shutdown**: resources released and `*:stopped` events published in the real reverse order across this app's actual plugin set; a service reference obtained before shutdown fails safely (not silently, not by throwing an unrelated error) when called afterward.

### Scenario checks — real business/user journeys

A wiring check only proves components *can* talk; it does not prove a real journey produces the right result. For every journey the host app's product actually offers that spans more than one real plugin, write at least one scenario test: call the real services directly, in the same order and with the same data shape that journey would actually produce them, and assert the final business-meaningful outcome — not merely that no call threw.

- Keep this list to the journeys that actually matter to the product (its critical paths), not every combination that is technically possible — a scenario suite that tries to cover everything becomes slow and brittle for no extra confidence, the same failure mode general testing guidance warns about for broad end-to-end suites.
- A scenario test calls services the same way the shirazeh precedent does: directly, through the real bound `services` object, never by rendering a component or simulating a UI interaction.
- When a scenario's real output depends on genuinely external I/O (network, a remote API), fake only that I/O — never the plugins producing or consuming it.

### What this suite deliberately does not cover

Whether a **route or UI interaction correctly invokes** the right real service with the right arguments is `ui-testing-standard/SKILL.md`'s "UI Entry Point" responsibility (`mountForRoute`, tested with fake services) — not this suite's. This suite starts one layer below that: it assumes the UI will call the correct service correctly (proven elsewhere, with fakes) and instead proves that when a service *is* called the way a real journey would call it, the real plugins behind it produce the right result together. Splitting it this way keeps both suites UI-free or logic-free respectively, with no gap and no overlap: routing correctness is tested once, with fakes; cross-plugin business correctness is tested once, with the real Core.

## Boundary with other skills

- **Never re-verify Core's own internal mechanisms in the abstract** (that group-based startup, cascading deactivation, or event enrichment are implemented correctly) — that is `pey-core`'s own test suite's job, already covered by its `docs/testing.md`. This suite only verifies that *this app's* specific real plugins and `bootstrap.json` compose correctly together.
- **Never add this kind of test inside a plugin's own `tests/`** — `plugin-testing-standard/SKILL.md` owns that folder and explicitly fakes the Core there; a plugin resisting isolated testing is reported per its Architecture Feedback Clause, not fixed by smuggling a real-Core test into the plugin.
- **UI component-level tests stay in `ui-testing-standard/SKILL.md`** — this skill only touches the UI Entry Point's real service wiring, not individual component rendering.

## File & structure

`src/tests/integration/<app-name>-wiring.test.js` for wiring/contract checks, `src/tests/integration/<app-name>-scenarios.test.js` for scenario checks — kept as separate files even though they share the same environment, because they answer different questions and tend to grow at different rates. Split either further by feature or flow once it grows unwieldy. Same conventions as the other testing-standard skills: Arrange–Act–Assert, `node:test` + `node:assert` only, test names as `should_<expected outcome>_when_<condition>`, every test independent with no shared mutable state.

## Delivery Gate

All tests must pass before the app is considered delivered — same rule as `plugin-testing-standard/SKILL.md` and `ui-testing-standard/SKILL.md`.
