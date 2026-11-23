# 2026-spd-ws1-homework-starter

Starter repo for **Homework 1.1 «Your team harness, in one PR»** of the SPD «Team AI Engineering» workshop.
A tiny Next.js app (Team Tasks) with an API, SQLite migrations and tests. The app is small on purpose: the homework
is the agent harness around it.

**Start here: [HOMEWORK.md](HOMEWORK.md)** — task, deliverables, how to submit, rubric.

## Run it

Needs Node.js 22.13 or newer (Node 24 LTS recommended) and Claude Code 2.1.281 or newer.

```bash
npm install
```

```bash
npm test
```

```bash
npm run dev
```

The app is at http://localhost:3000, the API at `/api/tasks` (`GET`, `POST`) and `/api/tasks/<id>` (`PATCH` with a status).

| Where | What |
|---|---|
| `lib/tasks.ts` | business rules: title and estimate limits, `todo → doing → done`, done is final (the code is the source of the numbers) |
| `lib/db.ts` | SQLite via `node:sqlite`, applies `db/migrations/*.sql` in order |
| `db/migrations/` | applied migrations — a good candidate for a protecting hook |
| `app/api/` | route handlers |
| `docs/ai/templates/` | templates for the homework |

There is deliberately **no** `AGENTS.md`, no `.claude/` and no CODEOWNERS yet. Adding them is the homework.
Note: `npm run dev` (Next.js 16) creates an `AGENTS.md` with a managed Next.js block; see HOMEWORK.md.
