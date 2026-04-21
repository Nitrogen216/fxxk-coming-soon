---
name: extend-loop
description: Add more iterations to a reproduction loop that has timed out (status=timeout). Resumes from current state with all history and metrics preserved. Does not reset any prior work.
when_to_use: Use when the loop has timed out but metrics are close to target (gap < 2× tolerance), or when you want to give the loop more attempts after analyzing the outstanding issues.
argument-hint: "[--add-iterations N] [--effort balanced|max|beast] [--tighten-tolerance]"
allowed-tools:
  - Read
  - Write
  - Bash
---

# Extend Reproduction Loop

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null || echo "No LOOP_STATE.md found"`

## Instructions

Parse `$ARGUMENTS`:
- `--add-iterations N` → add N to `max_iterations` (default: 5)
- `--effort <level>` → upgrade effort level for extended run
- `--tighten-tolerance` → halve the current tolerance (e.g., 10% → 5%)

### Step 1: Validate State

Read `LOOP_STATE.md`:
- If `status == "success"` → report "Already successful. To run at higher fidelity, use `/reproduce --effort max`"
- If `status != "timeout"` → warn "Loop has not timed out yet. Run `/reproduce` to continue"
- If `status == "blocked"` → report "Loop is blocked. Fix the blocking error first, then run `/reproduce`"
- If `status == "timeout"` → proceed

### Step 2: Assess Whether to Extend

Check if extension is likely to help:
- If ALL metrics have `gap < 2 × tolerance`: likely to converge — EXTEND
- If any metric has `gap > 5 × tolerance` AND showed < 1% improvement in last 3 iters: plateau warning
  - Recommend `/debug-gap --depth deep` first, then extend
  - Still extend if user explicitly invoked this skill

### Step 3: Update Control Parameters

In `LOOP_STATE.md`:

```yaml
# Before
iteration_count: 10
max_iterations: 10
status: timeout

# After /extend-loop --add-iterations 5
iteration_count: 10      # preserved (counting continues)
max_iterations: 15       # extended
status: running          # reset to running
```

Apply `--effort` changes if specified (per effort-contract.md).
Apply `--tighten-tolerance`: multiply `tolerance` by 0.5.

### Step 4: Report and Resume

Print what will happen next, then invoke the reproduction loop:

```
Loop Extended
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Budget: 10 → 15 iterations (+5)
 Tolerance: 10% (unchanged)
 Status: running (was: timeout)

 Resuming from iteration 10:
   accuracy  83.9% / 84.7%  gap: 1.0%  ● CLOSE
   f1_score  75.4% / 76.2%  gap: 1.0%  ● CLOSE

 Top issue: f1_score — try label smoothing 0.1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Starting iteration 11...
```

Then continue the loop from PHASE 1 of the next iteration per `AGENT_LOOP_PROMPT.md`.
