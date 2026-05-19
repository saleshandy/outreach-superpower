# Outreach Superpower

A Claude Code plugin that turns a URL into a paste-ready cold email sequence, and helps you run the rest of the outreach workflow: find leads, enrich, audit, triage, analyze, manage.

## What is it

Cold outreach is slow because the inputs are scattered: a website, a vague sense of who to target, a few past wins, a blank Saleshandy campaign, and an inbox full of replies you haven't classified. Outreach Superpower is a set of 12 Claude Code skills that walks you from URL to ICP to a multichannel sequence, then keeps going: it finds prospects, enriches them, triages replies, audits deliverability, interprets analytics, and runs CRM operations against Saleshandy.

Every artifact persists to a local workspace folder so each step builds on the last. Destructive Saleshandy operations are gated behind explicit confirmation.

## Install

Install the Saleshandy plugin first (provides the MCP tools that `lead-finder` and `prospect-enrichment` delegate to):

```
claude plugin marketplace add github:saleshandy/saleshandy-plugin
claude plugin install saleshandy
```

Then install Outreach Superpower:

```
claude plugin marketplace add github:saleshandy/outreach-superpower
claude plugin install outreach-superpower
```

The router skill auto-loads whenever you mention cold email, outreach, ICP, sequences, replies, deliverability, analytics, or sales operations.

## Quickstart

### Pre-launch: URL to sequence in one session

1. Open a fresh Claude Code session in any working directory.
2. Type: *"Help me write cold emails for https://acme.com"*
3. The router routes to `strategy-architect`. It fetches your homepage, pricing, and customers pages, then shows a 10-item confirmation card. Edit any item or reply *"all good"*.
4. On confirm, three files land in `outreach-workspace/default/`: `company.md`, `strategy.md`, and a lite `icp.md`.
5. Type: *"Build the ICP."* The 14-step interview runs, pre-filling from the website extract. Step 15 emits a search-ready filter block that `lead-finder` consumes later.
6. Type: *"Generate a 4-email demo sequence."* The sequence generator writes a multichannel sequence (email plus optional LinkedIn, call, WhatsApp) with merge tags, spintext, and a psychology layer to `sequence.md`.
7. Type: *"Find 50 leads matching this ICP."* Routes to `lead-finder`, which uses the filter block to query Saleshandy Lead Finder.
8. Paste your sequence into Saleshandy and enroll the leads.

### Post-launch: two weeks in, what to look at

1. Type: *"I've been sending for 2 weeks, what should I look at?"*
2. The router checks workspace state and routes through the post-launch chain:
   - `inbox-triage` classifies recent replies into 8 categories and recommends gated reply actions
   - `analytics-interpreter` reads sequence stats, flags anomalies, and proposes A/B tests
   - `operations-audit` runs a 5-phase audit (deliverability, sending volume, sequence health, list hygiene, account health) with a prioritized fix list
3. Each skill writes its output to the workspace so you can act on it across sessions.

## The 12 skills

Sorted by lifecycle stage.

| Skill | Stage | When to use |
|---|---|---|
| `using-outreach-superpower` | router | Auto-loads on outreach intent. You don't invoke it directly. |
| `strategy-architect` | pre-launch | New project, just have a website URL. Extracts value prop, segments, personas, proof. |
| `icp-builder` | pre-launch | Build or tighten ICP via 14-step interview. Step 15 emits a search-ready filter block. |
| `service-productizer` | pre-launch | Run a service business; turn a service into a packaged offer. |
| `email-sequence-generator` | pre-launch | 3-7 step multichannel sequence with Saleshandy merge tags, spintext, and psychology layer. |
| `email-auditor` | pre-launch | Score existing copy across 11 dimensions (7 structural plus 4 craft) with deliverability red flags. |
| `lead-finder` | launch prep | ICP filter block to prospect list via `saleshandy:discover-and-enroll`. |
| `prospect-enrichment` | launch prep | Generate AI columns for personalization (custom snippets, opener hooks, qualification). |
| `inbox-triage` | post-launch | Classify replies into 8 categories (positive, objection, OOO, referral, unsubscribe, negative, neutral, other) with gated reply suggestions. |
| `operations-audit` | post-launch | 5-phase audit: deliverability, sending volume, sequence health, list hygiene, account health. |
| `analytics-interpreter` | post-launch | Anomaly detection on sequence stats, hypotheses, A/B test suggestions. |
| `crm-operations` | post-launch | Prospect, task, and sequence management with strict per-message and per-batch confirmation gates. |

See `docs/USAGE.md` for full per-skill examples.

## References

Five shared reference docs in `references/` are loaded on demand by the skills that need them:

| Doc | Loaded by |
|---|---|
| `execution-protocol.md` | router (3-mode execution: guided, autonomous, batch) |
| `inbox-radar.md` | `email-auditor`, `operations-audit` (deliverability red flag patterns) |
| `email-account-health.md` | `operations-audit` (sender reputation, warmup, bounce thresholds) |
| `settings.md` | `crm-operations`, `operations-audit` (Saleshandy account settings reference) |
| `industry-tone.md` | `email-sequence-generator`, `email-auditor` (tone calibration by industry) |

## Workspace

Every skill reads from and writes to a single workspace folder. Defaults:

- **Per-project (preferred):** `./outreach-workspace/<campaign>/` in your current working directory. `<campaign>` defaults to `default`.
- **Global fallback:** `~/.outreach-superpower/workspace/default/` if you're in your home directory or a folder with no obvious project context.
- **Custom campaign:** add `--campaign acme` to scope outputs to a named folder.

Files inside a workspace:

```
outreach-workspace/<campaign>/
  company.md                          written by strategy-architect
  strategy.md                         written by strategy-architect
  icp.md                              written by icp-builder (or strategy-architect lite)
  productized-offer.md                written by service-productizer
  sequence.md                         written by email-sequence-generator
  leads.md                            written by lead-finder
  enrichment-plan.md                  written by prospect-enrichment
  inbox-log.md                        written by inbox-triage
  audit-YYYY-MM-DD.md                 written by operations-audit
  analytics-snapshot-YYYY-MM-DD.md    written by analytics-interpreter
  crm-actions.md                      written by crm-operations (action log with confirmations)
  audits/
    YYYY-MM-DD-<slug>.md              written by email-auditor (one file per copy audit)
```

Workspace files stay local. Re-running any skill picks up where the last one left off.

## Privacy and data

Workspace files are plain markdown on your filesystem. The skills do not transmit them anywhere; only Claude reads them within a session. `WebFetch` (used by `strategy-architect`) only fetches public URLs you explicitly provide. Saleshandy MCP operations only run when you invoke a skill that calls them, and all destructive operations require explicit confirmation. No analytics, no telemetry, no cloud sync.

## Documentation

- Per-skill examples and transcripts: [`docs/USAGE.md`](docs/USAGE.md)
- Contributing, extending, and adding new skills: [`docs/DEVELOPING.md`](docs/DEVELOPING.md)

## License

MIT. See [LICENSE](LICENSE).

## Built by

[Saleshandy](https://saleshandy.com) (Ikigai Infotech). Saleshandy is a cold email outreach platform built for SDRs and founders who want to ship pipeline, not fight tools.
