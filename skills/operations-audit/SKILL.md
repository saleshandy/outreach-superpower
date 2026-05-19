---
name: operations-audit
description: Use when the user wants to audit their cold email setup, audit my deliverability, check sending health, diagnose an open rate drop, review their domains, run an operations audit, or audit their accounts. Triggers on "audit my setup," "audit my deliverability," "check sending health," "diagnose open rate drop," "review my domains," "operations audit," "audit my accounts." Pulls live account/sending data via Saleshandy MCP, cross-references thresholds in references/inbox-radar.md and references/email-account-health.md across 5 audit phases (DNS, account health, placement, engagement, content), writes a prioritized fix list (P0/P1/P2) to audit-YYYY-MM-DD.md. Read-only. No destructive operations.
---

# Operations Audit

Diagnose the sending side of a cold email program. Pull live data, cross-reference thresholds, identify the 1-2 real bottlenecks (not 15 minor ones), produce a prioritized fix list. This skill is read-only - it never sends, pauses, or modifies anything in Saleshandy.

## Inputs

- **Required:** live Saleshandy data via MCP. No workspace prerequisite.
- **Optional flags:**
  - `--account <id or email>` - audit a single sending account (default: all).
  - `--domain <name>` - audit all accounts under a single sending domain.
  - `--sequence <name or id>` - bias the engagement/content phases toward one sequence.
  - `--phases <list>` - skip phases (default: all 5). E.g., `--phases dns,placement` runs only DNS + Placement.

## Outputs

- Writes: `outreach-workspace/<campaign>/audit-YYYY-MM-DD.md` (UTC date). One file per audit run; appends a `-2`, `-3` suffix if multiple audits happen on the same UTC day.

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## References

- `references/email-account-health.md` - **source of truth** for DNS thresholds, setup score bands, warmup status interpretation, sending limits per ESP, the 5-bounce auto-pause rule.
- `references/inbox-radar.md` - **source of truth** for placement red flags (Spam %, Undelivered %), ESP-to-ESP diagnostics, engagement thresholds, and content suggestions (spammy words, length bands).

When citing a threshold in the audit output, cite the reference doc that supplies it. Example: *"Spam % is 14% (red flag at >10% per `references/inbox-radar.md`)."*

## Hard rules

1. **Read-only.** This skill does not send emails, pause accounts, modify sequences, or call any destructive Saleshandy MCP tool. If the audit identifies an account that should be paused, the report says so; the user (or `crm-operations`) takes the action.
2. **Diagnose top-down.** Run phases in order: DNS -> Account Health -> Placement -> Engagement -> Content. A broken DNS finding makes copy critique irrelevant. Don't recommend rewriting headlines if SPF is failing.
3. **Find the 1-2 real bottlenecks.** A 50-issue report is useless. Group findings into P0 (stop-everything), P1 (degrades performance), P2 (polish). The user should never see more than ~3 P0s.
4. **Cite the threshold and the reference.** Every finding includes the measured value, the threshold, and which reference doc supplies the threshold. No generic *"your warmup looks bad."*
5. **Tag every recommendation.** Each fix is tagged `[USER ACTION REQUIRED]` (needs human input - DNS changes, reconnections), `[AGENT]` (handled by another skill in this plugin), or `[QUICK WIN]` (<2 min, high impact).
6. **Never alarm unnecessarily.** If the setup is healthy, say so. Output: *"Setup looks solid across all 5 phases. No P0/P1 findings. See P2 polish items below."* Don't invent problems.

## MCP tools used

All tools live in the Saleshandy MCP server.

- `list_email_accounts` - enumerate every connected sending account (returns DNS status, warmup status, setup score, sending limits).
- `get_email_account_stats` - per-account stats: daily send count, bounce rate, complaint rate, warmup reply rate, recent placement signals.
- `get_consolidated_stats` - org-level rollup: total sends, total opens, total replies, total bounces, broken out by sequence and by sending account.
- `get_sequence_stats` - per-sequence performance: open rate, click rate, reply rate, unsubscribe rate, bounce rate per step.

The skill calls these in a fixed order (see Step 1 below). If the user passes `--account`, `--domain`, or `--sequence` flags, filter the MCP responses accordingly.

## Process

### Step 1 - Pull live data

Call MCP tools in this order. Stop and surface any tool error before continuing - a missing data source changes the audit.

