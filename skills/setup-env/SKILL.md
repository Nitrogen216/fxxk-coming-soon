# /setup-env — Environment Setup & Dataset Download

**Invocation**: `/setup-env [--python 3.10|3.11|3.12] [--gpu] [--skip-data]`

---

## Purpose

Initialize the complete runtime environment for a paper reproduction project. This skill is idempotent — safe to run multiple times. Use it at the start of a project, or whenever the environment is corrupted and needs rebuilding.

The loop agent runs this automatically in PHASE 0 (first iteration). Call it manually when:
- Setting up a new machine or container
- Dependencies got corrupted or version-conflicted
- Datasets are missing or incomplete
- You want to verify environment integrity before a long run

---

## Execution

### Step 1: Python Environment

```bash
# Create virtualenv with specified Python version (default: system Python 3)
python3 -m venv venv
source venv/bin/activate

# Verify Python version matches requirements
python --version
```

If `requirements.txt` doesn't exist, create a minimal one from `PROJECT_STRUCTURE.md → Technology Stack`.

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

On version conflict: check `RISKS_AND_NOTES.md → Library compatibility concerns`. Try installing packages one by one to isolate the conflict.

### Step 2: GPU Verification (`--gpu` flag or auto-detected)

```python
import torch
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"GPU count: {torch.cuda.device_count()}")
if torch.cuda.is_available():
    print(f"GPU name: {torch.cuda.get_device_name(0)}")
    print(f"GPU memory: {torch.cuda.get_device_properties(0).total_memory / 1e9:.1f} GB")
```

If GPU required but unavailable: check `DATA_AND_EVAL.md → Hardware requirements`. Report hardware mismatch but continue — CPU fallback may be possible for debugging.

### Step 3: Directory Structure

Create all directories defined in `PROJECT_STRUCTURE.md → Repository Tree`:

```bash
mkdir -p src/{models,data,experiments,utils}
mkdir -p tests configs logs results data/raw data/processed
```

### Step 4: Dataset Download (`--skip-data` to bypass)

Execute each command from `DATA_AND_EVAL.md → Dataset Download Commands`:

```bash
# Example — actual commands come from DATA_AND_EVAL.md
wget -O data/raw/dataset.tar.gz "https://..."
tar -xzf data/raw/dataset.tar.gz -C data/raw/
```

For each dataset:
1. Check if already downloaded (skip if file exists and size matches)
2. Download with progress display
3. Verify integrity (checksum if provided, else file size)
4. Extract if compressed

On download failure: log the URL and error. Try alternative sources from `DATA_AND_EVAL.md`. If no alternative: mark as `blocking_error` in `LOOP_STATE.md`.

### Step 5: Smoke Test

```python
# Verify all key imports work
import sys
sys.path.insert(0, 'src')

# Import each main module from PROJECT_STRUCTURE.md
from models.model import Model
from data.dataset import Dataset
from experiments.trainer import Trainer
print("All imports OK")

# Instantiate core classes with minimal config
model = Model(config={'hidden_dim': 32, 'num_layers': 1})
print(f"Model params: {sum(p.numel() for p in model.parameters()):,}")
```

On import failure: the module doesn't exist yet (needs implementation) or has a syntax error. Report which module and stop.

### Step 6: Environment Report

Print a summary:

```
Environment Setup Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Python:     3.10.12
 PyTorch:    2.1.0+cu118
 CUDA:       11.8  |  GPU: A100 80GB
 Key libs:   transformers 4.35.2, numpy 1.24.3
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Datasets:
   ImageNet     ✓  (1.28M train / 50K val)
   ✗ CIFAR-100  not found — run: python data/download.py --cifar100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Modules:     3/4 imports OK  |  models.attention MISSING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: PARTIAL — run /reproduce to implement missing modules
```

Update `LOOP_STATE.md` with environment status.
