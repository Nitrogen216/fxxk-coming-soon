# /check-impl — Verify Implementation Against Paper Documentation

**Invocation**: `/check-impl [--section all|structure|plan|eval] [--strict]`

---

## Purpose

Systematically audit the current codebase against all three implementation specification files — `PROJECT_STRUCTURE.md`, `IMPLEMENTATION_PLAN.md`, and `DATA_AND_EVAL.md` — to find gaps, mismatches, and deviations from the paper's documented requirements.

Think of this as a "paper compliance check." Run it before a long training run, or when the loop is stuck and you suspect a silent implementation error.

---

## Execution

### Section: Structure (`--section structure`)

Verify the codebase matches `PROJECT_STRUCTURE.md`:

1. **Directory tree check** — all directories in the documented tree exist
2. **File existence** — all documented source files exist
3. **Function signatures** — for each documented public function, verify:
   - Function exists in the specified file
   - Parameter names and count match
   - Return type annotation matches (if typed)
4. **Class hierarchy** — documented inheritance relationships are implemented
5. **Module imports** — each module can be imported without error

Report per-file:
```
src/models/attention.py
  ✓ MultiHeadAttention class exists
  ✓ forward(q, k, v, mask=None) signature matches
  ✗ scaled_dot_product() missing  — documented in PROJECT_STRUCTURE.md:L47
  ✓ imports: torch, torch.nn
```

### Section: Plan (`--section plan`)

Verify implementation follows `IMPLEMENTATION_PLAN.md`:

1. **Phase completion** — for each phase (Infrastructure/Data/Model/Eval), check if all documented components are implemented
2. **Validation checkpoints** — run each documented checkpoint test
3. **Integration points** — verify data flows correctly between modules (can instantiate pipeline end-to-end)
4. **Configuration system** — all hyperparameters in `configs/paper_config.yaml` match documented schema

Run a dry-forward pass:
```python
config = load_config('configs/paper_config.yaml')
dataset = Dataset(config, split='train')
batch = next(iter(DataLoader(dataset, batch_size=2)))
model = Model(config)
output = model(batch['input'])
loss = compute_loss(output, batch['target'], config)
print(f"Forward pass OK: loss={loss.item():.4f}")
```

### Section: Eval (`--section eval`)

Verify evaluation setup against `DATA_AND_EVAL.md`:

1. **Metric formulas** — for each primary metric, verify the implementation matches the documented formula
2. **Paper Target Metrics YAML** — verify `LOOP_STATE.md` targets are populated from this section
3. **Evaluation dataset** — verify test split matches paper's specification (size, source, preprocessing)
4. **Random seeds** — verify seeds are set in all required places (Python, NumPy, PyTorch, CUDA)
5. **Evaluation script** — `python eval.py --help` runs without error

---

## Strict Mode (`--strict`)

In addition to the above:
- Check every hyperparameter in `configs/paper_config.yaml` against the paper's reported values (from `DATA_AND_EVAL.md → Experimental Protocols`)
- Flag any value that differs from the paper by more than 5%
- Verify model parameter count matches paper's reported size (if stated)

---

## Output

```
Implementation Check — 2024-01-15 | Strict: OFF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Section: Structure
  Files:       14/15 exist  (missing: src/models/scheduler.py)
  Functions:   31/34 match  (missing: 3 in attention.py)
  Classes:     8/8 exist
  Imports:     13/14 OK     (data/augment.py import error)

Section: Plan
  Phase 1 (Infra):    ✓ complete
  Phase 2 (Data):     ✓ complete
  Phase 3 (Model):    ⚠ partial  — attention.py has 3 missing functions
  Phase 4 (Eval):     ✓ complete
  Forward pass:       ✓ OK (loss=2.341)

Section: Eval
  Metrics:         3/3 formulas match
  Targets loaded:  ✓ (2 primary metrics in LOOP_STATE.md)
  Test split:      ✓ 10,000 samples (matches paper)
  Seeds set:       ✓ python/numpy/torch/cuda

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Issues Found: 2 blocking, 1 warning

BLOCKING:
  1. src/models/attention.py: scaled_dot_product(), relative_position_bias(),
     apply_rope() missing — documented in PROJECT_STRUCTURE.md:L44-52
  2. src/data/augment.py: ImportError on albumentations — add to requirements.txt

WARNING:
  1. configs/paper_config.yaml: lr_scheduler not configured — paper uses cosine

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Recommendation: Fix 2 blocking issues before running experiment.
  Add issues to LOOP_STATE.md? [auto-added ✓]
```

Blocking issues are automatically added to `LOOP_STATE.md → outstanding_issues` with `priority: high`.
