# Autonomous Paper Reproduction Loop

**AGENT INSTRUCTIONS**: This is your primary control document. Read it fully before starting any work. Your mission is to autonomously reproduce the key experimental results from the research paper documented in this project.

> **Philosophy**: You run until success or resources are exhausted. Do NOT pause to ask the human whether to continue — the loop runs until you succeed, hit max_iterations, or encounter an unrecoverable error.

---

## Required Input Files

Verify all of these exist before starting. If any are missing, report which ones and stop.

| File | Purpose |
|------|---------|
| `PROJECT_STRUCTURE.md` | Architecture blueprint — module specs, function signatures |
| `IMPLEMENTATION_PLAN.md` | Phase-by-phase implementation roadmap |
| `DATA_AND_EVAL.md` | Target metrics, data pipelines, evaluation procedures |
| `RISKS_AND_NOTES.md` | Known pitfalls, debugging strategies, assumption ladder |
| `CLAIMS_AND_GATES.md` | Paper claims + milestone gate definitions (M0-M4) *(if missing: derive M0-M2 from DATA_AND_EVAL.md)* |
| `LOOP_STATE.md` | Loop state tracker (create from template if missing) |

---

## Loop Control Parameters

Read from `LOOP_STATE.md`. If not set, use these defaults:

```yaml
effort: balanced        # lite | balanced | max | beast
max_iterations: 10
tolerance: 0.10         # 10% — within this gap of paper target = success
reviewer: none          # none | codex | gpt (cross-model code review)
gpu_hour_limit: 4       # max GPU-hours per iteration before flagging
```

**Effort Presets** (overrides individual parameters):

| Effort | max_iter | tolerance | Coverage | Use Case |
|--------|----------|-----------|----------|----------|
| `lite` | 5 | 15% | Primary metrics only | Quick sanity check |
| `balanced` | 10 | 10% | All primary metrics | Standard reproduction |
| `max` | 20 | 5% | Primary + secondary | High-fidelity replication |
| `beast` | 50 | 2% | All metrics + ablations | Publication-quality |

---

## The Autonomous Reproduction Loop

### ── INITIALIZE ──────────────────────────────────────────────────

Run once per session, before entering the loop:

1. Read `LOOP_STATE.md` — if it doesn't exist, create it from the template
2. Read `DATA_AND_EVAL.md` → **`## Paper Target Metrics`** — load all targets
3. Read `CLAIMS_AND_GATES.md` → load milestone gate definitions (M0–M4)
   - If missing: synthesize M0 (dry-run), M1 (baseline metrics), M2 (primary metrics) from DATA_AND_EVAL.md
4. Determine current milestone from `LOOP_STATE.md → current_milestone`
5. Check stopping conditions:
   - `status == "success"` → print results table and stop
   - `iteration_count >= max_iterations` → write PROGRESS_REPORT.md and stop
   - `status == "blocked"` → print blocking issue and stop
6. Otherwise: enter the loop at PHASE 0 (if `iteration_count == 0`) or PHASE 1

---

### ── PHASE 0: Environment Setup ─────────────────────────────────
*Run only when `iteration_count == 0`*

```bash
# Create virtualenv
python -m venv venv && source venv/bin/activate

# Install dependencies (exact versions from requirements.txt)
pip install -r requirements.txt

# Verify key imports
python -c "import <core_library>; print('OK')"
```

Steps:
1. Create the full directory structure defined in `PROJECT_STRUCTURE.md`
2. Set up virtual environment and install all dependencies
3. Download datasets per `DATA_AND_EVAL.md` → **Dataset Download Commands** section
4. Verify dataset integrity (file sizes, record counts)
5. Set random seeds everywhere: Python `random`, `numpy`, `torch` (see `DATA_AND_EVAL.md`)
6. Run a smoke test: import all modules, instantiate core classes
7. Update `LOOP_STATE.md`: `status = "running"`, `iteration_count = 1`

**On environment failure**: log error details to `LOOP_STATE.md` under `blocking_error`, set `status = "blocked"`, stop.

---

### ── PHASE 1: Implement / Improve ──────────────────────────────

**First iteration** (`iteration_count == 1`):
- Implement the full codebase following `IMPLEMENTATION_PLAN.md` phase order:
  - Phase 1: Core Infrastructure (config, logging, utilities)
  - Phase 2: Data Pipeline (loaders, preprocessing, validation)
  - Phase 3: Model Implementation (architecture, training loop)
  - Phase 4: Evaluation Framework (metrics, reporting)
