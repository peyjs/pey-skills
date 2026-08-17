# Forms and Internationalization

Read this before building any `<form>`-bearing component, before displaying an error returned from a service, or before adding any user-facing text, date, number, or currency.

## Forms

- **Client-side constraints:** basic syntactic constraints (required, min/max length, pattern) use native HTML5 validation attributes — first line of feedback only, never a substitute for service-side validation.
- **Submission:** always intercepted (`event.preventDefault()`); the component gathers the entered values and calls the relevant service method directly.
- **Service-rejected data:** displayed with one fixed convention — an error element associated with the field via `aria-describedby`, never a raw `alert()` or an unassociated message. The error's `code` only decides which translated message to show; its `message` field is for logs and never dumped to the screen.
- **Success:** the component relies on the relevant domain event (e.g. `transactions:saved`) for anything else that needs to react.

## Internationalization And Localization

Handled uniformly from the start, not retrofitted:

- All user-facing text goes through a single translation lookup (`t('key.path', params)`), never hardcoded; dates, numbers, and currency through one fixed formatting utility.
- In the `pey.webui` kit this lookup is **injectable, not a plugin service**: `createI18n()` at `pey.webui/i18n` builds one frozen `{ t, format, language }` instance that the Entry Point injects as refs. In a project that instead provides it through a dedicated i18n plugin service, the consumption contract from the component's side is identical.
- **Contract details** — the fallback chain (active language → `fallbackLanguage` → kit `en` catalog → key; invalid input returns an empty string, never throws) and the fact that `t()` returns raw, unescaped text: see `pey.webui/docs/public-api.md`, section **I18n And Formatting**, and `pey.webui/docs/decisions.md` (`Injectable I18n`, `t() Returns Raw Text`).
- Components never assume text direction; layout uses logical CSS properties (`margin-inline-start` rather than `margin-left`).