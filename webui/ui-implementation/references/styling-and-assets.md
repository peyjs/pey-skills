# Styling and Assets

Read this before writing a component's styles, adding an icon/font/image, or making an interactive element accessible.

## Styling

- **Design tokens** (colors, spacing, typography, radii, shadows, etc.) are defined once as CSS Custom Properties on `:root`, in a single global tokens file. The only place visual constants are defined.
- Components consume tokens via `var(--token-name)` across the Shadow DOM boundary. Never hardcode a color, spacing value, or font that already has a token.
- Each component's styles live inside that component's own file, scoped by Shadow DOM. No separate global stylesheet per component, and no shared class-name convention (e.g. BEM) is needed, since Shadow DOM already prevents collisions.
- If a new visual value is needed repeatedly, promote it to a token in the global tokens file rather than repeating it inline.

## Assets (Icons, Fonts, Images)

- **Icons — default: shared SVG sprite.** One sprite sheet of `<symbol>` definitions, loaded once at the app shell level, referenced from any component via `<svg><use href="#icon-name"/></svg>`. Default specifically to avoid repeating the same icon markup across many components.
- **Icons — exception: inline SVG.** Only for a one-off icon not reused elsewhere, or an icon whose shape must change dynamically based on data (not just color/size, which tokens/CSS already handle).
- **Fonts:** declared once via `@font-face` in the global tokens/stylesheet file, referenced everywhere through a CSS Custom Property (e.g. `--font-family-base`), never redeclared per component.
- **Images:** stored under one fixed project-level assets folder, referenced by a stable relative path; never embed a large image as a base64 string in a template.

## Accessibility

First-class requirement, not an afterthought — applies inside Shadow DOM the same as anywhere else:

- Use semantic HTML elements first (`<button>`, `<nav>`, `<dialog>`, etc.) before reaching for a generic `<div>` plus ARIA.
- Any interactive element must be keyboard-operable (focusable in a sensible order, activatable via Enter/Space where applicable).
- ARIA attributes (`role`, `aria-label`, `aria-expanded`, etc.) are added when semantic HTML alone can't express the component's state (e.g. a custom dropdown).
- For components that trap or manage focus (modals, menus), focus must move to the component on open and return to the triggering element on close.
- Shadow DOM does not break these requirements on modern browsers; never skip accessibility on the assumption that Shadow DOM makes it someone else's problem.
