# Strategy: osi.financial_regulatory

**Agent source:** osi
**Result type:** financial_regulatory

## Synthesis instructions for ACE

When an `osi` row with `result_type=financial_regulatory` is present, it captures regulatory / financial-exposure findings (sanctions hits, adverse media, litigation, corporate filings, PEP status, AML flags).

- Lead with `payload.severity` and name the regime(s) involved (OFAC, EU, UK, UN, etc.) when available.
- Any sanctions/PEP hit is lead material — do not bury it.
- Quote `payload.findings[]` with source and date; stale findings (>24 months) should be noted as "historical."
- `payload.recommendations[]` are compliance actions — render as a checklist, not prose.
- If severity is `high` or `critical`, include a "do not proceed without compliance review" banner.
- Do not render legal opinion — cite findings, let the operator's compliance team interpret.
