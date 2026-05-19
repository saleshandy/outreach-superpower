---
name: email-sequence-generator
description: Use when the user wants to write cold email sequence copy, draft my sequence, create outreach sequence, build [N]-email sequence, write LinkedIn + email cadence, multichannel sequence, email cadence with merge tags, or generate follow-up emails. Triggers on "write cold email sequence," "draft my sequence," "create outreach sequence," "build 4-email sequence," "write LinkedIn + email cadence," "multichannel sequence," "email cadence with merge tags," "draft a cadence," "follow-up sequence." Reads `icp.md`, `company.md`, optionally `leads.md` and `enrichment-plan.md`. Runs an 8-phase architecture (discovery, ICP, goal, channel mix, psychology copy, AI personalization, settings, handoff) and writes paste-ready `sequence.md`.
---

# Email Sequence Generator

Build a psychology-driven, multichannel outreach sequence (email + LinkedIn + call + WhatsApp + custom tasks) and write it to `sequence.md` for paste into Saleshandy.

## Inputs

- **Required:** `outreach-workspace/<campaign>/icp.md`
- **Required:** `outreach-workspace/<campaign>/company.md`
- **Optional:** `outreach-workspace/<campaign>/leads.md` (real prospect signals for personalization)
- **Optional:** `outreach-workspace/<campaign>/enrichment-plan.md` (AI column names to reference as merge tags)
- **Flags:**
  - `--steps N` (default `4`)
  - `--channel email|multi` (default `email`)
  - `--campaign <name>` (default `default`)

## Outputs

- Writes: `outreach-workspace/<campaign>/sequence.md`

## References

- `references/inbox-radar.md` - deliverability red flags to avoid in copy (spam words, content patterns that hurt placement).
- `references/email-account-health.md` - sender capacity, account-warmup gating, recommended daily volume per sender.
- `references/execution-protocol.md` - Understand -> Plan -> Confirm -> Execute -> Verify loop.
- `references/industry-tone.md` - industry-specific lead-with / proof-style / CTA conventions (SaaS, Lead Gen, Digital Marketing, Recruitment, PR, E-commerce). Consulted in Phase 5.

## Hard rules

1. **Never write copy without business + ICP context.** If `company.md` or `icp.md` is missing, stop and route to `strategy-architect` / `icp-builder` first.
2. **Word limits are non-negotiable.** Opening email 60-100 words. Follow-ups 50-70. Breakup 40-60. Show word count on every email.
3. **No links, calendar URLs, or attachments in Email 1.** Links appear from Email 2+ only.
4. **One CTA per email, always a question.** Must pass the effort test: would saying "no" take more effort than saying "yes"?
5. **Every email gets spintax + merge-tag fallbacks.** Minimum 1 spin block; recommended 3 (subject + opener + CTA).
6. **No spam triggers.** Run every line against the spam list before output (see Phase 5 below).
7. **Respect user tone + emoji preference absolutely.** Ask once at Phase 5; carry through entire sequence.

## Process

### Phase 1 - Business discovery

Read `company.md`. Surface a one-screen Business Context Card: company, industry, product, target market, key value props, differentiators, proof points (with metrics). Ask: *"Does this look right? Anything to add or change?"* Wait for confirmation.

If `company.md` is missing: stop. *"No `company.md` in workspace. Run `strategy-architect` first to capture business context, then return."*

### Phase 2 - ICP confirmation

Read `icp.md`. Parse:

- Frontmatter: `version`, `primary_segment`, `buying_motion`, `segments`.
- Body: segment names from `# Segment: <name>` or `## Segment: <name>` headings. Prefer H2 first, then H1.
- Angle Kit (5 angles) under `# Cold Email Angle Kit`.

If multiple segments exist, ask:

> *Pick a segment:*
> 1. <Segment 1> [primary]
> 2. <Segment 2>
> 3. <Segment 3>
>
> Reply with the number, or hit Enter for primary.

If `icp.md` is missing: stop. *"No `icp.md` in workspace. Run `icp-builder` to define your target segments, then return. I'll wait."* This is an explicit handoff via the `using-outreach-superpower` router.

