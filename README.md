# fxxk-coming-soon

> Bridging the gap between research papers and executable code.

---

## 1. Background
In recent years, more and more research papers release GitHub repositories with only a **"Coming Soon"** message, or they choose not to release any code at all.  
This greatly slows down reproducibility and the progress of open science.

**fxxk-coming-soon** aims to help researchers use Large Language Models (LLMs) to quickly transform an English research paper into a well-defined project skeleton, ready for autonomous code implementation.

---

## 2. Core Idea
- Combine the target research paper content with the **Prompt** provided in this repository.
- Feed them into an LLM (recommended: **Claude 4**, **GPT-5**, or **Grok 4**).
- The LLM outputs **4 structured `.md` documentation files** describing the entire implementation plan.
- The user creates a local project folder, saves these `.md` files into it.
- Then, in that same workspace, use **Claude Code** (or another autonomous coding assistant) to implement the full codebase following the generated documentation.

---

## 3. Usage Workflow
1. **Prepare the paper**  
   Extract the relevant sections (especially the methodology) from the English research paper.

2. **Get the Prompt**  
   Open [`PAPER_TO_CODE_PROMPT.md`](PAPER_TO_CODE_PROMPT.md) in this repository.

3. **Run with LLM**  
   Provide both the extracted paper content and the prompt to an LLM (Claude 4, GPT-5, or Grok 4 are recommended for their reasoning and planning abilities).

4. **Save the outputs**  
   The LLM will produce:
   - `PROJECT_STRUCTURE.md`
   - `IMPLEMENTATION_PLAN.md`
   - `DATA_AND_EVAL.md`
   - `RISKS_AND_NOTES.md`  
   Save them into your local project folder.

5. **Implement the code**  
   Use **Claude Code** inside this folder to generate and refine the implementation according to the documentation.

---

## 4. Repository Structure

```
fxxk-coming-soon/
│
├── README.md                  # Project description & usage guide
├── PAPER_TO_CODE_PROMPT.md    # Main prompt template
├── examples/                  # Example outputs from real papers
│   └── demo-paper/            # The 4 generated .md files for one paper
├── templates/                 # Blank Markdown templates
│   ├── PROJECT_STRUCTURE.md
│   ├── IMPLEMENTATION_PLAN.md
│   ├── DATA_AND_EVAL.md
│   └── RISKS_AND_NOTES.md
└── docs/                      # Additional guides & notes
```



---

## 5. Recommended Tools
- **Claude 4** – Excellent long-context handling & planning ability  
- **GPT-5** – Balanced coding and reasoning capabilities  
- **Grok 4** – Fast and interactive for iteration  

---

## 6. License
MIT License
