# Inbox Radar (Deliverability Testing)

Inbox Radar is Saleshandy's deliverability testing layer. It measures whether emails land in Inbox / Spam / Other / Undelivered, and surfaces authentication, content, and provider-specific root causes. Use this reference when interpreting a placement report, diagnosing deliverability issues, or running new tests.


## What Inbox Radar measures

Email delivery is not binary. An email may:
- reach the **Inbox** (desired placement)
- land in **Spam** (main danger signal)
- go to **Other** (tabs/categories, alternate placement buckets)
- become **Undelivered** (strongest operational failure)

Inbox Radar answers:
- Are emails actually landing in the inbox?
- Which sender accounts perform better or worse?
- Is the problem content, authentication, or placement behavior?
- Is the issue specific to Gmail, Outlook, or another ESP?
- Should the user update content, pause sending, or fix setup?

In one sentence: **Inbox Radar tells you not just whether an email was sent, but whether it was trusted, delivered, and placed where it matters.**


## Key metrics

### Inbox Score (0-100)
A single summarized score for placement quality.

| Band | Interpretation | Action |
|---|---|---|
| 90-100 | Clean. Accounts landing in inbox. | No immediate action. |
| 50-89 | Mixed. Partial spam placement. | Identify affected ESPs/accounts; recommend top 1-2 fixes. |
| 0-49 | Significant placement issue. | Do not launch campaigns with these accounts until fixed. Run full diagnosis. |

### Placement percentages
- **Inbox %** - desired
- **Spam %** - red flag threshold: **>10% spam = problem**
- **Other %** - tabs/categories; not as bad as spam but not inbox either
- **Undelivered %** - red flag threshold: **>5% undelivered = infrastructure or authentication issue**

### Score is not the whole story
A score like 100 or 67 is useful, but always read:
- Spam %
- Undelivered %
- ESP-to-ESP placements
- Content suggestions
- Authentication tab


## Authentication signals (the technical root-cause layer)

Inbox Radar's Authentication tab shows per-account:

| Field | What it checks | Failure means |
|---|---|---|
| SPF | Sender authorization for the domain | Sending server not authorized for the domain |
| DKIM | Message signing/authenticity | Signing broken, or key not configured |
| DMARC | Policy/alignment | Alignment issue, often consequence of SPF/DKIM gaps |
| PTR (rDNS) | Reverse DNS lookup for sending IP | Reverse DNS not set; reduces trust with receiving servers |
| Domain Blacklist | Domain reputation | Domain flagged; more serious, harder to reverse |
| IP Blacklist | Infrastructure reputation | Sending IP flagged; check sending volume and warmup |

**Mapping authentication gaps to causes:**
- SPF fail -> sending server not authorized
- DKIM fail -> signing broken or key missing
- DMARC fail -> alignment issue, usually downstream of SPF/DKIM
- PTR fail -> reverse DNS missing; **Gmail-to-Gmail paths are especially sensitive to this**
- Domain Blacklist listed -> domain reputation damage (serious)
- IP Blacklist listed -> shared-IP or sending-volume issue


## ESP-to-ESP placement matrix

Inbox Radar measures cross-provider behavior, not just overall placement:

| Sender ESP | Recipient ESP | Inbox % | Spam % |
|---|---|---|---|
| Gmail | Gmail | ... | ... |
| Gmail | Outlook | ... | ... |
| Gmail | Yahoo | ... | ... |

A sender may perform well with one recipient provider and poorly with another. Deliverability cannot be generalized across all recipient environments.

**Diagnostic rule:** If one ESP shows 100% Spam and others show 100% Inbox, the problem is provider-specific, not a global reputation issue.


## Diagnostic process (when something is wrong)

When the user reports a deliverability problem (open rates dropped, spam complaints, sequences underperforming):

### Step 1 - Establish whether a recent test exists
Check Inbox Radar test list for completed tests from the last 7-14 days. If none, ask if the user wants to run one before diagnosing. **Most diagnostic questions can be answered from existing data without a new test.**

