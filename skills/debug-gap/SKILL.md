---
name: debug-gap
description: Systematically diagnose why a specific metric is not matching the paper's reported value. Audits data, architecture, hyperparameters, and evaluation code. Outputs ranked hypotheses with file locations and concrete fixes.
when_to_use: Use when a metric has a large or stubborn gap, when the loop has been stuck on the same metric for 3+ iterations, or after /check-plateau identifies stagnation.
argument-hint: "[--metric name] [--depth shallow|deep]"
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Debug Metric Gap

**Arguments**: $ARGUMENTS

## Current Metrics
!`python3 -c "
import sys
try:
    content = open('LOOP_STATE.md').read()
    idx = content.find('latest_metrics')
    if idx >= 0: print(content[idx:idx+600])
    else: print('No metrics yet')
except: print('Could not read LOOP_STATE.md')
" 2>/dev/null`

## Instructions

Parse `$ARGUMENTS`:
- `--metric <name>` → focus on this metric (default: worst-performing from `LOOP_STATE.md`)
- `--depth shallow` → read-only investigation, no code changes (default)
- `--depth deep` → also implement diagnostic scripts and run them

**If no `--metric` specified**: use the metric with the largest gap from `LOOP_STATE.md`.

---

### Shallow Analysis (always run)

Systematically check all four layers:

**Layer 1: Data**
- Is train/test split correct per `DATA_AND_EVAL.md`?
- Is normalization applied consistently to train and test?
- Is the evaluation dataset the same one the paper reports on?
- Any data leakage? Label errors?

**Layer 2: Architecture**
- Compare implemented model against `PROJECT_STRUCTURE.md` module specs
- Check for missing components, wrong layer order, incorrect activations
- Verify parameter count roughly matches paper (if reported)

**Layer 3: Hyperparameters**
- Extract paper's exact values from `DATA_AND_EVAL.md → Experimental Protocols`
- Compare against `configs/paper_config.yaml` field by field
- Flag any discrepancy, even small ones (0.1 vs 0.01 dropout matters)

**Layer 4: Evaluation code**
- Is the metric formula correct? (macro vs micro F1, top-1 vs top-5 accuracy)
- Is the test set exactly as specified?
- Are pre/post-processing steps identical for train and evaluation?

**Cross-reference**: Check `RISKS_AND_NOTES.md → Technical Risk Assessment` and `assumption_ladder` for known issues with this metric.

---

### Deep Analysis (with `--depth deep`)

Additionally:
1. Write `debug/check_metric.py` — compute metric on synthetic known-answer data
2. Write `debug/check_data.py` — validate data statistics vs paper claims
3. Write `debug/check_model.py` — print model summary, layer names, param count
4. Run all three scripts and report findings

---

## Output Format

```
Debugging: f1_score | Gap: 8.0% | Target: 76.2% | Achieved: 70.1%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Data:            ✓ Split correct | ✓ Normalization consistent
                  ⚠ Class imbalance: class 3 has 3× fewer samples

 Architecture:    ✓ Layer count correct
                  ? Attention heads: paper=8, code=12 → check Table 2

 Hyperparameters: ✓ LR: 3e-4 | ✗ Dropout: 0.3 vs paper's 0.1

 Evaluation:      ✗ Metric averaging: code uses micro-F1, paper uses macro-F1

 RISKS_AND_NOTES: ⚠ Assumption #3 flagged: "F1 averaging unspecified"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ranked Hypotheses:

  #1 [85%] Evaluation — wrong F1 averaging (micro → macro)
     File: src/eval/metrics.py:compute_f1()
     Fix:  Change average='micro' to average='macro'
     Expected gain: ~6-8%

  #2 [10%] Hyperparameter — dropout 0.3 vs paper's 0.1
     File: configs/paper_config.yaml:model.dropout
     Fix:  Set dropout: 0.1

  #3 [5%]  Architecture — attention heads 12 vs paper's 8
     File: configs/paper_config.yaml:model.num_heads
     Fix:  Set num_heads: 8

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Auto-adding to LOOP_STATE.md outstanding_issues: [hypothesis #1 as priority HIGH]
Run /loop-once --focus f1_score to apply fix #1
```

After analysis, automatically add the top hypothesis to `LOOP_STATE.md → outstanding_issues` as `priority: high`.
