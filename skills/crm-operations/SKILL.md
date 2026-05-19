---
name: crm-operations
description: Use when the user wants to manage my prospects, bulk update prospects, move prospects to a new sequence, change prospect status, update CRM, delete prospects, snooze tasks, skip tasks, or manage tasks. Triggers on "manage my prospects," "bulk update prospects," "move prospects to new sequence," "change prospect status," "update CRM," "delete prospects," "snooze tasks," "skip tasks," "manage tasks." Reads and writes the Saleshandy CRM via MCP with strict confirmation gates: any operation affecting >10 records requires explicit "yes, proceed"; delete and status-change operations on sequences require typed sequence-name confirmation; destructive ops are never chained. All destructive actions append-logged to crm-actions.md.
---

# CRM Operations

This is the highest-risk skill in the plugin. It can write to live CRM data, bulk-enroll prospects in active sequences, mark prospects as Do Not Contact, delete records, and change sequence states. Every destructive action passes through a confirmation gate calibrated to its blast radius, and every destructive action is append-logged to `crm-actions.md`.

The Saleshandy CRM is a prospect database + outreach tracking layer (not a deal pipeline). It is the single source of truth for who can be contacted, who has been contacted, and what state each prospect is in across all sequences. Editing it incorrectly at scale is the worst class of mistake this plugin can make.

## Inputs

- **Required:** live Saleshandy data via MCP. No workspace prerequisite for read ops.
- **Optional flags:**
  - `--sequence <id or name>` - scope reads or writes to a sequence.
  - `--account <id or email>` - scope to a sending account.
  - `--filter <field=value>` - filter prospects before any action.
  - `--limit <N>` - cap the number of records the operation will touch.

## Outputs

- **Read ops** (list, count, get-by-id): inline conversational answer. No file write.
- **Write ops** (any destructive call): append a row to `outreach-workspace/<campaign>/crm-actions.md` with timestamp, MCP tool, scope, and the user's confirmation evidence. This file is **append-only** - never overwritten, never edited.

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## References

- `references/execution-protocol.md` - **source of truth** for the Full Protocol mode and confirmation-gate pattern. CRM operations that touch >10 records or change sequence state run in Full Protocol mode (Understand -> Plan -> Confirm -> Execute -> Verify). Confirmation gates in this skill are calibrated against the gate matrix in that doc.

## Hard rules

1. **Any operation affecting more than 10 prospects, tasks, or sequence enrollments requires explicit "yes, proceed" confirmation** before the MCP call goes out. The confirmation card MUST show: count, a sample of 3-5 records, the exact MCP tool that will be called, and the scope (filter applied). A vague "yes" or "sure" does not pass the gate.
2. **Any DELETE or STATUS CHANGE on a sequence requires the user to type the sequence name verbatim as confirmation.** "Yes, delete sequence Q2 Outbound" is not enough. The user must type the sequence name exactly as it appears in Saleshandy. Case-insensitive match. This applies to: `delete_sequence`, `update_sequence_status` (when status is paused / stopped / archived), `remove_email_accounts_from_sequence` when it leaves the sequence with zero senders.
3. **Never chain destructive operations.** Each destructive call requires its own confirmation. If the user requests "delete sequence X and remove prospects Y and Z," queue them as three separate gates. Do NOT batch them into one "yes." After each gate fires and the call succeeds, confirm completion and prompt for the next one.
4. **`crm-actions.md` is append-only.** Every destructive call - successful or failed - appends a row. Never overwrite, never edit, never delete entries. The log is the audit trail. If the user runs this skill twice in a session, the second run appends to the same file.
5. **DNC is compliance, not a tag.** Marking a prospect as Do Not Contact (via `add_dnc_items` or assigning a DNC outcome) is a CAN-SPAM / GDPR compliance act. It cannot be undone the same way a tag is removed. ALWAYS confirm by name + reason before assigning DNC, even on a single prospect.
6. **Outcome is not a sequence stop.** Assigning "Interested" or "Meeting Booked" does NOT remove the prospect from active sequences. If the user's intent is to stop emails, explicitly state this and offer the separate operation (`update_sequence_status` for the prospect's enrollment, OR confirm the account-level setting "Consider prospect as finished if reply received" is ON).
7. **Read before writing.** Before any write op, fetch the current state of the record(s) you're about to change. Show the user current vs. proposed. Never edit blindly.
8. **Surface MCP errors verbatim.** Do NOT retry, do NOT silently skip, do NOT estimate around a failure. If a destructive call returns an error, log the failure to `crm-actions.md` with status `errored` and surface the error to the user. They decide whether to retry.

