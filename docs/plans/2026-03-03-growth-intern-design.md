# Growth Intern (Instagram) — High-level spec

## Summary

Build a **draft-only** “Growth Intern” that produces Instagram-ready drafts (**Reels, Carousels, Stories, Static**) for the International Intrigue brand. It should generate content in the **full Intrigue voice**, optimized for **audience growth** (global professionals age 20–40), at **4 posts/week**, and always require human approval before publishing.

## Goals (30 days)

- Grow reach and follows among global professionals (20–40), including humanities + finance students
- Turn daily “newsroom output” into IG-native packaging (not just reposts)
- Reduce the time cost of ideation + first drafts

## Constraints / guardrails

- Draft-only: **no auto-post**
- No hallucinated facts: every claim must be traceable to a source excerpt
- Brand voice: follow the Intrigue voice rules (source of truth lives in your Intrigue voice repo)
- Safety: avoid defamation, doxxing, hate, graphic violence; soften/omit sensitive details as needed
- **Brand-rep > polish**: we prefer **typos over wrong facts**. If a claim can’t be supported from the source text, **omit it** or **flag it** for human review.

## What AI is best at here (and what it struggles with)

### Best at

- **Repurposing**: turning one daily briefing into multiple IG-native angles (Reel script, carousel slides, story frames).
- **Synthesis + summarisation**: compressing long text into tight hooks, beats, and takeaways.
- **Evergreen explainers**: “what is X / why does X matter / how does X work” content that saves well.
- **Variation at scale**: generating 5–20 headline/hook options, CTA options, or slide order options without extra effort.
- **Consistency to templates**: reliably following your structure (e.g., Hook → 3 facts → Why it matters → CTA).
- **Light curation**: ranking candidate angles from the RSS feed by clarity + shareability (not by “truth”).
- **Copy editing**: shortening, tightening, and re-voicing drafts into the Intrigue style (given clear voice rules).

### Struggles with

- **Maps / visuals**: generating accurate maps, charts, or data viz (high error risk).
- **Truth verification**: it can *sound* correct while being wrong; it must not invent numbers, dates, names, or causal claims.
- **Over-analysis / “model-brain” takes**: confident theorising beyond the source text; keep it anchored to cited excerpts.
- **Trend sense**: it doesn’t reliably know what’s “currently popping off” on IG without real performance data.
- **Taste + editorial judgment**: what’s truly on-brand, what’s too spicy, what’s too insider, what’s too dull.
- **Risk calls**: defamation, sensitive conflict details, and real-world safety nuance still need human judgment.

## Anti-hallucination policy (brand-critical)

### The rule

The intern may only state factual claims that are **directly supported by the source newsletter text** (RSS item `description` / `content:encoded` or the fetched full post if we later add that).

If it can’t be supported, the intern must do one of:

- **Omit the claim**, or
- **Rewrite it as attribution** (“In today’s Intrigue briefing, we flagged…”) *and still include the excerpt*, or
- **Flag for human review** (explicitly mark as “needs verification” and do not include in on-screen text).

### Claim hygiene (what’s allowed vs not)

- **Allowed**: summarising what the newsletter itself says (with citation).
- **Not allowed**: introducing new facts, numbers, dates, names, or “obvious” background context that isn’t present in the source text.
- **No predictions**: avoid “will”, “is likely”, “inevitable”, unless the source explicitly frames it that way.
- **Numbers**: if a number appears, copy it **verbatim with its qualifier** (e.g., `~`, ranges, “reported”, “initially”, “as of Monday”). Never “round”, “update”, or “estimate”.

### Confidence labels (required)

Every draft must assign one of:

- **High**: the exact claim (or its key number/name) appears verbatim in the source excerpt.
- **Medium**: the claim is a close paraphrase of a clearly stated source sentence, with no new entities/numbers introduced.
- **Low**: anything that might be inference. Default action: **remove** or **flag for review**.

## Required evidence in outputs (“receipts”)

