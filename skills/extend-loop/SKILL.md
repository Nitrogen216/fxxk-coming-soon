# /extend-loop — Add Iterations to an Exhausted Loop

**Invocation**: `/extend-loop [--add-iterations N] [--effort balanced|max|beast] [--tighten-tolerance]`

---

## Purpose

When the loop reaches `max_iterations` without full success (status: `timeout`), this skill extends the budget and resumes from where it stopped. All history is preserved — the loop continues from the last iteration's state.

Use this instead of `/reset-loop` when you want to give the loop more attempts without discarding any prior work.

---

## When to Use

```
Status: TIMEOUT (reached 10/10 iterations)
Best: accuracy 83.9% (gap 1.0%), f1_score 75.4% (gap 1.0%)
Both metrics are CLOSE — just need 2-3 more iterations to converge
→ /extend-loop --add-iterations 5
```

vs.

```
Status: TIMEOUT (reached 10/10 iterations)
Best: accuracy 75.1% (gap 11.3%) — barely moved in last 3 iterations
Implementation may be fundamentally wrong
→ /reset-loop or /debug-gap --depth deep first
```

---

## Execution

### Step 1: Validate State

Read `LOOP_STATE.md`:
- If `status != "timeout"`: warn that extending a non-exhausted loop is unusual, ask to confirm
- If `iteration_count < max_iterations`: inform that the loop isn't exhausted yet — run `/reproduce` instead
- If `status == "success"`: inform that reproduction already succeeded, suggest `--effort beast` for higher fidelity

### Step 2: Update Control Parameters

Apply the new budget to `LOOP_STATE.md`:

```yaml
# Before
iteration_count: 10
max_iterations: 10
status: timeout

# After /extend-loop --add-iterations 5
iteration_count: 10      # preserved — counting continues
max_iterations: 15       # extended
status: running          # reset to running
```

If `--effort max` or `--effort beast`: also update `tolerance` per `skills/shared/effort-contract.md`.

If `--tighten-tolerance`: reduce tolerance by 50% (e.g., 10% → 5%) to aim for closer match.

### Step 3: Resume Loop

Continue from the current `outstanding_issues` in `LOOP_STATE.md`. The next iteration will be iteration N+1 where N is the last completed iteration.

Print what will happen next:

```
Resuming from iteration 10/10 → will run up to iteration 15

Top outstanding issue for next iteration:
  [HIGH] f1_score 1.0% gap — try tightening class weight schedule
  Proposed fix: Use label smoothing 0.1 per paper footnote 3

Run /reproduce to start the extended loop.
```

Then automatically invoke the reproduce loop.

---

## Flags

| Flag | Default | Effect |
|------|---------|--------|
| `--add-iterations N` | 5 | Add N to current max_iterations |
| `--effort level` | inherit | Change effort level for extended run |
| `--tighten-tolerance` | false | Halve the current tolerance threshold |

---

## Output

```
Loop Extended
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Previous budget:  10 iterations (exhausted)
 Extended budget:  15 iterations (+5)
 Tolerance:        10% → 10% (unchanged)

 Resuming from iteration 10:
   accuracy:  83.9% / 84.7%  gap: 1.0%  ● CLOSE
   f1_score:  75.4% / 76.2%  gap: 1.0%  ● CLOSE

 Next fix: [top outstanding_issue description]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Starting iteration 11...
```
