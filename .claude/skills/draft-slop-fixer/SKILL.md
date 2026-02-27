---
name: draft-slop-fixer
description: Clean up stream-of-consciousness academic drafts — resolve citation gaps, reword copy-paste chunks, fill in TODO markers, and delegate sourcing tasks. Stage 1 of the writing pipeline; must run BEFORE any reviewer or prose-quality pass.
argument-hint: "[path to drafting document]"
allowed-tools: ["Read", "Grep", "Glob", "Edit", "Write", "Task", "Bash", "WebSearch", "WebFetch"]
---

# Draft Slop Fixer

**Purpose:** Take the user's flow-state academic draft and do the grunt work they deferred while writing — the tedious citation lookups, the "need more info" placeholders, the copy-paste insertions that need rewording, and the organizational scaffolding that needs condensing.

**This is Stage 1 of the pipeline.** Do NOT attempt to improve prose quality, restructure arguments, or compress length. That's for later stages. Your only job is mechanical cleanup so the draft is ready for reviewers.

**Input:** `$ARGUMENTS` — path to the drafting markdown file.

---

## Phase 1: Scan & Classify Issues

Read the entire drafting document. Build a categorized issue list:

### Category A: TODO/Placeholder Markers
Patterns to find:
- `(need more info)`, `(need to expand)`, `(need to expand on this)`
- `(CITE)`, `(cite)`, `(citation needed)`, `[?]`, `[TODO]`, `(TODO)`
- `(find this)`, `(look up)`, `(check this)`, `(verify)`
- `(need exact quote)`, `(get page number)`
- Any parenthetical that reads as a note-to-self rather than content

**Action:** For each, search the project's notes infrastructure:
1. `articles/split_*/notes.md` — paper reading notes
2. `articles/theory_notes_*.md` — deep theory extractions
3. `articles/notes_*.md` — web-sourced notes
4. `docs/paper_skims.md` — comprehensive paper summaries
5. `replication_book/85_literature_sources.qmd` — synthesized lit review (or equivalent)
6. `articles/literature_filter_report.md` — tier assignments and section plans
7. `$OL/bibliographyCiteDrive.bib` — existing bibliography keys (adjust path to match project config)

Fill in the gap with sourced content. If the needed information isn't in any project file, flag it for manual resolution (don't hallucinate citations).

### Category B: Uncited Claims
Statements that reference specific papers, authors, or findings without proper citation syntax:
- "Author 1988 shows..." → needs `@author1988descriptor` or equivalent
- "as Author found..." → needs specific key
- Author-year references without BibTeX key format

**Action:** Match to existing bib keys. If no key exists, check if the paper is in the filter report and flag for bib addition.

### Category C: Copy-Paste Insertions
Chunks that are clearly verbatim from Claude's Quarto notes or theory notes — identifiable by:
- Sudden shift to formal/encyclopedic tone mid-paragraph
- Bullet-point lists embedded in narrative prose
- Technical detail density that breaks the author's voice
- Block quotes without attribution or integration

**Action:** Reword to match the surrounding narrative voice. Preserve the factual content but make it sound like the *author* wrote it, not like it was pasted from reference notes. Keep the author's characteristic directness and informality where that's the dominant voice.

### Category D: Organizational Scaffolding
Markdown headers, section markers, or inline notes that are structural reminders, not final content:
- `### Where to find X in notes`
- `--- INSERT THEORY HERE ---`
- `[from ch. 85 section II.1]`
- Headers that describe what should go in a section rather than titling it

**Action:** Either:
1. Replace with a proper transition sentence that achieves the same organizational purpose, OR
2. Remove entirely if the content that follows makes the organization obvious

---

## Phase 2: Execute Fixes

Work through the issue list **in document order** (top to bottom). For each fix:

1. **Read the surrounding context** (2-3 paragraphs around the issue) to understand the author's argument
2. **Source the needed information** from project files
3. **Write the fix** in the author's voice — direct, informal-academic, with clear economic reasoning
4. **Mark the fix** with an HTML comment `<!-- SLOP-FIX: [description of what was changed] -->` so the author can review

### Voice Guidelines
The author writes in a distinctive style:
- Direct and confident, not hedgy
- Uses concrete examples over abstract generalization
- Comfortable with informal phrasing in drafts ("this is exactly what X predicts")
- Thinks in terms of mechanisms and channels, not just correlations
- References papers by what they *do* for the argument, not just what they *say*

When rewording copy-paste chunks, match this voice. Don't make it sound like a textbook.

### Citation Format
- In markdown drafts: use `@authorYYYYdescriptor` Quarto syntax
- If the draft uses informal references ("Author 1988"), convert to proper key
- If a paper isn't in the bib, add it using the delegate protocol below

---

## Phase 3: Delegate

After fixing what you can, produce a delegation report for remaining issues:

### For missing citations/bib entries:
→ Hand to **bib-fixer** agent (list of entries to add)

### For missing paper content (paper not in notes):
→ Flag for user: "Paper X is referenced but has no notes on disk. Need to read or web-source."

### For claims that need PDF verification:
→ Flag for user: "Claim about X on p.Y needs verification against original PDF."

### For organizational decisions:
→ Flag for user: "This section might belong in §2.1 vs §2.3 — your call."

---

## Phase 4: Output

Write the cleaned draft to the same file (overwrite), preserving the original in a backup:
1. Copy original to `[filename]_BACKUP_[date].md`
2. Write cleaned version with `<!-- SLOP-FIX -->` markers
3. Produce a summary report:

```
SLOP-FIX REPORT: [filename]
Date: [date]

Issues found: [count]
  Category A (TODO/placeholders): [count] found, [count] resolved, [count] flagged
  Category B (uncited claims): [count] found, [count] resolved, [count] flagged
  Category C (copy-paste chunks): [count] found, [count] reworded
  Category D (scaffolding): [count] found, [count] resolved

Delegation needed:
  - [list of items that need user decision or additional sourcing]

Bib entries to add:
  - [list of missing bib keys]

Ready for Stage 2 (narrative-reviewer): YES/NO
  If NO: [what's blocking]
```

---

## What This Skill Does NOT Do

- **Does not restructure arguments** — that's the organizer (stage 6)
- **Does not improve prose quality** — that's McCloskey (stage 7)
- **Does not compress length** — that's the compressor (stage 8)
- **Does not check economic logic** — that's the domain-reviewer (stage 5)
- **Does not move content to Overleaf** — that's condense-to-overleaf (stage 4)

This skill is the **minimum viable cleanup** so that everything downstream has clean input.
