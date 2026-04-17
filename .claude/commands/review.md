---
name: review
description: Code review по протоколу проекта. Проверяет архитектурные границы, naming consistency, TypeScript стиль, import order. Usage: /review [file-or-directory]
---

Perform a code review on the specified files or the current git diff if no argument given.

## What to check

### 1. Architecture Boundaries (must)
- No cross-domain imports between `bus/` modules
- No importing Pages from other layers
- Core does not import from Bus or Platform
- Platform does not import from Bus

### 2. Naming Consistency (must)
- Entity name matches across: GraphQL file → generated type → hook name → directory → constant
- Root CSS class matches folder name
- Component suffix `Component` for smart components, no suffix for elements
- `PropsType` for props, `ReturnedType` for hook returns
- All types end with `Type` suffix
- `is` prefix for booleans, `JSX` suffix for markup vars, `CX` suffix for classnames

### 3. TypeScript (must)
- No `as Type` casting
- No `interface` (use `type` only)
- No runtime `enum`
- No bare `any` without comment + ticket
- Named exports (except page default exports)

### 4. Import Order (imho)
- Correct group order: Core > Platform > Elements > Hooks > Templates > Views > Components > Other
- Comment headers present (`// Core`, `// Elements`, etc.)
- Blank lines between groups

### 5. SCSS (must for .scss files)
- `px()` only used in `mq/*.scss` files, not in `index.module.scss`
- Colors through `palette.scss` variables, no direct `var(--...)`
- Max nesting depth 3
- Root class matches folder name

### 6. Security (must)
- No raw `dangerouslySetInnerHTML` — must use `InnerHTML` component or `DOMPurify.sanitize()`
- Latin characters only in identifiers

### 7. React Patterns (imho)
- Props destructuring: ≤3 in signature, >3 in body
- `useCallback`/`useMemo` have complete dependency arrays
- Event handlers have explicit return types
- Errors logged with `logger.client.push`

## Severity levels

- **must** — blocking: bug, regression, security, rule violation, dead code
- **imho** — non-blocking: readability, convention preference, architecture suggestion
- **note** — neutral: question, clarification

## Output format

```
## Review: {file(s)}

### must
- [{file}:{line}] Description of issue

### imho
- [{file}:{line}] Suggestion

### note
- [{file}:{line}] Question or observation

### Summary
N must / N imho / N note
{verdict: approve / request changes}
```

If reviewing git diff (no arguments), run `git diff --cached` first, then `git diff` for unstaged changes.
