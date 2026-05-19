---
name: inbox-triage
description: Use when the user wants to review replies, triage their inbox, categorize replies, decide who to respond to, process unread replies in bulk, review unread, or handle replies. Triggers on "review replies," "what should I reply to," "triage my inbox," "categorize replies," "process inbox," "review unread," "handle replies." Reads live Saleshandy unified inbox via MCP, classifies each reply across 8 categories (interested, not-now, objection, OOO, auto-responder, unsubscribe, forwarded, referral), drafts a per-reply suggested action, sends replies only after explicit per-message confirmation, appends every decision to inbox-log.md.
---

# Inbox Triage

Walk through new prospect replies in the Saleshandy unified inbox, classify each one, propose a suggested action (and draft reply where appropriate), and only send a response after the user explicitly approves it. Append every decision to `inbox-log.md` so the campaign has a paper trail.

## Inputs

- **Required:** live Saleshandy data via MCP. No workspace prerequisite.
- **Optional:** `outreach-workspace/<campaign>/sequence.md` for sequence context (which step prompted the reply, what the original goal was).
- **Optional flags:**
  - `--since <duration>` - default `24h`. Accepts `24h`, `7d`, etc.
  - `--unread-only` - default `true`. Restricts to unread threads.
  - `--account <email>` - limit to a single sending account.
  - `--sequence <name or id>` - limit to a single sequence.

## Outputs

- Appends to: `outreach-workspace/<campaign>/inbox-log.md` (one row per reply triaged).

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## References

- See `references/email-account-health.md` for sender-side context when a reply hints at deliverability problems (bounce notice, postmaster complaint).

## Hard rules

1. **`reply_to_email` requires explicit per-message confirmation.** The user must type `send #N` (or equivalent) for every individual reply before that MCP call is made. No exceptions.
2. **No bulk reply, ever.** Bulk approval like `send all` is not honored - the skill must walk the user through one reply at a time. The user can type `skip` or `next` to move on quickly, but cannot pre-approve a batch.
3. **Never reply to "not-interested," "unsubscribe," or hostile messages.** Mark and log. For unsubscribe, recommend the user remove the prospect from all sequences (handled by `crm-operations`).
4. **Respect the 90-day inbox retention.** If the user asks about replies older than 90 days, explain Saleshandy only retains 90 days of inbox history and the data is no longer available.
5. **Sign from the sending account.** Replies go out from the same account that sent the original sequence email. Do not change the from-address.
6. **Log every decision, even skips.** An entry in `inbox-log.md` for `skipped (not-now)` is just as important as one for `sent`. The log is the source of truth for what was decided.

## MCP tools used

All tools live in the Saleshandy MCP server.

- `get_unread_email_threads_count` - quick unread count for the diagnostic header.
- `get_email_list` - list email threads (filterable by date, account, sequence, unread status).
- `get_email_thread` - fetch a specific thread including all messages and prospect metadata.
- `get_email_content` - fetch the full body of a specific email when the thread payload is truncated.
- `reply_to_email` - send a reply. **Per-message confirmation required.**

## Classification taxonomy (8 classes)

| Class | Definition | Default suggested action |
|---|---|---|
| **interested** | Prospect shows interest, wants to learn more, asks a substantive question | Draft a reply: answer their question first, then a low-friction CTA (specific time, not "book a call"). Send only after user approves. |
| **not-now** | Timing isn't right, prospect explicitly defers ("ask later," "talk to me in Q3") | Mark as not-now. Do NOT auto-reply. Suggest the user re-engage in 60-90 days. Log with `re_engage_at` if a date is mentioned. |
| **objection** | Specific pushback (price, fit, competitor in use, no budget, "we built this in-house") | Draft a reply that acknowledges the objection, surfaces a relevant differentiator or case study, and asks one clarifying question. Send only after user approves. |
| **OOO** | Out-of-office auto-reply with a return date | Snooze the prospect until the stated return date. Do NOT reply. Log the OOO duration. |
| **auto-responder** | Generic auto-reply that is NOT OOO (autoresponder confirming receipt, "we'll get back to you," ticketing system, IT bot) | Mark as auto-responder. Do NOT reply. Log it. Flag for the user if multiple auto-responders come from the same domain (deliverability signal). |
| **unsubscribe** | Explicit opt-out: "unsubscribe," "stop emailing," "remove me," "do not contact" | Mark as unsubscribe. Do NOT reply. Recommend `crm-operations` to remove from ALL sequences. Legal requirement - handle immediately. |
| **forwarded** | Passed to a colleague: "I'm forwarding to [name]," "this should go to [name]," "[name] handles this for us" | Log the forwarded-to contact (name + email if disclosed). Suggest the user add the new contact to the sequence or start a fresh thread with them. Do NOT reply to the original sender unless they explicitly invited follow-up. |
| **referral** | Named-replacement decision-maker: "I'm not the right person, talk to [name]," "[name] would be a better fit" | Draft a short thank-you reply to the original sender (low priority - user confirms). Log the referred contact (name + email if shared). Recommend the user start a fresh thread referencing the introduction. |

