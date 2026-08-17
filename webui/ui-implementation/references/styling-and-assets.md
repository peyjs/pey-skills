# Styling and Assets

Read this before writing a component's styles, adding an icon/font/image, or making an interactive element accessible.

## Styling

- **Design tokens** are defined once as CSS Custom Properties on `:root` and shipped by the kit at `pey.webui/tokens.css`; components consume them via `var(--token-name)` across the Shadow DOM boundary — never hardcode a value that already has a token.
- A component's styles belong to the component, scoped by Shadow DOM; no shared class-name convention (e.g. BEM) is needed. Small components inline `<style>` in `render()`; a nontrivial component keeps a separate CSS file loaded via `PeyElement`'s `stylesheetHref()`.
- If a new visual value is needed repeatedly, promote it to a token rather than repeating it inline.

## External Stylesheets And Icons

Both are implemented by the kit, with contracts fixed in `pey.webui/docs/decisions.md`:

- **CSS Attach Method** — a component with a separate stylesheet opts in via `PeyElement`'s `stylesheetHref()` / `stylesheetNonce()` hooks and the runtime at `pey.webui/base/attach-style-sheet`; the final attach method is preload-and-cache-`<style>`. Inlining carries a documented CSP consequence: `style-src` must permit the inlined `<style>`.
- **Icon Sprite Contract** — default is the shared external SVG sprite, resolved exclusively from `assetBaseUrl` via `pey.webui/base/icon-sprite`; the fallback is that the Entry Point loads the sprite text once and the needed `<symbol>` is cloned into the component's Shadow DOM — never the whole sprite injected into the document. The external-reference contract stays provisional on WebKit until the engine matrix passes.
- **Fonts and images:** declared/stored once at the project level; fonts via `@font-face` in the tokens file, never redeclared per component.

## Accessibility

First-class requirement, applies inside Shadow DOM the same as anywhere else:

- Use semantic HTML elements first; add ARIA only when semantics alone can't express the state.
- Any interactive element must be keyboard-operable.
- Components that trap/manage focus move focus in on open and back to the trigger on close.
- Shadow DOM does not exempt a component from accessibility.