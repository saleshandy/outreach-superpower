# email-auditor scenarios

1. **Generic audit (no workspace).** User pastes:

   ```
   Subject: Quick question
   Hi {First Name}, hope you're doing well! I'm reaching out from Acme. We help companies grow. Are you free for a 30-min call next week to discuss?
   ```

   Expected:
   - 7-criterion table with 1-5 ratings + per-criterion feedback
   - Overall score (e.g., 18/35)
   - Strengths section
   - Improvements section with rewritten subject + body
   - File written: `outreach-workspace/default/audits/2026-04-27-quick-question.md`

2. **Context-aware audit (icp.md present).** Workspace has `icp.md` with primary segment "VP Engineering at Series A SaaS." User pastes a generic email targeting "founders."

   Expected: audit calls out the targeting mismatch (CTA doesn't match segment buying motion).

3. **History.** Run scenario 1 twice with different emails. Verify `audits/` accumulates 2 files; older audit not overwritten.

4. **Body-only paste (subject fallback).** User pastes:

   ```
   Hi {First Name},

   Saw you recently expanded your engineering team. We help startups hire 3x faster with our AI-powered ATS. Free for a 10-min call this week?

   Sarah
   ```

   Expected:
   - Skill detects no `Subject:` prefix and the first line is a greeting → asks once: *"What's the subject line?"*
   - User answers: *"Quick question about your hiring process"*
   - Skill re-runs detection, proceeds with audit.
   - File written: `audits/2026-04-27-quick-question-about.md` (subject slug uses the supplied subject).
