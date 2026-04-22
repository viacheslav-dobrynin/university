# Commit messages

- Never add a message body to commits — use only the subject line.
- Never add "Co-Authored-By" or any other trailers.

# Preflight checks

Run at the start of every session before doing anything else. Do not defer.

- **Playwright MCP**: open the production URL (https://viacheslav-dobrynin.github.io/structures-and-algorithms-in-databases-and-distributed-systems/) and confirm the page loads.
- **GitHub Actions**: `gh run list --workflow=deploy.yml --limit=1` returns results.
- If any preflight check fails, fix it before proceeding.

**Preflight is not optional.** Beginning a task without preflight this session → stop and run it first.

# Content work

For any task that adds, edits, or reviews content pages under `courses/` (`.md` / `.mdx`), invoke the `/develop` skill. It carries the Planner→Generator→Evaluator pipeline, fact-check relay, and ISSUES.md coordination rules.
