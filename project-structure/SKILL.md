---
name: bonyan-project-structure
description: Use when scaffolding a brand-new Bonyan-architecture project from scratch, or when an existing project's folder layout seems to deviate from the standard layout and needs to be assessed or aligned. Triggers on requests like "set up a new Bonyan project", "where should this file go", "is this project structured correctly", "align the folder structure".
---

# Project Structure (Bonyan)

Decides **where things live** in a Bonyan-architecture repository — folder names, their nesting, and how framework dependencies are obtained. It does not decide what goes inside a plugin (`plugin-implementation/SKILL.md`), inside the UI layer (`ui-implementation/SKILL.md`), or inside the release build script (`release-build/SKILL.md`) — this skill only gives those skills a consistent place to put their work.

## Facts come from the project, never from memory

Before scaffolding or judging a structure, check what kind of repository this actually is (see below), and if it already has files, read its current layout before proposing anything. Never invent a folder name that isn't in this skill's references.

## Step 1 — identify the project kind

- **Host app**: a project that deploys an actual running application — has, or will have, `bootstrap.json`, `index.html`, several plugins, and a UI. Read `references/host-app-layout.md`.
- **Standalone package**: the repository itself IS one plugin, one adapter, the core (`pey-core`), or the UI kit (`pey.webui`) — consumed by host apps, not a running application on its own. Read `references/standalone-package-layout.md`.

If the task doesn't make this clear, ask rather than guess.

## Dependencies rule (applies to both kinds)

- `pey-core`, every `pey.*` plugin, and `pey.webui` are always a real `package.json` dependency (an npm registry version or a pinned `github:` reference) — never a hand-copied source tree living inside this project, and never a separate "vendor" install tree.
- `npm ci`/`npm install` populates the standard `node_modules/` folder; nothing renames it, relocates it, or installs a second copy elsewhere. `node_modules/` is always git-ignored.
- In development, native ESM plus the import map in `index.html` reference packages directly under `./node_modules/...`. Turning this into a production bundle is `release-build/SKILL.md`'s job, not this skill's — this skill only makes sure the dependency is declared and installed in the normal place so that skill has something real to bundle.

## When to touch it

- **New project**: before any plugin or UI code is written — this is the first step, ahead of `plugin-implementation-plan/SKILL.md`.
- **Existing project**: when asked to check or align its structure. Default to **reporting deviations only** (expected path vs. actual path, one line each) — moving files in a working project risks breaking relative imports and the import map, so apply a move only when explicitly asked. When you do move something, update every relative import and import-map entry that pointed at the old path, then run the project's existing tests before calling the task done.

## Never

- Invent a different name or location for `node_modules`, or create a parallel/isolated install tree for `pey-core`, any `pey.*` plugin, `pey.webui`, or third-party browser packages.
- Hand-copy `pey-core`, a `pey.*` plugin, or `pey.webui` into the project's own source tree instead of declaring it as a real dependency.
- Move or rename files in an existing project without explicit permission.
- Decide the content that goes inside `src/host/*`, a plugin folder, or `src/ui/*` — that belongs to `plugin-implementation/SKILL.md` and `ui-implementation/SKILL.md`, not this skill.
