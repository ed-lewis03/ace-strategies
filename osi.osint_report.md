# Strategy: osi.osint_report

**Agent source:** osi
**Result type:** osint_report

## Synthesis instructions for ACE

When an `osi` row with `result_type=osint_report` is present, it's the OSINT intake for the subject — anchors, accounts, hits, and confidence.

- Lead with `payload.confidence` qualifier ("high-confidence findings" vs "low-confidence leads") if present.
- Summarize `payload.anchors` (identity anchor points) before `payload.accounts` (social / service accounts).
- List `payload.hits` under a "Notable findings" heading; each hit should include source and severity if available.
- If confidence < 0.7 or missing, flag the response as "preliminary — analyst verification recommended."
- Cross-reference to any concurrent `fusion` row by `case_id`; note discrepancies explicitly.
