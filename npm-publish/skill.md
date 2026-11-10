---
name: bonyan-npm-publish
description: Use when asked to prepare a Bonyan-architecture plugin, pey-core, or any standalone package for publishing to npm — fixing package.json metadata, scope, the LICENSE file, and the shipped file list. Never runs the actual publish. Triggers on requests like "prepare this plugin for npm", "get this package ready to publish", "is this package publish-ready", "آماده‌ی انتشار npm کن".
---

# npm Publish Readiness (Bonyan)

This skill prepares a project that is itself an installable package — a plugin, `pey-core`, or anything laid out like `project-structure/references/standalone-package-layout.md` — for an eventual `npm publish`. It never runs `npm publish`, `npm login`, `npm adduser`, or anything else that touches real npm credentials or the registry. Preparing a package and publishing it are two separate steps; publishing stays a deliberate, explicit action the person takes themselves.

A host app is never the target of this skill: host apps are not npm packages.

## Two separate identities — never confuse them

Every Bonyan plugin already has an internal identifier: the `name` field in its `manifest.json` (`vendor.name` form, e.g. `pey.storage`), used by `bootstrap.json` and the host app's runtime registry. The npm package name in `package.json` is a different identity for a different purpose (registry distribution, not runtime lookup), and the two are allowed to differ on purpose.

**Never touch, while doing this work:**
- `manifest.json`'s own `name` field
- any `bootstrap.json` plugin registration
- a plugin's folder name under `src/plugins/<vendor.name>/`

Only `package.json`'s publish-facing fields change.

## Checklist — `package.json`

- **`name`**: must carry the project's real npm scope (e.g. `@pey/storage`). Read the scope from a sibling package that already uses it or that the person already decided on — never invent a new scope yourself. If no scope has ever been established for this family of packages, stop and ask instead of picking one.
- **`version`**: leave untouched. Version numbers are `changelog/references/release.md`'s job, not this skill's.
- **`private`**: must not be `true`, and the key should usually be absent entirely — a private package can never be published.
- **`license`**: must name a real license (e.g. `"MIT"`) **and** a matching `LICENSE` file must exist at the project root with real text (actual copyright holder and year, not a placeholder). If the field is set but the file is missing, write the file with that license's standard text. If neither the field nor a license decision exists, stop and ask — choosing a license is not this skill's call.
- **`publishConfig`**: must include `{ "access": "public" }` whenever `name` is scoped and the package is meant to be public. Without it, npm treats a scoped package as private and refuses to publish it on a free account.
- **`repository`, `homepage`, `bugs`**: fill from the project's actual git remote (`git remote get-url origin`) — never a guessed or hand-typed URL.
- **`files`**: must list exactly the project's real shipped content — entry point, `manifest.json` (if a plugin), its `lib/`/`src/`, `README.md`, `CHANGELOG.md`, `LICENSE`, `docs/`. Never tests, never `node_modules`, never internal planning or report artifacts.
- **`keywords`**: the package's own domain words plus whatever shared family keywords the project already uses elsewhere (e.g. matching GitHub topics, if known). Don't invent a long, unrelated list.
- **`engines`**: keep whatever Node floor the project already declares or tests against; don't change it silently.

`dependencies` and `devDependencies` are out of scope for this skill — that is `project-structure`'s and `plugin-implementation`'s job.

## Verify before reporting done

Run `npm pack --dry-run` and read the printed file list. Compare it against `files`: anything unexpected (a stray `.env`, a leaked `tests/` folder, a report file) means `files` is wrong — fix it and re-run until the list matches exactly what should ship.

## Never

- Run `npm publish`, `npm login`, `npm adduser`, or any command that touches real npm credentials or the registry.
- Touch `manifest.json`'s plugin name, a `bootstrap.json` entry, or a plugin's folder name under `src/plugins/<vendor.name>/`.
- Change the `version` field, or `dependencies`/`devDependencies`.
- Invent an npm scope, a license choice, or a repository URL — read them from the project, or stop and ask.

