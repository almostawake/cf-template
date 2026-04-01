# CLAUDE.md

## Reference Documents

- **`CLAUDE-ARCH.md`** — Technology choices, file structure, naming conventions, deployment. Read before writing any code.
- **`CLAUDE-PROCESS.md`** — Feature lifecycle, agent team coordination, review process. Read before planning or starting a feature.

Do not deviate from either document without discussing with the user first.

## Quick Rules

- Read existing code before modifying it. Understand the patterns in use.
- Follow the patterns already established in the codebase. Consistency over novelty.
- Keep changes minimal. Solve what was asked — nothing more.
- Run `npm run check` and `npm run test` after making changes.

## Do Not

- Add dependencies not listed in `CLAUDE-ARCH.md` without asking.
- Create directories outside the structure defined in `CLAUDE-ARCH.md`.
- Write raw SQL — use Drizzle's query builder.
- Hand-edit the `Env` interface or migration files.
- Modify `CLAUDE-ARCH.md`, `CLAUDE.md`, `CLAUDE-PROCESS.md`, or `wrangler.jsonc` without explicit user instruction.
- Add logging, telemetry, or monitoring unless specifically asked.
- Create utility abstractions for one-off operations.
- Create documentation files unless specifically asked.
