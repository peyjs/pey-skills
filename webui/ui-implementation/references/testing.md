# Testing

Read this before writing or reviewing any test for a product component or page.

The kit ships its own suite for the base class and runtime modules under its `tests/`; the guidance below covers the tests an application writes for its own product components and pages.

## Testing a Component

Component logic must be testable without a running Core or a real service, in the same spirit as the architecture's testability principles for plugins:

- Because a component only ever receives references handed to it (never fetching them itself), a test constructs a component and hands it **fake/in-memory service objects** matching the same method contracts — exactly as an adapter can be swapped for an in-memory version in tests.
- A component's rendered output must be verifiable by inspecting its shadow root's content after feeding it known state/props — no test should need to simulate the Core, the Event Bus, or a real backend.
- A component's **DOM `CustomEvent`s** (bubbled out of the shadow root) are asserted by attaching a DOM listener in the test, not by inspecting internal state.
- A component's **`ui:*` Event Bus events** are asserted through the Event Bus publish path (`publishUIEvent()`), not with a DOM listener — the two mechanisms are distinct and are never asserted interchangeably.
