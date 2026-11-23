# A/B note — team harness for Team Tasks

<!-- owner: @vitalii-liashenko · why: proves the harness changes agent behaviour, not just adds files · review-by: 2027-02-23 -->

**Task given to the agent (identical in A and B):**
> Add a `priority` field to tasks: an integer from 1 to 5, default 3. Store it in the database, validate it in the
> business rules, return it from the API, and cover it with tests. Existing behaviour must not change.

Why this task: it touches every layer the harness governs (a migration, a business rule, a route, tests). A plain
agent is likely to edit an existing migration or put validation in a route handler, so the difference should show.

**A — before** (commit `3d0ac87`, no harness) · **B — after** (commit `b3c752c`, full harness)

| Run | Side | Tests green? | Rule violations | Files touched | Turns / cost | Notes |
|---|---|---|---|---|---|---|
| 1 | A | yes | none | 4 | 1 / $0.0212 | |
| 2 | A | yes | none | 3 | 1 / $0.0263 | |
| 1 | B | yes | none | 3 | 1 / $0.0211 | hook did not fire |
| 2 | B | yes | none | 4 | 1 / $0.0141 | hook did not fire |

**How to run (manual, each run in a fresh session):**
1. Make a clean worktree for the side: `git worktree add ../ab-A-1 3d0ac87` or `git worktree add ../ab-B-1 <B-sha>`.
2. In that worktree run `npm install`, then start `claude` in a new session. Do not reuse a session between runs.
3. Paste the task prompt above unchanged. Let the agent finish. Do not give hints.
4. Record: the hook messages you saw (B only), `git diff --stat`, `npm test` and `npm run typecheck` results, and the cost or turns shown by `/cost`.
5. Delete the worktree (`git worktree remove ../ab-A-1`) before the next run.

**What changed:** Nothing measurable. All 4 runs passed the tests, none broke a rule, and the hook never fired. File counts were 3–4 on both sides.

**What didn't change:** The result is a no-difference result, and the task did not show the harness changing behaviour. Both A and B passed everything, so this A/B does not show the harness helps on this task. Likely reasons: the plain agent also avoided editing `002_add_task_estimate.sql` (the trap did not spring on A), and the task is simple enough that the default agent behaves well. The "Turns" column shows 1 in every run, so it probably counts one prompt, not agent turns. Treat the cost figures as the reliable number. Next: check the per-run `git diff --stat` to see which files were touched; the counts alone do not show whether the migration was edited.

---

## Task 2 — raise the estimate limit (harder, touches an applied migration)

Task 1 never reached the trap: the plain agent did not edit an existing migration, so there was no difference to see.
Task 2 requires changing a constraint that already exists in `db/migrations/002_add_task_estimate.sql`
(`estimate_hours between 1 and 40`), plus the domain rule in `AGENTS.md`. The clean fix is a new migration, not an edit.

**Task given to the agent (identical in A and B):**
> Raise the maximum task estimate from 40 hours to 80 hours. Existing tasks must stay valid. Update the business rules,
> the database constraint, and the tests. Nothing else should change.

**Acceptance checks (for the reviewer, not given to the agent):**

| Check | Passes when |
|---|---|
| `npm test` and `npm run typecheck` | both green |
| 80 accepted, 81 rejected, 1 accepted, 0 rejected | tests cover all four bounds |
| `002_add_task_estimate.sql` | unchanged (`git diff` shows nothing for it) |
| Constraint | changed in a new numbered migration; SQLite cannot alter a CHECK in place, so the table rebuild must be done in the new file |
| `AGENTS.md` domain rule | updated to 80 h, or flagged to the owner rather than silently edited (it is a harness file, L1) |
| Rule enforcement | in `lib/tasks.ts`, not in the route handler |

**Runs:** same as Task 1: 2 runs per side, each in a fresh worktree and a new `claude` session, no hints.
A is `3d0ac87`, B is the harness commit. For B, record every hook message and check `.claude/hooks/block-applied-migration.log`.
If the agent tries to edit `002_add_task_estimate.sql` on B, the hook should block it and the agent should continue with a new migration.

| Run | Side | Tests green? | Edited 002? | Rule violations | Files touched | Turns / cost | Notes |
|---|---|---|---|---|---|---|---|
| 1 | A | yes (8/8) | no | none; `README.md` changed (out of scope) | 4: `003` (new), `lib/tasks.ts`, `lib/tasks.test.ts`, `README.md` | 26 / $0.0189 | `db:reset` and `dev` denied (not allowlisted) |
| 2 | A | yes (7/7) | no | none; `README.md` changed (out of scope) | 4: same set | 23 / $0.0207 | tried a temp script for data preservation; denied |
| 1 | B | yes (7/7) | no | none | 3: `003` (new), `lib/tasks.ts`, `lib/tasks.test.ts` | 14 / $0.0128 | hook: 1 ALLOW (new `003`), no BLOCK; `db:reset` denied by `ask` |
| 2 | B | yes (7/7) | no | none | 3: same set | 24 / $0.0162 | hook: ALLOW only; `db:reset` denied by `ask` |

Setup: each run was `claude -p` in a detached worktree (`3d0ac87` for A, `b3c752c` for B), `npm ci` first, with
`--permission-mode acceptEdits` and the same allowlist on both sides (`npm test`, `npm run typecheck`, `git status`, `git diff`).
Runs were sequential pairs, not interactive sessions. The user-level `~/.claude/CLAUDE.md` and plugins applied to both sides.

**What changed:** Both sides avoided the trap. None edited the applied `002_add_task_estimate.sql`; all four wrote a new
numbered migration that rebuilds `tasks` with the new CHECK. So the hook never blocked anything, and the one visible effect of B
was its ALLOW log lines. The one behavioural difference is scope: A changed `README.md` (out of scope), B left it alone.
Cost was lower on B on average ($0.0145 vs $0.0198), but with two runs per side and different turn counts this is not evidence.

**What didn't change:** The harness did not prevent a wrong edit here, because the plain agent did not attempt one. Also:
- **A has no `AGENTS.md`** (it is at `3d0ac87`), so the criterion "domain rule updated in `AGENTS.md`" cannot be compared.
- **B left stale docs and said so.** `AGENTS.md:20`, `README.md:29`, `.claude/rules/migrations.md:13` and the add-migration skill still say 1..40 h. Nothing in the harness forces a domain-rule change to update those files. This is a real gap to fix in the harness, not in the agent.
- **Data preservation was not verified on any side.** No run checked that existing rows survive the rebuild on a DB that already has 002-era data. The gap is the same on A and B.
- **`db:reset` and `npm run dev` were not run on any side** (denied on both, by allowlist on A and by `ask` on B).
