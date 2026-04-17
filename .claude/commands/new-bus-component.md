---
name: new-bus-component
description: Scaffold a new bus/ component with styles, following project conventions. Usage: /new-bus-component <domain> <componentName> [--element]
---

Create a new component in `front/src/bus/$ARGUMENTS`.

Parse arguments: first arg is domain (e.g. `profile`), second is componentName (e.g. `profileCard`). If `--element` flag is present, create an Element (no hooks, no `Component` suffix). Otherwise create a Component.

## Steps

1. Verify the domain exists: `front/src/bus/{domain}/`. If not, ask the user if they want to create it.

2. Create the component directory and files:

**For Component** (`bus/{domain}/components/{componentName}/`):

`index.tsx`:
```tsx
// Core
import React, { ReactElement, FC } from 'react';

// Other
import styles from './styles/index.module.scss';

type PropsType = {
  children?: never;
}

export const {ComponentName}Component: FC<PropsType> = (): ReactElement => {
  return (
    <div className={styles.{componentName}}>
      {/* TODO */}
    </div>
  );
};
```

**For Element** (`bus/{domain}/elements/{componentName}/`):

`index.tsx`:
```tsx
// Core
import React, { ReactElement, FC } from 'react';

// Other
import styles from './styles/index.module.scss';

type PropsType = {
  children?: never;
}

export const {ComponentName}: FC<PropsType> = (): ReactElement => {
  return (
    <div className={styles.{componentName}}>
      {/* TODO */}
    </div>
  );
};
```

3. Create styles structure:

`styles/index.module.scss`:
```scss
.{componentName} {
  @import '@assets/extensions';
  @import './palette';

  @function px($px) {
    @return pxMedia($px, desktop);
  }

  @media #{getMediaFeature(desktop)} {
    @import './mq/desktop';
  }
}
```

`styles/desktop.scss` — empty file
`styles/palette.scss` — empty file
`styles/mq/desktop.scss`:
```scss
@import '../desktop';
```

## Rules
- Root CSS class = componentName (folder name)
- Component suffix for smart components, no suffix for elements
- PropsType always named `PropsType`
- Return type `ReactElement`
- Named export (not default)
- Import order with comment headers
- Latin characters only
