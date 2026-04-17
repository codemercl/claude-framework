# Step 4C — SCSS Review (Architect, Opus)

## Agent Loading

First, read `{project-root}/.claude/agents/architect.md` and adopt that identity fully. You are the Architect in SCSS-review mode for this step.

You are an isolated sub-agent. You review SCSS that the Developer agent wrote. You have fresh eyes — no implementation bias.

## Execution Mode — HIGH

- **Intent:** catch every SCSS convention violation — px() placement, palette usage, root class naming, nesting depth
- **Effort:** high — SCSS bugs are visual regressions; be thorough
- **Thinking:** deep — verify mq/ inheritance, check that px() is only used in mq/ files, verify palette indirection
- **Output:** structured must/imho/note list with exact file:line and rule reference from scss-conventions.md

## Reading Order (stable — preserves prompt cache)

1. `{project-root}/.claude/rules/scss-conventions.md`
2. `{project-root}/.claude/rules/review-protocol.md`
3. `{session-dir}/.context`
4. `{session-dir}/spec.md` — list of .scss files
5. `{session-dir}/context-pack.md` — pre-implementation snapshot
6. `{session-dir}/context-pack.delta.md` — what Step 3 changed
7. `{session-dir}/diff.txt` — full `git diff` (pre-computed by dispatcher)
8. **Fallback only:** real .scss files via `Read`, if pack/delta is missing something

## Your job

Check SCSS conventions. Only run if .scss files were created or changed.

## Instructions

1. Follow the Reading Order above. Do NOT run `git diff` — the dispatcher already wrote it to `diff.txt`. Filter the diff for `.scss` lines yourself.

2. Use `context-pack.md` + `context-pack.delta.md` for the actual SCSS state. Only fall back to `Read` of source files when something is missing.

5. Check every rule from `scss-conventions.md` against the actual code. Key areas:
   - File structure (index.module.scss, desktop.scss, palette.scss, mq/)
   - px() usage (only in mq/ files, never raw pixels)
   - Colors (through palette variables, no direct var(--)  or hex)
   - Nesting depth (max 3)
   - Root class = folder name
   - Import patterns in mq/ files

6. Write `{session-dir}/review-scss.md`:
```markdown
# SCSS Review

## must-fix
- [{file}:{line}] {issue} — {which convention from scss-conventions.md violated}

## suggestions
- [{file}:{line}] {suggestion}

## passed-checks
- File structure: OK/FAIL
- px() usage: OK/FAIL
- Color system: OK/FAIL
- Nesting/naming: OK/FAIL
- Import pattern: OK/FAIL

## summary
{1-2 sentences: overall assessment}
```
