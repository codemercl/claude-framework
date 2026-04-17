# Skill: Component/Element Scaffold

Create a React component or element following project conventions.

## When to use

When creating any new UI unit in bus/ or core/.

## Component vs Element

| | Component | Element |
|--|-----------|---------|
| Suffix | `{Name}Component` | `{Name}` (no suffix) |
| Location | `components/` | `elements/` |
| Hooks | YES — queries, mutations, Redux | NO — props only |
| Logic | YES — state, effects, handlers | NO — pure render |
| Example | `ProfileCardComponent` | `Avatar`, `Badge` |

## Component Template

```typescript
// Core
import React, { ReactElement, FC } from 'react';

// Elements
import { Typography } from 'ui-kit';

// Hooks
import { use{Hook} } from '../../hooks/use{Hook}';

// Other
import styles from './styles/index.module.scss';

type PropsType = {
  children?: never;
};

export const {Name}Component: FC<PropsType> = (): ReactElement => {
  const { data, isLoading } = use{Hook}();

  const contentJSX = data && (
    <div className={styles.content}>
      <Typography text={data.title} variant='h3' />
    </div>
  );

  return (
    <div className={styles.{folderName}} data-attr='{folderName}'>
      {contentJSX}
    </div>
  );
};
```

## Element Template

```typescript
// Core
import React, { ReactElement, FC } from 'react';

// Elements
import { Typography } from 'ui-kit';

// Other
import styles from './styles/index.module.scss';

type PropsType = {
  title: string;
  isActive: boolean;
  children?: never;
};

export const {Name}: FC<PropsType> = ({ title, isActive }: PropsType): ReactElement => {
  const containerCX = cx({
    [styles.{folderName}]: true,
    [styles.isActive]: isActive,
  });

  return (
    <div className={containerCX}>
      <Typography text={title} variant='body1' />
    </div>
  );
};
```

## Props rules

- ≤3 props → destructure in signature: `({ title, isActive }: PropsType)`
- >3 props → destructure in body: `(props: PropsType) => { const { a, b, c, d } = props; }`
- Always include `children?: never`

## Conditional JSX

```typescript
// Extract to variable with JSX suffix
const titleJSX = isActive && (
  <Typography text={title} variant='h3' />
);

// Never inline complex conditionals in return
return <>{titleJSX}</>;
```

## With cx() classnames

```typescript
import cx from 'classnames';

const containerCX = cx({
  [styles.{folderName}]: true,
  [styles.isActive]: isActive,
  [styles.isDisabled]: isDisabled,
});
```

## Styles

Use the `scss-module` skill to create the styles/ directory.
