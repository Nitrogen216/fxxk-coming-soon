---
name: check-plateau
description: Detect whether the reproduction loop has stalled on a metric — no meaningful improvement over the last N iterations. Reports plateau status, identifies likely cause, and recommends an escalation strategy to break the plateau.
when_to_use: Use when a metric hasn't improved for several iterations, when /evaluate shows a ⚠ PLATEAU flag, or before deciding whether to extend the loop or change strategy fundamentally.
argument-hint: "[--metric name] [--window N] [--threshold 0.01]"
allowed-tools:
  - Read
  - Bash
---

# Plateau Detection Analysis

**Arguments**: $ARGUMENTS

## Iteration History
!`python3 -c "
import sys, re
try:
    content = open('LOOP_STATE.md').read()
    idx = content.find('iteration_history')
    if idx>=0: print(content[idx:idx+2000])
    else: print('No iteration history found')
except Exception as e: print(f'Error: {e}')
" 2>/dev/null`

## Instructions

Parse `$ARGUMENTS`:
- `--metric <name>` → analyze this specific metric (default: worst-performing)
- `--window N` → look at last N iterations (default: 3)
- `--threshold 0.01` → improvement below this % is considered "no progress" (default: 1%)

---

### Step 1: Extract Metric History

From `LOOP_STATE.md → iteration_history`, build a time series for the target metric:

```
Iter  achieved   delta_abs   delta_rel
1     0.701      —           —
2     0.724      +0.023      +3.3%
3     0.737      +0.013      +1.8%
4     0.741      +0.004      +0.5%  ← sub-threshold
5     0.743      +0.002      +0.3%  ← sub-threshold
6     0.744      +0.001      +0.1%  ← sub-threshold  PLATEAU (3 iters)
```

### Step 2: Classify Plateau Type

Determine WHY progress has stalled:

**Type A: Asymptotic** — metric is improving but slowing down, approaching a local maximum
- Pattern: monotonically increasing but Δ shrinking each iteration
- Likely cause: approaching local optimum, learning rate too high for fine-tuning
- Fix: reduce learning rate by 10×, add learning rate decay, or try different optimizer

**Type B: Oscillating** — metric goes up and down without net progress
- Pattern: alternating increases and decreases around a mean
- Likely cause: learning rate too high, gradient clipping too loose, label noise
- Fix: reduce LR, add gradient clipping, check data quality

**Type C: Early Ceiling** — metric hit a hard limit well below target
- Pattern: rapid early improvement then hard stop (e.g., 70% → 71% → 71% → 71%)
- Likely cause: architectural bottleneck, wrong metric implementation, data issue
- Fix: run `/debug-gap --depth deep`, check metric formula, check model capacity

**Type D: Random Walk** — metric bounces with no trend
- Pattern: large variance, no monotonic direction
- Likely cause: non-deterministic training, seed not fixed, data shuffling issue
- Fix: fix random seeds, reduce stochasticity, check data pipeline

### Step 3: Cross-Reference with Assumption Ladder

Read `RISKS_AND_NOTES.md → assumption_ladder`. Check if any unvalidated assumption might explain the plateau. Flag the most relevant assumption.

### Step 4: Generate Escalation Strategy

Based on plateau type, recommend specific actions:

```
Plateau Analysis: f1_score | Window: 3 iterations | Threshold: 1%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Metric trend (last 4 iterations):
  Iter 3: 70.1% → Iter 4: 75.4% → Iter 5: 75.8% → Iter 6: 75.9%
  Δ (last 3):  +5.3%  →  +0.4%  →  +0.1%
  Verdict: PLATEAU TYPE A (asymptotic convergence)

Target gap remaining: 0.4% (target 76.2%, achieved 75.9%)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Diagnosis: Approaching local optimum under current LR schedule.
The model is very close to target. Current approach is converging.

Escalation options (ranked):
  1. [RECOMMENDED] Reduce LR by 10×: may unlock the last 0.4% without
     architecture changes. Low risk.
     → Edit: configs/paper_config.yaml: lr: 3e-5 (was 3e-4)

  2. Run 1 more iteration at current settings: given 0.4% gap and
     0.1% last delta, might pass on its own.
     → Run: /loop-once

  3. Check assumption #4 in RISKS_AND_NOTES.md: "F1 threshold for
     positive class unspecified" — still unvalidated.
     → Run: /debug-gap --metric f1_score --depth deep

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Auto-adding to LOOP_STATE.md: plateau_detected: true, plateau_type: A
```

Update `LOOP_STATE.md` with plateau status and recommended fix.
