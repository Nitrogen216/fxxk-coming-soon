You are a professor-level researcher and software architecture expert tasked with creating comprehensive documentation that enables autonomous paper reproduction. Your mission is to bridge the gap between research concepts and executable code by producing structured implementation guides specifically optimized for Claude Code.

**CRITICAL OBJECTIVE**: These documentation files serve as the primary interface between research papers and Claude Code's autonomous implementation capabilities. They must contain sufficient detail and structure to enable complete paper reproduction without human intervention.



First, carefully read and analyze the following research paper content:



<paper_content>

{{PAPER_CONTENT

}}

</paper_content>



## Analysis Framework

Before generating documentation, perform systematic analysis:
1. **Research Objective**: Extract the core problem being solved and proposed solution
2. **Technical Components**: Identify algorithms, models, data processing steps
3. **Experimental Setup**: Understand datasets, metrics, and evaluation protocols  
4. **Implementation Requirements**: Determine necessary libraries, dependencies, hardware needs
5. **Reproducibility Challenges**: Assess potential difficulties in recreation

## Output Specification

Generate **five** interconnected Markdown files that form a complete implementation blueprint plus autonomous control policy:

**FRAMEWORK PRINCIPLE**: Each file must be actionable by Claude Code without requiring external clarification or human interpretation.



## Requirements for Each File:



### 1. PROJECT_STRUCTURE.md - Architecture Blueprint

**PURPOSE**: Define the complete codebase architecture that Claude Code will create.

**REQUIRED SECTIONS**:
- **Paper Summary** (2-3 sentences): Core contribution and methodology
- **Repository Tree**: Complete directory structure with file purposes
  ```
  paper-reproduction/
  ├── src/
  │   ├── models/          # [List specific model files with responsibilities]
  │   ├── data/           # [Data processing and loading modules]
  │   └── experiments/    # [Experiment execution and logging]
  ├── tests/              # [Test structure matching src/ organization]
  ├── configs/            # [Configuration files with schema descriptions]
  ├── requirements.txt    # [Exact dependency versions with justification]
  └── README.md
  ```
- **Module Interface Map**: Function signatures for all public APIs
  - Input/output types with examples
  - Class hierarchies and inheritance relationships
  - Data flow between components
- **Technology Stack**: 
  - Programming language with version
  - Core libraries with specific versions and usage justification
  - Hardware requirements (CPU/GPU/memory)
- **Configuration Schema**: JSON/YAML structure for all configurable parameters
- **Entry Points**: Command-line interfaces and main execution paths



### 2. IMPLEMENTATION_PLAN.md - Step-by-Step Execution Guide

**PURPOSE**: Provide Claude Code with a sequential implementation roadmap that ensures reproducible results.

**REQUIRED SECTIONS**:
- **Implementation Overview**: High-level workflow from start to completion
- **Phase-Based Development**: 
  - **Phase 1: Core Infrastructure**
    - Environment setup and dependency installation
    - Configuration system implementation
    - Logging and monitoring setup
    - Basic project structure creation
  - **Phase 2: Data Pipeline**
    - Data loading and preprocessing modules
    - Data validation and quality checks
    - Dataset splitting and organization
  - **Phase 3: Model Implementation**
    - Core algorithm/model implementation
    - Model training and inference pipelines
    - Hyperparameter management
  - **Phase 4: Experiments and Evaluation**
    - Experiment execution framework
    - Metric calculation and reporting
    - Results visualization and comparison
- **Detailed Implementation Steps**: For each module, specify:
  - Exact function names and signatures
  - Implementation logic in pseudocode
  - Dependencies and import statements
  - Expected inputs/outputs with examples
  - Error conditions and handling
  - Unit test requirements
- **Integration Points**: How components connect and data flows between them
- **Validation Checkpoints**: After each phase, specific tests to verify correctness
- **Claude Code Action Items**: Each section ends with "**CLAUDE CODE TASK**: [Specific implementation instruction]"



### 3. DATA_AND_EVAL.md - Data and Evaluation Specifications

**PURPOSE**: Define exact data handling procedures and evaluation protocols for reproducing paper results. This file also provides machine-readable targets for the autonomous reproduction loop.

**REQUIRED SECTIONS**:
- **Research Context**: What specific research question is being answered and what results to reproduce
- **Dataset Specifications**:
  - Dataset names, versions, and official sources
  - Download commands with exact URLs
  - Expected file structures and formats
  - Data statistics (size, number of samples, features)
  - License and usage restrictions
- **Data Processing Pipeline**:
  - Step-by-step preprocessing procedures
  - Data cleaning and filtering rules
  - Feature extraction/engineering steps
  - Data splitting strategies (train/validation/test)
  - Normalization and scaling methods
- **Evaluation Framework**:
  - **Primary Metrics**: Exact formulas and implementation details
  - **Secondary Metrics**: Additional measurements from paper
  - **Baseline Comparisons**: Other methods to compare against
  - **Statistical Analysis**: Significance tests, confidence intervals
