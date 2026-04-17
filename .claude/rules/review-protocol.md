---
globs: "**/*.ts,**/*.tsx,**/*.scss,**/*.graphql"
---

# Code Review Protocol

## Severity Scale (ONLY three levels)

- **must** — blocking: bug, regression, security hole, explicit rule violation, dead code, incident risk. Format: two sentences max + "Fix:" line required.
- **imho** — non-blocking: readability, convention preference, architecture suggestion. Format: single sentence, no Fix: line.
- **note** — neutral: question, observation, clarification. Format: single sentence.

Do NOT use: high, low, critical, should, warning, or any other severity labels.

## Comment Language

All review comments MUST be in Russian (ru).

## Comment Format

```
**must**: Description of the issue.
Fix: What needs to change.

**imho**: Suggestion for improvement.

**note**: Question or observation.
```

## Review Checklist

1. Architecture boundaries (must) — no cross-domain bus/ imports, layer violations
2. Naming consistency (must) — entity name matches across GraphQL/hook/type/directory
3. TypeScript style (must) — no `as`, no `interface`, no `enum`, `Type` suffix
4. Import order (imho) — 8 groups with comment headers
5. SCSS conventions (must) — px() placement, palette, root class naming
6. Security (must) — DOMPurify, no raw dangerouslySetInnerHTML
7. Console.log removal (must) — no console.log/console.error in committed code
8. Unused code (must) — no dead imports, variables, functions
