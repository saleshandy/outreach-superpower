# icp-builder scenarios

1. **Cold start.** Empty workspace. User: *"Build me an ICP."*
   - Expected: 14-step interview, one question per message, each with 2-3 numbered suggestions.
   - Final output: `icp.md` (version: v1) with ICP Summary table, 3 Segment Cards, Targeting Filters, Angle Kit (5 angles), Disqualifiers.

2. **Skip questions answerable from `company.md`.** Workspace has `company.md` from strategy-architect run.
   - Expected: skill skips pre-fillable steps. Progress tracker denominator adjusts (e.g. `Step X / 10` if 4 pre-filled, `Step X / 7` if 7 pre-filled).
   - Tiered pre-fill rules:
     - `confidence: high` -> pre-fill Steps 1, 2, 5, 6, 8, 9, 13 (7 of 14).
     - `confidence: medium` -> pre-fill Steps 1, 2, 5, 13 (4 of 14).
     - `confidence: low` -> re-ask all 14 from scratch.

3. **Vague answer follow-up.** At Step 5 ("best 10 customers"), user replies "tech companies."
   - Expected: skill asks one tightening follow-up before continuing.

4. **Skip command.** User types "skip" mid-interview.
   - Expected: skill accepts, moves on, marks the skipped field as `[SKIPPED]` in output.

5. **v2 mode.** Re-run with existing `icp.md` v1 in workspace.
   - Expected: skill detects v1, asks: *"Generating v2 (tightening). Share reply data from your last 50-100 sends?"* Then runs a shorter v2 flow.

6. **Lite ICP exists (from strategy-architect).** Workspace has `icp.md` with `source: strategy-architect-lite` and `needs_tightening: true`.
   - Expected: skill detects lite version, runs full 14-step interview (skipping pre-filled), upgrades to full v1 with `source: icp-builder-interview`.

7. **Abandon and resume.** User answers Steps 1-7, then session ends. New session: user invokes icp-builder again.
   - Expected: skill detects no `icp.md` exists (full interview never reached Step Z), restarts from Step 1, but pre-fills any steps answerable from `company.md` if present. Output a one-liner: *"No prior ICP found. Restarting from Step 1."*
   - v0.2 enhancement (not tested in v0.1): write `icp.draft.md` after Step 7 to enable true resume.
