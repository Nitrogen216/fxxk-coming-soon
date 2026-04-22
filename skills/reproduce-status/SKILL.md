---
name: reproduce-status
description: Print a compact status snapshot of the current reproduction project. Reads LOOP_STATE.md and shows iteration count, milestone progress, metric gaps, and suggested next action. No experiments are run.
when_to_use: Use at the start of any session to orient yourself, or to quickly check progress mid-loop.
argument-hint: "[--verbose]"
allowed-tools:
  - Read
  - Bash
---

# Reproduction Status Snapshot

## Current State
!`cat LOOP_STATE.md 2>/dev/null || echo "LOOP_STATE.md not found"`

## Claims & Gates
!`cat CLAIMS_AND_GATES.md 2>/dev/null | head -60 || echo "No CLAIMS_AND_GATES.md found"`

## Instructions

Read `LOOP_STATE.md` and print a compact status table. If `LOOP_STATE.md` is missing, report "Not initialized — run /init-project first, then /setup-env, then /reproduce".

```
[fxxk-coming-soon] Paper Reproduction Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Paper:      [from PROJECT_STRUCTURE.md summary]
 Status:     RUNNING | SUCCESS | TIMEOUT | BLOCKED
 Milestone:  M1 ✓ baseline | M2 ⟳ method in progress
 Iteration:  4 / 10 | Effort: balanced (10% tol)

 Metrics (latest):
   accuracy    84.7% → 83.9%  gap 1.0%  ● CLOSE
   f1_score    76.2% → 75.4%  gap 1.0%  ● CLOSE
   precision   81.0% → 80.5%  gap 0.6%  ✓ PASS

 Top issue:   f1_score — check macro/micro averaging
 Plateau:     None detected

 Commands:
   /reproduce          — resume full loop
   /loop-once          — one manual iteration
   /debug-gap          — diagnose top failing metric
   /evaluate           — full metric report
   /extend-loop --add-iterations 5  — if timed out
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

If `status.state == "success"`:
```
[fxxk-coming-soon] ✓ REPRODUCTION COMPLETE
 All metrics within tolerance after N iterations
 Milestone: M2 passed
 Report: REPRODUCTION_REPORT.md
```
