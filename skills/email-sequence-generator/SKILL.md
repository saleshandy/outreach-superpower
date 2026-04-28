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

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-saleshandy-superpowers`.

All file reads/writes happen in `outreach-workspace/<campaign>/`.

## Process

### Step 1 - Read context

Read these files in order, citing each as you go:

- `outreach-workspace/<campaign>/icp.md` (REQUIRED). Extract from frontmatter: `version`, `primary_segment`, `buying_motion`. From body: list of segments (3+), Angle Kit (5 angles).
- `outreach-workspace/<campaign>/company.md` (helpful). Extract: industry, value prop, services, differentiators, proof points (with metrics).
- `outreach-workspace/<campaign>/productized-offer.md` (helpful, if present). Extract: offer name, outcome one-liner, scope.

If `icp.md` is missing -> announce handoff: *"No `icp.md` in workspace. I need an ICP first. Run `icp-builder` to define your target segments, then return here. I'll wait."* Stop.

This is an explicit handoff, not a direct invoke. Claude handles the routing via the `using-saleshandy-superpowers` router skill: it detects the announcement, runs icp-builder, then resumes email-sequence-generator once `icp.md` is written.

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

If `productized-offer.md` exists, default to goal 2 (lead-gen) and skip this question unless user overrides.

### Step 3 - Ask: target segment

List segments from `icp.md` body by name. User picks one (or types custom). Default to `primary_segment` from frontmatter if user just hits enter.

### Step 4 - Ask: sequence length

> *Sequence length?*
> 1. 3 emails (short cold outreach)
> 2. 4 emails (standard)
> 3. 5 emails (extended nurture)
> 4. 6-7 emails (comprehensive follow-up)

### Step 5 - Generate sequence

**Email design rules (strict):**
- 150-250 words per email
- `{First Name}`, `{Company}`, `{Industry}`, `{Role}` merge tags (Saleshandy syntax)
- At least one `{spin}A|B|C{endspin}` block per email - preferred: subject + opener + CTA each get a spin variant
- Industry-specific terminology pulled from company.md
- One CTA per email
- Mobile-friendly formatting (short paragraphs, white space)

**Sequence structure:**
- **Email 1:** Value prop + problem identification
- **Middle emails:** Social proof, educational content, objection handling, "why now"
- **Final email:** Strong CTA with urgency

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

**Buying-motion adjustment:** if `icp.md` frontmatter shows `buying_motion: self-serve`, prefer low-commitment CTAs ("Try our free tool"). If `demo`, prefer "Open to a 15-min call?" If `both`, alternate across the sequence.

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
