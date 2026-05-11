# AGENTS.md — Project Rules for AI Coding Agents

This file defines the rules, workflow, and artifact contracts for projects built
with **leo-agaile-builder**. `CLAUDE.md`, `CODEX.md`, role prompts, and command
adapters must defer to this file when contracts overlap.

---

## 1. Operating Principles

Agents must apply these four principles on every sprint:

1. **Think before coding.** State assumptions. Surface ambiguity. Ask when the
   next step would otherwise invent scope.
2. **Simplicity first.** Build the smallest thing that satisfies the approved
   success criteria. Do not add speculative abstractions or features.
3. **Surgical changes.** Touch only files required by the sprint. Mention
   unrelated cleanup; do not perform it.
4. **Goal-driven execution.** Convert work into verifiable criteria, implement
   against them, and record the checks that were run.

See `docs/PRINCIPLES.md` for examples.

---

## 2. Project Rules

### Rule 1. Domain-Driven Organization

Organize code by domain first, then by layer. The default layers are:

| Layer | Responsibility |
|---|---|
| `domain` | Entities, value objects, aggregates, domain services. Pure business logic, no I/O. |
| `application` | Use cases and application services. Orchestrates domain objects and uses interfaces for infrastructure. |
| `infrastructure` | Persistence, external APIs, framework bindings, and implementations of application interfaces. |
| `interfaces` | HTTP, CLI, UI, or other external entry points. |

Domain terms belong in `docs/glossary.md`. PM proposes glossary additions in
`01-prd.md`; Developer applies approved additions when code introduces the term;
QA marks missing glossary updates as blocking.

### Rule 2. File-Header Annotations

Repo-authored source, test, commentable schema, and migration files must start
with an annotation header. Generated files, vendored files, lockfiles, binaries,
docs, sprint artifacts, and formats that cannot carry comments are excluded
unless the project defines a wrapper convention.

Required fields:

```text
@domain
@layer
@purpose
```

Annotation fields are registered in `docs/annotations.md`. New fields must be
added to the registry before use. Projects should enforce required annotations
through their lint/check pipeline; QA also verifies them.

### Rule 3. Sprint Memory and Sliding Window

Sprints persist artifacts under `.claude/state/sprints/`. Planning loads bounded
prior context using `.claude/config.json -> sprintHistory`.

Selection policy:

1. Load the `recentCount` most recent `completed` sprints.
2. The `/sprint-plan` main loop makes a first-pass domain estimate from the
   feature request, existing glossary, and prior sprint metadata. This estimate
   is only for context loading.
3. Also load older `completed` sprints whose `domain` or `tags` match the
   estimated domain.
4. Also load `completed` sprints whose `domain` or `tags` are in
   `alwaysIncludeDomains`.
5. Apply `tokenBudget` by dropping the oldest selected sprints first.

`aborted` and `in_progress` sprints are excluded unless the user explicitly asks
to inspect them.

### Rule 4. Language-Neutral Conventions

Each project chooses its own stack. The template only requires:

- Automated lint/format or equivalent checks.
- Required file-header annotations.
- Consistent DDD layer names within the project.
- No commented-out code.
- Tests for approved success criteria, written by Developer unless the design
  explicitly justifies why none are needed.

See `docs/CONVENTIONS.md`.

---

## 3. Sprint Workflow

Triggered by `/sprint <feature description>` in Claude Code, or by asking Codex
to run a sprint.

1. `/sprint-plan` creates or updates `01-prd.md`.
   - Gate: user approves, revises, or aborts the PRD.
2. `/sprint-execute` writes `02-design.md`, then Developer implements code,
   tests, glossary updates, and `03-implementation-notes.md`.
   - If Architect rejects the PRD, route back to `/sprint-plan`.
   - Gate: user approves, revises, or aborts the implementation.
3. `/sprint-review` writes `04-review.md`.
   - If implementation issues exist, route back to `/sprint-execute`.
   - If PRD/scope issues exist, route back to `/sprint-plan`.
4. `/sprint-retro` writes `05-retro.md` and finalizes `meta.json`.

The main loop appends gate feedback and route-back-to-Plan summaries to
`meta.json.revision_notes` and passes them to the next phase invocation.
Route-backs to `/sprint-execute` rely on `04-review.md` as the trace and are
not duplicated into `revision_notes`; this keeps execution feedback in the
review artifact instead of copying it into sprint metadata.
Each entry must use this shape:

```json
{
  "phase": "plan | execute | review",
  "at": "<ISO 8601 UTC timestamp>",
  "source": "gate | review | architect",
  "action": "revise | waive_open_question | route_back",
  "note": "<human-readable feedback or reason>"
}
```

Revising or re-executing overwrites the phase artifact; `01-prd.md`,
`02-design.md`, and `03-implementation-notes.md` must include a short
`Revision History` section when replacing prior content because of a gate
revision, route-back, or repeated phase run.
Loop-backs keep `meta.json.status` as `in_progress`.

---

## 4. Role Boundaries

| Role | Reads | Writes |
|---|---|---|
| PM | feature request, glossary, selected prior sprints, revision notes | `01-prd.md`; `meta.json.domain` |
| Architect | approved PRD, glossary, annotations, relevant files, review feedback if any | `02-design.md` |
| Developer | approved design, referenced files, glossary, annotations | code, tests, `docs/glossary.md`, `03-implementation-notes.md` |
| QA-Reviewer | PRD, design, implementation notes, changed files | `04-review.md` only |
| Main loop | all artifacts | `05-retro.md`; `meta.json.status`; `meta.json.completed_at`; `meta.json.tags`; `meta.json.revision_notes` |

QA-Reviewer must not edit implementation code or tests. Missing tests for
approved success criteria are blocking. Optional follow-up tests are non-blocking
and should become retro action items.

Parallel Developer agents are allowed only when `02-design.md` assigns disjoint
file sets. The main loop merges partial implementation notes.

---

## 5. Artifact Contract

Each sprint directory has this shape:

```text
.claude/state/sprints/sprint-NNN-<slug>/
├── 01-prd.md
├── 02-design.md
├── 03-implementation-notes.md
├── 04-review.md
├── 05-retro.md
└── meta.json
```

`meta.json`:

```json
{
  "sprint_id": "sprint-001-add-login",
  "slug": "add-login",
  "domain": "auth",
  "tags": [],
  "status": "in_progress",
  "created_at": "2026-05-11T10:00:00Z",
  "completed_at": null,
  "feature_request": "<verbatim user input>",
  "revision_notes": []
}
```

Allowed `status` values:

- `in_progress`
- `completed`
- `aborted`

`tags` is an array of additional domain strings touched by the sprint, excluding
the primary `domain`, and is finalized by the main loop during retro. Sprint
numbers may skip when a sprint is aborted; the next sprint id is still one more
than the highest existing sprint number.
`revision_notes` remains in `meta.json` after completion for auditability.

Review issues must include stable ids, for example
`ISSUE-001-missing-login-test`, so repeated blocking issues can be counted
across review loops. QA assigns issue ids monotonically within a sprint and
keeps the same id for the same underlying defect across review loops. The
repeated-issue counter resets after user revision, route-back to Plan, abort, or
a new sprint.

---

## 6. Pointers

- Sprint workflow: `docs/AGILE.md`
- Code conventions: `docs/CONVENTIONS.md`
- Annotation registry: `docs/annotations.md`
- Glossary: `docs/glossary.md`
- Principle examples: `docs/PRINCIPLES.md`
