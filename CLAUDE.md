# Commit messages

- Never add a message body to commits — use only the subject line.
- Never add "Co-Authored-By" or any other trailers.

# Agent pipeline

The standard pipeline for content tasks is: **Planner -> Generator -> Evaluator**. This sequence is mandatory — no step may be skipped.

- Every content change that is pushed MUST be reviewed by the evaluator before it is considered done.
- If the orchestrator (user or orchestrating agent) makes changes directly (e.g., fixing a chart), the evaluator MUST still be invoked afterward.
- If the evaluator requests changes, the fix-and-re-evaluate cycle repeats until the evaluator approves.

# Preflight checks

Before starting a new task cycle, verify that critical infrastructure works:
- **Playwright MCP**: confirm the MCP server starts and can open a page (run a quick smoke test).
- **GitHub Actions**: confirm `gh run list` works and the repo has a deploy workflow.
- If any preflight check fails, fix it before proceeding with the task.
