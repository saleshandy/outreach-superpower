# icp-builder scenarios

1. **Cold start.** Empty workspace. User: *"Build me an ICP."*
   - Expected: 14-step interview, one question per message, each with 2-3 numbered suggestions.
   - Final output: `icp.md` (version: v1) with ICP Summary table, 3 Segment Cards, Targeting Filters, Angle Kit (5 angles), Disqualifiers.

2. **Skip questions answerable from `company.md`.** Workspace has `company.md` from strategy-architect run.
   - Expected: skill skips Step 1 (website/product) since it's known. Progress tracker shows "Step X / 13" or notes skipped.
   - Skip rules: if `company.md` has `confidence: high|medium`, pre-fill Steps 1, 2, 5, 13 and ask only for confirmation. If `confidence: low`, re-ask from scratch.

3. **Vague answer follow-up.** At Step 5 ("best 10 customers"), user replies "tech companies."
   - Expected: skill asks one tightening follow-up before continuing.

4. **Skip command.** User types "skip" mid-interview.
   - Expected: skill accepts, moves on, marks the skipped field as `[SKIPPED]` in output.

5. **v2 mode.** Re-run with existing `icp.md` v1 in workspace.
   - Expected: skill detects v1, asks: *"Generating v2 (tightening). Share reply data from your last 50-100 sends?"* Then runs a shorter v2 flow.

6. **Lite ICP exists (from strategy-architect).** Workspace has `icp.md` with `source: strategy-architect-lite` and `needs_tightening: true`.
   - Expected: skill detects lite version, runs full 14-step interview (skipping pre-filled), upgrades to full v1 with `source: icp-builder-interview`.
