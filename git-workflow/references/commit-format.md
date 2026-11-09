# Commit Message Format (Conventional Commits 1.0.0)

```
<type>(<scope>)!: <subject>
```

Default is the single line above. Use a body and footer only when the person asks you to run the commit (see `committing.md`) and the change is non-trivial or breaking.

## Types

| Type | Use for | Version effect |
|---|---|---|
| `feat` | new capability | MINOR |
| `fix` | bug fix | PATCH |
| `docs` | README, CHANGELOG, `docs/` only | none |
| `refactor` | code change with no behavior change | none |
| `test` | tests only | none |
| `perf` | performance improvement | PATCH |
| `build` | `package.json`, tooling, dependencies | none |
| `ci` | CI configuration | none |
| `style` | formatting only | none |
| `chore` | maintenance that fits nowhere else; release commits | none |
| `revert` | reverting an earlier commit | depends |

When a task mixes types, pick the one describing the most important change (a `feat` with its tests and docs is `feat`).

## Scope

Optional, lowercase, one word: the area touched, such as `manifest`, `service`, `config`, `readme`, `changelog`, `docs`. Omit it for broad changes.

## Subject

Imperative ("add", not "added"/"adds"), lowercase first letter, no trailing period, at most 72 characters. Say what the commit does, not what you did.

## Breaking changes

Add `!` after type/scope. A breaking change is one a dependent plugin must adapt to: removing or renaming a service, method, event, or config key; changing `params`/`returns` or event `data` incompatibly. When writing a full commit, add a footer: `BREAKING CHANGE: <what changes and what dependents must do>`.

## Examples

Good:

```
feat(service): add list method for stored preference keys
fix(service): return null instead of throwing for whitespace-only keys
docs(readme): sync dependencies with manifest
feat(manifest)!: rename remove to delete in service contract
```

Bad: `update stuff`, `Fix bug`, `feat: Added new feature.`, `WIP`, a subject over 72 characters, several messages for one task, Persian text.
