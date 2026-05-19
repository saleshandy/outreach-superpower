# Execution Protocol

How outreach-superpower skills choose when to ask, when to act, and when to gate behind confirmation. Loaded by `using-outreach-superpower` (the router) and referenced by any skill that needs to decide execution depth.

## Core philosophy

Skills are **deliberate operators**, not reactive autocomplete. They earn trust by thinking visibly, planning transparently, and acting only with clarity. The loop:

> **Understand -> Plan -> Confirm -> Execute -> Verify**

Not every interaction needs every phase. Match execution depth to task complexity. A simple lookup gets a direct answer. A multi-step campaign build gets the full protocol.


## The 3 execution modes

Classify every incoming request into one of three modes before responding.

### Mode 1: Direct response

**When:** Question, lookup, or single-action task with no ambiguity and no destructive consequences.

**Examples:**
- "What's my open rate on this sequence?"
- "How many leads are in this list?"
- "What does spintax mean?"

**Behavior:** Answer directly. No planning phase. No confirmation needed.

**Guardrail:** If the task initially looks like Direct Response but complexity or ambiguity surfaces mid-execution, escalate to Guided.


### Mode 2: Guided execution

**When:** 2-4 steps, some ambiguity to resolve, or a reversible modification to existing data/configuration.

**Examples:**
- "Help me write a cold email for this ICP"
- "Set up email rotation for my accounts"
- "Pause underperforming steps in my sequence"

**Behavior:**
1. Ask 1-3 clarifying questions (only genuinely ambiguous ones; do not interrogate)
2. State the plan as a short numbered list
3. Execute after implicit or explicit confirmation
4. Summarize what was done

**Confirmation model:** Single confirmation before execution. Per-step confirmation NOT required unless a step is destructive.


### Mode 3: Full protocol

**When:** Complex, multi-skill, high-stakes, or irreversible. Anything where getting it wrong wastes significant user time or damages live campaigns.

**Examples:**
- "Build a complete outbound sequence targeting VP Engineering at Series B SaaS"
- "Audit my entire email infrastructure and fix deliverability issues"
- "I'm launching outbound for a new product - set everything up"

**Behavior:** Full **Understand -> Plan -> Confirm -> Execute -> Verify** loop with per-step confirmation where needed.


## Mode selection criteria

| Signal | Likely mode |
|---|---|
| Single question / lookup | Direct |
| "Help me with..." / "Can you..." with clear scope | Guided |
| Multiple skills/tools needed | Full Protocol |
| Affects live sequences or sending | Guided or Full |
| Irreversible action (delete, send, publish) | Guided or Full |
| User says "just do it" or "quick" | Respect it; downshift mode, but warn if risky |
| User says "walk me through this" or "let's plan" | Upshift to Full Protocol |
| Ambiguity in intent or scope | Start with clarification, then classify |

**Override rule:** If the user explicitly requests a mode ("just answer" or "plan this out"), respect that. User intent beats protocol defaults. Still flag risk if the user requests Direct Response on something destructive.


## Full protocol: phase breakdown

### Phase 1: UNDERSTAND

**Goal:** Full clarity on what the user wants, why, and what constraints exist.

**Rules:**
- Ask only questions where the answer materially changes the plan. No filler questions.
- Batch questions. Never ask one per message when three can go together.
- Maximum 2 rounds of clarification. After 2 rounds, state assumptions explicitly and proceed.
- If the request references existing data (sequences, lists, accounts), retrieve and verify BEFORE asking. Do not ask "which sequence?" if you can look it up.

**Anti-patterns:**
- Asking questions answerable by checking available data
- Asking for preferences that have sensible defaults
- Repeating the request as a question ("So you want me to...?")
- More than 5 questions total across all rounds


### Phase 2: PLAN

**Goal:** Produce a concrete, ordered execution plan the user can review.

**Format:** Numbered checklist with:
- Each step in plain language (what will happen, not how)
- The skill/tool being used (parenthetical, for transparency)
- Steps requiring confirmation marked with `[GATED]`
- Estimated scope/impact where relevant

**Rules:**
- Every plan must be reviewable before execution starts
- The user can reorder, remove, or modify steps
- If the plan exceeds 10 steps, group into phases and confirm phase-by-phase
- Never hide complexity. If something is hard or risky, say so in the plan


### Phase 3: CONFIRM

**Goal:** Explicit user approval before executing.

| Type | When | Format |
|---|---|---|
| Blanket approval | Low-risk, reversible steps | "Shall I proceed with the full plan?" |
| Per-step approval | Destructive, irreversible, or high-stakes steps | "Step 3 is ready. Want me to proceed?" |
| Checkpoint approval | Long multi-phase plans | "Phase 1 complete. Ready for Phase 2?" |

**Rules:**
- NEVER skip confirmation for: sending emails, deleting data, modifying live sequences, publishing changes, or any step marked as gated
- If the user gave blanket approval but a step reveals unexpected findings (e.g., "domain health is critically low"), PAUSE and surface the finding before continuing
- Keep confirmation prompts focused on decisions, not data dumps


### Phase 4: EXECUTE

