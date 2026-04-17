---
globs: "**/*.scss"
---

# SCSS Conventions

## File Structure (required for every component with styles)

```
{component}/styles/
├── index.module.scss    (required — root class, @imports, breakpoint declarations)
├── desktop.scss         (required — shared desktop+ styles, can be empty)
├── mobile.scss          (optional — shared mobile styles)
├── palette.scss         (required if colors used — $var: var(--token))
└── mq/                  (required — one file per breakpoint)
    ├── phonePortrait.scss
    ├── phoneLandscape.scss
    ├── phoneLandscapePlus.scss
    ├── tabletPortrait.scss
    ├── tabletLandscape.scss
    ├── desktop.scss
    ├── desktopWide.scss
    ├── desktopHD.scss
    ├── desktopMega.scss
    └── touchScreen.scss  (optional)
```

## index.module.scss Rules

- Root class name = folder name: `.profileCard { ... }` for `profileCard/`
- Imports order: `@import '@assets/extensions'` → `@import './palette'` → shared styles → breakpoints
- Only non-responsive styles allowed directly: `position`, `z-index`, `display`, `flex` properties
- No responsive pixel values — those go in mq/ files via `px()`
- No styles inside `@media` blocks — only `@import './mq/breakpointName'`
- `px()` function is declared per-breakpoint before each `@media` block

## px() Function

- Declared in `index.module.scss` per breakpoint: `@function px($px) { @return pxMedia($px, desktop); }`
- Can ONLY be used in `mq/*.scss` files (after declaration)
- All pixel values in mq-files: `px(24)`, never raw `24px`
- Cannot be used in `index.module.scss`, `desktop.scss`, or `mobile.scss` directly

## mq/ File Inheritance

- Desktop mq-files: `@import '../desktop';`
- Mobile mq-files: `@import '../mobile';` if `mobile.scss` exists, otherwise `@import '../desktop';`
- Then breakpoint-specific overrides using `px()`

## Colors

- ALL colors through `palette.scss` variables: `$textColor: var(--blackWhiteScale100)`
- Never use `var(--...)` directly in component SCSS
- Never hardcode hex (#fff) or rgb() colors

## General Rules

- Max nesting depth: 3 levels
- BEM-like nesting: `&Title`, `&Container`, `&.hasData`
- `:global` for third-party library classes and SVG elements
- Common animation durations: 0.2s, 0.3s, 0.7s, 0.8s, 1s
- Available mixins: `@include colorIcon()`, `@include hideTextWithEllipsis()`
- Single quotes for strings
- SMACSS property sort order (enforced by stylelint)
