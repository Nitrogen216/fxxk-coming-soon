# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**fxxk-coming-soon** is a documentation framework that bridges the gap between research papers and executable code. The repository contains:

- A master prompt template (`PAPER_TO_CODE_PROMPT.md`) for generating structured implementation documentation
- Markdown templates for organizing project planning
- A workflow designed specifically for Claude Code autonomous implementation

The core concept: Use the prompt template with an LLM to convert research papers into 4 structured `.md` files, then use Claude Code to implement the actual codebase following this documentation.

## Repository Architecture

```
fxxk-coming-soon/
├── README.md                  # Main project documentation
├── PAPER_TO_CODE_PROMPT.md    # Master prompt template (200+ lines)
├── templates/                 # Blank Markdown templates in Chinese
│   ├── PROJECT_STRUCTURE.md
│   ├── IMPLEMENTATION_PLAN.md
│   ├── DATA_AND_EVAL.md
│   └── RISKS_AND_NOTES.md
└── example/                   # Directory for example outputs
```

## Key Files and Their Purpose

### PAPER_TO_CODE_PROMPT.md
- Main prompt template for LLM interaction
- Contains detailed specifications for generating implementation documentation
- Optimized for Claude Code autonomous coding workflows
- Defines requirements for all 4 output documentation files

### Template Files (templates/)
- Currently contain basic Chinese template structures
- Act as placeholders for the actual English documentation that gets generated
- Define the schema that the prompt template produces

## Development Workflow

This repository supports a specific workflow:

1. **Paper Analysis**: Use `PAPER_TO_CODE_PROMPT.md` with research paper content in an LLM
2. **Documentation Generation**: LLM outputs 4 structured `.md` files
3. **Project Setup**: Create new project folder with the generated documentation
4. **Implementation**: Use Claude Code in that project to implement following the documentation

## Working with Documentation Templates

When modifying templates or the prompt:
- Templates are currently in Chinese but the generated outputs should be in English
- The prompt template specifies Claude Code optimization requirements
- Each template corresponds to a specific aspect of implementation planning

## No Build/Test Commands

This is a documentation repository with no executable code, package.json, or build processes. All files are Markdown documentation meant for human and LLM consumption.

## Claude Code Implementation Notes

When working on actual implementation projects using this framework:
- The generated documentation will specify exact function signatures and module dependencies
- Implementation should follow the modular approach outlined in the generated `IMPLEMENTATION_PLAN.md`
- Testing structure will be defined in the project-specific documentation
- Configuration and dependencies will be specified in the generated `PROJECT_STRUCTURE.md`