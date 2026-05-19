---
name: prospect-enrichment
description: Use when the user wants to enrich prospects, add an AI column for personalization, personalize at scale, build personalization columns, auto-personalize, or batch-enrich leads. Triggers on "enrich prospects," "add AI column," "personalize at scale," "AI column," "enrich my leads," "build personalization columns," "auto-personalize." Reads `leads.md` and `icp.md`, designs one or more enrichment columns, estimates credit cost, persists a plan to `enrichment-plan.md`. Optionally executes via Saleshandy MCP behind explicit per-batch confirmation; never overwrites a non-empty field without per-field confirmation.
---

# Prospect Enrichment

Design AI columns that personalize cold outreach at scale, persist the plan, and only execute behind confirmation gates.

## Inputs

- **Required:** `outreach-workspace/<campaign>/leads.md` (to know prospect count and available fields).
- **Required:** `outreach-workspace/<campaign>/icp.md` (for angle / pain-point context).
- **Optional:** `outreach-workspace/<campaign>/company.md` (for sender-side tone and positioning).

## Outputs

- Writes: `outreach-workspace/<campaign>/enrichment-plan.md`

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## Hard rules

1. **Never overwrite a non-empty prospect field without explicit per-field user confirmation.** This applies to both plan-only and execute-via-MCP modes. If a column the user wants to write to already has data in any rows, surface the row count and require a typed `yes, overwrite N rows` before proceeding. No exceptions.
2. **Default delivery mode is plan-only.** Direct MCP execution requires the user to explicitly choose "execute" at Step 5. Plan-only mode never spends credits.
3. **Show cost math before every execution.** `prospects_in_scope × credits_per_row = total_credits`. Refuse to call `enrich_companies` / `enrich_contacts` without an explicit `yes, proceed` from the user after that math is on screen.
4. **Always run a test on 5-10 prospects before scaling.** Even in execute mode. A failed prompt at 10 rows costs ~150 credits; a failed prompt at 500 rows costs ~7,500.

## Process

### Step 1 - Read inputs and surface scope

Read `leads.md`, `icp.md`, and `company.md` (if present). Extract:

- **Prospect count** from `leads.md` frontmatter (`total_prospects`).
- **Available source fields** by inspecting the People table headers in `leads.md` (e.g., First Name, Title, Company, Domain, City, LinkedIn).
- **Top angles** from `icp.md` Angle Kit (if present) for personalization theming.
- **Sender voice** from `company.md` (if present) - affects tone of generated lines.

Output a one-line scope summary:

> *Found N prospects across M companies in `leads.md`. Available fields: <comma list>. ICP segment: <primary segment>. Ready to design enrichment columns.*

### Step 2 - Identify the personalization goal

Ask the user one question with numbered options:

```
What kind of personalization do you want?

  1. Icebreaker / opener (one custom line per prospect, role-based)
  2. Pain-point summary (custom hook based on their company stage)
  3. Tech-stack callout (reference what they use today)
  4. News-based hook (recent funding / hiring / product launch)
  5. Custom AI column (you'll describe the prompt)

Reply with a number, or describe what you want in your own words.
```

If the user picks multiple (e.g., "1 and 3"), proceed to design two columns in parallel.

### Step 3 - Design each column

For every column the user wants, draft:

| Field | Example |
|---|---|
| Column name | `Career Icebreaker (AI)` |
| Source fields needed | `First Name`, `Title`, `Company` |
| Prompt template | "Write one sentence acknowledging {{First Name}}'s role as {{Title}} at {{Company}}, referencing the typical scaling challenge at that stage. Single sentence. No emoji. No exclamation mark." |
| Output format | `Single Sentence` |
| Model | `Gemini 2.5 Pro` (recommended default) |
| Credit cost per row | `~15` (varies by model - see cost table below) |
| Rows that will skip | Estimate count of prospects missing any required field |

Show all proposed columns in one summary table. Ask: *"Confirm these columns, or edit any (`#1 prompt: <new text>`, `#2 source fields: <list>`, `remove #N`, etc.)?"* Loop until user types `yes` / `looks good`.

### Step 4 - Estimate credit cost (cost table)

Per-row credit costs by model and template family. Source: `Imports-custom-gpts/New skills/PB-07-AI-ENRICHMENT.md`.

