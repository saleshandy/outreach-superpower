---
name: analytics-interpreter
description: Use when the user wants to explain my metrics, understand what's working, diagnose a drop in opens, analyze my campaigns, interpret my sequence stats, see a performance report, or get an analytics summary. Triggers on "explain my metrics," "what's working," "why did opens drop," "interpret my sequence stats," "analyze my campaigns," "what should I A/B test next," "diagnose drop in opens," "performance report," "analytics summary." Pulls live sequence/account data via Saleshandy MCP, computes 30-day baselines, flags anomalies (>1 stddev or >20% relative change), generates hypotheses, suggests A/B tests. Writes analytics-snapshot-YYYY-MM-DD.md. Read-only.
---

# Analytics Interpreter

Read live Saleshandy analytics, compare to baseline, surface what changed, generate root-cause hypotheses, and propose 1-3 A/B tests prioritized by expected lift. This skill is read-only - it never modifies sequences, prospects, or accounts.

## Inputs

- **Required:** live Saleshandy data via MCP. No workspace prerequisite.
- **Optional flags:**
  - `--sequence <id or name>` - restrict analysis to one sequence (default: all active sequences).
  - `--account <id or email>` - restrict analysis to one sending account.
  - `--days N` - lookback window for the current period (default: 7 days for diagnostics, 30 for full reports).
  - `--baseline-days N` - rolling window for the baseline calculation (default: 30 days, ending where the current window starts).

## Outputs

- Writes: `outreach-workspace/<campaign>/analytics-snapshot-YYYY-MM-DD.md` (UTC date). One snapshot per run. If two runs happen on the same UTC day, suffix `-2`, `-3`, etc.

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## Hard rules

1. **Read-only.** No destructive MCP calls. If a recommendation requires action (pause a sequence, remove an account), the snapshot says so and the user takes the action through `crm-operations` or directly in Saleshandy.
2. **Cite the metric, the baseline, the threshold.** Every anomaly entry shows: current value, baseline value, % change, the threshold that flagged it. No vague "opens look low."
3. **Cite the "until today" caveat.** Saleshandy counts Opens/Replies/Clicks cumulatively through today regardless of the date filter. Always state this when presenting period-bounded engagement numbers, so users interpret correctly.
4. **Anomaly cap.** Surface no more than 5 anomalies in chat. Write all of them to the snapshot file. Triage by impact: deliverability > engagement > polish.
5. **Hypotheses must be testable.** Each hypothesis names what to check next (a specific MCP call, a specific setting, a specific sequence step). No "maybe it's deliverability."
6. **A/B test suggestions must specify the variable, the variant, and the success metric.** No "try different subjects."

## MCP tools used

All tools live in the Saleshandy MCP server.

- `list_sequences` - enumerate sequences (filter to active for the current period).
- `get_sequence_stats` - per-sequence performance: open rate, click rate, reply rate, unsubscribe rate, bounce rate, per step.
- `get_consolidated_stats` - org-level rollup: total sends, opens, replies, bounces, broken out by sequence and sending account.
- `get_email_account_stats` - per-account: daily send count, bounce rate, complaint rate, warmup reply rate.

## Process

### Step 1 - Define the windows

Set two windows before pulling data:

- **Current window:** last `--days N` (default 7 for "why did X drop" intent, 30 for general reports). Anchored to "now" in UTC.
- **Baseline window:** previous `--baseline-days N` ending where the current window starts (default 30 days). E.g., if current is days 0-6, baseline is days 7-36.

State both windows in the chat header so the user can confirm the scope.

### Step 2 - Pull live data

Call MCP tools in this order. Surface any error verbatim - do not fabricate a baseline if a call fails.

1. `list_sequences` - get all active sequences. Apply `--sequence` filter if set.
2. `get_sequence_stats` per active sequence, for BOTH windows.
3. `get_consolidated_stats` - org-level rollup for both windows.
4. `get_email_account_stats` per account (parallel if possible), for both windows.

Output a one-line scope summary:

> *Reading N sequences and M accounts. Current window: <date>-<date>. Baseline: <date>-<date>.*

### Step 3 - Compute baselines and deltas

For each metric (open rate, reply rate, positive reply rate, click rate, bounce rate, unsubscribe rate), per sequence and per account:

1. Compute the baseline mean over the baseline window.
2. Compute the baseline standard deviation (rolling daily values if granularity allows; otherwise period-over-period).
3. Compute the current-window value.
4. Compute: absolute delta, relative delta (%), and z-score (deltas in stddevs).

