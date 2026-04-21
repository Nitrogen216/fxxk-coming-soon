# /reproduce-status — Quick Status Check

**Invocation**: `/reproduce-status`

---

## Purpose

Print a compact, human-readable summary of the current reproduction state. No analysis, no execution — just read `LOOP_STATE.md` and report.

Use this to get a quick snapshot when resuming a session.

---

## Execution

1. Read `LOOP_STATE.md` (if missing: report "Not started")
2. Print compact status

---

## Output

```
[fxxk-coming-soon] Paper Reproduction Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Paper:      "Attention Is All You Need" (example)
 Status:     RUNNING
 Iteration:  4 / 10
 Effort:     balanced (tolerance: 10%)
 Started:    2024-01-15  Last update: 2024-01-15

 Metrics:
   accuracy    84.7% → 83.9%  gap 1.0%  ● CLOSE
   f1_score    76.2% → 75.4%  gap 1.0%  ● CLOSE
   precision   81.0% → 80.5%  gap 0.6%  ✓ PASS

 Top issue: accuracy gap — weight initialization (Xavier vs Kaiming)

 Resume with: /reproduce
 One iteration: /loop-once
 Deep debug: /debug-gap --metric accuracy
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `status == "success"`:
```
[fxxk-coming-soon] ✓ REPRODUCTION COMPLETE
 All 3/3 metrics within 10% tolerance after 7 iterations
 Report: REPRODUCTION_REPORT.md
```
