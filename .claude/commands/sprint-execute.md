---
description: Phase 2. Runs Architect, then Developer implementation.
argument-hint: [sprint-id]
---

# /sprint-execute

Run the execution phase for `$ARGUMENTS`.

If omitted, use the most recent `in_progress` sprint with `01-prd.md`.

## Preconditions

- `01-prd.md` exists.
- `01-prd.md` has no blocking Open Questions unless `meta.json.revision_notes`
  contains an entry with `action: "waive_open_question"` for that question.

## Steps

### 1. Architect

Provide Architect with:

- `01-prd.md`
- `meta.json.language`
- `docs/glossary.md`
- `docs/annotations.md`
- `meta.json.revision_notes`
- prior `04-review.md`, if this is a loop-back

Architect filters existing files by matching `@domain` first, then relevant
`@layer`. If no annotations exist for a first or bootstrap sprint, Architect may
inspect only obvious entry points and project configuration files before writing
the design.

Architect writes `02-design.md`. If the PRD is not executable, Architect writes
`ROUTE_BACK: plan` under `## Route Back` with a short reason; the orchestrator
greps this marker and routes back to Plan.

When overwriting an existing `02-design.md` because of a gate revision,
route-back, or repeated phase run, include `Revision History`.

### 2. Developer

Developer reads `02-design.md` and implements only the approved file plan.
Architect, Developer, and implementation notes use `meta.json.language` for
human-readable artifact text. Code identifiers, commands, and existing source
text remain in the project's natural language.

Developer is responsible for:

- code changes
- tests for success criteria, unless the design explicitly justifies none
- approved `docs/glossary.md` additions
- `03-implementation-notes.md`

When overwriting existing notes because of a gate revision, route-back, or
repeated phase run, include `Revision History`.

#### Parallel Developer dispatch

If `02-design.md` has no `## Work Packages` section, run Developer once,
sequentially. Default and unchanged.

If `02-design.md` has `## Work Packages`, the main loop dispatches one
background Developer session per package and merges their partial notes:

1. For each `WP-k` in topological order honoring `Depends on:`, dispatch a
   background Developer session named `<sprint-id>/WP-k`. With Claude Code
   Agent View this is `claude --agent developer --bg "<brief>"` or, from
   `claude agents`, `developer <brief>`. The Developer agent's
   `isolation: worktree` frontmatter makes each session run in its own
   `.claude/worktrees/<id>/` checkout, so parallel sessions cannot stomp on
   each other's files.
2. The brief must include: the sprint id, the absolute path to `02-design.md`,
   the package's `Files:` set, the covered success criteria, the
   `Acceptance checks:`, and the partial notes path
   `.claude/state/sprints/<sprint-id>/03-impl-WP-k.md`.
3. Sessions that share no dependency edge run concurrently. Sessions with
   `Depends on:` an unfinished package wait until that package's session
   reaches `Completed`.
4. The main loop watches each session through `claude agents` (or
   `claude logs <id>` from the shell), answering `Needs input` rows.
5. When every WP session reaches `Completed`, the main loop merges all
   `03-impl-WP-k.md` files into the canonical
   `.claude/state/sprints/<sprint-id>/03-implementation-notes.md`:
   - `Files changed` lists are unioned and deduplicated.
   - `Tests added` and `Verification performed` are unioned.
   - `Deviations from design` and `TODOs left open` are concatenated with the
     emitting `WP-k` prefixed to each entry.
   - The canonical notes file is the only writer of
     `03-implementation-notes.md`; partial files stay on disk for traceability.
6. The main loop then merges each worktree's changes back into the main
   checkout. The recipe and sharp edges live in `docs/AGENT_VIEW.md`.
7. If any WP session ends in `Failed`, edits files outside its package's
   `Files:` set, or skips a declared `Acceptance check:`, treat it as the
   existing "Developer edits files outside its assignment" violation and stop.

### 3. Hand Back

Print:

- paths to `02-design.md` and `03-implementation-notes.md`
- files created, modified, and deleted
- tests and verification commands run
- deviations and TODOs

## Failure Modes

- Architect writes no design: surface the error and stop.
- Developer edits files outside its assignment: surface as blocking and stop.
- Lint/tests fail: record results in implementation notes and hand back for the
  approval gate.
