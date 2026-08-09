---
name: bonyan-ui-coding-standard
description: Use whenever writing or reviewing JavaScript code for the UI layer of a Bonyan-architecture project — components, the UI Entry Point, the base class, the shared state store. Triggers whenever UI code is being written, not only when the user explicitly asks for "standards" or "style".
---

# UI Coding Standards

General coding standards for all UI-layer code. Component-specific architecture rules (Shadow DOM, `connect()`, render strategy, ownership, etc.) belong to the `bonyan-ui` skill — read that first for anything architectural. This document governs code style only.

If anything here appears to conflict with a `bonyan-ui` reference file (e.g. `component-basics.md`, `composition.md`), the `bonyan-ui` reference file wins — stop and report the conflict rather than resolving it by picking one.

## Module System
- Always use ES Modules (`import` / `export`).
- Never use `require` / `module.exports`.
- No bundler-only syntax (no `?raw`, `?inline`, or other loader-specific import suffixes) — see `bonyan-ui`'s Technology Baseline: what's written in the file is what the browser runs, unmodified.

## General Principles
- Every function does exactly one thing.
- Aim to keep functions under 30 lines; treat this as a guideline, not a hard limit — split when a function does too much, unless there is a documented reason to keep it as is.
  - **Named exception:** a component's `render()` method (see `bonyan-ui/references/component-basics.md`) may exceed this where the length comes from the markup itself, not from embedded logic. If `render()` is long because it contains conditionals, loops, or computed values beyond simple interpolation, extract those into small private helper functions and call them from the template string — the template's *structure* can be long, its *logic* should not be.
- Avoid magic values; define constants near the top of the file (e.g. event names used in `subscriptions()`, `data-action` values used in `handleEvent()`).
- Avoid duplicated logic; extract repeated logic into named functions.
- Prefer explicit, readable code over clever code.
- Avoid hidden side effects — a delegated `handleEvent()` branch should do one clear thing per `data-action` value, not several unrelated things gated behind one condition.

## Naming
- Use English names.
- Variables and functions: `camelCase`.
- Classes: `PascalCase` (component class names, e.g. `ExampleWidget`).
- Constants: `UPPER_SNAKE_CASE` (e.g. `TAG`, `MAX_RETRY_COUNT`).
- Files: `kebab-case`, matching the custom element tag's second segment (see `bonyan-ui/references/file-layout.md`).
- **`TAG` is a conventional exception, noted separately so it doesn't create doubt about other constants:** the *variable name* `TAG` still follows `UPPER_SNAKE_CASE` like any other constant — it is not an exception to that naming rule. What's different is only its *string value*, which is a DOM custom-element tag name and must itself be lowercase kebab-case per the Custom Elements spec (e.g. `const TAG = 'app-example-widget';`). Don't generalize from this to other constants — every other constant's value takes whatever type/format it needs, with no special-casing.
- Custom element tag names (the string values held by `TAG`): follow `bonyan-ui/references/file-layout.md`'s `app-<name>` / `app-page-<name>` convention exactly — this is a naming rule, not a style preference, and takes precedence over generic class-naming intuition.
- Names must be descriptive; avoid single-letter names and vague abbreviations.
- Boolean state fields and functions start with `is`, `has`, or `can` (e.g. `#isOpen`, `isConnected`).
- Prefer `error` over `err`; prefer `config` over `cfg`.

## File Organization
Every UI source file should follow this order:
1. Short file responsibility comment
2. Imports
3. Constants (including the `TAG` constant for a component file)
4. The class definition, in this internal order: private fields → `connect()` → lifecycle hooks (`connectedCallback`/`disconnectedCallback`) → `subscriptions()` → public API methods → private helper methods → `render()` → `handleEvent()`
   - **Private helper methods** (small functions extracted from `render()` or `handleEvent()` per the function-length guidance above) go in one group, placed after the public API methods and before `render()` — never interleaved between public methods, and never placed above `connect()`. A helper used only by `render()` and a helper used only by `handleEvent()` both live in this same group; there is no need to split it further by which method calls them.
5. `customElements.define()` + exports

Rules:
- One component per file — see `bonyan-ui/references/file-layout.md` for the folder/file mapping; this document does not redefine that layout, only points to it.
- Aim to keep files under 200 lines; treat this as a guideline, not a hard limit — split a component's markup into smaller child components (per `bonyan-ui/references/composition.md`) rather than growing one file indefinitely.
- Do not create unnecessary files. A component's styles may live in the same file as a template literal or in a co-located `.css` file per `file-layout.md` — pick one approach per project and stay consistent, don't mix per-component.

