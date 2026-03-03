# Growth Intern — Checkpoint (resume from here)

## Where we are

We’ve drafted and agreed a high-level spec for an Instagram “Growth Intern” that:

- Is **draft-only** (no auto-post)
- Pulls content ideas from the daily newsletter via **Beehiiv RSS**
- Prioritizes **brand reputation and correctness** over polish
- Uses an explicit **anti-hallucination policy** with receipts

## Source feed

- Beehiiv RSS: `https://rss.beehiiv.com/feeds/xuxM39ExvC.xml`

## Non-negotiables (brand safety)

- Typos are acceptable; **confident wrong facts are not**.
- If a claim can’t be supported directly by the newsletter text:
  - **omit it**, or
  - **flag it** for human review (don’t ship it)

Every draft must include an internal **Evidence Ledger**:

- claim → quoted source excerpt → source URL → confidence (High/Medium)

## Content pillars (locked)

1) Evergreen explainers  
2) Numbers  
3) Polls (Stories-first)  
4) History echo (only if source supports it)  
5) Newsletter content carousels (direct repackaging)

## Canonical spec doc

- `docs/plans/2026-03-03-growth-intern-design.md`

## What’s still open (next decisions)

- **Carousel visual system**: templates, fonts, colors, layout rules, default slide count
- **CTA/link strategy**: link-in-bio vs Stories link sticker vs “comment keyword”

## Next step options (choose one)

- **A) Finish the spec**: lock visual system + CTA/link strategy.
- **B) Run a 1-week pilot**: generate 4 posts/week (draft-only) with Evidence Ledgers.
- **C) Implement a tiny tool**: RSS → IdeaCards → weekly plan → draft bundles (still approval-only).

## How to resume later (exactly)

Tell the assistant:

- “Resume Growth Intern from checkpoint.”
- “Open `docs/plans/2026-03-03-growth-intern-checkpoint.md` and continue with option A/B/C.”

