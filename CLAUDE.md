# Commit messages

- Never add a message body to commits — use only the subject line.
- Never add "Co-Authored-By" or any other trailers.

# Agent pipeline

The standard pipeline for content tasks is: **Planner -> Generator -> Evaluator**. This sequence is mandatory — no step may be skipped.

- Every content change that is pushed MUST be reviewed by the evaluator before it is considered done.
- If the orchestrator (user or orchestrating agent) makes changes directly (e.g., fixing a chart), the evaluator MUST still be invoked afterward.
- If the evaluator requests changes, the fix-and-re-evaluate cycle repeats until the evaluator approves.

**STOP CHECKPOINT — before marking any task done, answer these questions explicitly:**
1. Was the evaluator invoked after the last push? If not — invoke it now.
2. Were any changes made directly (not via generator)? If yes — invoke the evaluator now.
3. Has the evaluator given an explicit "approve" verdict? If not — the task is not done.

Skipping the evaluator is a process failure. It must be logged and the evaluator must be run retroactively.

# Preflight checks

**Run these checks at the start of every session before doing anything else.** Do not defer them to when a tool is first needed — by then a failure will block progress mid-task.

- **Playwright MCP**: open the production URL (https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/) and confirm the page loads.
- **GitHub Actions**: run `gh run list --workflow=deploy.yml --limit=1` and confirm it returns results.
- If any preflight check fails, fix it before proceeding with any task.

**Preflight is not optional.** If you are beginning a task without having run preflight this session, stop and run it first.

# ISSUES.md

`ISSUES.md` at the repo root is the canonical list of known site quality issues.

- **Owner**: the evaluator writes and updates ISSUES.md after every full audit.
- **Format**: issues grouped by priority (P0–P4). Each issue has a short title, description, and status marker (open / FIXED with date).
- **Updates**: when a fix is applied and confirmed by the evaluator, mark the issue `FIXED` inline (do not delete it). When the evaluator runs a new full audit, it may add new issues or close stale ones.
- **The orchestrator must not silently fix issues without updating ISSUES.md.** Every fix that addresses a listed issue must update its status.

# Content quality standards

All content pages (.md / .mdx) must meet these standards:

- **References (Список литературы)**: every content page that makes factual claims, cites algorithms, or presents complexity analysis MUST include a "Список литературы" section at the bottom with at least one authoritative source (textbook, paper, or official documentation). Evaluator will reject pages missing references.
- **Frontmatter title**: every content page must have a YAML frontmatter `title:` field — no URL-slug titles in the sidebar.
- **No stub pages**: pages must have substantive content before being pushed. Empty stubs must not be navigable from the sidebar.
