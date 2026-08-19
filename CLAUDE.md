# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm install` — install dependencies
- `npm run dev` — run the server with auto-reload (`node --watch server.js`)
- `npm start` — run the server normally
- `npm test` — run the full test suite (Node's built-in test runner, via `node --test`)
- `npm test -- tests/users.test.js` — run a single test file
- `npm run lint` — run ESLint over the whole repo

There is no build step; this is plain CommonJS Node.js.

## Architecture

This is a small Express API with a layered structure:

- `server.js` — creates the Express app, mounts route modules under their base paths (`/users`, `/health`), and only calls `app.listen` when run directly (`require.main === module`). This lets `tests/*.test.js` `require("../server")` and drive it with `supertest` against an unstarted app.
- `routes/*.js` — one router module per resource. Routes validate input and shape HTTP responses (status codes, error bodies), but never touch data directly — they delegate to `db/store.js`.
- `db/store.js` — the only place that owns data. It's an in-memory array-backed store standing in for a real database; state resets on every server restart. All data access (reads and writes) goes through functions exported here, not through direct array manipulation in routes.

The intended flow for a new resource action is: route handler validates the request → calls a `db/store.js` function → route handler maps the result to an HTTP response (200/201/404/400, etc.).

## Tests

Tests use Node's built-in `node:test` + `node:assert`, with `supertest` for HTTP assertions against the exported `app`. Files under `tests/` ending in `.test.js` are auto-discovered by `node --test`.

Some test files are grading/acceptance tests for course exercises (e.g. `tests/update-user.test.js`, `tests/notes.test.js`) and are called out as "don't edit" in comments at the top of the file — treat that as authoritative and implement against them rather than modifying them, unless the user explicitly says otherwise.

`tests/notes.test.js` asserts that a `NOTES.md` file (uppercase, at the repo root) exists and has real content — this is a course deliverable, not application code.

## CI

`.github/workflows/ci.yml` runs on every push/PR: `npm install`, `npm run lint`, `npm test`, on Node 22. Keep changes lint-clean and green under `npm test` before opening a PR.
