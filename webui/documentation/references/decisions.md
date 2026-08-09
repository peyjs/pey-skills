# UI Map — Decisions

Covers: *why* a product or UX decision about the UI was made — not what the UI currently does technically (every other UI Map file covers that). This is the file that answers "why does it work this way?" months or years later, when the person who'd remember isn't in the conversation anymore. Since this project is built incrementally by AI models across many sessions, without this file the reasoning behind a choice exists only in a past chat transcript, which is effectively lost.

## When to Add an Entry

Add an entry whenever a decision was made that:
- wasn't the "obvious" default (e.g. choosing to show a confirmation dialog before a destructive action, choosing to hide a feature behind a flag, choosing a specific empty-state message over a generic one),
- could reasonably have gone another way, and a future session might otherwise "fix" it back to the more obvious option without knowing it was already considered and rejected,
- affects more than one component, or shapes a pattern other components are expected to follow.

Don't log routine technical choices already covered by `bonyan-ui`'s fixed conventions (e.g. "we used Shadow DOM" — that's just following the architecture, not a project decision) or by `shared-infrastructure.md` (a token or key's existence, which is a fact, not a decision needing justification).

## Entry Template

Copy this for each new entry, append to the bottom of this file (chronological order, oldest first, so the log itself reads as project history). **Every entry gets a unique, stable ID** — `UI-001`, `UI-002`, and so on, incrementing, never reused even if a decision is later reversed (a reversal is a new entry that references the old ID, not a renumbering). Other files (`user-flows.md`, a component's own doc) reference a decision by this ID, not by its title, since titles can be edited without breaking a reference but an ID reference must stay stable.

```
### UI-<NNN>: <short title>
**Date:** <YYYY-MM-DD>
**Decision:** <one or two sentences — what was decided>
**Context:** <why this came up — what problem or question prompted it>
**Alternatives considered:** <what else was on the table, briefly, and why it wasn't chosen>
**Consequences:** <what this means going forward — e.g. "every destructive action in this project must follow this same confirmation pattern">
**Related:** <link to the component doc(s) or user flow this touches, if any>
```

## Log

_(entries go below, oldest first)_
