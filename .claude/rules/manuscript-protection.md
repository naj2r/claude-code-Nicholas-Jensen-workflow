# Manuscript Protection Rule

**The user writes the paper. Claude does not.**

## Scope: What's Protected vs. What's Autonomous

| Layer | Files | Claude Autonomy |
|-------|-------|-----------------|
| **Overleaf (protected)** | `$OL/Sections/*.tex` (all prose sections), `$OL/presentation.tex` | Mechanical fixes only — no substantive writing without explicit instruction |
| **Overleaf (unprotected)** | `$OL/bibliographyCiteDrive.bib` (or project .bib file), `$OL/files/tab/**/*.tex` (Stata/R tables), `$OL/preamble.tex`, `$OL/Sections/figures-and-tables.tex`, `$OL/Sections/appendix.tex` | Full autonomy — these are infrastructure |
| **Quarto book** | `replication_book/*.qmd` | **Full autonomy** — Claude writes, expands, and edits freely |
| **Session logs & notes** | `quality_reports/**`, `articles/**`, `explorations/**` | **Full autonomy** — internal working documents |

($OL is defined in your project's CLAUDE.md — typically the Overleaf sync directory.)

**The Quarto book is Claude's workspace.** Write detailed notes, draft prose, expand chapters — all fine. The user draws from these when writing the Overleaf paper themselves.

## Protected Files

The **prose sections** of the paper are manuscript-protected. These are all `$OL/Sections/*.tex` files that contain narrative prose (introduction, background/literature, data, methods, results, conclusion, etc.).

Also protected: `$OL/presentation.tex` (Beamer slides).

**NOT protected** (infrastructure — Claude has full autonomy):
- `figures-and-tables.tex` (or equivalent) — `\input{}` wrappers for Stata/R tables
- `appendix.tex` (or equivalent) — `\input{}` wrappers for appendix tables

These files contain no prose, only `\input{}` commands. Claude may add, remove, reorder, or fix table inclusions freely.

## What Claude MAY Do to Protected Files (Without Asking)

| Action | Example | Why OK |
|--------|---------|--------|
| Fix broken BibTeX keys | `author2009wrong` → `author2009right` | Mechanical — no prose changed |
| Fix LaTeX compilation errors | Missing `}`, undefined `\ref` targets | Mechanical — preserves user's text |
| Add `\input{}` for tables | Wiring a new Stata/R table into the paper | Infrastructure — no prose changed |
| Fix `\sym` or `\scalebox` issues | Table rendering commands | Infrastructure |

## What Claude MUST NOT Do (Without Explicit User Instruction)

| Action | Example of Violation | What Should Have Happened |
|--------|---------------------|--------------------------|
| Write or rewrite prose | Replacing Lorem ipsum with a TODO stub | Ask the user or leave it alone |
| Uncomment section headers | `% \section{Results}` → `\section{Results}` | Leave commented unless asked |
| Add TODO comments | Inserting `% TODO: Write results...` | Put suggestions in session log instead |
| Delete user content | Removing placeholder text | Comment it out at most, or ask first |
| Change section labels | `section-results1` → `section-results` | Ask the user which label they want |
| Draft paragraphs into .tex | Writing contribution statements | Put draft in markdown/QMD, user moves it |

## The Test

Before editing any protected file, ask yourself:

> **Did the user explicitly request this specific change in this conversation?**

- If YES → proceed.
- If NO but it's a mechanical fix (broken key, compilation error) → proceed.
- If NO and it involves prose, structure, or content → **STOP. Do not edit.**

Put the suggestion in a session log, markdown note, or QMD chapter instead. The user will move it into the paper themselves.

## Where to Put Draft Content Instead

| What You Want to Write | Where to Put It |
|----------------------|-----------------|
| Draft paragraph for a section | `quality_reports/drafts/section-N-draft.md` |
| Suggested edits to existing prose | Session log or chat output |
| Notes for the user to reference | `explorations/` or `quality_reports/` |
| Citation suggestions | `articles/` or chat output |

## Integration with own-writing-check Skill

Run `/own-writing-check` after any session that touched Overleaf files. The `manuscript-critic` agent will audit all changes and flag any that violated this rule.
