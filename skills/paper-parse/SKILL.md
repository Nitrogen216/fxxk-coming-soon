# /paper-parse — Extract Key Implementation Info from Paper

**Invocation**: `/paper-parse [--file path/to/paper.pdf|paper.md|paper.txt] [--focus metrics|arch|data|hyper|all]`

---

## Purpose

Read a research paper and extract the specific information needed to either:
1. Fill in `PAPER_TO_CODE_PROMPT.md` intelligently before documentation generation (Stage 0)
2. Manually verify or supplement the generated `DATA_AND_EVAL.md`, specifically the `## Paper Target Metrics` YAML block

Think of this as a structured paper reading skill focused entirely on reproducibility: what numbers need to be hit, what data is needed, what architecture is used.

---

## Input

Provide the paper as:
- A local PDF: `/paper-parse --file arxiv_paper.pdf`
- A plain-text extraction: `/paper-parse --file paper.txt`
- Markdown copy-paste: just include the paper content in your message with `/paper-parse`

---

## Extraction Targets

### `--focus metrics` (default)

Extract all reported quantitative results:

- Main results tables (Table 1, Table 2, ...)
- Ablation study tables
- Figures showing performance curves (describe key values)
- Comparison with baselines

Output format — ready to paste into `DATA_AND_EVAL.md → Paper Target Metrics`:

```yaml
## Paper Target Metrics
# Source: [Paper Title], [Venue Year]
primary_metrics:
  - name: accuracy
    value: 0.847
    higher_is_better: true
    source: "Table 2, row 'Ours (full model)', column 'Test Acc.'"
    notes: "Averaged over 3 seeds per paper footnote 2"

  - name: f1_score
    value: 0.762
    higher_is_better: true
    source: "Table 2, row 'Ours (full model)', column 'F1'"

secondary_metrics:
  - name: params_millions
    value: 85.3
    higher_is_better: false
    source: "Table 3, Model Size column"

ablation_targets:
  - name: accuracy_without_attention
    value: 0.801
    source: "Table 4, ablation row '-Attention'"
```

### `--focus arch`

Extract model architecture details:
- Layer types, count, dimensions
- Activation functions
- Normalization strategy
- Attention mechanism details (heads, dim, type)
- Positional encoding
- Any unusual architectural choices

Output as structured notes referencing specific paper sections/figures.

### `--focus data`

Extract dataset information:
- Dataset name, version, official source URL
- Download method (HuggingFace, official site, script)
- Split sizes (train/val/test)
- Any preprocessing described in paper
- Data augmentation used during training

### `--focus hyper`

Extract all hyperparameters reported in the paper:

```yaml
hyperparameters:
  # Training
  learning_rate: 3.0e-4
  lr_schedule: "cosine with 500-step warmup"
  batch_size: 256
  epochs: 100
  optimizer: "AdamW (β1=0.9, β2=0.999, ε=1e-8)"
  weight_decay: 0.01
  gradient_clip: 1.0

  # Model
  hidden_dim: 768
  num_layers: 12
  num_heads: 12
  dropout: 0.1
  ff_dim: 3072

  # Training details
  seed: 42           # "if specified"
  fp16: true
  gradient_accumulation: 4
```

### `--focus all`

Run all four extractions in sequence.

---

## Output

Writes results to `paper_parse_output.md` in the project directory. This file is for human reference and manual transfer — it is NOT read automatically by the loop agent.

```
Paper Parse Complete
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Paper:  "Attention Is All You Need" (example)
 Focus:  all

 Extracted:
   Primary metrics:    3
   Ablation targets:   6
   Hyperparameters:    18
   Dataset sources:    2
   Architecture notes: 4 sections

 Output: paper_parse_output.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Next steps:
  1. Copy Paper Target Metrics block → DATA_AND_EVAL.md
  2. Cross-check hyperparameters → configs/paper_config.yaml
  3. Verify dataset URLs → DATA_AND_EVAL.md download commands
```

---

## When to Use

| Situation | Action |
|-----------|--------|
| Before Stage 1 (doc generation) | Run with `--focus all` to prepare rich input for `PAPER_TO_CODE_PROMPT.md` |
| After Stage 1, verifying targets | Run with `--focus metrics` to double-check YAML block |
| Loop is stuck, suspecting wrong target | Run `--focus metrics` to re-verify the number you're targeting |
| Architecture mismatch suspected | Run `--focus arch` to compare paper vs code |
