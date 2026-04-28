---
name: email-sequence-generator
description: Use when user wants a cold email sequence with merge tags and spintext for Saleshandy. Triggers on "write cold email sequence," "draft a cadence," "generate emails for my campaign," "X-email sequence," "follow-up sequence," or "outreach emails."
---

# Email Sequence Generator

Generate a 3-7 email cold sequence with merge tags and spintext, formatted for paste-into-Saleshandy.

## Inputs

- **Required:** `icp.md` (auto-chained via router if missing).
- **Helpful:** `company.md` for industry voice, proof points, differentiators.
- **Helpful:** `productized-offer.md` if pitching a productized service.

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

All file reads/writes happen in `outreach-workspace/<campaign>/`.

## Process

### Step 1 - Read context

Read these files in order, citing each as you go:

- `outreach-workspace/<campaign>/icp.md` (REQUIRED). Extract:
  - From frontmatter: `version`, `primary_segment`, `buying_motion`, `segments` (count).
  - From body: segment names by reading `# Segment: <name>` H1 headings (or `## Segment: <name>` H2 - accept either) under the Segment Cards section. The Angle Kit (5 angles) lives under a `# Cold Email Angle Kit` heading.
  - **Parsing rule:** if multiple segment heading levels exist, prefer H2 first, then H1.
- `outreach-workspace/<campaign>/company.md` (helpful). Extract: industry, value prop, services, differentiators, proof points (with metrics).
- `outreach-workspace/<campaign>/productized-offer.md` (helpful, if present). Extract: offer name, outcome one-liner, scope.

**Angle Kit fallback:** if `icp.md` exists but no Angle Kit section is found (e.g., user wrote icp.md by hand without running icp-builder), generate 5 angles inline before drafting emails:

1. Pick the goal from Step 2.
2. Pick the segment from Step 3.
3. Use `company.md` differentiators + proof points + the segment's pain.
4. Generate 5 angles in the format `Angle name | Subjects (3) | Openers (2) | CTAs (2)`. Display them in chat before generating sequence - gives user a chance to course-correct.

Note in `sequence.md` body: *"Angles generated inline (no Angle Kit found in icp.md). Run `icp-builder` to write a persistent Angle Kit."*

If `icp.md` is missing -> announce handoff: *"No `icp.md` in workspace. I need an ICP first. Run `icp-builder` to define your target segments, then return here. I'll wait."* Stop.

This is an explicit handoff, not a direct invoke. Claude handles the routing via the `using-outreach-superpower` router skill: it detects the announcement, runs icp-builder, then resumes email-sequence-generator once `icp.md` is written.

### Step 2 - Ask: goal

> *What's the campaign goal?*
>
> 1. Book a Demo / Sales Call (auto: product hooks, calendar)
> 2. Lead Generation (auto: pain hooks, ROI angles)
> 3. Backlink Outreach (auto: value exchange, content showcase)
> 4. Talent Recruitment (auto: culture, growth)
> 5. Promotions / Discounts (auto: urgency, exclusivity)
> 6. PR Outreach (auto: newsworthy, expert positioning)
> 7. Partnership Development (auto: mutual benefit)
> 8. Nurture / Engage Existing Prospects (auto: value-add, progressive)
>
> Reply 1-8 or describe a custom goal.

If `productized-offer.md` exists, ask:

> *Pitching `<offer_name>` as lead-gen (the typical goal for productized offers). Proceed (Enter), or pick a different goal (1-8)?*

### Step 3 - Ask: target segment

List segments from `icp.md` body, marking the primary clearly:

> *Pick a segment:*
> 1. <Segment 1> [primary]
> 2. <Segment 2>
> 3. <Segment 3>
> 4. <Productized offer's segment from productized-offer.md, if present>
>
> Reply with the number, or hit Enter to use the primary segment.

