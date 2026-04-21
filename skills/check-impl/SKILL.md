---
name: check-impl
description: Audit the current implementation against all three paper documentation files (PROJECT_STRUCTURE.md, IMPLEMENTATION_PLAN.md, DATA_AND_EVAL.md). Finds missing functions, wrong configurations, and metric formula errors. Auto-populates LOOP_STATE.md with blocking issues found.
when_to_use: Use before a long training run, when metrics are unexpectedly low with no obvious cause, or to verify implementation completeness after writing new code.
argument-hint: "[--section all|structure|plan|eval] [--strict]"
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Implementation Audit

**Arguments**: $ARGUMENTS

## Documentation Files Present
!`ls PROJECT_STRUCTURE.md IMPLEMENTATION_PLAN.md DATA_AND_EVAL.md RISKS_AND_NOTES.md 2>&1`

## Source Files
!`find src/ -name "*.py" 2>/dev/null | head -30 || echo "No src/ directory yet"`

## Instructions

Parse `$ARGUMENTS`:
- `--section structure` → audit against PROJECT_STRUCTURE.md only
- `--section plan` → audit against IMPLEMENTATION_PLAN.md only
- `--section eval` → audit against DATA_AND_EVAL.md only
- `--section all` → all three (default)
- `--strict` → also compare hyperparameter values vs paper's reported numbers

---

### Section: Structure

Read `PROJECT_STRUCTURE.md`. For each documented file and function:
1. Verify file exists in `src/`
2. Verify each public function/class exists (grep by name)
3. Verify function signature matches (parameter count and names)
4. Verify the file can be imported without error

Report per-file status.

### Section: Plan

Read `IMPLEMENTATION_PLAN.md`. Verify:
1. All four phases have implementations:
   - Phase 1 (Infra): config loading, logging utilities
   - Phase 2 (Data): dataset loader, preprocessing pipeline
   - Phase 3 (Model): core model, training loop
   - Phase 4 (Eval): metric computation, reporting
2. Run a dry forward pass: `Dataset → DataLoader → Model → Loss` (sanity check)
3. Verify `configs/paper_config.yaml` covers all parameters in documented schema

With `--strict`: compare each config value against paper's reported hyperparameters from `DATA_AND_EVAL.md → Experimental Protocols`. Flag any mismatch > 5%.

### Section: Eval

Read `DATA_AND_EVAL.md`. Verify:
1. Each primary metric has an implementation (grep for function name)
2. The `## Paper Target Metrics` YAML block is populated in `LOOP_STATE.md`
3. Evaluation script runs: `python eval.py --help` exits 0
4. Random seeds are set in all required locations (Python/NumPy/PyTorch/CUDA)
5. Test dataset matches paper's specification (check `DATA_AND_EVAL.md → Dataset Specifications`)

---

## Output Format

```
Implementation Audit — 2024-01-15 | Strict: OFF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Structure:  14/15 files ✓ | 31/34 functions ✓ | 13/14 imports OK
Plan:       Phase 1 ✓ | Phase 2 ✓ | Phase 3 ⚠ partial | Phase 4 ✓
            Forward pass: ✓ (loss=2.341)
Eval:       3/3 metric formulas match | Seeds: ✓ all set

BLOCKING issues (added to LOOP_STATE.md):
  1. src/models/attention.py: 3 functions missing — see PROJECT_STRUCTURE.md:L44
  2. src/data/augment.py: ImportError on albumentations — add to requirements.txt

WARNINGS:
  1. configs/paper_config.yaml: lr_scheduler not set (paper uses cosine warmup)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Recommendation: Fix 2 blocking issues before /run-experiment
```

Auto-add all `BLOCKING` issues to `LOOP_STATE.md → outstanding_issues` with `priority: high`.
