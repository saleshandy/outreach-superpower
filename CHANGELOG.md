# Changelog

## 0.2.0 - 2026-05-19

### Added
- 6 new skills: `lead-finder`, `prospect-enrichment`, `inbox-triage`, `operations-audit`, `analytics-interpreter`, `crm-operations`
- 5 shared reference docs in `references/`: `execution-protocol.md`, `inbox-radar.md`, `email-account-health.md`, `settings.md`, `industry-tone.md`
- Multichannel support in `email-sequence-generator` (email + LinkedIn + call + WhatsApp + custom tasks)
- Search-ready filter block emitted by `icp-builder` Step 15 (consumed by `lead-finder`)
- New workspace files: `leads.md`, `enrichment-plan.md`, `inbox-log.md`, `audit-YYYY-MM-DD.md`, `analytics-snapshot-YYYY-MM-DD.md`, `crm-actions.md`

### Changed
- `email-sequence-generator`: full rewrite using PB-01 8-phase architecture + PB-02 psychology layer
- `icp-builder`: PB-10 ICP framework (firmographics/technographics/behavioral/intent) woven into existing 14 steps + new Step 15 for search-ready filter block
- `email-auditor`: expanded to 11 dimensions (7 structural + 4 craft) + deliverability red flag scan from `inbox-radar.md`
- `using-outreach-superpower` (router): 3-mode execution protocol, phase awareness (pre-launch vs post-launch), 6 new routing entries, prerequisite chaining for new skills

### Notes
- Plugin identity broadened from platform-agnostic URL-to-sequence to deep Saleshandy power-user companion
- All destructive Saleshandy MCP operations gated behind explicit user confirmation (per-message for replies; per-batch + typed-name for sequence deletes/status-changes)
- See `docs/plans/2026-05-19-outreach-superpower-expansion-design.md` in the parent repo for the full design rationale

## 0.1.0 - 2026-04-27

### Added
- Router skill `using-outreach-superpower` with auto-chaining and a 7-row routing decision table
- `strategy-architect` - website-first outreach strategy extraction with 10-item confirmation card and WebFetch fallback to pasted content
- `icp-builder` - 14-step ICP interview with v1/v2 versioning, tiered pre-fill from `company.md` (high/medium/low confidence), 3 segment cards, 5-angle Cold Email Angle Kit
- `service-productizer` - turn services into productized offers with 6-component structure (Offer Name, Outcome, Scope, Timeline, Pricing, Proof) and 5-question lite ICP
- `email-sequence-generator` - 3-7 email cold sequences with merge tags + spintext for Saleshandy, 8 goal types, buying-motion-aware CTA alternation, per-position word counts
- `email-auditor` - 7-dimension audit (Subject, Opener, Body, CTA, Formatting, Personalization, Spam) with rewritten suggestions; context-aware when `icp.md` is present; dated audit history
- File-based workspace at `./outreach-workspace/<campaign>/` with global fallback to `~/.outreach-superpower/workspace/default/`
- 6 E2E test scenarios + per-skill scenario tests for manual verification
- User-facing docs: `README.md`, `docs/USAGE.md`, `docs/DEVELOPING.md`
- MIT LICENSE

### Workspace contract (frontmatter fields downstream skills depend on)
- `company.md`: source, url, extracted_at, confidence
- `strategy.md`: goal, channels, success_metric
- `icp.md`: version, generated_at, segments, disqualifiers, source, primary_segment, buying_motion, campaign
- `productized-offer.md`: service, offer_name, generated_at, icp_source, icp_buying_motion
- `sequence.md`: goal, icp_segment, email_count, generated_at, icp_version, offer_name
- `audits/<date>.md`: audited_at, overall_score, icp_aware, subject_preview

### Known limitations
- Scenario tests are manual (no automated runner - v0.2 candidate)
- ICP interview state does not persist across new sessions (TodoWrite is in-session only - v0.2 candidate: write `icp.draft.md` mid-interview)
- No LinkedIn / multi-channel sequence skills yet (designed for, not built)
- No direct Saleshandy API integration (paste sequences manually for now)
