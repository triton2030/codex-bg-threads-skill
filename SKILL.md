---
name: 1codex-bg-threads
description: >
  Codex background-thread orchestration, not ordinary same-thread subagents.
  Use for explicit background workers, many writing agents, callback/stop
  signals, heartbeat/watchdog loops, worker threads that must use their own
  subagents, or autonomous worktree fanout followed by merge-owner integration.
---

# 1codex-bg-threads

## Contract

Use this skill only for explicit background-thread orchestration. A background
worker is a separate Codex thread; a same-thread subagent is not this skill's
target and is not a reliable recursive root.

Shape: orchestrator -> background worker thread -> optional nested subagents.
The orchestrator owns mission, ledger, worker ids, stale decisions, acceptance,
and final merge. Each worker owns one outcome, one write scope, and its own
inner delegation.

Only create user-visible threads when the user asked for background threads,
workers, new threads, long-running loops, or this skill. Otherwise use the
normal same-thread/multi-agent route.

## Model Target

Target prompts to `GPT-5.5`: outcome first, then done criteria, write scope,
allowed side effects, evidence, output shape, and stop rule. Do not pin a model
unless the user asked or the plan names model routing as a tradeoff. Exact model
overrides are live tool-surface facts, not durable skill text.

## Routing

Default to the smallest mode that preserves safety:

- **Light**: one background worker, no nested subagents.
- **Nested**: worker must call its own subagents. Prompt must literally say
  `используй субагентов`.
- **Long**: worker may outlive the active turn. Add heartbeat + watchdog.
- **Parallel writer**: independent write scopes only.
- **Sequential writer**: shared surface, dependency, or uncertain ownership.
- **Worktree fanout merge**: independent worktree writers, then one Local
  merge-owner integrates accepted outputs.

Worktrees are isolation, not ceremony. Merge-owner is reconciliation, not
ceremony. Use either only when the risk they remove is real.

## References

- [`references/templates.md`](references/templates.md): worker, scout,
  heartbeat, done, watchdog, and merge-owner prompt blocks.
- [`references/codex-facts.md`](references/codex-facts.md): current Codex
  facts for Local/Worktree/Cloud, automations, subagents, custom agents, and
  source links.
- [`references/worktree-merge.md`](references/worktree-merge.md): autonomous
  strategy for N worktree writers -> one Local merge-owner.

Load only the branch-relevant reference.

## Protocol

1. State mission: outcome, done-state, forbidden surfaces, checks, max useful
   worker count, parallelism safety.
2. Partition by write scope. If scopes overlap, run sequentially, use read-only
   scouts, or give the edit to one merge-owner.
3. Choose environment: Local for read-only/sequential/low-risk current checkout
   work; Worktree for isolated parallel writes; Cloud only when that surface is
   intentionally selected.
4. Launch workers with a prompt based on `templates.md`.
5. Keep an orchestrator ledger:

   ```text
   worker_thread_id | task | mode | environment | write_scope | heartbeat_at | status | next_poll
   ```

6. If the created id was not in the initial prompt, send it to the worker as a
   follow-up. `unknown-current-thread` is only a worker fallback; the
   orchestrator must map it.
7. Wait with polling. Heartbeat proves liveness, not correctness.
8. Accept a worker only after reading enough status, diff, checks, and evidence.
9. Merge only after every worker is `success|blocked|failed`; include, skip, or
   retry each worker explicitly.

## Worker Prompt Minimum

Every worker prompt includes:

- one outcome and observable done criteria;
- mode: `light|nested|long|parallel|sequential`;
- environment: `local|worktree|cloud`;
- model: runtime default unless user requested an available override;
- allowed write scope and explicit do-not-touch scope;
- verification command or evidence expectation;
- orchestrator/source thread id for callbacks, or fallback final block;
- `WORKER_DONE` shape and stop rule: after done/block/fail, wait.

Nested workers also include:

- `используй субагентов`;
- preflight: if `spawn_agent`/`wait_agent` or equivalent is unavailable, return
  `WORKER_BLOCKED` with `nested_subagents: unavailable`;
- division of inner work, `SUBAGENT_RESULT` expectation, wait, reconciliation,
  and final `nested_evidence`.

Long workers also include heartbeat cadence. For hour-scale work, prefer a
thread automation/watchdog that reads worker threads, reports only changed
done/blocked/failed/stale states, and stops when all workers resolve.

## Signals

Use the exact block shapes from `templates.md`. The required final fields are:

```text
WORKER_DONE
worker_thread_id: ...
status: success|blocked|failed
mode: light|nested|long|parallel|sequential
write_scope: ...
nested_subagents: used|unavailable|attempted|not-needed
nested_evidence: ...
summary: ...
verification: ...
remaining_risk: ...
next_action: accept|review_diff|send_followup|start_merge_owner|replace_worker
```

For `nested_subagents: used`, the orchestrator must verify transcript evidence:
subagents were launched, waited for, returned results, and were reconciled.

## Failure Checks

- **False parallelism**: overlapping files, generated artifacts, lockfiles,
  migrations, shared config, or one owner surface.
- **Fake nesting**: worker claims nested subagents without spawn/wait/reconcile
  evidence.
- **Callback illusion**: no source thread id and no fallback final block.
- **Heartbeat confusion**: liveness treated as quality proof.
- **Premature merge**: summary accepted without diff/check/evidence.
- **Orphaned worker**: no ledger, next poll, heartbeat, or automation.
- **Worktree ritual**: isolation used where sequential/local work is safer.
- **Merge ritual**: merge-owner created despite independent outputs.

## Completion

Done means all launched workers resolved; every worker had a mode and write
scope; nested workers contained `используй субагентов`; nested evidence was
verified before acceptance; parallel writers had independent scopes or were
converted to sequential/merge-owner flow; callbacks were received or polled;
merge-owner was used only when outputs needed reconciliation; final response
separates completed, blocked, failed, unchecked, and next action.
