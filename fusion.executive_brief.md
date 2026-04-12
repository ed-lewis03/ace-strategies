# Strategy: fusion.executive_brief

**Agent source:** fusion
**Result type:** risk_brief

## Synthesis instructions for ACE

When a `fusion` row with `result_type=risk_brief` is present, treat it as the authoritative executive-risk view for the subject.

- Lead with the EDRI score and tier from the row's top-level fields.
- Quote the `summary` field verbatim for the one-line headline.
- Drill into `payload.kpis` only when the user asks about vendor specifics (netcraft / vanishid / zignal).
- If `payload.hitl_triggered == true`, state "Under analyst review" and surface `payload.hitl_execution_id`.
- The brief PDF is in `evidence_urls[0]` — link it if the user asks for the full report.
- Do NOT re-derive EDRI from KPIs; trust the stored `edri_score`.
