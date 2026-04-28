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
