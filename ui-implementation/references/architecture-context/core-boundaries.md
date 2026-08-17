# Core Boundaries (as they affect the UI)

Self-contained summary — only the parts of Bonyan's Core architecture relevant to implementing the UI layer. For the complete architecture, the project's `core.md` and `ui.md` remain the full reference; this file exists so a UI task doesn't require reading those in full each time.

## What the Core Is

The Core is the smallest, most stable part of the system: a Registry (service addresses), an Event Bus (one-way announcements), and lifecycle management. It contains no business logic and provides no services itself — all services come from plugins.

## Why the UI Never Touches the Core or Registry Directly

The Registry is never handed to anyone — not a plugin, not the UI. The Core binds only the services declared in a manifest (for a plugin) or the UI manifest's `dependencies` (for the UI) at startup, and after that, calls happen directly between the caller and the service — the Core is not in the middle of the call itself.

This means: only the **UI Entry Point** — the one place with a UI manifest — ever receives bound service references, Event Bus access, and the UI's final configuration from the Core. Every component in the UI layer receives what it needs only from the Entry Point, never from the Core directly. This is why `../component-basics.md`'s base class never does its own service lookup: there is nothing for it to look up against, architecturally — a component has no channel to the Core at all.

## Event Bus — What It Guarantees and Doesn't

- **One-way, no response.** A part publishes an event to announce something happened; it doesn't wait for or expect a reply.
- **No ordering guarantee across listeners.** If multiple parts listen to the same event, don't rely on any particular order between them.
- **The Core completes every event** with `id`, `source` (forgery-proof, taken from the publishing plugin's manifest name, or the UI's manifest name for UI-published events), and `timestamp`. The publisher only supplies `type` and `data`.
- **`data` is identifying information only** — never a full entity, never sensitive data (passwords, tokens, personal info). A listener that needs full data calls the relevant service.
- This is why UI components re-fetch from a service when reacting to a domain event rather than trusting the event payload to contain everything.

## UI Action Events vs. Domain Events

Two different things share the Event Bus:

- **Domain events** (`pluginname:eventname`, past tense) announce something that happened in the system, e.g. `transactions:saved`.
- **UI action events** (`ui:eventname`) announce a lightweight UI-only change, e.g. `ui:filter-changed`. These never replace a direct service call for a core operation (saving, deleting, etc.) — the primary path for executing an operation is always a direct service call.

`ui:component-error` (see `../component-basics.md`) is a UI action event in this sense — published by the base class, declared in the UI manifest, visible to any listener on the Event Bus like any other event.

## Entry Point Lifecycle (Startup and Shutdown)

The kit implements this lifecycle in `createApplicationSetup()` (`pey.webui/shell/create-application-setup`, see `pey.webui/docs/entry-point.md`); the sequence below is the rationale behind that helper's behavior.

The Entry Point has the same two-way lifecycle contract with the Core that a plugin does, adapted for the UI's role:

- **Startup order:** the UI starts only after infrastructure and surviving application plugins are active — never before. This is why the Entry Point can safely assume every service in its `dependencies.required` (see `ui-manifest.md`) is already bound and callable by the time it runs.
- **Shutdown order:** `core:shutdown` is a **notification, not a command** — the Core then invokes the Entry Point's registered synchronous `context.onShutdown`, and that handler is where the UI performs its cleanup (releasing resources such as open connections and timers, and giving every mounted component a chance to clean up, which happens naturally if unmounting each component correctly triggers `disconnectedCallback`, per the base class's contract in `../component-basics.md`). This completes **before any plugin shuts down** — the reverse of startup order.
- The Entry Point does not need to notify the Core when it's done — the Core notifies the Entry Point through its entry point (the same notification path used for plugins), and the UI stopping is treated as complete once that path returns.
- If the Entry Point fails at startup, the Core publishes `core:ui-failed` (not `core:plugin-failed`, since the UI is not a plugin) and the system continues without a UI — this does not block `core:ready`.

This sequencing matters concretely for anyone building the Entry Point: it must implement a real shutdown handler that does more than nothing, because it is the one place responsible for making sure every mounted component's cleanup logic actually runs before the plugins it depends on go away underneath it.