1. `list_email_accounts` -> roster of accounts, their DNS state, warmup status, setup score.
2. `get_email_account_stats` (per account, parallel if possible) -> sending health metrics.
3. `get_consolidated_stats` -> org-level rollup for placement and engagement phases.
4. `get_sequence_stats` (per active sequence) -> per-sequence engagement.

Output a one-line scope summary:

> *Auditing N accounts across M domains, K active sequences. Window: last 30 days unless otherwise noted.*

### Step 2 - Phase 1: DNS authentication

**Reference:** `references/email-account-health.md` (DNS authentication checks section).

For each sending domain in scope, check:

| Check | Threshold | Source |
|---|---|---|
| SPF record present and passing | Must be `pass`. Any other state -> P0. | `email-account-health.md` |
| DKIM record present and passing | Must be `pass`. Any other state -> P0. | `email-account-health.md` |
| DMARC record present at `p=none` minimum | Missing -> P0. `p=none` is acceptable, `p=quarantine`/`p=reject` is stronger. | `email-account-health.md` |
| PTR (rDNS) present | Missing -> P1 (Gmail-to-Gmail paths especially sensitive). | `email-account-health.md` |
| Domain blacklist clean | Listed -> P0 (reputation damage). | `inbox-radar.md` (authentication signals) |
| IP blacklist clean | Listed -> P0 or P1 depending on shared-vs-dedicated. | `inbox-radar.md` |

For every failing check, provide the **exact** DNS record to add (not generic instructions). Example DMARC:

```
Type:  TXT
Name:  _dmarc
Value: v=DMARC1; p=none; rua=mailto:dmarc@<domain>
```

State the propagation expectation: 24-48 hours.

### Step 3 - Phase 2: Account health

**Reference:** `references/email-account-health.md` (Setup Score, Warmup status, Daily sending limits sections).

For each account in scope:

| Check | Threshold | Severity |
|---|---|---|
| Setup Score | <80 -> flag and fix. 80-89 -> P2 tighten. 90-100 -> healthy. | P0 if <80, P2 if 80-89 |
| Warmup status | Must be `Complete` AND setup score 80+ for cold sends. `Stopped`/`Paused`/`Active (Day X/21)` with active campaigns -> P0. | P0 if cold campaigns running on un-warmed account |
| Warmup reply rate | Target 30%+. Below 20% -> P1 (reputation problem). | P1 |
| Bounce rate (last 30 days) | <1% strong, 1-2% average, 2-3% needs attention, >3% P0 pause. | P0 if >3%, P1 if 2-3% |
| Spam complaint rate | <0.05% strong, 0.05-0.1% average, >0.1% P1, >0.3% P0 (Gmail/Outlook industry standard). | P0 if >0.3%, P1 if 0.1-0.3% |
| Daily volume vs ESP cap | Per `email-account-health.md` daily limits table. >cap -> P0. >recommended -> P1. | P0 if >cap |
| Account in multiple active sequences | Recommended: each account in ONE active sequence. Violations -> P2. | P2 |
| 5-bounce auto-pause rule | If any account auto-paused, surface immediately - investigate content/DNS/list. | P0 |

For any account with setup score <80, enumerate **which** of the 6 setup components is missing (SPF, DKIM, DMARC, Custom Tracking Domain, Email Signature, Warmup enabled). Provide the exact fix for each.

### Step 4 - Phase 3: Placement

**Reference:** `references/inbox-radar.md` (Key metrics, ESP-to-ESP placement matrix, Common red flags sections).

Check the most recent Inbox Radar test results (last 7-14 days). If no recent test exists, recommend the user run one before relying on this phase's findings.

| Check | Threshold | Severity |
|---|---|---|
| Inbox Score (0-100) | 90-100 clean, 50-89 mixed, 0-49 significant issue. | P0 if <50, P1 if 50-89 |
| Spam % | >10% is a red flag. | P0 if >10% |
| Undelivered % | >5% is infrastructure / auth failure. | P0 if >5% |
| ESP-to-ESP placement | One ESP showing 100% Spam while others Inbox -> provider-specific issue. Investigate that ESP path. | P1 (provider-specific) |
| All accounts show same auth failures | DNS-level infrastructure problem (not per-account). | P0 |
| One account fails, others pass | Account-level setup problem. | P1, scoped to that account |
| Delivery times abnormally long | Greylisting / deferred delivery. | P2 monitor |
| Score declining gradually over weeks | Reputation eroding (warmup, list quality, sending volume). | P1 |