**If `productized-offer.md` exists and its `icp_source: appended-to-icp.md`:** include its target segment in the picker (numbered after icp.md's segments).

**Default behavior:** if user replies with empty input, use `primary_segment` from icp.md frontmatter.

### Step 4 - Ask: sequence length

> *Sequence length?*
> 1. 3 emails (short cold outreach)
> 2. 4 emails (standard)
> 3. 5 emails (extended nurture)
> 4. 6-7 emails (comprehensive follow-up)

### Step 5 - Generate sequence

**Email design rules (strict):**

**Word count by position:**
- **Email 1 (cold open):** 150-220 words (full value prop + problem framing)
- **Email 2-3 (middle):** 120-180 words (proof, objection handling, why now)
- **Email 4+ (bumps / final):** 50-100 words (short, punchy CTA)

Last email in any sequence = "bump" length (50-100 words) regardless of total length, unless explicitly the only email after Email 1.

- `{First Name}`, `{Company}`, `{Industry}`, `{Role}` merge tags (Saleshandy syntax)
- Industry-specific terminology pulled from company.md
- One CTA per email
- Mobile-friendly formatting (short paragraphs, white space)

**Spintext rules:**
- **Minimum:** 1 `{spin}A|B{endspin}` block per email (anywhere - subject, opener, body, or CTA).
- **Recommended:** spintext on subject + opener + CTA (3 blocks per email). Maximizes A/B coverage in Saleshandy.
- **Format:** `{spin}variant A|variant B{endspin}` - Saleshandy picks one randomly per send.

**Sequence structure** (Problem-Solution-Proof-CTA framework, applied progressively across the sequence):
- **Email 1:** Problem identification + Solution preview (value prop)
- **Middle emails (2-3):** Proof (metrics, social proof) + objection handling + "why now"
- **Final email:** Strong CTA with urgency. Bump-style if length >= 4.

**Per-goal hooks** - use these as the opening angle:

| Goal | Email 1 hook |
|---|---|
| book-demo | Product value + calendar nudge |
| lead-gen | Pain point + ROI |
| backlinks | Value exchange + content reference |
| recruitment | Culture + growth opportunity |
| promotions | Urgency + exclusive offer |
| pr | Newsworthy angle + expert positioning |
| partnership | Mutual benefit + roadmap |
| nurture | Value-add content |

**Buying-motion adjustment** (uses icp.md frontmatter `buying_motion`):

| buying_motion | CTA pattern |
|---|---|
| `self-serve` | All emails: low-commitment CTAs (e.g., "Try our free tool," "Check the demo video") |
| `demo` | All emails: 15-min call CTAs (e.g., "Open to a 15-min call?") |
| `both` | Email 1, 3, 5: self-serve CTAs. Email 2, 4, 6, 7: demo CTAs. Final email always demo regardless of position. |

**Productized-offer infusion:** if `productized-offer.md` is present, use the offer name in subject lines, reference the scope + outcome in body 1-2 emails, and default the final-email CTA to the offer's entry CTA.

### Step 6 - Output

For each email N of the sequence:

```markdown
## Email N
**Subject:** {spin}<A>|<B>{endspin}

**Body:**
Hi {First Name},

{spin}<opening A>|<opening B>{endspin}

<value prop pulled from company.md>

<proof point with metric>

{spin}<CTA A>|<CTA B>{endspin}

Best,
[Your Name]
[Your Company]
```

### Step 7 - Write `sequence.md`

Write `outreach-workspace/<campaign>/sequence.md`.

Frontmatter:

```yaml
---
goal: book-demo | lead-gen | backlinks | recruitment | promotions | pr | partnership | nurture
icp_segment: <segment name from icp.md>
email_count: 4
generated_at: YYYY-MM-DD
icp_version: v1 | v2
offer_name: "<from productized-offer.md if present>"
---
```

Body:
- Title: `# <Goal> sequence for <segment>`
- Email 1...N as shown in Step 6
- Footer: `**Saleshandy import:** Head to https://my.saleshandy.com/sequence and paste each email.`

If `icp.md` is `version: v1`, append a second footer: *"Generated against ICP v1. Re-run after 50-100 sends with v2 for tighter results."*

### Step 8 - Cite

End chat with: *"Sequence written to `sequence.md`. Pulled segment '<X>' from `icp.md`, proof points from `company.md`<, offer details from `productized-offer.md` if applicable>."*