### Step 4 - Flag anomalies

An anomaly meets ANY of these conditions:

- z-score > 1.0 (current is more than 1 stddev from baseline mean), OR
- relative change > 20% in either direction, OR
- current value crosses a benchmark band (e.g., open rate fell from "good" 35-50% into "needs attention" 20-35%).

**Direction matters.** A drop in opens is an anomaly. A spike in opens is also an anomaly (could be a deliverability shift, a fixed bug, or a tracking pixel issue worth understanding).

### Step 5 - Apply diagnostic logic

For each anomaly, walk the decision tree to propose hypotheses. Cite which step in the tree fired.

**Open rate dropped:**
1. Subject lines: changed recently? Spam-trigger words? Length >7 words or >60 chars?
2. Sender accounts: any new accounts (warmup not complete)? Bounce rate spike?
3. DNS / placement: ran an operations-audit recently? Inbox Score dropped?
4. Timing: send window changed? Holiday or off-peak?
5. List quality: new prospect batch added? Pre-verified before enrollment?

**Open rate spiked:**
1. Tracking pixel restored after being off?
2. Auto-responder flood (e.g., post-holiday OOO replies inflate "opens" via image-loading clients)?
3. List shift: same audience, or new ICP that's more pixel-friendly (Gmail vs Outlook ratio)?
4. New sequence with much better targeting?

**Reply rate dropped, opens stable:**
1. Body copy: changed recently? Too long? Multiple CTAs?
2. CTA quality: question vs command? Specific time vs "let's chat"?
3. Personalization: AI columns weakening? Spintax fatigue?
4. ICP fit: same audience or new segment with different intent?

**Reply rate spiked, opens stable:**
1. New copy variant winning? Confirm with per-step stats.
2. ICP narrower / better matched?
3. Seasonal: budget cycle, conference season?

**Bounce rate spiked:**
1. New prospect batch added without verification?
2. One account causing it (per-account stats)?
3. DNS / sender reputation eroded (cross-check with `operations-audit`)?

**Unsubscribe rate spiked:**
1. Step volume increased (over-emailing)?
2. ICP misalignment (wrong audience)?
3. Subject lines too aggressive?

**Positive sentiment dropping (replies still coming):**
1. Tone shift in latest copy variant?
2. ICP drift (replying audience changed)?

### Step 6 - Generate A/B test suggestions

For each anomaly, suggest 1-3 A/B tests **only if** the hypothesis is testable via a sequence variant or setting change. Format:

```
A/B test #1: <name>
Variable:    <what changes> (e.g., subject line, send time, body framing, CTA)
Variant A:   <current state>
Variant B:   <proposed change with specific text or setting>
Success:     <metric to watch> (e.g., open rate, reply rate)
Minimum sample: <sample size for confidence, typically 200-500 sends per variant>
Estimated lift: <if any prior pattern suggests one; otherwise "unknown - measure first">
```

Cap suggestions at 3 across the whole snapshot. Prioritize by:
1. Anomaly severity (z-score magnitude * impact direction).
2. Sample size available (sequences with >500 sends can finish a test quickly).
3. Ease of implementation (subject variant > body rewrite > whole-sequence rebuild).

### Step 7 - Benchmark every number

Pair every reported metric with its benchmark band so the user can interpret. Bands per `references/email-account-health.md` and `references/inbox-radar.md`:

| Metric | Excellent | Good | Needs attention | Broken |
|---|---|---|---|---|
| Open rate | >50% | 35-50% | 20-35% | <20% |
| Reply rate | >5% | 3-5% | 1-3% | <1% |
| Positive reply rate | >60% of replies | 40-60% | 20-40% | <20% |
| Click rate | >3% (if tracking on) | 1-3% | 0.5-1% | <0.5% |
| Bounce rate | <0.5% | 0.5-1% | 1-3% | >3% |
| Unsubscribe rate | <0.5% | 0.5-1% | 1-2% | >2% |

State both the value and the band: *"Open rate is 28% (needs attention, 20-35% band)."*

### Step 8 - Write the snapshot file

Use the format below. UTC date for the filename.

### Step 9 - Chat output

Open with a one-sentence verdict that connects the dots. Then list up to 5 anomalies in priority order. Then list up to 3 A/B test suggestions. Close with a chaining recommendation (e.g., "Run `operations-audit` to investigate the bounce spike," or "Run `email-auditor` on Step 1 copy if you want a creative critique").

