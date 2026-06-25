# Codex Background Threads Skill

Unofficial Codex skill for orchestrating explicit background worker threads.

It is for cases where the main Codex thread should stay the orchestrator while
separate background threads do bounded work, send `WORKER_DONE` / heartbeat
signals, optionally use their own nested subagents, and sometimes merge isolated
worktree outputs back into one result.

This is not for ordinary same-thread subagents.

## Use When

- You want explicit background worker threads.
- Several writing agents need independent scopes.
- Long-running workers need heartbeat/watchdog polling.
- A worker thread must use its own subagents.
- Multiple worktree workers should be integrated by one merge-owner.

## Install

Clone this repository into your Codex skills folder:

```sh
mkdir -p ~/.codex/skills
git clone https://github.com/triton2030/codex-bg-threads-skill.git ~/.codex/skills/1codex-bg-threads
```

Then ask Codex:

```text
Use $1codex-bg-threads for explicit background workers with heartbeat and WORKER_DONE stop signals.
```

## Files

- `SKILL.md`: compact router and orchestration contract.
- `references/templates.md`: worker, scout, heartbeat, done, watchdog, and merge-owner prompt blocks.
- `references/codex-facts.md`: current Codex facts and source links.
- `references/worktree-merge.md`: worktree fanout -> merge-owner strategy.
- `agents/openai.yaml`: optional Codex skill metadata.

## Operating Principle

Use the smallest safe mode. Worktrees and merge-owner threads are safety tools,
not rituals. Heartbeat proves liveness, not correctness. Accept worker output
only after checking status, diff, verification, and nested-subagent evidence when
nested mode was used.

## License

MIT.
