---
name: paper-parse
description: Read a research paper and extract key implementation information — target metrics, model architecture, datasets, and hyperparameters. Outputs ready-to-paste YAML blocks for DATA_AND_EVAL.md Paper Target Metrics section.
when_to_use: Use before Stage 1 (documentation generation) to prepare rich structured input, or to verify that generated Paper Target Metrics match the paper's actual reported numbers.
argument-hint: "[--file path/to/paper.pdf] [--focus metrics|arch|data|hyper|all]"
allowed-tools:
  - Read
  - Bash
  - Write
---

# Parse Research Paper

**Arguments**: $ARGUMENTS

## Instructions

Parse `$ARGUMENTS`:
- `--file <path>` → read from this file (PDF, txt, or md). If not provided: expect paper content in the conversation.
- `--focus metrics` → extract quantitative results only (default)
- `--focus arch` → model architecture details
- `--focus data` → dataset specs and download info
- `--focus hyper` → hyperparameter values from paper
- `--focus all` → all four extractions

This skill is for **pre-Stage-1** use. Output goes to `paper_parse_output.md` for human review, NOT read automatically by the loop.

---

### `--focus metrics` (default)

Extract all quantitative results from the paper's main tables and figures:

- Identify all Tables containing results (Table 1, Table 2, ...)
- For each result row labeled "Ours", "Proposed", or the paper's method name: extract every numeric column
- For ablation tables: extract baseline row AND each ablation variant
- Note which metric is the primary one (usually highlighted or in the abstract)

Output as ready-to-paste YAML:

```yaml
## Paper Target Metrics
# Source: "[Paper Title]", [Venue Year]
# Note: copy this block into DATA_AND_EVAL.md
primary_metrics:
  - name: accuracy
    value: 0.847
    higher_is_better: true
    source: "Table 2, row 'Ours (full)', column 'Test Acc.'"
    notes: "Averaged over 3 seeds per footnote 2"
    eval_command: "python eval.py --metric accuracy"

secondary_metrics:
  - name: params_millions
    value: 85.3
    higher_is_better: false
    source: "Table 3, Model Size column"

ablation_targets:
  - name: accuracy_without_attention
    value: 0.801
    source: "Table 4, row '-Attention'"
```

### `--focus arch`

Extract model architecture structured notes:
- Layer types, count, and dimensions
- Attention mechanism (heads, key_dim, type)
- Normalization (pre-norm vs post-norm, BN vs LN)
- Activation functions
- Positional encoding
- Output head design
- Any unusual or novel architectural choices
Note the paper section and figure for each detail.

### `--focus data`

Extract dataset information:
- Dataset name, version, and official source URL
- Download method (HuggingFace, official site, script)
- Train/val/test split sizes
- Preprocessing described in paper
- Data augmentation used during training vs evaluation
- Any dataset-specific preprocessing not commonly documented

### `--focus hyper`

Extract hyperparameter table:

```yaml
hyperparameters:
  learning_rate: 3.0e-4         # Table X / Section Y
  lr_schedule: "cosine, 500-step warmup"
  batch_size: 256
  epochs: 100
  optimizer: "AdamW (β1=0.9, β2=0.999, ε=1e-8)"
  weight_decay: 0.01
  gradient_clip: 1.0
  hidden_dim: 768
  num_layers: 12
  num_heads: 12
  dropout: 0.1
  seed: 42   # or "not specified"
  fp16: true
```

---

## Output

Writes to `paper_parse_output.md`. Print summary:

```
Paper Parse Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Focus: all | Source: paper.pdf

 Extracted:
   Primary metrics:    3 (ready for DATA_AND_EVAL.md)
   Ablation targets:   6
   Hyperparameters:    18 (cross-check configs/paper_config.yaml)
   Architecture notes: Table 1, Figure 2, Section 3.2
   Dataset sources:    2 URLs found

 Output: paper_parse_output.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next:
  Copy Paper Target Metrics block → DATA_AND_EVAL.md
  Cross-check hyperparameters → configs/paper_config.yaml
```