When citing placement findings, always include the ESP-to-ESP breakdown when one is available - a global Spam% of 8% can hide a Gmail-to-Outlook path running at 60% spam.

### Step 5 - Phase 4: Engagement

**Reference:** `references/inbox-radar.md` for thresholds; cross-checked with `get_consolidated_stats` and `get_sequence_stats`.

Per-sequence and org-level metrics over the last 30 days:

| Metric | Strong | Average | Needs attention | Broken | Severity |
|---|---|---|---|---|---|
| Open rate | 45-55%+ | 30-45% | 20-30% | <20% | P0 if <20%, P1 if 20-30% |
| Reply rate | 5%+ | 2-4% | 1-2% | <1% | P0 if <1%, P1 if 1-2% |
| Positive reply rate | 2-3%+ | 1-2% | 0.5-1% | <0.5% | P1 if <0.5% |
| Click rate | n/a (recommend tracking off for cold) | - | - | - | P2 if click tracking is on |
| Unsubscribe rate | <0.1% | 0.1-0.3% | 0.3-1% | >1% | P0 if >1%, P1 if 0.3-1% |
| Bounce rate | <1% | 1-2% | 2-3% | >3% | P0 if >3% |

Diagnostic rules to apply:

- **Low opens (<20%) + clean DNS + clean placement** -> targeting or subject-line issue, not deliverability.
- **High opens (>40%) + near-zero replies (<1%)** -> copy / personalization / ICP mismatch, not deliverability.
- **All replies negative** -> ICP misalignment. Flag for `icp-builder` re-run.
- **Sudden drop after a working baseline** -> recent change (new domain, volume spike, content change). Surface the change first.
- **Click rate on cold email** -> recommend turning off click tracking (URL redirects flag spam filters; cold emails should have minimal links anyway).

### Step 6 - Phase 5: Content

**Reference:** `references/inbox-radar.md` (Content suggestions section).

For each active sequence (and each step within it), scan for:

| Check | Threshold | Severity |
|---|---|---|
| Spam-trigger words in subject or body | Any present -> P1, list which words. | P1 |
| Body length outside 60-150 word ideal range | <40 or >200 -> P1, suggest target range. | P1 |
| Subject length >7 words or >60 characters | -> P2 polish. | P2 |
| Spintax presence | <1 block per email -> P1 (duplicate content risk across sends). | P1 |
| HTML formatting / images / logos in cold emails | Any present -> P0, rewrite to text-only. | P0 |
| Links in first email (step 1) | >0 links -> P1, remove or move to step 2+. | P1 |
| Open tracking enabled on cold sequences | -> P2 recommend off (pixel hurts placement, data unreliable). | P2 |
| Click tracking enabled on cold sequences | -> P2 recommend off. | P2 |
| Missing merge tag fallbacks (e.g., `{{First Name}}` with no fallback like `{{First Name|there}}`) | -> P2. | P2 |

Don't critique copy quality at the headline/hook level - that's `email-auditor`'s job. This phase is structural / deliverability-oriented only. If the user wants creative critique, recommend running `email-auditor` against each step.

### Step 7 - Synthesize the prioritized fix list

Collapse all findings into **one ordered list**. Within each priority band, order by impact (deliverability > engagement > polish).

Group rule:

- **P0** - stop-everything issues. Cold sends should pause until resolved. Examples: missing SPF, bounce rate >3%, account auto-paused, sending from primary domain, Inbox Score <50.
- **P1** - meaningful drag on performance. Fix this week. Examples: warmup reply rate <20%, low open rate masking a placement issue, missing PTR, spam-trigger words throughout copy.
- **P2** - polish items. Schedule for next sprint. Examples: subject length, missing merge tag fallbacks, click tracking still on, account in multiple sequences.

If a finding fits two bands (e.g., bounce rate 2.5% straddles P1/P0), choose the more severe.

Cap the list at 12 items in chat. If more were found, write them all to the file but summarize with *"+N more lower-priority items in the file."*

### Step 8 - Write `audit-YYYY-MM-DD.md`

Use the format below. UTC date for the filename. If a file with the same name exists, suffix `-2`, `-3`, etc.

### Step 9 - One-sentence verdict + next check-in

Open the chat output with a one-sentence verdict that connects the dots:

