# Outreach Superpower

A Claude Code plugin that turns a website URL into a paste-ready cold email sequence.

## What is it

Cold outreach is slow because the inputs are scattered. You have a website, a vague sense of who to target, maybe a few past wins, and a blank Saleshandy campaign waiting to be filled. Outreach Superpower is a set of six Claude Code skills that walks you from URL to ICP to a 4-email sequence in one session, persisting every artifact to a local workspace folder so each step builds on the last.

The plugin ships six skills: a router that auto-loads on outreach intent, a strategy architect that extracts your value prop and segments from your website, an ICP builder that runs a 14-step interview, a service productizer that turns an agency offering into a packaged offer, an email sequence generator that drafts 3 to 7 emails with Saleshandy merge tags and spintext, and an email auditor that scores existing copy across 7 dimensions. All outputs go to markdown files you can edit, version, and reuse.

## Install

The plugin is not on the official Claude Code marketplace yet. Install via the self-hosted GitHub source:

```bash
claude marketplace add github:saleshandy/outreach-superpower
claude plugin install outreach-superpower
```

Once installed, the router skill auto-loads whenever you mention cold email, outreach, ICP, or sales sequences in any Claude Code session.

## Quickstart

Cold start, 5 minutes from URL to sequence:

1. Open a fresh Claude Code session in any working directory.
2. Type: *"Help me write cold emails for https://acme.com"*
3. The router announces it loaded, sees no workspace, and routes to `strategy-architect`. It fetches your homepage, pricing, and customers pages, then shows a 10-item confirmation card (value prop, ICP segments, personas, proof, differentiators, etc.). Edit any item or reply *"all good"*.
4. On confirm, three files land in `outreach-workspace/default/`: `company.md`, `strategy.md`, and a lite `icp.md`. The skill suggests running `icp-builder` next.
5. Type: *"Build the ICP."* The interview starts. With `confidence: high` from the website extract, the skill pre-fills 7 of the 14 questions; you confirm or edit each. The full ICP (summary, 3 segment cards, targeting filters, 5 angles, disqualifiers) writes to `icp.md`.
6. Type: *"Generate a 4-email demo sequence."* The sequence generator reads `icp.md` and `company.md`, asks for goal (book demo) and segment (primary), and writes a 4-email sequence with merge tags and spintext to `sequence.md`.
7. Open `sequence.md`, copy each email, and paste into a new Saleshandy sequence at `https://my.saleshandy.com/sequence`.

## The 6 skills

| Skill | When to use |
|---|---|
| `using-outreach-superpower` | Auto-loads on outreach intent. You don't invoke it directly. |
| `strategy-architect` | New project, just have a website URL. |
| `icp-builder` | Build or tighten your Ideal Customer Profile via the 14-step interview. |
| `service-productizer` | Run a service business; turn a service into a productized offer. |
| `email-sequence-generator` | Generate cold email sequences for Saleshandy (3 to 7 emails, merge tags + spintext). |
| `email-auditor` | Score and improve existing cold email copy across 7 dimensions. |

See `docs/USAGE.md` for full per-skill examples.

## Workspace

Every skill reads from and writes to a single workspace folder. Defaults:

- **Per-project (preferred):** `./outreach-workspace/<campaign>/` in your current working directory. `<campaign>` defaults to `default`.
- **Global fallback:** `~/.outreach-superpower/workspace/default/` if you're in your home directory or a folder with no obvious project context.
- **Custom campaign:** add `--campaign acme` (or `acme-q2`, `client-name`, etc.) to scope outputs to a named folder. Useful when you run multiple campaigns side by side.

Files inside a workspace:

```
outreach-workspace/<campaign>/
  company.md             written by strategy-architect
  strategy.md            written by strategy-architect
  icp.md                 written by icp-builder (or strategy-architect lite)
  productized-offer.md   written by service-productizer
  sequence.md            written by email-sequence-generator
  audits/
    YYYY-MM-DD-<slug>.md written by email-auditor (one file per audit)
```

Workspace files stay local. Re-running any skill picks up where the last one left off.

## Privacy and data

Workspace files are plain markdown on your filesystem. The skills do not transmit them anywhere; only Claude reads them within a session, the same way it reads any other file you open. `WebFetch` (used by `strategy-architect`) only fetches public URLs you explicitly provide. No analytics, no telemetry, no cloud sync.

## Documentation

- Per-skill examples and transcripts: [`docs/USAGE.md`](docs/USAGE.md)
- Contributing, extending, and adding new skills: [`docs/DEVELOPING.md`](docs/DEVELOPING.md)

## License

MIT. See [LICENSE](LICENSE).

## Built by

[Saleshandy](https://saleshandy.com) (Ikigai Infotech). Saleshandy is a cold email outreach platform built for SDRs and founders who want to ship pipeline, not fight tools.
