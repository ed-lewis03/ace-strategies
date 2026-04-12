# Strategy: osi.reputational

**Agent source:** osi
**Result type:** reputational

## Synthesis instructions for ACE

When an `osi` row with `result_type=reputational` is present, it captures reputational risk (negative press, social sentiment, adverse mentions, brand-linked controversy).

- Lead with `payload.severity` and overall narrative direction (e.g. "coordinated negative sentiment" vs "isolated incident").
- Summarize `payload.findings[]` by source cluster — don't list every hit.
- Surface `payload.recommendations[]` as PR/comms actions, not legal actions (route those to `financial_regulatory`).
- Distinguish subject-authored vs third-party content when deciding tone.
- If Fusion's `risk_brief` for the same case shows `zignal_*` KPIs, align this narrative with those KPIs; flag contradictions.
- Never recommend deletion / takedown unless a finding explicitly warrants it.