## Code Documentation
- Every file starts with a short responsibility comment.
- Every public method — including a component's public API methods (see `bonyan-ui/references/composition.md`) and the fixed `connect()` method — is documented: purpose, inputs, and return value.
- Public method documentation must use JSDoc format (`@param`, `@returns`, `@throws`).
- Comment business rules and non-obvious decisions. A UI file should have very few of these, since business rules never belong in a component — if a comment is explaining a business rule rather than a display decision, that's a signal the logic may be in the wrong place (see `bonyan-ui/references/decision-guide.md`).
- Do not comment obvious code.
- Comments are written in English.

## Resource Management
- Remove every Event Bus subscription and DOM event listener during cleanup — in this codebase, that means the base class's `disconnectedCallback` handles it centrally (see `bonyan-ui/references/component-basics.md`); a component overriding `disconnectedCallback` must always call `super.disconnectedCallback()`.
- Store every timer and clear it during cleanup (`disconnectedCallback`).
- Abort in-flight async work on cleanup (`AbortController`, or at minimum the `isConnected` guard) — see `bonyan-ui/references/templates.md`'s page-component example.
- Release references to large fetched objects when no longer needed — concretely, this means explicitly setting the field to `null` inside `disconnectedCallback` (e.g. `this.#account = null;`), not merely letting the field go out of scope or relying on garbage collection to happen "eventually." Do this once all of this component's own cleanup work is done, alongside aborting in-flight work — the base class's `disconnectedCallback` contract does not currently require access to a component's fields, so nulling them can happen before or after the `super.disconnectedCallback()` call; treat "before/after" as unspecified rather than assuming an order, and only fix an explicit order if `bonyan-ui/references/component-basics.md` is updated to document one. This keeps the pattern uniform across components rather than leaving "release" to mean something different per implementation. Do not let a component's local field become a de facto cache while it's still connected (see `bonyan-ui/references/decision-guide.md`) — this rule is about cleanup on disconnect, not about avoiding the field's existence while mounted.

## Error Handling
This section covers code-level error hygiene inside a component's own logic (e.g. inside `handleEvent()`, inside a helper called from `render()`). It does not replace the base class's error boundary described in `bonyan-ui/references/component-basics.md` — that boundary is what catches an error that escapes here; write clean, non-silent error handling in your own code so the boundary is a safety net, not the primary mechanism.
- Never ignore errors silently.
- Handle, log, or intentionally propagate errors.
- Async functions that handle their own errors must use `try/catch`.
- If an async function intentionally propagates errors (letting the base class's boundary catch them), document it with a short comment saying so.
- User-facing error messages must be safe, short, and understandable — and must go through the `code` → translated string → display path in `bonyan-ui/references/forms-and-i18n.md`, never a raw thrown message.
- Do not expose stack traces or internal implementation details to users.
- Do not expose raw `Error` objects outside the local error-handling path.
- Do not include sensitive data in error messages, in `ui:component-error`'s payload, or in anything logged.
- Use stable error codes when machine-readable errors are required — see `bonyan-ui/references/architecture-context/error-structure.md`.

## Async Patterns
- Use `async` and `await`.
- Avoid raw `.then()` and `.catch()` chains unless they are clearly simpler.
- Use sequential `await` when order matters.
- Use `Promise.all` only for independent operations.
- Do not wrap already-async code in `new Promise()`.
- Async functions must return a value or throw.
- Any async work started from a lifecycle hook or an event handler (e.g. `connectedCallback`, or a `handleEvent()` branch that itself awaits a service call) must guard against the component having been disconnected before it resolves — see the `isConnected` pattern in `bonyan-ui/references/templates.md`. This is a hard rule for any async work anywhere in a component, not only inside `connectedCallback` and not a stylistic suggestion: check `this.isConnected` immediately after every `await` that is followed by a state update or `requestRender()` call, regardless of which method the `await` lives in.

## Security and Privacy
- Do not log secrets, tokens, credentials, personal data, or raw sensitive payloads — including inside `ui:component-error`'s data, per `bonyan-ui/references/component-basics.md`.
- Validate any data read from the DOM (form input, `dataset` values) before passing it to a service call — but only **syntactic/type-level** validation (e.g. confirming a field isn't empty when a value is required for the call to even make sense, coercing a string input to a number before sending it, checking an HTML5 constraint already reflected in the form). This is not business-rule validation — deciding whether a value is *acceptable by product rules* (a minimum amount, an allowed date range, uniqueness) is never a component's job; that decision belongs entirely to the service, per `bonyan-ui/references/decision-guide.md` and `bonyan-ui/references/forms-and-i18n.md`. The line: "is this even the right shape to send" is a component concern; "is this an allowed value" is not.
- Client-side validation, syntactic or otherwise, is a UX convenience only — never a trust or correctness boundary. The service call is always made and always re-validates.
- Do not trust a value just because it came from another component's property/attribute.
- Fail safely when input is invalid — show the fixed error-display convention, never let a bad value silently proceed to a service call.
