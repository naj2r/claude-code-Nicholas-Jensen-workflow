---
name: lit-synthesizer
description: Rate, organize, and synthesize paper summaries into a condensed literature review. Scores papers by relevance to the study, groups by subtopic, and eliminates bloat. No permission needed for bulk processing.
allowed-tools: Read, Write, Edit, Glob, Grep, Task
argument-hint: [path-to-paper-skims-or-notes-directory]
---

# Literature Synthesizer: Rate, Organize, and Condense

**CORE PRINCIPLE: No pausing, no permission. Process all papers in one pass.** Read the input, rate every paper, organize by subtopic, condense redundancies, and write the synthesized output. This is a bulk operation.

## When This Skill Is Invoked

The user has a collection of paper summaries (from `split-pdf` notes or `paper_skims.md`) and wants them:
1. **Rated** by relevance to the study
2. **Organized** by subtopic category
3. **Condensed** into a synthesized literature review without bloat or redundancy

## Input

`$ARGUMENTS` — one of:
- Path to `paper_skims.md` or similar collection file
- Path to a directory containing `notes.md` files (from split-pdf reads)
- `all` — process everything in `docs/` and `articles/` directories

If no argument is given, default to:
```
docs/paper_skims.md
```

## Step 1: Collect All Paper Summaries

Read all available paper notes from:
1. `docs/paper_skims.md` — the master skim file
2. `articles/split_*/notes.md` — any deep-read notes from split-pdf
3. Any other `.md` files in `docs/` that contain paper summaries

Merge them. If a paper has both a skim AND split-pdf notes, **use the split-pdf notes** (more detailed).

## Step 2: Load Study Context

Load study parameters from `study-parameters.md` (or equivalent project configuration). This provides:
- The study's research question and key findings
- The primary identification strategy and method
- The main theoretical mechanism(s) being tested

**[Load from study-parameters.md]** — pass this context to the agent in Step 3.

## Step 3: Launch the Lit-Synthesizer Agent

Spawn a Task agent with `subagent_type: "general-purpose"` and `model: "sonnet"` that:

1. Reads ALL paper summaries
2. Rates each paper 1-5 using the agent's scoring rubric (see `.claude/agents/lit-synthesizer.md`)
3. Assigns primary + optional secondary subtopic codes
4. Identifies redundancies and overlaps
5. Writes the synthesized output

**Agent prompt template:**
```
You are the lit-synthesizer agent. Read the agent specification at:
.claude/agents/lit-synthesizer.md

Then read all paper summaries provided below and produce the synthesized output.

Study context:
[INSERT CONTENT FROM study-parameters.md — title, method, key findings, theoretical interpretation]

[PAPER SUMMARIES HERE]

Write the output to:
docs/literature_synthesis.md
```

## Step 4: Write the Synthesized Output

The agent writes to:
```
docs/literature_synthesis.md
```

This file contains:
1. **Relevance rankings table** — every paper scored and categorized
2. **Synthesized review by subtopic** — integrated narrative, NOT paper-by-paper summaries
3. **Gaps this study fills** — concise list
4. **Papers to cut** — low-relevance papers with justification

## Step 5: Update the Master Lit Review

After synthesis, update the literature sources QMD chapter (`replication_book/85_literature_sources.qmd` or equivalent) with the synthesized content:
- Replace the paper-by-paper inventory with the synthesized subtopic narratives
- Update the theoretical framework section with the organized structure
- Embed the relevance rankings table
- Add the theory-to-empirics linkage table

## Bloat Detection Rules

The synthesizer MUST compress the following patterns:

| Pattern | Action |
|---------|--------|
| Multiple papers making the same empirical point | Merge into one sentence citing all papers |
| Repeated "Connection to Our Study" for papers in same subtopic | State relevance ONCE per subtopic section |
| Multiple abstract-length summaries of similar papers | Condense to 1-2 sentences per paper within a narrative paragraph |
| Redundant institutional background | One paragraph max for the relevant institutional context |
| Repeated theoretical mechanism explanations | One theoretical paragraph, cite all supporting papers |
| Papers with similar methods and similar findings | Lead with the highest-impact version, parenthetically cite the rest |

## Target Compression

| Input | Target Output |
|-------|--------------|
| 20+ individual paper summaries (~300 lines each) | ~150-200 lines of synthesized narrative |
| Repeated per-paper "Connection" sections | ~20 lines of grouped connection text |
| 8-dimension extraction per paper | Selective dimensions only (skip methodology unless relevant to study design) |

## Output Quality Check

Before finalizing, verify:
- [ ] Every Tier 1 paper (from the plan) is rated 4-5 and appears in the synthesis
- [ ] No subtopic section exceeds 1 page (~40 lines) of prose
- [ ] No two consecutive paragraphs make the same point
- [ ] All BibTeX keys use `authorYYYYdescriptor` format
- [ ] Effect sizes and p-values are preserved (not rounded or paraphrased away)

## Quick Reference

| Step | Action |
|------|--------|
| **Collect** | Gather all paper summaries from skims + split-pdf notes |
| **Load** | Pull study context from study-parameters.md |
| **Rate** | Score 1-5 relevance, assign subtopic codes |
| **Condense** | Merge redundancies, compress per-paper to per-subtopic |
| **Write** | Output to `docs/literature_synthesis.md` |
| **Update** | Propagate to literature sources QMD chapter |
