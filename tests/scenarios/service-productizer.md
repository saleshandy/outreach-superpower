# service-productizer scenarios

1. **Agency happy path.** User: *"I run an SEO agency. Productize my link-building service."* Workspace empty.
   - Expected: skill detects no `company.md`, announces handoff to `strategy-architect` and waits. Router loads strategy-architect, which asks for URL. User provides URL. After `company.md` is written, service-productizer resumes: lists extracted services -> user picks "link-building" -> productizes -> asks if user wants the 3-email sequence -> on yes, hands off to email-sequence-generator.

2. **With company.md present.** User: *"Productize my podcast production service."*
   - Expected: skill reads company.md service list, asks which to pitch first.

3. **Service not in extracted list.** User picks "Custom: AI strategy consulting."
   - Expected: skill accepts, asks 4 follow-up questions to fill gaps (target outcome, scope, timeline, differentiator).

4. **Skip auto-chain.** After productizing, user says "no, just save the offer."
   - Expected: skill writes `productized-offer.md`, doesn't invoke email-sequence-generator.

5. **Existing icp.md from icp-builder.** Workspace has full v1 icp.md with 3 segments and `primary_segment` in frontmatter. User productizes a service.
   - Expected: skill writes the offer-specific ICP segment as a NEW segment card appended to existing icp.md (not overwriting). Cites: *"Appended segment 'X' to existing icp.md."*