### Step 2 - Read the Overview tab
- Overall Inbox% vs Spam% vs Other% vs Undelivered%
- Inbox Score (0-100)
- ESP-to-ESP breakdown
- Placement trend chart: sudden drop or gradual decline?

Key questions:
- Universal across all ESPs, or isolated to one?
- Spam % above 10%? (red flag)
- Undelivered % above 5%? (infrastructure/auth signal)

### Step 3 - Check the Authentication tab
Read SPF, DKIM, DMARC, PTR, Domain Blacklist, IP Blacklist for each sender account. Map failures to root causes (see Authentication table above).

### Step 4 - Check the Detailed Analysis tab
Per-sender-account results and per-ESP breakdowns. Surfaces:
- Which sender accounts have the worst placement?
- Is the problem from one account or spread across all?
- Which specific seed mailboxes received to Spam vs Inbox?
- Are delivery times abnormally long? (could indicate greylisting)

### Step 5 - Present a structured diagnosis

```
Inbox Radar Diagnosis
=====================
Test: [Test Name]
Last run: [Date]

Overall result:
  Inbox: [X]%   Spam: [X]%   Other: [X]%   Undelivered: [X]%
  Inbox Score: [0-100]

Placement breakdown by ESP:
  [Sender] -> [Recipient]:  [X]% Inbox, [X]% Spam
  [mark any rows with Spam > 0% with a flag]

Authentication:
  SPF:              [Pass / Fail]
  DKIM:             [Pass / Fail]
  DMARC:            [Pass / Fail]
  PTR (rDNS):       [Pass / Fail]
  Domain Blacklist: [Clean / Listed]
  IP Blacklist:     [Clean / Listed]

Detailed findings:
  Accounts affected: [list]
  ESPs affected: [list]
  Delivery times: [normal / delayed]

Root cause:
  [Plain-language explanation. Identify whether the root cause is:
   content, authentication, provider-specific behavior, blacklist,
   or infrastructure.]

Recommendations:
  1. [Highest-impact fix]
  2. [Second fix]
  3. [Optional/longer-term improvement]

Next step: Re-run test after fixing [item 1] to confirm improvement.
```


## Diagnostic rules

- **Never present a single score as the full story.** Always read all three tabs.
- **If Spam% is 0% but Undelivered% is high**, prioritize Authentication. Often signals a hard reject at the receiving server.
- **If one ESP shows 100% Spam and others 100% Inbox**, the problem is provider-specific.
- **If all accounts show the same authentication failures**, it's a DNS/infrastructure problem. If only one account fails, it's account-level setup.
- **A low Inbox Score is a diagnosis trigger, not a verdict.** Check the Authentication tab and Detailed Analysis tab before declaring "your deliverability is bad" - the fix is completely different for each root cause.


## Common red flags and their causes

| Red flag | Likely cause | First place to look |
|---|---|---|
| Spam % > 10% | Content triggers, auth gap, or reputation | Authentication tab + content suggestions |
| Spam concentrated on Gmail-to-Gmail | PTR (rDNS) missing or content triggers | Authentication tab (PTR row) |
| Undelivered % > 5% | Hard reject at server level; often auth | Authentication tab |
| One ESP 100% Spam, others Inbox | Provider-specific issue (block, content trigger for that ESP) | ESP-to-ESP matrix |
| All accounts show same auth failures | DNS/infrastructure problem | Domain DNS settings |
| One account fails, others pass | Account-level setup issue | That account's specific config |
| Delivery times abnormally long | Greylisting or deferred delivery | Detailed Analysis tab |
| Score declining gradually over weeks | Reputation eroding (warmup, list quality, sending volume) | Sending behavior history |
| Score dropped suddenly after content change | Content triggers | Content suggestions tab |


## Content suggestions (the prescriptive layer)

Inbox Radar surfaces content-quality issues:

- **Spammy words detected** - e.g., avoid words like "Marketing", urgency language, all-caps
- **Email length outside ideal range** - **ideal: 60-150 words**
- Other setup or authentication issues depending on outcome

