---
name: save-checkpoint
description: Create a git commit capturing current code, configs, LOOP_STATE.md, and metrics. Auto-generates commit message from current metrics. Archives best model checkpoint metrics for recovery.
when_to_use: Use after a successful iteration, before a risky code change, or at the end of a working session to ensure no progress is lost.
argument-hint: '[--message "description"] [--tag label]'
disable-model-invocation: false
allowed-tools:
  - Bash
  - Read
---

# Save Reproducible Checkpoint

**Arguments**: $ARGUMENTS

## Current Git Status
!`git status --short 2>/dev/null | head -20`

## Current Metrics
!`python3 -c "
try:
    c = open('LOOP_STATE.md').read()
    idx = c.find('latest_metrics')
    if idx>=0: print(c[idx:idx+400])
except: print('No metrics')
" 2>/dev/null`

## Instructions

Parse `$ARGUMENTS`:
- `--message "description"` → use as commit message prefix
- `--tag <label>` → also create a git tag (e.g., `repro/iter-6-fix-f1`)

### Step 1: Stage Files

```bash
git add src/ configs/ LOOP_STATE.md requirements.txt
git add REPRODUCTION_REPORT.md PROGRESS_REPORT.md CLAIMS_AND_GATES.md 2>/dev/null || true
git add results/best_checkpoint/metrics.json 2>/dev/null || true
```

Never stage: `venv/`, `data/raw/`, `*.pt`, `*.bin`, `*.safetensors`, `logs/`, `__pycache__/`.

### Step 2: Generate Commit Message

If `--message` provided: use it as-is.

Otherwise, auto-generate from `LOOP_STATE.md`:

```
Iteration N: accuracy 83.9% (+1.8%), f1 75.4% (+5.3%)
Fix: [top fix from fixes_applied[-1]]

Primary metrics: accuracy 83.9%/84.7% (1.0% gap) | f1_score 75.4%/76.2% (1.0% gap)
Status: 1/2 metrics passing | Effort: balanced | Iter: N/10
```

### Step 3: Commit

```bash
git commit -m "$MESSAGE"
```

### Step 4: Tag (with `--tag`)

```bash
git tag "repro/$LABEL" -m "Iteration N checkpoint"
```

### Step 5: Update Best Checkpoint

If current iteration has best metrics so far:
```bash
mkdir -p results/best_checkpoint
# Save metrics.json (not weights — too large for git)
echo '{"iteration": N, "accuracy": X, "f1": Y}' > results/best_checkpoint/metrics.json
git add results/best_checkpoint/metrics.json && git commit --amend --no-edit
```

## Output

```
Checkpoint Saved
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Commit:  abc1234 "Iteration 6: accuracy 83.9%..."
 Files:   12 changed
 Tag:     repro/iter-6 (if --tag used)
 Best:    results/best_checkpoint/ updated (accuracy 82.1% → 83.9%)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restore: git checkout repro/iter-6
```
