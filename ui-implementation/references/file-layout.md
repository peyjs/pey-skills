# File and Folder Layout

What the consuming application owns and creates. The `pey.webui` kit ships its entire internal structure — `base/`, `shell/`, `routing/`, `i18n/`, `assets/`, `docs/`, `shared-state.js`, `tokens.css` — and those folders are part of the kit itself, never created or modified by the application; their exact structure is in `pey.webui/docs/overview.md`. What the application must get right is the small fixed set below, so that files written by different sessions land in the same place and the Core finds them.

## Consuming The Kit

- **Install** from GitHub, not the npm registry — see `pey.webui/README.md` (Installation): `git clone <repository-url> pey.webui`, then `npm install /path/to/pey.webui` inside the consuming application.
- **Copy the application templates** into your own UI root: `templates/application-manifest.json` → `ui/manifest.json` and `templates/application-entry-point.js` → `ui/index.js`, then adapt them (pages, routes, dependencies, events, translations).
- `ui/manifest.json` and `ui/index.js` are the application UI artifacts the Core loads for UI startup.

## Product Components And Pages

- Product components live under `ui/components/<component-name>/`, product pages under `ui/pages/<page-name>/` — one folder per component/page:

```
ui/
├── manifest.json            # copied from templates/application-manifest.json
├── index.js                 # copied from templates/application-entry-point.js
├── components/<name>/       # <name>.js (+ <name>.css if not inlined in the JS template)
└── pages/<page-name>/       # <page-name>.js (+ <page-name>.css)
```

- **File/folder names:** lowercase, hyphens, descriptive — same rule as `architecture-context/naming-conventions.md` (no abbreviations).
- **Custom element tags:** the product prefix is chosen once by the application (e.g. `app-`): `app-<component-name>` for components, `app-page-<page-name>` for pages. The `pey-` prefix belongs to the kit's own components (e.g. `pey-page-fallback`) — never reuse it for product code.
- **Route registry entries** (in `ui/index.js`) reference `pages/<page-name>/<page-name>.js` — the loader function always points here, never into `components/`.

## Tests

One shared `tests/` folder at the application root, NOT colocated with source (see `bonyan-ui-testing-standard`), mirroring `ui/components/` and `ui/pages/` one-for-one:

```
tests/
├── components/<component-name>/<component-name>.test.js
└── pages/<page-name>/<page-name>.test.js
```

## What The Application Does NOT Build

The kit's own folders — `base/`, `shell/`, `routing/`, `i18n/`, `assets/`, `docs/`, plus `shared-state.js` and `tokens.css` — are part of `pey.webui` itself; the application neither creates nor modifies them, and never reimplements what they ship (the base class, the shared-state store, page host/loader/fallback, i18n, tokens, the icon sprite). Deviate from this layout only if this file is updated first.