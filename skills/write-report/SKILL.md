# /write-report — Generate Reproduction Report

**Invocation**: `/write-report [--type success|progress|failed] [--compare-table]`

---

## Purpose

Formalize the current reproduction state into a structured Markdown report. Auto-detects report type from `LOOP_STATE.md` if `--type` is not specified.

- `success` → writes `REPRODUCTION_REPORT.md` — the full, citable reproduction result
- `progress` → writes `PROGRESS_REPORT.md` — best results so far, what remains
- `failed` → writes `FAILURE_REPORT.md` — what was tried, why it didn't work, recommendations

The loop agent calls this automatically on termination, but you can invoke it manually at any point to snapshot current progress.

---

## Report Types

### REPRODUCTION_REPORT.md (success)

Written when all primary metrics pass within tolerance.

```markdown
# Reproduction Report

**Paper**: [title from PROJECT_STRUCTURE.md]
**Original Authors**: [from paper]
**Reproduced by**: Claude Code (fxxk-coming-soon framework)
**Date**: 2024-01-15
**Status**: SUCCESS ✓

---

## Reproduction Summary

| Metric | Paper Value | Reproduced | Gap | Status |
|--------|-------------|------------|-----|--------|
| accuracy | 84.7% | 85.1% | 0.5% | ✓ PASS |
| f1_score | 76.2% | 75.8% | 0.5% | ✓ PASS |

**Effort**: balanced (10% tolerance, 10 max iterations)
**Iterations needed**: 6
**Total runtime**: ~4 hours

---

## Key Implementation Details

[Fill: any important deviations from paper, clarifications needed, assumptions made]

---

## Iteration History

| Iter | accuracy | f1_score | Fix Applied |
|------|----------|----------|-------------|
| 1 | 75.1% | 63.2% | Initial implementation |
| 2 | 80.3% | 68.4% | Fixed per-channel normalization |
| 3 | 82.1% | 70.1% | Reduced LR + warmup |
| 4 | 83.9% | 75.4% | Switched to macro-F1 |
| 5 | 84.5% | 75.6% | Fixed weight initialization |
| 6 | 85.1% | 75.8% | Tuned dropout 0.3→0.1 |

---

## Environment

- Python 3.10.12 / PyTorch 2.1.0+cu118
- Hardware: NVIDIA A100 80GB
- Key dependencies: [from requirements.txt]

---

## Reproducibility

To reproduce:
```bash
git clone <this-repo>
cd <project>
/setup-env
python main.py --config configs/paper_config.yaml --seed 42
```

Random seeds used: Python=42, NumPy=42, PyTorch=42, CUDA=42
```

### PROGRESS_REPORT.md (progress / timeout)

Written when max_iterations reached without full success.

```markdown
# Progress Report

**Status**: TIMEOUT (max iterations reached)
**Best iteration**: 6 / 10
**Iterations completed**: 10

## Best Results Achieved

| Metric | Target | Best Achieved | Gap | Status |
|--------|--------|--------------|-----|--------|
| accuracy | 84.7% | 83.9% | 1.0% | ● CLOSE |
| f1_score | 76.2% | 70.1% | 8.0% | ✗ FAIL |

## Remaining Gaps Analysis

### f1_score (gap: 8.0%) — Top Priority
**Most likely cause**: [from LOOP_STATE.md outstanding_issues]
**Suggested next fix**: [...]
**Estimated iterations to close**: 2-3

## Recommendations

1. Run `/extend-loop --add-iterations 5` to continue
2. Or `/debug-gap --metric f1_score --depth deep` for deeper analysis
3. Consider `--effort max` for tighter tolerance (currently balanced/10%)
```

### FAILURE_REPORT.md (failed / blocked)

Written when a blocking error prevents any progress.

Contains: blocking error description, all attempted fixes, diagnostic information, and recommended manual intervention steps.

---

## `--compare-table` Flag

Adds a side-by-side comparison section showing:
- Paper's full results table (from `DATA_AND_EVAL.md → Results Validation`)
- Reproduced results for same metrics
- Baseline comparisons if paper reported them

---

## Output Location

| Report Type | File |
|------------|------|
| success | `REPRODUCTION_REPORT.md` |
| progress / timeout | `PROGRESS_REPORT.md` |
| failed / blocked | `FAILURE_REPORT.md` |

All reports are written to the project root. Existing reports are archived to `reports/` before overwriting.
