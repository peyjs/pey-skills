# Forms and Internationalization

Read this before building any `<form>`-bearing component, before displaying an error returned from a service, or before adding any user-facing text, date, number, or currency.

## Forms

Every form-bearing component follows the same fixed pattern, so forms built by different sessions behave identically:

- **Client-side constraints:** basic, purely syntactic constraints (required, min/max length, pattern) use native HTML5 validation attributes on the input elements. First line of feedback only — never a substitute for validation on the service/plugin side.
- **Submission:** always intercepted (`event.preventDefault()`), never a real network submission. The component gathers entered values (e.g. via `FormData` or direct field reads) and calls the relevant service method directly with that data — the same primary-path rule as any other user action (see `architecture-context/service-contract.md`).
- **Service-rejected data:** if the service call returns a standard Bonyan error (e.g. a validation failure), display it using one fixed convention: an error message element associated with the relevant field via `aria-describedby`, never a raw `alert()` or an unassociated floating message.
  - **`code` vs `message`:** the error's `code` (see `architecture-context/error-structure.md`) is never shown to the user directly — it only decides *which* user-facing message to show (mapped through the i18n lookup below, so displayed text is translatable and consistent) or drives logic (e.g. highlighting a field). The error's `message` field is for logs/developers and is not guaranteed to be in the user's language — never dump it onto the screen as-is. Always: `code` → translated user-facing string → display.
- **Success:** on success, the component does not assume anything about what changed elsewhere; it relies on the relevant domain event (e.g. `transactions:saved`) for any other part of the UI that needs to react.

## Internationalization and Localization

If the product supports more than one language, this is handled uniformly from the start, not retrofitted:

- All user-facing text is retrieved through a single fixed translation lookup (e.g. `t('key.path', params)`), never hardcoded as a literal string inside a template. Whether this lookup is backed by a dedicated i18n infrastructure plugin's service (handed down by the Entry Point like any other service) is a decision for that plugin's manifest — every component consumes it the same single way regardless.
- Dates, times, numbers, and currency are always formatted through one fixed formatting utility (backed by a service or shared helper handed down by the Entry Point), never ad hoc string concatenation or a hardcoded format pattern inside a component.
- A component never bakes in an assumption about text direction (LTR/RTL); layout uses logical CSS properties (`margin-inline-start` rather than `margin-left`, etc.) so the same component works in either direction.
