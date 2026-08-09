# Testing

Read this before writing or reviewing any test for a component or for the shared base class.

## Testing a Component

Component logic must be testable without a running Core or a real service, in the same spirit as the architecture's testability principles for plugins:

- Because a component only ever receives references handed to it (never fetching them itself), a test constructs a component and hands it **fake/in-memory service objects** matching the same method contracts — exactly as an adapter can be swapped for an in-memory version in tests.
- A component's rendered output must be verifiable by inspecting its shadow root's content after feeding it known state/props — no test should need to simulate the Core, the Event Bus, or a real backend.
- DOM events a component dispatches (`ui:*` events or bubbled `CustomEvent`s) must be assertable by attaching a listener in the test, not by inspecting internal state.

## Testing the Shared Base Class

**The shared base class itself must have its own dedicated test suite**, separate from and in addition to any individual component's tests. Since the base class is where nearly all cross-cutting behavior lives — reference handing, render invocation, the input-preservation exception, Event Bus subscribe/unsubscribe, the error boundary, `ui:component-error` publishing — a bug there silently affects every component built on top of it.

At minimum, the base class's tests must cover, using a minimal throwaway subclass:

- A thrown render error is caught and produces the fallback markup and `ui:component-error`.
- Event Bus subscriptions registered on connect are removed on disconnect.
- The UI-local state store's `subscribe`/`unsubscribe` and equality behavior (see `shared-state.md`) behave as specified.
- The delegated listener is attached exactly once regardless of how many times the component re-renders.
