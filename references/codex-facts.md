# Codex Facts

## Surface

- Official pattern: `Create a separate background thread in a worktree for this project to update the tests.`
- "Codex loops" is not a stable official primitive. Map it to threads, worktrees, automations, goals, subagents, or SDK/app-server loops.
- Multiple threads can run at once; do not let two threads modify the same files.
- Thread modes: Local, Worktree, Cloud. Local/Worktree run on the user's machine.

## Worktree

- Use Worktree for isolated parallel writes or background work while Local stays usable.
- Use Local for read-only scouts, sequential workers, or intentional current-checkout edits.
- Worktrees require Git, start detached by default, and live under `$CODEX_HOME/worktrees`.
- One branch cannot be checked out in two worktrees. Do not merge by checking out a worker branch locally while it is active there.
- Ignored files needed by managed worktrees must be listed in `.worktreeinclude`; tracked files do not belong there.

## Automation

Thread automation only when next wake depends on current thread context. Prompt: what to poll, relevant states, stale/blocked/done classification, stop/archive/ask-user rule. Test manually first. Full-access unattended automation is high risk.

## Subagents

- Codex spawns subagents only on explicit request.
- Same-thread subagent != background worker thread.
- Nested mode belongs inside a background worker thread.
- `agents.max_depth` default: `1` (root depth `0`, direct child allowed, deeper nesting blocked). Raise only for deliberate recursive delegation.
- Raising depth risks fan-out, token/latency cost, resource load, and weaker control.
- `agents.max_threads` default: `6`.
- `spawn_agents_on_csv` timeout default: 1800s/worker when unset.

## Custom Agents

Use only for persistent nondefault worker config: role/model/reasoning/sandbox/MCP/skills. Personal: `~/.codex/agents/`; project: `.codex/agents/`. Required: `name`, `description`, `developer_instructions`. Parent runtime overrides still apply.

Sources: [prompting](https://developers.openai.com/codex/prompting), [app features](https://developers.openai.com/codex/app/features), [automations](https://developers.openai.com/codex/app/automations), [worktrees](https://developers.openai.com/codex/app/worktrees), [subagents](https://developers.openai.com/codex/subagents).
