# Saleshandy Settings (Behavioral Impact)

How each Saleshandy settings area shapes outreach behavior. This is a reference for what changes when a setting is toggled, who is affected, and which settings have outsized downstream impact. Use this when auditing setup, troubleshooting unexpected behavior, or planning configuration changes.


## Mental model

The Settings area is best understood as 6 systems, not a flat list:

1. **Identity & authentication** - My Profile (timezone, login, password)
2. **Sending operations** - Email Accounts, Schedules, Out of Office, Custom Tracking Domain
3. **Data model & classification** - Prospect Fields, Prospect Outcomes, Do Not Contact List
4. **Account-wide governance** - Users & Teams, Admin Settings
5. **Integrations & automation** - Integrations, Webhook, API Key, MCP
6. **Branding & commerce** - Whitelabel, Billing & Subscription


## Interpretation rules

### Rule 1: Scope matters
Every settings page has a scope:
- **User scope** - affects only the logged-in user (My Profile, OOO, Schedules)
- **Company scope** - affects the whole workspace (Admin Settings, Users & Teams, DNC, Prospect Fields)
- **Plan scope** - behavior depends on subscription tier (SSO, Whitelabel, Teams, advanced API)

### Rule 2: High-risk settings aren't always visually dramatic
A small toggle in Admin Settings can have more downstream impact than a large visible page. Treat every Admin Settings toggle as high-impact even if the UI is small.

### Rule 3: Save behavior is local
Most settings pages use **local save buttons per section or modal**, not one master save. If a user reports a change didn't stick, ask which specific block they edited and whether they saved that block.

### Rule 4: Plan gates are silent
Plan-gated features (SSO, Whitelabel, Teams, advanced API) often appear in Settings but are greyed out without a visible error. Check the plan first via Billing & Subscription.

### Rule 5: Settings changes are rarely reversible at scale
Many changes take immediate effect across all active sequences and all users. Before suggesting any change to Admin Settings, Schedules, or Custom Tracking Domain, ask whether the user has active sequences running. If they do, explain the in-flight impact.


## Admin Settings (the policy engine)

The highest-impact page in Settings. Changes here affect **every team member, every sequence, and every prospect in the workspace simultaneously.**

### Sequence behavior

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| Send first email as text only | OFF | First step of every sequence is stripped of HTML | Improves cold-outreach deliverability; removes visual formatting for all campaigns |
| Sending priority | New prospects first | Controls whether sending capacity goes to new prospects or follow-ups first | Affects which emails go out when daily limits are reached |

### Reply & tracking behavior

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| **Consider prospect as finished if reply received** | **ON** | When a prospect replies, they're marked Finished and removed from the sequence | **The most impactful toggle in the product.** If OFF, follow-ups continue even after reply. Change only if the use case explicitly requires continuing outreach after reply. |
| AI Categorization in Unified Inbox | ON | Auto-labels replies (Interested, Not Interested, Meeting Booked, OOO, etc.) | Speeds inbox triage; labels can be wrong - human review still recommended |
| Track opens for 1:1 emails | Configurable | Counts open events for direct (non-sequence) emails | Inflates open metrics if left on without intent |
| Track clicks for 1:1 emails | Configurable | Counts click events for direct emails | Same as above |
| Track external replies | ON | Captures replies from a different address (e.g., assistant replying for the prospect) | Widens reply capture; increases inbox volume |
| Track same-domain replies | Configurable | Captures internal/same-domain replies (e.g., colleagues forwarding) | Turn OFF for enterprise accounts where internal chatter shouldn't count as prospect replies |
| Ignore emails from specific domains/emails | - | Suppresses tracking events from listed addresses | Useful for filtering out internal QA/test addresses |

