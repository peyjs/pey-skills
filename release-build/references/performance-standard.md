# Initial-Load Performance Standard

Detail behind `SKILL.md`: principles, measurable acceptance numbers, and reference code patterns that any release build produced or fixed by this skill must satisfy, for a project (plugin, UI, or host app) that ships native ESM to a static host (e.g. GitHub Pages).

This document covers only code the project owns and is allowed to change: `index.html`, its own build script, and its own deploy workflow. The core (`pey-core`) and any `pey.*` plugin are out of scope — as `SKILL.md` states, they are never reviewed or edited.

## Vocabulary

- **Eager path**: everything needed for first paint; without it the user sees nothing or cannot start the main task.
- **Lazy path**: loaded only in response to a runtime action or condition (a highlight language only when it appears in the text, a page not yet routed to).
- **Static vs dynamic import**: `import x from 'y'` is analyzable at build time. `import('y')` is analyzable and chunkable at build time only when `'y'` is a literal string (not a runtime-constructed value, e.g. `new URL(...)`).
- **Waterfall**: the serial chain where the browser cannot discover an import until the file containing it has been downloaded and parsed; the depth of this chain, not total bytes, is the main driver of latency.

## Principles

### Principle 1 — Classify eager/lazy by real usage, not by the import map
The import map only resolves specifiers for the browser; it is not the bundling boundary. A module gets its own chunk only if it is reached **exclusively** by a dynamic `import()` with a literal string. Anything reached by a static import anywhere must be merged into the eager bundle that statically imports it, even if it is also listed in the import map.

### Principle 2 — The eager path must reduce to a small, fixed file count
Target: the initial load (through the end of `boot()` or equivalent) resolves to **5 or fewer JavaScript files**, regardless of how many npm packages sit behind it. The import map shipped to production should only contain specifiers that are genuine dynamic-`import()` targets.

### Principle 3 — No production cost for dev-only tooling
Sourcemaps, docs, tests, and dev-only tooling must not be fetched by the browser without need.

## Acceptance criteria

| Metric | Threshold |
|---|---|
| Total requests on initial load | ≤ 25 |
| JS files on the eager path | ≤ 5 |
| Time to interactive (cold cache) | ≤ 3s on a typical network |

These numbers are a starting point; what matters more than the exact figure is having an explicit, measured threshold (Network tab or Lighthouse) checked before every release. A report should always separate how much slowness is architectural (this project's own build) versus likely network/CDN (e.g. the user's network path to the hosting CDN) — changing this project's build does not fix the latter.

## Choosing bundler entries correctly

- Main entry = the app's real entry point (e.g. `main.js`).
- Extra entries = only literal strings that are genuinely dynamic-`import()` targets in the project's own code.
- Any import-map target reached only by a static import must be removed from the entry list and left for the bundler to merge naturally.

The concrete esbuild configuration shape to apply this with is in `SKILL.md` ("Standard tool: esbuild").

## Pre-release checklist

- [ ] The build was actually run, and the eager-path JS file count was measured; it is ≤ 5.
- [ ] No import-map target that is only ever statically imported is a separate bundler entry.
- [ ] Sourcemaps are off, or unreachable by end users, in the release build.
- [ ] A real measurement (Network tab / Lighthouse), before and after, is recorded against the table above.

## Out of scope for this document

UI/UX design, plugin domain architecture (`prepare`/`activate`), and the internal behavior of `pey-core` or any `pey.*` plugin are intentionally not covered here; this document only concerns the initial-load performance of the project's own code. The standard tool used to apply these principles (esbuild) is specified in `SKILL.md`, not here — these acceptance numbers are what any build must meet, independent of tooling.
