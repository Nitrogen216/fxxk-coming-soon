# /loop-once — Run One Iteration

**Invocation**: `/loop-once [--focus metric_name] [--phase implement|execute|evaluate]`

---

## Purpose

Execute exactly one iteration of the reproduction loop with full control. Use this when you want to:
- Manually step through the loop and inspect results between iterations
- Focus attention on a specific failing metric
- Re-run only one phase (e.g., just evaluate without re-implementing)
- Debug a specific issue before resuming the full loop

---

## Execution

### Full iteration (default)

1. Read `LOOP_STATE.md` — get current iteration number and outstanding issues
2. **IMPLEMENT**: apply the top fix from `outstanding_issues`, or do full implementation if `iteration_count == 0`
3. **EXECUTE**: run the experiment, save to `logs/iter_${N}.log`
4. **EVALUATE**: compute metrics, compare to targets
5. **UPDATE STATE**: write results to `LOOP_STATE.md`
6. **REPORT**: print iteration summary

### Phase-specific (`--phase`)

- `--phase implement` — only run PHASE 1, then stop
- `--phase execute` — only run PHASE 2 (assumes implementation is done)
- `--phase evaluate` — only run PHASE 3 on existing results

### Metric-focused (`--focus`)

`/loop-once --focus f1_score`

Instructs the implement phase to prioritize fixes related to the named metric:
1. Pull all issues from `outstanding_issues` where `metric == f1_score`
2. Implement the highest-priority fix for that metric
3. Run a targeted evaluation (not full experiment if possible)
4. Report only that metric's result

---

## State Management

After each iteration:

```yaml
# LOOP_STATE.md update
iteration_count: N+1
latest_metrics:
  [updated with new results]
iteration_history:
  - iteration: N
    achieved: {metric: value, ...}
    issues_found: [list]
    fixes_applied: [list]
```

The iteration count is incremented **before** doing work, so crashes don't cause infinite retries on the same iteration.

---

## Output

```
── Iteration 4 ──────────────────────────────────────────────
 Fix applied: Switched to macro-weighted F1 in metrics.py:42

── Execution ────────────────────────────────────────────────
 Runtime: 12m 34s | GPU mem peak: 8.2 GB
 Log: logs/iter_4.log

── Results ──────────────────────────────────────────────────
 Metric      Before    After     Δ        Status
 accuracy    82.1%     83.9%     +1.8%    ● CLOSE (gap 3.1%)
 f1_score    70.1%     75.4%     +5.3%    ● CLOSE (gap 1.0%)
─────────────────────────────────────────────────────────────
 Progress: 0→1 metrics passing | 1 iteration remaining

Next hypothesis: accuracy gap 3.1% — check weight initialization (see RISKS_AND_NOTES.md)
Run /loop-once again or /reproduce to continue automatically
```