## Operation classes

Every CRM call falls into one of four classes. The class determines the gate.

### Class 1: Read (low-risk, no confirmation)

Lookups, counts, and inspections. No data change. No gate.

| MCP tool | Use |
|---|---|
| `list_clients` | enumerate clients (agency / multi-tenant context) |
| `list_fields` | enumerate prospect fields available for filter/edit |
| `list_tasks` | list tasks (by sequence, by assignee, by status) |
| `get_task_by_id` | fetch one task |
| `get_task_counts` | aggregate counts by status |
| `get_task_assignee_list` | enumerate task assignees |

These can run freely. State the scope in the answer so the user can interpret correctly.

### Class 2: Single-record write (medium-risk, summary confirmation)

Modifies exactly one record. Confirmation required but no typed scope.

| MCP tool | Confirmation format |
|---|---|
| `skip_task` | "Skip task #N (<task name>) for <prospect>? Yes/no." |
| `snooze_task` | "Snooze task #N until <date>? Yes/no." |
| `complete_task` | "Mark task #N complete? Yes/no." |
| `update_task_note` | Show old note + new note. "Apply? Yes/no." |

Gate fires on the first ambiguous response. A clear "yes" passes. After the call, append to `crm-actions.md`.

### Class 3: Bulk write (HIGH-RISK, explicit count confirmation)

Modifies more than 10 records in one call. Confirmation requires "yes, proceed" verbatim AND the count + sample card.

| MCP tool | What to show |
|---|---|
| `bulk_skip_tasks` | count + sample of 3-5 tasks + reason for skipping |
| `bulk_snooze_tasks` | count + sample + snooze duration |
| `add_leads_to_sequence` (bulk) | count + sample of leads + sequence name + sequence status (Active/Draft/Paused) + entry step |
| `add_dnc_items` (bulk) | count + sample + reason + reminder that this is a compliance act |

Confirmation card format:

```
Bulk operation: <name>
MCP tool: <exact tool name>
Scope: <filter applied or "manually selected"> -> <N> records
Sample (first 3-5):
  - <record 1>
  - <record 2>
  - <record 3>
  ...
Side effects:
  <e.g., "Sequence is Active - emails will start sending on next scheduled slot.">
  <e.g., "DNC affects all current and future sequences for these prospects.">

Type "yes, proceed" to execute, or describe the change you want.
```

A response that is not literally "yes, proceed" (or "yes proceed" / "proceed yes") does NOT pass. "Sure" or "ok" or "do it" do not pass. State this expectation in the card.

### Class 4: Destructive (HIGHEST-RISK, typed sequence-name confirmation)

Operations that delete data, change sequence state in production, or remove senders from a live sequence.

| MCP tool | Typed confirmation required |
|---|---|
| `delete_sequence` | User types sequence name verbatim |
| `update_sequence_status` (paused / stopped / archived) | User types sequence name verbatim |
| `remove_email_accounts_from_sequence` (when removal leaves zero senders) | User types sequence name verbatim |
| `update_sequence_priority_distribution` (any in-flight live sequence) | User types sequence name verbatim |
| `update_sequence_settings` (when changing send schedule on live sequence) | User types sequence name verbatim |

Confirmation card format:

```
DESTRUCTIVE: <operation name>
MCP tool: <exact tool>
Target: <sequence name>
Current state: <e.g., "Active, 1,250 prospects enrolled, last send 4 hours ago">
After this call: <e.g., "Sequence deleted permanently. All enrollment history preserved in prospect records, but the sequence and its in-flight tasks will be removed.">
Impact: <e.g., "1,250 prospects will stop receiving emails from this sequence. They remain in CRM with their history intact.">

To confirm, type the sequence name verbatim:
> <sequence name>
```

