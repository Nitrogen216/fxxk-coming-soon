# LOOP_STATE.md — Reproduction Loop State

<!-- 
  This file is read and updated by Claude Code automatically after each iteration.
  Human-editable: the Loop Control section at the top.
  Do not manually edit sections below "Latest Metrics" — Claude Code owns those.
-->

---

## Loop Control
<!-- Edit these parameters to adjust loop behavior -->

```yaml
effort: balanced          # lite | balanced | max | beast
max_iterations: 10
tolerance: 0.10           # 0.10 = 10% gap from paper target counts as pass
reviewer: none            # none | codex | gpt
gpu_hour_limit: 4         # flag if iteration exceeds this many GPU hours
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

## Latest Metrics

| Metric | Target | Achieved | Gap | Status |
|--------|--------|----------|-----|--------|
| *(populated after first evaluation)* | | | | |

---

## Iteration History

| # | Key Metric | Issues Found | Fix Applied |
|---|-----------|--------------|-------------|
| *(populated automatically)* | | | |

---

## Outstanding Issues
<!-- Ordered by priority. Claude Code picks the top item each iteration. -->
<!-- You may add/reorder manually to guide the loop. -->

| Priority | Metric | Gap | Hypothesis | Proposed Fix | File Hint |
|----------|--------|-----|------------|--------------|-----------|
| *(populated after first evaluation)* | | | | | |

---

## Applied Fixes Log

| Iteration | Fix Description | Files Changed | Result |
|-----------|----------------|---------------|--------|
| *(populated automatically)* | | | |

---

## Notes
<!-- Human notes about this reproduction. Claude Code will read these. -->

*(Add any context about the paper, known issues, or constraints here)*