## `analytics-snapshot-YYYY-MM-DD.md` format

```yaml
---
snapshot_at: YYYY-MM-DDTHH:MMZ
campaign: <campaign>
current_window: { from: <date>, to: <date>, days: N }
baseline_window: { from: <date>, to: <date>, days: N }
scope:
  sequences: <count or list>
  accounts: <count or list>
verdict: "<one-sentence verdict>"
anomaly_count: <N>
ab_tests_suggested: <N>
---
```

```markdown
# Analytics Snapshot - <campaign> - YYYY-MM-DD

## Verdict

<one sentence>

## Scope

- Sequences: <list>
- Accounts: <list>
- Current window: <date>-<date> (<N> days)
- Baseline window: <date>-<date> (<N> days)
- Note: Saleshandy counts Opens/Replies/Clicks cumulatively through today regardless of the date filter. Engagement numbers may include events that occurred after the window end.

## Headline numbers

| Metric | Current | Baseline | Delta | Band | Notes |
|---|---|---|---|---|---|
| Open rate | <%> | <%> | <+/- %> | <band> | <if anomaly> |
| Reply rate | <%> | <%> | <+/- %> | <band> | |
| Positive reply rate | <%> | <%> | <+/- %> | <band> | |
| Bounce rate | <%> | <%> | <+/- %> | <band> | |
| Unsubscribe rate | <%> | <%> | <+/- %> | <band> | |

## Anomalies (priority order)

### 1. <anomaly title>
- **Scope:** <sequence name or "all sequences" or per-account>
- **Metric:** <metric>
- **Current:** <value> (<band>)
- **Baseline:** <value> over <N> days
- **Delta:** <relative %> (<absolute>)
- **z-score:** <value>
- **Hypotheses:**
  1. <hypothesis with specific next check>
  2. <hypothesis with specific next check>
- **Suggested next:** <skill chain or A/B test reference>

### 2. <next anomaly>
(same structure)

## A/B test suggestions

### Test 1: <name>
- **Variable:** <what changes>
- **Variant A:** <current>
- **Variant B:** <proposed, with specific copy or setting>
- **Success metric:** <which metric>
- **Minimum sample:** <N sends per variant>
- **Estimated lift:** <if any>

### Test 2: <name>
(same)

## What's working

(call out healthy signals so user doesn't break what's not broken)

- <healthy metric 1 with band>
- <healthy metric 2 with band>

## Derived insights

| Insight | Value | What it means |
|---|---|---|
| Reply-to-open ratio | <%> | Body copy quality (once opened, do they reply?) |
| Meeting-to-reply ratio | <%> | Reply handling quality (where applicable) |
| Pipeline per prospect | $<X> | Revenue efficiency (where deal values populated) |

## Recommended next

- <chained skill or action>
- <chained skill or action>
```

## Edge cases

- **No sequences active:** *"No active sequences in window. Nothing to analyze. Activate a sequence and re-run after 5-7 days of sends."* Exit cleanly.
- **Baseline window has insufficient data** (e.g., new sequence with only 5 days of history): widen baseline to all available, note the limitation in the snapshot, and flag any anomaly as "low-confidence baseline."
- **MCP tool error mid-run:** surface the error verbatim, skip the affected sequence/account, mark it `inconclusive` in the snapshot. Do NOT estimate or interpolate.
- **Single sequence with <100 sends in window:** sample size too small for reliable anomaly detection. Note "small sample - directional only" on any flagged anomaly.
- **User asks "why did opens drop" but opens are actually fine:** state the open rate and its band, note that no drop was detected against baseline, and ask if they were comparing to a different reference point.
- **Snapshot run twice on same UTC day:** suffix filename `-2`, `-3`. Never overwrite.
- **`get_consolidated_stats` and `get_sequence_stats` disagree on a number:** prefer sequence-level for per-sequence findings, consolidated for org rollup. Cite which one was used.
- **All anomalies are positive (everything got better):** lead with *"All metrics improved or held steady against baseline."* Still surface the changes so the user understands what happened and can lean into it.
- **User says "interpret stats" with no scope:** default to last 7 days vs. previous 30 days, all active sequences, all accounts. State the defaults so they can override.
- **`get_email_account_stats` returns plan-limit error:** skip the account-level phase, surface the limit, recommend upgrade. Don't estimate around it.
- **A flagged anomaly correlates across multiple metrics** (e.g., bounce up + open down): collapse into one root-cause finding rather than three. Show the correlation in the snapshot.