The user types the name. The skill compares (case-insensitive, trim whitespace). If it matches, the gate passes. If not, prompt once: *"That doesn't match. The sequence name is '<name>'. Type it exactly to confirm, or type 'cancel' to abort."* If the second attempt also fails or the user types anything else, cancel and log to `crm-actions.md` with status `aborted`.

### Operations explicitly NOT in this skill

Some operations live in other skills or are deliberately omitted from CRM ops:

- **Sending emails (`reply_to_email`):** handled by `inbox-triage` with per-message confirmation.
- **Updating sequence steps / variants / step content:** out of scope here. Use Saleshandy UI for content authoring; `email-auditor` for critique.
- **Creating new sequences / steps (`create_sequence`, `add_sequence_step`):** out of scope here. Use Saleshandy UI; this skill operates on existing sequences.
- **Schedule / DNC list creation (`create_schedule`, `create_dnc_list`):** out of scope. Manage in Saleshandy UI; bulk additions to existing DNC lists are in scope via `add_dnc_items`.

If a user asks for one of these, route them: *"That's out of scope for crm-operations. Create it in Saleshandy, then come back to enroll / DNC / manage it here."*

## MCP tools used

### Read (Class 1)
- `list_clients`
- `list_fields`
- `list_tasks`
- `get_task_by_id`
- `get_task_counts`
- `get_task_assignee_list`

### Single-record write (Class 2)
- `skip_task`
- `snooze_task`
- `complete_task`
- `update_task_note`

### Bulk write (Class 3)
- `bulk_skip_tasks`
- `bulk_snooze_tasks`
- `add_leads_to_sequence`
- `add_dnc_items`

### Destructive (Class 4)
- `delete_sequence`
- `update_sequence_status`
- `update_sequence_priority_distribution`
- `update_sequence_settings`
- `remove_email_accounts_from_sequence`

## Process

### Step 1 - Classify the request

Identify which operation class fires. If ambiguous (e.g., "delete prospects who haven't opened in 90 days" could be DNC, hard delete, or sequence removal), ask one scoping question: *"To confirm: do you want to (A) hard-delete these prospects from CRM, (B) mark them DNC and stop all future outreach, or (C) just remove them from this sequence?"*

### Step 2 - Pull current state (read before write)

For any write op:

1. Run the matching read MCP call to fetch the affected records.
2. Show count, sample, current state.
3. If the count exceeds the limit the user implied (e.g., they said "a few" but the filter matches 800), pause and re-confirm scope before proceeding.

### Step 3 - Apply the gate

Based on the operation class, present the matching confirmation card. Wait for the user's response. Do NOT pre-emptively call the MCP tool.

### Step 4 - Execute

After the gate passes, call the MCP tool. Surface the response. If it returns an error, log to `crm-actions.md` with status `errored` and the error text; surface to the user; do not retry without explicit instruction.

### Step 5 - Append to `crm-actions.md`

Append one row immediately after the MCP call returns (success or failure). Do NOT buffer until end-of-session. If the user cancels mid-flow, the rows already written must persist.

### Step 6 - Verify and chain

After a destructive call succeeds:

1. State what happened factually (e.g., *"Deleted sequence 'Q2 Outbound'. 1,250 enrollments archived to prospect records."*).
2. If the user has a queued follow-up destructive op (e.g., "now DNC those 1,250 prospects"), treat it as a NEW request and run Step 1-5 again. No batching.
3. Recommend a chained skill if relevant (e.g., after bulk DNC: *"Run `analytics-interpreter` to see how this changes your active prospect base."*).

## `crm-actions.md` format

The file is **append-only**. Each destructive call writes one row. The header is written once on first append.

```yaml
---
file: crm-actions.md
campaign: <campaign>
note: append-only audit log of every destructive CRM operation. Never overwrite.
---
```

