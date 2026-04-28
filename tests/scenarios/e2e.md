# E2E scenarios

These exercise the full plugin (router + multiple skills + auto-chaining + workspace persistence).

These are MANUAL tests. They cannot run inside a subagent - each requires a fresh Claude Code session with the plugin installed locally. Run them after Task 10 (local install) completes.

## E2E-1: Cold start, full SaaS workflow
1. Empty workspace. User: *"Help me write cold emails for https://saleshandy.com targeting SaaS founders."*
2. Expected:
   - Router → strategy-architect (extracts site, asks confirmation).
   - User confirms.
   - Router suggests next: "ICP build or sequence?"
3. User: *"Build the ICP."*
4. Expected: icp-builder runs. Steps 1, 2, 5, 13 pre-filled from `company.md` (high confidence - `strategy-architect` extracted them). Steps 6, 8, 9 also pre-filled if confidence is high (per the tiered pre-fill rule).
5. User: *"Now generate a 4-email demo sequence."*
6. Expected: email-sequence-generator runs, segment auto-defaulted to `primary_segment`, writes sequence.md with 4 emails containing merge tags + spintext per design rules.
7. User pastes Email 1 back: *"Audit this."*
8. Expected: email-auditor runs, scores 7 dimensions, suggests rewrites - context-aware (cites ICP segment + buying motion).

**Verify final workspace:**

```
outreach-workspace/default/
├── company.md
├── strategy.md
├── icp.md
├── sequence.md
└── audits/
    └── 2026-04-27-*.md
```

## E2E-2: Service business workflow
1. Empty workspace. User: *"I run an SEO agency at https://example-seo-agency.com. Help me sell my link-building service."*
2. Expected:
   - Router → service-productizer.
   - service-productizer detects no `company.md`, announces handoff: "Run `strategy-architect` first." Router loads strategy-architect.
   - strategy-architect runs, writes `company.md` (with services as sub-bullets per the parsing rule).
   - service-productizer resumes.
   - Lists extracted services. User picks "link-building."
   - Productizes (offer name, outcome, scope, timeline, pricing, proof).
   - Asks 5 ICP questions for the offer.
   - Writes `productized-offer.md`. Appends offer-specific segment to `icp.md` (with `[NEEDED]` markers per the 4→9 mapping).
   - Asks: "draft 3-email sequence?" - user says yes.
   - Hands off to email-sequence-generator with `goal: lead-gen` and the offer's segment.

**Verify:** workspace has `company.md`, `productized-offer.md`, `icp.md` (with appended segment), `sequence.md`.

## E2E-3: Multi-campaign isolation
1. Run E2E-1 with `--campaign saas`.
2. Run E2E-2 with `--campaign agency`.
3. Verify: `outreach-workspace/saas/` and `outreach-workspace/agency/` are separate; no cross-pollution.

## E2E-4: ICP versioning
1. Use workspace from E2E-1.
2. Re-invoke icp-builder. User: *"I want to tighten the ICP."*
3. Expected: icp-builder detects `version: v1`, prompts (Turn 1): "Send your reply data when ready, I'll wait." User pastes reply data.
4. icp-builder runs Turn 2: 5 tightening questions.
5. Verify: `icp.md` is now `version: v2`; `icp.v1.md` exists as archive.

## E2E-5: Failure modes
1. **WebFetch blocked.** strategy-architect should detect homepage fetch failure → ask user to paste homepage content. Continue with paste fallback (`source: manual-paste` or `mixed`).
2. **icp-builder mid-interview, user types "skip" 4 times.** Skill should accept skips for non-core steps (e.g., 4, 7, 12), but refuse to skip core steps (3, 5, 8) with a clarification.
3. **email-sequence-generator with no icp.md, user denies auto-chain.** Skill should error gracefully: *"I need an ICP first. Run icp-builder, or paste your target segment manually."*
4. **strategy-architect re-run on existing workspace.** Should ask: "Existing strategy files found. (a) overwrite, (b) version, or (c) merge?" Default (b). icp.md with `version: v2` should be protected (never overwritten).
5. **email-auditor on body-only paste.** Should ask "What's the subject line?" once, then proceed.

## E2E-6: Frontmatter contract integrity
For each workspace file written across E2E-1 through E2E-4, verify the YAML frontmatter parses correctly and contains all required fields:
- `company.md`: source, url, extracted_at, confidence
- `strategy.md`: goal, channels, success_metric
- `icp.md`: version, generated_at, segments, disqualifiers, source, primary_segment, buying_motion, campaign
- `productized-offer.md`: service, offer_name, generated_at, icp_source, icp_buying_motion
- `sequence.md`: goal, icp_segment, email_count, generated_at, icp_version, offer_name (optional)
- `audits/<date>.md`: audited_at, overall_score, icp_aware, subject_preview

Spot-check: each frontmatter is valid YAML (parses without error in a YAML parser, e.g., `python -c "import yaml; yaml.safe_load(open('FILE').read().split('---')[1])"`).

## Pass criteria

All 6 E2E scenarios complete without manual intervention beyond expected user inputs. Failure modes (E2E-5) trigger the documented graceful handlers, not silent failures.

## Run order

E2E-1 first (validates the happy path of all 5 skills + router). Only proceed to E2E-2 once E2E-1 passes. E2E-3 through E2E-6 can run in any order after E2E-1 + E2E-2.
