# /debug-gap — Analyze Metric Gaps

**Invocation**: `/debug-gap [--metric name] [--depth shallow|deep]`

---

## Purpose

Systematically diagnose why a specific metric is not matching the paper's reported value. Generates ranked hypotheses, optionally implements diagnostic tests, and proposes concrete fixes with file locations.

Use this when the loop is stuck — multiple iterations haven't improved a specific metric.

---

## Execution

### Identify Target Metric

If `--metric` not specified: use the metric with the largest gap from `LOOP_STATE.md`.

```
Target metric: f1_score | Gap: 8.0% | Iterations stuck: 2
```

---

### Shallow Analysis (default)

Read-only investigation — no code changes:

1. **Data layer check**
   - Is the train/test split correct per `DATA_AND_EVAL.md`?
   - Is data normalization applied consistently to train and test?
   - Are there any data leakage vectors?

2. **Model architecture check**
   - Compare implemented architecture against `PROJECT_STRUCTURE.md` module specs
   - Check for missing components, wrong layer order, incorrect activations
   - Verify output dimensions match paper's reported model size

3. **Hyperparameter audit**
   - Extract paper's exact hyperparameters from `DATA_AND_EVAL.md → Experimental Protocols`
   - Compare against `configs/paper_config.yaml`
   - Flag any discrepancies (even small ones like 0.1 vs 0.01 dropout)

4. **Evaluation code audit**
   - Is the metric formula correct? (e.g., macro vs micro F1, top-1 vs top-5 accuracy)
   - Is the evaluation set exactly as specified in the paper?
   - Are pre/post-processing steps identical to training?

5. **Cross-reference with `RISKS_AND_NOTES.md`**
   - Check `## Technical Risk Assessment` for known issues with this metric type
   - Check `## Common Implementation Pitfalls`

---

### Deep Analysis (`--depth deep`)

Implements diagnostic code and runs targeted experiments:

1. Everything from shallow analysis, plus:
2. **Implement diagnostic scripts** in `debug/`:
   - `debug/check_data.py` — validate data statistics match paper claims
   - `debug/check_metric.py` — compute metric on a synthetic known-answer dataset
   - `debug/check_model.py` — print model summary, compare parameter counts
3. **Run diagnostics** and report findings
4. **Implement and test** the highest-probability fix from the hypothesis list

---

## Output Format

```
Debugging: f1_score | Gap: 8.0% | Target: 76.2% | Achieved: 70.1%
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Data Layer:        ✓ Split correct (80/10/10 as per paper)
                   ✓ Normalization consistent
                   ⚠ Class imbalance: class 3 has 3x fewer samples

Architecture:      ✓ Layer count matches (6 transformer blocks)
                   ✓ Hidden dim correct (768)
                   ? Attention heads: paper says 8, code has 12

Hyperparameters:   ✓ LR: 3e-4 (matches)
                   ✗ Dropout: 0.3 in code vs 0.1 in paper (Table 2)
                   ✓ Batch size: 32 (matches)

Evaluation:        ✗ Metric: code uses micro-F1, paper uses macro-F1
                              (this is the most likely cause of 8% gap)
                   ✓ Test split: correct

RISKS_AND_NOTES:   ⚠ Risk #3 flagged: "F1 averaging strategy not specified"
                      → Default assumption was micro-F1, may be wrong

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ranked Hypotheses:

  #1 [85%] Evaluation — wrong F1 averaging (micro vs macro)
     File: src/eval/metrics.py:compute_f1()
     Fix:  Change average='micro' to average='macro'
     Expected gain: ~6-8%

  #2 [10%] Hyperparameter — dropout 0.3 vs paper's 0.1
     File: configs/paper_config.yaml:model.dropout
     Fix:  Set dropout: 0.1
     Expected gain: ~1-2%

  #3 [5%]  Architecture — attention heads 12 vs paper's 8
     File: configs/paper_config.yaml:model.num_heads
     Fix:  Set num_heads: 8
     Expected gain: ~1%

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Recommendation: Apply fix #1 immediately. Run /loop-once --focus f1_score
```

---

## Integration with Loop

After `/debug-gap`, the top hypothesis is automatically added to `LOOP_STATE.md → outstanding_issues` as the highest-priority item. The next `/loop-once` or `/reproduce` call will implement it first.
