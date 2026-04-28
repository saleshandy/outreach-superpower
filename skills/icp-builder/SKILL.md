---
name: icp-builder
description: Use when user wants to build, refine, version, or tighten an Ideal Customer Profile for cold outreach. Triggers on "build my ICP," "who should I target," "tighten my ICP," "create segment cards," "ICP for outbound," or "v2 my ICP."
---

# ICP Builder

Run a structured 14-step interview to produce a cold-email-usable ICP. One question per message. Generate ICP v1 (fast) -> ICP v2 (tight after 50-100 sends).

## Inputs

- **Optional (helpful):** `company.md` from `strategy-architect` - skip pre-fillable questions when confidence is high/medium.
- **Optional:** existing `icp.md` - triggers v2 mode (if v1 exists) or lite-upgrade (if `source: strategy-architect-lite`).

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-saleshandy-superpowers`.

All file reads/writes happen in `outreach-workspace/<campaign>/`.

## Operating rules

1. **One question per message.** Never bundle multiple questions in one turn.
2. **Suggestions every question.** Always offer 2-3 numbered options + "or reply with your own answer." Think hard about defaults from `company.md` when available.
3. **Visible progress tracker.** Format: `Step X / 14` (or `Step X / N` if some are pre-filled and skipped, e.g. `Step 3 / 10`). Show this on every question.
4. **Tighten vague answers.** If a reply is vague (e.g. "tech companies", "everyone", "small business"), ask one focused follow-up before moving on.
5. **Accept "skip".** If the user types "skip", move on and record the field as `[SKIPPED]` in the final output.
6. **Tables in output.** ICP Summary, Segment Cards, Targeting Filters, and Angle Kit all render as markdown tables. The user has explicitly asked for tables.

## Process

### Step 0 - Detect mode

Read the workspace before asking anything:

- If `icp.md` exists with frontmatter `version: v1` AND `source: icp-builder-interview` -> **v2 mode** (jump to Step A).
- Else if `icp.md` exists with `source: strategy-architect-lite` (and `needs_tightening: true`) -> **lite-upgrade mode**: run the full v1 interview but pre-fill heavily from the lite ICP and `company.md`. On write, replace the lite file (move it to `icp.lite.v1.md` first) and set `source: icp-builder-interview`.
- Else -> **fresh v1 interview**.

Also read `company.md` if present. Note the `confidence` flag:

- `confidence: high` or `medium` -> Steps 1, 2, 5, 13 are pre-fillable. Show the pre-filled answer and ask only for confirmation ("Confirm or edit?").
- `confidence: low` -> re-ask from scratch even if `company.md` has data; the extracted values aren't trustworthy.

### v1 interview (14 steps - preserve exactly)

For each step, output:

- The progress tracker (`Step N / 14`, or adjusted denominator if pre-filled steps were skipped)
- The question
- 2-3 numbered suggestions (use `company.md` context when relevant)
- The line: *"Reply with a number, or share your own answer."*

The 14 questions:

| # | Question |
|---|---|
| 1 | Tell me your website or what you're selling (1 line) |
| 2 | Who is it for today? (current paying users, if any) |
| 3 | What problem do they pay you to solve? (in their words) |
| 4 | What outcome do they want? (measurable) |
| 5 | Your best 10 customers: what do they have in common? (industry/size/geo) |
| 6 | Who is a bad fit? (who churns / complains / never buys) |
| 7 | What's the "why now"? (trigger events) |
| 8 | Who feels the pain daily? (job titles) |
| 9 | Who approves budget? (job titles) |
| 10 | Typical price point / contract size? (range is fine) |
| 11 | Sales motion: self-serve, demo, or both? |
| 12 | Key competitors or alternatives they use today? |
| 13 | Proof: what can you credibly claim? (results, logos, numbers) |
| 14 | Where will you source leads? (Saleshandy/LinkedIn/CSV/etc.) |

**Pre-fill rules** (when `company.md` exists with `confidence: high|medium`):

- Step 1 (website/product) - pre-fill from `company.md` value prop / services. Confirm only.
- Step 2 (current paying users) - pre-fill from extracted ICP segments / personas.
- Step 5 (firmographics common to best customers) - pre-fill from extracted ICP segments / size band / geo.
- Step 13 (proof) - pre-fill from extracted proof points.

For each pre-filled step, output: *"From `company.md`: <value>. Confirm, or share an edit."* Adjust the denominator (e.g. `Step 3 / 10` if 4 steps were pre-filled and confirmed in a single batch).

If `confidence: low`, ignore pre-fill and ask all 14 from scratch.

**Vague-answer follow-up triggers:**

- Single-word industry replies ("SaaS", "tech", "agencies")
- Replies that don't include any number, role, or specific descriptor
- Replies under 5 words on Steps 3, 4, 5, 7, 8, 9

When triggered, ask one tightening question (e.g. *"Tech is broad - which subset? B2B SaaS, dev tools, fintech, ecommerce platforms?"*), then continue.

### Step A - v2 mode (5-step tightening interview)

If v2 mode triggered in Step 0, ask first:

> *Generating v2 (tightening). Share reply data from your last 50-100 sends - which segments responded, which didn't, what objections came up, any new patterns?*

Then run this 5-step flow (one question per message, with 2-3 suggestions each):

| # | Question |
|---|---|
| 1 | Which segment had the highest reply rate? |
| 2 | Which segment(s) should we drop? |
| 3 | Best new angle that emerged from replies? |
| 4 | New disqualifiers learned? |
| 5 | New trigger events you noticed? |

After Step 5, regenerate the 5-section deliverables (Step Z) with the tightened data. Move the existing `icp.md` to `icp.v1.md` before writing the new file with `version: v2`.

### Step Z - Final deliverables

After the last interview question, output **all five sections in chat AND write them into `icp.md`**:

**A) ICP Summary**

| Field | Value |
|---|---|
| Who it's for | ... |
| Who it's not for | ... |
| Primary pain | ... |
| Primary outcome | ... |
| Why now (trigger) | ... |
| Buying roles | ... |
| Proof points | ... |

**B) Segment Cards (3)** - one table per segment. Keep segments few (1 primary, 2 secondary).

| Field | Value |
|---|---|
| Segment name | ... |
| Firmographics | ... |
| Technographics | ... |
| Common pain | ... |
| Trigger events | ... |
| Messaging angle | ... |
| Objections + counters | ... |
| Disqualifiers | ... |
| Example titles to target | ... |

**C) Targeting Filters** (Saleshandy / LinkedIn ready)

| Filter | Value |
|---|---|
| Industry | ... |
| Company size | ... |
| Geography | ... |
| Keywords | ... |
| Job titles | ... |
| Tech stack signals | ... |
| Funding/intent signals | ... |

**D) Cold Email Angle Kit (5 angles)** - one table per angle.

| Field | Value |
|---|---|
| Angle name | ... |
| Subject lines (3) | 1. ... / 2. ... / 3. ... |
| Opening patterns (2) | 1. ... / 2. ... |
| CTA options (2) | 1. ... / 2. ... |

**E) Disqualifiers** - bulleted list of who NOT to target (industries, sizes, signals, behaviors).

Every segment must pass the test: *"Can I find these people with filters in Saleshandy/LinkedIn?"* If not, tighten before writing.

### Step Final - Write file

Write `outreach-workspace/<campaign>/icp.md`. Frontmatter:

```yaml
---
version: v1 | v2
generated_at: YYYY-MM-DD
segments: 3
disqualifiers: <count>
source: icp-builder-interview
---
```

Versioning rules:

- v1 fresh write -> just create `icp.md`.
- v2 -> first move existing `icp.md` to `icp.v1.md`, then write the new v2 file.
- Lite-upgrade -> move existing lite `icp.md` to `icp.lite.v1.md` first, then write the new full v1 file with `source: icp-builder-interview`.

End the run with: *"ICP v1 saved. Run me again after 50-100 sends to generate ICP v2."*

## TodoWrite usage

For the 14-step interview, create a TodoWrite list at the start of Step 1 with one todo per question:

- `Step 1 - website / product`
- `Step 2 - current paying users`
- `Step 3 - problem they pay to solve`
- ... (through Step 14)
- `Generate deliverables (Step Z)`
- `Write icp.md`

Mark a todo `in_progress` when you ask the question, `completed` when the user answers. Pre-filled steps that the user confirms in one batch can be marked `completed` together. This lets the skill resume mid-interview if context switches.

For v2 mode (5-step flow), use a shorter TodoWrite list with one todo per tightening question + final write.

---

*Run me again after 50-100 sends to generate ICP v2.*