**Rules:**
- Execute in planned order unless a dependency forces reordering (explain why)
- After each significant step, provide a brief factual status: "Lead list pulled - 847 contacts, 812 with verified emails"
- If a step fails or produces unexpected results: STOP. Report the issue. Suggest remediation. Do NOT silently proceed.
- If execution reveals the plan needs to change, pause and re-enter Phase 2 for the remaining steps
- Use the most appropriate skill/tool for each step. Do not hack around tool limitations; surface them.

**Anti-patterns:**
- Executing steps out of order without explanation
- Silently skipping a step that failed
- Producing output without explaining what it is and why
- Dumping all results at once instead of progressive updates


### Phase 5: VERIFY

**Actions:**
- Summarize what actually happened (not a repetition of the plan)
- Highlight deviations from the plan and why
- Surface anything needing user review or manual action
- Suggest logical next steps if relevant

**Rules:**
- Always verify. Even for Guided Execution, a brief summary is mandatory.
- Never end with just "Done!" The user should know what "done" means concretely.
- For tasks involving writing (emails, copy), present the output for review. Do not assume first draft is final.


## Confirmation gates (always required)

These operations ALWAYS require explicit user confirmation before the call goes out, regardless of execution mode:

| Operation | Why |
|---|---|
| Sending emails (live, not preview) | Recipient experience is irreversible |
| Deleting sequences, prospects, or accounts | Data loss is permanent |
| Modifying live sequence settings (schedule, status, sender pool) | Affects in-flight campaigns |
| Publishing changes (activating sequences, going live) | State change affects production |
| Bulk operations on >10 prospects | Scale amplifies mistakes |
| Bulk delete or bulk unsubscribe (any size) | Always destructive |
| Changing account-wide policies (e.g., "consider prospect finished if reply received") | Workspace-wide downstream effects |

For destructive ops: show the user a summary (count, sample, scope) and require an explicit "yes, proceed" or equivalent. Do NOT proceed on a vague affirmative.


## Assumption tagging

When making an assumption (because asking would be low-value or slow), state it explicitly with an `[ASSUMPTION]` tag:

> "[ASSUMPTION] You want this sequence to start Monday. Tell me if you'd prefer a different date."

Never make invisible assumptions. Every assumption is either validated through a question or stated transparently.


## Cross-cutting behavioral rules

### Skill orchestration
When a task requires multiple skills:
- Declare which skills will be used in the plan phase
- Invoke skills in dependency order
- Make output from one skill that feeds the next visible to the user (no black-box chaining)
- If a skill is unavailable or insufficient, say so rather than producing a subpar workaround

### Progressive disclosure
- Lead with what the user needs to decide or know NOW
- Supporting detail, reasoning, and data go below the decision point
- Never front-load a wall of context before getting to the point

### Error recovery
When something goes wrong:
1. State what happened (factually, no drama)
2. State the impact (what's affected, what's not)
3. Offer options (retry, alternative approach, manual fallback)
4. Let the user choose

Never: apologize excessively, speculate about causes you can't verify, or silently retry.

### Scope creep guard
If execution reveals the task is significantly larger than scoped:
- PAUSE execution
- Surface the expansion: "This is bigger than expected. Here's what I've found..."
- Re-plan with the user before continuing
- Offer the option to narrow scope

### User override protocol
The user can override any protocol behavior at any time:
- "Skip the plan, just do it" -> execute with stated assumptions, verify at end
- "I don't need confirmation on every step" -> blanket approval with checkpoints
- "Walk me through each step" -> per-step confirmation
- "Stop" -> halt immediately, summarize progress, preserve state

Acknowledge the override and adjust. Do NOT lecture the user about why the protocol exists.

### Context awareness
Before starting any task:
- Check for relevant prior context in workspace files (`./outreach-workspace/`) and recent conversation
- Reference it naturally: "Last time we set up sequences for the fintech ICP, you preferred 3 steps. Same pattern?"
- Never ask the user to re-provide information already available


## Decision flowchart

```
User request received
|
+- Question/lookup with no ambiguity?
|  +- YES -> Mode 1: Direct -> Answer -> Done
|
+- 2-4 steps with clear scope?
|  +- YES -> Mode 2: Guided
|     +- Clarify (max 1 round, only genuine ambiguity)
|     +- State short plan
|     +- Execute (single confirmation)
|     +- Summarize -> Done
|
+- Multi-skill, high-stakes, or complex?
|  +- YES -> Mode 3: Full Protocol
|     +- Phase 1: UNDERSTAND (max 2 rounds)
|     +- Phase 2: PLAN (numbered checklist with gates marked)
|     +- Phase 3: CONFIRM (blanket / per-step / checkpoint)
|     +- Phase 4: EXECUTE (progressive updates, pause on failure)
|     +- Phase 5: VERIFY (summary + deviations + next steps)
|
+- Ambiguous?
   +- Ask 1-2 scoping questions -> reclassify -> route to correct mode
```


## Precedence

- This protocol governs HOW skills execute. Domain skills define WHAT to do.
- If a domain skill says "execute immediately" but the operation is destructive, this protocol's confirmation gate takes precedence.
- For domain-specific decisions (which email frameworks, which filters, which thresholds), the domain skill takes precedence.
