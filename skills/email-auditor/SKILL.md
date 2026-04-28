---
name: email-auditor
description: Use when user pastes existing email copy and wants it reviewed, scored, or improved. Triggers on "audit this email," "review my cold email," "rate this copy," or any pasted email body with subject + body structure.
---

# Email Auditor

Audit pasted cold email copy across 7 dimensions, rate each 1-5, output strengths + rewritten suggestions, and persist the audit to a dated file.

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

### Step 3 - Score 7 criteria (1=Poor, 5=Excellent)

| Criterion | What to check |
|---|---|
| **Subject Line** | Clear, engaging, relevant; curiosity without clickbait; no spam triggers; personalization present; >5 words; <60 chars |
| **Opening Line** | Grabs attention; personalized; relevant to recipient's industry/role; avoids "Hope you're doing well" |
| **Body Copy** | Clear, concise, valuable; recipient-focused (not product-focused); credibility/social proof present; <150-200 words |
| **Call-to-Action** | Clear next step; encourages reply; binary CTA preferred ("Open to a 10-min call?") |
| **Formatting & Readability** | Short sentences; no walls of text; conversational tone; mobile-friendly |
| **Personalization & Human Touch** | References work/company/industry; feels 1:1 not mass; empathetic |
| **Spam & Deliverability** | No spam triggers, excessive caps, exclamation marks; <=1 link; subject <60 chars; body <200 words |

**ICP-aware adjustments (only if icp.md was loaded):**
- **Personalization & Human Touch:** weight against segment fit. Flag if email targets generic "founders" when ICP says "VP Engineering at Series A SaaS."
- **Call-to-Action:** flag mismatches with the buying motion (e.g., "Book a 30-min call" CTA when ICP says self-serve preferred).
- **Body Copy:** flag if value prop doesn't match the segment's primary pain.

State the segment name explicitly in feedback when raising an ICP-related issue: *"Doesn't fit your ICP segment 'VP Engineering at Series A SaaS' - they prefer self-serve."*

### Step 4 - Output to chat

```markdown
## Audit: <subject line preview>

**Overall: X/35** (sum of 7 criterion ratings)

| Criterion | Rating | Feedback |
|---|---|---|
| Subject Line | X/5 | ... |
| Opening Line | X/5 | ... |
| Body Copy | X/5 | ... |
| Call-to-Action | X/5 | ... |
| Formatting & Readability | X/5 | ... |
| Personalization & Human Touch | X/5 | ... |
| Spam & Deliverability | X/5 | ... |

### Strengths
- ...

### Improvements
- **Subject:** <rewritten suggestion>
- **Opener:** <rewritten suggestion>
- **Body:** <rewritten suggestion>
- **CTA:** <rewritten suggestion>

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
overall_score: X/35
icp_aware: true | false
subject_preview: <first 60 chars>
---
```

Body: include the original copy (for reference), then the full audit output above.

### Step 6 - Cite the source

If `icp.md` was used, end the audit with a one-liner: *"Critique informed by `icp.md` segment '<segment name>'."*