If a reply is ambiguous (could be `interested` OR `objection`, etc.), pick the higher-priority class (`interested` > `objection` > `not-now` > others) and flag the ambiguity in the user-facing summary.

## Per-class drafting rules

When drafting reply text (for `interested`, `objection`, and sometimes `referral`):

1. **Read their reply first.** The draft must directly address what they said. No generic follow-ups.
2. **Match their tone.** Casual reply -> casual draft. Formal -> formal.
3. **Answer their question if they asked one.** Don't redirect to a call without answering.
4. **Keep replies short.** 50-80 words max.
5. **Links are OK in replies** (unlike cold Email 1). Case studies, calendar links, resources are appropriate.
6. **Suggest specific times** instead of "let's find a time." "How's Thursday or Friday this week?" beats "when are you free?"
7. **One CTA per reply.** Don't stack asks.
8. **Sign-off matches the sending account's voice** (pull from `sequence.md` if available, otherwise from prior thread context).

For hostile/angry replies (rare): do NOT draft. Surface verbatim with `[NEEDS HUMAN]` tag and let the user write the reply themselves.

## Process

### Step 1 - Pull the inbox

Call `get_unread_email_threads_count` first for the diagnostic header. Then call `get_email_list` with the filters from user flags (default: `--unread-only=true --since=24h`).

Output the diagnostic card:

```
Inbox triage - <campaign>
Window: <since>  |  Accounts: <all or one>  |  Sequences: <all or one>

Unread threads: <N>
  - By sequence: <breakdown>
  - By sending account: <breakdown>

Fetching threads for classification...
```

If `N = 0`, output *"Inbox is clean. No unread replies in the requested window."* and exit.

### Step 2 - Fetch and classify each thread

For each thread in the list:

1. Call `get_email_thread` to get the full conversation context.
2. If the thread body is truncated, call `get_email_content` for the latest reply.
3. Classify into one of the 8 classes using the taxonomy above.
4. Extract:
   - Prospect identifier: `<Name> <<email>>`
   - Sequence + step the original send came from
   - Reply timestamp
   - Verbatim quote of the reply (first 200 chars)
   - Forwarded-to or referred-to contact (if applicable)

### Step 3 - Build the batch summary

Output one card per reply, in priority order (`interested` first, then `objection`, then `referral`, then `forwarded`, then `not-now`, then `OOO`, then `auto-responder`, then `unsubscribe`):

```
[Reply #N]  <class>  -  <Name> (<Title>, <Company>)
Sequence: <name> (step <K>)
Replied: <timestamp>

Their reply:
> "<first 200 chars of reply, verbatim>"

Suggested action: <per-class action>

Draft reply (if applicable):

  <draft, 50-80 words>

Approve? Type:
  send #N    - confirm and send via MCP
  edit #N    - revise the draft (then re-confirm)
  skip #N    - log decision but don't send
  next       - move to next reply (skip + log)
```

### Step 4 - Walk the batch one-by-one

For each reply, wait for explicit per-message confirmation. Honor these typed commands:

- `send #N` -> invoke `reply_to_email` with the drafted body. After success, log to `inbox-log.md` with status `sent`. If MCP returns an error, surface verbatim, do NOT retry without confirmation, log the failure.
- `edit #N` -> ask: *"Paste the revised reply, or describe the change (e.g., 'shorter, drop the case study link')."* Apply the edit. Re-display the card. Loop until the user types `send #N`, `skip #N`, or `cancel #N`.
- `skip #N` -> log to `inbox-log.md` with status `skipped` and any per-class default (e.g., "skipped, marked not-now, re-engage in 90d"). Do NOT call `reply_to_email`.
- `next` -> equivalent to `skip` for the current reply, move on.
- `cancel` -> stop the triage flow. Log whatever was decided so far. Don't write a half-finished summary.

If the user tries to pre-approve a batch (`send all`, `approve everything`), refuse with: *"Bulk reply is disabled. Walk through each one - type `send #N`, `skip #N`, or `next`."*

### Step 5 - Append to `inbox-log.md`

After every decision (sent OR skipped), append a row immediately. Do not buffer to the end of the session - if the user cancels mid-flow, the partial log must survive.

