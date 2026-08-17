# UI Map — Shared Infrastructure

Covers: a consumer-facing summary of the shared base class, the *live inventory* of what's actually in use for shared state, design tokens, and i18n — not the fixed technical contracts themselves, which live in `bonyan-ui`'s reference files and remain authoritative. This file tracks what the project has actually built on top of those contracts, and goes stale fastest if not updated alongside the code — treat updating it as part of the change, not a follow-up.

## Shared Base Class
Brief description of what every component gets "for free" (reference injection via `connect()`, the render cycle, delegated event handling, Event Bus subscription lifecycle, the error boundary) — a short consumer-facing summary, not a re-explanation of `ui-implementation/references/component-basics.md`, which remains the authority on its exact behavior. Link there for details. Note the file location (`ui/base/bonyan-element.js`, per `ui-implementation/references/file-layout.md`) and anything project-specific added on top of the baseline contract, if any.

## App Shell
Where the top-level shell lives (`ui/shell/app-shell.js`, per `ui-implementation/references/file-layout.md`) and what it's responsible for in this project specifically (layout chrome, navigation, whatever wraps every page).

## UI-Local Shared State Store — Live Key Inventory
Which keys are actually in use across the project, what each one means, and which components read/write each. This is a live inventory, not a repeat of `ui-implementation/references/shared-state.md`'s API description (link there for the fixed `get`/`set`/`subscribe` contract and its equality/immutability rules).

| Key | Meaning | Read by | Written by |
|---|---|---|---|
| _e.g. `theme`_ | _e.g. current color theme_ | _components..._ | _component/setting that changes it_ |

## Design Tokens — Live Inventory
The actual tokens defined in this project's global tokens file (`ui/tokens.css`, per `ui-implementation/references/file-layout.md`) — names and what each represents (e.g. `--color-primary`, `--spacing-md`) — so a new component can be built using existing tokens instead of a session inventing a near-duplicate one.

**This table is an index of names and intent only — it is never the source of truth for a token's actual value.** `tokens.css` itself is always the authority for the real value; if this table and `tokens.css` ever disagree on a value, `tokens.css` wins and this table is simply out of date and needs correcting. Do not copy actual values (hex codes, pixel sizes) into this table — copying them here is exactly what causes the drift this note is meant to prevent. Link to `tokens.css` for anyone who needs the real value.

| Token | Represents |
|---|---|
| _e.g. `--color-primary`_ | _e.g. primary brand color, used on primary actions_ |

## i18n Key Convention
If the project uses the translation lookup described in `ui-implementation/references/forms-and-i18n.md`, note where the translation source lives and the key-naming convention in use (e.g. `page.section.label`), so new keys stay consistent with existing ones. Not a full key listing — that belongs in the translation source itself, not here.