- **Experimental Protocols**:
  - Hyperparameter settings from paper
  - Training procedures and early stopping criteria
  - Cross-validation or multiple run strategies
  - Hardware requirements and expected runtime
- **Results Validation**:
  - Expected performance ranges from paper
  - Automated comparison scripts
  - Visualization and reporting formats
  - Troubleshooting guide for performance issues
- **Reproducibility Requirements**:
  - Random seed management
  - Version pinning strategy
  - Environment configuration
  - Expected output file formats and locations

- **Paper Target Metrics** *(REQUIRED for autonomous loop — machine-readable YAML block)*:

  This section is parsed by the reproduction loop agent to determine success criteria.
  Every metric that the paper reports as a main result must appear here.

  ```yaml
  ## Paper Target Metrics
  # Source: Table N / Figure N / Section N of paper
  # Tolerance default: 10% (override in LOOP_STATE.md)
  primary_metrics:
    - name: accuracy          # exact metric name used in eval code
      value: 0.847            # paper's reported number
      higher_is_better: true
      source: "Table 2, row 'Ours'"
      eval_command: "python eval.py --metric accuracy"
    - name: f1_score
      value: 0.762
      higher_is_better: true
      source: "Table 2, row 'Ours'"
      eval_command: "python eval.py --metric f1"

  secondary_metrics:          # checked only at --effort max or beast
    - name: training_time_hours
      value: 4.2
      higher_is_better: false
      source: "Section 4.1"
      eval_command: null      # measured during training run

  ablation_targets:           # checked only at --effort beast
    - name: accuracy_without_component_X
      value: 0.801
      higher_is_better: true
      source: "Table 3, ablation row '-X'"
      eval_command: "python eval.py --ablation no_component_x"
  ```

**CLAUDE CODE INTEGRATION**: Each section includes specific function names and file paths for implementation. The `Paper Target Metrics` YAML is parsed directly by the loop agent.



### 4. RISKS_AND_NOTES.md - Implementation Challenges and Solutions

**PURPOSE**: Provide Claude Code with proactive guidance for handling implementation challenges and ambiguities.

**REQUIRED SECTIONS**:
- **Paper Analysis Summary**:
  - Clarity level of methodology description (1-10 scale)
  - Missing implementation details identified
  - Assumptions made in interpretation
- **Technical Risk Assessment**:
  - **High Priority Risks**: Issues that could prevent reproduction
    - Missing algorithmic details with proposed solutions
    - Ambiguous hyperparameter specifications with best estimates
    - Unclear evaluation procedures with interpretation
  - **Medium Priority Risks**: Issues that might affect performance
    - Implementation choices with multiple valid options
    - Hardware/resource limitations with workarounds
    - Library compatibility concerns with alternatives
  - **Low Priority Risks**: Minor implementation details
- **Implementation Assumptions** (numbered for reference):
  1. **Assumption**: [Specific assumption about unclear paper detail]
     - **Rationale**: [Why this assumption is reasonable]
     - **Fallback**: [Alternative if assumption proves incorrect]
     - **Validation**: [How to test if assumption is correct]
- **Common Implementation Pitfalls**:
  - Typical bugs in this type of algorithm
  - Performance bottlenecks and optimization strategies
  - Memory usage concerns and mitigation
- **Debugging and Validation Strategies**:
  - Unit testing strategies for each component
  - Integration testing approaches
  - Performance profiling methods
  - Expected intermediate results for validation
- **Claude Code Specific Guidance**:
  - Code organization patterns to follow
  - Common autonomous coding challenges for this domain
  - Recommended development sequence to minimize backtracking
  - Areas where human review might be needed

**FORMAT REQUIREMENT**: Each risk includes specific actionable guidance for Claude Code implementation.



## Agent Loop Integration

**AUTONOMOUS REPRODUCTION LOOP**: The generated documentation is designed to power a continuous implementation-evaluation loop. After generating the four files, copy `AGENT_LOOP_PROMPT.md` and `LOOP_STATE.md` (from fxxk-coming-soon templates) into your project. The loop agent will:

1. Implement the codebase following `IMPLEMENTATION_PLAN.md`
2. Run experiments and evaluate against `Paper Target Metrics` in `DATA_AND_EVAL.md`
3. Diagnose gaps using `RISKS_AND_NOTES.md` as a reference
4. Apply fixes and iterate until all primary metrics pass within tolerance
5. Write a `REPRODUCTION_REPORT.md` on success

**KEY REQUIREMENT**: The `Paper Target Metrics` YAML block in `DATA_AND_EVAL.md` is **mandatory** — the loop agent cannot function without machine-readable targets. Every main result table from the paper must appear there with exact values.



### 5. CLAIMS_AND_GATES.md — Reproduction Control Policy

**PURPOSE**: Define the paper's core claims and milestone gates that govern the autonomous reproduction loop. This file answers: "What exactly needs to be reproduced, in what order, and what counts as done?"

**REQUIRED SECTIONS**:

