# architect-reviewer (architect-reviewer) – opus

## Prompt

```
Annotate story S3-sku-detail-view's acceptance criteria + nfrs.md coverage. Author architectural_notes for ALL of story S3-sku-detail-view's ACs: AC1-row-opens-sku-detail, AC2-detail-untracked-fields-show-not-tracked. Every one of this story's ACs MUST end up with a non-empty "architectural_notes" field in its acs/<AC>.json (the layer it lives in + how it realizes the design) — the design gate verifies that, and the spec-author's "layer" field does NOT count. architectural_notes are per-AC, so author the scoped ACs above even when the feature-level architecture.json already exists from an earlier story. For every E2E AC, ALSO pass consort-annotate-ac --ui-seams with the ia.md data-testid seam token(s) the AC's UI drives — the column / cell / region / empty-state / form field it asserts (e.g. "stock-cell-tracking,stock-not-tracked"). A deterministic gate checks each seam is named in <PROJECT_ROOT>/.consort/design/ia.md; if the AC needs a surface the IA does not yet name, declare the seam you need anyway and the gate routes the ux-designer to add it to ia.md. In architecture.json, make an EXPLICIT service_backed call (required): set service_backed:true if the feature persists data (a DB table/migration) or carries business logic, and then you MUST declare boundary, service, and repository layers (plus a "models" PACKAGE app/models/, one module per domain object, NOT a flat app/models.py, when it persists entities); set false ONLY for a trivial static/read-through endpoint. An Infra-layer AC or a migration/schema/storage NFR while service_backed is false hard-blocks the gate. When service_backed:true you MUST also declare architecture.json persistence_invariants[]: the DB-level guarantees the schema enforces (each with id, type one of unique|foreign_key|cascade|not_null|check|transactional|migration_reversible, table, and a one-line brief), covering unique/composite keys, foreign keys + cascade rules, NOT NULL / CHECK constraints, any transactional-atomicity boundary, and migration reversibility. The test-strategist must cover each with a real-branch test; a service_backed feature with no persistence_invariants hard-blocks the gate. When THIS story introduces a NEW read/query shape over the schema — a filter, lookup, sort, or join an AC drives (e.g. "list stock filtered by location", "look up a SKU's rows by sku") that is an INDEX candidate — declare it in architecture.json access_patterns[] (id, a one-line brief, and the table + column(s) it reads) so the DBA is pulled back in to reconsider the physical design for it; this is how a later READ story evolves the schema, since a story that adds no persistence_invariant would otherwise skip the DBA. Declare an access_pattern ONLY for a genuinely new query shape (not for every read, and not for one already covered by a prior story's pattern or a primary-key lookup). REUSE the established project architecture conventions (set by F1-stock-visibility): boundary=app/routes (react), service=app/services, repository=app/repositories, models=app/models, infrastructure=app/database.py. Declare the SAME role -> module paths in architecture.json, do NOT remap or rename an established layer; a divergent layout hard-blocks the spec gate and mismatches the inherited code. Author this story's architectural_notes FRESH from the ACs + IA on disk this turn. On a reflect-revise re-run the self-heal clears this story's prior architectural_notes BY DESIGN, so an absent/empty prior is the expected revise state — re-derive against the corrected ACs + IA. Write every artifact under the ABSOLUTE artifact root <PROJECT_ROOT>/.consort (this feature: <PROJECT_ROOT>/.consort/features/F1-stock-visibility/; this story: <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/); use that absolute path and never resolve or guess the project root yourself.CROSS-STORY CONTEXT — this story is NOT reviewed in isolation. The feature's OTHER stories' ACs, the architecture's open_decisions, and its mandated (not-null) persistence fields are below; use them for the cross-story checks (an AC contradicting a gated sibling AC; a mandated field no sibling submit AC supplies). This IS that context — do not run `consort-cross-story-context` yourself.

{
  "current_story": "S3-sku-detail-view",
  "sibling_stories": [
    {
      "story": "S1-file-stock-record",
      "acs": [
        {
          "ac_id": "AC1-file-record-via-form",
          "status": "draft",
          "layer": "E2E",
          "given": "the operator is on the stock record form",
          "when": "they enter a SKU, a location, a quantity and a tracking code and submit",
          "then": "the form confirms the record was filed without a full-page reload, and the record is stored with exactly those values for that (sku, location)",
          "architectural_notes": "E2E client<->server round-trip: the React StockRecordPage (client/src/pages) submits via client/src/api/stockRecords.ts to POST /api/stock-records and renders the inline success status from the real response without a reload (R5). Boundary app/routes/stock_records.py validates + delegates to app/services/stock_record_service.py, which persists through app/repositories/stock_record_repository.py into stock_records on the paired branch (R4)."
        },
        {
          "ac_id": "AC2-inline-required-field-errors",
          "status": "draft",
          "layer": "E2E",
          "given": "the operator is on the stock record form",
          "when": "they submit with a required field (SKU, location or quantity) left blank",
          "then": "each blank field shows an inline message naming that field next to it, no generic 'bad request' is shown, and no record is stored",
          "architectural_notes": "E2E: the field-named rejection is server-decided — input validation lives at the boundary (app/routes/stock_records.py request schema) and returns a 422 field-errors envelope; the client maps each error to its field__error next to the field via describeError, never a generic 'bad request' (R7). Cross-cutting: validation owned by the API boundary; nothing reaches the service/repository so no row is stored."
        },
        {
          "ac_id": "AC3-inline-invalid-quantity-error",
          "status": "draft",
          "layer": "E2E",
          "given": "the operator is on the stock record form",
          "when": "they submit a quantity that is not a non-negative whole number (e.g. -3 or 'abc')",
          "then": "an inline message naming the quantity field appears next to it and no record is stored",
          "architectural_notes": "E2E: a present-but-invalid quantity (negative or non-integer) is rejected by the boundary schema (non-negative int) with a 422 envelope naming 'quantity', rendered inline at stock-record-quantity-error (R7). Defence in depth: the stock_records CHECK quantity >= 0 (PI4) guarantees no negative is ever stored (R2) even if the boundary were bypassed."
        },
        {
          "ac_id": "AC4-optional-detail-omitted",
          "status": "draft",
          "layer": "API",
          "given": "the operator is on the stock record form",
          "when": "they submit a valid SKU, location and quantity with the optional tracking code (and its batch / serial detail) left empty",
          "then": "the record is accepted and stored with the optional detail absent (null), not rejected and not stored as an empty or placeholder value",
          "architectural_notes": "API write rule on POST /api/stock-records: the service (app/services/stock_record_service.py) normalizes blank/whitespace optional detail (tracking_code) to None before the repository writes, so the nullable column stores NULL, never '' or a placeholder (R9 upstream). Verified by a pytest-bdd test against the paired branch reading the stored row back."
        },
        {
          "ac_id": "AC5-same-key-updates-in-place",
          "status": "draft",
          "layer": "API",
          "given": "a stock record already exists for (sku S, location L)",
          "when": "the operator files a record for the same (S, L) again with different quantity or tracking code",
          "then": "the existing record is updated in place with the new values, exactly one record exists for (S, L), and no duplicate is created",
          "architectural_notes": "API write rule: the repository (app/repositories/stock_record_repository.py) upserts atomically with INSERT ... ON CONFLICT (sku, location) DO UPDATE inside the service-owned transaction, backed by the unique (sku, location) constraint (PI1), so a re-file updates in place and returns created=false (R3). Verified on the paired branch: exactly one row for (S, L) carrying the new values."
        }
      ]
    },
    {
      "story": "S2-stock-by-location-table",
      "acs": [
        {
          "ac_id": "AC1-table-lists-records-by-location",
          "status": "draft",
          "layer": "E2E",
          "given": "stock records exist across more than one location",
          "when": "the operator opens the home view",
          "then": "a table shows one row per record with its SKU, location, quantity and tracking code, grouped or ordered by location so that each location's stock reads together",
          "architectural_notes": "E2E client<->server read: the React home page (client/src/pages) fetches via client/src/api/stockRecords.ts on mount and renders one stock-row per record under location-group headers. Ordering is server-decided: app/routes/stock_records.py delegates to app/services/stock_record_service.py, whose repository (app/repositories/stock_record_repository.py) selects stock_records ORDER BY location, sku (AP1) on the paired branch."
        },
        {
          "ac_id": "AC2-new-record-appears-without-reload",
          "status": "draft",
          "layer": "E2E",
          "given": "the operator is in the app and the home table is loaded",
          "when": "they file a new stock record and return to the home view",
          "then": "the new row appears in the table, and the SPA never does a full-page reload",
          "architectural_notes": "E2E SPA round-trip: after POST /api/stock-records (EP1) the operator returns to / via client-side routing and the home page re-fetches GET /api/stock-records on route mount, so the new row renders from the real server list without a full-page reload (R5). No client cache owns truth; the repository read on the paired branch is the source."
        },
        {
          "ac_id": "AC3-untracked-fields-show-not-tracked",
          "status": "draft",
          "layer": "E2E",
          "given": "a record exists with some optional details (par level, batch, serial / tracking code) absent and others present",
          "when": "the operator views that record's row in the home table",
          "then": "each absent optional field shows an explicit 'not tracked', each present field shows its value (every field judged on its own), and the row is never blank or broken",
          "architectural_notes": "E2E rendering half of R9: the server returns tracking_code null (stored NULL per S1 AC4) and F1 has no batch/serial/par_level columns (OD2, OD3), so the client cell component renders a per-field not-tracked pill for each null/absent field and the value otherwise, judged field by field. Presentation logic lives in client/src/components; the boundary never substitutes placeholder text."
        },
        {
          "ac_id": "AC4-empty-state-when-no-stock",
          "status": "draft",
          "layer": "E2E",
          "given": "no stock records exist",
          "when": "the operator opens the home view",
          "then": "an explicit teaching empty state explains that no stock has been filed yet and points to the form to file one, not a blank page or an empty table grid",
          "architectural_notes": "E2E server-decided empty state: with stock_records empty on the paired branch, GET /api/stock-records returns [] and the home page renders the empty-state (with empty-state-cta to /stock/new) instead of the stock-table grid. The decision is driven by the real empty response, not a mocked fixture."
        }
      ]
    }
  ],
  "open_decisions": [
    {
      "id": "OD1-existing-key-update-vs-reject",
      "question": "Is a re-filed (sku, location) updated in place or rejected?",
      "decision_status": "resolved",
      "resolved_by_story": "S1-file-stock-record",
      "resolution": "Updated in place via atomic upsert (PI1, AC5); response 200 created=false; UI shows 'Record updated'."
    },
    {
      "id": "OD2-batch-serial-storage",
      "question": "Are batch/serial stored as separate columns in F1, or only within the combined tracking_code?",
      "decision_status": "resolved",
      "resolved_by_story": "S1-file-stock-record",
      "resolution": "F1 persists only the combined nullable tracking_code; separate batch/serial columns are F6. The IA's batch/serial inputs are not persisted in F1 and S2/S3 render batch/serial as 'not tracked'."
    },
    {
      "id": "OD3-par-level",
      "question": "Does F1 capture par level?",
      "decision_status": "resolved",
      "resolved_by_story": "S1-file-stock-record",
      "resolution": "No column in F1; par level always renders 'not tracked'."
    },
    {
      "id": "OD4-quantity-zero",
      "question": "Is quantity 0 allowed?",
      "decision_status": "resolved",
      "resolved_by_story": "S1-file-stock-record",
      "resolution": "Allowed (CHECK quantity >= 0); a zero-quantity record is still listed in S2/S3."
    }
  ],
  "required_persistence_fields": [
    {
      "invariant_id": "PI2-not-null-sku-location",
      "table": "stock_records",
      "brief": "sku and location are NOT NULL: an insert omitting either is rejected by the database."
    },
    {
      "invariant_id": "PI3-not-null-quantity",
      "table": "stock_records",
      "brief": "quantity is NOT NULL: an insert omitting quantity is rejected by the database. tracking_code is deliberately nullable (no not_null invariant) so omitted optional detail stores NULL."
    }
  ]
} Be terse: produce ONLY the required artifact file(s) on disk, then stop with at most a one-line confirmation. Do NOT print a plan, a summary of what you did, rationale, tables, or restate the artifacts to stdout, that output is wasted latency. The files on disk are the deliverable, not your prose.
```

