---
name: bonyan-ui-documentation
description: Use when writing or updating documentation for a Bonyan-architecture UI component, page component, the UI Entry Point, the shared base class, the UI-local shared state store, design tokens, product/UX decisions behind the UI, user flows, or the project-wide UI map. Triggers on requests like "document this component", "write the README for this page", "document the UI layer", "what components does this UI have", "why did we build it this way", "map out the onboarding flow".
---

# UI Documentation

This skill covers two different levels of documentation — pick the right one, they are not interchangeable:

- **Component-level:** one component or page, filled out after it's implemented and tested. → `references/component-documentation.md`
- **Project-level (the "UI Map"):** the whole UI layer as it exists today, plus *why* it looks the way it does — not just what exists, but the reasoning and user-facing flows behind it. There is exactly one UI Map per project, kept current as the project grows, split across several small files rather than one long document — open only the file relevant to what you're updating.

## UI Map — Reference Files

| When you're about to... | Read/update this file |
|---|---|
| Get oriented, or find which file to update | `references/ui-map/README.md` |
| Add, remove, or rename a component/page, or update the route table | `references/ui-map/component-index.md` |
| Change what the Entry Point does at startup/shutdown, or its dependencies | `references/ui-map/entry-point.md` |
| Change the shared base class's behavior, add a shared-state key, or add/change a design token | `references/ui-map/shared-infrastructure.md` |
| Record *why* a UX or product decision was made for the UI | `references/ui-map/decisions.md` |
| Describe a multi-page user journey and why it's structured that way | `references/ui-map/user-flows.md` |

Each UI Map file says at its top what it covers and, just as importantly, what it explicitly does not — so content doesn't end up duplicated across two of them.

## General Rule

Keep every section factual and short; these are read by other developers/models integrating or modifying this code later, not written once and forgotten. A UI Map file that hasn't been touched in a long time despite the UI changing underneath it is worse than no documentation — it actively misleads. Updating the relevant UI Map file is part of finishing a UI change, not an optional follow-up.
