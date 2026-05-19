---
name: using-outreach-superpower
description: Use when the user mentions cold email, outreach, prospecting, ICP, target audience, sales sequences, email copy review, productizing a service, or any SDR/AE workflow. Loads first to orient routing across the outreach-superpower suite.
---

# Outreach Superpower - Router

You are operating in the outreach-superpower plugin. Before responding, identify the user's outreach intent, locate any existing workspace artifacts, and route to the precise sub-skill. Auto-chain prerequisites.

## Execution mode

For mode selection (Direct / Guided / Full Protocol), see `references/execution-protocol.md`.

Quick rule:

- **Direct mode** - one-shot, unambiguous intent with no destructive consequences (e.g., "what does spintax mean?", a single audit of pasted copy). Route, answer, done.
- **Guided mode** - 2-4 steps, some ambiguity, or reversible modification (e.g., "help me draft a sequence for this ICP"). Clarify minimally, state the plan, execute on confirmation.
- **Full Protocol** - multi-skill, high-stakes, irreversible, or live-data-affecting (e.g., "launch this sequence to 500 prospects," bulk delete, account-wide policy changes). Run the full Understand -> Plan -> Confirm -> Execute -> Verify loop. Per-step confirmation for destructive ops.

When in doubt, upshift one level. Never downshift on operations marked as confirmation gates in `references/execution-protocol.md`.

## Workspace location

Default: `./outreach-workspace/<campaign>/` in the current working directory. `<campaign>` defaults to `default`.

If the current dir is the user's home (`~`) or has no obvious project context, fall back to `~/.outreach-superpower/workspace/default/`.

If user passes `--campaign <name>` in their request, use that as the campaign folder.

## Announce on load

On first invocation in a session, output one line:

> *Outreach Superpower loaded. Routing to `<skill>`. Workspace: `<path>`. Existing artifacts: <comma-separated filenames or "none">.*

## Routing decision table

| User intent / phrase | Skill |
|---|---|
| "build/define ICP", "who should I target", "who is my ICP", "tighten my ICP", "who should I email" | icp-builder |
| "outreach strategy", "audit my website for outreach", "extract from my site", "build a campaign from URL" | strategy-architect |
| "productize my service", "package my agency offering", "turn my service into a product" | service-productizer |
| "write cold email sequence", "draft a cadence", "X-email campaign", "follow-up sequence", "outreach emails" | email-sequence-generator |
| "audit/review/rate this email", "score my email", "is this email good", or pasted email body | email-auditor |
| "find prospects", "build target list", "search companies", "search people", "source leads" | lead-finder |
| "enrich", "add AI column", "personalize at scale", "research prospects", "fill in missing data" | prospect-enrichment |
| "review replies", "triage inbox", "categorize replies", "what should I respond to" | inbox-triage |
| "audit setup", "check deliverability", "diagnose sending health", "is my domain healthy" | operations-audit |
| "explain metrics", "what's working", "diagnose drop", "why did opens fall", "interpret stats" | analytics-interpreter |
| "manage prospects", "bulk update", "move CRM", "pause prospects", "tag leads" | crm-operations |

*Note: "audit" alone is ambiguous - if the user pastes email copy, route to `email-auditor`; if they reference a website URL or company, route to `strategy-architect`; if they mention deliverability, sending health, or domains, route to `operations-audit`; if neither signal, ask which they meant.*

## Phase awareness

When the user's intent is ambiguous, check workspace state before routing. The workspace phase biases the route:

| Workspace state | Likely phase | Biased toward |
|---|---|---|
| Only `company.md` / `strategy.md` exist (no `icp.md`) | Pre-targeting | `icp-builder` |
| `icp.md` exists, no `leads.md` | Targeting | `lead-finder` |
| `leads.md` exists, no enriched columns | Pre-personalization | `prospect-enrichment` |
| `sequence.md` exists in workspace | Post-launch | `inbox-triage`, `analytics-interpreter`, `operations-audit`, `crm-operations` |

