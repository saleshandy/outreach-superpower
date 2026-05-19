---
name: lead-finder
description: Use when the user wants to find prospects matching their ICP, build a target list, search companies, search people, source prospects, find leads matching criteria, or get verified emails for an ICP. Triggers on "find prospects matching ICP," "build target list," "search companies," "search people," "find leads matching [criteria]," "source prospects," "get verified emails for [ICP]." Reads `icp.md` filters, formulates a Saleshandy Lead Finder search via `saleshandy:discover-and-enroll`, persists results to `leads.md`.
---

# Lead Finder

Turn an ICP into a verified prospect list, ready to enroll into a sequence. This skill is the workflow layer; the actual search is delegated to `saleshandy:discover-and-enroll`.

## Inputs

- **Required:** `outreach-workspace/<campaign>/icp.md` with a `## Search-ready filters` section (a yaml block produced by `icp-builder`). If that section is missing, see graceful-degradation in Step 1.
- **Optional flags:**
  - `--limit N` - cap the result list (default `50`). Hard ceiling: `500` per run.
  - `--include-phones` - request phone reveal alongside email. Adds ~6x credit cost per prospect.
- **Optional file:** `company.md` for tone/context when summarizing results.

## Outputs

- Writes: `outreach-workspace/<campaign>/leads.md`

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## Hard rules

1. **Delegate the search.** Never call Saleshandy MCP tools (`sage_search`, `enrich_contacts`, `import_prospects_*`) directly. Always go through `saleshandy:discover-and-enroll`.
2. **Confirm credit cost before invoking.** Estimate `limit × 1 credit` for email-only, `limit × 7 credits` if `--include-phones`. Show the math and wait for explicit user confirmation before delegating.
3. **Never overwrite an existing `leads.md` silently.** If one exists, ask: *"Existing `leads.md` found (N rows, dated YYYY-MM-DD). (a) overwrite, (b) append, (c) version (move to `leads.v1.md`)?"* Default to (c) on no answer.

## Process

### Step 1 - Read ICP filters (with graceful degradation)

Read `outreach-workspace/<campaign>/icp.md`. Look for a yaml block under a `## Search-ready filters` heading. Expected shape:

```yaml
firmographics:
  industries: [SaaS, Fintech]
  employee_range: [51, 500]
  hq_countries: [US, UK]
  funding_stages: [Series A, Series B]
technographics:
  uses: [HubSpot]
  not_uses: [Outreach.io]
behavioral_triggers:
  - hired_VP_Sales_last_90_days
intent_signals:
  - downloaded_cold_email_content
```

**If the section is present:** parse it into a structured filter block. Proceed to Step 2.

**If `icp.md` exists but the section is missing** (common until `icp-builder` Batch 3 lands):

1. Read the rest of `icp.md` - Segments table, Targeting Filters table, Disqualifiers.
2. Draft a filter block from what you find. Tag every inferred value with `[ASSUMPTION]`.
3. Show the user the drafted filter block in a code block and ask: *"`icp.md` doesn't have a structured `Search-ready filters` section yet. I inferred this from the rest of the file - confirm or edit before I search?"*
4. Apply edits the user supplies (parsing `field: new value` lines), re-display once, loop until user types `yes` / `proceed`.
5. After confirmation, do NOT write the inferred block back to `icp.md` - `icp-builder` owns that file. Just hold the confirmed filters in memory for Step 2.

**If no `icp.md` exists at all:** stop. Output: *"No `icp.md` found in `outreach-workspace/<campaign>/`. Run `icp-builder` first to define your ICP, then come back here. (Or run `strategy-architect` if you don't even have a company brief yet.)"*

### Step 2 - Formulate the natural-language search query

`saleshandy:discover-and-enroll` accepts a free-text description. Translate the structured filter block into a single, specific sentence. Aim for:

- **One sentence**, comma-separated qualifiers.
- **Concrete titles and seniority** (not "decision-makers").
- **Numeric ranges** (not "mid-market").
- **2-3 strongest signals only** - extra qualifiers narrow the universe more than they help.

Examples:

| ICP filter set | Formulated query |
|---|---|
| SaaS, 51-500 emp, US/UK, VP/Director Engineering | "VPs and Directors of Engineering at US/UK B2B SaaS companies with 51-500 employees" |
| Fintech Series B+, hiring sales reps, VP Sales | "VPs of Sales at US fintech companies, Series B or later, currently hiring sales reps" |
| Agency, 10-50 emp, agency-services, founder/CEO | "Founders and CEOs of 10-50 person marketing or sales agencies in the US" |

Show the formulated query to the user before invoking. They can edit it.

### Step 3 - Confirm scope + credit cost

Output exactly this card:

