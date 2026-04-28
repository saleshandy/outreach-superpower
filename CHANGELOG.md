# Changelog

## 0.1.0 - 2026-04-27

### Added
- Router skill `using-saleshandy-superpowers` with auto-chaining and a 7-row routing decision table
- `strategy-architect` - website-first outreach strategy extraction with 10-item confirmation card and WebFetch fallback to pasted content
- `icp-builder` - 14-step ICP interview with v1/v2 versioning, tiered pre-fill from `company.md` (high/medium/low confidence), 3 segment cards, 5-angle Cold Email Angle Kit
- `service-productizer` - turn services into productized offers with 6-component structure (Offer Name, Outcome, Scope, Timeline, Pricing, Proof) and 5-question lite ICP
- `email-sequence-generator` - 3-7 email cold sequences with merge tags + spintext for Saleshandy, 8 goal types, buying-motion-aware CTA alternation, per-position word counts
- `email-auditor` - 7-dimension audit (Subject, Opener, Body, CTA, Formatting, Personalization, Spam) with rewritten suggestions; context-aware when `icp.md` is present; dated audit history
- File-based workspace at `./outreach-workspace/<campaign>/` with global fallback to `~/.saleshandy-superpowers/workspace/default/`
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
