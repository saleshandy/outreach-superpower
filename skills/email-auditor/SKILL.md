---
name: email-auditor
description: Use when user pastes existing email copy and wants it reviewed, scored, or improved. Triggers on "audit my emails," "audit this email," "review this copy," "review my cold email," "score this email," "rate my outreach," "rate this copy," "is this email good," or any pasted email body with subject + body structure.
---

# Email Auditor

Audit pasted cold email copy across 11 dimensions (7 structural + 4 craft), rate each 1-5, list deliverability red flags from `references/inbox-radar.md`, output strengths + rewritten suggestions, and persist the audit to a dated file.

## Inputs

- **Required:** pasted email copy (detect subject + body)
- **Optional (helpful):** `icp.md` in workspace - enables context-aware critique

## Workspace path

Default campaign: `default`. If user invocation includes `--campaign <name>`, use that. Full rule: see `using-outreach-superpower`.

## Process

### Step 1 - Detect copy structure

Parse the pasted text for:
- **Subject line:** look for explicit `Subject:` prefix first. If absent, fall back to: first non-empty line that does NOT start with greeting tokens (`Hi`, `Hello`, `Hey`, `Dear`, `Good`) AND is shorter than 80 characters.
- **Body:** everything after the subject line.
- **Sender name / signature** (optional): bottom-of-paste lines after a blank line.

If no subject line is detectable by either rule, ask once: *"What's the subject line?"* Then re-run detection with the supplied subject before proceeding to Step 2.

### Step 2 - Read context (if available)

Read `outreach-workspace/<campaign>/icp.md` if it exists. Extract:
- Primary segment name (e.g., "VP Engineering at Series A SaaS")
- Buying motion (self-serve / demo / both - from icp.md fields)
- Top 1-2 angles from the Angle Kit (if present)

Set `icp_aware: true` in the audit frontmatter. Use the extracted info during Step 3 scoring (see ICP-aware adjustments below).

### Step 3 - Score 11 criteria (1=Poor, 5=Excellent)

#### 3a. Structural dimensions (7)

| Criterion | What to check |
|---|---|
| **Subject Line** | Clear, engaging, relevant; curiosity without clickbait; no spam triggers; personalization present; >5 words; <60 chars |
| **Opening Line** | Grabs attention; personalized; relevant to recipient's industry/role; avoids "Hope you're doing well" |
| **Body Copy** | Clear, concise, valuable; recipient-focused (not product-focused); credibility/social proof present; <150-200 words |
| **Call-to-Action** | Clear next step; encourages reply; binary CTA preferred ("Open to a 10-min call?") |
| **Formatting & Readability** | Short sentences; no walls of text; conversational tone; mobile-friendly |
| **Personalization & Human Touch** | References work/company/industry; feels 1:1 not mass; empathetic |
| **Spam & Deliverability** | No spam triggers, excessive caps, exclamation marks; <=1 link; subject <60 chars; body <200 words |

#### 3b. Craft dimensions (4) - from PB-02

| Criterion | What to check |
|---|---|
| **Tone match** | Copy matches the ICP brand voice / segment expectations (professional / conversational / witty / bold / warm). Flag if the copy is bro-y when ICP is enterprise; flag if it's stiff when ICP is creator/SMB. If `icp.md` is absent, infer tone from the copy itself and call out internal inconsistency (e.g., warm opener, aggressive CTA). |
| **Hook strength** | First-line specificity, relevance, curiosity factor. Score down: "Hope this finds you well," generic "I came across your company," role/title-only references. Score up: named achievement, named pain, named recent event, specific observation. |
| **Spintax usage** | Count of `{spin}...{endspin}` blocks; presence in high-frequency phrases (greeting, body, sign-off, CTA). Target: minimum 1 in greeting, 2-3 in body, 1 in sign-off. Flag if 0 blocks present (no variant diversity, fails A/Z testing). Flag if variants spin between tonally inconsistent registers (e.g., "Hey" vs "Dear Sir"). |
| **Word economy** | Body word count. Target <120 for email body (PB-02: 60-100 for opening, 50-70 for follow-up). Score down: unnecessary preamble ("I just wanted to reach out"), filler sentences that don't pass the "So What?" test, sentences >25 words. |

