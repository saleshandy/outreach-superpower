# Industry tone reference

Industry-specific conventions for cold outreach. Use this when the ICP sits in one of the industries below to choose the right opener angle, proof format, and CTA shape. Sourced from PB-02 (cold email playbook).

These are starting points, not rules. If the prospect's behavior contradicts the table (e.g., a SaaS buyer who clearly responds to scarcity), follow the signal.


## Industry-specific best practices

| Industry | Lead With | Proof Style | CTA Style |
|---|---|---|---|
| SaaS | Operational pain (not feature lists) | Specific metrics: "cut [process] time by X%" | Demo or walkthrough |
| Lead Gen Agencies | Numbers | "Generated [X] leads in [timeframe]" | "See if this could work for you" |
| Digital Marketing | A quick audit or tip (value first) | Specific issue on their site | "Want me to share what I found?" |
| Recruitment / HR | Growth and culture | Conversation, not a pitch | Low-pressure, opportunity-focused |
| PR / Media | Current news or trends | Expert positioning | Keep it SHORT - journalists are busy |
| E-commerce / Retail | Seasonal hooks | Scarcity acceptable here | Personalize by purchase history |


## How to use this

- **In `email-sequence-generator` (Phase 5: Psychology-driven copy):** after picking the ICP segment, check whether it falls into one of these industries. If yes, bias the opener pattern toward the "Lead With" column, the body proof toward the "Proof Style" column, and the CTA toward the "CTA Style" column. If the ICP straddles two industries (e.g., a SaaS-for-agencies product), default to the buyer-side industry, not the seller-side.
- **In `email-auditor`:** when the pasted copy targets an industry above, use the table to flag mismatches. Example: a SaaS-targeted email that leads with seasonal hooks instead of operational pain is a low Tone-match score even if the copy reads well.
- **In `icp-builder`:** when capturing segments, note which row applies. Persist as a hint in `icp.md` so downstream skills don't re-derive it.

If the ICP industry isn't on this list, fall back to the general PB-02 rules (specificity in opener, named proof in body, binary CTA).