### Prospect governance

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| **Allow adding one prospect in multiple sequences** | **OFF** | When OFF, same email cannot be in two active sequences simultaneously | **Second most impactful toggle.** Turn ON only if the use case explicitly requires parallel outreach. Risk: same prospect gets simultaneous emails from different sequences. |
| Restrict based on active sequences only | Configurable | When multi-sequence is blocked, whether restriction applies only to ACTIVE or also PAUSED/DRAFT | Affects whether paused-sequence prospects can be re-enrolled |
| Verify email during CSV import | Configurable | Runs email verification as part of import | Increases import time; reduces bounce risk |
| Smart re-verification | Configurable | Re-verifies emails previously flagged risky after a set interval | Reduces stale verification; consumes verification credits |
| Skip interval (days) | - | How many days must pass before re-verification is attempted | Governs verification credit consumption |

### Lead Finder behavior

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| Auto-save revealed leads as prospects | Configurable | Revealed Lead Finder contacts auto-added to CRM | Can generate many CRM records without explicit import |
| Default reveal type | Work email | Whether Lead Finder defaults to revealing work or personal email | Affects deliverability and GDPR compliance depending on use case |

### Team visibility

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| Team members can see each other's prospects | Configurable | When ON, all team members view all prospects in the workspace | Affects privacy, data siloing, collaboration breadth |

### Security

| Toggle | Default | What it does | Downstream effect |
|---|---|---|---|
| SSO (Single Sign-On) | Configurable | Enforces SSO for workspace login | **Once enforced, users without SSO credentials immediately lose access.** Confirm SSO is fully configured before enabling. |
| OTP requirement | Configurable | Adds one-time password layer to login | Hardens authentication; adds friction |


## Sending operations

### Email Accounts (sender infrastructure)

Routes to the same Email Accounts module available from main navigation - it's a shared module, not a settings-only page. Controls:
- Sender account availability
- Warmup status
- Setup completeness
- Inbox placement visibility
- Sender allocation across sequences

See `references/email-account-health.md` for the detailed account-health model, setup score thresholds, and warmup behavior.

### Schedules

Defines **when the sending engine is allowed to operate.** Reusable timing templates inherited by sequences.

- **Add new schedule** - creates a new timing template
- **Make It Default** - changes the default schedule for new sequences (existing sequences keep their assigned schedule)

Cross-reference: Schedule changes do NOT retroactively affect existing sequences. Existing sequences keep their currently assigned schedule until manually updated.

### Out of Office (OOO)

Humane automation safeguard preventing poorly timed follow-ups.

When enabled, OOO replies cause:
- The prospect is marked Out of Office
- The prospect is paused **in all active sequences** (not just the one that received the reply)
- If a return date is detected: prospect auto-resumes on that date
- If no date: prospect resumes after the configured fallback duration

Configuration:
- Master enable switch
- Pause by detected return date (auto-resume)
- Fallback pause days (used when no date detected)
- Fixed pause-days mode
- Allowed pause range: **1-100 days**

This is cross-team: if one team member's sequence gets the OOO reply, the prospect is paused for all other team members' sequences too.

### Custom Tracking Domain (CTD)

Replaces default Saleshandy tracking links with a sender-controlled subdomain. Affects:
- Branded tracking identity
- Click-tracking trust and consistency
- Deliverability perception
- Linkage between sender identity and tracking identity

**Best practice:** Use the same domain family as the sending domain. Using a different tracking domain than the sending domain raises ESP suspicion.

**Setup steps:**
1. Add Custom Domain modal
2. Enter subdomain (e.g., `track.yourdomain.com`)
3. Add CNAME record at DNS provider: `track.yourdomain.com -> watch.saleshandy.com`
4. **Cloudflare users:** set the record to DNS-only (gray cloud icon) - proxied mode breaks tracking
5. Click Verify & Save in Saleshandy
6. DNS propagation: up to 72 hours

Removing a CTD breaks click and open tracking in all active sequences until a new one is configured.


## Data model & classification

### Prospect Fields

The **data contract** for personalization, imports, filtering, and CRM context.

Two tabs:
- **Custom Fields** - account-specific attributes for segmentation, personalization, enrichment storage, AI-generated context
- **System Fields** - built-in schema used by the rest of the product

