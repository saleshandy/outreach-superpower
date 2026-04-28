---
name: service-productizer
description: Use when user wants to turn a service into a packaged offer for cold outreach. Triggers on "productize my service," "package my offering," "I run an agency," "turn my service into a product," or "build an offer from my service."
---

# Service Productizer

Turn a raw service description into a productized offer with name, outcome, scope, timeline, pricing guidance, proof, and an offer-specific ICP. Optionally auto-chains to `email-sequence-generator` to draft the cold sequence.

## Inputs

- **Required:** `company.md` (auto-chains to `strategy-architect` if missing).
- **Optional:** existing `productized-offer.md` (will be replaced or versioned), existing `icp.md` (offer-specific ICP appends as a new segment card).

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-saleshandy-superpowers`.

All file reads/writes happen in `outreach-workspace/<campaign>/`.

## Process

### Step 1 - Read company context

Read `outreach-workspace/<campaign>/company.md`. Extract the service list from the `Services / products` section (each sub-bullet is one service, in the form `<name>: <one-line description>`).

**Parsing rule:** split each sub-bullet on the FIRST colon. Everything before is the service name; everything after is the description. If no colon, treat the whole line as the name with empty description. Examples that work:
- `SEO: link-building campaigns (DR 50+ targets)` -> name: `SEO`, description: `link-building campaigns (DR 50+ targets)`
- `Podcast production` -> name: `Podcast production`, description: empty

If `company.md` is missing -> announce: *"No `company.md` found in workspace. I need a website extracted first. Run `strategy-architect` with the user's website URL (or paste their homepage), then return here once `company.md` exists. I'll wait."*

This is an explicit handoff, not a direct invoke. Claude handles the routing via the `using-saleshandy-superpowers` router skill: it detects the announcement, runs strategy-architect, then resumes service-productizer once `company.md` is written.

If `company.md` exists but the `Services / products` section is empty or absent, ask the user to paste their service list (one per line) before continuing.

### Step 2 - List services + ask which to pitch first

Output:

> *From your website, here are the services I extracted. Which do you want to productize first?*
>
> 1. <Service 1>
> 2. <Service 2>
> 3. <Service 3>
>
> Or type a custom service name with a 1-line description.

Wait for the user's pick before moving to Step 3.

### Step 3 - Productize using this structure

For the chosen service, generate the following six items. No fluff, no broad claims. Numbers beat adjectives.

**A) Offer Name** - generate 3 specific, marketable candidates. Examples (template only - generate names that fit the user's industry):
- *"30-Day Pipeline Jumpstart"* (SaaS / B2B services)
- *"Inbox-Ready LinkedIn SDR"* (sales)
- *"Quarterly Compliance Review"* (legal / finance / regulated industries)
- *"AI Hiring Sprint"* (HR tech)

Ask user to pick or refine.

**B) Outcome-Based One-Liner** - *"We help [ICP] get [specific result] in [timeframe] without [objection]."* Generate this from the service description and any customer outcomes already in `company.md`.

**C) Scope (What's Included)** - bulleted deliverables with clear boundaries:
- Specific quantities (e.g., "30 qualified leads," not "lots of leads")
- Specific channels / artifacts (e.g., "2 campaign variants on LinkedIn + email")
- Specific reporting / iteration cadence (e.g., "weekly reporting + A/B optimization")

**D) Timeline & Format** - specify duration + delivery method (e.g., *"4-week sprint, async via Slack + Notion"*).

**E) Pricing Guidance** (optional) - tier or range. Frame value, not cost.

**F) Proof Point** - ask the user for a case study, testimonial, or client win for this specific service. If none, mark `[NEEDED]` in the file and proceed without blocking.

**Custom-service handling.** For custom services not in `company.md`, ask 4 follow-up questions before generating A-F:
- *"What measurable outcome does this service deliver to clients?"*
- *"What's the typical engagement length and format?"*
- *"Who's the ideal client (size / industry / role)?"*
- *"How does this differ from what clients could do in-house or with a competitor?"*

Use those answers as inputs to A-F.

### Step 4 - Define ICP for *this offer*

Lighter-weight than full `icp-builder`. Ask 5 questions. One per message is fine but bundling is acceptable here since the offer-specific ICP is meant to be quick:

> *For this offer specifically:*
> 1. What company size + type fits best?
> 2. Industry vertical(s)?
> 3. Target persona / job title?
> 4. What pain or trigger event makes them buy now?
> 5. Who's a bad fit? (1-2 disqualifiers - who shouldn't buy this)

If `icp.md` exists with `source: icp-builder-interview`, pre-fill suggested answers from the existing ICP's primary segment and let the user confirm or override.

### Step 5 - Write `productized-offer.md`

Write to `outreach-workspace/<campaign>/productized-offer.md`.

Frontmatter:

```yaml
---
service: <selected service>
offer_name: "<Name>"
generated_at: YYYY-MM-DD
icp_source: appended-to-icp.md | embedded-only
icp_buying_motion: <copy from icp.md if appended; else from Step 4 answers>
---
```

Note: *"`icp_source: appended-to-icp.md` means the offer's segment was added to icp.md and is the canonical source - read from icp.md frontmatter for `buying_motion`. `embedded-only` means no icp.md existed; offer's ICP is in this file's body."*

Body sections:

- `# <Offer Name>`
- `**Outcome:** <one-liner>`
- `**Scope:**` (bulleted)
- `**Timeline:** ...`
- `**Pricing:** ...`
- `**Proof:** ...` (or `[NEEDED]`)
- `## ICP for this offer`
  - Company size & type: ...
  - Industry: ...
  - Persona: ...
  - Pain / trigger: ...