> *"Your deliverability is solid but your ICP is too broad and your opening lines are generic - that's why you're getting opens but no replies."*

Close with a check-in timeline:

> *"Re-audit after 200 sends (roughly 5-7 days) or after you implement the P0 fixes. Run `operations-audit` again with the same flags."*

If `crm-operations` would help action a finding (e.g., remove bouncing prospects), suggest chaining to it. If `email-auditor` would help on a content finding, suggest that. The audit itself is read-only - downstream skills do the writes.

## `audit-YYYY-MM-DD.md` format

```yaml
---
audited_at: YYYY-MM-DDTHH:MMZ
campaign: <campaign>
scope:
  accounts: <count or "all">
  domains: <list>
  sequences: <list or "all">
phases_run: [dns, account_health, placement, engagement, content]
findings:
  p0: <count>
  p1: <count>
  p2: <count>
verdict: "<one-sentence verdict>"
---
```

```markdown
# Operations Audit - <campaign> - YYYY-MM-DD

## Verdict

<one sentence>

## Scope

- Accounts in scope: <list>
- Domains: <list>
- Sequences: <list>
- Window: last 30 days

## Prioritized fix list

### P0 (stop-everything)

#### 1. <finding title>
- **Phase:** <DNS | Account Health | Placement | Engagement | Content>
- **Severity:** P0
- **Current value:** <measured>
- **Threshold:** <from reference doc>
- **Reference:** `references/<file>.md`
- **Recommended action:** <specific fix>
- **Tag:** [USER ACTION REQUIRED] | [AGENT] | [QUICK WIN]
- **Effort:** <e.g., "5 min + 24-48h DNS propagation">

#### 2. <next P0>
(same structure)

### P1 (fix this week)

(same per-finding structure)

### P2 (polish)

(same per-finding structure)

## What's working

(call out healthy signals so the user doesn't change things that don't need changing)

- DKIM passing across all accounts.
- Warmup running on all accounts >21 days.
- Bounce rate at 1.2% (under 2% threshold).
- ...

## Phase-by-phase data appendix

(verbatim metrics pulled from MCP, for traceability)

### Phase 1: DNS
<table per account: SPF / DKIM / DMARC / PTR / blacklist status>

### Phase 2: Account Health
<table per account: Setup Score / Warmup / Bounce % / Complaint % / Daily volume>

### Phase 3: Placement
<Inbox Score / Spam % / Undelivered % / ESP-to-ESP matrix>

### Phase 4: Engagement
<per-sequence: Open % / Reply % / Positive Reply % / Bounce % / Unsub %>

### Phase 5: Content
<per-sequence-step: word count / spintax blocks / spam words / links / tracking on-off>

## Next check-in

Re-audit after 200 sends or 5-7 days. Run `operations-audit` with the same flags to compare.
```

## Edge cases

- **No accounts connected:** stop. *"No sending accounts found via `list_email_accounts`. Connect at least one account before running an audit."*
- **No active sequences:** Phases 4 and 5 skip with a note. Phases 1-3 still run.
- **No recent Inbox Radar test in last 14 days:** Phase 3 produces a flag *"No recent placement test. Recommend running an Inbox Radar test before relying on placement data."* Other phases still run.
- **MCP tool error mid-audit:** surface the error verbatim, skip the affected phase, mark it as `inconclusive` in the report. Do NOT silently substitute or estimate.
- **User passes `--phases dns` only:** run just that phase. The verdict reflects partial scope.
- **Audit run twice on the same UTC day:** suffix the filename `-2`, `-3`. Do NOT overwrite the morning audit with the afternoon one.
- **All findings clean (P0 = P1 = 0, only P2):** lead with *"Setup looks solid."* List the P2 polish items, but make clear no urgent action is needed.
- **Account sending from primary company domain:** this is **always** P0 regardless of other metrics. Flag it loudly and recommend a subdomain or separate sending domain.
- **5-bounce auto-pause triggered on an account:** surface immediately at the top of P0 with the likely root causes (content flagged, URLs blocked, auth issue, invalid addresses, ESP restriction). Suggest the user investigate before reconnecting.
- **`get_consolidated_stats` and `get_sequence_stats` disagree** (e.g., on bounce rate): use sequence-level for per-sequence findings, consolidated for org-level. Cite which one in the report.
- **Plan limit blocks an MCP call:** surface the limit, skip the phase, recommend the relevant upgrade path. Do NOT estimate around it.
