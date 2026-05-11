# leo-agile-builder

Agile-style multi-agent sprint template for Claude Code and Codex.

It provides four role prompts, phase commands, and persistent sprint artifacts
under `.claude/state/sprints/`.

[한국어](#한국어) · [English](#english)

---

## 한국어

### 사용법

이 레포를 GitHub 템플릿으로 사용한 뒤 다음 명령으로 시작합니다.

```text
/lang ko
/sprint "hello-world CLI 명령 추가"
```

Codex에서는:

```text
스프린트 언어를 한국어로 설정
sprint 실행: hello-world CLI 명령 추가
```

스프린트가 만드는 파일:

```text
.claude/state/sprints/sprint-001-hello-world-cli/
├── 01-prd.md
├── 02-design.md
├── 03-implementation-notes.md
├── 04-review.md
├── 05-retro.md
└── meta.json
```

새 스프린트는 항상 Plan 단계에서 시작하며, PRD 승인 게이트가 통과되거나
명시적으로 waive된 후에만 Execute 단계로 진행합니다.

`/lang <언어>` 로 이후 스프린트 아티팩트의 언어를 설정합니다. 현재 설정은
`/lang show` 로 확인합니다.

### 커스터마이즈

| 변경 대상 | 수정 위치 |
|---|---|
| 워크플로우 / 아티팩트 계약 | `AGENTS.md`, `.claude/commands/*.md` |
| 역할 동작 | `.claude/agents/<role>.md` |
| 아티팩트 언어 | `/lang <언어>` 또는 `.claude/state/language.json` |
| 슬라이딩 윈도우 정책 | `.claude/config.json` |
| 어노테이션 필드 | `docs/annotations.md` |
| 도메인 용어 | `docs/glossary.md` |

---

## English

### Usage

Use this repo as a GitHub template, then run:

```text
/lang en
/sprint "add a hello-world CLI command"
```

Codex equivalent:

```text
Set sprint language to English
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

New sprints always begin with the Plan phase. Execution starts only after the
PRD approval gate is approved or explicitly waived.

`/lang <language>` sets the language for future sprint artifacts. Use
`/lang show` to inspect the current setting.

### Customize

| Need | Edit |
|---|---|
| Workflow or artifact contract | `AGENTS.md` and `.claude/commands/*.md` |
| Role behavior | `.claude/agents/<role>.md` |
| Artifact language | `/lang <language>` or `.claude/state/language.json` |
| Sliding-window policy | `.claude/config.json` |
| Annotation fields | `docs/annotations.md` |
| Domain vocabulary | `docs/glossary.md` |

---

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
│   └── state/
│       ├── language.json
│       └── sprints/
├── .codex-plugin/plugin.json
├── agents/                    # Codex role adapters; edit .claude/agents instead
└── commands/                  # Codex command adapters; edit .claude/commands instead
```

`.codex-plugin/plugin.json` describes this template when installed as a Codex
plugin.

## Template usage

This repository is meant to be used via GitHub's **Template repository** feature
(Settings → General → check "Template repository").

When a new repo is created from this template, the workflow at
`.github/workflows/template-cleanup.yml` runs once on the first push to `main`
and:

- replaces this README with a minimal `# <repo-name>` stub, and
- removes the cleanup workflow itself.

The cleanup is skipped on the template repository (where
`repository.is_template == true`), so this README stays intact here.

## License

MIT. See `LICENSE`.
