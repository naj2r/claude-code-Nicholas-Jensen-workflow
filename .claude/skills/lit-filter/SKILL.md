---
name: lit-filter
description: Organize, critique inclusion value, and filter redundancy across all paper notes. Run AFTER split-pdf reading is complete for all papers. Produces a curated roster and shortened summaries.
allowed-tools: Read, Write, Edit, Glob, Grep, Task
argument-hint: [no arguments needed]
---

# Lit-Filter: Organize, Critique, and Condense Paper Notes

## When to Use
Run this skill AFTER all (or most) papers have been read via split-pdf and have notes.md files in their split directories.

## Workflow

### Step 1: Collect All Notes (haiku — pure lookup)
```
Glob: articles/split_*/notes.md
Read each notes.md file
Extract: paper key, triage score, subtopic codes, connection paragraph
```

Build a quick-reference table:
```markdown
| # | Paper | Score | Subtopic | Pages | Connection Summary |
```

### Step 2: Launch Filter Agent (sonnet — requires judgment)
Spawn the `lit-filter` agent with:
- The collected table from Step 1
- The study parameters (key findings, treatment or identification approach)
- Instructions to produce `articles/literature_filter_report.md`

The agent will:
1. Critique each paper's inclusion value (Tier A/B/C/D)
2. Identify redundancy clusters
3. Write shortened summaries
4. Flag gaps in coverage

### Step 3: Apply to QMD (main conversation)
After the agent returns:
1. Read `articles/literature_filter_report.md`
2. Use the tier assignments to structure the literature sources QMD chapter
3. Use the shortened summaries as the basis for the prose synthesis
4. Use the gap analysis to target the literature harvest searches

## Model Routing
| Task | Model | Why |
|------|-------|-----|
| Globbing notes.md files | haiku | Pure file lookup |
| Reading/collecting notes | haiku | Extracting structured fields |
| Critiquing inclusion value | sonnet | Requires domain judgment |
| Identifying redundancy | sonnet | Needs to compare arguments |
| Writing summaries | sonnet | Condensing requires understanding |
| Applying to QMD | opus | Restructuring into narrative prose |

## Output
`articles/literature_filter_report.md` — the single artifact this skill produces.
