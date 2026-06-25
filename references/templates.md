# Templates

Separate background threads only. Not same-thread subagents.

## Worker

```text
Ты separate background worker, not an ordinary same-thread subagent.
Use $1codex-bg-threads.

Outcome:
- Do: <one outcome>.
- Done when: <observable criteria>.

Scope:
- Project: <path/id>.
- Mode: light|nested|long|parallel|sequential.
- Model: default unless user requested an available override.
- Environment: local|worktree|cloud.
- Write scope: <allowed files/folders>.
- Do not touch: <forbidden/shared surfaces>.
- If scope must expand: WORKER_BLOCKED.

Rules:
- Keep diff reviewable.
- Reuse local patterns.
- Verify with: <commands/evidence>.
- Send WORKER_DONE to <orchestrator/source_thread_id>; if callbacks unavailable, end this thread with WORKER_DONE.
- After done/block/fail, stop.
```

Nested-only add:

```text
используй субагентов.
Run nested delegation only inside this background thread.
Preflight: if spawn_agent/wait_agent or equivalent is unavailable, WORKER_BLOCKED with nested_subagents: unavailable.
Use subagents for: <independent angles>.
Require SUBAGENT_RESULT; wait; reconcile; worker owns final diff.
```

Long-only add:

```text
Heartbeat before long phases and every <cadence>.
If waiting on nested subagents >60s: WORKER_HEARTBEAT phase=waiting-on-nested-subagents.
```

## Read-Only Scout

```text
Ты read-only background scout.
Investigate: <question>.
Do not edit, branch, commit, PR, or create durable artifacts.
Mode: light|nested. If nested, include the nested-only block.
Return findings with file refs, confidence, WORKER_DONE.
```

## Signals

```text
WORKER_STARTED
worker_thread_id: <id>
task: <short>
mode: light|nested|long|parallel|sequential
model: <default|explicit>
write_scope: <scope>
heartbeat_cadence: <cadence|none>
```

```text
WORKER_HEARTBEAT
worker_thread_id: <id>
status: running
phase: reading|editing|testing|blocked-on-approval|waiting|waiting-on-nested-subagents
write_scope: <scope>
progress: <short>
next_check_after: <duration>
```

```text
SUBAGENT_RESULT
role: <role>
status: clear|risk|blocked|failed
evidence: <file/command/result>
risk: <risk|none>
```

```text
WORKER_BLOCKED
worker_thread_id: <id>
status: blocked
reason: <blocker>
needs: <orchestrator/user action>
safe_to_continue: yes|no
```

```text
WORKER_DONE
worker_thread_id: <id>
status: success|blocked|failed
mode: light|nested|long|parallel|sequential
model: <default|explicit>
write_scope: <scope>
nested_subagents: used|unavailable|attempted|not-needed
nested_evidence: <spawn/wait/reconcile evidence or reason>
summary: <changed>
verification: <checks/not run>
remaining_risk: <risk|none>
next_action: accept|review_diff|send_followup|start_merge_owner|replace_worker
```

## Watchdog Automation

```text
Wake every <interval> while active workers remain.
On wake:
1. Read active worker threads.
2. Update ledger.
3. Verify nested evidence before accepting nested_subagents: used.
4. Report only new DONE/BLOCKED/FAILED/stale.
5. Ask stale workers for HEARTBEAT or DONE.
6. Stop when all workers are resolved and final status is reported.
```

## Merge Owner

```text
Ты merge-owner background thread in Local mode.
Use $1codex-bg-threads.

Outcome:
- Integrate accepted worker outputs into one coherent final state.

Inputs:
- Worker packets: <ids/summaries/diffs>.
- Allowed write scope: <merge scope>.
- Do not touch: <forbidden>.
- Mode: light unless independent conflict/regression/verification review justifies nested.

Rules:
- Inspect before applying.
- Reconcile shared surfaces once.
- Verify integrated result: <commands>.
- If verification fails, fix merge-caused issues only or WORKER_BLOCKED.
- Return WORKER_DONE to <orchestrator/source_thread_id>.
```