```markdown
# CRM Actions Log - <campaign>

| Timestamp (UTC) | Operation class | MCP tool | Scope | Count | Confirmation evidence | Status | Notes |
|---|---|---|---|---|---|---|---|
| 2026-05-19T14:23Z | bulk-write | bulk_snooze_tasks | sequence="Q1 Outbound" assignee=jyot | 47 | "yes, proceed" | success | snoozed 7 days |
| 2026-05-19T14:28Z | destructive | update_sequence_status | sequence="Q2 Outbound" -> paused | 1,250 enrollments | typed "Q2 Outbound" | success | paused per user request after low reply rate |
| 2026-05-19T14:32Z | bulk-write | add_dnc_items | filter: replied="not interested" last 60d | 18 | "yes, proceed" | success | DNC compliance batch |
| 2026-05-19T15:01Z | destructive | delete_sequence | sequence="Old Test Sequence" | 0 enrollments | typed "Old Test Sequence" | success | empty test sequence cleanup |
| 2026-05-19T15:14Z | destructive | delete_sequence | sequence="Fintech Outreach Q1" | 847 enrollments | typed "fintech outreach Q2" (mismatch) | aborted | user typed wrong name; no call made |
| 2026-05-19T15:22Z | bulk-write | add_leads_to_sequence | sequence="Warm Follow-Up" entry=Step 1 | 83 | "yes, proceed" | success | warm prospects from Opens >=3 + Replies=0 filter |
| 2026-05-19T15:30Z | single-write | skip_task | task #4421 | 1 | "yes" | errored | MCP returned PLAN_LIMIT; no retry |
```

**Columns:**
- **Timestamp (UTC)** - ISO 8601, when the call was made.
- **Operation class** - `single-write`, `bulk-write`, or `destructive`. (Class 1 read ops are NOT logged here.)
- **MCP tool** - exact MCP tool name.
- **Scope** - filter / target / sequence name.
- **Count** - records affected.
- **Confirmation evidence** - exactly what the user typed to pass the gate.
- **Status** - `success`, `errored`, or `aborted`.
- **Notes** - reason, error text, or context.

## Common task flows

### Flow A - Look up a prospect (Class 1, no gate)

User: *"What's the status of vatsal patel?"*

1. `list_tasks` with filter `prospect_email=<email>` (or use `--filter`).
2. `get_task_by_id` for any tasks in flight.
3. Present a Prospect Card: name, email, status, sequences enrolled, last contacted, opens/clicks/replies.
4. Offer next actions: *"Add to a different sequence? Assign an outcome? Add a manual task?"*

No log entry. Read-only.

### Flow B - Snooze one task (Class 2, single confirmation)

User: *"Snooze task #4421 for a week."*

1. `get_task_by_id` to confirm the task exists and fetch its details.
2. Show: task name, current due date, proposed snooze date, prospect, sequence.
3. *"Snooze task #4421 ('Call John Doe') from 2026-05-19 to 2026-05-26? Yes/no."*
4. On "yes": call `snooze_task`. On success: log to `crm-actions.md`. State completion.

### Flow C - Bulk enroll 83 prospects (Class 3, "yes, proceed" gate)

User: *"Enroll those 83 warm prospects in my Warm Follow-Up sequence."*

1. Read current state via filter or stored selection. Confirm count is 83.
2. Read sequence status via `list_sequences` filtered to the named sequence. Confirm it is Active (or Draft / Paused) and which entry step.
3. Show the bulk confirmation card with count, sample of 3-5 prospects, sequence name, status, entry step, side effect ("emails will start sending on next scheduled slot if Active").
4. Wait for "yes, proceed" verbatim.
5. On confirmation: call `add_leads_to_sequence`. Surface result. Append to `crm-actions.md`.

### Flow D - Delete a sequence (Class 4, typed name)

User: *"Delete the Q2 Outbound sequence."*

1. Read sequence state via `list_sequences` -> get name, status, enrollment count, last activity.
2. Show the destructive card with current state and impact.
3. Prompt: *"Type the sequence name verbatim to confirm: > Q2 Outbound"*
4. User types. Compare case-insensitive, trim whitespace.
5. If match: call `delete_sequence`. Log success.
6. If mismatch: prompt once more. If still wrong or cancel: log `aborted`, do nothing.

