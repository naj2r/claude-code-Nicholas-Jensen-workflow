# /consolidate — Cherry-Pick Additions from Source Documents

## Purpose
Compare the current draft against earlier drafts, co-author notes, and Overleaf sections. Cherry-pick content that merits inclusion but isn't there yet. Adversarial process: fixer proposes, critic evaluates, fixer rebuts, critic decides.

## Trigger
Run when the user asks to consolidate, merge, or compare drafts. Default: reads the latest draft in the relevant working directory and compares against provided source documents.

## Inputs
- **Current draft**: Latest draft QMD or markdown file (user-specified, or most recent in working directory)
- **Comparison documents** (user provides, typically):
  - Earlier Overleaf `.tex` sections
  - Co-author notes/drafts
  - Co-author `.bib` files for citation discovery
  - Previous QMD drafts

## Process

### Step 1: Consolidator-Fixer (opus)
Spawn `consolidator-fixer` agent via Task tool with:
- The full text of the current draft
- The full text of each comparison document
- Instruction: identify content NOT in the current draft that would improve it

Output: Proposed additions saved to `quality_reports/consolidation/[date]_fixer_proposals.md`

### Step 2: Consolidator-Critic — Round 1 (sonnet)
Spawn `consolidator-critic` agent with:
- The current draft
- The fixer's proposals

Output: Verdicts (APPROVED/REJECTED/MODIFIED) saved to `quality_reports/consolidation/[date]_critic_round1.md`

### Step 3: Consolidator-Fixer — Rebuttal (opus)
Spawn `consolidator-fixer` again with:
- The critic's rejections only
- Instruction: make a case for why the critic is wrong on each rejection

Output: Rebuttals saved to `quality_reports/consolidation/[date]_fixer_rebuttal.md`

### Step 4: Consolidator-Critic — Round 2 (sonnet)
Spawn `consolidator-critic` again with:
- The fixer's rebuttals
- Instruction: reconsider each rejection. Change verdict only if rebuttal identifies something missed.

Output: Final verdicts saved to `quality_reports/consolidation/[date]_critic_final.md`

### Step 5: Apply Approved Changes
- Apply all APPROVED and MODIFIED additions to a new draft version (next version number in the series)
- Add any new bib entries to the project bibliography file
- Document the full adversarial exchange in a new QMD subsection at the bottom of the draft (so the user can arbitrate disagreements)

### Step 6: Verify
- Run `bib-checker` (haiku) on updated draft
- Verify render with `quarto render`

## Rules
- Never treat non-co-author writing as the user's voice without permission
- Additions are ADDITIVE — don't delete or replace existing content unless the replacement is clearly better
- Both fixer and critic document their arguments — the user is the arbiter/tiebreaker
- If critic deems no further changes after Round 2, the process ends
- Max 2 rounds of adversarial exchange
- New bib entries follow `authorYYYYdescriptor` format

## Model Assignment
- consolidator-fixer: **opus** (needs deep economic reasoning to argue for inclusions)
- consolidator-critic: **sonnet** (lighter-touch gatekeeper; asymmetric by design)
- bib-checker: haiku

## Sequencing Rule
The consolidator is a **bottleneck**. No downstream pipeline stages (narrative-reviewer, proofreader, domain-reviewer, organizer, McCloskey, compressor) should run until the consolidator process is fully complete. This prevents idiosyncratic timing of different proposed revisions from contaminating the draft.
