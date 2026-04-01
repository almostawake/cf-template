# Process

This document describes how features move from idea to merged code, how agent teams coordinate, and how the project stays consistent over time.

## Feature Lifecycle

### 1. Define

The user describes the feature in conversation. This doesn't need to be a formal spec — a clear statement of what the user sees or does and any constraints is enough.

Before any code is written, confirm:
- What is the user-facing outcome?
- Does this require new database tables or changes to existing ones?
- Does this need authentication?
- Does this fall within the patterns in `CLAUDE-ARCH.md`, or does it require an architectural decision first?

If the feature touches a pattern listed under "When to Escalate" in `CLAUDE-ARCH.md`, stop and extend `CLAUDE-ARCH.md` before proceeding.

### 2. Plan

Break the feature into work units before writing code. Use plan mode or discuss with the user. Each work unit must be:

- **Independently implementable.** No unit should depend on another unit's uncommitted work.
- **Non-conflicting.** Two units should not modify the same files. If they must, sequence them — don't parallelise.
- **Vertically sliced.** A unit that adds a new resource owns the full stack: schema, migration, API endpoint, validation schema, and test. Never split a single resource across agents.

A good plan looks like:
```
Feature: Project management

Unit 1 — Projects resource
  Files: src/lib/db/schema.ts, src/lib/validation.ts,
         src/pages/api/projects.ts, src/pages/api/projects/[id].ts,
         tests/api/projects.test.ts
  Agent: worktree

Unit 2 — Projects list page (depends on Unit 1)
  Files: src/pages/projects.astro, src/components/ProjectCard.astro
  Agent: after Unit 1 merges

Unit 3 — Project detail page (depends on Unit 1)
  Files: src/pages/projects/[id].astro, src/components/ProjectStatus.svelte
  Agent: after Unit 1 merges
```

Note how Units 2 and 3 can run in parallel after Unit 1 completes, because they don't touch the same files.

**What the plan must include for each unit:**
- The files that will be created or modified (list them explicitly)
- Whether it runs in a worktree or the main tree
- Dependencies on other units, if any

### 3. Execute

Agents work on their units. Each agent follows these rules:

**Before starting:**
- Read `CLAUDE-ARCH.md` for the relevant sections.
- Read any existing files you'll be modifying.
- If you're adding to the database schema, read the current `src/lib/db/schema.ts`.

**While working:**
- Follow the patterns in `CLAUDE-ARCH.md` and the conventions already in the codebase.
- Stay within scope. Do not fix, refactor, or improve code outside your unit.
- If you discover something broken or inconsistent that's outside your scope, note it in your output — don't fix it silently.

**Before reporting back:**
- Run `npm run check` to fix formatting and lint.
- Run `npm run test` to verify nothing is broken.
- Report what you created/modified and any issues encountered.

### 4. Integrate

After agents complete their work:

- Merge worktree branches into the main branch.
- Run the full test suite: `npm run test`.
- Run `npm run check:ci` to verify formatting.
- Resolve any integration issues (import paths, type mismatches between units).

If a unit's tests pass in isolation but fail after integration, the integration fix is done in the main tree, not by re-running the agent.

### 5. Review

After integration, review the complete change:

- **Completeness:** Does the implementation match what was asked? Are there missing cases?
- **Consistency:** Does the code follow `CLAUDE-ARCH.md`? Are naming conventions followed?
- **Tests:** Does every API endpoint have a test? Do tests cover at least the happy path and one error case?
- **Scope:** Was anything changed that wasn't part of the feature? If so, why?
- **Dependencies:** Were any new packages added? Are they in the approved list in `CLAUDE-ARCH.md`?

If review finds issues, fix them directly — don't re-run the full agent workflow for small corrections.

### 6. Update Documentation

After the feature is complete and reviewed:

- **`CLAUDE-ARCH.md`** — Update only if the feature introduced a new architectural pattern, a new service binding, a new dependency, or a decision that future work needs to know about. Do not update `CLAUDE-ARCH.md` for routine feature additions.
- **`CLAUDE-PROCESS.md`** — Update if the team learned something about coordination that should change the process. This should be rare.
- **No feature-level docs** — Do not create README files, docs folders, or markdown files describing individual features unless the user specifically asks. The code, tests, and `CLAUDE-ARCH.md` are the documentation.


## Agent Coordination Rules

These rules prevent agents from stepping on each other:

### File ownership

- Each work unit lists its files upfront. No agent modifies files outside its list.
- `src/lib/db/schema.ts` is a shared file — only one agent modifies it at a time. If multiple units need schema changes, sequence them.
- `wrangler.jsonc`, `CLAUDE.md`, `CLAUDE-ARCH.md`, and `CLAUDE-PROCESS.md` are never modified by agents unless the user explicitly instructs it.
- `package.json` changes (new dependencies) require user approval. Agents should note "this unit needs package X" in their output rather than installing it.

### Worktree usage

- Use worktrees (`isolation: "worktree"`) for units that create or modify multiple files, especially when other agents are working in parallel.
- Small, single-file changes (e.g., adding a component) can work in the main tree if no other agent is active.
- Worktree agents cannot see each other's changes until merged. Plan accordingly — don't depend on another worktree agent's output.

### Sequencing

- Units that modify the database schema run first. Everything that depends on the schema waits.
- Frontend units that depend on API endpoints wait for those endpoints to be merged.
- Independent frontend units (pages that don't share components) can run in parallel.

### Communication

- Agents report back to the orchestrating conversation with: what was done, what files were changed, and any issues or questions.
- If an agent encounters ambiguity (the task could be interpreted multiple ways), it should ask rather than guess.
- If an agent discovers a bug or inconsistency outside its scope, it reports it but does not fix it.


## Code Review Checklist

Use this when reviewing agent output or completed features:

- [ ] Does it match what was requested? Nothing more, nothing less.
- [ ] Does it follow `CLAUDE-ARCH.md` patterns (file location, naming, error handling, validation)?
- [ ] Are Cloudflare bindings accessed via `locals.runtime.env`?
- [ ] Is input validated with Zod before reaching the database?
- [ ] Does every endpoint have a top-level try/catch?
- [ ] Are tests written and do they pass?
- [ ] Does `npm run check:ci` pass?
- [ ] Are there any new dependencies? Are they approved in `CLAUDE-ARCH.md`?
- [ ] Is the database schema change in `src/lib/db/schema.ts` (not scattered)?
- [ ] Are IDs generated with `crypto.randomUUID()`?