For each post draft, include an **Evidence Ledger** (not for IG publishing, for internal review):

- `claim`: the exact sentence that will appear in the IG draft (on-screen or caption)
- `source_excerpt`: 1–3 sentences quoted verbatim from the newsletter
- `source_url`: link to the issue
- `confidence`: High/Medium

If a post can’t be supported with an Evidence Ledger, it should not ship.

## Inputs (sources of truth)

### 1) Daily newsletter as “source ideas” (Beehiiv RSS)

The intern should pull topic seeds from the daily newsletter via the Beehiiv RSS feed:

- **RSS feed**: `https://rss.beehiiv.com/feeds/xuxM39ExvC.xml`

**Ingestion checklist**

- **Fetch cadence**: once daily (e.g. morning local time) + on-demand manual run
- **Lookback window**: last 7 days (or last 10 items, whichever is larger)
- **Dedupe key**: prefer `<guid>`; fallback to `link`; fallback to `title + pubDate`
- **Store raw**: keep the original RSS item payload for traceability/debugging
- **Failure mode**: if RSS fetch fails, reuse last successful cache and flag “stale sources”

**What to extract from each RSS item**

- **Metadata**: `title`, `pubDate`, `link`, `category[]`
- **Summary cues**: `description`
- **Visual cue** (if present): `enclosure[@url]` (thumbnail)
- **Full content** (if present): `content:encoded` (HTML)

**Normalization**

- Convert `content:encoded` HTML → plain text (preserve headings + lists)
- Strip sponsor blocks / boilerplate when possible
- Create a `NewsletterIssue` object:

```json
{
  "id": "guid-or-link",
  "published_at": "ISO8601",
  "title": "...",
  "url": "...",
  "categories": ["..."],
  "description": "...",
  "thumbnail_url": "...",
  "body_text": "..."
}
```

**Idea mining (turn one issue into multiple IG prompts)**

For each `NewsletterIssue`, extract 5–12 “idea atoms”:

- **Hook**: 1–2 sentence scroll-stopper
- **Core claim(s)**: 3–6 bullet facts *from the issue text*
- **Why it matters**: 1–2 lines
- **Angle options** (choose 2–4):
  - “Zoom out” geopolitical frame
  - “Explainer in 60s”
  - “Myth vs reality”
  - “3 implications”
  - “Map/picture prompt” (if issue contains a strong visual)
  - “Quiz/poll prompt” (for Stories)
- **CTA**: “Read the full briefing” + link-in-bio style phrasing (no hard sell)

Store these as `IdeaCard`s:

```json
{
  "source": "beehiiv_rss",
  "source_issue_url": "...",
  "source_issue_title": "...",
  "published_at": "ISO8601",
  "hook": "...",
  "facts": ["..."],
  "angle_options": ["..."],
  "ig_formats": ["reel", "carousel", "story", "static"],
  "risk_flags": ["graphic", "defamation", "politics-sensitive"],
  "confidence": "high|medium|low"
}
```

**Selection rule**

Each week, the intern proposes a draft plan by selecting **4 IdeaCards** (one per post), aiming for:

- 2 “evergreen explainer” angles (teachable, saves well)
- 2 “timely” angles (fast-moving, high curiosity)
- Mix of regions + topics; avoid repeating the same conflict all week

### 2) Brand voice rules (Intrigue voice repo)

The intern must use the Intrigue voice repo as a hard constraint when drafting:

- Captions, on-screen text, and scripts should follow the voice rules
- If the voice rules conflict with “generic IG best practice,” voice wins

## Outputs

For each selected weekly post, generate:

- **Reel draft**: 20–45s script + on-screen text beats + B-roll suggestions + caption + hashtags
- **Carousel draft**: 6–10 slides (each slide: headline + 1–2 lines) + caption + alt text
- **Story set**: 3–6 frames including at least one poll/quiz sticker prompt + copy
- **Static post** (optional): single-slide “hook + takeaway” + caption

All drafts include:

- **Source citations**: lines/quotes pulled from the newsletter issue text
- **“No-hallucination check”**: any numeric/stat claim must be directly sourced
- **Evidence Ledger**: claim → excerpt → link → confidence (required)

### Weekly roundup (recommended recurring format)

Once per week, generate a “Weekly roundup” pack sourced only from the last 7 days of RSS items:

- **Carousel**: “5 things you missed this week” (5 slides + title slide + wrap-up slide)
- **Stories**: 5 frames, each with a poll (“Surprised?” / “Good idea?” / “Who wins?”)
- **Caption**: one-paragraph summary + 5 bullet headlines + CTA to newsletter

Rules:

- Each item must link back to its source issue (internal traceability even if IG doesn’t show links).
- Prefer **breadth** (regions/topics) over going deep on one conflict.

## Workflow (weekly cadence)

- **Daily (optional, low effort)**: ingest RSS → generate IdeaCards for that day’s issue
- **Weekly (required)**:
  - Pull last 7 days of RSS items
  - Generate IdeaCards
  - Propose a 4-post content plan (Mon–Sun spacing, or your preferred days)
  - Produce drafts for approval

## Content pillars (locked)

The intern should consistently produce drafts across these pillars (mix week to week):

1) **Evergreen explainers**
- Goal: high-saves, durable posts that teach recurring concepts.
- Format: carousel or Reel: “What is X?” / “How does X work?” / “Why does X matter?”
- Rule: if the explainer includes any factual examples, they must be sourced (Evidence Ledger).

2) **Numbers**
- Goal: one striking, sourced number → meaning + signal.
- Format: single-slide or 6–8 slide carousel (“The number”, “What it measures”, “Why it matters”).
- Rule: copy the number and its qualifier **verbatim** from the newsletter (no rounding/updates).

3) **Polls (Stories-first)**
- Goal: audience calibration + engagement while staying safe.
- Format: 3–6 story frames, each with a poll/quiz prompt.
- Rule: polls must be grounded in the newsletter framing; avoid prompts that require unsourced claims.

4) **History echo**
- Goal: light analogy when the newsletter itself invites it.
- Format: carousel: “This echoes X (here’s why)”.
- Rule: only allowed when the source text explicitly references the historical comparison (or provides the facts needed to state the comparison without inference). Otherwise: omit or flag for human review.

5) **Newsletter content carousels (repackaging)**
- Goal: easiest, highest-throughput repackaging of the daily briefing.
- Format: 6–10 slide carousel derived directly from one issue:
  - Slide 1: hook
  - Slides 2–6: key facts (sourced)
  - Slide 7: “Why it matters”
  - Slide 8: CTA (“Read today’s briefing”)
- Rule: prefer quoting/paraphrasing the issue over adding new context.

## Approaches (2–3 options) + recommendation

### Approach A — RSS-only idea mining (fastest)

- Use only the RSS feed’s `title/description/content:encoded` to build IdeaCards.
- Pros: simplest, low fragility, no extra fetching
- Cons: if `content:encoded` is truncated, idea atoms may be weaker

### Approach B — RSS + fetch full post page for richer extraction

- Use RSS `link`, then fetch the post page and extract full text (and images) for IdeaCards.
- Pros: richer facts, better quotes, better visuals
- Cons: more brittle parsing; needs robust HTML extraction + rate limits

### Approach C — Hybrid: RSS automation + manual “pin” from editor

- RSS ingestion runs daily; you (human) “pin” the best 2–3 issues/angles each week.
- Pros: maximum editorial control, avoids off-brand topics
- Cons: requires a small manual step

**Recommendation**: start with **Approach A**, but design the data model so we can upgrade to **B** later without reworking the pipeline (same `NewsletterIssue` / `IdeaCard` objects).

## Open questions (to finalize in this doc)

- Preferred “visual system” for carousels (template style, fonts, colors)
- Link strategy: link-in-bio vs “comment keyword” vs Stories link sticker

