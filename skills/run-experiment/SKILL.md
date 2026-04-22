---
name: run-experiment
description: Execute the paper reproduction experiment pipeline with structured logging, automatic error classification (dependency/logic/data/oom), up to 3 retries, and best-checkpoint tracking.
when_to_use: Use to run the main experiment, test a configuration change, do a dry-run sanity check, or resume from a checkpoint.
argument-hint: "[--config path] [--dry-run] [--resume] [--tag label]"
allowed-tools:
  - Bash
  - Read
  - Write
---

# Execute Experiment

**Arguments**: $ARGUMENTS

## Current Iteration
!`python3 -c "
try:
    c=open('LOOP_STATE.md').read()
    for l in c.splitlines():
        if 'iteration_count' in l: print(l.strip())
except: print('iteration_count: unknown')
" 2>/dev/null`

## Available Configs
!`ls configs/*.yaml configs/*.yml 2>/dev/null || echo "No config files found yet"`

## Instructions

Parse `$ARGUMENTS`:
- `--config <path>` → use this config file (default: `configs/paper_config.yaml`)
- `--dry-run` → append `--max_steps 10` or equivalent; just verify the execution path
- `--resume` → find latest checkpoint in `results/` and pass to training script
- `--tag <label>` → attach label to logs for this run

**Current iteration N**: read from `LOOP_STATE.md → status.iteration_count`.

### Pre-run Checks

1. Verify venv is activated: `python -c "import sys; print(sys.prefix)"`
2. Verify config file exists
3. Estimate disk usage; warn if < 10GB available
4. If `--resume`: find `results/iter_${N-1}/final_checkpoint/` or `results/best_checkpoint/`

### Execution

```bash
source venv/bin/activate

python main.py \
  --config $CONFIG \
  --output results/iter_${N}/ \
  --seed 42 \
  $([[ --dry-run ]] && echo "--max_steps 10") \
  $([[ --resume ]] && echo "--resume $CHECKPOINT") \
  2>&1 | tee logs/iter_${N}.log
```

Write metadata before running:
```json
{
  "iteration": N, "tag": "$TAG",
  "start_time": "ISO timestamp",
  "config": "$CONFIG",
  "git_hash": "$(git rev-parse --short HEAD)",
  "status": "running"
}
```
→ save to `logs/iter_${N}_meta.json`

### Progress Monitoring

While running, watch `logs/iter_${N}.log` for:
- Loss diverging (NaN, inf, > 100× initial) → stop immediately
- OOM error → attempt batch size halving, restart
- Periodic metric reports → confirm evaluation is running

### Error Classification & Recovery

| Error Pattern | Type | Auto-fix |
|--------------|------|---------|
| `ModuleNotFoundError` | `dependency_error` | `pip install <pkg>`, retry once |
| `RuntimeError: CUDA out of memory` | `oom_error` | Halve batch size in config, retry |
| `FileNotFoundError: data/` | `data_error` | Run `/setup-env --skip-pip`, retry once |
| `loss: nan` | `logic_error` | Save checkpoint, stop — needs code fix |
| `AssertionError`, shape mismatch | `logic_error` | Stop — needs code fix |

After 3 retries with no success: log to `LOOP_STATE.md` as `execution_failed: true`, do not retry further.

### Post-run

On success:
1. Save checkpoint to `results/iter_${N}/final_checkpoint/`
2. If metrics improved: copy to `results/best_checkpoint/`
3. Update `logs/iter_${N}_meta.json`: `status=complete`, `end_time`, `duration_minutes`

```
Experiment Complete — Iteration 4 | tag: fix-f1-averaging
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Duration: 23m 41s  |  GPU peak: 11.2 GB
 Log: logs/iter_4.log
 Final metrics in log: val_accuracy=0.839, val_f1=0.754
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next: /evaluate
```
