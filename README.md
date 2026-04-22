# fxxk-coming-soon

> Autonomous paper reproduction: from PDF to reproduced results, with a continuous agent loop that runs until it succeeds.

---

## What Is This?

Many research papers release GitHub repos with only a **"Coming Soon"** message — or no code at all. **fxxk-coming-soon** bridges that gap with a two-stage approach:

1. **Documentation Generation**: Use `PAPER_TO_CODE_PROMPT.md` with any capable LLM to convert a research paper into **5 structured `.md` files** — a complete implementation blueprint plus a reproduction control policy.
2. **Autonomous Reproduction Loop**: Drop those files + `AGENT_LOOP_PROMPT.md` into a new project and run `/reproduce`. Claude Code loops through **milestone gates** (M0→M1 baseline→M2 method) — implement → execute → evaluate → diagnose → improve — until metrics are reproduced or the budget runs out.

---

## Repository Structure

```
fxxk-coming-soon/
│
├── PAPER_TO_CODE_PROMPT.md    # Stage 1: master LLM prompt → 4 doc files
├── AGENT_LOOP_PROMPT.md       # Stage 2: master loop controller for Claude Code
│
├── skills/                    # Claude Code slash commands (copy to target project)
│   │
│   │   # Loop Control
│   ├── reproduce/SKILL.md         # /reproduce — start the full loop
│   ├── loop-once/SKILL.md         # /loop-once — single iteration, manual control
│   ├── extend-loop/SKILL.md       # /extend-loop — add iterations to exhausted loop
│   ├── reset-loop/SKILL.md        # /reset-loop — fresh start (preserves code)
│   │
│   │   # Status & Evaluation
│   ├── reproduce-status/SKILL.md  # /reproduce-status — quick snapshot
│   ├── evaluate/SKILL.md          # /evaluate — check metrics vs targets
│   ├── write-report/SKILL.md      # /write-report — generate formal report
│   │
│   │   # Execution
│   ├── setup-env/SKILL.md         # /setup-env — init environment + download data
│   ├── run-experiment/SKILL.md    # /run-experiment — execute with logging
│   ├── save-checkpoint/SKILL.md   # /save-checkpoint — git snapshot
│   │
│   │   # Diagnosis
│   ├── debug-gap/SKILL.md         # /debug-gap — diagnose metric failures
│   ├── check-impl/SKILL.md        # /check-impl — audit code vs paper docs
│   │
│   │   # Paper Analysis
│   ├── paper-parse/SKILL.md       # /paper-parse — extract info from paper
│   │
│   └── shared/
│       ├── effort-contract.md     # effort level definitions (lite/balanced/max/beast)
│       └── loop-contract.md       # invariants all loop skills must honor
│
├── templates/                 # Templates for target paper reproduction projects
│   ├── LOOP_STATE.md              # Loop state tracker (copy to target project)
│   ├── CLAIMS_AND_GATES.md        # Paper claims + milestone gate template (NEW)
│   ├── project-CLAUDE.md          # CLAUDE.md template for target projects
│   ├── PROJECT_STRUCTURE.md       # Architecture blueprint template
│   ├── IMPLEMENTATION_PLAN.md     # Implementation plan template
│   ├── DATA_AND_EVAL.md           # Data & evaluation template
│   └── RISKS_AND_NOTES.md         # Risks & notes template (with assumption ladder)
│
└── README.md
```

---

## The Complete Workflow

### Stage 1: Generate Documentation (5-10 minutes)

1. Extract the paper's methodology sections (abstract, method, experiments)
2. Open `PAPER_TO_CODE_PROMPT.md` and paste paper content into `{{PAPER_CONTENT}}`
3. Submit to a capable LLM (Claude 4, GPT-5, Grok 4)
4. Save the **5 generated files**:
   - `PROJECT_STRUCTURE.md` — architecture blueprint with exact function signatures
   - `IMPLEMENTATION_PLAN.md` — phase-by-phase implementation roadmap
   - `DATA_AND_EVAL.md` — data pipelines + **machine-readable target metrics YAML**
   - `RISKS_AND_NOTES.md` — known challenges, debugging strategies, assumption ladder
   - `CLAIMS_AND_GATES.md` — **paper claims frozen + M0-M4 milestone gate definitions**

### Stage 2: Autonomous Reproduction Loop (runs until success)

Create a new project folder with the 4 generated files, then:

```bash
# Copy loop infrastructure from fxxk-coming-soon
cp /path/to/fxxk-coming-soon/AGENT_LOOP_PROMPT.md .
cp /path/to/fxxk-coming-soon/templates/LOOP_STATE.md .
cp /path/to/fxxk-coming-soon/templates/project-CLAUDE.md ./CLAUDE.md
mkdir -p .claude/commands
cp -r /path/to/fxxk-coming-soon/skills/* .claude/commands/

# Launch Claude Code and start the loop
claude
> /reproduce
```

Claude Code will run autonomously:

