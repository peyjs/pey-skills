# Release, Tags, and Push (only when the person explicitly asked)

## Tag format

- Always `v<MAJOR>.<MINOR>.<PATCH>`, for example `v1.2.0`. The first release is `v1.0.0`.
- The version must equal `manifest.json` → `version` and the CHANGELOG heading (without the `v`).
- Tags are annotated: `git tag -a v1.2.0 -m "v1.2.0"`.
- Never move, delete, or reuse a tag that was pushed. A wrong release is fixed with a new PATCH version.

## Release steps (run each only if the person permitted it)

1. Working tree clean (`git status`) and tests passing.
2. Prepare the release with `changelog/SKILL.md`: decide the version, update `CHANGELOG.md`, and make the version identical in `manifest.json`, `package.json`, and the README badge.
3. Commit only those release files: `chore(release): 1.2.0`.
4. Tag that commit (needs tagging permission).
5. Push (needs push permission): `git push origin main`, then `git push origin v1.2.0`, naming the one tag.

If the person permitted only some steps, do those and stop; report what remains.

## Never, even with permission to push

- `git push --force` or `--force-with-lease`.
- `git push --tags` (push exactly one named tag).
- Tagging a commit whose `manifest.json` version differs from the tag.
- Deleting a remote tag or branch.

Each of these needs a separate, explicit request that names the action.

## Useful read-only commands

`git tag --sort=-v:refname | head`, `git describe --tags --abbrev=0`, `git log <last-tag>..HEAD --oneline`.
