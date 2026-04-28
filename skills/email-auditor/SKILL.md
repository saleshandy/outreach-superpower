---
name: email-auditor
description: Use when user pastes existing email copy and wants it reviewed, scored, or improved. Triggers on "audit this email," "review my cold email," "rate this copy," or any pasted email body with subject + body structure.
---

# Email Auditor

Audit pasted cold email copy across 7 dimensions, rate each 1-5, output strengths + rewritten suggestions, and persist the audit to a dated file.

## Inputs

- **Required:** pasted email copy (detect subject + body)
- **Optional (helpful):** `icp.md` in workspace - enables context-aware critique

## Process

### Step 1 - Detect copy structure

Parse the pasted text for:
- Subject line (after "Subject:" or first line if short)
- Body
- Sender name / signature (optional)

If no subject line is detectable, ask once: *"What's the subject line?"*

### Step 2 - Read context (if available)

Read `outreach-workspace/<campaign>/icp.md` if it exists. Note primary segment + segment-specific buying motion. This sharpens the critique.

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

### Step 4 - Output to chat

```markdown
## Audit: <subject line preview>

**Overall: X/35**

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
