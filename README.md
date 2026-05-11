# leo-agaile-builder

Agile-style multi-agent sprint template for Claude Code and Codex.

It provides four role prompts, phase commands, and persistent sprint artifacts
under `.claude/state/sprints/`.

## Quickstart

Use this repo as a template, then run:

```text
/sprint "add a hello-world CLI command"
```

Codex equivalent:

```text
Run a sprint for: add a hello-world CLI command
```

The sprint writes:

```text
.claude/state/sprints/sprint-001-hello-world-cli/
├── 01-prd.md
├── 02-design.md
├── 03-implementation-notes.md
├── 04-review.md
├── 05-retro.md
└── meta.json
```

## Files

```text
.
├── AGENTS.md                  # Canonical rules and workflow contract
├── CLAUDE.md                  # Claude Code entrypoint
├── CODEX.md                   # Codex entrypoint
├── docs/                      # Workflow, conventions, annotations, glossary
├── .claude/
│   ├── config.json
│   ├── agents/
│   ├── commands/
│   └── state/sprints/
├── .codex-plugin/plugin.json
├── agents/                    # Codex role adapters; edit .claude/agents instead
└── commands/                  # Codex command adapters; edit .claude/commands instead
```

`.codex-plugin/plugin.json` describes this template when installed as a Codex
plugin.

## Customize

| Need | Edit |
|---|---|
| Workflow or artifact contract | `AGENTS.md` and `.claude/commands/*.md` |
| Role behavior | `.claude/agents/<role>.md` |
| Sliding-window policy | `.claude/config.json` |
| Annotation fields | `docs/annotations.md` |
| Domain vocabulary | `docs/glossary.md` |

## License

MIT. See `LICENSE`.
