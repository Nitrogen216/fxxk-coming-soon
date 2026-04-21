# LOOP_STATE.md — Reproduction Loop State

<!-- 
  Read and updated by Claude Code after each iteration.
  HUMAN-EDITABLE: Loop Control section (top) and Notes (bottom).
  Do NOT manually edit sections below "Milestone Progress" — Claude Code owns those.
-->

---

## Loop Control
<!-- Edit these to adjust loop behavior at any time -->

```yaml
effort: balanced          # lite | balanced | max | beast
max_iterations: 10
tolerance: 0.10           # 0.10 = 10% gap from paper target counts as pass
reviewer: none            # none | codex | gpt
gpu_hour_limit: 4         # flag if iteration exceeds this GPU-hour estimate
```

---

## Current Status

```yaml
iteration_count: 0
status: not_started       # not_started | running | success | timeout | blocked | interrupted
blocking_error: null      # populated if status == "blocked"
last_updated: null        # ISO timestamp
```

---

## Milestone Progress

```yaml
current_milestone: M0     # M0 | M1 | M2 | M3 | M4
milestones:
  M0_sanity:
    status: pending         # pending | passed | failed
    passed_at_iteration: null
    notes: "dry-run exits 0, imports work, data loads"
  M1_baseline:
    status: pending
    passed_at_iteration: null
    notes: "baseline method achieves paper's reported baseline metrics"
    baseline_targets: {}    # populated from CLAIMS_AND_GATES.md
  M2_method:
    status: pending
    passed_at_iteration: null
    notes: "proposed method achieves all primary metrics within tolerance"
  M3_ablation:
    status: pending         # only checked if effort >= max
    passed_at_iteration: null
  M4_review:
    status: pending         # only checked if effort == beast or reviewer != none
    reviewer_score: null
    reviewer_verdict: null  # ready | almost | not_ready
    reviewer_blockers: []
```

---

## Latest Metrics

| Metric | Target | Achieved | Gap | Status |
|--------|--------|----------|-----|--------|
| *(populated after first evaluation)* | | | | |

---

## Plateau Detection

```yaml
plateau_detected: false
plateau_metric: null
plateau_type: null        # A: asymptotic | B: oscillating | C: ceiling | D: random
plateau_since_iteration: null
plateau_escalation_applied: null
```

---

## Iteration History

| # | Milestone | Key Metric | Issues Found | Fix Applied |
|---|-----------|-----------|--------------|-------------|
| *(populated automatically)* | | | | |

---

## Outstanding Issues
<!-- Ordered by priority. Claude Code picks the top item each iteration. -->
<!-- You may add/reorder manually to guide the loop. -->

| Priority | Metric | Milestone | Gap | Hypothesis | Proposed Fix | File Hint |
|----------|--------|-----------|-----|------------|--------------|-----------|
| *(populated after first evaluation)* | | | | | | |

---

## Applied Fixes Log

| Iteration | Fix Description | Files Changed | Result |
|-----------|----------------|---------------|--------|
| *(populated automatically)* | | | |

---

## Notes
<!-- Human notes about this reproduction. Claude Code will read these. -->

*(Add any context about the paper, known issues, constraints, or manual decisions here)*