**Angle Kit fallback:** if `icp.md` exists but no Angle Kit, generate 5 angles inline (format: `Angle name | Subjects (3) | Openers (2) | CTAs (2)`) before drafting emails. Display in chat first. Note in `sequence.md`: *"Angles generated inline (no Angle Kit in icp.md). Run `icp-builder` to persist."*

### Phase 3 - Goal

> *What's the campaign goal?*
>
> 1. Book a demo / sales call (most common, B2B SaaS)
> 2. Start a conversation / get a reply (enterprise, long-cycle)
> 3. Lead generation (productized offers, ROI hooks)
> 4. Re-engage cold leads (prospects went silent)
> 5. Backlink / partnership outreach (SEO, content)
> 6. PR / media outreach (journalists, influencers)
> 7. Recruitment / talent outreach
> 8. Custom (describe it)
>
> Reply 1-8 or describe a custom goal.

Combine business + ICP + goal into a Use Case Statement and confirm:

```
Business: <company> - <one-line product>
Target: <segment> at <firmographics>
Pain: <primary pain from icp.md>
Goal: <selected goal>
Angle: <best matching Angle Kit angle>
Proof: <strongest proof point from company.md>
```

### Phase 4 - Multichannel architecture

Ask channel mix only if `--channel multi` or user did not specify. Default to email-only otherwise. Recommend a template:

**Template A - Email + LinkedIn (default for B2B):**
```
Day 0:  LinkedIn - Connection Request
Day 1:  Email 1 - pain-point opener
Day 4:  Email 2 - case-study proof
Day 5:  LinkedIn - Message (different angle)
Day 8:  Email 3 - peer movement / social proof
Day 13: Email 4 - breakup
```

**Template B - Email + LinkedIn + Call (high-value):** add Day 8 call task for prospects with 2+ opens.

**Template C - Email only (volume):** Day 0 / 3 / 7 / 12 across 4 steps.

**Template D - Full multichannel (enterprise):** add WhatsApp Day 12, Post Interaction Day 17.

**Channel limits / type rules (Saleshandy):**

| Channel | Auto / Manual | Char limit | Notes |
|---|---|---|---|
| Email | Auto | n/a | Subject 3-7 words mobile |
| LinkedIn Connection | Manual task | 300 char note | No pitch, no links |
| LinkedIn Message | Manual task | 8000 (keep <200 words) | Different angle from email |
| LinkedIn InMail | Manual | Subject 200, body 1900 | Pro+ accounts |
| LinkedIn View Profile | Manual | n/a | Signals interest |
| Call task | Manual | 3000 chars script | Include objection handling |
| WhatsApp | Manual | 6000 (keep <100 words) | Casual tone |
| Custom task | Manual | 3000 chars | Any manual action |

**Sender capacity gate:** before locking sequence length, check `references/email-account-health.md` mental model. If sender accounts in use have Setup Score <70 or are still in warmup, recommend lower daily volume and warn the user; don't silently exceed safe sending capacity.

Output the proposed architecture as a step list (`Day N - Channel - Step type - Psychology tactic`). Get user `yes` before writing copy.

### Phase 5 - Psychology-driven copy

**Ask once, carry through:**

1. *"What tone fits your brand and audience?"*
   - Professional / Conversational (default) / Witty / Bold / Warm
2. *"Emojis allowed?"* - Yes / No (default) / Subject lines only

Adapt all copy to selected tone. If user opts out of emojis, replace emoji-dependent formats (Emoji Breakup, emoji-only subjects) with text-only variants (numbers-only subjects, "closing the loop" breakup).

**Cognitive bias toolkit** - deploy 2-3 biases per email; never more than 3:

| Bias | Where to deploy | Trigger |
|---|---|---|
| Curiosity Gap (Zeigarnik) | Subject, opener, CTA | Open a loop, don't close it - "I found something about {{Company Name}}..." |
| Loss Aversion | Opening body | Frame as loss - "you're leaving X on the table" |
| Reciprocity | Step 2-3 body | Give first - "I ran an audit on {{Company Name}} - it's yours" |
| Social Proof | Step 2-3 body | Peer movement - "67% of [industry] teams made this shift in Q1" |
| Pattern Interrupt | Subject, breakup | Break format - one-liner, numbered choice, "this is a cold email" |
| Commitment | Opening CTA | Micro-yes - "Is this even on your radar?" |
| Von Restorff | Subject lines | One anomalous element - just a number "11.3%" |
| Authority | Step 2 body | Expert/data reference - "according to [report]..." |
| Endowment | Step 2-3 | Create ownership - "your custom audit is ready" |
| Autonomy | Later CTAs | Give permission to say no - "1, 2, or 3?" |
| IKEA Effect | Mid-sequence | Ask for their input - "am I wrong about this?" |
| Pratfall Effect | Mid-late opener | Strategic small flaw - "I've rewritten this 4 times" |

