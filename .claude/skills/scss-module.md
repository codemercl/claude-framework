# Skill: SCSS Module Scaffold

Create a complete SCSS module for a component/element. This is the most boilerplate-heavy pattern in the project — 9 mandatory breakpoints.

## When to use

When creating any new component or element that needs styles.

## File Structure

```
{component}/styles/
├── index.module.scss
├── desktop.scss        (required, can be empty)
├── mobile.scss         (optional)
├── palette.scss        (required if colors used)
└── mq/
    ├── phonePortrait.scss
    ├── phoneLandscape.scss
    ├── phoneLandscapePlus.scss
    ├── tabletPortrait.scss
    ├── tabletLandscape.scss
    ├── desktop.scss
    ├── desktopWide.scss
    ├── desktopHD.scss
    ├── desktopMega.scss
    └── touchScreen.scss    (optional)
```

## index.module.scss (exact template)

Root class name MUST equal the component's folder name.

```scss
.{folderName} {
  @import '@assets/extensions';
  @import './palette';

  @function px($px) {
    @return pxMedia($px, phonePortrait);
  }

  @media #{getMediaFeature(phonePortrait)} {
    @import './mq/phonePortrait';
  }

  @function px($px) {
    @return pxMedia($px, phoneLandscape);
  }

  @media #{getMediaFeature(phoneLandscape)} {
    @import './mq/phoneLandscape';
  }

  @function px($px) {
    @return pxMedia($px, phoneLandscapePlus);
  }

  @media #{getMediaFeature(phoneLandscapePlus)} {
    @import './mq/phoneLandscapePlus';
  }

  @function px($px) {
    @return pxMedia($px, tabletPortrait);
  }

  @media #{getMediaFeature(tabletPortrait)} {
    @import './mq/tabletPortrait';
  }

  @function px($px) {
    @return pxMedia($px, tabletLandscape);
  }

  @media #{getMediaFeature(tabletLandscape)} {
    @import './mq/tabletLandscape';
  }

  @function px($px) {
    @return pxMedia($px, desktop);
  }

  @media #{getMediaFeature(desktop)} {
    @import './mq/desktop';
  }

  @function px($px) {
    @return pxMedia($px, desktopWide);
  }

  @media #{getMediaFeature(desktopWide)} {
    @import './mq/desktopWide';
  }

  @function px($px) {
    @return pxMedia($px, desktopHD);
  }

  @media #{getMediaFeature(desktopHD)} {
    @import './mq/desktopHD';
  }

  @function px($px) {
    @return pxMedia($px, desktopMega);
  }

  @media #{getMediaFeature(desktopMega)} {
    @import './mq/desktopMega';
  }

  @media #{getMediaFeature(touchScreen)} {
    @import './mq/touchScreen';
  }
}
```

## desktop.scss (template)

```scss
// Shared desktop+ styles. All px values use px() from mq files.
```

## palette.scss (template)

```scss
$textColor: var(--blackWhiteScale100);
$backgroundColor: var(--blackWhiteScale0);
$linkDefault: var(--primaryLight);
$linkHover: var(--primaryDark);
```

## mq/*.scss (all desktop breakpoints)

```scss
@import '../desktop';

// Breakpoint-specific styles using px()
// Example: padding: px(16);
```

## mq/phonePortrait.scss, phoneLandscape.scss, etc. (mobile)

If `mobile.scss` exists:
```scss
@import '../mobile';
```

If `mobile.scss` does NOT exist:
```scss
@import '../desktop';
```

## Rules

- No raw pixel values in mq files — always `px(24)`, never `24px`
- No `var(--...)` in component SCSS — use palette variables
- No styles directly in `@media` blocks in index.module.scss — only `@import`
- Max nesting depth: 3 levels
- No hardcoded hex colors — use palette