Custom field types: Text, Number, Long Text, Date, Dropdown, Currency. Each can have a fallback text value used when the field is empty.

**Deleting a custom field permanently removes the field and all data stored in it from every prospect record.** Irreversible.

### Prospect Outcomes

The **reply interpretation system.** Defines how replies are labeled and how sentiment is mapped.

Two tabs:
- **Custom Outcome** - user-defined outcomes (up to 100)
- **System Outcome** - built-in outcomes

Custom outcome constraints:
- Up to 100 custom outcomes
- Each carries a sentiment tag
- Names: 1-20 characters
- Outcomes have **account-wide impact**

### Do Not Contact List (DNC)

The **suppression and compliance layer.** Matching domains/emails are blocked from future outreach.

Scope options:
- **Global** (All Clients) - blocks across the entire account
- **Specific Client** - client-scoped suppression

Adding to DNC is safe (protective). Removing requires more care.


## Account-wide governance

### Users & Teams

The people and permission layer.

**Users tab** - who has access, role assignment, login/logout activity, IP/location visibility, invite capability.

**Teams tab** - collaboration structure beyond individual users. Plan-gated; requires Pro or above.

**Adding a team member workflow:**
1. Settings -> Users & Teams -> Invite User
2. Enter email; assign role: Member (standard) or Admin (elevated)
3. User receives invite email and sets their own password
4. After joining: they connect their own email account via Settings -> Email Accounts
5. They inherit account-level Admin Settings but manage their own profile, schedule, OOO

**Caveats:**
- Role at invite determines workspace-wide permissions
- Team Management requires Pro plan; Starter accounts can only use the owner
- Admin role grants access to Admin Settings, Users & Teams, Billing. Explain this before promoting.


## Integrations & automation

### Integrations
- CRM (Salesforce, HubSpot, Zoho, Pipedrive)
- Zapier / automation
- LinkedIn automation (Aimfox, HeyReach)
- Video personalization tools (Sendspark, Pitchlane, Weezly)

Each is a connection/auth flow. Most are additive (safe to connect).

### Webhook (real-time outbound event bridge)

Webhooks push activity events to external endpoints. Configuration:
- Webhook Name
- Webhook URL
- HTTP Headers (optional)
- Trigger event checkboxes
- Test before save

Supported events: email sent, email opened, link clicked, reply received, bounced, unsubscribed, prospect finished, prospect outcome changed, sequence paused, email account disconnected, inbox radar test completed, task-related events.

**Deleting a webhook** stops all automations using it immediately. No recovery.

### API Key (machine access)

API keys allow external apps to authenticate against the platform API.

**Deleting an API key** immediately revokes access for any integration using that key. Cannot be recovered - the integration must be re-keyed.

API docs are at the Develop with Saleshandy Swagger interface (real endpoint groups: analytics, clients, do-not-contact, email-accounts, enrichment, fields, etc.).

### MCP (AI operating interface)

MCP is the connectivity layer for AI assistants. Without MCP, humans use the UI manually. With MCP, AI assistants interact with the workspace through structured configuration.

Setup choices: ChatGPT Pro/Team/Enterprise, Claude, Claude Code, VS Code, Cursor IDE, Other Tool.

For "Other Tool": generates a ready-to-use MCP URL, copy/config options, HTTP and stdio examples. The generated key shows up in API Keys as `MCP - Custom Integration`.

Disconnect is available for connected assistants like Claude and ChatGPT.


## Branding & commerce

### Whitelabel

- **Logo & Branding** - company name, login logo, platform icon, favicon
- **Domain Configuration** - branded domain replacing default Saleshandy URLs (DNS-backed)

Plan-gated. Affects visual identity, client-facing professionalism, branded product access.

### Billing & Subscription

Commercial entitlement system. Controls:
- Feature availability
- Usage limits (prospect, sending, AI credits, verification credits)
- Upgrade gates (SSO, Whitelabel, advanced collaboration)
- Billing operations

