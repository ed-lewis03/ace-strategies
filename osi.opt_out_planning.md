# Strategy: osi.opt_out_planning

**Agent source:** osi
**Result type:** opt_out_planning

## Synthesis instructions for ACE

When an `osi` row with `result_type=opt_out_planning` is present, it represents a plan to suppress or remove the subject's data from brokers and data aggregators.

- Lead with `payload.priority` (e.g. `urgent`, `standard`, `maintenance`).
- List `payload.brokers[]` grouped by priority tier, not alphabetically — operator runs top tier first.
- Render `payload.plan[]` as an ordered checklist with per-step action owners if available.
- Surface expected latency per broker if the payload includes it; otherwise say "broker-dependent."
- If a concurrent `physical_safety` row exists with severity `high`/`critical`, escalate this plan's effective priority to at least `urgent` in the response.
- Note that opt-out is ongoing — recommend re-scan cadence (default: quarterly) if no cadence is in the payload.
