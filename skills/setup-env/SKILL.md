---
name: setup-env
description: Initialize the Python environment, install dependencies, download datasets, and verify the setup with a smoke test. Idempotent — safe to run multiple times.
when_to_use: Use at project start, after environment corruption, when datasets are missing, or before a long training run to verify everything is ready.
argument-hint: "[--python 3.10|3.11|3.12] [--gpu] [--skip-data]"
allowed-tools:
  - Bash
  - Read
  - Write
---

# Environment Setup

**Arguments**: $ARGUMENTS

## System Information
!`python3 --version 2>&1; pip --version 2>&1; nvidia-smi 2>/dev/null | head -10 || echo "No GPU detected"`

## Instructions

Parse `$ARGUMENTS`:
- `--python 3.10|3.11|3.12` → create venv with that Python version
- `--gpu` → run extra GPU verification steps
- `--skip-data` → skip dataset download (useful when data is already present)

This skill is **idempotent** — always check if a step is already done before repeating it.

### Step 1: Python Environment

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

If `requirements.txt` doesn't exist: create a minimal one from `PROJECT_STRUCTURE.md → Technology Stack` section.

On pip conflicts: install packages one by one to isolate the issue. Check `RISKS_AND_NOTES.md → Library compatibility concerns`.

### Step 2: Directory Structure

Create all directories from `PROJECT_STRUCTURE.md → Repository Tree`:

```bash
mkdir -p src/{models,data,experiments,utils} tests configs logs results data/{raw,processed}
```

Skip existing directories silently.

### Step 3: GPU Verification (always check, even without `--gpu`)

```python
import torch
print(f"CUDA: {torch.cuda.is_available()}, GPUs: {torch.cuda.device_count()}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}, Memory: {torch.cuda.get_device_properties(0).total_memory/1e9:.1f}GB")
```

Report hardware. If GPU required but absent: warn but continue — CPU debugging is still possible.

### Step 4: Dataset Download (skip with `--skip-data`)

For each dataset in `DATA_AND_EVAL.md → Dataset Download Commands`:
1. Check if already downloaded (skip if file exists and size is correct)
2. Download with progress
3. Verify checksum or file size
4. Extract if compressed

On download failure: try alternative sources from `DATA_AND_EVAL.md`. If no alternative: set `blocking_error` in `LOOP_STATE.md`.

### Step 5: Smoke Test

```python
import sys; sys.path.insert(0, 'src')
# Import each main module listed in PROJECT_STRUCTURE.md
# Instantiate model with minimal config
# Run forward pass on random data
```

On import failure: module not yet implemented (needs `/reproduce`) or has a syntax error (needs fix).

### Step 6: Report

```
Environment Setup Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Python:    3.10.12
 PyTorch:   2.1.0+cu118 | CUDA 11.8 | A100 80GB
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Datasets:
   ✓ ImageNet    (1.28M train / 50K val)
   ✗ CIFAR-100   not found → run: python data/download.py --cifar100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Smoke test: 3/4 imports OK | models.attention MISSING (not yet implemented)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: PARTIAL — /reproduce will implement missing modules
```

Update `LOOP_STATE.md` environment section with setup result.
