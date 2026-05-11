# CODEX.md

Codex should treat `AGENTS.md` as the canonical workflow contract.

## Usage

```text
Run a sprint for: add a hello-world CLI command
Run sprint-plan for sprint-001-hello-world-cli
Run sprint-execute for sprint-001-hello-world-cli
Run sprint-review for sprint-001-hello-world-cli
Run sprint-retro for sprint-001-hello-world-cli
```

## Adapter Rules

- `.claude/commands/*.md` and `.claude/agents/*.md` are canonical.
- `commands/*.md` and `agents/*.md` are Codex adapters only.
- Preserve the two approval gates from `/sprint`.
- If sub-agents are unavailable, perform the role serially in the main Codex
  loop while preserving the same write boundaries.
- Write artifacts under `.claude/state/sprints/`.
