# LOOP_STATE.md — Reproduction Loop State

<!--
  Single source of truth for loop progress. Read and updated by Claude Code after each iteration.

  FORMAT POLICY: Every data section below is YAML inside a ```yaml``` code block.
  Do NOT convert to Markdown tables — skills parse these blocks as YAML.

  HUMAN-EDITABLE: `loop_control` and `notes` sections only.
  Claude Code OWNS: everything else. Do not hand-edit.
-->

---

## Loop Control

<!-- Edit these to adjust loop behavior at any time -->

```yaml
loop_control:
  effort: balanced              # lite | balanced | max | beast
  max_iterations: 10
  tolerance: 0.10               # 0.10 = 10% gap from paper target counts as pass
  reviewer: none                # none | codex | gpt
  gpu_hour_limit: 4             # per-iteration soft cap
  total_gpu_hour_budget: 40     # cumulative cap across all iterations
```

---

## Current Status

```yaml
status:
  state: not_started            # not_started | running | success | timeout | blocked | interrupted
  iteration_count: 0
  total_gpu_hours_consumed: 0.0
  blocking_error: null          # populated if state == "blocked"
  last_updated: null            # ISO timestamp
```

---

## Milestone Progress

```yaml
milestones:
  current: M0                   # M0 | M1 | M2 | M3 | M4

  M0_sanity:
    status: pending             # pending | passed | failed
    passed_at_iteration: null
    notes: "dry-run exits 0, imports work, data loads"

  M1_baseline:
    status: pending
    passed_at_iteration: null
    notes: "baseline method achieves paper's reported baseline metrics"
    baseline_targets: {}        # auto-populated from CLAIMS_AND_GATES.md baseline_metrics

  M2_method:
    status: pending
    passed_at_iteration: null
    notes: "proposed method achieves all primary metrics within tolerance"

  M3_ablation:
    status: pending             # only checked if effort >= max
    passed_at_iteration: null

  M4_review:
    status: pending             # only checked if effort == beast or reviewer != none
    reviewer_score: null
    reviewer_verdict: null      # ready | almost | not_ready
    reviewer_blockers: []
```

---

## Latest Metrics

```yaml
latest_metrics:
  # One entry per metric in DATA_AND_EVAL.md → Paper Target Metrics → primary_metrics
  # Populated after first evaluation.
  # Example:
  # accuracy:
  #   target: 0.847
  #   achieved: 0.821
  #   gap: 0.031
  #   gap_pct: "3.1%"
  #   status: fail              # pass | fail
  #   measured_at_iteration: 3
```

---

## Plateau Detection

```yaml
plateau:
  detected: false
  metric: null
  type: null                    # A: asymptotic | B: oscillating | C: ceiling | D: random
  since_iteration: null
  escalations_applied: []       # list of strategies tried, e.g. ["lr_reduce_10x", "gradient_clip"]
```

---

## Iteration History

```yaml
iteration_history:
  # One entry appended per completed iteration.
  # Example:
  # - iteration: 1
  #   milestone: M1
  #   achieved: {accuracy: 0.751}
  #   issues_found: ["data normalization computed globally, not per-channel"]
  #   fixes_applied: ["fixed mean/std to per-channel in data/preprocessing.py"]
  #   gpu_hours: 0.8
  #   log_path: "logs/iter_1.log"
```

---

## Outstanding Issues

<!-- Priority-ordered. Claude Code picks the top item each iteration. -->
<!-- You may add/reorder manually to guide the loop. -->

```yaml
outstanding_issues:
  # Example:
  # - priority: high            # high | medium | low
  #   metric: f1_score
  #   milestone: M2
  #   gap: "8.0%"
  #   hypothesis: "Paper uses macro-F1; code uses micro"
  #   proposed_fix: "Change average='micro' to average='macro' in metrics.py:compute_f1"
  #   file_hint: "src/eval/metrics.py:42"
  #   added_at_iteration: 2
```

---

## Applied Fixes Log

```yaml
fixes_applied:
  # Example:
  # - iteration: 1
  #   description: "Fixed per-channel data normalization"
  #   files_changed: ["src/data/preprocessing.py"]
  #   result: "accuracy 0.751 → 0.803 (+7.0%)"
```

---

## Notes

<!-- Human notes about this reproduction. Claude Code will read these. -->

*(Add any context about the paper, known issues, constraints, or manual decisions here)*
