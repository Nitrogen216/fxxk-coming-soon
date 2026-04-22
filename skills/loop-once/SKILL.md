---
name: loop-once
description: Run exactly one iteration of the reproduction loop with full manual control. Reads top outstanding issue, applies fix, executes experiment, evaluates results, and updates LOOP_STATE.md.
when_to_use: Use for step-by-step debugging, testing a specific fix before committing to a full loop, or when you want to review results between iterations.
argument-hint: "[--focus metric_name] [--phase implement|execute|evaluate]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
---

# Run One Reproduction Iteration

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null || echo "No LOOP_STATE.md — run /setup-env first"`

## Outstanding Issues
!`python3 -c "
import sys
try:
    content = open('LOOP_STATE.md').read()
    idx = content.find('outstanding_issues')
    if idx >= 0:
        print(content[idx:idx+800])
    else:
        print('No outstanding issues found')
except: print('Could not read LOOP_STATE.md')
" 2>/dev/null`

## Instructions

Parse `$ARGUMENTS`:
- `--focus <metric>` → only address issues affecting that metric this iteration
- `--phase implement` → only run PHASE 1, then stop
- `--phase execute` → only run PHASE 2 (assumes implementation is done)
- `--phase evaluate` → only run PHASE 3 on existing results

**Full iteration** (default, no --phase flag):

1. **Check stopping conditions** (per loop-contract.md):
   - If `status.state == "success"` → announce and stop
   - If `status.iteration_count >= loop_control.max_iterations` → announce timeout, suggest `/extend-loop`
   - If `status.total_gpu_hours_consumed >= loop_control.total_gpu_hour_budget` → announce budget exhausted, stop

2. **Increment** `status.iteration_count` in `LOOP_STATE.md` BEFORE doing any work

3. **IMPLEMENT**: apply top fix from `outstanding_issues`
   - If `status.iteration_count == 1` and no code exists: full implementation per `IMPLEMENTATION_PLAN.md`
   - If `--focus <metric>`: only fix issues tied to that metric
   - Document each change immediately in `fixes_applied`

4. **EXECUTE**: run experiment
   - `python main.py --config configs/paper_config.yaml 2>&1 | tee logs/iter_${N}.log`
   - Classify errors; retry up to 3 times for recoverable types

5. **EVALUATE**: compute metrics vs targets
   - Parse log for metric values; compare to `DATA_AND_EVAL.md` Paper Target Metrics
   - Update `latest_metrics` in `LOOP_STATE.md`

6. **UPDATE STATE**: write full iteration record to `LOOP_STATE.md`

7. **PLATEAU CHECK**: if this metric improved < 1% for 3+ consecutive iterations → flag `⚠ PLATEAU`

## Output

```
── Iteration 4 ─────────────────────────────────────────────
 Phase:  implement → execute → evaluate
 Fix:    Switched to macro-weighted F1 (metrics.py:42)

── Execution ───────────────────────────────────────────────
 Runtime: 12m 34s  |  Log: logs/iter_4.log

── Results ─────────────────────────────────────────────────
 Metric      Before    After     Δ        Status
 accuracy    82.1%     83.9%     +1.8%    ● CLOSE
 f1_score    70.1%     75.4%     +5.3%    ● CLOSE
────────────────────────────────────────────────────────────
 Milestone: M2 in progress (0/2 metrics passing)

Next fix hypothesis: accuracy 3.1% gap — Xavier vs Kaiming init
Run /loop-once again or /reproduce to continue automatically
```
