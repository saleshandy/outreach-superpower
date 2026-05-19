# Email Account Health

Sender account health, warmup status, DNS authentication, and common diagnostic patterns. Use this reference when auditing account setup, interpreting health scores, troubleshooting deliverability, or planning sender allocation.


## Account health columns (what to read first)

Every connected email account exposes:

- Sender Name / Email
- Active Sequences (count)
- Inbox Radar Test (link to placement report)
- Warm-up Status (Stopped / Active / Paused / Complete)
- Deliverability Score
- Setup Score (0-100)
- Inbox Score
- SPF (pass/fail)
- DKIM (pass/fail)
- DMARC (pass/fail)
- PTR / rDNS (pass/fail)


## Setup Score (0-100)

Setup Score is the composite health metric. It checks **6 things per account**:

1. SPF record configured
2. DKIM record configured
3. DMARC record configured
4. Custom Tracking Domain (CTD) configured
5. Email Signature set
6. Warmup enabled

| Band | Interpretation | Action |
|---|---|---|
| 90-100 | Healthy. Safe for production. | Continue. |
| 80-89 | OK but worth tightening. | Flag minor gaps. |
| <80 | **At risk. Flag and fix.** | Identify which of the 6 is missing; provide exact DNS/setting fix. |

**Target: 100% (all 6 green).** Below 80% is the threshold where the account should be flagged and a fix proposed before continued use.


## Warmup status

| Status | What it means | Ready for cold sends? |
|---|---|---|
| Stopped | Warmup never started, or disconnected | No |
| Active (Day X/21) | Currently warming, day X of warmup period | No |
| Paused | Manually paused | No |
| Complete | Warmup period finished | Yes, if setup score is also 80+ |

**Ready = warmup shows "Complete" AND setup score is 80+.**

### Warmup defaults and best practices

| Setting | Recommended | Why |
|---|---|---|
| Duration before cold sends | 3-4 weeks minimum | Build reputation with ESPs |
| Keep running alongside cold sends | Yes, always | Maintains reputation |
| Reply rate target | 30%+ | Below 20% = problem |
| When to enable ramp-up | After 3 weeks warmup | Not before |
| Ramp-up start | 5-10 emails/day | Gradual increase |
| Ramp-up increment | 10-20% daily | Slow and steady |

**Hard rule:** Never recommend cold sending before 3 weeks of warmup. Non-negotiable.

### Warmup troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Warmup reply rate < 20% | Account reputation low, or DNS issues | Check SPF/DKIM/DMARC. Reduce cold send volume. |
| Warmup status "Stopped" | TrulyInbox disconnected or account disconnected | Reconnect TrulyInbox; check account connection. |
| Warmup not improving after 3 weeks | Domain reputation damaged | Pause cold sends entirely. Warmup only for 2 more weeks. Check blacklist status. |
| "Warmup complete" but score still low | Other factors: bounces, spam complaints, DNS | Warmup isn't the only factor. Run full diagnostic. |


## DNS authentication checks

The 4 records that govern sender trust:

| Record | What it checks | Failure means |
|---|---|---|
| SPF | Sender authorization for the domain | Sending server not authorized |
| DKIM | Message signing/authenticity | Signing broken, key not configured |
| DMARC | Policy/alignment | Alignment issue (often downstream of SPF/DKIM gaps) |
| PTR (rDNS) | Reverse DNS lookup for sending IP | Reverse DNS not set; reduces receiving-server trust. **Gmail-to-Gmail paths are especially sensitive.** |

### When fixing DNS, provide exact records

Generic instructions are not enough. Provide the exact value to add. Example DMARC record:

```
Type:  TXT
Name:  _dmarc
Value: v=DMARC1; p=none; rua=mailto:dmarc@<their-domain>
```

DNS propagation takes 24-48 hours. **Set this expectation explicitly** before re-running checks.


## The 4 common diagnostic patterns

### Pattern 1: Account keeps disconnecting

Distinguish three states - they have different remediations:

| State | Symptom | Fix | Side effect |
|---|---|---|---|
| Disconnected | Toggle off, credentials expired | Reconnect | Prospects in active sequences resume |
| Paused | Manually paused | Toggle back on | None (no credentials needed) |
| **DELETED** | Account removed entirely | Cannot re-link the original | **Prospects in active sequences are orphaned. Must delete and re-add prospects.** Always warn. |

Most common cause for Google accounts: OAuth tokens revoked periodically. Reconnection must include ALL permission checkboxes. Partial permissions = immediate disconnection.

For SMTP/IMAP: verify App Password is still valid (not the regular password).

### Pattern 2: Setup Score dropped

Investigation order:

1. **DNS records:** SPF / DKIM / DMARC / PTR. Identify which is now failing.
2. **Sending behavior:**
   - Daily volume vs limit
   - **Bounce rate >3% = investigate. >5% = pause immediately.**
   - Spam complaints rate
   - Is this account in multiple sequences? (splits capacity, makes diagnosis harder)
3. **Warmup status:**
   - Is warmup still running?
   - What's the warmup reply rate? (target 30%+; below 20% = problem)