- After each phase: run its validation checkpoint before proceeding

**Milestone-aware implementation** (`iteration_count > 1`):

Before choosing what to implement, check `LOOP_STATE.md → current_milestone`:
- **M0 → M1 (baseline)**: Implement and evaluate the baseline first. Do NOT touch the proposed method code yet. The baseline must pass before method reproduction begins.
- **M1 → M2 (method)**: Baseline is confirmed. Now implement the proposed method's key components.
- **M2 → M3 (ablation)**: Only when `effort >= max`. Implement ablation variants.
- **M2/M3 → M4 (review)**: Only when `effort == beast` or `reviewer != none`. Run `/handoff-review`.

Then apply the top fix from `outstanding_issues` that matches the current milestone:
- Do NOT rewrite working code — surgical, minimal changes only
- Reference `RISKS_AND_NOTES.md → assumption_ladder` for ambiguous details
- Log each change immediately to `LOOP_STATE.md → fixes_applied`

**If `reviewer != none`** (cross-model review):
- After implementation, prepare a code review summary (key changes, concerns)
- Submit to reviewer model; wait for structured feedback
- Incorporate feedback before proceeding

---

### ── PHASE 2: Execute ────────────────────────────────────────────

Run the main experiment pipeline:

```bash
# Main entry point (from IMPLEMENTATION_PLAN.md)
python main.py --config configs/paper_config.yaml 2>&1 | tee logs/iter_${N}.log
```

Rules:
- Save all output to `logs/iter_${N}.log` (where N = current iteration)
- If execution crashes, classify the error type:
  - `dependency_error`: missing package or version mismatch
  - `logic_error`: assertion failure, shape mismatch, NaN
  - `data_error`: file not found, format mismatch
  - `oom_error`: out-of-memory on GPU/CPU
- Attempt automatic fix for known error types; retry up to **3 times**
- If still failing after 3 retries: log to `LOOP_STATE.md` as `execution_failed`, proceed to PHASE 3 with partial results
- If estimated runtime exceeds `gpu_hour_limit`: pause, log a warning, and proceed with partial results

---

### ── PHASE 3: Evaluate ──────────────────────────────────────────

Extract and compare metrics:

1. Parse `logs/iter_${N}.log` for metric values using patterns from `DATA_AND_EVAL.md`
2. Also run the evaluation script if one exists: `python eval.py --results results/iter_${N}/`
3. For **each metric** M listed in `DATA_AND_EVAL.md → Paper Target Metrics`:

```
gap_M = |achieved_M - target_M| / |target_M|
pass_M = (gap_M <= tolerance)
```

4. Update `LOOP_STATE.md → latest_metrics` with all results
5. Save full evaluation output to `results/iter_${N}/eval_summary.json`

---

### ── PHASE 3.5: Plateau Check ───────────────────────────────────

After every evaluation, check stagnation on the worst-failing metric:

```
If the metric improved < 1% (abs) for the last 3 consecutive iterations:
  → Set plateau_detected: true in LOOP_STATE.md
  → Classify plateau type:
      A: monotone but slowing  → reduce LR by 10×
      B: oscillating           → reduce LR + gradient clipping
      C: hard ceiling          → /debug-gap --depth deep, re-examine metric formula
      D: random walk           → fix all random seeds, reduce data stochasticity
  → Apply type-specific escalation INSTEAD OF the usual top-priority fix
```

If plateau detected AND used ≥ 70% of iteration budget: proactively write `PROGRESS_REPORT.md` snapshot and suggest `/extend-loop` or strategy change — but continue unless max_iterations reached.

---

### ── PHASE 4: Diagnose & Decide ─────────────────────────────────

**Check milestone gates first** (from `CLAIMS_AND_GATES.md`):

- If current milestone is M0: verify `python main.py --dry-run` exits 0 → advance to M1
- If current milestone is M1 (baseline): check baseline metrics pass → advance to M2
- If current milestone is M2 (method): check all primary metrics pass → SUCCESS
- If effort ≥ max and M2 passed: check ablation targets → advance to M3
- If effort == beast and M3 passed: run `/handoff-review` → M4

**Case A: Current milestone gate passes** → `ADVANCE MILESTONE`

Update `LOOP_STATE.md → current_milestone`. If M2 passes → SUCCESS.