| Template family / model | Cost per row | Use for |
|---|---|---|
| Gemini 2.0 Flash | ~0.64 | High-volume simple lines (basic icebreakers) |
| LinkedIn Connect Msg | ~0.64 | LinkedIn touchpoints |
| Ideal Customer Finder | ~0.64 | ICP from company domain |
| Perplexity Sonar Pro | ~8.24 | Deep web research on specific people |
| Gemini 2.5 Pro (default) | ~14.30 | Balanced quality / cost personalization |
| Email Subject / Content Generator | ~15 | Subject lines, email bodies |
| Career / Role / Industry Icebreakers | ~15 | Role-based opener lines |
| GPT 4.1 | ~32 | Reliable versatile research |
| Company Research (Competitors / Offerings / Founders / B2B-B2C) | ~81 | Deep company research |
| `enrich_companies` MCP | ~0.1 / company | Company-level firmographic fill |
| `enrich_contacts` MCP - email reveal | ~1 / contact | Email reveal only |
| `enrich_contacts` MCP - email + phone | ~7 / contact | Email + phone reveal |

The `enrich_companies` / `enrich_contacts` figures are from the `saleshandy:enrich-leads` skill description. AI-column costs are from PB-07-AI-ENRICHMENT.

Output the per-column estimate AND the run-total:

```
Cost estimate:

  Column 1 - Career Icebreaker (AI) - Gemini 2.5 Pro - ~15 / row
    N rows × 15 = ~N×15 credits
  Column 2 - Tech-Stack Callout (AI) - Perplexity Sonar Pro - ~8.24 / row
    N rows × 8.24 = ~N×8.24 credits

  Total estimated: ~<sum> credits
  Test run (5 rows per column): ~<sum_for_5> credits

Your balance: <known if user told you; else "check Saleshandy header">.
```

### Step 5 - Choose delivery mode

Ask the user one question:

```
Two ways to ship this:

  (a) Plan-only - I write enrichment-plan.md with paste-ready column
      names + prompts. You configure them in Saleshandy UI (Enrich
      Prospects → Write Your Own Prompt). Zero credits used here.
      DEFAULT.

  (b) Execute via MCP - I call enrich_companies / enrich_contacts
      directly. Requires per-batch confirmation. Test run on 5-10
      first; only scale after you review the test outputs. Spends
      credits.

Reply (a), (b), or "explain again".
```

**If (a) plan-only:** skip to Step 6 (write plan, exit).

**If (b) execute via MCP:**

1. Confirm overwrite policy per column:
   - If the target column name in Saleshandy already exists with values, surface the row count: *"`Career Icebreaker (AI)` has values in K rows. Type `yes, overwrite K rows` to overwrite, `fill empties only` to keep existing, or `cancel`."*
   - The `fill empties only` policy maps to Saleshandy's "Fill in Empty Fields" action - non-destructive.
2. Run a test on 5-10 prospects per column. Show outputs verbatim. Ask: *"Quality OK? (yes / edit prompt / cancel)"*
3. Only after `yes`, prompt for full run: *"Run for all N prospects? Total cost: ~X credits. Type `yes, proceed`."*
4. Invoke `enrich_companies` (for company-level data) or `enrich_contacts` (for person-level data via the MCP). For AI-column content generation (icebreakers, subject lines, etc.) the Saleshandy AI Columns feature isn't directly exposed as an MCP tool - you generate the personalization text yourself (you are the AI), then propose pushing it as a custom field via the import API. Be explicit with the user about this limitation; the plan-only path is usually cleaner here.
5. After execution: append a "## Run log" section to `enrichment-plan.md` with `run_at`, mode, prospects affected, credits spent, and any failures.

### Step 6 - Write `enrichment-plan.md`

Write the plan using the format below. If `enrichment-plan.md` already exists, ask once: *"Existing `enrichment-plan.md` found. (a) overwrite, (b) append run, (c) version (move to `enrichment-plan.v1.md`)?"* Default to (c).

### Step 7 - Suggest next skill

Output one line:

> *Workspace written: `enrichment-plan.md` (K columns designed, ~X credits estimated, mode: plan-only | executed). Recommended next: when these columns are populated, run `email-sequence-generator` to draft a sequence that references them as merge tags.*

