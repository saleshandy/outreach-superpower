# Usage Guide

Per-skill examples and transcripts for `outreach-superpower`. Skills are listed in the natural workflow order: router, strategy-architect, icp-builder, service-productizer, email-sequence-generator, email-auditor.

Every skill reads and writes to `./outreach-workspace/<campaign>/`, where `<campaign>` defaults to `default`. Pass `--campaign acme` in your message to scope to a different folder.

## using-outreach-superpower (router)

The router is the only skill that auto-loads. You will never invoke it by name; it activates on outreach-related phrasing and decides which sub-skill to run.

### When to use

You don't. The router fires automatically when you mention cold email, outreach, prospecting, ICP, target audience, sales sequences, email copy review, productizing a service, or any SDR/AE workflow.

### Inputs

- The user's first message in the session.
- The current working directory (used to locate `outreach-workspace/<campaign>/`).

### What you get

- A one-line announcement on first load: workspace path, target skill, list of existing artifacts.
- Auto-chaining: if you ask for a sequence but no `icp.md` exists, the router invokes `icp-builder` first, then resumes.

### Example transcript

```
You: I want to write cold emails for my company https://acme.com

Claude (router): Outreach Superpower loaded. Routing to strategy-architect.
                 Workspace: ./outreach-workspace/default/. Existing artifacts: none.
                 (strategy-architect now runs)
```

```
You: Generate a sequence for my mid-market SaaS segment.

Claude (router): No icp.md found. Running icp-builder first, then resuming
                 email-sequence-generator.
```

### Tips

- Phrase requests in plain English. The router matches on intent, not keywords.
- If two skills could fit, the router asks which you meant rather than guessing.
- Pass `--campaign acme-q2` anywhere in your message to scope all reads and writes to that campaign folder.

---

## strategy-architect

### When to use

You have a website URL and no other artifacts yet. Run this first to seed the workspace with extracted facts before any other skill.

### Inputs

- A website URL (or pasted homepage content if `WebFetch` fails).
- Optional: `--campaign <name>` to scope the output folder.

### What you get

Three files written to `outreach-workspace/<campaign>/`:

- `company.md` with frontmatter (`source`, `url`, `extracted_at`, `confidence: high|medium|low`) and a body listing industry, value prop, services, differentiators, proof points, and constraints.
- `strategy.md` with frontmatter (`goal`, `channels`, `success_metric`) and a body covering why-now triggers, entry CTA, objections, and sequence design recommendations.
- `icp.md` (lite version, frontmatter `source: strategy-architect-lite`, `needs_tightening: true`) with a summary and 3 candidate segments.

In chat: a 10-item confirmation card (value prop, ICP segments, personas, regions, proof, differentiators, objections, entry offer, constraints, success metric) for you to confirm or edit before files are written.

### Example transcript

```
You: Build an outreach plan from https://acme.com

Claude: (fetches homepage, pricing, customers, about pages)

        Please confirm or edit (reply with item number + change, or "all good"):

        1. Value prop: All-in-one workflow automation for ops teams
        2. ICP segments: Mid-market B2B SaaS, fintech ops, ecommerce ops
        3. Primary personas: Head of Operations, RevOps Manager
        4. Regions/language: US, UK, EU (English)
        5. Proof to cite: 200+ customers, 40% time savings (Bevy case study)
        ...
        10. Success metric: meetings [ASSUMPTION]

        Proceed with these? (yes / proceed with assumptions / make edits #1, #4...)

You: #2: drop ecommerce, add B2B services agencies. #10: SQLs.

Claude: (re-displays updated card) Proceed with these?

You: yes

Claude: Workspace written: company.md, strategy.md, icp.md (lite).
        Recommended next: run icp-builder to tighten the ICP, or
        email-sequence-generator to draft a sequence.
```

### Tips

