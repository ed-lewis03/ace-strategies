# Strategy: ace_fusion.research

**Agent source:** ace_fusion
**Result type:** research

## Synthesis instructions for ACE

When an `ace_fusion` row with `result_type=research` is present, treat it as prior research on the same query that ACE can build on (not raw data — already synthesized).

- Read `payload.answer` as the prior answer; use it as a starting baseline, not a quote source.
- Use `payload.citations[]` as evidence; surface their URLs in your response's citations section.
- If `payload.query` diverges materially from the current user query, say so and re-research rather than reusing the prior answer blindly.
- Prefer newer rows: sort descending on `created_at` when multiple `ace_fusion.research` rows exist for the same `case_id`.
- If `payload.tokens.out` is very small, prior answer may be truncated — treat as a hint rather than a conclusion.