## `enrichment-plan.md` format

```yaml
---
generated_at: YYYY-MM-DD
campaign: <campaign>
prospects_in_scope: <int>
columns: <int>
delivery_mode: plan-only | executed
total_credits_estimated: <int>
total_credits_spent: <int or "N/A - plan-only">
---
```

```markdown
# Enrichment Plan - <campaign>

## Goal

<one paragraph from Step 2 - what personalization angle and why>

## Source data summary

- Prospects: <N> (from `leads.md`)
- Available fields: <comma list>
- Missing fields that will skip rows: <field>: <count of prospects missing it>

## Columns

### Column 1: <name>

- **Target field name:** `<name>` (will create new custom field in Saleshandy)
- **Source fields:** `{{First Name}}`, `{{Title}}`, `{{Company}}`
- **Model:** <model>
- **Output format:** Single Sentence | Short Paragraph | Email | etc.
- **Cost per row:** ~<N> credits
- **Estimated rows that will skip** (missing source fields): <count>
- **Overwrite policy:** new column | fill empties only | overwrite N rows (confirmed)
- **Prompt:**

  ```
  <exact prompt, with {{merge tags}} as the user will paste them into Saleshandy>
  ```

### Column 2: <name>

(same structure)

## Cost summary

| Column | Model | Rows | Cost / row | Total |
|---|---|---|---|---|
| 1 | Gemini 2.5 Pro | N | 15 | N×15 |
| 2 | Perplexity Sonar Pro | N | 8.24 | N×8.24 |
| **Total** | | | | **<sum>** |

Test run (5 rows per column, recommended first): <sum_for_5> credits.

## How to ship (plan-only mode)

1. Open Saleshandy → CRM → Enrich Prospects (sparkle icon, top-right).
2. Choose "Write Your Own Prompt" (or pick a matching template).
3. For each column above:
   - Target column: paste the column name verbatim.
   - Model: pick the listed model.
   - Output format: pick the listed format.
   - Paste the prompt.
   - Click "Preview" first - runs on up to 5 prospects. Review.
   - If quality is good: Save → Run for 10 → review again → Run for All.
4. After all columns are populated, reference them as `{{Column Name}}` merge tags in your email sequence steps.

## Run log

(populated only in execute-via-MCP mode)

- <YYYY-MM-DD HH:MM>: ran column 1 test (5 rows). 5 enriched, 0 skipped. 75 credits.
- <YYYY-MM-DD HH:MM>: ran column 1 full (N rows). ...
```

## Edge cases

- **`leads.md` missing:** route user to `lead-finder` first.
- **`icp.md` missing:** proceed if `leads.md` is present, but flag: *"Without `icp.md`, personalization angles will be generic. Run `icp-builder` for better targeting hooks."*
- **Prospect count = 0:** stop. *"`leads.md` shows zero prospects. Re-run `lead-finder` with broader filters."*
- **All proposed source fields missing for every prospect:** stop. *"None of the prospects in `leads.md` have `Title` populated, which all proposed columns require. Run `enrich_contacts` to fill basic firmographics first, or pick a column type that uses fields you do have (City, Company name)."*
- **User requests a column that would overwrite a non-empty existing field:** surface the row count, require explicit typed confirmation (`yes, overwrite K rows`). Hard-coded; cannot be bypassed by flags.
- **Credit balance unknown:** ask user to check the Saleshandy builder header. Don't proceed to execute mode without a stated balance.
- **User asks to skip the test run:** refuse. *"Test runs are non-negotiable. A bad prompt at scale wastes credits faster than anything else."* Offer to lower the test row count from 10 to 5 instead.
- **User wants to chain columns (column B uses column A's output):** explicit support - note in the plan that column B requires column A to be populated first. Order in the plan reflects dependency. Warn the user that they must run column A fully before starting column B.
- **Custom field name conflicts with a standard Saleshandy field (e.g., user names a column `Job Title`):** warn loudly. Suggest a suffix like `Job Title (AI)` to disambiguate.
- **User in execute mode hits 0 successful enrichments on test run:** stop, surface the failures (skipped vs. errored), and revise the prompt or source-field selection before retrying. Do not auto-retry.
