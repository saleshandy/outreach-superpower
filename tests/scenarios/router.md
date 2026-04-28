# Router scenarios

For each scenario, in a fresh Claude Code session with this plugin installed:

1. **Cold start, no workspace.** User says: *"Help me write cold emails for my SaaS."*
   - Expected: router loads → invokes `strategy-architect` (no workspace yet) → asks for URL.

2. **Workspace exists, sequence intent.** User says: *"Generate a sequence for the demo goal."*
   - Workspace has `icp.md`, `company.md`.
   - Expected: router loads → invokes `email-sequence-generator` directly (skips icp-builder).

3. **Sequence intent, no ICP.** User says: *"Write me a 4-email sequence."*
   - Workspace empty.
   - Expected: router announces auto-chain → invokes `icp-builder` → on completion, resumes `email-sequence-generator`.

4. **Audit-only.** User pastes an email body.
   - Expected: router loads → invokes `email-auditor`. Does not require workspace.

5. **Service intent.** User says: *"I run an agency, help me productize my SEO offering."*
   - Expected: router loads → invokes `service-productizer`.