Tiers observed: Starter, Pro, Scale, Scale Plus.


## High-impact settings ranked

These have the biggest downstream effect on product behavior:

1. **Admin Settings** - policy, tracking, duplicates, security, visibility
2. **Email Accounts** - sender health and operational readiness
3. **Custom Tracking Domain** - tracking trust, DNS alignment, deliverability
4. **Out of Office** - pause/resume automation quality
5. **Schedules** - send timing governance
6. **Prospect Fields** - data schema and personalization inputs
7. **Prospect Outcomes** - reply interpretation and reporting logic
8. **Do Not Contact List** - compliance and suppression safety
9. **API Key / MCP / Webhook / Integrations** - automation and external control
10. **Billing & Subscription** - available features and ceilings


## Guardrails

### Hard stops (explicit confirmation required)

| Action | Where | Risk | Required check |
|---|---|---|---|
| Change "Consider prospect as finished if reply received" to OFF | Admin Settings | Follow-ups resume to prospects who have already replied across all active sequences | "This will resume follow-ups to prospects who have already replied, in every active sequence. Confirm?" |
| Enable "Allow adding one prospect in multiple sequences" | Admin Settings | Same prospect can receive simultaneous emails from multiple campaigns | "Prospects may receive emails from multiple sequences at the same time. Is that intentional?" |
| Delete Account | My Profile | Permanent account deletion - all data, sequences, prospects, templates lost | "This permanently deletes the entire account and all data. Cannot be undone. Are you certain?" |
| Enforce SSO | Admin Settings -> Security | Users without SSO credentials immediately lose login access | "Once enabled, all users must log in via SSO. Anyone without SSO credentials will be locked out immediately." |
| Delete custom tracking domain | Custom Tracking Domain | All tracking links in active sequences break until a new domain is set up | "Removing this domain will break click and open tracking in all active sequences until a new tracking domain is configured." |

### Irreversible actions

| Action | Consequence |
|---|---|
| Delete Account | All data permanently destroyed |
| Delete API Key | Any integration using that key immediately loses access; cannot be recovered |
| Delete Webhook | All automations using that webhook stop immediately |
| Remove email account (not disconnect) | Prospects in active sequences stop receiving emails; re-adding the same account does not restore the link |
| Delete prospect field | Field and all data stored in it are permanently removed from all prospect records |

### Safe actions

| Action | Why safe |
|---|---|
| Changing name / timezone / phone in My Profile | User-scoped, no operational impact |
| Adding a new Schedule | Additive; existing sequences not affected until applied |
| Editing OOO fallback duration | Affects future OOO detections only, not currently paused prospects |
| Creating a new API Key | Additive; existing keys remain active |
| Adding webhook | Additive; only new events trigger it |
| Adding prospect outcomes / custom fields | Additive; nothing changed, only new options available |
| Adding to DNC list | Protective; prevents unwanted contact, no harm to existing data |
| Toggling email account connection ON (reconnect) | Restores sending without affecting sequences |


## Behavior rules

1. **Admin Settings is not personal preferences.** If a user says "I want to change a setting," confirm whether they mean personal (My Profile, OOO, Schedules) or account-wide (Admin Settings). Different scopes.
2. **The reply-finished toggle is the most important toggle in the product.** "Consider prospect as finished if reply received" is ON by default. If a user reports emails still going out after a reply, check this toggle first.
3. **Settings save is local.** Most pages save per-section. If a user says changes didn't stick, ask which section and whether they saved that specific block.
4. **Plan gates are silent.** If a user can't find or use a feature, check their plan first via Billing & Subscription.
5. **Settings changes take effect immediately at scale.** Before any change to Admin Settings, Schedules, or Custom Tracking Domain, ask whether they have active sequences running. If yes, explain the in-flight impact.


## One sentence

Settings is the control center that determines how the account behaves before, during, and around outreach execution.
