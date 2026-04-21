# Effort Contract

All skills in this framework respect the `--effort` parameter. This contract defines the behavior at each level.

## Levels

### `lite`
```yaml
max_iterations: 5
tolerance: 0.15         # 15% gap from paper target
coverage: primary_only  # only primary metrics from DATA_AND_EVAL.md
ablations: false
assurance_gates: false
```
**Use when**: Quick sanity check — does the implementation basically work?

### `balanced` (default)
```yaml
max_iterations: 10
tolerance: 0.10         # 10% gap
coverage: all_primary   # all primary metrics
ablations: false
assurance_gates: false
```
**Use when**: Standard paper reproduction — does it reproduce main results?

### `max`
```yaml
max_iterations: 20
tolerance: 0.05         # 5% gap
coverage: primary_and_secondary
ablations: true         # if paper has ablation table
assurance_gates: false
```
**Use when**: High-fidelity replication — closely match all reported numbers.

### `beast`
```yaml
max_iterations: 50
tolerance: 0.02         # 2% gap
coverage: all           # primary + secondary + ablations
ablations: true
assurance_gates: true   # metric-audit, code-audit required
statistical_tests: true # if paper reports significance tests
figure_matching: true   # training curves qualitatively match paper
```
**Use when**: Publication-quality reproduction — every table and figure matched.

---

## Assurance Gates (beast only)

At `--effort beast`, two audits are mandatory before declaring success:

### metric-audit
- Verify every metric formula matches paper's exact definition
- Check evaluation dataset matches paper's specification
- Confirm no data leakage between train/test sets

### code-audit
- Verify no hardcoded dataset-specific hacks
- Confirm hyperparameters come from config (not hardcoded)
- Check random seed management is complete and consistent

Success is only declared after both audits pass.

---

## Applying Effort

Skills read effort from `LOOP_STATE.md`. To change mid-run:

1. Edit `LOOP_STATE.md` → update `effort:` and derived parameters
2. Run `/reproduce` — it will pick up the new settings
