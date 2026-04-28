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

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

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

**Failure handling per page:**
- Treat as failure for that page: HTTP 4xx/5xx, fetch returns no extractable content, fetch times out (>20s), or response is an interstitial (Cloudflare challenge, bot block detected via missing expected content patterns).
- Skip the failing page and continue with remaining pages.
- If the **homepage** specifically fails, abort the fetch loop and request paste of homepage content (homepage is required).
- If 2+ secondary pages fail, proceed with what you have. Note in `company.md` body which pages were unavailable. Only request paste for missing high-signal pages (e.g., customers/case-studies, if no proof points were extracted from other pages).
- Set `source: mixed` in `company.md` frontmatter when both fetched and pasted content contributed.

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

Also accept:
- `delete #<n>` or `remove #<n>` → replace value with `[REMOVED]`
- `add to #<n>: <text>` → append rather than replace
- Comma- or newline-separated multi-edits in one reply: `#1: foo, #4: bar`
- For rearrangement (`swap #2 with #3`), DON'T guess. Ask: *"Want to restate the card with the new order?"*

Loop until the user types `yes`, `all good`, or `proceed with assumptions`.

### Step 6 - Write workspace files

Write all three files to `outreach-workspace/<campaign>/`.

**Existing-files handling:**
- `company.md` and `strategy.md`: if they already exist, ask once: *"Existing strategy files found. (a) overwrite, (b) version (move existing to `.v1.md`), or (c) merge edits into existing?"* Default to (b) on no answer.
- `icp.md`: NEVER overwrite if frontmatter shows `version: v2` OR `source: icp-builder-interview` (those are tightened ICPs from icp-builder; preserve them). If only the lite `source: strategy-architect-lite` exists, version it (move to `icp.lite.v1.md`) and write fresh lite.

**Confidence flag rules** (set in `company.md` frontmatter):
- `high`: 5+ pages fetched cleanly, all 10 confirmation card items extracted without `[ASSUMPTION]` labels.
- `medium`: 3-4 pages fetched, OR 1-3 of the 10 items are `[ASSUMPTION]` labeled.
- `low`: homepage-only fetch, OR 4+ items are `[ASSUMPTION]` labeled, OR all content was pasted by user.

`icp-builder` reads this to decide whether to skip pre-filled answers (`high`/`medium` → confirm, `low` → re-ask from scratch).

**`company.md`:**

```yaml
---
source: website-scrape | manual-paste | mixed
url: <url>
extracted_at: YYYY-MM-DD
confidence: high | medium | low
---
# <Company Name>
<!-- Values may be tagged [ASSUMPTION] when inferred. -->
- Industry: ...
- Size band: ...
- Value prop: ...
- Services / products:
  - <name>: <one-line description>
  - <name>: <one-line description>
  (one bullet per service; values may be tagged [ASSUMPTION] when inferred)
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
