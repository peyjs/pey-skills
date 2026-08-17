---
name: bonyan-coding-standard
description: Use whenever writing or reviewing JavaScript code for a Bonyan-architecture project — naming, file structure, error handling, async patterns, module system. Triggers whenever code is being written, not only inside a plugin-build task.
---

# Coding Standards
General coding standards for all tasks. Architecture-specific rules belong to specialized agents or task instructions.

This document governs code style only. If anything here appears to conflict with a Bonyan architecture reference file (e.g. `plugin-implementation/references/plugin-fundamentals.md`, `plugin-implementation/references/adapter-contract.md`), the architecture file wins — stop and report the conflict rather than resolving it by picking one.

## Module System
- Always use ES Modules (`import` / `export`).
- Never use `require` / `module.exports`.

## General Principles
- Every function does exactly one thing.
- Aim to keep functions under 30 lines; treat this as a guideline, not a hard limit — split when a function does too much, unless there is a documented reason to keep it as is.
- Avoid magic values; define constants near the top of the file.
- Avoid duplicated logic; extract repeated logic into named functions.
- Prefer explicit, readable code over clever code.
- Avoid hidden side effects.

## Naming
- Use English names.
- Variables and functions: `camelCase`.
- Classes: `PascalCase`.
- Constants: `UPPER_SNAKE_CASE`.
- Files: `kebab-case`.
- Names must be descriptive; avoid single-letter names and vague abbreviations.
- Boolean functions start with `is`, `has`, or `can`.
- Prefer `error` over `err`; prefer `config` over `cfg`.

## File Organization
Every source file should follow this order:
1. Short file responsibility comment
2. Imports
3. Constants
4. Private functions
5. Public functions or exports

Rules:
- One responsibility per file.
- Aim to keep files under 200 lines; treat this as a guideline, not a hard limit — split when a file covers too much, unless there is a documented reason to keep it as is.
- Do not create unnecessary files.

## Code Documentation
- Every file starts with a short responsibility comment.
- Every public function is documented: purpose, inputs, and return value.
- Public function documentation must use JSDoc format (`@param`, `@returns`, `@throws`).
- Comment business rules and non-obvious decisions.
- Do not comment obvious code.
- Comments are written in English.

## Resource Management
- Remove every subscription and event listener during cleanup.
- Store every timer and clear it during cleanup.
- Release references to large objects when no longer needed.
- Cleanup must happen in the appropriate lifecycle method, such as `stop`, `shutdown`, or `disconnectedCallback`.

## Error Handling
This section covers code-level error hygiene. Any error crossing a plugin's or adapter's boundary must additionally use Bonyan's standard error structure (`code`/`message`/`source`/`type`/`timestamp`/`detail`) — see `plugin-implementation/references/plugin-fundamentals.md`. These two are complementary, not separate concerns: write clean, non-silent error handling in code, and shape what leaves the boundary into that structure.
- Never ignore errors silently.
- Handle, log, or intentionally propagate errors.
- Async functions that handle their own errors must use `try/catch`.
- If an async function intentionally propagates errors, document it.
- User-facing error messages must be safe, short, and understandable.
- Do not expose stack traces or internal implementation details to users.
- Do not expose raw `Error` objects outside the local error-handling path.
- Do not include sensitive data in error messages.
- Use stable error codes when machine-readable errors are required.

## Async Patterns
- Use `async` and `await`.
- Avoid raw `.then()` and `.catch()` chains unless they are clearly simpler.
- Use sequential `await` when order matters.
- Use `Promise.all` only for independent operations.
- Do not wrap already-async code in `new Promise()`.
- Async functions must return a value or throw.

## Security and Privacy
- Do not log secrets, tokens, credentials, personal data, or raw sensitive payloads.
- Validate external input before use.
- Do not trust input just because it comes from another internal file.
- Fail safely when input is invalid.