If your ICP is in a specific industry, check `references/industry-tone.md` for lead-with / proof-style / CTA conventions (SaaS, Lead Gen, Digital Marketing, Recruitment, PR, E-commerce) before selecting a hook framework.

**Hook frameworks (subject + opener):**

| Framework | Subject pattern | Opener pattern |
|---|---|---|
| Pain-point opener | "{{Company Name}} is leaving replies on the table" | "Most [role]s I talk to say [pain]. Sound familiar?" |
| Specificity | "FinCo went from 14 days to 4 hours" | "Saw {{Company Name}} just [specific recent event]" |
| Curiosity | "the mistake most [role]s make with [process]" | "I found something about {{Company Name}}'s [area]..." |
| Brutally honest | "this is a cold email (but a good one)" | "Look, I know this is a cold email. Here's why it's worth reading:" |
| Pratfall self-deprecating | "3rd attempt at this email" | "This is probably the 11th cold email in your inbox today..." |
| Number-only (Von Restorff) | "11.3%" / "$47K" / "6 minutes" | "That's what FinCo cut deploy time to. Here's how:" |
| Wrong person? | "am I emailing the wrong person?" | "I've been trying to reach whoever handles [area] at {{Company Name}}" |

**Format toolkit (use when conventional doesn't fit):**

- **One-Liner** - 10-25 words. C-suite, VPs. One question, no filler.
- **Emoji Breakup** (if emojis on) / **Closing the Loop** (text-only) - 8-20 word breakup. Most disarming final.
- **Numbered Choice** - 30-50 words. Mid-late sequence. "1 = interested, 2 = not now, 3 = wrong person."
- **P.S. Lead** - Lead with the P.S. since people read it first.
- **Honest Pitch** - 50-70 words. Audiences fatigued by cold email. "Full transparency - this is a cold email."
- **I Was Wrong** - 30-50 word mid-sequence reframe. "Looking back, I led with the wrong thing."
- **Micro-Video Tease** - "Recorded a 43-second walkthrough for {{Company Name}}." Don't include the link.

**Word limits (strict, show count on every email):**

| Step | Words | Notes |
|---|---|---|
| Email 1 (cold open) | 60-100 (aim 75) | No links, no calendar, no attachments |
| Follow-ups 2-3 | 50-70 | Different angle each time |
| Breakup / final | 40-60 | Graceful, never guilt-trip |

**Spintax rules:**

- Minimum: 1 `{spin}A|B{endspin}` block per email (anywhere).
- Recommended: 3 blocks per email (subject + opener + CTA). 3 options each = 27+ unique combinations.
- Variations must be tonally similar. Never spin between radically different registers.
- Show unique-combination count on each email.
- Close every block with `{endspin}`.

**Merge tags:**

- `{{First Name|'there'}}` and `{{Company Name|'your company'}}` always with fallbacks.
- Avoid `{{Role}}` unless data quality is verified in `leads.md`.
- Words inside merge tag brackets are exempt from spam-word scan.

**Spam-word scan** - reject any line containing:

- Financial: free, guarantee, discount, profit, save, cash, cheap, bargain
- Urgency: act now, hurry, expires, limited time, don't miss, instant
- Marketing: click here, subscribe, opt in, bulk email, increase sales
- Scam: 100% free, no catch, risk free, dear friend, winner
- Pressure: bonus, exclusive, last chance, only, prize, unlimited

If a line trips the filter, rewrite and flag the change to the user.

**Per-step psychology mapping (default):**

| Step | Primary tactic | Secondary tactic |
|---|---|---|
| LinkedIn Connect (Day 0) | Reciprocity | Authority |
| Email 1 (Day 1) | Pattern Interrupt (subject) + Loss Aversion (body) | Specificity |
| Email 2 (Day 4) | Social Proof (case study) + Specificity | Authority |
| LinkedIn Message (Day 5) | Reciprocity (share insight) | Zeigarnik (open loop) |
| Email 3 (Day 8) | Social Proof (peer movement) + Loss Aversion | Zeigarnik |
| Call Task (Day 8) | Authority + Reciprocity | - |
| Email 4 / breakup (Day 13) | Breakup Effect (scarcity) | Loss Aversion |

**Buying-motion adjustment** (from icp.md frontmatter `buying_motion`):

| buying_motion | CTA pattern |
|---|---|
| `self-serve` | Low-commitment: "Try the free tool," "Check the 2-min demo video" |
| `demo` | Short call: "Open to a 15-min walkthrough?" |
| `both` | Email 1, 3 self-serve; Email 2, 4 demo. Final email always demo. |

**Surface choices to the user:** show bias selection per email ("Email 1: Pattern Interrupt + Loss Aversion") and offer alternatives before writing.

### Phase 6 - AI personalization

Read `enrichment-plan.md` if present. Pull exact column names from the Columns section. Reference them as merge tags in body copy, verbatim.

**Common AI column types to reference (or suggest if no plan exists):**

| Column | Generates | Use in |
|---|---|---|
| `{{personalized_first_line}}` | Custom opener from LinkedIn / company news | Email 1 opening |
| `{{ps_line}}` | Personal P.S. from recent content / achievements | Email 2-3 P.S. |
| `{{pain_point}}` | Role + stage specific challenge | Email 1 body |
| `{{company_insight}}` | Recent funding / hiring / launches | Email 2 body |
| `{{competitor_usage}}` | Whether they use a known competitor | Step 2-3 angle |
| `{{compliment}}` | Specific, real compliment | LinkedIn or email opener |
| `{{case_study_match}}` | Best-matching case study | Email 2 body |

If no `enrichment-plan.md` exists, suggest 2-3 columns the user could add via `prospect-enrichment`. Show how the email would change: side-by-side "without AI columns" vs. "with AI columns." Note the typical reply-rate delta (generic spintax around 3% to AI-column 8%).

### Phase 7 - Settings

Recommend defaults; let user override:

| Setting | Default | Why |
|---|---|---|
| Send window | Tue-Thu 9-11am prospect timezone | Highest open windows |
| Daily volume per sender | Check `references/email-account-health.md` - typically 30-50/day for warm accounts, 10-15 for accounts still in warmup | Avoid placement degradation |
| Sender rotation | Round-robin across healthiest accounts not in other active sequences | Diversify spam-trap risk |
| Stop on reply | ON | Always |
| Open tracking | ON (HTML) | Track engagement; needed for AI column performance |
| Unsubscribe link | Text-based footer | Saleshandy default |
| Step delay | 3-5 days (cold) / 5-7 days (re-engagement) | Spacing from PB-02 |

### Phase 8 - Launch handoff

1. Final-review checklist (display in chat):
   - [ ] Word counts within limits per step
   - [ ] Spam-word scan clean
   - [ ] Spintax + merge-tag fallbacks set on every email
   - [ ] No links / calendar / attachments in Email 1
   - [ ] Each follow-up uses a different angle
   - [ ] Bias choices match tone preference
   - [ ] AI columns referenced by exact name (if `enrichment-plan.md` exists)
2. Write paste-ready `sequence.md` (format below).
3. Suggest next skills:
   - If `leads.md` not yet enrolled: *"Enroll prospects via Saleshandy CRM, then run `inbox-triage` after first wave to handle replies."*
   - For ongoing performance: *"Run `operations-audit` after 50-100 sends to diagnose weak steps."*
   - For deliverability checks: *"Run an Inbox Radar test on the sender accounts before launch (see `references/inbox-radar.md`)."*

## `sequence.md` format

```yaml
---
campaign: <campaign>
generated_at: YYYY-MM-DD
goal: book-demo | start-conversation | lead-gen | re-engage | backlinks | pr | recruitment | custom
icp_segment: <segment name from icp.md>
icp_version: v1 | v2
channel_mix: email-only | email-linkedin | email-linkedin-call | full-multi
steps: <total step count across all channels>
tone: professional | conversational | witty | bold | warm
emojis: yes | no | subject-only
offer_name: "<from productized-offer.md if present>"
ai_columns_used: ["{{personalized_first_line}}", "{{ps_line}}", ...]  # empty list if none
---
```

```markdown
# <Goal> sequence for <segment>

## Architecture

| Step | Day | Channel | Type | Primary bias | Secondary bias |
|---|---|---|---|---|---|
| 1 | 0 | LinkedIn | Connection | Reciprocity | Authority |
| 2 | 1 | Email | Pain opener | Pattern Interrupt | Loss Aversion |
| ... | ... | ... | ... | ... | ... |

## Step 1 - Day 0 - LinkedIn Connection Request

**Note (max 300 chars):**

{spin}Hi|Hey{endspin} {{First Name|'there'}}, {spin}noticed|saw{endspin} {{Company Name}} is [specific observation]. I work with [similar role] on [topic] - would love to connect.

- Characters: ~180
- Psychology: Reciprocity (connection, not pitch) + Authority (peer signal)

---

## Step 2 - Day 1 - Email: Pain-Point Opener

**Subject A:** {{First Name|'there'}}, deploys breaking?
**Subject B:** your eng team's bottleneck
**Subject C:** shipping speed at {{Company Name|'your company'}}

**Body:**

{{personalized_first_line}}

{spin}Most CTOs I talk to at this stage say the same|Here's what I keep hearing from CTOs your size{endspin}: team {spin}tripled but the deploy pipeline didn't|grew 3x but infra stayed the same{endspin}.

Means engineers {spin}debugging YAML, not shipping product|spending 30% of the week on pipeline fires{endspin}.

{spin}Worth a quick look at how we fix that|Open to seeing how other teams solved this{endspin}?

{spin}Cheers|Best{endspin},
{{Sender Name}}

P.S. {{ps_line}}

- Words: 82
- Spintax: 48+ unique combinations
- Merge tags: First Name, Company Name, Sender Name, personalized_first_line, ps_line
- Psychology: Loss Aversion + Pattern Interrupt + Specificity + AI personalization

---

(Repeat Step N format for each step)

## Pre-launch checks

- [ ] All prospect emails verified
- [ ] AI columns populated for all prospects in scope
- [ ] Test email sent to internal address
- [ ] Sequence Score run (Saleshandy)
- [ ] Sender Setup Score >=70 (see `references/email-account-health.md`)
- [ ] Inbox Radar test passed (see `references/inbox-radar.md`)

## Saleshandy import

Head to https://my.saleshandy.com/sequence and paste each email as a separate step. Map LinkedIn / Call / WhatsApp / Custom as manual tasks at the matching day.
```

If `icp.md` is `version: v1`, append: *"Generated against ICP v1. Re-run after 50-100 sends with v2 for tighter results."*

## Edge cases

- **`company.md` missing:** route to `strategy-architect`.
- **`icp.md` missing:** route to `icp-builder`. Do not invent ICP.
- **Multiple segments, no choice given:** pick `primary_segment` from frontmatter; note the assumption.
- **No Angle Kit in `icp.md`:** generate inline (5 angles), display, then proceed. Note in `sequence.md` body.
- **`enrichment-plan.md` references columns user hasn't populated yet:** generate sequence anyway, note: *"This sequence references AI columns from `enrichment-plan.md`. Populate them in Saleshandy CRM (Enrich Prospects) before enrolling, or the merge tags will render blank."*
- **`--channel multi` but `--steps 3`:** warn that 3 steps is tight for multichannel; recommend 5+ for proper email + LinkedIn cadence.
- **User picks "witty" tone but ICP is regulated (finance, healthcare):** warn once. *"Witty tone may not land in regulated industries. Want to switch to professional or conversational?"*
- **User picks emojis-off and Phase 5 wants a Von Restorff emoji subject:** swap to the number-only variant automatically.
- **Existing `sequence.md` in workspace:** ask once - *"Existing `sequence.md` found. (a) overwrite, (b) version (move to `sequence.v1.md`), (c) cancel?"* Default to (b).
- **Word count drift during generation:** if a draft email exceeds the limit by >20%, regenerate before output. Don't ship long emails and apologize.
- **Spam-word trip in user-supplied copy:** rewrite the line and explain the change. Do not silently ship triggers.

---

*Sequence written. Next: enroll prospects and run `inbox-triage` after first wave.*
