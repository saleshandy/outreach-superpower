# Developing

For contributors extending `saleshandy-superpowers` with new skills, new channels, or workspace contract changes.

## Plugin layout

```
saleshandy-superpowers/
  .claude-plugin/
    plugin.json                  manifest (name, version, author, keywords)
  skills/
    using-saleshandy-superpowers/
      SKILL.md                   router; auto-loads on outreach intent
    strategy-architect/
      SKILL.md                   website -> company.md + strategy.md + lite icp.md
    icp-builder/
      SKILL.md                   14-step interview -> icp.md
    service-productizer/
      SKILL.md                   service -> productized-offer.md
    email-sequence-generator/
      SKILL.md                   icp.md -> sequence.md (Saleshandy-ready)
    email-auditor/
      SKILL.md                   pasted copy -> audits/YYYY-MM-DD-<slug>.md
  tests/
    scenarios/
      router.md                  routing decisions and edge cases
      strategy-architect.md      website extraction scenarios
      icp-builder.md             interview flow (fresh, lite-upgrade, v2)
      service-productizer.md     productize from company.md
      email-sequence-generator.md  sequence with icp.md present
      email-auditor.md           paste-only and ICP-aware paths
      e2e.md                     end-to-end SDR + service-business flows
    transcripts/                 captured manual test runs (gitkept)
  docs/
    USAGE.md                     per-skill end-user examples
    DEVELOPING.md                this file
  README.md
  CHANGELOG.md
  LICENSE
```

## How to add a new skill

1. Create `skills/<skill-name>/`.
2. Add a `SKILL.md` with frontmatter. The `description` field MUST start with "Use when..." and list the triggers explicitly. Example:

   ```yaml
   ---
   name: <skill-name>
   description: Use when user wants <X>. Triggers on "<phrase 1>", "<phrase 2>", or "<phrase 3>".
   ---
   ```

3. Write the body following the existing skills' shape: Inputs, Workspace path, Process (numbered steps), and a final Cite step where the skill announces what it wrote and what to run next.
4. Add a scenario file at `tests/scenarios/<skill-name>.md` with at least 3 cases: happy path, missing-prereq path, and one edge case (vague input, blocked fetch, existing-file overwrite, etc.).
5. If the skill should auto-load on intent, update the router's routing table in `skills/using-saleshandy-superpowers/SKILL.md`. Add a row to:
   - **Routing decision table**: map intent phrases to the new skill.
   - **Auto-chaining rules**: declare required and helpful prereqs.
   - **Downstream chains**: declare what skill the new one suggests next (or mark as terminal).
6. Update the README's "The 6 skills" table.
7. Bump `version` in `.claude-plugin/plugin.json` per the versioning rules below.
8. Commit with a descriptive message: `Add <skill-name> skill`.

## How to add a new channel

Worked example: `linkedin-sequence-generator`.

The plugin's skills are deliberately split into channel-neutral (`strategy-architect`, `icp-builder`, `service-productizer`) and channel-specific (`email-sequence-generator`, `email-auditor`). Adding a LinkedIn sibling means cloning the email-specific shape and pointing it at LinkedIn-specific angles in `icp.md`.

Steps:

1. Create `skills/linkedin-sequence-generator/SKILL.md` with the same Inputs / Process structure as `email-sequence-generator`.
2. Read the same prereqs: `icp.md` (required), `company.md` (helpful), `productized-offer.md` (helpful).
3. Pull the LinkedIn-specific angle from the Angle Kit. If the Kit only has email angles, generate LinkedIn-specific opener patterns inline (shorter, no subject line, connection-request first).
4. Triggers: "LinkedIn message", "LinkedIn DM", "connection request", "InMail sequence", "LinkedIn cadence".
5. Write to `linkedin-sequence.md` in the workspace. Frontmatter mirrors `sequence.md` but with `channel: linkedin` and no subject lines.
6. Add a scenario file `tests/scenarios/linkedin-sequence-generator.md`.
7. Update the router:
   - Add a row to the routing decision table for LinkedIn intents.
   - Add prereqs to the auto-chaining table (`icp.md` required).
   - Add to downstream chains (suggests `email-auditor` is not relevant; either terminal or suggests a future `linkedin-auditor`).
8. Update the README's skill table to show 7 skills.
9. Bump the minor version (new skill is a feature add, not a breaking change).

The same recipe works for `phone-script-generator`, `multichannel-sequence-generator`, `linkedin-auditor`, etc. The workspace contract stays stable; only new files appear.

## Workspace contract

The workspace is the only state shared between skills. Every file uses YAML frontmatter as a typed contract that downstream skills read without parsing the markdown body.

For the canonical source, see `docs/plans/2026-04-27-saleshandy-superpowers-design.md` in the parent repo's design doc.

### File summary

