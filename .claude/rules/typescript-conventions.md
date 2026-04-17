---
globs: "**/*.ts,**/*.tsx"
---

# TypeScript Conventions

## Types

- NEVER use `interface` — always `type` alias
- NEVER use `as Type` casting — use type guards or fix types upstream
- NEVER use runtime `enum` — use string union types
- NEVER use `unknown` without a type guard
- ALL custom types end with `Type` suffix: `UserType`, `ButtonType`, `PropsType`
- Component props type is always named `PropsType` (not `ButtonPropsType`)
- Hook return type is always named `ReturnedType` (unless returning a built-in type like void, boolean, string)
- Component return type: `ReactElement`
- Named exports everywhere except Next.js pages (default export)
- No bare `any` without inline comment + ticket reference
- Use type-only imports when importing only types: `import type { UserType } from './types'`
- Explicit return types on all public functions and hooks
- Narrow types over broad: prefer specific union over `string`, `Record<string, unknown>` only when shape is truly unknown
- `tsconfig.json` has `strict: true` enabled

## Import Organization (8 Groups)

Strict order with `// GroupName` comment headers and blank lines between groups. Alphabetical within each group.

```
// Core          — React, Next.js, third-party libraries
// Platform      — @platform/*, @assets/*
// Elements      — ui-kit, core/elements/*, pure sub-elements (props in, JSX out, no hooks/state)
// Hooks         — custom hooks
// Templates     — core/templates/*
// Views         — views
// Components    — components
// Other         — styles, types, constants, helpers
```

No file extensions in imports. Project-internal imports use relative paths. Allowed aliases only: `@platform`, `@assets`, `@routes`. No mixed default + named imports from same module.

## React Component Patterns

- **Components** (owns behavior: hooks, state, effects) — suffix with `Component`: `FooterComponent`
- **Elements** (pure render: props in, JSX out, nothing else) — no suffix: `Avatar`, `Badge`
- **Standalone elements**: `src/core/elements/` — reusable across project
- **Local sub-elements**: colocated in component directory — used only by parent component
- Props destructuring: ≤3 props in function signature, >3 in body
  - Exceptions: forwardRef, spread props `{...props}`, when props needed as object
- Extract conditional JSX to variables with `JSX` suffix: `const titleJSX = condition ? <Title /> : null`
- Use `React.memo` for UI-Kit heavy components and stable-props components
- Dynamic imports with loading state: `dynamic(() => import('../form'), { loading: () => <Loader />, ssr: false })`
- Always include `children?: never` in PropsType to explicitly disable children
- Use `<>` fragments instead of unnecessary wrapper `<div>`s
- Use early returns for guard clauses
- CSS Modules path: always `./styles/index.module.scss`

## Hooks

- `useCallback`/`useMemo` MUST have complete dependency arrays (no empty `[]` if dependencies exist)
- Event handlers: descriptive names ending with `Handler` — `const onCloseHandler`, NOT generic `handleClick`
- Props accepting handlers: use `on` prefix — `onSubmit={handleSubmitHandler}`
- Always explicit return type: `(): void => { ... }` or `async (): Promise<void> => { ... }`
- Error logging: `logger.client.push({ name: 'hookName error', error })`
- User-facing errors: `prepare.error.message(error)`
- Async handler pattern: try-catch with logger in catch
- State updates depending on previous: use functional form `setPrev(prev => prev + 1)`
- Never mutate state directly
- Beyond 3 levels of prop drilling — use context or custom hooks

## Naming Conventions

- `is` prefix for booleans: `isActive`, `isLoading`
- `JSX` suffix for markup variables: `const titleJSX = ...`
- `Handler` suffix for event handlers: `const onCloseHandler = ...`
- `CX` suffix for classnames: `const containerCX = cx({...})`
- `prev` prefix for previous state values: `const prevCount = usePrevious(count)`
- Entity name must be consistent across all layers — change one, change all in same MR
- One name, one concept — `userId` must refer to the same field everywhere
- Latin/ASCII characters only — no Cyrillic in identifiers, file names, comments
- Root CSS class: matches directory name (for index.tsx) or file name (for non-index)

## UI Kit

- Import components from `'ui-kit'` base path (not subpaths)
- Import types from: `'ui-kit/dist/source/elements/select/types'`
- Never create custom button/input/checkbox when ui-kit provides one
- Available: Stack, OffsetWrapper, GalleryGrid, Button, Checkbox, Select, Typography, Alert, and 20+ more
- `cx()` pattern: `const containerCX = cx({ [styles.container]: true, [styles.isActive]: isActive })`

## Linting

- No `eslint-disable` without justified reason and inline comment
- No `console.log` / `console.error` left in code — must severity
- MUST use yarn, not npm. No `package-lock.json`.
- Dependency versions: exact only — no `^`, `~`, `>=`
- Single quotes everywhere including JSX: `<div className='container'>`
