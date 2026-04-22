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
├── PAPER_TO_CODE_PROMPT.md    # Stage 1: master LLM prompt → 5 doc files
├── AGENT_LOOP_PROMPT.md       # Stage 2: master loop controller for Claude Code
│
├── skills/                    # Claude Code slash commands (copy to target project)
│   │
│   │   # Project Setup
│   ├── init-project/SKILL.md      # /init-project — verify docs, create LOOP_STATE, sync baselines
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
│   ├── check-plateau/SKILL.md     # /check-plateau — detect stagnation
│   │
│   │   # Review & Paper Analysis
│   ├── handoff-review/SKILL.md    # /handoff-review — external review handoff
│   ├── paper-parse/SKILL.md       # /paper-parse — extract info from paper
│   │
│   └── shared/
│       ├── effort-contract.md     # effort level definitions (lite/balanced/max/beast)
│       └── loop-contract.md       # invariants all loop skills must honor
│
├── templates/                 # Templates for target paper reproduction projects
│   ├── LOOP_STATE.md              # Loop state tracker (copy to target project)
│   ├── CLAIMS_AND_GATES.md        # Paper claims + milestone gate template
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
3. Submit to a capable LLM (Claude Opus 4.7, GPT-5, Grok 4)
4. Save the **5 generated files**:
   - `PROJECT_STRUCTURE.md` — architecture blueprint with exact function signatures
   - `IMPLEMENTATION_PLAN.md` — phase-by-phase implementation roadmap
   - `DATA_AND_EVAL.md` — data pipelines + **machine-readable target metrics YAML**
   - `RISKS_AND_NOTES.md` — known challenges, debugging strategies, assumption ladder
   - `CLAIMS_AND_GATES.md` — **paper claims frozen + M0-M4 milestone gate definitions**

### Stage 2: Autonomous Reproduction Loop (runs until success)

Create a new project folder with the 5 generated files, then:

```bash
# Copy loop infrastructure from fxxk-coming-soon
cp /path/to/fxxk-coming-soon/AGENT_LOOP_PROMPT.md .
cp /path/to/fxxk-coming-soon/templates/LOOP_STATE.md .
cp /path/to/fxxk-coming-soon/templates/project-CLAUDE.md ./CLAUDE.md

# Install skills to .claude/skills/ (Claude Code's official skill directory)
mkdir -p .claude/skills
cp -r /path/to/fxxk-coming-soon/skills/* .claude/skills/

# Launch Claude Code, initialize, and start the loop
claude
> /init-project    # verifies files, creates LOOP_STATE.md, syncs baseline_targets
> /setup-env       # installs deps and downloads data
> /reproduce       # runs the autonomous loop
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

## How Skills Work

Skills are Claude Code **slash commands** — plain Markdown files that Claude reads and acts on when you type `/skill-name` in the Claude Code chat.

### Installation

Skills live in `.claude/skills/<skill-name>/SKILL.md` inside your project. Claude Code discovers them automatically on startup.

```
your-project/
└── .claude/
    └── skills/
        ├── reproduce/
        │   └── SKILL.md    # → /reproduce
        ├── evaluate/
        │   └── SKILL.md    # → /evaluate
        └── ...