| File | Written by | Required frontmatter fields |
|---|---|---|
| `company.md` | strategy-architect | `source`, `url`, `extracted_at`, `confidence` (high/medium/low) |
| `strategy.md` | strategy-architect | `goal`, `channels`, `success_metric` |
| `icp.md` (lite) | strategy-architect | `version: v1`, `source: strategy-architect-lite`, `needs_tightening: true`, `segments` |
| `icp.md` (full) | icp-builder | `version` (v1/v2), `generated_at`, `segments`, `disqualifiers`, `source: icp-builder-interview`, `primary_segment`, `buying_motion`, `campaign` |
| `productized-offer.md` | service-productizer | `service`, `offer_name`, `generated_at`, `icp_source`, `icp_buying_motion` |
| `sequence.md` | email-sequence-generator | `goal`, `icp_segment`, `email_count`, `generated_at`, `icp_version`, `offer_name` |
| `audits/YYYY-MM-DD-<slug>.md` | email-auditor | `audited_at`, `overall_score`, `icp_aware`, `subject_preview` |

### Key invariants

- `icp.md` with `source: icp-builder-interview` is NEVER overwritten by `strategy-architect`. The lite version is only ever overwritten if no full ICP exists yet.
- `productized-offer.md` appends a new segment card to existing `icp.md` rather than overwriting any existing segment. The `segments:` count increments by 1.
- Every audit creates a new dated file in `audits/`. History is preserved.
- Confidence flag in `company.md` controls how much `icp-builder` pre-fills. `high` = 7 of 14 steps pre-filled. `medium` = 4 of 14. `low` = none.

## Testing

Today, the test scenarios in `tests/scenarios/` are MANUAL. There is no automated runner. Each scenario is a markdown file describing a user request, expected routing decisions, and expected workspace mutations.

To run a scenario:

1. Open a fresh Claude Code session in a clean throwaway directory (`mkdir /tmp/sh-test && cd /tmp/sh-test`).
2. Read the scenario file and follow it as the user's side of the conversation.
3. Capture the transcript and save it to `tests/transcripts/<scenario>-<date>.md`.
4. Diff the resulting workspace against what the scenario specifies. Inspect frontmatter explicitly.

Scenarios cover:

- `router.md`: 5 intent variations, 2 ambiguous cases, 1 multi-campaign case.
- `strategy-architect.md`: clean fetch, partial fetch, blocked fetch, manual paste fallback.
- `icp-builder.md`: fresh interview, lite-upgrade, v2 mode, vague-answer triggers.
- `service-productizer.md`: known service, custom service, append-to-existing-icp.md flow.
- `email-sequence-generator.md`: with full Angle Kit, without Angle Kit (inline generation), self-serve vs demo CTA alternation.
- `email-auditor.md`: paste only, ICP-aware critique, missing subject detection.
- `e2e.md`: full SDR scenario (URL to sequence) and full service-business scenario.

**v0.2 candidate:** build a simple test harness that runs scenarios against the SDK and asserts on workspace state. Until then, manual testing is the contract.

## Style rules

- **No emdashes.** Anywhere. Use regular dashes (-), commas, or rewrite the sentence. Before committing, grep the long-dash character (Unicode U+2014) across `skills/`, `docs/`, and `README.md`; the result must be empty.
- **Frontmatter is the contract.** Adding a new optional field is a minor bump. Renaming or removing an existing field is a breaking change (major bump). Treat frontmatter as a public API.
- **Announce-and-wait for handoffs, not direct invokes.** When skill A needs skill B's output, A announces the missing prereq and stops. The router detects the announcement, runs B, then re-enters A. This keeps each skill stateless and testable.
- **Cite the source.** When a skill uses data from another file, cite it in chat (e.g., *"Pulling segment 'Mid-market B2B SaaS' from icp.md"*).
- **Label assumptions.** Tag inferred data with `[ASSUMPTION]` so the user can challenge before files are written.
- **Workspace-first.** Every skill reads the workspace before asking the user. Don't re-ask for data that's already in `company.md` or `icp.md`.
- **Persist outputs.** Every skill writes a workspace file, not just chat output. Chat is ephemeral; files are the contract.

## Versioning

Semver. We're in `v0.x` (pre-release).

| Change | Bump |
|---|---|
| Bug fix in a skill (typo, clarification, internal reordering) | patch (0.1.0 -> 0.1.1) |
| New skill, new optional frontmatter field, new routing rule | minor (0.1.0 -> 0.2.0) |
| Renamed or removed frontmatter field, changed file path, breaking workspace contract | major (0.x.0 -> 1.0.0) |

Once we hit `v1.0.0`, the workspace contract and frontmatter schemas freeze. Any breaking change requires a major bump and a migration note in `CHANGELOG.md`.

Update `version` in `.claude-plugin/plugin.json` and add a `CHANGELOG.md` entry on every commit that touches user-visible behavior.
