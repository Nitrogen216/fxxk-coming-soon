# /reproduce — Start Autonomous Paper Reproduction

**Invocation**: `/reproduce [--effort lite|balanced|max|beast] [--reviewer none|codex|gpt]`

---

## Purpose

Start or resume the full autonomous paper reproduction loop. Reads all 4 documentation files, initializes state, and runs continuously until success criteria are met or max iterations are exhausted.

This is the primary entry point. Call it once and let it run.

---

## Pre-flight Checklist

Before entering the loop, verify:

```
[ ] PROJECT_STRUCTURE.md    — architecture blueprint exists
[ ] IMPLEMENTATION_PLAN.md  — step-by-step guide exists
[ ] DATA_AND_EVAL.md        — contains "## Paper Target Metrics" section
[ ] RISKS_AND_NOTES.md      — challenge guidance exists
[ ] AGENT_LOOP_PROMPT.md    — master loop controller exists
```

If any file is missing: report exactly which files are absent, stop.

---

## Argument Handling

Parse invocation arguments and apply to `LOOP_STATE.md`:

| Argument | Default | Effect |
|----------|---------|--------|
| `--effort lite` | — | max_iter=5, tolerance=15% |
| `--effort balanced` | ✓ | max_iter=10, tolerance=10% |
| `--effort max` | — | max_iter=20, tolerance=5% |
| `--effort beast` | — | max_iter=50, tolerance=2% + ablations |
| `--reviewer none` | ✓ | No cross-model review |
| `--reviewer codex` | — | GPT-based code review after implementation |
| `--reviewer gpt` | — | GPT-based code review after implementation |

If `LOOP_STATE.md` already exists with a different effort level: honor the existing setting unless user explicitly passes `--effort`.

---

## Execution

1. **Parse arguments** — apply effort and reviewer settings to `LOOP_STATE.md`
2. **State check** — read `LOOP_STATE.md`:
   - `status == "success"` → print results table, ask if user wants `--effort max` run
   - `status == "timeout"` → print progress, offer to continue with `--effort beast`
   - `status == "blocked"` → print blocking error, suggest manual intervention
   - Otherwise → continue from `iteration_count`
3. **Execute loop** — follow `AGENT_LOOP_PROMPT.md` exactly:
   - PHASE 0: Environment setup (first iteration only)
   - PHASE 1: Implement / Improve
   - PHASE 2: Execute
   - PHASE 3: Evaluate
   - PHASE 4: Diagnose & Decide → loop or stop
4. **Report on completion** — print final status

---

## Output on Completion

Print a human-readable summary table:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Reproduction: [Paper Name]
 Status: SUCCESS | TIMEOUT | BLOCKED
 Iterations: N / max_N | Effort: balanced
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Metric       Target    Achieved  Gap      Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 accuracy     84.7%     85.1%     0.5%     ✓ PASS
 f1_score     76.2%     75.8%     0.5%     ✓ PASS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

On success: announce `REPRODUCTION_REPORT.md` location.
On failure: list top 3 remaining gaps and suggested next steps.