**Case B: M2 metric ALL pass** → `SUCCESS`

```
1. Write REPRODUCTION_REPORT.md
2. Update LOOP_STATE.md: status = "success", current_milestone: "M2"
3. If effort >= max: check M3 ablations before stopping
4. Print success summary and stop
```

**Case C: `iteration_count >= max_iterations`** → `TIMEOUT`

```
1. Write PROGRESS_REPORT.md: best milestone reached, remaining gaps
2. Update LOOP_STATE.md: status = "timeout"
3. Print: "Best milestone: M[N]. Best metrics: [table]. Remaining gaps: [list]"
4. Suggest: /extend-loop --add-iterations 5  OR  /check-plateau
5. Stop the loop
```

**Case D: Metrics fail, plateau detected** → `ESCALATE`

Apply plateau-specific strategy (see PHASE 3.5). Continue loop.

**Case E: Metrics fail, no plateau, iterations remain** → `DIAGNOSE AND CONTINUE`

| Check | What to Look For |
|-------|-----------------|
| **Milestone order** | M1 (baseline) not confirmed before attempting M2? |
| Data integrity | Wrong split, incorrect preprocessing, label errors |
| Architecture | Missing layer, wrong activation, wrong dimensions |
| Hyperparameters | Any value differs from `DATA_AND_EVAL.md → Experimental Protocols` |
| Evaluation code | Wrong metric formula (macro vs micro), wrong test split |
| Assumption ladder | Which assumption in `CLAIMS_AND_GATES.md` might be wrong? |
| Numerical stability | NaN/Inf in gradients, loss explosion |

Generate ordered hypothesis list. Update `outstanding_issues`. Increment `iteration_count`. Return to **PHASE 1**.

---

## Output Files

| File | Created When | Content |
|------|-------------|---------|
| `logs/iter_N.log` | Every iteration | Raw execution stdout/stderr |
| `results/iter_N/` | Every iteration | Model outputs, checkpoints, raw metrics |
| `LOOP_STATE.md` | Every iteration | Updated loop state and diagnostics |
| `REPRODUCTION_REPORT.md` | On SUCCESS | Final comparison table, methodology notes |
| `PROGRESS_REPORT.md` | On TIMEOUT | Best results, remaining gaps, next steps |

### REPRODUCTION_REPORT.md Format

```markdown
# Reproduction Report

**Paper**: [Paper title from PROJECT_STRUCTURE.md]
**Status**: SUCCESS
**Iterations**: N
**Effort**: balanced

## Results

| Metric | Paper Target | Achieved | Gap | Status |
|--------|-------------|----------|-----|--------|
| accuracy | 84.7% | 85.1% | 0.5% | ✓ PASS |
| f1_score | 76.2% | 75.8% | 0.5% | ✓ PASS |

## Key Implementation Notes

[Any important deviations or clarifications from the paper]

## Iteration History

[Brief log of what changed each iteration]
```

---

## LOOP_STATE.md Schema

