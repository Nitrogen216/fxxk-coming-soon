---
name: write-report
description: Generate a formal reproduction report. Auto-detects report type from LOOP_STATE.md. Writes REPRODUCTION_REPORT.md on success, PROGRESS_REPORT.md on timeout, FAILURE_REPORT.md on blocking errors.
when_to_use: Use when the reproduction loop completes (success or timeout), to document current progress at any point, or when handing off the project to a reviewer.
argument-hint: "[--type success|progress|failed] [--compare-table]"
allowed-tools:
  - Read
  - Write
  - Bash
---

# Generate Reproduction Report

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null || echo "No LOOP_STATE.md found"`

## Best Results
!`cat results/best_checkpoint/metrics.json 2>/dev/null || echo "No checkpoint metrics found"`

## Instructions

Parse `$ARGUMENTS`:
- `--type success|progress|failed` → override auto-detected type
- `--compare-table` → add side-by-side comparison with paper's full results table

**Auto-detect type** from `LOOP_STATE.md → status`:
- `success` → REPRODUCTION_REPORT.md
- `timeout` or no status → PROGRESS_REPORT.md
- `blocked` → FAILURE_REPORT.md

Archive any existing report to `reports/` before writing new one.

---

### REPRODUCTION_REPORT.md (success)

```markdown
# Reproduction Report: [Paper Title]

**Status**: SUCCESS ✓
**Date**: [today]
**Iterations needed**: N / max_N
**Effort**: balanced (10% tolerance)

---

## Metric Comparison

| Metric | Paper Value | Reproduced | Gap | Status |
|--------|-------------|------------|-----|--------|
| accuracy | 84.7% | 85.1% | 0.5% | ✓ PASS |

## Milestone Summary

| Milestone | Description | Status |
|-----------|-------------|--------|
| M0 | Sanity (dry-run) | ✓ passed iteration 0 |
| M1 | Baseline reproduction | ✓ passed iteration 2 |
| M2 | Method reproduction | ✓ passed iteration 6 |

## Key Implementation Notes
[Deviations from paper, clarifications, assumptions made]

## Iteration Log
[Compact history from LOOP_STATE.md iteration_history]

## Environment & Reproducibility
- Python / PyTorch / CUDA versions
- Hardware used
- Random seeds: 42 (all)
- Command: `python main.py --config configs/paper_config.yaml --seed 42`
```

### PROGRESS_REPORT.md (timeout)

Includes best achieved, remaining gaps, top 3 hypotheses for closing them, and suggested next steps (`/extend-loop`, `/debug-gap`, specific fix).

### FAILURE_REPORT.md (blocked)

Includes: blocking error description, all attempted fixes, diagnostic findings, and concrete manual intervention instructions.

---

## Output

Announce report location and key summary:
```
Report written: REPRODUCTION_REPORT.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Paper:    [title]
 Status:   SUCCESS after 6 iterations
 M1 (baseline): passed iteration 2
 M2 (method):   passed iteration 6
 All 2/2 primary metrics within 10% tolerance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