- If `WebFetch` is blocked or returns thin content, paste the homepage and customers/case-studies pages directly. The skill notes `source: manual-paste` and proceeds.
- The `confidence` flag in `company.md` controls how much `icp-builder` pre-fills. High confidence skips up to 7 of the 14 interview steps.
- Re-running this skill on a workspace that already has a tightened `icp.md` (from `icp-builder`) preserves the tight version. The lite version is only ever overwritten if no full ICP exists yet.

---

## icp-builder

### When to use

You want a tight, segment-by-segment ICP that downstream skills can actually use to write copy. Either you have a fresh workspace from `strategy-architect`, or you want to upgrade an existing lite ICP, or you want to generate v2 after a real campaign.

### Inputs

- Optional but helpful: `company.md` from `strategy-architect`. Pre-fills questions when the confidence flag is high or medium.
- Optional: existing `icp.md`. If `version: v1` and `source: icp-builder-interview`, runs in v2 mode (tightening from real reply data). If `source: strategy-architect-lite`, runs the full v1 interview but pre-fills heavily from the lite version.

### What you get

A single `icp.md` written to `outreach-workspace/<campaign>/`, with frontmatter (`version`, `generated_at`, `segments`, `disqualifiers`, `source`, `primary_segment`, `buying_motion`, `campaign`) and five body sections:

- **A. ICP Summary** (table): who it's for, who it's not, primary pain, primary outcome, why now, buying roles, proof points.
- **B. Segment Cards (3)**: one table per segment with firmographics, technographics, pain, triggers, messaging angle, objections + counters, disqualifiers, example titles.
- **C. Targeting Filters**: industry, size, geo, keywords, job titles, tech stack signals, intent signals (Saleshandy / LinkedIn ready).
- **D. Cold Email Angle Kit**: 5 angles, each with 3 subject lines, 2 opening patterns, 2 CTA options.
- **E. Disqualifiers**: bulleted list of who NOT to target.

In chat: a step counter (`Step X / 14` or adjusted for pre-filled steps), one question per turn, with 2 to 3 numbered suggestions plus an "or share your own answer" option.

### Example transcript

```
You: Build my ICP.

Claude: (reads company.md, finds confidence: high, plans to skip 7 steps)

        Step 1 / 7
        From company.md: "All-in-one workflow automation for ops teams." Confirm
        with y, or share an edit.

You: y

Claude: Step 2 / 7
        From company.md: ICP segments are "Mid-market B2B SaaS, fintech ops,
        services agencies." Confirm with y, or share an edit.

You: y

Claude: Step 3 / 7 (Step 3 of original 14)
        What problem do they pay you to solve? In their words.

        1. Manual handoffs across tools eat their week.
        2. Ops sprawl creates audit risk and missed SLAs.
        3. They can't scale customer onboarding without hiring.

        Reply with a number, or share your own answer.

You: 1, plus they're tired of stitching Zapier.

Claude: Step 4 / 7
        What outcome do they want? Measurable.
        ...

(after step Z)

Claude: ICP v1 saved. Run me again after 50-100 sends to generate ICP v2.
```

### Tips

- When the skill asks a question and you already know the answer, type it directly. The numbered suggestions are scaffolding, not a forced multiple-choice.
- The skill flags vague answers ("tech companies", "everyone") and asks one tightening question before continuing.
- After 50 to 100 sends in Saleshandy, run the skill again. It detects the existing v1 and switches to v2 mode, asking only 5 questions about reply data.

---

## service-productizer

### When to use

You run an agency or service business and want to package one of your offerings as a productized offer for cold outreach. You need an offer name, scope, timeline, pricing guidance, proof, and a tight ICP before drafting any sequence.

### Inputs

