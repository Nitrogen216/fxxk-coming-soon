---
name: evaluate
description: Check current reproduction progress against paper targets without running new experiments. Reads existing logs and LOOP_STATE.md, computes metric gaps, shows iteration trend, and suggests next action.
when_to_use: Use after running an experiment, at the start of a session, or when you want a status update without re-running anything.
argument-hint: "[--verbose] [--iteration N]"
allowed-tools:
  - Bash
  - Read
---

# Evaluate Reproduction Progress

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null || echo "No LOOP_STATE.md found — run /reproduce first"`

## Latest Log Tail
!`tail -50 logs/iter_$(cat LOOP_STATE.md 2>/dev/null | grep 'iteration_count:' | awk '{print $2}').log 2>/dev/null || echo "No logs found yet"`

## Instructions

Parse `$ARGUMENTS`:
- `--verbose` → also show per-iteration trend table
- `--iteration N` → show results from a specific iteration instead of latest

Steps:
1. Read `LOOP_STATE.md` → get `iteration_count`, `status`, `latest_metrics`, `tolerance`
2. Read `DATA_AND_EVAL.md` → load `## Paper Target Metrics` YAML block
3. If `results/iter_${N}/eval_summary.json` exists but `LOOP_STATE.md` is stale: re-parse metrics from log
4. For each metric: compute `gap = |achieved - target| / |target|`, status = pass/fail
5. For `--verbose`: build trend table from `iteration_history` in `LOOP_STATE.md`

## Output Format

```
Reproduction Status — Iteration 3/10 | Effort: balanced | Tolerance: 10%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Milestone: M1 (baseline) ✓ passed | M2 (method) in progress
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Metric        Target    Achieved  Gap      Δ prev   Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 accuracy      84.7%     82.1%     3.1%     +1.8%    ● CLOSE
 f1_score      76.2%     70.1%     8.0%     +3.2%    ✗ FAIL
 precision     81.0%     80.5%     0.6%     +0.1%    ✓ PASS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Overall: 1/3 primary metrics within tolerance (10%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Top Issues:
  1. [HIGH]  f1_score 8.0% gap — suspected class weighting issue
  2. [MED]   accuracy 3.1% gap — possible weight init mismatch

Plateau check: f1_score improved 0.3% last 2 iters — /check-plateau to analyze

Next: /loop-once  |  /debug-gap --metric f1_score  |  /reproduce (resume full loop)
```

Status symbols:
- `✓ PASS` — within tolerance
- `● CLOSE` — within 2× tolerance
- `✗ FAIL` — outside 2× tolerance
- `⚠ PLATEAU` — < 1% improvement last 3 iterations
