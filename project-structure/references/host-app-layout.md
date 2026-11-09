# Host App Layout

The canonical layout for a Bonyan-architecture **host app**: a project that assembles plugins and a UI into an actual running application and deploys it.

```
<project>/
├── .github/workflows/        — CI; the release-build step itself is release-build/SKILL.md's concern
├── bootstrap.json
├── index.html
├── package.json
├── package-lock.json
├── LICENSE
├── README.md
├── CHANGELOG.md
├── docs/                      — this *host app's own* documentation, as a whole product; not any plugin's
│   ├── introduction.md        — per introduction/SKILL.md
│   └── ...                    — any number of other project-specific documents (architecture notes,
│                                 decisions log, user guide, etc.) — there is no fixed list; a host app
│                                 accumulates whatever documents its own product actually needs here
├── scripts/                  — build/CI tooling; never shipped to the browser, so never under src/
└── src/
    ├── host/                 — the "Runtime Host" (pey-core's own term), exactly:
    │   ├── main.js            — boots PeyCore
    │   ├── config-source.js   — ConfigSource
    │   ├── env-source.js      — EnvSource
    │   └── module-loader.js   — ModuleLoader (loadManifest/loadEntry)
    ├── plugins/
    │   └── <vendor.name>/     — one folder per plugin, each laid out exactly like
    │                            references/standalone-package-layout.md (manifest.json, index.js,
    │                            src/, tests/, docs/, its own README.md and CHANGELOG.md) — a plugin is
    │                            portable by design ("moves to another project unchanged"), so it carries
    │                            its own docs/README/changelog with it instead of sharing the host's root
    │                            ones. The one exception is LICENSE: a plugin living inside a host app's
    │                            repo does not need its own — it's covered by the host app's root LICENSE;
    │                            only a plugin published as its own separate repository needs one.
    ├── ui/                    — the project's own UI: composes pey.webui, never forks it
    │   ├── components/
    │   ├── pages/
    │   ├── assets/
    │   └── i18n/
    │                            (content is ui-implementation/SKILL.md's contract)
    └── tests/                 — host-level / UI-level / integration tests not specific to one plugin
```

`node_modules/` sits at the repository root from a normal `npm install`; it is not pictured above because it is never part of the committed tree (always git-ignored) — see `SKILL.md`'s "Dependencies rule" for what must live in it.

## Root `docs/` is the host app's own, never a plugin's

Every plugin carries its own complete `docs/` (plus its own `README.md` and `CHANGELOG.md`) inside `src/plugins/<name>/`, written per `plugin-documentation/SKILL.md`, `readme/SKILL.md`, and `changelog/SKILL.md` applied *to that plugin folder*, not to the project root. The root `docs/` is a separate, second application of `introduction/SKILL.md` (and any other project-specific document) describing the host app as a whole product — its own introduction, not a copy of or a substitute for any plugin's reference.

## Why `src/host/`, not `src/app/`

`pey-core`'s own documentation names this concept the **Runtime Host** and describes its contract as exactly three injected sources — `configSource`, `envSource`, `moduleLoader` — plus whatever constructs `PeyCore` and starts it. The folder name and its four files mirror that contract one-to-one, so there is never a question of where a given responsibility lives: the file name already says which injected source it is.

## Why no isolated vendor tree

A separate install tree for browser-safe dependencies only ever existed to keep `devDependencies` out of what ships to users. `release-build/SKILL.md`'s bundler only includes what is actually reachable from a real entry point, so that isolation has no remaining benefit — it only adds a second lockfile and a second install step that can silently drift from the real upstream package. That drift has already caused a real problem in this ecosystem: a hand-copied `pey-core` fell behind the published version and kept a performance bug the published version no longer had.

## Boundary with other skills

This document only says **where** things go. What goes inside a plugin folder is `plugin-implementation/SKILL.md`'s contract; what goes inside `src/ui/` is `ui-implementation/SKILL.md`'s contract; how `scripts/` turns this into a release artifact is `release-build/SKILL.md`'s contract.
