---
name: reset-loop
description: Reset the reproduction loop state for a fresh run. Multiple modes from safe (preserve code, archive state) to hard (full clean). Always creates a git commit before resetting.
when_to_use: Use when you want to restart from scratch, after a fundamental implementation change, or to benchmark different approaches.
argument-hint: "[--keep-history] [--keep-code] [--hard]"
disable-model-invocation: true
allowed-tools:
  - Bash
  - Read
  - Write
---

# Reset Reproduction Loop

**Arguments**: $ARGUMENTS

## Current State
!`cat LOOP_STATE.md 2>/dev/null | head -20 || echo "No LOOP_STATE.md — nothing to reset"`

## Warning

`/reset-loop` with `--hard` is destructive. This skill **always creates a git commit first** to preserve recovery ability. If git is not initialized: report and stop unless user explicitly confirms.

## Instructions

Parse `$ARGUMENTS`:
- `--keep-history` → copy `iteration_history` into new LOOP_STATE.md marked "pre-reset"
- `--keep-code` → preserve `src/` and `configs/` but clear logs, results, venv
- `--hard` → clear everything except `src/`, `configs/`, `data/raw/` (avoid re-download)
- No flags → safe reset: archive state, fresh LOOP_STATE.md, keep everything else

### Step 1: Git Commit (always)

```bash
git add -A
git commit -m "Pre-reset checkpoint: iter ${N}, best ${METRIC}=${VALUE}"
```

If commit fails (nothing staged): still proceed, just warn that recovery requires prior commits.

### Step 2: Archive Current State

```bash
mkdir -p archive/
cp LOOP_STATE.md archive/loop_state_$(date +%Y%m%d_%H%M%S).md
cp REPRODUCTION_REPORT.md archive/ 2>/dev/null || true
cp PROGRESS_REPORT.md archive/ 2>/dev/null || true
```

### Step 3: Apply Reset Mode

**Default (safe)**:
- Create fresh `LOOP_STATE.md` from template
- Inherit `effort:` and `max_iterations:` from previous run
- Keep `src/`, `configs/`, `logs/`, `results/`, `data/`

**`--keep-history`**: Same as default, plus copy old `iteration_history` into new file under `# Pre-reset history` section.

**`--keep-code`**: Default + also clear `logs/`, `results/iter_*/` (keep `results/best_checkpoint/`), delete `venv/`.

**`--hard`**: `--keep-code` + also delete `data/processed/`. Print: "Hard reset complete. Keep data/raw/ to avoid re-downloading."

### Step 4: Initialize Fresh State

Fresh `LOOP_STATE.md` inherits:
- `effort:` from archived state (continuity)
- `max_iterations:` from archived state
- Everything else reset to defaults (`iteration_count: 0`, `status: not_started`)

## Output

```
Loop Reset Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Mode:     default (safe)
 Archived: archive/loop_state_20240115_143022.md
 Previous: 10 iterations | best accuracy 83.9%
 Pre-reset commit: git abc1234

 Fresh LOOP_STATE.md:
   iteration_count: 0 | status: not_started
   effort: balanced (inherited)

 Kept: src/ configs/ logs/ results/ data/
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ready. Run /reproduce to start fresh.
```