## Tools used

- Bash {"command":"cd <PROJECT_ROOT>/.consort/features/F1-stock-visibility && cat stories/S3-sku-detail-view/acs/*.json && cat architecture.json && grep -n -i \"detail\\|sku-\" ../../design/ia.md | head -60","description":"Read S3 ACs, architecture, and IA detail seams"}
- Bash {"command":"grep -n \"stock-row\" ../../design/ia.md | head -3; grep -n \"^#\" architecture.md","description":"Check row seam and architecture.md sections"}
- Bash {"command":"python3 - <<'EOF'\nimport json\np='architecture.json'; a=json.load(open(p))\nif not any(c['id'].startswith('EP3') for c in a['api_contracts']):\n    a['api_contracts'].append({\"id\":\"EP3-get-stock-record\",\"method\":\"GET\",\"path\":\"/api/stock-records/{sku}\",\"query\":[\"location\"],\"shape\":\"object\",\"item\":\"Single record matching (sku, location) exactly: {id: int, sku: string, location: string, quantity: int, tracking_code: string|null}. No batch/serial/par_level fields in F1 (client renders them 'not tracked'). 404 body {detail: string} when no record matches; 422 when location is missing.\",\"story\":\"S3-sku-detail-view\"})\njson.dump(a,open(p,'w'),indent=2,ensure_ascii=False)\nEOF\ncat >> architecture.md <<'EOF'\n\n## S3-sku-detail-view addendum\n\n- New endpoint EP3 `GET /api/stock-records/{sku}?location=` (object) in `app/routes/stock_records.py` -> `stock_record_service.get_record` -> `stock_record_repository.get_by_sku_location`. The (sku, location) equality lookup is served by the existing UNIQUE (sku, location) index (PI1), so no new access_pattern is declared and no schema change is made.\n- Client: `client/src/pages/SkuDetailPage` fetches EP3 via `client/src/api/stockRecords.ts` on route mount; the home `stock-row` is a react-router link to `/sku/:sku?location=` (SPA navigation, R5). Per-field \"not tracked\" rendering reuses the shared not-tracked component from S2 (R9 rendering half).\n- Risk: a 404 (record deleted between list and click) renders an explicit not-found state, not a blank view.\n- Test strategy: AC1 and AC2 are Playwright E2E against the live API on the paired branch; EP3 shape/404 covered by a pytest-bdd API test on the paired branch. Sign-off: proceed (architect-reviewer).\nEOF\nL=<PROJECT_ROOT>/scripts/lk\ncd <PROJECT_ROOT>\n$L consort-annotate-ac --feature F1-stock-visibility --story S3-sku-detail-view --ac AC1-row-opens-sku-detail --layer E2E --api-contract EP3-get-stock-record --ui-seams \"stock-row,sku-detail,detail-sku,detail-location,detail-quantity,detail-tracking-code\" --notes \"E2E client<->server read: clicking a home stock-row performs SPA navigation to /sku/:sku?location= (no full-page reload, R5); client/src/pages SkuDetailPage fetches GET /api/stock-records/{sku}?location= via client/src/api/stockRecords.ts and renders only that record. The boundary app/routes/stock_records.py delegates to app/services/stock_record_service.py, whose repository (app/repositories/stock_record_repository.py) selects the single row by exact (sku, location) on the paired branch, served by the PI1 unique index.\" && \\\n$L consort-annotate-ac --feature F1-stock-visibility --story S3-sku-detail-view --ac AC2-detail-untracked-fields-show-not-tracked --layer E2E --api-contract EP3-get-stock-record --ui-seams \"sku-detail,detail-tracking-code,detail-batch,detail-serial,detail-par-level,detail-not-tracked\" --notes \"E2E rendering half of R9 on the detail view: the server returns tracking_code null (stored NULL per S1 AC4) and F1 has no batch/serial/par_level columns (OD2, OD3), so SkuDetailPage renders a per-field detail-not-tracked pill for each null/absent field and the value otherwise, judged field by field. Presentation logic lives in client/src/components (shared with S2's not-tracked rendering); the boundary never substitutes placeholder text.\" && \\\n$L consort-response-formatter --role architect-reviewer --feature F1-stock-visibility --story S3-sku-detail-view","description":"Add EP3 contract, architecture addendum, annotate ACs, self-check"}

## Final reasoning

I annotated both S3 ACs as `E2E`, with notes, UI seams named in `ia.md`, and a link to the new EP3 endpoint (`GET /api/stock-records/{sku}?location=`). I also added EP3 to `architecture.json` and an S3 section to `architecture.md`; the architect-reviewer self-check passes. No new access pattern was needed, because the existing unique (sku, location) index already serves this lookup.
