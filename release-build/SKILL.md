---
name: bonyan-release-build
description: Use when a Bonyan-architecture project (typically a host app) needs a release/publish build set up from scratch for a static host (e.g. GitHub Pages), or when an existing build script already exists and must be created, fixed, or reviewed so the deployed app does not ship a one-file-per-package ESM waterfall. Triggers on requests like "set up the build for GitHub Pages", "package this for release", "the deployed site takes long to load", "why are there 100+ requests", "review the build script".
---

# Release Build (Bonyan)

Turns a Bonyan-architecture project's dev-mode native-ESM source (import map + raw files, no build step) into a correct, deployable build for a static host. Default behavior is to **do the work** — create the build script if none exists, or fix it if one exists and is wrong — the same way `plugin-implementation/SKILL.md` implements plugin code directly rather than only describing what to implement. Only stop short of changing files when the person is clearly asking for an explanation or a review opinion, not a build.

## Standard tool: esbuild, as a transport-only step

The Bonyan architecture is build-tool-free by design (`ui-implementation/SKILL.md`, rule 13: "no build tools... what's written in the file is what runs in the browser"). That rule governs **authoring**: no bundler-only syntax, no dev server transforming files on the fly, no framework compiling the source into something else. It does not forbid a **transport-only** step that repackages the exact same, unmodified source into fewer files purely for network efficiency at deploy time — nothing about how the code is written or what it does changes; only how many HTTP round trips it takes to deliver it.

Within that scope, this skill always uses **esbuild** (already the precedent in this ecosystem), for a reason beyond consistency: esbuild performs this exact job — merge entries, split only what you tell it to, no plugins, no loader configuration, no dev server — without the on-the-fly transform pipeline or dev-server concept that something like Vite would introduce, which would be a much closer brush with "a build tool changes what runs." Add it only as a `devDependency`; it never reaches the browser or the authored source.

- A dev run still uses plain static serving + the import map — unbundled, unmodified native ESM, exactly as `ui-implementation` requires. `esbuild` only runs for the release artifact.
- If a project already deliberately uses a different bundler, do not silently replace it — that is an existing architectural decision, not a gap. Apply this skill's principles (entry classification, eager/lazy split, file-count targets) within that tool's own equivalent configuration, and only propose switching tools if asked.

## Strict scope: only the project's own build surface

This skill creates or edits only files the project owns and is allowed to change: `index.html` (its import map), its own build script (e.g. `scripts/build-*.mjs`), and its own deploy workflow. It never inspects or edits `pey-core` or any `pey.*` framework plugin, whether that code is an external dependency or vendored inside the project's own tree — same rule as `pey.webui` in `ui-implementation/SKILL.md`: framework code is never touched, only the project's own code is. If initial-load slowness seems to originate inside the framework itself rather than this project's own build, say so in one sentence and stop there — nothing to implement.

## Facts come from the project, never from memory

Never decide what to bundle from reading code alone. Open the real project and determine, for every import-map specifier in `index.html`, whether it is reached by a static `import` somewhere, by a dynamic `import()` with a literal string, or both. If a build step already exists, actually run it and count the real output (`find dist -name "*.js" | wc -l`, `du -sh dist`) before judging it correct or broken.

## Workflow

1. **Determine the starting point.** Does a build script and a deploy workflow already exist?
   - **No build step (greenfield)**: the project ships raw `index.html` + import map + `src/`/`node_modules` directly. You are creating the build from nothing.
   - **Existing build step**: read it fully before changing anything. Preserve any behavior not related to bundling correctness (asset copying, versioning, test running in CI, etc.).
2. **Map the import graph.** Extract every specifier from the import map. For each one, grep the project's real source:
   - `grep -rln "from ['\"]<specifier>['\"]" src` → reached by a static import somewhere.
   - `grep -rn "import(['\"]<specifier>['\"])" src` → reached by a dynamic import with a literal string.
   A specifier reached by a static import anywhere is **eager**, even if it is also dynamically imported elsewhere. A specifier reached **only** by a literal dynamic import is **lazy**.
3. **Build or fix the esbuild configuration** (or the project's existing tool's equivalent, per the exception above) so that:
   - `entryPoints` are the app's real entry point(s) plus only the lazy specifiers found above — never every import-map target indiscriminately:
     ```js
     await build({
       entryPoints: [appEntry, ...lazySpecifierPaths],
       outdir: 'dist',
       bundle: true,       // everything reached by a static import merges in
       splitting: true,    // shared code between lazy chunks is factored out, not duplicated
       format: 'esm',
       minify: true,
       sourcemap: false,   // or 'external' with no reference shipped to users
     });
     ```
   - The import map shipped in the built `index.html` is trimmed to contain only the lazy specifiers — the eager ones no longer need a browser-visible mapping.
   - Dev-only files (tests, scripts, raw docs sources) are not part of what the browser can fetch in production, unless the project has a specific reason to ship them.
   - The deploy workflow uploads the **built** output, not the raw source tree.
   - `esbuild` is added only to `devDependencies`, never a runtime dependency.
4. **Run the build for real and validate** against `references/performance-standard.md`: eager-path JS file count ≤ 5, a real measurement if the site is reachable (Network tab / Lighhouse request count ≤ 25). If a target is not met, keep refining the entry classification — do not relax the target.
5. **Run the project's existing tests** afterward; a bundling change must not alter runtime behavior.
6. **Close out the task** the same way any Bonyan implementation task closes:
   - Changelog entry per `changelog/SKILL.md`.
   - `readme/SKILL.md` only if the install/build command changed.
   - One-line commit message per `git-workflow/SKILL.md`; run no git command unless asked.

## When only a review is asked for, not a build

If the person asks "why is this slow" or "review the build" without asking you to fix it, do steps 1–4 as a read-only assessment and report findings (exact file:line, exact real numbers from actually running the build) instead of editing files. Resume to steps 5–6 only once asked to apply the fix.

## Architecture Feedback Clause

If you cannot tell whether a specifier is reached eagerly or only lazily (e.g. it is imported through a runtime-constructed path you cannot grep for, such as a plugin entry resolved from `bootstrap.json`), do not guess — leave it as its own entry (safe default: more chunks, never a broken import) and say so in the closing report, rather than silently merging something that might only be used lazily.

## Never

- Inspect, question, or edit `pey-core` or any `pey.*` plugin's own source, vendored or not — out of scope, full stop.
- Treat the import map as the bundling boundary; it only resolves specifiers for the browser, it does not decide what should be a separate chunk.
- Claim a file count or request count without having actually run the build or inspected the real deploy output.
- Merge a specifier into the eager bundle when you are not sure it is never lazy-only.
- Change behavior unrelated to the bundling/build task (plugin logic, UI behavior, config semantics) while doing this work.
