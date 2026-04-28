---
name: strategy-architect
description: Use when user wants an outreach strategy from their website, says "build a cold outreach plan from my site," "audit my site for outreach," or starts a fresh outreach project with just a URL. Extracts ICP/personas/proof/differentiators and produces a 10-item confirmation card.
---

# Strategy Architect

Pull all needed inputs from the user's website, confirm them briefly, then produce a decision-ready outreach plan + lightweight ICP.

## Inputs

- **Required:** website URL (or pasted content if WebFetch fails)
- **Optional:** `--campaign <name>` to scope output folder

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-saleshandy-superpowers`.

## Process

### Step 1 - Ask for URL

Output one line:

> *Share your website URL and I'll extract the rest. (Or paste your homepage if you'd rather.)*

If the user already supplied a URL in their initial message, skip the prompt and proceed to Step 2.

### Step 2 - Fetch pages

Use WebFetch on (in priority order, stop at first failure or 6 pages):

1. Homepage (`/`)
2. `/pricing`
3. `/customers` or `/case-studies`
4. `/about` or `/about-us`
5. `/security` or `/compliance`
6. `/blog` (recent post titles only - for trend signals)

If WebFetch is unavailable or repeatedly fails: ask the user to paste home + customers/case-studies page text. Note `source: manual-paste` (or `mixed` if some pages fetched and others pasted) in `company.md` frontmatter.

### Step 3 - Extract these 10 items

1. **Value prop** - one-line synthesis from headline + subhead
2. **ICP segments** - 3-4 industries / size bands / geos
3. **Primary personas** - buyer titles, end users
4. **Regions/language**
5. **Proof to cite** - case study metrics, logos, testimonials, certifications
6. **Key differentiators** - unique mechanisms, integrations, SLAs, security
7. **Likely objections** - implied from FAQ / comparison pages
8. **Entry offer/CTA** - demo, pilot, free trial, ROI review
9. **Constraints** - GDPR, regulated verticals, no-phone, etc.
10. **Success metric** - meetings, SQLs, pipeline (default: ask)

Label any guessed item with `[ASSUMPTION]`. Be brief, direct, specific. Numbers beat adjectives.

### Step 4 - Show confirmation card

```
Please confirm or edit (reply with item number + change, or "all good"):

1. Value prop: <synthesis>
2. ICP segments: <3-4 segments>
3. Primary personas: <titles>
4. Regions/language: <regions>
5. Proof to cite: <metrics/logos>
6. Key differentiators: <bullets>
7. Likely objections: <bullets>
8. Entry offer/CTA: <demo/pilot/etc.>
9. Constraints: <GDPR/no phone/etc.>
10. Success metric: <meetings/SQLs/pipeline>

Proceed with these? (yes / proceed with assumptions / make edits #1, #4...)
```

### Step 5 - Apply edits if requested, re-show card

Parse edits in the form `#<n>: <new value>` (also accept `<n>:`, `item <n>`, `change <n> to`). Apply each edit to the corresponding item, then re-display the full updated card and ask again: *"Proceed with these? (yes / proceed with assumptions / make edits #1, #4...)"*

Loop until the user types `yes`, `all good`, or `proceed with assumptions`.

### Step 6 - Write workspace files

Write all three files to `outreach-workspace/<campaign>/`. If files already exist, ask before overwriting.

**`company.md`:**

```yaml
---
source: website-scrape | manual-paste | mixed
url: <url>
extracted_at: YYYY-MM-DD
confidence: high | medium | low
---
# <Company Name>
- Industry: ...
- Size band: ...
- Value prop: ...
- Services / products: ...
- Differentiators: ...
- Proof points: ...
- Constraints: ...
```

**`strategy.md`:**

```yaml
---
goal: book-demo | lead-gen | partnership | recruitment | promotion | pr | nurture
channels: [email]
success_metric: meetings | SQLs | pipeline
---
# Strategy: <goal>
- Why-now triggers: ...
- Entry offer / CTA: ...
- Likely objections + counters: ...
- Sequence design recommendation: ...
```

**`icp.md` (lite - full version comes from `icp-builder`):**

```yaml
---
version: v1
source: strategy-architect-lite
needs_tightening: true
segments: 3
---
# ICP Summary
| Field | Value |
|---|---|
| Who it's for | ... |
| Primary pain | ... |
| Primary outcome | ... |
| Why now | ... |
| Buying roles | ... |
| Proof points | ... |

# Segments (3)
[Generated from extracted ICP segments]

# Recommended next step
Run `icp-builder` to tighten this ICP via the 14-step interview.
```

### Step 7 - Confirm + suggest next skill

Output: *"Workspace written: `company.md`, `strategy.md`, `icp.md` (lite). Recommended next: run `icp-builder` to tighten the ICP, or `email-sequence-generator` to draft a sequence."*
