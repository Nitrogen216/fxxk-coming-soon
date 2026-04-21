---
name: handoff-review
description: Prepare a structured review handoff package for an external reviewer (human or another model). Packages current results, evidence, and a structured question set. On receiving review feedback, parses verdict and updates LOOP_STATE.md with next actions.
when_to_use: Use at milestone M4 (external review), when effort=beast, or when you want an independent assessment of whether the reproduction is credible before declaring success.
argument-hint: "[--reviewer human|codex|gpt] [--receive] [--verdict score]"
allowed-tools:
  - Read
  - Write
  - Bash
---

# External Review Handoff

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null | head -40 || echo "No LOOP_STATE.md"`

## Latest Results
!`cat results/best_checkpoint/metrics.json 2>/dev/null || echo "No checkpoint metrics"`

## Instructions

Parse `$ARGUMENTS`:
- `--reviewer human|codex|gpt` → target reviewer type (default: human)
- `--receive` → parse incoming review feedback from the conversation and update LOOP_STATE.md
- `--verdict <score>` → manually record reviewer verdict (1-10 score)

---

### Mode 1: Prepare Handoff Package (default)

Generate `REVIEW_HANDOFF.md` with everything a reviewer needs:

```markdown
# Review Handoff: [Paper Title]

**Date**: [today]
**Iteration**: N / max_N
**Claimed status**: M2 passed (method reproduction) / M1 only / etc.

---

## What to Review

### 1. Metric Claims

| Metric | Paper Value | Achieved | Gap | Source |
|--------|-------------|---------|-----|--------|
| accuracy | 84.7% | 85.1% | 0.5% | Table 2, "Ours" |

### 2. Raw Evidence

- Training logs: `logs/iter_6.log`
- Evaluation output: `results/iter_6/eval_summary.json`
- Best checkpoint metrics: `results/best_checkpoint/metrics.json`
- Evaluation script: `python eval.py --config configs/paper_config.yaml`

### 3. Claim-to-Evidence Mapping

| Claim | Evidence | Confidence |
|-------|---------|-----------|
| accuracy within 1% of paper | logs/iter_6.log:L423 `val_acc: 0.851` | HIGH |
| macro-F1 used per paper | src/eval/metrics.py:L42 | HIGH |

### 4. Key Implementation Decisions

[List of non-obvious choices, assumptions, and paper ambiguities resolved]

### 5. Changes Since Last Review

[Git diff summary if prior review exists]

### 6. Unresolved Issues

[Outstanding issues from LOOP_STATE.md that didn't affect primary metrics]

---

## Reviewer Questions

1. Do the metric values in `logs/iter_6.log` match what's claimed above?
2. Is the evaluation protocol (dataset split, metric formula) correct per the paper?
3. Are there any implementation shortcuts that would inflate metrics unfairly?
4. Is there evidence of data leakage or evaluation set contamination?
5. Are the claimed results credible given the training curves in the logs?

---

## Expected Response Format

Please return your review as:

```json
{
  "score": 7,           // 1-10: credibility of reproduction claim
  "verdict": "almost",  // ready | almost | not_ready
  "critical_blockers": ["list of must-fix issues"],
  "minimum_fixes": ["list of specific changes needed"],
  "notes": "free-form observations"
}
```
```

Write `REVIEW_HANDOFF.md` and print its location.

---

### Mode 2: Receive Review Feedback (`--receive`)

Parse reviewer's JSON response from the conversation (or `--verdict <score>`).

Update `LOOP_STATE.md`:
```yaml
reviewer_score: 7
reviewer_verdict: almost        # ready | almost | not_ready
reviewer_blockers:
  - "Evaluation uses wrong test split"
reviewer_minimum_fixes:
  - "Use hold-out test set, not val set"
milestone_M4: pending           # passed | pending | failed
```

If `verdict == "ready"`: set milestone M4 passed, call `/write-report`.
If `verdict == "almost"`: add blockers to `outstanding_issues`, continue loop.
If `verdict == "not_ready"`: add critical issues, suggest `/debug-gap` or `/reset-loop`.

```
Review Received
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Score:    7/10
 Verdict:  ALMOST
 Blockers: 1 critical (wrong test split)
 M4:       pending

Next: Fix "use hold-out test set" → auto-added to LOOP_STATE.md
Run /loop-once to apply fix, then /handoff-review again
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
