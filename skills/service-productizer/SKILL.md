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

If `company.md` is missing, announce the auto-chain explicitly:

> *No `company.md` found. Running `strategy-architect` first to extract services from your website, then resuming.*

Invoke `strategy-architect` with whatever context the user already provided (URL, pasted content, vertical hint). After it completes and writes `company.md`, return here and continue at Step 2.

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

**A) Offer Name** - generate 3 specific, marketable candidates (e.g., *"30-Day Pipeline Jumpstart"*, *"Inbox-Ready LinkedIn SDR"*, *"AI Hiring Sprint"*). Ask the user to pick one or refine.

**B) Outcome-Based One-Liner** - *"We help [ICP] get [specific result] in [timeframe] without [objection]."* Generate this from the service description and any customer outcomes already in `company.md`.

**C) Scope (What's Included)** - bulleted deliverables with clear boundaries:
- Specific quantities (e.g., "30 qualified leads," not "lots of leads")
- Specific channels / artifacts (e.g., "2 campaign variants on LinkedIn + email")
- Specific reporting / iteration cadence (e.g., "weekly reporting + A/B optimization")

**D) Timeline & Format** - specify duration + delivery method (e.g., *"4-week sprint, async via Slack + Notion"*).

**E) Pricing Guidance** (optional) - tier or range. Frame value, not cost.

**F) Proof Point** - ask the user for a case study, testimonial, or client win for this specific service. If none, mark `[NEEDED]` in the file and proceed without blocking.

**Custom-service handling.** If the user picked a service that wasn't in the extracted `company.md` list, ask 3 follow-up questions before generating A-F:

> 1. What measurable outcome does this service deliver to clients?
> 2. What's the typical engagement length and format?
> 3. Who's the ideal client (size / industry / role)?

Use those answers as inputs to A-F.

### Step 4 - Define ICP for *this offer*

Lighter-weight than full `icp-builder`. Ask 4 questions. One per message is fine but bundling is acceptable here since the offer-specific ICP is meant to be quick:

> *For this offer specifically:*
> 1. What company size + type fits best?
> 2. Industry vertical(s)?
> 3. Target persona / job title?
> 4. What pain or trigger event makes them buy now?

If `icp.md` exists with `source: icp-builder-interview`, pre-fill suggested answers from the existing ICP's primary segment and let the user confirm or override.

### Step 5 - Write `productized-offer.md`

Write to `outreach-workspace/<campaign>/productized-offer.md`.

Frontmatter:

```yaml
---
service: <selected service>
offer_name: "<Name>"
generated_at: YYYY-MM-DD
icp_buying_motion: self-serve | demo | both
---
```

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
- Append the offer-specific ICP as a NEW segment card at the end of the Segment Cards section (using the same table schema icp-builder uses).
- Do NOT modify or overwrite existing segments.
- Increment the `segments:` count in `icp.md` frontmatter by 1.
- Cite in chat: *"Appended segment '<offer name>' to existing icp.md."*

If `icp.md` does not exist, the offer-specific ICP lives only inside `productized-offer.md`. Suggest running `icp-builder` next if the user wants a full ICP.

### Step 6 - Auto-chain offer

Output:

> *Offer saved to `productized-offer.md`. Want me to draft a 3-email cold sequence for this offer? (y/n)*

If the user replies **yes** (or any clear affirmative), invoke `email-sequence-generator` with `goal: lead-gen` and the segment derived from this offer's ICP. Pass the offer name and outcome one-liner so the sequence's hook reflects the productized offer rather than the raw service.

If the user replies **no** (or any clear decline), respect it. Do NOT invoke `email-sequence-generator`. End with:

> *Offer ready. Run `email-sequence-generator` whenever you want sequences.*
