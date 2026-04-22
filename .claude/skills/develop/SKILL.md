---
name: develop
description: Content workflow for courses site — runs Planner→Generator→Evaluator, relays fact-check output verbatim, coordinates ISSUES.md updates. Invoke for any task that adds, edits, or reviews .md/.mdx pages under courses/.
---

# /develop — content pipeline orchestration

## Pipeline (mandatory)

Planner → Generator → Evaluator. No step may be skipped.

- Every content push MUST be reviewed by the evaluator before being considered done.
- Direct edits by the orchestrator (e.g. fixing a chart) still require evaluator afterward.
- If evaluator requests changes → fix-and-re-evaluate until approve.

**STOP CHECKPOINT — before marking any task done:**
1. Evaluator invoked after the last push? If not — invoke now.
2. Any direct edits (not via generator)? If yes — invoke evaluator now.
3. Explicit "approve" verdict? If not — task is not done.

Skipping the evaluator is a process failure; log it and run evaluator retroactively.

## Fact-check relay

The fact-checker subagent returns a sentence-by-sentence markdown table with a clickable URL in every verdict row. **Pass this table through to the user verbatim.** Do not compress, paraphrase, or replace it with a prose summary — that is халтура.

## ISSUES.md (orchestrator rule)

The evaluator owns `ISSUES.md` (see evaluator agent definition for format and maintenance rules). Orchestrator-side rule: **do not silently fix listed issues without updating their status in ISSUES.md.** Every fix addressing a listed issue must bump its status to `FIXED (YYYY-MM-DD)`.
