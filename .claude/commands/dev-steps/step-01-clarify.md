# Step 1 — CLARIFY (Analyst, Sonnet)

## Agent Loading

First, read `{project-root}/.claude/agents/analyst.md` and adopt that identity fully. You are the Analyst for this step.

You are an isolated sub-agent. You have NO prior context. Everything you need is in the session files and the codebase.

## Execution Mode — MEDIUM

- **Intent:** exploration + question formulation + context pack assembly
- **Effort:** medium — balance breadth of exploration with decisiveness; don't explore endlessly
- **Thinking:** focused — enough to spot risks, gaps, and affected areas; not enough to plan solutions
- **Output:** structured findings in spec.md + numbered questions in questions.md + factual context-pack.md

## Your job

Analyze the user's task, explore the codebase, gather context, formulate questions.

## Instructions

1. Read `{session-dir}/spec.md` to get the task description.

2. Explore the codebase at `{project-root}/front/src/` to understand what exists:
   - Which `bus/` domain(s) does this task touch?
   - Which layer(s): pages, core, bus, platform, init?
   - Will it need new files or modify existing?
   - Does it involve GraphQL (new queries/mutations)?
   - Does it involve SCSS changes?
   - Read the key files that will be affected.

3. Check for architecture constraints:
   - Cross-domain imports between bus/ modules are forbidden
   - core/ cannot import from bus/ or platform/
   - platform/ cannot import from bus/
   - Only pages/ can import from everything

4. Update `{session-dir}/spec.md` Context section:
   ```markdown
   # Context
   ## Understanding
   {2-3 sentences: what you understood the task to be}

   ## Affected domains
   - bus/{domain}: {why}

   ## Affected files (existing)
   - {path}: {what changes needed}

   ## New files needed
   - {path}: {purpose}

   ## Dependencies
   - {any hooks, components, types that will be needed from other modules}

   ## Risks
   - {architecture violations, breaking changes, edge cases}
   ```

5. **MANDATORY: Formulate questions.** Write `{session-dir}/questions.md`:
   ```markdown
   # Questions

   ## Questions
   1. {question about scope — one page? all themes? one theme?}
   2. {question about implementation approach if multiple options exist}
   3. {question about edge cases, mobile/desktop, existing behavior preservation}

   ## Success criteria (propose, ask for confirmation)
   Propose 3-5 verifiable criteria that describe "task done correctly" from the user's perspective — not from a convention/lint perspective. These are semantic goals, not technical checks.

   Examples of GOOD success criteria:
   - User can filter profiles by country and see only matching profiles
   - Existing search functionality still works (regression check)
   - Filter is visible on mobile (breakpoint: phonePortrait)
   - Empty result state shows "no profiles match" instead of blank page

   Examples of BAD success criteria (avoid — these are convention checks, not goals):
   - ❌ TypeScript compiles without errors (that's validate step)
   - ❌ Import order follows conventions (that's review step)
   - ❌ Uses px() in mq files (that's scss-conventions.md)

   End this section with: "Confirm or edit these criteria before Step 2 starts."
   ```

   You MUST ask at least 1 question AND propose at least 3 success criteria, even if the task seems clear. The criteria surface hidden intent that neither the task description nor the conventions capture.

6. **MANDATORY: Build Context Pack.** Write `{session-dir}/context-pack.md` — a self-contained snapshot of relevant project code that downstream steps (Plan / Review / Fix) will read INSTEAD of re-reading source files. This is the single biggest token-saver in the pipeline.

   ### Pack structure
   ```markdown
   # Context Pack

   _Built by Analyst in Step 1. Downstream agents (Architect, Developer) read this instead of re-reading source files. Updates after Step 3 live in `context-pack.delta.md`._

   ## Affected files (full content)
   {For each file in the spec.md "Affected files" list — paste its FULL content in a fenced block tagged with the path. Example below.}

   ### front/src/bus/profile/components/profileCard/index.tsx
   ```tsx
   {full file content here}
   ```

   ### front/src/bus/profile/components/profileCard/styles/index.module.scss
   ```scss
   {full file content here}
   ```

   ## Reference patterns (snippets only)
   {When the task is "do X like component Y already does", paste 20-50 line snippets from Y as inspiration. Snippets, not full files.}

   ### Similar pattern: front/src/bus/profile/components/searchFilters/index.tsx
   ```tsx
   {30-50 relevant lines}
   ```

   ## Type definitions in scope
   {Paste type/interface definitions that the affected files import. Pull only the types, not the whole types file.}

   ```ts
   // from front/src/bus/profile/types/profileType.ts
   export type ProfileType = { ... };
   ```

   ## Glossary
   {Optional: 1-line definitions of domain-specific names the architect should know.}
   ```

   ### Pack rules
   - **Hard cap: 80KB total.** Use `wc -c` to verify after writing. If over 80KB — drop reference patterns first, then trim less-relevant types, then ask user to narrow the scope (write a warning into spec.md and questions.md).
   - **Full content** for files in "Affected files (existing)" from spec.md Context. Truncating these is dangerous because the architect needs to see the complete current state.
   - **Snippets only** for reference patterns and types. 20-50 lines per snippet, focused on what's relevant.
   - **No file > 50KB raw**. If a single affected file is huge (e.g. a 2000-line legacy component), include only the imports + type signatures + the specific functions touching the task; note the truncation explicitly in the file's section.
   - **Self-validation**: after writing, grep the pack for the names of the key entities the task mentions. If grep returns zero, the pack is incomplete — fix it before exiting.

   Run after writing the pack:
   ```bash
   wc -c "{session-dir}/context-pack.md"
   ```
   Print the size in your output. If > 80KB — write a warning to spec.md Risks section.

7. Update spec.md status to `step: clarified`.
