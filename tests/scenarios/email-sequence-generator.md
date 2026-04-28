# email-sequence-generator scenarios

1. **Happy path.** Workspace has icp.md (full v1 from icp-builder, with `primary_segment` and `buying_motion` in frontmatter), company.md. User: *"Generate a 4-email sequence for booking demos."*
   - Expected: skill skips ICP/company questions (frontmatter introspection), asks segment + length, generates 4 emails with merge tags + spintext, writes sequence.md.

2. **No ICP.** Workspace empty. User: *"Generate a sequence."*
   - Expected: skill announces handoff to icp-builder. Router loads icp-builder, which runs full interview, writes icp.md, then resumes email-sequence-generator.

3. **Goal coverage** (8 sub-scenarios - run as separate sessions; spot-check 2 if time-constrained).
   - 3a. book-demo: sequence references product demo + calendar.
   - 3b. lead-gen: sequence opens with pain point + ROI angle.
   - 3c. backlinks: sequence references user's content + value exchange.
   - 3d. recruitment: sequence frames culture + growth opportunity.
   - 3e. promotions: sequence uses urgency + exclusive offer framing.
   - 3f. pr: sequence pitches newsworthy angle + expert positioning.
   - 3g. partnership: sequence proposes mutual benefit + roadmap.
   - 3h. nurture: sequence delivers value-add content with progressive engagement.

4. **Saleshandy syntax.** Generated sequence.md must contain `{First Name}`, `{Company}`, and at least one `{spin}A|B{endspin}` block per email.

5. **Length range.** Test with 3, 5, and 7 emails. Each respects the structure: first = value/problem, middle = social proof/objection, final = strong CTA.

6. **Pulled from productized-offer.md.** Workspace has icp.md + productized-offer.md. User: *"Sequence for my Pipeline Jumpstart offer."*
   - Expected: skill reads productized-offer.md, uses offer name in subject lines, references scope + outcome in body, defaults goal to lead-gen.

7. **ICP v1 footer.** When icp.md frontmatter shows `version: v1`, the generated sequence.md ends with: *"Generated against ICP v1. Re-run after 50-100 sends with v2 for tighter results."*
