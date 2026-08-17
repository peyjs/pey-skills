# UI Map — Index

This file is the entry point to the project's UI Map. The UI Map is split across several small files instead of one long document, because a single project-wide file grows indefinitely as components are added and becomes exactly the kind of document nobody actually reads or keeps current.

**There is exactly one UI Map per project.** Keep it under `ui/docs/` (see `ui-implementation/references/file-layout.md`), sitting alongside the rest of the UI source. (This skill's own reference files below are its internal organization under this `references/` folder — the internal folder name doesn't need to match the project's actual `ui/docs/` folder name; what matters is that the project folder itself is named `docs`.)

## What's in Each File

| File | Covers | Does NOT cover |
|---|---|---|
| `component-index.md` | Which components/pages exist, where, and the route table | Any individual component's API/events/state — that's the component's own doc, per `component-documentation.md` |
| `entry-point.md` | What the Entry Point does (startup, shutdown, route registry, config distribution) | The base class's behavior, or any single component's dependencies |
| `shared-infrastructure.md` | The base class (consumer-facing summary), the shared state store's live key inventory, design tokens in use, i18n key convention | The fixed technical contracts themselves (those live in `ui-implementation/references/component-basics.md`, `ui-implementation/references/shared-state.md`, `ui-implementation/references/styling-and-assets.md`, `ui-implementation/references/forms-and-i18n.md` — this file only tracks what the project actually uses) |
| `decisions.md` | *Why* a product/UX decision about the UI was made, and what alternatives were considered | *What* the UI currently does technically — that's every other file here |
| `user-flows.md` | Multi-page user journeys and the reasoning behind how they're structured | Single-page/single-component detail — link out to `component-index.md` instead of re-describing a page here |

## When to Update Which File

- Added/removed/renamed a component or page → `component-index.md`.
- Changed what the Entry Point requests or does at boot/shutdown → `entry-point.md`.
- Added a new design token, a new shared-state key, or changed what the base class provides → `shared-infrastructure.md`.
- Made a UX/product call that shaped how something in the UI works, especially if it wasn't the "obvious" choice → `decisions.md`.
- Built or changed a flow spanning more than one page → `user-flows.md`.

If a change touches more than one of these, update all the relevant files in the same pass — don't leave one out because the task felt like it was "mainly" about a different file.
