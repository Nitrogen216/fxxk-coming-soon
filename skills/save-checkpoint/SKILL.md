# /save-checkpoint — Snapshot Current Reproducible State

**Invocation**: `/save-checkpoint [--message "description"] [--tag label]`

---

## Purpose

Create a git commit that captures the current state: code, configuration, metrics, and loop state. This creates a recoverable snapshot so that no progress is ever lost between sessions or after bad experiments.

Run this after a successful iteration, before a risky code change, or at the end of a working session.

---

## What Gets Committed

| Included | Excluded (in .gitignore) |
|----------|--------------------------|
| All source code (`src/`) | Model weights (`*.pt`, `*.bin`, `*.safetensors`) |
| Configuration files (`configs/`) | Raw datasets (`data/raw/`) |
| `LOOP_STATE.md` | Full experiment logs (`logs/`) |
| `requirements.txt` | Cache directories (`__pycache__/`, `.cache/`) |
| `REPRODUCTION_REPORT.md` (if exists) | Virtual environment (`venv/`) |
| `PROGRESS_REPORT.md` (if exists) | Temporary files |

Exception: `results/best_checkpoint/` — the best model weights **are** committed in a compressed form if under 100MB.

---

## Execution

### Step 1: Stage Files

```bash
git add src/ configs/ LOOP_STATE.md requirements.txt
git add REPRODUCTION_REPORT.md PROGRESS_REPORT.md 2>/dev/null || true
```

### Step 2: Generate Commit Message

Auto-generate from current `LOOP_STATE.md` state if `--message` not provided:

```
Iteration 6: accuracy 83.9% (+1.8%), f1 75.4% (+5.3%)
Fix: Switched to macro-weighted F1 in metrics.py

Primary metrics: accuracy 83.9%/84.7% (1.0% gap) | f1_score 75.4%/76.2% (1.0% gap)
Status: 1/2 metrics passing | Effort: balanced | Iter: 6/10
```

### Step 3: Commit

```bash
git commit -m "<generated or provided message>"
```

### Step 4: Tag (optional, with `--tag`)

```bash
git tag "repro/iter-6-fix-f1" -m "Iteration 6 checkpoint: f1 gap 8.0%→1.0%"
```

---

## Best Checkpoint Archive

If the current results are the best so far (per `LOOP_STATE.md` iteration history):

```bash
mkdir -p results/best_checkpoint
cp results/iter_${N}/final_checkpoint/* results/best_checkpoint/
echo '{"iteration": 6, "accuracy": 0.839, "f1": 0.754}' > results/best_checkpoint/metrics.json
git add results/best_checkpoint/metrics.json
```

Only commit `metrics.json` — not the weights themselves (too large for git).

---

## Output

```
Checkpoint Saved — Iteration 6
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Commit:  abc1234 "Iteration 6: accuracy 83.9%..."
 Files:   12 changed, 847 insertions(+), 23 deletions(-)
 Tag:     repro/iter-6-fix-f1

 Best checkpoint updated: results/best_checkpoint/
   accuracy: 83.9% (prev best: 82.1%)
   f1_score: 75.4% (prev best: 70.1%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
To restore this checkpoint: git checkout abc1234
```

---

## Recovery

To restore a previous checkpoint:

```bash
git log --oneline | grep "repro/"  # list all checkpoints
git checkout repro/iter-6-fix-f1   # restore specific checkpoint
```

To see what changed between two checkpoints:

```bash
git diff repro/iter-4 repro/iter-6
```
