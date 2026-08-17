# Error Structure

Self-contained summary of Bonyan's standard error object, based on `error.md`. Read this whenever handling or displaying an error returned from a service call.

## The Fields

Every error in Bonyan has:

- **`code`** — a unique, stable identifier for this type of error. Logic decisions are always made on this field, never on `message` text.
- **`message`** — a human-readable description, for logging/debugging. Not guaranteed to be in the end user's language and not meant for direct display to them.
- **`source`** — the plugin/part where the error occurred (`vendor.name` format).
- **`type`** — one of `startup`, `operational`, or `critical`. A service call rejected from the UI's perspective is almost always `operational` — the plugin handled it itself and returned a structured error rather than letting something raw leak out.
- **`timestamp`** — when it occurred.
- **`detail`** *(optional)* — extra debugging info. Never sensitive data (passwords, tokens, personal info) — not even here.

## What This Means for the UI

- **Never display `message` directly to the user.** It's for developers/logs. Always map `code` to a translated, user-facing string (see `../forms-and-i18n.md`'s `code` → translated string → display rule).
- **Never display `detail`.** It may contain internal debugging context not meant for an end user, and must never contain sensitive data in the first place — but displaying it is not the error structure's intended use regardless.
- **Branch logic on `code`, never on `message`.** E.g. deciding which form field to highlight, or whether to show a retry button, reads `code`.
- **A component receiving an error from a service call is receiving an already-standardized object.** The plugin itself is responsible for converting any raw/external error into this shape before it ever reaches the UI (per `error.md`'s rule that raw errors never leak past a plugin's boundary) — the UI doesn't need to normalize anything itself, only read `code` and act on it.
