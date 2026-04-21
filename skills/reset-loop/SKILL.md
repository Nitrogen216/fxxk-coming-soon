# /reset-loop — Reset Loop for a Fresh Run

**Invocation**: `/reset-loop [--keep-history] [--keep-code] [--hard]`

---

## Purpose

Reset the reproduction loop state to start fresh, while preserving what you've built. Use this when:
- You want to restart after significantly changing the implementation
- The loop got stuck in a bad state and needs cleaning
- You want to test a completely different approach
- You're benchmarking: comparing different implementations from scratch

By default, resets the loop state but keeps all code, configs, and the full iteration history (archived).

---

## Reset Modes

### Default (`/reset-loop`)

Safe reset — preserves everything recoverable:
- Archive current `LOOP_STATE.md` to `archive/loop_state_YYYYMMDD_HHMMSS.md`
- Create fresh `LOOP_STATE.md` from template (`not_started`, `iteration_count: 0`)
- Keep all code, configs, requirements
- Keep `logs/` and `results/` directories (just start new iter numbering)
- Keep `REPRODUCTION_REPORT.md` / `PROGRESS_REPORT.md` if they exist

### Keep History (`--keep-history`)

Same as default, but also copies `iteration_history` from the old `LOOP_STATE.md` into the new one, clearly marked as "pre-reset":

```yaml
iteration_history:
  # --- Pre-reset iterations (archived run) ---
  - iteration: "archive/1"
    achieved: {accuracy: 0.751}
    fixes_applied: ["initial implementation"]
  # --- New run starts below ---
```

### Keep Code (`--keep-code`)

Reset everything EXCEPT the source code:
- Fresh `LOOP_STATE.md`
- Clear `logs/` and `results/`
- Clear installed packages (delete `venv/`)
- **Keep**: `src/`, `configs/`, `requirements.txt`
- Use when: testing if the same code reproduces from scratch

### Hard Reset (`--hard`)

Full reset — start from zero:
- Fresh `LOOP_STATE.md`
- Delete `logs/`, `results/`
- Delete `venv/`
- Delete `data/processed/` (keep `data/raw/` to avoid re-downloading)
- **Keep**: documentation files, `src/`, `configs/`
- Use when: the implementation was fundamentally wrong and needs total reimplementation

---

## Execution

```bash
# Always create a git commit before resetting
git add -A && git commit -m "Pre-reset checkpoint: iter ${N}, best accuracy ${X}"

# Archive current state
mkdir -p archive/
cp LOOP_STATE.md archive/loop_state_$(date +%Y%m%d_%H%M%S).md

# Fresh LOOP_STATE.md
cp templates/LOOP_STATE.md LOOP_STATE.md  # from fxxk-coming-soon templates

# [--keep-code / --hard only] remove dirs
rm -rf logs/ results/ venv/   # --hard
```

---

## Output

```
Loop Reset Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Mode:     default (safe)
 Archived: archive/loop_state_20240115_143022.md
   Previous run: 10 iterations | best accuracy 83.9%

 Fresh LOOP_STATE.md:
   iteration_count: 0
   status: not_started
   effort: balanced (inherited from previous run)

 Kept:
   ✓ src/   ✓ configs/   ✓ requirements.txt
   ✓ logs/  ✓ results/   ✓ REPRODUCTION_REPORT.md

 Pre-reset checkpoint committed: git abc1234
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Ready. Run /reproduce to start fresh.
```

---

## Warning

`/reset-loop --hard` is destructive. It always creates a git commit first, but if git is not initialized or the commit fails, it will prompt for confirmation before proceeding.
