# UI Map — Component and Page Index

Covers: which components/pages exist, where their files are, and the route table. Does not cover any individual component's API/events/state — see that component's own doc per `component-documentation.md`.

## Component and Page Index

A flat table: name, whether it's a regular component or page-level, one-line purpose, and a link to its documentation file if it has one.

A component or page with no dedicated doc file (per `component-documentation.md`'s default/exception rule — some small components only carry a file-header comment, per `bonyan-ui-coding-standard`) still gets a row here; link to its source file itself instead of a doc file in that case, so this index stays complete even for entries that aren't individually documented in detail. **This applies identically to page components** — a small page with no dedicated doc file follows the exact same fallback, with no special-casing just because it's a page rather than a regular component.

| Name | Kind | Purpose | Doc / Source |
|---|---|---|---|
| _e.g. `app-example-widget`_ | Component | _one line_ | _link_ |
| _e.g. `app-page-accounts-detail`_ | Page | _one line_ | _link_ |

## Route Table

For every route the Entry Point's route registry maps (see `bonyan-ui/references/composition.md`): the route pattern, which page component handles it, and whether it's lazily loaded (per `bonyan-ui/references/composition.md`'s Lazy Loading section).

| Route Pattern | Page Component | Lazy-Loaded? |
|---|---|---|
| _e.g. `/accounts/:id`_ | _e.g. `app-page-accounts-detail`_ | _yes/no_ |