### Flow E - "Delete all prospects who haven't opened in 90 days" (ambiguous, then Class 4)

User: *"Delete all prospects who haven't opened in 90 days."*

1. **Disambiguate first.** *"To confirm: do you want to (A) hard-delete these prospects from CRM, (B) mark them DNC and stop all future outreach, or (C) remove them from current sequences only?"*
2. Read prospect count via filter. Suppose: 1,847 prospects match.
3. Apply the matching gate:
   - For (A) hard-delete: the operation is destructive at scale. Show count + sample of 5 + impact ("permanently removes records from CRM, including all engagement history"). Require typed confirmation matching the filter description (e.g., "yes, delete 1847 prospects with no opens in 90 days").
   - For (B) DNC: bulk-write class with "yes, proceed" gate. Show count + sample + compliance reminder.
   - For (C) remove from sequences: typed sequence-name confirmation per affected sequence (could be multiple).
4. **Never chain.** If the user picks (A) AND (B), run two separate gates back-to-back.

### Flow F - "Pause the underperforming sequence" (Class 4)

1. Identify the sequence. If ambiguous, ask which one (don't guess).
2. Read sequence state.
3. Show destructive card: sequence will go to `paused`, in-flight emails stop on next scheduled slot, prospects stay enrolled.
4. Prompt for typed sequence name.
5. On match: `update_sequence_status` with `status=paused`. Log.

## Edge cases

- **User says "yes" not "yes, proceed" for a bulk op:** prompt once: *"For bulk operations I need 'yes, proceed' verbatim. Type it to confirm, or describe the change."* On second non-match: cancel and log nothing (no MCP call was made, so nothing to log; do not write an aborted row for a never-attempted op).
- **User types the wrong sequence name twice on a destructive op:** cancel, log `aborted` with the typed strings, do not retry.
- **MCP tool returns plan-limit error:** log `errored` with the limit message, surface to user, recommend the upgrade path. Do NOT estimate around it.
- **User asks to delete a sequence that has in-flight tasks scheduled in the next hour:** flag this prominently in the destructive card. Suggest pausing first, waiting for tasks to clear, then deleting.
- **User asks to bulk-DNC prospects who replied "interested":** challenge the intent. *"DNC on prospects who replied positively looks like an error. Do you mean to mark them 'Not Now' or remove from this sequence specifically?"* Require explicit re-confirmation of intent before proceeding.
- **Same prospect in multiple sequences and user wants to remove them from one:** confirm scope - one sequence or all? If they say "all," that may require sequential calls per sequence (no batch tool for cross-sequence removal). State this and confirm each.
- **User runs crm-operations after inbox-triage flagged unsubscribes:** the recommended chain is to bulk-DNC the flagged prospects. Pull the list from `inbox-log.md` if it exists, present the count and prospects, run the bulk-DNC gate.
- **User wants to update_sequence_settings on a Draft sequence:** Class 4 gate still applies for any setting change that affects send behavior. Draft sequences not yet activated are lower-risk, but the gate stays - state in the card that the sequence is Draft and emails are not yet sending.
- **`add_leads_to_sequence` adds duplicates that are already in the sequence:** Saleshandy deduplicates by email. State this and report the effective count after dedup once the MCP call returns.
- **User cancels mid-flow ("stop" / "cancel" / "abort"):** halt immediately. Log nothing for ops that didn't fire. For ops that already ran, the log row is already written; surface the partial state.
- **`update_sequence_priority_distribution` on a sequence with no active senders:** the change is moot. State this and ask if the user wants to add senders first via Saleshandy UI before adjusting distribution.
- **User asks "what destructive ops have I run today?":** read `crm-actions.md`, filter to today's UTC date, summarize. This is a read op on a workspace file, not on Saleshandy data; no MCP call needed.
- **`crm-actions.md` doesn't exist yet:** create it on first append. Seed with the header block.
- **`crm-actions.md` has been manually edited:** detect by checking for any row with a missing column. Surface a warning *"The audit log appears to have been edited outside this skill. Continuing to append, but be aware history may be incomplete."* Still append the new row. Never refuse to write because of a malformed prior row.
