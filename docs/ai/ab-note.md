# A/B note — team harness for Team Tasks

<!-- owner: @vitalii-liashenko · why: proves the harness changes agent behaviour, not just adds files · review-by: 2027-02-23 -->

**Task given to the agent (identical in A and B):**
> Add a `priority` field to tasks: an integer from 1 to 5, default 3. Store it in the database, validate it in the
> business rules, return it from the API, and cover it with tests. Existing behaviour must not change.

Why this task: it touches every layer the harness governs (a migration, a business rule, a route, tests). A plain
agent is likely to edit an existing migration or put validation in a route handler, so the difference should show.

**A — before** (commit `3d0ac87`, no harness) · **B — after** (commit `<sha of the commit with CODEOWNERS>`, full harness)

| Run | Side | Tests green? | Rule violations | Files touched | Turns / cost | Notes |
|---|---|---|---|---|---|---|
| 1 | A | | | | | |
| 2 | A | | | | | |
| 1 | B | | | | | |
| 2 | B | | | | | |

**How to run (manual, each run in a fresh session):**
1. Make a clean worktree for the side: `git worktree add ../ab-A-1 3d0ac87` or `git worktree add ../ab-B-1 <B-sha>`.
2. In that worktree run `npm install`, then start `claude` in a new session. Do not reuse a session between runs.
3. Paste the task prompt above unchanged. Let the agent finish. Do not give hints.
4. Record: the hook messages you saw (B only), `git diff --stat`, `npm test` and `npm run typecheck` results, and the cost or turns shown by `/cost`.
5. Delete the worktree (`git worktree remove ../ab-A-1`) before the next run.

**What changed:** <fill in after the runs: concrete differences in diff and behaviour; quote the hook's message if it fired>

**What didn't change:** <be honest; "no difference" on a row is a valid result>
