# Standalone Package Layout

For a repository that **is** a single Bonyan/Pey package — one reusable plugin, an adapter, the core (`pey-core`), or the UI kit (`pey.webui`) — not a host app that assembles several of these into a running project.

Real example already in this ecosystem (`pey-core`'s own repository):

```
<package>/
├── package.json
├── manifest.json      — identity for a plugin/adapter package; the core carries one for itself too
├── index.js            — the package's public entry point (what a consuming project imports)
├── src/                 — implementation files, flat or grouped by topic; no host/, plugins/, or ui/
├── tests/
├── docs/
├── README.md
├── CHANGELOG.md
└── LICENSE
```

There is no `bootstrap.json`, no `index.html`, and no `src/host/` — this package is *consumed by* a host app's `bootstrap.json` and import map; it does not have its own. A host app that depends on this package declares it as a real `package.json` dependency, per `project-structure/SKILL.md`'s "Dependencies rule" and `host-app-layout.md`.

## This is also the layout for a plugin living inside a host app

A plugin under a host app's `src/plugins/<vendor.name>/` follows this exact layout (`manifest.json`, `index.js`, `src/`, `tests/`, `docs/`, its own `README.md`/`CHANGELOG.md`) — a plugin is portable by design, so it is self-contained wherever it lives, whether that is its own repository, an installed `node_modules/` dependency, or a folder inside a host app. The only difference inside a host app: it does not need its own `LICENSE`, since the host app's root `LICENSE` already covers it.

## If the package also ships its own demo or docs site

Some packages additionally deploy a small example/demo as a static site. That demo is itself a host app in miniature and should follow `references/host-app-layout.md` inside its own clearly separate subfolder (e.g. `demo/`), never mixed into the package's own `src/`.
