# /run-experiment — Execute Experiment with Logging

**Invocation**: `/run-experiment [--config path] [--dry-run] [--resume] [--tag label]`

---

## Purpose

Execute the main experiment pipeline from `IMPLEMENTATION_PLAN.md → Entry Points` with structured logging, error classification, and automatic recovery. This skill is the execution engine called by the reproduction loop — it can also be invoked standalone for debugging or manual runs.

Use standalone when:
- You want to re-run a specific configuration without triggering a full loop iteration
- Debugging execution failures interactively
- Testing a specific code change before committing it to the loop

---

## Pre-Execution Checks

Before running:
1. Verify `venv` is activated — run `python -c "import sys; print(sys.prefix)"` and check it's the project venv
2. Verify the config file exists: `configs/paper_config.yaml` (or `--config` path)
3. Check disk space: estimated log + checkpoint size vs available space
4. If `--resume`: find the latest checkpoint in `results/` and pass it to the training script

---

## Execution

### Determine Run Command

From `IMPLEMENTATION_PLAN.md → Entry Points`, extract the main execution command. Common patterns:

```bash
# Training run
python main.py --config configs/paper_config.yaml

# With specific output directory (use current iteration number)
python main.py \
  --config configs/paper_config.yaml \
  --output results/iter_${N}/ \
  --seed 42
```

If `--dry-run`: append `--max_steps 10` or equivalent to run only a few steps for sanity checking.

### Logging

Pipe all output to iteration log:

```bash
python main.py --config configs/paper_config.yaml \
  2>&1 | tee logs/iter_${N}.log
```

Also write a metadata file `logs/iter_${N}_meta.json`:

```json
{
  "iteration": 3,
  "tag": "fix-lr-warmup",
  "start_time": "2024-01-15T14:23:00Z",
  "config": "configs/paper_config.yaml",
  "git_hash": "abc1234",
  "status": "running"
}
```

### Progress Monitoring

While the experiment runs, periodically check `logs/iter_${N}.log` for:
- Training loss trend (should decrease, not explode)
- Validation metrics appearing (confirms evaluation is running)
- OOM errors or CUDA errors (exit immediately)
- `NaN` in loss (stop, log, report `logic_error`)

Report progress every 10 minutes or on significant events.

---

## Error Classification & Recovery

| Error Pattern | Type | Auto-Recovery |
|--------------|------|---------------|
| `ModuleNotFoundError` | `dependency_error` | `pip install <package>`, retry once |
| `RuntimeError: CUDA out of memory` | `oom_error` | Halve batch size in config, retry |
| `FileNotFoundError: data/...` | `data_error` | Run `/setup-env --skip-pip`, retry once |
| `AssertionError`, shape mismatch | `logic_error` | Log, no retry — needs code fix |
| `loss: nan` after step N | `logic_error` | Log last valid checkpoint, stop |
| Process killed (OOM RAM) | `oom_error` | Reduce workers/prefetch, retry |

After 3 retries with no success: update `LOOP_STATE.md` with `execution_failed: true` and the error details.

---

## Post-Execution

On success:
1. Parse final metrics from `logs/iter_${N}.log` (patterns from `DATA_AND_EVAL.md`)
2. Save final model checkpoint to `results/iter_${N}/final_checkpoint/`
3. Copy best checkpoint to `results/best_checkpoint/` if metrics improved
4. Update `logs/iter_${N}_meta.json`: set `status = "complete"`, `end_time`, `duration_minutes`

Print execution summary:

```
Experiment Complete — Iteration 4
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Tag:       fix-lr-warmup
 Duration:  23m 41s  |  GPU peak: 11.2 GB
 Log:       logs/iter_4.log
 Checkpoint: results/iter_4/final_checkpoint/

 Final metrics seen in log:
   val_accuracy: 0.839
   val_f1:       0.754
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next: /evaluate to compare against paper targets
```

---

## Flags

| Flag | Effect |
|------|--------|
| `--config path` | Use alternate config file (default: `configs/paper_config.yaml`) |
| `--dry-run` | Run 10 steps only — verifies execution path without full training |
| `--resume` | Load latest checkpoint before running |
| `--tag label` | Attach a label to this run's logs (e.g., `--tag fix-dropout`) |