**Protective actions** to take while diagnosing a score drop:
- Pause cold sends from the account
- Remove from Sender Rotation in active sequences
- Increase warmup ratio
- After DNS fix + propagation: re-check and re-enable if healthy

### Pattern 3: Account allocation across sequences

**Principle: each account should ideally be in ONE active sequence.** Sharing accounts across sequences splits daily capacity and makes deliverability diagnosis much harder.

**Allocation rules:**
- 1 domain (typically 3 accounts) per sequence when possible
- Never put the same account in multiple active sequences
- Keep 1-2 domains in reserve for new campaigns
- Warming accounts -> no sequences yet
- Match domain reputation to sequence priority: healthiest accounts -> most important sequence

### Pattern 4: Account auto-paused (5-bounce rule)

Saleshandy auto-pauses an account after **5 consecutive block bounces**.

| Bounce type | Cause | Severity | Action |
|---|---|---|---|
| Hard bounce | Email address doesn't exist | Permanent | Remove prospect. Verify lists better. |
| Block bounce | Sender ESP blocking | Systematic | Investigate content, DNS, list quality |
| Soft bounce | Temporary (mailbox full, server down) | Temporary | Saleshandy retries automatically |

After 3 retries, block bounce reclassifies as hard bounce.

**5 block bounces in a row means something systematic is wrong.** Possible causes:
- Content flagged as spam by sending ESP
- URLs in email blocked by ESP
- Authentication issue (SPF/DKIM/DMARC)
- High rate of unknown/invalid addresses
- ESP security restriction activated

**If this is the only account in a sequence, the entire sequence is paused too.**


## Daily sending limits

| ESP | Recommended | Hard limit | Notes |
|---|---|---|---|
| Gmail / Microsoft (personal) | 15/day | 50/day | Personal accounts |
| GSuite / O365 (business) | 15/day | 150/day | Business accounts |
| Yahoo | 15/day | 1,000/day | High limit, deliverability varies |
| Zoho | 15/day | 2,000/day | - |
| GoDaddy | 15/day | 800/day | - |
| Sendgrid / other transactional | 15/day | 4,000/day | Transactional ESPs |

Saleshandy-purchased Google Workspace and Microsoft 365 accounts: **30/day in-platform limit**.

**Time between emails:** Random 60-190 seconds (default). Roughly 60/hour max. A 9-hour sending window yields a theoretical ~540/day maximum (further limited by per-account caps).

### Ramp-up configuration

Toggle ON after warmup period (3+ weeks). Recommended:
- Start: 5-10 emails/day
- Daily increment: 10-20%
- Never exceeds the account's maximum limit
- Includes both new emails AND follow-ups
- Example: start 10, +10% daily -> reaches max in ~33 days


## ESP matching

Saleshandy can match sender ESP with recipient ESP for better deliverability:
- Google account -> sends to Gmail recipients preferentially
- Microsoft account -> sends to Outlook/Microsoft recipients preferentially

When setting up Sender Rotation, mix ESPs if possible. Saleshandy auto-matches sender to recipient ESP for better inbox placement.


## Connection methods (reference)

### Google OAuth (recommended for Gmail/GSuite)

- Critical: check ALL permission checkboxes during OAuth. Partial permissions = connection failure or disconnection.
- "Could not authenticate" usually means 2-Step Verification not enabled, or wrong password type (must be App Password).

### SMTP/IMAP (any provider)

Standard hosts:
- Gmail: smtp.gmail.com, port 587 (TLS) or 465 (SSL)
- Outlook/O365: smtp.office365.com, port 587
- Yahoo: smtp.mail.yahoo.com, port 465/587
- Zoho: smtp.zoho.com, port 465/587
- GoDaddy: smtpout.secureserver.net, port 465/587
- Yandex: smtp.yandex.com, port 465

Gmail SMTP requires App Password, not regular password. Test SMTP and IMAP separately before saving.

### Bulk SMTP/IMAP via CSV

Max 100 emails per CSV, 5MB max.


## Behavior rules

1. **Always check all accounts proactively** when the user mentions account health.
2. **Flag accounts below 80 setup score** with specific issues and fixes.
3. **Distinguish disconnected vs deleted.** Deleted = prospects orphaned. Always warn.
4. **Never recommend cold sending before 3 weeks warmup.** Non-negotiable.
5. **Always recommend keeping warmup running** alongside cold sends.
6. **Recommend 1 domain per sequence** for clean separation. Never the same account in multiple active sequences.
7. **Provide exact DNS records** when fixing SPF/DKIM/DMARC, not generic instructions. Include exact values for the user's ESP.
8. **Explain auto-pause clearly** when it happens. Users often don't know what "5 consecutive block bounces" means.
9. **Check bounce rate during every diagnostic.** Above 3% = investigate. Above 5% = pause immediately.
10. **Detailed warmup metrics live in TrulyInbox externally** (the Warm-up Dashboard button opens TrulyInbox).
11. **Always check if an account is in multiple active sequences** - this is a common cause of problems.
12. **When allocating accounts to sequences:** healthiest -> highest priority. Warming accounts -> no sequences yet. Reserve pool for new campaigns.