```

### Invoking a Skill

Type the skill name with a leading `/` in Claude Code's chat:

```
/reproduce                            # no arguments
/reproduce --effort max               # with flags
/debug-gap --metric accuracy --depth deep
/loop-once --focus f1_score
```

Arguments after the skill name are available inside the skill as `$ARGUMENTS`.

### Skill Modes

Skills can operate in two modes:

| Mode | How | When used |
|------|-----|-----------|
| **Standard** | Claude reads the skill content and acts in your current conversation | Most skills |
| **Forked subagent** (`context: fork`) | Skill runs in an isolated subagent with no conversation history | `/reproduce` — long-running autonomous loop |

`/reproduce` uses `context: fork` so the loop runs fully isolated and doesn't pollute your conversation context. The subagent reads `LOOP_STATE.md` for all state — nothing is passed via memory.

### Controlled Invocation

`/reset-loop` has `disable-model-invocation: true` — Claude cannot trigger it automatically. Only you can run it by typing `/reset-loop`. This prevents accidental state resets during the autonomous loop.

### Shell Injection (Pre-execution)

Skills use `` !`command` `` to load live context before Claude sees the skill content:

```markdown
## Current State
!`cat LOOP_STATE.md`
```

This runs the shell command **before** the skill prompt is sent to Claude, so Claude sees the actual current state rather than a static template.

### Effort Levels

All loop skills respect `--effort`:

| Level | Max Iterations | Tolerance | Coverage |
|-------|---------------|-----------|----------|
| `lite` | 5 | 15% | Primary metrics only |
| `balanced` *(default)* | 10 | 10% | All primary metrics |
| `max` | 20 | 5% | Primary + secondary + ablations |
| `beast` | 50 | 2% | Everything + assurance audits |

```
/reproduce --effort max
/reproduce --effort beast --reviewer gpt
```

---

## Available Skills

### Project Setup

| Command | Description | Notes |
|---------|-------------|-------|
| `/init-project [--force-reset-state] [--skip-baseline-sync]` | Verify the 5 Stage-1 doc files, create `LOOP_STATE.md`, auto-sync M1 `baseline_targets` from `CLAIMS_AND_GATES.md` | **Run this first** — before `/setup-env` or `/reproduce` |

### Loop Control

| Command | Description | Notes |
|---------|-------------|-------|
| `/reproduce [--effort lite\|balanced\|max\|beast] [--reviewer none\|codex\|gpt]` | Start or resume the full autonomous loop | Runs in isolated subagent (`context: fork`) |
| `/loop-once [--focus metric] [--phase implement\|execute\|evaluate]` | Run exactly one iteration with manual control | Stops after one cycle for review |
| `/extend-loop [--add-iterations N] [--effort level]` | Add iterations to a timed-out loop | Resumes from current state, no data lost |
| `/reset-loop [--keep-history] [--keep-code] [--hard]` | Reset loop state for a fresh run | **Requires explicit invocation** — cannot be auto-triggered |

### Status & Evaluation

| Command | Description | Notes |
|---------|-------------|-------|
| `/reproduce-status` | Compact milestone + metric snapshot | Reads only; no experiments run |
| `/evaluate [--verbose] [--iteration N]` | Check metric gaps against paper targets | Shows trend across iterations |
| `/write-report [--type success\|progress\|failed]` | Generate formal reproduction report | Auto-detects type from `LOOP_STATE.md` |

### Execution

| Command | Description | Notes |
|---------|-------------|-------|
| `/setup-env [--python 3.10\|3.11\|3.12] [--gpu] [--skip-data]` | Init Python env, install deps, download data | Idempotent — safe to run multiple times |
| `/run-experiment [--config path] [--dry-run] [--resume] [--tag label]` | Execute pipeline with structured logging | Auto-retries up to 3× on failure |
| `/save-checkpoint [--message "text"] [--tag label]` | Git-commit current state as snapshot | Preserves best metrics in `results/best_checkpoint/` |

### Diagnosis

| Command | Description | Notes |
|---------|-------------|-------|
| `/debug-gap [--metric name] [--depth shallow\|deep]` | Root-cause analysis for a failing metric | Outputs ranked hypotheses with file locations |
| `/check-impl [--section all\|structure\|plan\|eval] [--strict]` | Audit code against all 5 paper doc files | Finds missing functions, wrong configs |
| `/check-plateau [--metric name] [--window N] [--threshold 0.01]` | Detect stagnation and classify type | Types: A (asymptotic), B (oscillating), C (ceiling), D (random) |

### Review & Paper Analysis

| Command | Description | Notes |
|---------|-------------|-------|
| `/handoff-review [--reviewer human\|codex\|gpt] [--receive]` | Prepare external review package; parse verdict | Use `--receive` to ingest reviewer JSON verdict |
| `/paper-parse [--file path] [--focus metrics\|arch\|data\|hyper\|all]` | Extract implementation info from paper | Outputs ready-to-paste YAML for `DATA_AND_EVAL.md` |

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