**ICP-aware adjustments (only if icp.md was loaded):**
- **Personalization & Human Touch:** weight against segment fit. Flag if email targets generic "founders" when ICP says "VP Engineering at Series A SaaS."
- **Call-to-Action:** flag mismatches with the buying motion (e.g., "Book a 30-min call" CTA when ICP says self-serve preferred).
- **Body Copy:** flag if value prop doesn't match the segment's primary pain.
- **Tone match:** weight against the ICP's declared brand voice or segment expectations. State the segment name explicitly when calling out a mismatch.

State the segment name explicitly in feedback when raising an ICP-related issue: *"Doesn't fit your ICP segment 'VP Engineering at Series A SaaS' - they prefer self-serve."*

### Step 3c - Deliverability red flags

After scoring the 11 dimensions, run a focused deliverability scan against `references/inbox-radar.md`. This is a separate check (not double-counted in the Spam & Deliverability score) that surfaces concrete risk patterns the user should fix before sending.

Scan for:

| Red flag | Source rule |
|---|---|
| Spam-trigger words (e.g., free, guarantee, click here, act now, 100%, urgent, winner) | `inbox-radar.md` "Content suggestions" + PB-02 spam-word list |
| Body word count outside 60-150 word ideal range | `inbox-radar.md` "Content suggestions" |
| Subject >60 characters or ALL CAPS or 3+ exclamation marks | inbox-radar formatting concerns |
| Multiple links in opening email (>1 link, or any link in step 1) | PB-02 + inbox-radar |
| Imbalanced text-to-link or text-to-image ratio (e.g., 1 link in 5-word body) | inbox-radar |
| Generic unverified merge tags without fallbacks (`{{First Name}}` without `\|'there'`) | PB-02 merge-tag rules |
| Subject line uses urgency stack: "URGENT", "ACT NOW", "Limited time" | inbox-radar spam-trigger words |

Output format for this section in the audit:

```markdown
### Deliverability red flags

- **Spam trigger detected:** "free trial" in body line 3. Suggest: "no-commitment pilot" or "14-day test run."
- **Length warning:** body is 187 words; ideal range is 60-150 (per `references/inbox-radar.md`).
- (...if no flags found, write: "No red flags detected. See `references/inbox-radar.md` for the full deliverability framework.")
```

If the email has 3+ red flags, recommend the user run an Inbox Radar test before sending and cite the reference.

### Step 4 - Output to chat

```markdown
## Audit: <subject line preview>

**Overall: X/55** (sum of 11 criterion ratings)
**Structural: X/35** | **Craft: X/20**

| Criterion | Rating | Feedback |
|---|---|---|
| Subject Line | X/5 | ... |
| Opening Line | X/5 | ... |
| Body Copy | X/5 | ... |
| Call-to-Action | X/5 | ... |
| Formatting & Readability | X/5 | ... |
| Personalization & Human Touch | X/5 | ... |
| Spam & Deliverability | X/5 | ... |
| Tone match | X/5 | ... |
| Hook strength | X/5 | ... |
| Spintax usage | X/5 | ... |
| Word economy | X/5 | ... |

### Deliverability red flags
- (list from Step 3c, or "No red flags detected" if clean)

### Strengths
- ...

### Improvements
- **Subject:** <rewritten suggestion>
- **Opener:** <rewritten suggestion>
- **Body:** <rewritten suggestion>
- **CTA:** <rewritten suggestion>
- **Spintax:** <added variants if missing>

*(Improvements list pointed fixes per component; Rewritten draft is the synthesized paste-ready copy.)*

### Rewritten draft
Subject: ...
Body: ...
```

### Step 5 - Write audit file

Filename: `audits/YYYY-MM-DD-<slugified-subject>.md`. Slug = first 5 words of subject, lowercase, hyphenated. If file exists, append `-2`, `-3`, etc.

Frontmatter:

```yaml
---
audited_at: YYYY-MM-DD
overall_score: X/55
structural_score: X/35
craft_score: X/20
deliverability_flags: <count>
icp_aware: true | false
subject_preview: <first 60 chars>
---
```

Body: include the original copy (for reference), then the full audit output above (11 dimensions + deliverability red flags + strengths + improvements + rewritten draft).

### Step 6 - Cite the source

- If `icp.md` was used, end the audit with a one-liner: *"Critique informed by `icp.md` segment '<segment name>'."*
- If any deliverability red flags fired, end with: *"Deliverability framework: `references/inbox-radar.md`. Re-run an Inbox Radar test after revisions."*
