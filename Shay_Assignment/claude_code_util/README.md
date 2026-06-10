# Claude Code — How It Was Used in This Assignment

This folder documents the use of [Claude Code](https://claude.ai/code) (Anthropic's AI-powered CLI) as a core part of the development workflow for this assignment.

---

## What is Claude Code?

Claude Code is an agentic AI assistant that runs directly in the terminal and IDE. It can read, write, and execute code; run shell commands; manage git; and maintain persistent memory across sessions — acting as a collaborative engineering partner rather than a one-shot code generator.

---

## How Claude Code drove this project

### 1. Project scaffolding & environment setup
- Detected that `ipykernel` was missing from the venv, installed it, and registered a named Jupyter kernel (`vi-ml`) so the notebook ran against the correct Python 3.9 environment
- Created `Shay_Assignment/` as the canonical output directory and reorganised files into it
- Wrote `.gitignore`, `requirements.txt`, and `README.md` in one pass

### 2. Notebook debugging
Claude Code executed the EDA notebook end-to-end and caught three runtime bugs introduced by a `pandas 1.3` incompatibility with `Period`-dtype columns:
- `datetime64` vs `date` comparison in claims window filter
- `fillna(0)` on a merged frame that included a `Period`-typed column (three separate occurrences)

Each fix was targeted and minimal — no surrounding code was changed.

### 3. EDA & knowledge capture (`CLAUDE.md`)
After running all 35 notebook cells, Claude Code synthesised the outputs into structured EDA conclusions and wrote them directly into `CLAUDE.md` — a persistent project-memory file that accumulates context across sessions. This means future Claude Code sessions on this project start with full awareness of data quality issues, signal strengths, and feature engineering decisions already made.

See [`CLAUDE.md`](./CLAUDE.md) for the full accumulated context.

### 4. Git & GitHub setup
- Created the initial commit with appropriate staging (data files excluded, plots included)
- Created the public GitHub repository via the GitHub REST API
- Pushed the branch and set upstream tracking

### 5. Persistent memory & phase gating
Claude Code maintains a `memory/` directory that persists facts about the user, project, and feedback across sessions. The project `CLAUDE.md` enforces phase gates ("never proceed to the next phase without approval") which Claude Code respects — summarising findings after each phase and waiting for explicit go-ahead before continuing.

---

## CLAUDE.md — the project brain

`CLAUDE.md` is the key artifact of Claude Code usage. It acts as a living project document that Claude Code reads at the start of every session. Over the course of this assignment it accumulated:

- Data schema and timeline
- EDA conclusions (churn rates, signal strengths, data quality issues)
- Feature engineering decisions
- Deliverable status (which are complete, which are pending)
- Rules and phase gates

A copy is included in this folder: [`CLAUDE.md`](./CLAUDE.md)
