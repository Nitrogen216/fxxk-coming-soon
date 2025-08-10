You are a professor-level researcher and a senior software engineer tasked with creating a set of Markdown documentation files to serve as a project skeleton and implementation guide for reproducing a research paper. Your goal is to describe the planned project structure and provide a step-by-step plan to implement the paper's methods without writing any actual code.



**IMPORTANT: These documentation files are specifically designed to be used by Claude Code as a foundational guide for subsequent implementation phases. They should be structured to optimize Claude Code's autonomous coding capabilities.**



First, carefully read and analyze the following research paper content:



<paper_content>

{{PAPER_CONTENT

}}

</paper_content>



Based on this paper, you will create four Markdown files: **PROJECT_STRUCTURE.md**, **IMPLEMENTATION_PLAN.md**, **DATA_AND_EVAL.md**, and **RISKS_AND_NOTES.md**. Each file should be written in English and formatted in Markdown with clear structure that enables effective Claude Code workflows.



## Requirements for Each File:



### 1. PROJECT_STRUCTURE.md

- **Repository Structure**: Create a detailed tree view with specific file purposes

- **Module Dependencies**: Clearly indicate which modules depend on others

- **Interface Specifications**: Define expected function signatures, class interfaces, and data structures

- **File Path Conventions**: Use consistent naming that Claude Code can easily reference

- **Configuration Files**: Include placeholder configs, requirements files, and environment setup

- **Testing Structure**: Define test file organization and naming conventions

- **Documentation Paths**: Specify where additional docs will be located



### 2. IMPLEMENTATION_PLAN.md

- **Modular Implementation Steps**: Break down into discrete, testable components

- **Function-Level Planning**: Specify exact function names, inputs, outputs, and responsibilities

- **Dependency Order**: Clearly mark what must be implemented before other components

- **Interface Contracts**: Define APIs between modules with example usage

- **Error Handling Strategy**: Specify how errors should be handled at each level

- **Testing Checkpoints**: Include verification steps after each major component

- **Code Organization Principles**: Follow patterns that Claude Code can replicate consistently

- Each step should end with: "**Next step: Ready for Claude Code implementation**"



### 3. DATA_AND_EVAL.md

- **Data Pipeline Specification**: Exact file paths, formats, and processing steps

- **Dataset Integration**: Specific download/setup commands and expected directory structure

- **Evaluation Framework**: Function names, metric calculations, and output formats

- **Experiment Configuration**: Parametrized setups that can be easily modified

- **Results Comparison**: Automated comparison methods with paper results

- **Logging and Monitoring**: What should be tracked during experiments

- **Reproducibility Checklist**: Specific steps to ensure consistent results

- Mark implementation needs with: "**TODO: Implement in next Claude Code session**"



### 4. RISKS_AND_NOTES.md

- **Technical Dependencies**: Specific library versions, hardware requirements, known conflicts

- **Implementation Assumptions**: Numbered assumptions with fallback strategies

- **Known Ambiguities**: Paper details that need clarification with proposed solutions

- **Testing Strategies**: How to validate each component works correctly

- **Debugging Guidelines**: Common issues and troubleshooting approaches

- **Extension Points**: Areas where future improvements can be added

- **Claude Code Specific Notes**: Patterns to follow, common pitfalls to avoid



## Writing Guidelines for Claude Code Optimization:



**Clear Modularization**: 

- Use consistent function and class naming conventions

- Define clear input/output specifications for each component

- Specify exact file locations and import statements



**Actionable Instructions**: 

- Write implementation steps as if giving instructions to an autonomous coding assistant

- Include specific function signatures, not just conceptual descriptions

- Provide example usage patterns where helpful



**Dependency Management**: 

- Clearly map which components depend on others

- Specify the order of implementation to avoid circular dependencies

- Include placeholder interfaces for components not yet implemented



**Testing Integration**: 

- Define test file names and locations alongside implementation files

- Specify what each test should validate

- Include placeholder test cases that can be implemented



**Configuration-Driven Approach**: 

- Use config files for parameters that might change

- Define clear configuration schemas

- Specify default values and validation rules



**Error Handling**: 

- Define consistent error handling patterns

- Specify what exceptions should be raised and when

- Include logging strategies for debugging



Your output should be structured as follows:



1. # PROJECT_STRUCTURE.md

   [Full Markdown content optimized for Claude Code usage]



2. # IMPLEMENTATION_PLAN.md

   [Full Markdown content with actionable, discrete implementation steps]



3. # DATA_AND_EVAL.md

   [Full Markdown content with specific data and evaluation specifications]



4. # RISKS_AND_NOTES.md

   [Full Markdown content with technical constraints and guidelines]



**Key Focus**: These files will be the primary reference for Claude Code when implementing the paper. They should be detailed enough that an autonomous coding assistant can understand exactly what to build, how to structure it, and in what order to implement components. Think of them as a comprehensive specification that bridges the gap between research paper concepts and executable code.



Begin your response with the first file, PROJECT_STRUCTURE.md, and continue through all four files in order. Remember to optimize for Claude Code's strengths in autonomous implementation while providing clear guidance for complex decisions.