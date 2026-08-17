# UI Manifest

Self-contained summary of the UI manifest's structure, based on `ui.md`. Read this whenever adding a new service dependency, a new `ui:*` event, or when setting up the UI Entry Point for the first time.

## The UI Is Not a Plugin, But Has a Manifest

The UI layer is not a plugin — it has no `type`, and no `provides` section, since it never provides services to anyone. But like a plugin, it has a manifest (defining its access boundary) and an entry point, both read by the Core at startup. The Core grants the UI access only to what's declared in this manifest — nothing more.

## Structure

```json
{
  "name": "myapp.ui",
  "version": "1.0.0",
  "architecture": "0.11",
  "description": "Main application UI",
  "dependencies": {
    "required": [
      "myapp.transactions.service",
      "myapp.accounts.service"
    ],
    "optional": [
      "myapp.reports.service",
      "myapp.logger.service"
    ]
  },
  "events": [
    {
      "name": "ui:filter-changed",
      "description": "The user has changed the display filter",
      "data": [
        { "name": "filterId", "type": "string" }
      ]
    },
    {
      "name": "ui:component-error",
      "description": "A component's render or event handler threw and was caught by the shared base class",
      "data": [
        { "name": "componentTag", "type": "string" },
        { "name": "code", "type": "string" }
      ]
    }
  ]
}
```

- **`dependencies.required` / `dependencies.optional`:** every service any component in the UI might need must be listed here — this is the *only* set of services the Entry Point can ever hand down to a component. A component asking for something not in this list has nothing to receive, architecturally (see `core-boundaries.md`).
- **`events`:** every event the UI itself publishes, including every `ui:*` event any component might fire — declared once here, not per component. This must include `ui:component-error` (see `component-basics.md`) if the shared base class's error boundary is in use, which it always is per this skill's core rules.
- **No `provides` section** — the UI never provides services.
- **No `type` field** — the UI is not a plugin.

## Practical Consequence for Component Work

Before a component can be given access to a new service, that service must first appear in the UI manifest's `dependencies`. Before a component can publish a new `ui:*` event, that event must first appear in the UI manifest's `events`. Neither of these is something an individual component file changes on its own — both are changes to the one shared UI manifest, made deliberately, the same way a plugin's manifest is the single source of truth for what that plugin may do.

## UI Configuration

Like a plugin, the UI receives a final configuration object from the Core at startup — the Entry Point consumes it as **`context.config`** (already merged, with any `${ENV_VAR}` placeholders resolved), without depending on how the Core built it. Only the Entry Point receives this object; no component asks the Core for it directly, for the same reason a component never asks the Core for a service (see `core-boundaries.md`).

Distribution to components follows the same pattern as service references:

- The Entry Point reads whatever it needs from its configuration object at startup (e.g. feature flags, default page size, theme defaults).
- If a component needs a specific configuration value, the Entry Point passes that value down explicitly — as a plain property/argument, the same way it hands down a service reference — rather than handing the component the whole configuration object or letting the component read from some shared global.
- A component never imports or reads `bootstrap.json`, `process.env`, or any configuration source directly — configuration reaches a component only through what the Entry Point explicitly gave it, exactly as with services.

### The `pey.webui` Kit's Shared Config

The kit recognizes exactly these shared keys (defaults from `DEFAULT_UI_CONFIG` in `pey.webui/contracts`):

| Key | Type | Default | Meaning |
|---|---|---|---|
| `theme` | `string` | `system` | Initial presentation theme. |
| `language` | `string` | `en` | Initial locale identifier. |
| `direction` | `ltr \| rtl \| auto` | `auto` | Initial writing direction. |
| `fallbackPath` | `string \| null` | `null` | Optional safe navigation target; only non-empty string validation belongs to the UI. |
| `assetBaseUrl` | `string` | `./assets` | Asset location resolved against the consuming Entry Point's `import.meta.url`. |

Rules:

- Only the application Entry Point reads `context.config`; components receive selected values through connection refs. There is deliberately **no `styleNonce` key**: the CSP nonce for external stylesheets reaches a component through `refs` and `stylesheetNonce()` instead (see `docs/configuration.md` in the kit).
- **Assets resolve exclusively from `assetBaseUrl`** — never from the Router's `basePath`, `document.baseURI`, or a hardcoded path. The Entry Point normalizes `assetBaseUrl` to an absolute directory URL with exactly one trailing slash; child assets use `new URL(childPath, assetBaseUrl)`. Router `basePath` is navigation-only.

References: `pey.webui/contracts` (`DEFAULT_UI_CONFIG`), `pey.webui/docs/configuration.md`, and `pey.webui/docs/decisions.md` (Configuration Contract / Module Loading).