```yaml
# ── Control ───────────────────────────────────────
effort: balanced            # lite | balanced | max | beast
max_iterations: 10
tolerance: 0.10
reviewer: none
gpu_hour_limit: 4

# ── Current Status ────────────────────────────────
iteration_count: 3
status: running             # not_started | running | success | timeout | blocked | interrupted
blocking_error: null
last_updated: "2024-01-15T14:23:00Z"

# ── Milestone Progress ────────────────────────────
current_milestone: M2       # M0 | M1 | M2 | M3 | M4
milestones:
  M0_sanity:
    status: passed
    passed_at_iteration: 0
  M1_baseline:
    status: passed
    passed_at_iteration: 2
    baseline_targets: {accuracy: 0.762, f1_score: 0.701}
  M2_method:
    status: pending
    passed_at_iteration: null
  M3_ablation:
    status: pending         # only checked if effort >= max
  M4_review:
    status: pending         # only checked if effort == beast or reviewer != none
    reviewer_score: null
    reviewer_verdict: null  # ready | almost | not_ready
    reviewer_blockers: []

# ── Latest Metrics ────────────────────────────────
latest_metrics:
  accuracy:
    target: 0.847
    achieved: 0.821
    gap: "3.1%"
    status: fail
  f1_score:
    target: 0.762
    achieved: 0.701
    gap: "8.0%"
    status: fail

# ── Plateau Detection ─────────────────────────────
plateau_detected: false
plateau_metric: null
plateau_type: null          # A: asymptotic | B: oscillating | C: ceiling | D: random
plateau_since_iteration: null
plateau_escalation_applied: null

# ── Iteration History ─────────────────────────────
iteration_history:
  - iteration: 1
    milestone: M1
    achieved: {accuracy: 0.751}
    issues_found: ["data normalization computed globally, not per-channel"]
    fixes_applied: ["fixed mean/std to per-channel in data/preprocessing.py"]
  - iteration: 2
    milestone: M1
    achieved: {accuracy: 0.803}
    issues_found: ["learning rate too high, loss oscillating"]
    fixes_applied: ["reduced lr 1e-3→3e-4, added 100-step linear warmup"]

# ── Outstanding Issues (priority-ordered) ─────────
outstanding_issues:
  - priority: high
    metric: f1_score
    milestone: M2
    gap: "8.0%"
    hypothesis: "Paper uses macro-F1; code uses micro"
    proposed_fix: "Change average='micro' to average='macro' in metrics.py:compute_f1"
    file_hint: "src/eval/metrics.py:42"
  - priority: medium
    metric: accuracy
    milestone: M2
    gap: "3.1%"
    hypothesis: "Weight init differs from paper (Xavier vs Kaiming)"
    proposed_fix: "Switch to nn.init.xavier_uniform_ per paper Section 3.2"
    file_hint: "src/models/model.py:_init_weights"

# ── Applied Fixes Log ─────────────────────────────
fixes_applied:
  - iteration: 1
    description: "Fixed per-channel data normalization"
    files_changed: ["src/data/preprocessing.py"]
    result: "accuracy 0.751 → 0.803 (+7.0%)"
  - iteration: 2
    description: "Reduced LR with warmup schedule"
    files_changed: ["configs/paper_config.yaml", "src/training/trainer.py"]
    result: "accuracy 0.803 → 0.821 (+2.2%)"
```

---

## Safety Limits

- Never run an experiment estimated to exceed `gpu_hour_limit` (default 4h) without logging a warning
- Always save the best checkpoint before starting a new iteration that modifies the model
- Never delete `results/` or `logs/` directories
- If you encounter data that looks like it might be corrupted: stop, log, report to user
- Verify dataset checksums before long training runs when checksums are available

---

## Skill Reference

All available as slash commands in `.claude/skills/`:

### Loop Control
```
/reproduce [--effort lite|balanced|max|beast] [--reviewer none|codex|gpt]
    Start or resume the full autonomous reproduction loop

/loop-once [--focus metric_name] [--phase implement|execute|evaluate]
    Run exactly one iteration with full control

/extend-loop [--add-iterations N] [--effort level] [--tighten-tolerance]
    Add more iterations to an exhausted (timeout) loop

/reset-loop [--keep-history] [--keep-code] [--hard]
    Reset loop state for a fresh run (preserves code by default)
```

### Status & Evaluation
```
/reproduce-status
    Quick status snapshot from LOOP_STATE.md

/evaluate [--verbose]
    Check current progress without running experiments

/write-report [--type success|progress|failed] [--compare-table]
    Generate formal reproduction report (auto-detected from loop state)
```

### Execution
```
/setup-env [--python 3.10|3.11|3.12] [--gpu] [--skip-data]
    Initialize environment, install dependencies, download datasets

/run-experiment [--config path] [--dry-run] [--resume] [--tag label]
    Execute the experiment pipeline with logging and error recovery

/save-checkpoint [--message "description"] [--tag label]
    Git-commit current state as a recoverable snapshot
```

### Diagnosis
```
/debug-gap [--metric name] [--depth shallow|deep]
    Systematically diagnose why a specific metric is failing

/check-impl [--section all|structure|plan|eval] [--strict]
    Audit implementation against all paper documentation files

/check-plateau [--metric name] [--window N] [--threshold 0.01]
    Detect stagnation and classify plateau type (A/B/C/D)
```

### Review & Paper Analysis
```
/handoff-review [--reviewer human|codex|gpt] [--receive]
    Prepare external review package; parse reviewer verdict into LOOP_STATE.md

/paper-parse [--file path] [--focus metrics|arch|data|hyper|all]
    Extract key implementation info from the paper before generating docs
```

---

**The loop runs until you succeed or exhaust max_iterations. No human confirmation needed. Go.**
