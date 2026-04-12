# Strategy: osi.physical_safety

**Agent source:** osi
**Result type:** physical_safety

## Synthesis instructions for ACE

When an `osi` row with `result_type=physical_safety` is present, it captures physical-safety risk findings (location exposure, doxxing, direct threats, travel/residence risk).

- Lead with `payload.severity` (expected values: `critical` | `high` | `medium` | `low` | `informational`).
- If severity is `critical` or `high`, preface the response with an explicit action-required framing.
- Quote the most severe item from `payload.findings[]` verbatim before summarizing.
- Always surface `payload.recommendations[]` — these are operator actions, not advisory.
- If any finding references a location or travel pattern, do NOT repeat the raw location in the headline; reference it as "a specific location" and defer detail to the drill-down.
- If a concurrent `reputational` or `financial_regulatory` row has overlapping findings, note the correlation.
