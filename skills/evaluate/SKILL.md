# /evaluate — Check Reproduction Progress

**Invocation**: `/evaluate [--verbose] [--iteration N]`

---

## Purpose

Inspect the current state of paper reproduction without running any new experiments. Reads existing logs and results, computes metrics, and reports gap analysis against paper targets.

Use this after running `/loop-once` or when you want a snapshot of current progress.

---

## Execution

1. **Read `LOOP_STATE.md`** — get current iteration, status, latest metrics
2. **Read `DATA_AND_EVAL.md`** — load all target metrics from `## Paper Target Metrics`
3. **Check for fresh results** — if `results/iter_${N}/` exists but `LOOP_STATE.md` is not updated:
   - Run the evaluation script: `python eval.py --results results/iter_${N}/`
   - Parse metrics from the script output
   - Update `LOOP_STATE.md` with results
4. **Compute gap analysis** for each metric:
   ```
   gap = |achieved - target| / |target|
   status = "pass" if gap <= tolerance else "fail"
   delta_from_prev = achieved_N - achieved_{N-1}
   ```
5. **Print results table** (see Output Format)

---

## Output Format

### Standard (default)

```
Reproduction Status — Iteration 3/10 | Effort: balanced | Tolerance: 10%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Metric        Target    Achieved  Gap      Δ prev   Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 accuracy      84.7%     82.1%     3.1%     +1.8%    ● CLOSE
 f1_score      76.2%     70.1%     8.0%     +3.2%    ✗ FAIL
 precision     81.0%     80.5%     0.6%     +0.1%    ✓ PASS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Overall: 1/3 primary metrics within tolerance (10%)
 Best gap: precision 0.6% | Worst gap: f1_score 8.0%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Top Outstanding Issues:
  1. [HIGH]  f1_score 8.0% gap — suspected class weighting issue
  2. [MED]   accuracy 3.1% gap — possible weight init mismatch

Next: /loop-once to attempt fixes | /debug-gap --metric f1_score for deep analysis
```

### Verbose (`--verbose`)

Adds per-iteration trend table showing metric evolution across all iterations.

---

## Status Symbols

| Symbol | Meaning | Gap |
|--------|---------|-----|
| `✓ PASS` | Within tolerance | ≤ tolerance |
| `● CLOSE` | Nearly passing | ≤ 2× tolerance |
| `✗ FAIL` | Needs work | > 2× tolerance |
| `? UNKNOWN` | No results yet | — |
| `✗ CRASH` | Execution failed | — |
