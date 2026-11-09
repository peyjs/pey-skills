---
name: bonyan-git-workflow
description: Use when a task on a Bonyan-architecture plugin is finished and a commit message is needed, or when the person asks to commit, tag, release or push. Triggers on requests like "give me a commit message", "commit this", "tag this version", "push".
---

# Git Workflow (Bonyan)

Everything lives on `main`; do not create or switch branches. Commit messages and tags are in **English**.

## Default behavior: a one-line commit message, nothing executed

When you finish a task, or the person asks for a commit message, reply with **exactly one line** in a code block that summarizes the whole task, whether the work was one small step or many. The person is not technical and will paste it as is, so:

- Format: `type(scope): subject`. Scope is optional; omit it when the work touched several areas.
- Subject: imperative mood, lowercase first letter, no trailing period, at most 72 characters.
- Always one line, never several messages and never a body or footer, even when the work mixed unrelated changes. Describe the most important change; do not chain changes with "and" or commas.
- Types: `feat` (new capability), `fix` (bug fix), `docs`, `refactor`, `test`, `build`, `chore`. For the full list and breaking changes, read `references/commit-format.md`.
- If the change breaks existing users of the plugin (renamed/removed service, method, event, or config key), write `!` after the type (`feat(manifest)!: ...`) and add one plain sentence after the code block saying what breaks.

Example reply ending: 

```
feat(service): add list method for stored preference keys
```

## Permissions

| Action | Default | Needs |
|---|---|---|
| Suggesting a commit message | always | nothing |
| `git add` / `git commit` | not done | the person explicitly says to commit |
| creating a tag | not done | the person explicitly says to tag or release |
| `git push` | not done | the person explicitly says to push |

Permission is per action and per occasion: "commit" does not mean "tag", "tag" does not mean "push", and an earlier permission does not carry over. If anything is unclear or a git command fails unexpectedly, stop and report.

## Read a reference only when needed

Read the matching file before acting, and only then:

- Writing a message beyond the simple case (breaking change, unsure of type): `references/commit-format.md`
- The person asked you to commit: `references/committing.md`
- The person asked for a release, a tag, or a push: `references/release-and-tags.md`

## Never

- Run any git command that changes the repository without the matching explicit request.
- Give more than one line, or a message in Persian.
- Hide a breaking change.
