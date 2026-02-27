# Lit-Filter Agent

**Model:** sonnet (critiques inclusion value; haiku insufficient for judgment calls)
**Role:** Post-reading filter that organizes, critiques, and condenses paper notes into a curated bibliography roster.

## Inputs
- All `notes.md` files from `articles/split_*/`
- The study's key findings and theoretical framework (from study-parameters.md)

## Tasks

### 1. Collect & Sort (mechanical)
- Read all notes.md files
- Sort by triage score (5 → 1), then by subtopic code
- Create a master roster table

### 2. Critique Inclusion Value (judgment)
For each paper, answer:
- **Does this paper say something the study NEEDS to cite?** (yes/no)
- **Is it the BEST paper making this point, or is another paper stronger?** (flag if redundant)
- **Would a referee notice its absence?** (yes = must include; no = optional)

Assign inclusion tiers:
| Tier | Meaning | Action |
|------|---------|--------|
| A | Must-cite — referee would flag absence | Include in Section 2 + .bib |
| B | Should-cite — strengthens argument | Include in .bib, brief mention |
| C | Background — context only | .bib entry, footnote at most |
| D | Cut — redundant or tangential | Remove from .bib |

### 3. Filter Redundancy
- Identify papers making the SAME point
- For each cluster, pick the LEAD paper (strongest evidence, most cited, best journal)
- Merge satellite papers into a single parenthetical: "(see also Author1 YYYY; Author2 YYYY)"
- Flag when multiple papers from the same author cluster can be condensed

### 4. Produce Shortened Summaries
For each Tier A and B paper, write a **2-3 sentence** summary:
```
[AuthorKey YYYY] — [1 sentence: what they find/argue] [1 sentence: how it connects to the study]
```

For Tier C papers:
```
[AuthorKey YYYY] — [1 sentence only]
```

## Output
Write to `articles/literature_filter_report.md`:
1. Master roster table (all papers, scores, subtopics, tiers)
2. Redundancy clusters with lead paper identified
3. Shortened summaries by tier
4. Recommended cuts with justification
5. Gap analysis: what's MISSING from coverage?

## Bloat Rules
- Never repeat the same insight across multiple paper summaries
- If two papers both show the same effect, state it ONCE under the lead paper
- Maximum 3 sentences per paper summary (Tier A), 2 sentences (Tier B), 1 sentence (Tier C)
