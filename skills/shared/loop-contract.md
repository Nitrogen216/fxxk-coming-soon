# Loop Contract

All loop-aware skills must honor these invariants. They are non-negotiable.

## State File Invariants

- **Always read** `LOOP_STATE.md` before starting any loop operation
- **Always write** `LOOP_STATE.md` after completing any loop operation
- **Never skip** state updates, even if the iteration fails or crashes
- **Treat** `LOOP_STATE.md` as the single source of truth for loop progress

## Iteration Counting

- Increment `iteration_count` at the **START** of each iteration, before doing work
- This ensures max_iterations is respected even if the agent crashes mid-iteration
- Never decrement iteration_count, even if an iteration was unproductive

## Milestone Progression (enforced order)

Skills must progress through milestones in order — never skip:

```
M0 (sanity) → M1 (baseline) → M2 (method) → M3 (ablation) → M4 (review)
```

**Critical**: M2 (method) MUST NOT be attempted before M1 (baseline) passes. If M1 is pending, all implementation work focuses on baseline reproduction. This prevents wasting iterations on method improvements when the baseline itself is broken.

## Stopping Conditions (checked in this priority order)

1. `status == "success"` — M2 (and M3 if effort≥max) gates passed → STOP, report success
2. `iteration_count >= max_iterations` — budget exhausted → STOP, report timeout, suggest /extend-loop
3. `status == "blocked"` — unrecoverable error → STOP, report blocking issue
4. User interruption → STOP, update state with `status = "interrupted"`

No other conditions justify stopping the loop.

## Plateau Rules

When `plateau_detected == true`:
- Do NOT apply the same type of fix that was applied in the last plateau-stalled iteration
- Escalate to the plateau-type-specific strategy (see loop-contract for types A/B/C/D)
- Record `plateau_escalation_applied` in LOOP_STATE.md
- After 2 plateau escalations with no improvement → set `status = "blocked"` with a clear message

## Fix Strategy

Each iteration addresses exactly ONE issue from `outstanding_issues`:
- The item with the highest priority
- If tied: the metric with the largest gap
- Make **surgical, minimal changes** — do not refactor unrelated code
- Document the change immediately in `LOOP_STATE.md`

## Checkpoint Management

Before any iteration that modifies the model:
- Back up the best-performing checkpoint to `results/best_checkpoint/`
- Never overwrite this backup with a worse result

## Error Recovery

On execution failure:
1. Classify error type (dependency / logic / data / oom)
2. Attempt automatic fix for known patterns (up to 3 retries)
3. If still failing: add to `outstanding_issues`, continue loop
4. If error prevents any evaluation: set `status = "blocked"`, stop

## Cross-Model Review Protocol

When `reviewer != none`:
- Pass the **file paths** to the reviewer, never summaries
- Never pass the reviewer outputs from previous rounds (preserves independence)
- Treat reviewer feedback as advisory — incorporate with judgment, not blindly

## Artifact Communication

Skills communicate through files, not memory:
```
LOOP_STATE.md         → loop state and metrics
logs/iter_N.log       → raw execution output
results/iter_N/       → model outputs and checkpoints
outstanding_issues    → diagnosis for next iteration
```

Do not rely on in-memory state to pass information between phases within an iteration.
