# Sprint Workflow

`AGENTS.md` defines the contract. This file is a short walkthrough.

## Flow

```text
INPUT
  -> PLAN    -> Gate 1: approve / revise / abort
  -> EXECUTE -> Gate 2: approve / revise / abort
  -> REVIEW  -> ready | loop to execute | loop to plan
  -> RETRO
DONE
```

`/sprint <feature>` creates sprint state first, then enters PLAN. It must not
enter EXECUTE until Gate 1 approves or waives the PRD gate.
The sprint copies `.claude/state/language.json` into `meta.json.language`; all
phase artifacts use that language for human-readable text.

## Phases

| Phase | Command | Owner | Output |
|---|---|---|---|
| Plan | `/sprint-plan [sprint-id]` | PM | `01-prd.md` |
| Execute | `/sprint-execute [sprint-id]` | Architect + Developer | `02-design.md`, code, tests, `03-implementation-notes.md` |
| Review | `/sprint-review [sprint-id]` | QA-Reviewer | `04-review.md` |
| Retro | `/sprint-retro [sprint-id]` | Main loop | `05-retro.md`, finalized `meta.json` |

## Route-Back Rules

- PRD or scope problem: loop to Plan.
- Implementation, test, glossary, annotation, or verification problem: loop to
  Execute.
- Repeated blocking issues are counted by required stable issue id in
  `04-review.md`.

## Sprint Memory

Planning loads recent completed sprints, then the main loop uses a first-pass
domain estimate to load additional completed sprints with matching `domain` or
`tags`. Aborted and in-progress sprints are ignored unless the user asks for
them.

## Minimal Example

```text
/sprint "add a hello-world CLI command"
```

The sprint creates `.claude/state/sprints/sprint-001-hello-world-cli/` with the
five markdown artifacts and `meta.json`. Developer writes the implementation and
tests; QA reviews without editing code.
