# email-sequence-generator scenarios

1. **Happy path.** Workspace has icp.md (full v1 from icp-builder, with `primary_segment` and `buying_motion` in frontmatter), company.md. User: *"Generate a 4-email sequence for booking demos."*
   - Expected: skill skips ICP/company questions (frontmatter introspection), asks segment + length, generates 4 emails with merge tags + spintext, writes sequence.md.

2. **No ICP.** Workspace empty. User: *"Generate a sequence."*
   - Expected: skill announces handoff to icp-builder. Router loads icp-builder, which runs full interview, writes icp.md, then resumes email-sequence-generator.

3. **Goal coverage.** Run for each of 8 goals: book-demo, lead-gen, backlinks, recruitment, promotions, pr, partnership, nurture.
   - Expected: each generates a goal-appropriate sequence (e.g., backlinks references the user's content; recruitment frames culture).

4. **Saleshandy syntax.** Generated sequence.md must contain `{First Name}`, `{Company}`, and at least one `{spin}A|B{endspin}` block per email.

5. **Length range.** Test with 3, 5, and 7 emails. Each respects the structure: first = value/problem, middle = social proof/objection, final = strong CTA.

6. **Pulled from productized-offer.md.** Workspace has icp.md + productized-offer.md. User: *"Sequence for my Pipeline Jumpstart offer."*
   - Expected: skill reads productized-offer.md, uses offer name in subject lines, references scope + outcome in body, defaults goal to lead-gen.

7. **ICP v1 footer.** When icp.md frontmatter shows `version: v1`, the generated sequence.md ends with: *"Generated against ICP v1. Re-run after 50-100 sends with v2 for tighter results."*
