# CODEX.md

Codex should treat `AGENTS.md` as the canonical workflow contract.

## Usage

```text
Run a sprint for: add a hello-world CLI command
Set sprint language to Korean
Run sprint-plan for sprint-001-hello-world-cli
Run sprint-execute for sprint-001-hello-world-cli
Run sprint-review for sprint-001-hello-world-cli
Run sprint-retro for sprint-001-hello-world-cli
```

## Adapter Rules

- `.claude/commands/*.md` and `.claude/agents/*.md` are canonical.
- `commands/*.md` and `agents/*.md` are Codex adapters only.
- Language selection is handled by `.claude/commands/lang.md`; new sprints copy
  `.claude/state/language.json` into `meta.json.language`.
- A new sprint starts with `/sprint-plan` after state creation; do not proceed
  to `/sprint-execute` before the PRD approval gate.
- Preserve the two approval gates from `/sprint`.
- If sub-agents are unavailable, perform the role serially in the main Codex
  loop while preserving the same write boundaries.
- Write artifacts under `.claude/state/sprints/`.
- Claude Code Agent View (`docs/AGENT_VIEW.md`) is a Claude-Code-only runtime
  for parallel Developer dispatch. Codex sprints follow the same contract
  sequentially; the `## Work Packages` section in `02-design.md`, when
  present, is consumed by a single Developer in dependency order.