- Required: `company.md` (the skill auto-handoffs to `strategy-architect` if missing).
- Optional: existing `productized-offer.md` (will be versioned), existing `icp.md` (the offer's ICP appends as a new segment card).

### What you get

A `productized-offer.md` written to `outreach-workspace/<campaign>/`, with frontmatter (`service`, `offer_name`, `generated_at`, `icp_source`, `icp_buying_motion`) and a body covering offer name, outcome one-liner, scope (bulleted with quantities), timeline and format, pricing guidance, proof point, and an offer-specific ICP block.

If `icp.md` already exists, the offer's ICP appends as a new segment card (existing segments are preserved, the `segments:` count increments).

### Example transcript

```
You: Productize my service.

Claude: (reads company.md, parses the Services list)

        From your website, here are the services I extracted. Which do you want
        to productize first?

        1. SEO: link-building campaigns (DR 50+ targets)
        2. Content marketing: weekly blog production
        3. Paid ads: Google + Meta campaign management

        Or type a custom service name with a 1-line description.

You: 1

Claude: (generates 3 offer name candidates)

        Pick or refine an offer name:
        1. "DR 50+ Backlink Sprint"
        2. "Editorial Authority Quarter"
        3. "90-Day Domain Authority Lift"

You: 1

Claude: (generates outcome one-liner, scope, timeline, pricing guidance, asks
        for proof, then runs the 5-question lightweight ICP)

        Offer saved to productized-offer.md. Want me to draft a 3-email cold
        sequence for this offer? (y/n)

You: y

Claude: Handing off to email-sequence-generator with goal: lead-gen and segment
        "DR 50+ Backlink Sprint - SaaS marketing leads."
```

### Tips

- Keep scope bullets concrete: "30 qualified backlinks from DR 50+ sites" beats "lots of high-quality backlinks".
- If you don't have a proof point yet, the skill marks it `[NEEDED]` and writes the file anyway. Add the proof later by editing the file directly.
- When `icp.md` exists, the appended segment card has `[NEEDED]` markers for fields the lightweight 5-question flow doesn't cover (technographics, objections + counters). Run `icp-builder` later to fill them in.

---

## email-sequence-generator

### When to use

You have an `icp.md` (and ideally a `company.md`) and you want a paste-ready Saleshandy sequence. Works for cold demo outreach, lead-gen, backlink outreach, recruitment, promotions, PR, partnerships, or nurture.

### Inputs

- Required: `icp.md` (auto-chained via the router if missing).
- Helpful: `company.md` for industry voice, proof points, differentiators.
- Helpful: `productized-offer.md` if you're pitching a productized service.

### What you get

A `sequence.md` written to `outreach-workspace/<campaign>/`, with frontmatter (`goal`, `icp_segment`, `email_count`, `generated_at`, `icp_version`, `offer_name`) and a body containing each email in this format:

```
## Email N
**Subject:** {spin}<A>|<B>{endspin}

**Body:**
Hi {First Name},

{spin}<opening A>|<opening B>{endspin}
...
{spin}<CTA A>|<CTA B>{endspin}

Best,
[Your Name]
[Your Company]
```

Word counts follow the rules: Email 1 is 150 to 220 words (full value prop), middle emails are 120 to 180 words (proof and objection handling), final emails are 50 to 100 words (bump-style CTA).

A footer points to `https://my.saleshandy.com/sequence` for the paste step.

### Example transcript

```
You: Generate a 4-email demo sequence.

Claude: (reads icp.md and company.md, cites the segment)

        What's the campaign goal?
        1. Book a Demo / Sales Call (auto: product hooks, calendar)
        2. Lead Generation
        3. Backlink Outreach
        ...
        Reply 1-8 or describe a custom goal.

You: 1

Claude: Pick a segment:
        1. Mid-market B2B SaaS Ops Leads [primary]
        2. Fintech Ops
        3. Services Agencies

        Reply with the number, or hit Enter to use the primary segment.

You: 1

Claude: Sequence length?
        1. 3 emails
        2. 4 emails (standard)
        3. 5 emails
        4. 6-7 emails

You: 2

Claude: (generates 4 emails with merge tags and spintext, writes sequence.md)

        Sequence written to sequence.md. Pulled segment "Mid-market B2B SaaS Ops
        Leads" from icp.md, proof points from company.md.
```

### Tips

- If `icp.md` doesn't have a Cold Email Angle Kit (e.g., you wrote it by hand), the skill generates 5 angles inline and shows them in chat before drafting. You can course-correct before the emails are written.
- The `buying_motion` frontmatter on `icp.md` controls CTA phrasing. `self-serve` gets low-commitment CTAs; `demo` gets 15-min call CTAs; `both` alternates.
- Re-run after generating ICP v2 (post-50 to 100 sends) for tighter copy. Sequences carry an `icp_version` frontmatter field so you can track which ICP a campaign came from.

---

## email-auditor

### When to use

You have existing cold email copy (drafted by you, a teammate, or another tool) and want it scored before you send. Useful as a final gate after `email-sequence-generator`, or standalone when you're reviewing copy you didn't generate here.

### Inputs

- Required: pasted email copy (subject + body; the skill detects structure).
- Optional but helpful: `icp.md` in the workspace. Enables ICP-aware critique (CTA mismatch with buying motion, value prop mismatch with primary pain, persona drift).

### What you get

A scored audit written to `outreach-workspace/<campaign>/audits/YYYY-MM-DD-<slug>.md`, with frontmatter (`audited_at`, `overall_score`, `icp_aware`, `subject_preview`) and a body containing:

- **Overall score** (X/35, sum of 7 criteria each rated 1 to 5).
- **Per-criterion table**: subject line, opening line, body copy, CTA, formatting and readability, personalization and human touch, spam and deliverability.
- **Strengths**: what to keep.
- **Improvements**: pointed fixes per component.
- **Rewritten draft**: paste-ready copy that addresses the issues.

In chat: the same output, plus a one-line citation if `icp.md` was used.

### Example transcript

```
You: Audit this:

     Subject: Quick question about Acme's ops stack

     Hi {First Name}, hope you're doing well! I noticed Acme is growing fast
     and wanted to reach out. We help companies like yours automate workflows
     across 200+ tools. Open to a 30-min call next week to walk through how
     we've helped 200+ customers save 40% of their ops time?

     Best, Jane

Claude: (reads icp.md, sees buying_motion: self-serve, primary segment is
        "Mid-market B2B SaaS Ops Leads")

## Audit: Quick question about Acme's ops stack

**Overall: 22/35**

| Criterion | Rating | Feedback |
|---|---|---|
| Subject Line | 4/5 | Clear and specific. Could trim "Quick" (filler). |
| Opening Line | 2/5 | "Hope you're doing well" is generic. Open with the trigger. |
| Body Copy | 3/5 | Value prop fits, but "200+ tools" is vague vs. ICP's pain. |
| Call-to-Action | 2/5 | 30-min call mismatches your ICP's self-serve buying motion. |
| Formatting & Readability | 3/5 | Single block; break into 2-3 short paragraphs. |
| Personalization & Human Touch | 4/5 | "Acme is growing fast" is good; could cite a specific signal. |
| Spam & Deliverability | 4/5 | Clean. No spam triggers, no excessive caps. |

### Strengths
- Subject is short and specific to the recipient's company.
- No spam triggers or pushy language.

### Improvements
- **Opener:** Replace "hope you're doing well" with the trigger you noticed.
- **CTA:** Switch to a low-commitment ask (link to a 90-second demo video).
- **Body:** Replace "200+ tools" with the specific tools Acme likely uses.

### Rewritten draft
Subject: Acme's ops stack
Body:
Hi {First Name},

Saw Acme's hiring 3 RevOps roles this quarter - usually a sign the current
stack is breaking under volume.

We help mid-market SaaS ops teams cut handoff time across HubSpot, Snowflake,
and Linear by 40%. Bevy hit that in 6 weeks.

Worth a 90-second demo video? {spin}I'll send the link.|Reply "yes" and I'll
share it.{endspin}

Best,
Jane

Critique informed by icp.md segment "Mid-market B2B SaaS Ops Leads".
```

### Tips

- If you paste copy without a `Subject:` prefix, the skill asks for the subject once before scoring.
- Each audit lives in its own dated file, so history accumulates. Re-auditing the same email after edits creates a new file rather than overwriting.
- Without `icp.md`, the audit still runs but `icp_aware: false` is set in frontmatter. Scores are based on general cold email best practices only.