Examples:

- User says "audit it" with a `sequence.md` present but no pasted copy -> bias toward `operations-audit` (sending-side audit), not `email-auditor`.
- User says "what's next" with `icp.md` + `leads.md` present -> suggest `email-sequence-generator` or `prospect-enrichment`.

Always announce the phase bias before routing:

> *Workspace state: `icp.md` + `leads.md` present. Routing to `prospect-enrichment` (pre-personalization phase).*

## Auto-chaining rules

Before invoking a skill, check its prerequisites against the workspace. If a required file is missing, invoke the producing skill first, then resume.

| Skill | Required prereq | Helpful prereq |
|---|---|---|
| strategy-architect | (none - entry point) | - |
| icp-builder | - | company.md |
| service-productizer | company.md | - |
| email-sequence-generator | icp.md | company.md |
| email-auditor | (paste) | icp.md |
| lead-finder | icp.md (with Search-ready filters block) | company.md |
| prospect-enrichment | leads.md | icp.md |
| inbox-triage | (live Saleshandy data) | sequence.md |
| operations-audit | (live Saleshandy data) | - |
| analytics-interpreter | (live Saleshandy data) | sequence.md |
| crm-operations | (live Saleshandy data) | - |

**Prerequisite chaining for the new skills:**

- `lead-finder` needs `icp.md` with a `## Search-ready filters` section. If missing, route to `icp-builder` first.
- `prospect-enrichment` needs `leads.md`. If missing, route to `lead-finder` first (which itself may route to `icp-builder`).
- `inbox-triage`, `operations-audit`, `analytics-interpreter`, `crm-operations` operate on live Saleshandy data via MCP, not workspace files. No workspace prerequisite. Invoke directly.

When auto-chaining, announce it:

> *No `icp.md` found. Running `icp-builder` first, then resuming `email-sequence-generator`.*

> *No `leads.md` found. Running `lead-finder` first, then resuming `prospect-enrichment`.*

### Downstream chains (suggested next skill)

Some skills end by offering to invoke the next skill in a typical workflow:

| Skill | Suggests next |
|---|---|
| strategy-architect | icp-builder OR email-sequence-generator |
| icp-builder | lead-finder OR email-sequence-generator |
| service-productizer | email-sequence-generator |
| lead-finder | prospect-enrichment |
| prospect-enrichment | email-sequence-generator |
| email-sequence-generator | email-auditor (audit the draft) |
| email-auditor | (none - terminal) |
| inbox-triage | crm-operations (act on triaged replies) |
| operations-audit | (none - terminal; user fixes setup) |
| analytics-interpreter | email-auditor OR email-sequence-generator (fix the weak step) |
| crm-operations | (none - terminal) |

When a sub-skill ends, it should output a one-liner like: *"Recommended next: run `email-sequence-generator` to draft a sequence."* The user accepts or declines.

## The 4 rules every sub-skill obeys

1. **Workspace-first.** Read existing files before asking the user.
2. **Persist outputs.** Every skill writes a workspace file, not just chat.
3. **Cite the source.** When using data from another file, cite it (e.g., *"Pulling segment 'mid-market SaaS founders' from `icp.md`"*).
4. **Label assumptions.** Tag inferred data with `[ASSUMPTION]` so the user can challenge.

## Decision flow

```dot
digraph route {
    "user request" -> "audit (pasted email)?" [label="copy in message"];
    "audit (pasted email)?" -> "email-auditor" [label="yes"];
    "audit (pasted email)?" -> "intent type?" [label="no"];
    "intent type?" -> "strategy-architect" [label="strategy/website"];
    "intent type?" -> "icp-builder" [label="ICP"];
    "intent type?" -> "service-productizer" [label="service/agency"];
    "intent type?" -> "needs ICP?" [label="sequence"];
    "needs ICP?" -> "icp-builder + chain" [label="no icp.md"];
    "needs ICP?" -> "email-sequence-generator" [label="icp.md exists"];
}
```
