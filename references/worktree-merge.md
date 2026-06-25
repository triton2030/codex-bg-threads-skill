# Worktree Merge

Independent writing slices -> one coherent Local checkout.

```text
user task
-> orchestrator in original Local/main
-> N Worktree workers with disjoint scopes
-> wait for every DONE/BLOCKED/FAILED
-> merge-owner thread in original Local
-> inspect diffs, integrate, verify
```

## Use / Skip

Use: 2-6 independent write scopes, each verifiable, final result must be one diff.

Skip: same files; shared design decision; migration/schema/lockfile/config as main output; dirty base without exclusions; one verification loop must steer all work; user asked for same-thread subagents.

## Orchestrator

1. Record original path, branch, `HEAD`, dirty state, final checkout.
2. Partition by write scope, not speed. Shared/generated/config surfaces belong to merge-owner unless one worker owns them entirely.
3. Launch each worker as background Worktree thread. Prompt: `Use $1codex-bg-threads`.
4. Require merge packet. No worker merges into Local/main.
5. Ledger: worker id, worktree path, base rev, scope, heartbeat, status, changed files, merge readiness.
6. For long waits, use thread automation to poll and report only state changes.
7. Launch merge-owner only after every worker is `success|blocked|failed`; explicitly include/skip/retry each worker.

## Worker Merge Packet

```text
WORKER_DONE
worker_thread_id: <id>
status: success|blocked|failed
mode: parallel
environment: worktree
worktree_path: <path>
base_rev: <original HEAD/branch>
head_rev: <commit or worktree HEAD>
write_scope: <scope>
changed_files: <list>
merge_method: patch|cherry-pick|manual-rebuild|handoff
summary: <changed>
verification: <checks>
remaining_risk: <risk|none>
next_action: include_in_merge|skip|retry|needs_review
```

Workers leave reviewable diff/commit and stop.

## Merge-Owner

New background thread in original Local checkout.

1. `git status -sb`; confirm base drift.
2. For each accepted worker: inspect packet + diff; classify `apply-clean|apply-with-edits|manual-rebuild|skip|blocked`.
3. Apply one worker at a time; semantic integration over blind patch stacking.
4. Patch fails -> rebuild from diff/summary; do not force.
5. Semantic conflict -> final user outcome + repo ownership rules beat worker-local choices.
6. Reconcile shared/generated/config surfaces once.
7. Run integrated verification plus focused slice checks when cheap.
8. Check conflict markers / `git diff --check` when available.
9. Return `WORKER_DONE` with included/skipped workers, changed files, verification, risks, cleanup candidates.

Do not delete/archive worker evidence before acceptance.

## Merge-Owner Prompt

```text
Ты merge-owner background thread in Local mode for <original_project_path>.
Use $1codex-bg-threads.

Merge accepted worker worktree outputs into one coherent final diff.
Inspect before applying; do not blindly stack patches.

Inputs:
- base: <branch/rev>
- accepted: <worker ids>
- skipped/blocked: <ids + reason>
- packets: <compact packets>
- allowed scope: <files/folders>
- forbidden: <surfaces>

Protocol:
1. Check status/base drift.
2. Inspect each diff.
3. Apply/rebuild one worker at a time.
4. Reconcile shared surfaces once.
5. Run: <verification>.
6. WORKER_DONE with included/skipped, changed files, verification, risk, cleanup.
Stop.
```

Done: all workers resolved; included diffs inspected; merge-owner ran in original Local; final diff coherent; verification reported; skipped/blocked separated; cleanup listed, not executed.

Sources: [worktrees](https://developers.openai.com/codex/app/worktrees), [automations](https://developers.openai.com/codex/app/automations), [review](https://developers.openai.com/codex/app/review), [subagents](https://developers.openai.com/codex/subagents), [prompting](https://developers.openai.com/codex/prompting).
