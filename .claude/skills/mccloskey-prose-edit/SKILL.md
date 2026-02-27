---
name: mccloskey-prose-edit
description: Run McCloskey prose quality audit on Overleaf manuscript section(s). Critic scores, fixer suggests, critic re-grades. Output to QMD.
disable-model-invocation: true
argument-hint: "[section number or 'all'] — e.g., '2' for 2-background.tex, 'all' for full paper"
allowed-tools: ["Read", "Grep", "Glob", "Write", "Task", "Edit"]
---

# McCloskey Prose Editor

Run a McCloskey *Economical Writing* compliance audit on Overleaf manuscript sections. Uses the adversarial critic→fixer→critic pattern: the critic scores, the fixer suggests, the critic re-grades. All output goes to the Quarto book — **never to Overleaf files**.

## Arguments

- `[section number]` — e.g., `2` for `2-background.tex`, `5` for `5-results.tex`
- `all` — audit all manuscript prose sections

## Workflow

### Step 1: Load Inputs

```
Read $OL/Sections/[N]-[name].tex          # The manuscript section
Read docs/mccloskey_rules_summary.md      # The rulebook (21 rules, severity tiers)
```

($OL is defined in your project's CLAUDE.md — typically the Overleaf sync directory.)

If argument is `all`, read all prose sections and process each sequentially.

### Step 2: Spawn McCloskey Critic (Round 1)

```python
Task(
    subagent_type="general-purpose",
    model="sonnet",
    prompt="""You are the McCloskey Critic agent.
    [Include full agent instructions from .claude/agents/mccloskey-critic.md]

    RULES SUMMARY:
    [Include docs/mccloskey_rules_summary.md content]

    MANUSCRIPT TEXT:
    [Include the section text]

    Produce a MCCLOSKEY COMPLIANCE AUDIT report with score and violation list."""
)
```

**Early exit:** If score ≥ 90, write a brief "passing" note to `replication_book/prose_revisions.qmd` and skip the fixer.

### Step 3: Spawn McCloskey Fixer (Round 1)

```python
Task(
    subagent_type="general-purpose",
    model="sonnet",
    prompt="""You are the McCloskey Fixer agent.
    [Include full agent instructions from .claude/agents/mccloskey-fixer.md]

    RULES SUMMARY:
    [Include rules summary]

    CRITIC'S VIOLATION REPORT:
    [Include critic output from Step 2]

    ORIGINAL MANUSCRIPT TEXT:
    [Include the section text]

    Generate SUGGESTED REVISIONS for every violation."""
)
```

### Step 4: Re-Spawn Critic (Round 2 — Re-Grade)

```python
Task(
    subagent_type="general-purpose",
    model="sonnet",
    prompt="""You are the McCloskey Critic agent performing a RE-GRADE.
    [Include critic instructions]

    ORIGINAL TEXT:
    [Include section text]

    FIXER'S SUGGESTED REVISIONS:
    [Include fixer output from Step 3]

    Evaluate each suggestion AS IF it were applied to the manuscript.
    Produce a revised MCCLOSKEY COMPLIANCE AUDIT with updated score.
    For each suggestion, state: APPROVED (resolves violation) or REJECTED (doesn't improve or introduces new violation)."""
)
```

**If score ≥ 90:** All approved suggestions are final. Go to Step 6.
**If score < 90:** Proceed to Step 5 (Round 2 fixer).

### Step 5: Fixer Round 2 (if needed)

```python
Task(
    subagent_type="general-purpose",
    model="sonnet",
    prompt="""You are the McCloskey Fixer agent, ROUND 2.
    [Include fixer instructions]

    CRITIC'S RE-GRADE REPORT:
    [Include critic Round 2 output — includes APPROVED/REJECTED for each suggestion]

    ORIGINAL TEXT + YOUR ROUND 1 SUGGESTIONS:
    [Include both]

    Refine ONLY the REJECTED suggestions. Keep APPROVED suggestions as-is.
    Generate improved revisions addressing the critic's specific objections."""
)
```

After Round 2, spawn the critic one final time to score. Take whichever round scored higher. **Cap at 2 rounds** to control costs.

### Step 6: Write Results to QMD

Write the final bipartite problem→solution commentary to:
`replication_book/prose_revisions.qmd`

Format per the fixer agent's QMD writing specification: each violation gets a "Current text" block, a "Problem" explanation, and an "Approved revision" block.

If this is not the first audit (chapter already has content), **append** a new section header for the current run with date:

```markdown
---

## Audit Run: [YYYY-MM-DD]

### Section [N]: [Title] — Score: [Initial]/100 → [Final]/100
...
```

### Step 7: Log

Append to session log with `[writing]` subclassification:

```
[writing] Section 2: scored 78/100, 4 cardinal + 2 major violations. Revisions written to replication_book/prose_revisions.qmd
```

## Example Invocation

```
/mccloskey-prose-edit 2
```

This audits `$OL/Sections/2-background.tex` against all 21 McCloskey rules, generates suggested revisions, and writes the results to the Quarto book.

## File Dependencies

| File | Role |
|------|------|
| `docs/mccloskey_rules_summary.md` | Rules reference (read by both agents) |
| `.claude/agents/mccloskey-critic.md` | Critic instructions |
| `.claude/agents/mccloskey-fixer.md` | Fixer instructions |
| `$OL/Sections/*.tex` | Input (READ-ONLY — never modified) |
| `replication_book/prose_revisions.qmd` | Output (append-only) |

## Key Constraints

- **Manuscript protection applies.** This skill READS Overleaf files but NEVER writes to them.
- **Cost cap:** Maximum 2 critic→fixer rounds per section. ~4-5 sonnet calls per section.
- **The user applies revisions.** This skill produces suggestions. The user decides what to adopt.
- **Scoring is per-violation.** Multiple instances of the same rule = multiple deductions.
