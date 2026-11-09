# Committing (only when the person explicitly asked you to commit)

## Before committing

1. `git status` and `git diff` (staged and unstaged): know exactly what changed.
2. Run `npm test`. Do not commit with failing tests unless the person says so; tell them if they do.
3. Stage specific files by name (`git add <path>`), never `git add -A` without reviewing `git status`. Never stage secrets, `node_modules`, build output, or files unrelated to the task.
4. If the work contains clearly unrelated changes, ask whether to make one commit or split them; do not decide alone.

## Committing

- Use the message format in `commit-format.md`. For a non-trivial change add a body after a blank line (wrapped at 72 characters, explaining why, not restating the diff); for a breaking change add the `BREAKING CHANGE:` footer.
- Tests go in the same commit as the code they cover.
- After committing, run `git log --oneline -1` and tell the person the result in one sentence.

## Never, even with permission to commit

- `git reset --hard`, `git clean -fd`, `--no-verify`.
- Amending or rebasing commits that were already pushed.
- Editing `git config` (author identity included).
- Creating, switching, or merging branches.

Each of these needs a separate, explicit request that names the action.