If `inbox-log.md` doesn't exist yet in the campaign workspace, create it with the header below and append the first row.

### Step 6 - Output a session summary

After the batch finishes (or the user cancels):

```
Triage complete.

  Replies processed: <N>
  Sent:              <K>
  Skipped:           <S>
  Snoozed (OOO):     <O>
  Flagged unsubscribe: <U>  (recommend: run crm-operations to remove from sequences)
  Referrals/forwards: <R>   (recommend: add referred contacts to sequence)

Log written: outreach-workspace/<campaign>/inbox-log.md (+<N> rows)

Recommended next: crm-operations (to act on unsubscribes/referrals) or analytics-interpreter (to see how reply rate is trending).
```

## `inbox-log.md` format

The file is **append-only**. Each session adds rows to the table; the header is written once.

```yaml
---
file: inbox-log.md
campaign: <campaign>
note: append-only triage log; one row per reply decision
---
```

```markdown
# Inbox Triage Log - <campaign>

| Timestamp (UTC) | Prospect | Sequence (step) | Class | Suggested action | Status | Notes |
|---|---|---|---|---|---|---|
| 2026-05-19T14:23Z | Sarah Chen <sarah@finco.com> | SaaS CTO Outreach (step 2) | interested | Reply: answer no-code Q + Thu/Fri CTA | sent | - |
| 2026-05-19T14:24Z | Marcus Williams <m@finco.com> | SaaS CTO Outreach (step 2) | interested | Reply: send case study + 15-min ask | sent | edited before send |
| 2026-05-19T14:25Z | Raj Patel <raj@paytech.io> | Fintech VP Eng (step 3) | objection | Acknowledge price, surface ROI case | skipped | user wrote custom reply outside skill |
| 2026-05-19T14:26Z | Tom Anderson <tom@cloudstack.com> | SaaS CTO Outreach (step 1) | OOO | Snooze until 2026-05-26 | snoozed | OOO return date 2026-05-26 |
| 2026-05-19T14:27Z | Ana Lee <ana@healthapi.com> | Healthcare Ops (step 2) | unsubscribe | Remove from ALL sequences | flagged | run crm-operations to remove |
| 2026-05-19T14:28Z | Lisa Park <lisa@paytech.io> | Fintech VP Eng (step 2) | referral | Thank reply + log referred-to contact | sent | referred to: Diego Soto <diego@paytech.io> |
```

**Columns:**

- **Timestamp (UTC)** - ISO 8601, when the decision was made (not when the reply arrived).
- **Prospect** - `Name <email>` from the thread payload.
- **Sequence (step)** - `<sequence name> (step <K>)`. If the reply isn't tied to a known sequence, write `unknown`.
- **Class** - one of the 8 taxonomy values.
- **Suggested action** - short summary of what the skill proposed.
- **Status** - `sent`, `skipped`, `snoozed`, `flagged`, or `errored` (with reason in Notes).
- **Notes** - free-text: edits applied, referred-to contact, OOO return date, snooze date, MCP error message.

## Edge cases

- **No unread replies:** exit cleanly with *"Inbox is clean."* Do not write a no-op log entry.
- **Saleshandy plan limits the inbox:** Starter/Trial plans show only the last 15 conversations and cannot reply. Detect this from MCP error responses. Surface *"Your Saleshandy plan limits inbox access. Upgrade to view full history and reply from this skill."*
- **Reply timestamp older than 90 days:** Saleshandy retains 90 days only. If the user asks about an older reply, explain the retention limit.
- **Same prospect replied multiple times in the window:** Treat as one thread (Saleshandy threads them). Classify on the latest reply, but mention earlier touches in Notes if relevant.
- **Multiple people from the same company replied:** Flag in the session summary - high-intent signal worth multi-thread coordination. Suggest the user batch-reply to all of them with shared context.
- **MCP error during send:** Surface the error verbatim, log to `inbox-log.md` with status `errored`, do NOT retry. Ask the user how to proceed.
- **Reply from a different address than the sequence's send-to** (e.g., personal email): note in the log. The reply may not auto-link to the original sequence in Saleshandy. Tell the user.
- **Hostile / angry reply:** do NOT draft a reply. Tag `[NEEDS HUMAN]` in the card. Log with class `objection` and Note `hostile - user wrote reply outside skill`.
- **Ambiguous class (interested vs objection):** pick the higher-priority class, flag the ambiguity in the card so the user can correct before approving.
- **User runs triage twice in one day:** append to the same `inbox-log.md`. Do NOT version or overwrite. Append-only is the contract.
- **`--campaign default` and no workspace folder exists yet:** create `outreach-workspace/default/` and seed `inbox-log.md` on first append.
