# Router scenarios

For each of the 7 scenarios below, in a fresh Claude Code session with this plugin installed:

1. **Cold start, no workspace.** User says: *"Help me write cold emails for my SaaS."*
   - Expected: router loads → invokes `strategy-architect` (no workspace yet) → asks for URL.

2. **Workspace exists, sequence intent.** User says: *"Generate a sequence for the demo goal."*
   - Workspace setup: create `outreach-workspace/default/icp.md` and `outreach-workspace/default/company.md` with valid frontmatter and minimal content.

   Sample `icp.md` fixture:
   ```yaml
   ---
   version: v1
   generated_at: 2026-04-27
   segments: 3
   ---
   # ICP Summary
   | Field | Value |
   |---|---|
   | Who it's for | VP Engineering at Series A SaaS (50-200 employees) |
   | Primary pain | Hiring is slow and ATSes are bloated |
   | Primary outcome | First hire in 14 days |
   ```

   Sample `company.md` fixture:
   ```yaml
   ---
   url: https://example.com
   extracted_at: 2026-04-27
   confidence: high
   ---
   # Acme Recruit
   - Industry: HR Tech
   - Value prop: AI-powered ATS for startups
   ```

   - Expected: router loads → invokes `email-sequence-generator` directly (skips icp-builder).

3. **Sequence intent, no ICP.** User says: *"Write me a 4-email sequence."*
   - Workspace empty.
   - Expected: router announces auto-chain → invokes `icp-builder` → on completion, resumes `email-sequence-generator`.

4. **Audit-only.** User pastes:

   ```
   Subject: Quick question about your hiring process
   Hi {First Name},

   Hope you're doing well! I noticed Acme is hiring engineers. We help startups like yours hire 3x faster with our AI-powered ATS.

   Are you free for a 30-min call next week to discuss?

   Best,
   Sarah
   ```

   - Expected: router loads → invokes `email-auditor`. Does not require workspace.

5. **Service intent.** User says: *"I run an agency, help me productize my SEO offering."*
   - Expected: router loads → invokes `service-productizer`.

6. **Campaign override.** User says: *"Build outreach for https://acme.com --campaign acme-q2"*.
   - Expected: router loads → workspace path becomes `./outreach-workspace/acme-q2/` (not `./outreach-workspace/default/`) → invokes `strategy-architect` with that workspace.

7. **Ambiguous intent.** User says: *"Help me with cold email."* (no other context).
   - Expected: router asks a clarifying multi-choice question:
     > *"Got it. Which would help most?*
     > *(a) Build outreach strategy from a URL*
     > *(b) Define your ICP*
     > *(c) Productize a service offering*
     > *(d) Write a cold email sequence*
     > *(e) Audit existing email copy*"

   Then routes based on the user's choice.