```
INITIALIZE → read docs, load CLAIMS_AND_GATES.md, check milestone state
     ↓
PHASE 0: Environment setup  [first iteration only → M0 sanity gate]
     ↓
PHASE 1: Implement / Improve  [M1 baseline first, then M2 method]
     ↓  (baseline code until M1 passes; method code after M1 confirmed)
PHASE 2: Execute
     ↓  (run experiment, save logs/iter_N.log)
PHASE 3: Evaluate
     ↓  (compare metrics vs targets, check milestone gate)
PHASE 3.5: Plateau Check
     ↓  (detect stagnation, classify type A/B/C/D, escalate if needed)
PHASE 4: Decide
     ├── M2 gate passes → REPRODUCTION_REPORT.md → DONE ✓
     ├── MAX ITER → PROGRESS_REPORT.md → DONE ⏱
     ├── PLATEAU → escalate strategy → back to PHASE 1
     └── FAIL → diagnose → update LOOP_STATE.md → back to PHASE 1
```

Monitor `LOOP_STATE.md` to watch progress. The loop runs without human confirmation.

---

## Effort Levels

Control depth and rigor with `--effort`:

| Level | Max Iterations | Tolerance | Coverage |
|-------|---------------|-----------|----------|
| `lite` | 5 | 15% | Primary metrics only |
| `balanced` *(default)* | 10 | 10% | All primary metrics |
| `max` | 20 | 5% | Primary + secondary + ablations |
| `beast` | 50 | 2% | Everything + assurance audits |

```
/reproduce --effort max
```

---

## Available Skills

Copy `skills/` to `.claude/commands/` in your target project to enable all **15 skills**:

### Loop Control

| Command | Purpose |
|---------|---------|
| `/reproduce [--effort] [--reviewer]` | Start or resume the full autonomous loop |
| `/loop-once [--focus metric] [--phase]` | Run exactly one iteration manually |
| `/extend-loop [--add-iterations N]` | Add iterations to an exhausted loop |
| `/reset-loop [--keep-history] [--hard]` | Fresh start, preserving code by default |

### Status & Evaluation

| Command | Purpose |
|---------|---------|
| `/reproduce-status` | Quick status + milestone snapshot from LOOP_STATE.md |
| `/evaluate [--verbose]` | Check progress without running experiments |
| `/write-report [--type success|progress]` | Generate formal reproduction report |

### Execution

| Command | Purpose |
|---------|---------|
| `/setup-env [--gpu] [--skip-data]` | Initialize environment and download datasets |
| `/run-experiment [--dry-run] [--tag]` | Execute experiment with logging and error recovery |
| `/save-checkpoint [--message]` | Git-commit current state as recoverable snapshot |

### Diagnosis

| Command | Purpose |
|---------|---------|
| `/debug-gap [--metric] [--depth]` | Diagnose why a specific metric is failing |
| `/check-impl [--section] [--strict]` | Audit code against all paper documentation |
| `/check-plateau [--metric] [--window]` | Detect stagnation and classify plateau type |

### Review & Paper Analysis

| Command | Purpose |
|---------|---------|
| `/handoff-review [--reviewer] [--receive]` | Prepare external review package; parse verdict |
| `/paper-parse [--file] [--focus]` | Extract metrics/arch/hyper from paper (pre-Stage 1) |

---

## Key Design Principles

### From karpathy/autoresearch
- **Simplicity**: the loop is implement → execute → evaluate → iterate. No unnecessary complexity.
- **Single-metric focus per iteration**: target the worst-failing metric with a surgical fix
- **Git-based tracking**: every iteration is a checkpoint — nothing gets lost
- **No stopping until human says so**: the loop runs relentlessly until success

### From ARIS (wanshuiyin/Auto-claude-code-research-in-sleep)
- **Composable Markdown skills**: each `/command` is a plain `SKILL.md` file — works across any Claude Code setup
- **Effort levels**: `lite → balanced → max → beast` independently control depth
- **Artifact-based communication**: skills pass information through files (LOOP_STATE.md), not memory
- **Shared contracts**: `effort-contract.md` and `loop-contract.md` ensure skill consistency
- **Cross-model review**: optional `--reviewer codex|gpt` for adversarial code review

### fxxk-coming-soon specific
- **Paper as oracle**: the paper's reported metrics ARE the success criteria — no subjective judgment
- **Machine-readable targets**: `Paper Target Metrics` YAML in `DATA_AND_EVAL.md` parsed directly by the loop
- **Milestone gates**: `CLAIMS_AND_GATES.md` enforces baseline-first (M1 before M2) — prevents wasting iterations on method improvements when the baseline itself is broken
- **Plateau detection**: PHASE 3.5 classifies stagnation into 4 types and applies type-specific escalation — no more spinning on the same fix
- **Assumption ladder**: all ambiguous paper details documented with default/fallback/validation — the loop can self-debug using it

---

## What Gets Generated on Success

```
project/
├── REPRODUCTION_REPORT.md     # Full comparison table + notes
├── LOOP_STATE.md              # Complete iteration history
├── logs/
│   ├── iter_1.log
│   ├── iter_2.log
│   └── iter_N.log
└── results/
    ├── iter_1/
    ├── iter_2/
    └── best_checkpoint/
```

---

## Recommended LLMs for Stage 1 (Documentation Generation)

- **Claude Opus 4.7** — Best for complex methodologies, multi-component architectures
- **GPT-5** — Strong structured output, good at extracting exact hyperparameters
- **Grok 4** — Fast iteration, good for quick documentation drafts

---

## License

MIT
