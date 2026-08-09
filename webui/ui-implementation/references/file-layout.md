# File and Folder Layout

Fixed structure for the UI directory. For a project meant to grow over years with many independent contributions, an agreed folder layout matters as much as the code conventions — read this before creating any new UI file, not just before writing its contents.

## Fixed Folder Structure

```
ui/
├── manifest.json              # the UI manifest (see architecture-context/ui-manifest.md)
├── index.js                   # the Entry Point — the only file the Core loads directly
├── shell/
│   └── app-shell.js           # the top-level shell component the Entry Point mounts at startup
│                               # (see composition.md, Component Ownership) — a regular component
│                               # per component-basics.md, just always the first one mounted and
│                               # never unmounted for the lifetime of the running UI
├── base/
│   └── bonyan-element.js      # the one shared base class
├── shared-state.js            # the UI-local shared state store (see shared-state.md)
├── tokens.css                 # global design tokens (see styling-and-assets.md)
├── assets/
│   ├── icons.svg               # the shared SVG sprite (default icon strategy)
│   ├── fonts/
│   └── images/
├── i18n/
│   └── ...                    # translation lookups, per forms-and-i18n.md
├── docs/                      # the project-wide UI Map (see bonyan-ui-documentation's
│                               # references/ui-map/README.md) — README.md, component-index.md,
│                               # entry-point.md, shared-infrastructure.md, decisions.md,
│                               # user-flows.md. One shared folder for the whole project, distinct
│                               # from a single component's own .md file (see below), which stays
│                               # colocated with that component instead of living here.
├── components/                # non-page, reusable components
│   └── <component-name>/
│       ├── <component-name>.js
│       ├── <component-name>.css   # if not inlined in the JS template
│       └── <component-name>.md    # this component's own doc, if it has one — see
│                                   # bonyan-ui-documentation's component-documentation.md for
│                                   # when one is warranted; small components may skip this and
│                                   # rely on the file-header comment instead
└── pages/                     # page-level components, one per route
    └── <page-name>/
        ├── <page-name>.js
        ├── <page-name>.css
        └── <page-name>.md         # same rule as a component's .md file, above

tests/                          # one shared tests/ folder at the project root, NOT colocated with
│                                # source (see bonyan-ui-testing-standard) — mirrors ui/components/
│                                # and ui/pages/ one-for-one:
├── components/
│   └── <component-name>/
│       └── <component-name>.test.js
├── pages/
│   └── <page-name>/
│       └── <page-name>.test.js
└── base/
    └── bonyan-element.test.js  # the shared base class's own dedicated test suite (see
                                  # bonyan-ui/references/testing.md)
```

## Naming Rules

- **One component = one folder = one file pair** (`.js` + optional `.css`), folder and file share the same name.
- **Folder/file names:** lowercase, hyphens, descriptive — same rule as plugin/event naming in `architecture-context/naming-conventions.md` (no abbreviations).
- **Custom element tag names:** `app-<component-name>` for regular components, `app-page-<page-name>` for page-level components — replace `app` with the project's actual short prefix, decided once and used everywhere (mirrors a plugin's `vendor` prefix, see `architecture-context/naming-conventions.md`). This distinguishes page components from regular ones at a glance, consistent with `composition.md`'s rule that only the Entry Point may mount a page-level component.
- **Route registry entries** (in the Entry Point) reference `pages/<page-name>/<page-name>.js` — the loader function for lazy loading (see `composition.md`) always points here, never into `components/`.
- **`docs/` is the project-wide UI Map's folder name**, chosen specifically because it matches the common convention of a `docs` folder signaling "documentation lives here" — not a Bonyan-specific term. It holds only the project-level files listed above; it never holds a single component's own `.md` file, which stays colocated with that component instead (see the tree above).
- **`tests/` is one shared folder at the project root**, not colocated with source, mirroring `bonyan-ui-testing-standard`'s file/structure rule — every component/page test file has a matching path under `tests/` one level down from wherever its source lives under `ui/`.

## Why This Is Fixed, Not Flexible

An earlier draft of this skill left folder layout open. That was a mistake for a project meant to run for years across many independent AI sessions: without a fixed layout, each session could reasonably invent its own structure, and consistency would degrade the same way it would if templates or the base class weren't fixed. This layout is authoritative — deviate from it only if this file is updated first, the same discipline `component-basics.md` and the rest of this skill apply to every other convention.