**Existing-file handling for `productized-offer.md`:**
- If it already exists, ask once: *"Existing `productized-offer.md` found. (a) overwrite, (b) version (move existing to `productized-offer.v1.md`), or (c) keep existing and abort?"* Default to (b) on no answer.

**Append to existing `icp.md` (don't overwrite).** If `icp.md` exists in the workspace:

**Mapping the 4 Step-4 answers to icp-builder's 9-row segment card:**

| icp-builder segment card row | Source |
|---|---|
| Segment name | Generated from offer name + persona (e.g., "Mid-market SaaS VPs - Pipeline Jumpstart") |
| Firmographics | Step 4.1 (company size + type) + Step 4.2 (industry) |
| Technographics | `[NEEDED]` if not in `company.md` |
| Common pain | Step 4.4 (pain or trigger event) |
| Trigger events | Step 4.4 (trigger event component); split if Step 4.4 included both |
| Messaging angle | Generated from offer's outcome one-liner (Step 3B) |
| Objections + counters | `[NEEDED]` (run `icp-builder` to fill) |
| Disqualifiers | Step 4.5 (1-2 disqualifiers) |
| Example titles to target | Step 4.3 (target persona / job title) - single title; expand via icp-builder later |

Append the segment card with `[NEEDED]` markers for fields the lite ICP doesn't cover. Footer the appended section with: *"For full segment depth, run `icp-builder` to fill `[NEEDED]` fields."*

- Append the offer-specific ICP as a NEW segment card at the end of the Segment Cards section (using the same table schema icp-builder uses).
- Do NOT modify or overwrite existing segments.
- Increment the `segments:` count in `icp.md` frontmatter by 1.
- Cite in chat: *"Appended segment '<offer name>' to existing icp.md."*

If `icp.md` does not exist, the offer-specific ICP lives only inside `productized-offer.md`. Suggest running `icp-builder` next if the user wants a full ICP.

### Step 6 - Auto-chain offer

Output:

> *Offer saved to `productized-offer.md`. Want me to draft a 3-email cold sequence for this offer? (y/n)*

If **yes** -> announce: *"Handing off to `email-sequence-generator` with `goal: lead-gen` and segment '<offer name>'. Reading `productized-offer.md` and `icp.md` for context."* The router will load email-sequence-generator on the next turn.

If **no** -> end with: *"Offer ready. Run `email-sequence-generator` whenever you want sequences."*