```
About to invoke saleshandy:discover-and-enroll with:

  Query: <one-sentence description>
  Limit: <N>
  Phone reveal: <yes / no>
  Estimated credit cost: <math> (rough - actual cost depends on reveal success)

Proceed? (yes / edit query / change limit / cancel)
```

Wait for explicit confirmation. If user says edit / change, accept the change and re-show the card. Loop until `yes` or `cancel`.

**Cost math:**
- Email-only: `limit × 1 credit`
- Email + phone: `limit × 7 credits` (the ~7 figure comes from the `saleshandy:enrich-leads` skill description)
- Add a 10% safety margin if the filter set is unusually narrow (more reveals may fail and be retried).

### Step 4 - Invoke `saleshandy:discover-and-enroll`

Use the Skill tool to invoke `saleshandy:discover-and-enroll` with the confirmed natural-language description and limit. Do NOT call any `mcp__*` tool yourself.

The delegated skill handles: search, credit-cost gating with the user (it has its own confirmation flow), enrichment, and optionally enrollment into a sequence step. **Do not pass enrollment args from `lead-finder`** - `lead-finder` builds the list. Enrollment is a downstream concern handled either inside `saleshandy:discover-and-enroll` (if the user explicitly asks during its flow) or by a future sequence skill.

Capture from its result:
- Total companies matched
- Total people revealed (with email)
- Total people revealed (with phone, if `--include-phones`)
- Total credits actually used
- The list of revealed prospects with company, name, title, email, [phone]

If `saleshandy:discover-and-enroll` returns 0 results, skip to Edge cases below. If it returns more than the requested limit (rare), trim before persisting.

### Step 5 - Persist results to `leads.md`

Write `outreach-workspace/<campaign>/leads.md` using the file format below. Apply the existing-file handling rule from "Hard rules" above.

### Step 6 - Suggest next skill

Output one line:

> *Workspace written: `leads.md` (N prospects across M companies, X credits used). Recommended next: run `prospect-enrichment` to add AI personalization columns (icebreakers, role-based hooks) before enrolling.*

## `leads.md` format

```yaml
---
generated_at: YYYY-MM-DD
campaign: <campaign>
source: saleshandy-lead-finder
query: "<the natural-language query used>"
limit: <N>
include_phones: true | false
total_companies: <int>
total_prospects: <int>
credits_used: <int>
---
```

```markdown
# Leads - <campaign>

## Search query

> "<natural-language query>"

## Filters used

<paste the confirmed filter yaml block from Step 1 inline here - for traceability>

## Companies (top <M>)

| # | Company | Domain | Industry | Size | Country | Signals |
|---|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | ... | ... |
...

## People (N total)

| # | Name | Title | Company | Email | Phone | LinkedIn |
|---|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | ... | ... |
...

## Summary

- Companies matched: <int>
- People revealed: <int> (email) / <int> (phone)
- Credits used: <int>
- Credits remaining: <int if known, else "see Saleshandy">

## Recommended next step

Run `prospect-enrichment` to add AI personalization columns before enrolling these prospects into a sequence.
```

If the people list exceeds 50 rows, show the top 50 in the table and note: *"+ N more - see Saleshandy CRM for the full list."* The whole point of `leads.md` is a workspace summary, not a CSV dump.

## Edge cases

- **`icp.md` missing:** route user to `icp-builder` (per Step 1).
- **No `Search-ready filters` section in `icp.md`:** infer, confirm with user, hold in memory only (per Step 1).
- **Search too broad (>20k companies estimated):** before invoking, warn: *"Your filters match a very large universe. The search will only reveal the top N, but you'll get more value from tightening the filter set. Tighten now, or proceed with current filters?"*
- **Search returns 0 results:** suggest filter relaxation. Identify the most restrictive filter (usually narrowest geography or smallest size band) and propose widening it: *"Try widening `hq_countries` from `[US]` to `[US, UK, CA]`? Or relaxing `employee_range` from `[200, 300]` to `[100, 500]`?"* Do NOT auto-retry without confirmation.
- **User cancels at Step 3:** exit cleanly. Do not write `leads.md`.
- **`saleshandy:discover-and-enroll` errors mid-run:** report the error verbatim, surface any partial results it returned, ask user if they want to retry with adjusted filters.
- **Existing `leads.md` with `generated_at` < 24h ago:** offer (a) overwrite, (b) append, (c) version (default). Default action on no answer is (c).
- **`--limit` exceeds 500:** refuse and explain: *"Limits over 500 per run risk wasted credits on lower-fit prospects. Run two passes with tighter filters instead, or confirm explicitly with `--limit 500 --i-know-what-im-doing`."*
- **`--include-phones` but no `call` step in the sequence plan:** warn once: *"Phone reveal costs ~7x more per prospect. You're not using phones in your sequence yet - reveal email-only first?"* Proceed if user confirms.
