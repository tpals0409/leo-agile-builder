# Agent View

Optional Claude Code runtime for running parts of a sprint as parallel
background sessions. Sprints still follow the contract in `AGENTS.md`; Agent
View only changes **how the Execute phase is dispatched**.

Canonical reference: <https://code.claude.com/docs/en/agent-view>. Requires
Claude Code v2.1.139 or later (`claude --version`). Codex does not have Agent
View; Codex sprints stay sequential.

## What Agent View gives this template

- `claude agents` opens one dashboard for every background Claude Code session
  on the machine. Each row shows the session's state (`Working`,
  `Needs input`, `Completed`, `Failed`, `Idle`, `Stopped`) and its last
  one-line summary.
- `claude --bg "<prompt>"` (or `/bg` inside an interactive session) sends a
  session straight to the background. It keeps running with no attached
  terminal.
- `claude --agent <name> --bg "<prompt>"`, or `@<name>` / `<name> <prompt>`
  inside the dashboard, runs the named subagent (`pm`, `architect`,
  `developer`, `qa-reviewer`) as the session's **main agent**, with its full
  frontmatter applied.
- The `developer` agent in this template is declared with `isolation: worktree`,
  so every background dispatch lands in its own `.claude/worktrees/<id>/`
  checkout. Parallel writers cannot collide.
- Any question the agent asks the user surfaces as a `Needs input` row in the
  dashboard. The user answers from the peek panel (`Space`) without attaching.

## When to use it for a sprint

| Phase | Sequential | Agent View parallel |
|---|---|---|
| Plan | default | not useful — PRD is one document |
| Execute | default | recommended when Architect emits `## Work Packages` |
| Review | default | not in scope of this template; QA is short and read-only |
| Retro | default | not useful — main-loop only |

The PRD approval gate and the implementation approval gate still apply. They
arrive as `Needs input` rows when you ran `/sprint` itself in the background;
otherwise they arrive in the attached session exactly as today.

## Runbook A — parallel Execute

Run this after Gate 1 (PRD approval). It mirrors the dispatch recipe in
`.claude/commands/sprint-execute.md` Step 2.

1. **Confirm Work Packages.** Open
   `.claude/state/sprints/<sprint-id>/02-design.md` and confirm the
   `## Work Packages` section exists. Each package must have a strictly
   disjoint `Files:` set, a `Success criteria covered:` list, an optional
   `Depends on:`, and `Acceptance checks:`. If the section is missing, run
   Execute sequentially.
2. **Dispatch one session per package.** From a shell inside the project
   directory, for each `WP-k`:

   ```bash
   claude --agent developer --bg \
     "Implement WP-k from .claude/state/sprints/<sprint-id>/02-design.md.
      Files: <list>. Success criteria: <list>.
      Write partial notes to
      .claude/state/sprints/<sprint-id>/03-impl-WP-k.md.
      Do not touch any file outside the listed set."
   ```

   You can also dispatch from inside `claude agents`: type
   `developer Implement WP-k …` in the input. Either way, the session is
   named after its prompt; rename with `Ctrl+R` if you want
   `<sprint-id>/WP-k` exactly.
3. **Respect dependencies.** Packages with `Depends on: WP-x completion` must
   wait until the WP-x row reaches `Completed`. Dispatch them after that.
4. **Monitor.** Open `claude agents`. Sessions with state `Needs input` are
   waiting on you. Press `Space` to peek and answer; press `Enter` to attach
   if you need the full conversation.
5. **Merge worktrees.** Each session edited its own
   `.claude/worktrees/<id>/`. Move the changes into the main checkout before
   you delete the session — see *Sharp edges* below. Each session's
   `03-impl-WP-k.md` is part of its worktree's diff, so this step also brings
   the partial notes into the main checkout.
6. **Merge partial notes.** Once every worktree has been merged, the main
   loop (in your interactive `/sprint` session, or run by you manually)
   merges every `03-impl-WP-k.md` into the canonical
   `03-implementation-notes.md`. The merge rules are in
   `.claude/commands/sprint-execute.md`. Partial files stay on disk for
   traceability. If a WP session failed, do not auto-merge any worktree;
   bring the failure to the user at Gate 2.
7. **Delete the WP sessions.** In `claude agents`, select each completed WP
   row and press `Ctrl+X` twice within two seconds, or run
   `claude rm <id>` from the shell. The worktree is removed with the
   session.
8. **Proceed to Gate 2.** From here `/sprint` continues exactly as today.

## Runbook B — sprint as a background session

For long sprints you can run `/sprint` itself in the background and check on
it from `claude agents`.

```bash
claude --bg "/sprint <feature description>"
```

The session appears in `claude agents` with one-line summaries describing
what it is doing. At every approval gate it transitions to `Needs input`;
press `Space` to peek and answer (`yes` / `revise` / `waive` / `abort`).
Attach with `Enter` or `→` if you want to see the full PRD or design before
answering.

## Sharp edges

- **One `/sprint` at a time per checkout.** The sprint-number assignment in
  `/sprint` §1.2 picks `N = highest existing + 1`. Two concurrent `/sprint`
  invocations will pick the same `N` (in their respective worktrees) and
  collide on merge. Run one `/sprint` at a time; parallelism happens *inside*
  a sprint via Work Packages, not across sprints.
- **Worktrees are deleted with the session.** Pressing `Ctrl+X` to delete a
  WP session also removes `.claude/worktrees/<id>/`. Merge that worktree's
  changes into the main checkout first. Typical flow:
  - From the project root, `git worktree list` shows every worktree path
    Claude Code created.
  - Either commit in the worktree and merge its branch into your working
    branch, or copy the edited files into the main checkout, then delete the
    session.
- **Background sessions stop on machine sleep.** When the machine wakes,
  rows show as `Stopped`. Attach, peek, or reply to restart them; or run
  `claude respawn --all` to restart every stopped session at once.
- **Agent View is local.** Sessions live on the machine that started them;
  they do not roam between machines. Codex users do not see them — the Codex
  flow follows the same sprint contract sequentially.
- **`bypassPermissions` is one-time gated.** Starting a background session
  with `--permission-mode bypassPermissions` (or `auto`) is refused until you
  have accepted that mode once interactively, so a session you are not
  watching cannot act without prior approval.

## Disabling

Teams that do not want Agent View can turn it off:

- Per-machine: set `CLAUDE_CODE_DISABLE_AGENT_VIEW=1` in the environment.
- Per-org: set the managed setting `disableAgentView` to `true`. See the
  Claude Code permissions docs for managed-settings deployment.

With Agent View disabled, this template still works — every sprint runs
sequentially in one interactive session, and the `## Work Packages` section,
if produced, is consumed serially by a single Developer.
