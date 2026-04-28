# strategy-architect scenarios

1. **Happy path.** User says: *"Build me an outreach strategy for https://saleshandy.com"*.
   - Expected: WebFetch on home/pricing/customers/about -> 10-item confirmation card -> user types "all good" -> writes `company.md`, `strategy.md`, `icp.md` (lite, version: v1, marked needs-tightening).

2. **WebFetch fails.** Same flow, but block network access.
   - Expected: skill catches the failure, asks user to paste home + customers page content. Continues.

3. **Edits.** User responds with edits: "1: change to 'inbox-first cold email,' 4: add SOC2."
   - Expected: skill applies edits, re-shows updated card, asks for confirmation again.

4. **Custom campaign.** User invokes with `--campaign acme`.
   - Expected: workspace writes go to `outreach-workspace/acme/`, not `default/`.