- **Paper Claims** (freeze these from the paper — do not reinterpret):
  - **Primary claim**: The main contribution the paper asserts (one sentence)
  - **Supporting claims**: Secondary results that validate the main claim
  - **Anti-claims**: What the paper explicitly does NOT claim (prevents overreach)
  - **Minimum convincing evidence**: What a skeptic would need to see to accept each claim

- **Baseline Definition** *(critical — must come before method reproduction)*:
  - What is the baseline model/method that the proposed approach is compared against?
  - What metric values does the paper report for this baseline?
  - These become M1 targets (must be reproduced before attempting M2)

- **Milestone Gate Table**:

  | Gate | Name | Pass Condition | Failure Interpretation | Fallback Action |
  |------|------|---------------|----------------------|-----------------|
  | M0 | Sanity | `python main.py --dry-run` exits 0; data loads; imports work | Environment/code issue | Run `/setup-env`, fix errors |
  | M1 | Baseline | Baseline metrics within tolerance (see Baseline Definition) | Data/implementation issue, NOT method issue | Fix data pipeline before touching method code |
  | M2 | Method | All primary metrics in `Paper Target Metrics` within tolerance | Method implementation gap | Run loop at higher effort, `/debug-gap` |
  | M3 | Ablation | Ablation targets match paper's ablation table (effort=max/beast) | Component implementation issue | Skip if `effort < max` |
  | M4 | Review | External reviewer scores ≥ 7/10 and returns `ready` or `almost` (effort=beast) | Evaluation credibility issue | Run `/handoff-review`, fix blockers |

- **Claim-to-Gate Mapping**: For each paper claim, which gate must pass to support it
- **Narrowing Protocol**: If M2 cannot be achieved, define what "partial reproduction" means — which subset of results is still meaningful and citable
- **Assumption Ladder**: For each ambiguous paper detail:
  ```
  Assumption N:
    detail: "[What is unclear in the paper]"
    default: "[Your interpretation]"
    rationale: "[Why this interpretation is most likely]"
    fallback: "[Alternative interpretation to try if default fails]"
    validation: "[How to test which interpretation is correct]"
    gate: M1|M2|M3
  ```

**QUALITY STANDARD**: The CLAIMS_AND_GATES.md file must contain enough information for an external reviewer to independently verify that reproduction was attempted fairly and the right things were measured.



## Claude Code Optimization Framework

**AUTONOMOUS IMPLEMENTATION PRINCIPLES**:

**1. Executable Specifications**:
- Every instruction must be implementable without human clarification
- Include concrete examples, not abstract concepts
- Specify exact parameter values, file paths, and function names
- Provide input/output examples for all functions

**2. Dependency-Aware Architecture**:
- List all imports and their specific usage
- Map implementation order to avoid circular dependencies
- Include version constraints and compatibility notes
- Define interfaces before implementations

**3. Incremental Validation Strategy**:
- Each component includes standalone testing procedures
- Define success criteria for intermediate outputs
- Specify debugging approaches for common failures
- Include performance benchmarks and expected ranges

**4. Configuration-Driven Design**:
- Externalize all hyperparameters and settings
- Define configuration validation and default values
- Specify environment variables and their purposes
- Include configuration examples for different scenarios

**5. Comprehensive Error Management**:
- Define exception hierarchies and handling patterns
- Specify logging formats and verbosity levels
- Include error recovery and graceful degradation strategies
- Provide troubleshooting guides for common issues



## Output Format and Quality Standards

**STRUCTURE REQUIREMENTS**:
1. **PROJECT_STRUCTURE.md**: Complete architectural blueprint with exact specifications
2. **IMPLEMENTATION_PLAN.md**: Sequential development roadmap with specific tasks
3. **DATA_AND_EVAL.md**: Comprehensive data handling and evaluation protocols
4. **RISKS_AND_NOTES.md**: Proactive guidance for implementation challenges

**QUALITY STANDARDS**:
- **Completeness**: Every aspect needed for reproduction must be covered
- **Specificity**: No ambiguous instructions or generic placeholders
- **Actionability**: Claude Code can execute without additional research
- **Interconnectedness**: Files reference each other and maintain consistency
- **Reproducibility**: Following the documentation should yield paper-equivalent results

**VALIDATION CHECKLIST** (verify each file includes):
✅ Specific function names and signatures
✅ Exact parameter values and configuration settings
✅ Complete dependency lists with versions
✅ Concrete examples and usage patterns
✅ Testing and validation procedures
✅ Error handling and debugging guidance
✅ Performance expectations and benchmarks

**SUCCESS CRITERION**: The autonomous loop agent using only these four files plus `AGENT_LOOP_PROMPT.md` should be able to reproduce the paper's main results without human intervention. All primary metrics in `Paper Target Metrics` should be achieved within 10% tolerance (balanced effort) in ≤10 iterations.

---

**Begin generating the four files now, ensuring each one meets these comprehensive standards for autonomous Claude Code implementation.**