If content is flagged, recommend revising before re-running tests.


## Test types

| Type | What it does | When to use |
|---|---|---|
| One-time | Single snapshot | Pre-launch baseline; after a config change |
| Recurring daily | Continuous monitoring | High-volume sending; production accounts |
| Recurring weekly | Ongoing watchdog | Standard health monitoring |
| Recurring monthly | Long-range trend | Slow-tempo accounts; archival monitoring |
| External | User sends manually with a Test ID | When sending outside platform's connected accounts |

**Recurring tests turn Inbox Radar from one-off diagnostics into ongoing monitoring.** Disabling recurrence reverts the test to single-result snapshot.

**Draft state is not a failure state.** A test showing "Draft" or a score of "-" means the test has not been sent or results have not been collected yet, not that deliverability is poor.


## Running a new test (when proactive)

When testing before launch, after new accounts, or after changes:

1. **Confirm accounts** - which sender accounts to test? All of them, or specific ones?
2. **Check monthly quota** - tests are metered. Quota line on main page; warn the user if low.
3. **Configure** - pick recipient ESPs (Gmail, Outlook, Yahoo, Zoho - select all for broadest coverage); pick sender accounts; set frequency (one-time vs recurring).
4. **Set wait-time expectations** - results populate over minutes to hours as seed mailboxes report back. Outlook/Yahoo can take longer.
5. **Interpret on completion** - read all three tabs (Overview, Authentication, Detailed Analysis), present structured diagnosis.


## External tests

External tests are a different mode: the user sends the test email manually (outside the platform's connected-account flow). The test only produces results after the user actually sends.

**Setup logic:**
1. Choose recipient providers
2. Get a Test ID
3. Copy or download the current seed list
4. Manually send the email externally
5. Include the Test ID in the email body
6. Inbox Radar uses the delivered seed results to calculate placement

**Critical rule - seed list freshness:** If the seed account list changes between when the user copies it and when they send, the test shows a warning banner. The user must copy the updated list and resend. **Warn users setting up external tests to send promptly after copying.**


## Operational guardrails

### Hard stops (require explicit confirmation)

| Action | Risk | Required check |
|---|---|---|
| Delete a test | Removes all historical placement data permanently | Confirm: "Deleting this test removes all historical placement data permanently. Confirm?" |
| Share report link | Report becomes publicly accessible without login | Warn: "This link opens for anyone - no login required. Domain info, placement data, and suggestions will be visible." |
| Create external test with stale seed list | Results invalid if old seed list used | Confirm the user knows to copy current seed list and send promptly |

### Irreversible
- Delete test -> all placement history for that test is permanently gone
- Exhaust monthly quota -> cannot run more internal tests until billing reset

### Safe actions
- Creating a new internal test (additive)
- Switching time aggregation (display only)
- Filtering report by email account (display only)
- Viewing any tab (read-only)
- Toggling recurrence on/off (changes future runs only)
- Refreshing a test result (pulls latest)


## High-impact concepts to remember

1. **A low Inbox Score is a diagnostic trigger, not a verdict.** Read all three tabs before deciding the fix.
2. **Authentication and content are separate root causes.** A low score can be content, auth, blacklist/reputation, or provider-specific behavior. Fixes differ completely.
3. **Recurring tests = monitoring mode.** One-time tests = single snapshot.
4. **External tests require manual user action.** They don't produce results until the user sends.
5. **Shared reports are public.** No password protection; anyone with the link sees domain info and placement data.
6. **Monthly quota is a hard ceiling.** When exhausted, no new internal tests until billing reset.


## What Inbox Radar influences

Inbox Radar does not send campaigns directly, but it strongly informs:
- Whether sender accounts should be trusted for production sending
- Whether content should be revised
- Whether authentication needs fixing
- Whether a sequence or account should be paused
- Whether to investigate provider-specific placement issues

It's a **decision-support system** for outreach safety and optimization.
