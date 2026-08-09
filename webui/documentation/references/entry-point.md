# UI Map — UI Entry Point

Covers: what the project's actual UI Entry Point does. Not a component, so it doesn't use `component-documentation.md`'s template. Does not cover the base class's behavior or any individual component's dependencies — see `shared-infrastructure.md` and `component-index.md`/each component's own doc respectively.

## Location
Where the Entry Point's file lives — per `bonyan-ui/references/file-layout.md`, this is `ui/index.js`.

## Dependencies Requested from the Core
A plain-language walk-through of what this project's actual UI manifest declares in `dependencies` — not a duplicate of `bonyan-ui/references/architecture-context/ui-manifest.md` itself (link there for the full contract), just a project-specific summary of *which* services/config this Entry Point actually asked for and roughly why.

- **Required:** ...
- **Optional:** ...
- **Config keys read:** ...

## Startup Behavior
What the Entry Point does when the Core starts it: mounting the shell (see `shared-infrastructure.md` for what the shell is), setting up the route registry (see `component-index.md`'s Route Table for what's in it), and any other startup-specific behavior specific to this project.

## Shutdown Behavior
What happens on `core:shutdown`, per `bonyan-ui/references/architecture-context/core-boundaries.md`'s Entry Point Lifecycle section — resources released, components given the chance to clean up. Note anything project-specific here (e.g. a particular resource this Entry Point holds directly and must release itself).

## Route-to-Loader Wiring
How `mountForRoute(route)` (see `bonyan-ui/references/composition.md`) is implemented in this project — e.g. a simple lookup table, or something more elaborate — only if there's something project-specific worth noting beyond what `composition.md` already documents as the general pattern.
