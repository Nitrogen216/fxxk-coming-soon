---
name: reproduce
description: Start or resume the autonomous paper reproduction loop. Implements code, runs experiments, evaluates against Paper Target Metrics in DATA_AND_EVAL.md, diagnoses gaps, and iterates until all metrics pass or budget is exhausted. Supports effort levels and optional cross-model review.
when_to_use: Use when starting a paper reproduction project, resuming after a break, or running the full autonomous loop from scratch.
argument-hint: "[--effort lite|balanced|max|beast] [--reviewer none|codex|gpt]"
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
context: fork
---

# Autonomous Paper Reproduction Loop

**Arguments**: $ARGUMENTS

## Current Project State
!`cat LOOP_STATE.md 2>/dev/null || echo "LOOP_STATE.md not found — will initialize from template"`

## Pre-flight Check

Verify these files exist before starting. Stop and report if any are missing:
- `PROJECT_STRUCTURE.md`
- `IMPLEMENTATION_PLAN.md`
- `DATA_AND_EVAL.md` (must contain `## Paper Target Metrics` YAML block)
- `RISKS_AND_NOTES.md`
- `CLAIMS_AND_GATES.md` (milestone gates M0-M4 + assumption ladder)
- `AGENT_LOOP_PROMPT.md`

If `LOOP_STATE.md` is missing, initialize it from the template: the user should have copied `templates/LOOP_STATE.md` from fxxk-coming-soon to the project root. Create a minimal one if not present.

## Argument Parsing

Parse `$ARGUMENTS` for flags:
- `--effort lite|balanced|max|beast` → update `LOOP_STATE.md` effort field
- `--reviewer none|codex|gpt` → update `LOOP_STATE.md` reviewer field

Effort defaults if not specified: inherit from `LOOP_STATE.md`, else `balanced`.

## State Check

Read `LOOP_STATE.md`:
- `status == "success"` → print results table, offer to continue at higher effort
- `status == "timeout"` → print best results, suggest `/extend-loop --add-iterations 5`
- `status == "blocked"` → print blocking error, suggest manual fix + `/reproduce`
- Otherwise → continue from `iteration_count`

## Execution

Follow `AGENT_LOOP_PROMPT.md` exactly. Run continuously through:

```
INITIALIZE → PHASE 0 (first iter) → PHASE 1 → PHASE 2 → PHASE 3 → PHASE 4 → loop
```

**Milestone progression** (from `CLAIMS_AND_GATES.md` if it exists):
1. M0 — sanity: dry-run passes
2. M1 — baseline: baseline metrics within tolerance
3. M2 — method: all primary metrics within tolerance → SUCCESS
4. M3 — ablation (effort=max/beast only)
5. M4 — external review (effort=beast or `--reviewer` set)

Do NOT jump to M2 without passing M1 first.

## Completion Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Reproduction: [Paper Name]
 Status: SUCCESS | TIMEOUT | BLOCKED
 Milestone: M2 passed | stuck at M1
 Iterations: N / max_N | Effort: balanced
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Metric       Target    Achieved  Gap      Status
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 accuracy     84.7%     85.1%     0.5%     ✓ PASS
 f1_score     76.2%     75.8%     0.5%     ✓ PASS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
