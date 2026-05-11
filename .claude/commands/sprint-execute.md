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

Developer is responsible for:

- code changes
- tests for success criteria, unless the design explicitly justifies none
- approved `docs/glossary.md` additions
- `03-implementation-notes.md`

If the design allows parallel Developer agents, the main loop assigns disjoint
file sets. Parallel Developers must not write the canonical
`03-implementation-notes.md`; they return or write assignment-scoped partial
notes for the main loop to merge into one `03-implementation-notes.md`.

When overwriting existing notes because of a gate revision, route-back, or
repeated phase run, include `Revision History`.

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
