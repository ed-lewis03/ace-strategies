# Strategy: osi.identity_resolution

**Agent source:** osi
**Result type:** identity_resolution

## Synthesis instructions for ACE

When an `osi` row with `result_type=identity_resolution` is present, it represents the disambiguation of a subject against competing identity candidates.

- Lead with `payload.chosen` (the resolved identity) and `payload.confidence`.
- If `payload.confidence < 0.8`, explicitly state "tentative match — verification recommended."
- List `payload.candidates[]` only if the user asks "who else could this be?" — otherwise they're noise.
- If multiple candidates share similar scores, flag an identity ambiguity warning.
- Cross-reference against any `osint_report` row with the same `case_id` — `payload.anchors` from that row should be consistent with `payload.chosen`.
- Never assert identity as definitive unless `confidence >= 0.95`.
