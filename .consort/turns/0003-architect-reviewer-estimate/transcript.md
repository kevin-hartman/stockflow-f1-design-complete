# architect-reviewer-estimate (architect-reviewer) – opus

## Prompt

```
Estimate each proposed candidate feature with a t-shirt size (XS/S/M/L/XL) and write planning/estimates.json, so the Product Owner can commit a backlog that fits sprint capacity. Be terse: produce ONLY the required artifact file(s) on disk, then stop with at most a one-line confirmation. Do NOT print a plan, a summary of what you did, rationale, tables, or restate the artifacts to stdout, that output is wasted latency. The files on disk are the deliverable, not your prose.
```

## Tools used

- Read {"file_path":"<PROJECT_ROOT>/.consort/planning/feature-proposals.md","description":"Read candidate feature proposals"}
- Bash {"command":"cat .consort/planning/estimates.json 2>/dev/null || echo NONE","description":"Check for existing estimates file"}
- Write {"file_path":"<PROJECT_ROOT>/.consort/planning/estimates.json","description":"Write t-shirt size estimates for candidates","content":"{\n  \"estimates\": [\n    { \"feature_id\": \"F1-stock-visibility\", \"size\": \"L\", \"rationale\": \"Foundation: first schema + migration (unique sku,location upsert), full layered API, SPA shell + 3 UI stories with E2E against real branch.\" },\n    { \"feature_id\": \"F2-stock-adjustment\", \"size\": \"M\", \"rationale\": \"In-place correction plus audit table (timestamp + typed actor) and non-negative check; one form + API.\" },\n    { \"feature_id\": \"F3-inbound-receipt\", \"size\": \"M\", \"rationale\": \"Supplier reference data + receive write path into existing stock rows with inline validation.\" },\n    { \"feature_id\": \"F4-outbound-pick\", \"size\": \"M\", \"rationale\": \"Pick write path with transactional overcommit rejection and field-named error round-trip.\" },\n    { \"feature_id\": \"F5-cycle-count\", \"size\": \"M\", \"rationale\": \"Count capture, variance computation, and reconcile write; moderate UI flow.\" },\n    { \"feature_id\": \"F6-split-tracking-code-expand\", \"size\": \"L\", \"rationale\": \"Additive expand migration with delimiter backfill, dual-write, integrity probe surfaced in app; data-preservation risk.\" },\n    { \"feature_id\": \"F9-stock-search\", \"size\": \"S\", \"rationale\": \"Read-only filter over existing table with index access pattern and no-match empty state.\" },\n    { \"feature_id\": \"F7-multi-warehouse\", \"size\": \"XL\", \"rationale\": \"Re-keys stock scope across every table and endpoint with default-warehouse backfill migration.\" },\n    { \"feature_id\": \"F8-barcode-scan\", \"size\": \"L\", \"rationale\": \"Scan-driven input across receive/pick/adjust/count with 200ms feedback budget.\" },\n    { \"feature_id\": \"F10-retire-tracking-code\", \"size\": \"S\", \"rationale\": \"Contract migration dropping combined code with reversible round-trip test; gated on F6.\" }\n  ]\n}\n"}

## Final reasoning

I wrote `.consort/planning/estimates.json` with sizes for F1-stock-visibility (L), the only sprint-1 candidate, and for the nine deferred roadmap features (one XL, two L, four M, three S).
