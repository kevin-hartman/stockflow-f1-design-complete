# architect-reviewer (architect-reviewer) – opus

## Prompt

```
Annotate story S3-sku-detail-view's acceptance criteria + nfrs.md coverage. Author architectural_notes for ALL of story S3-sku-detail-view's ACs: AC1-row-opens-detail, AC2-detail-untracked-fields-show-not-tracked, AC3-unknown-record-not-found. Every one of this story's ACs MUST end up with a non-empty "architectural_notes" field in its acs/<AC>.json (the layer it lives in + how it realizes the design) — the design gate verifies that, and the spec-author's "layer" field does NOT count. architectural_notes are per-AC, so author the scoped ACs above even when the feature-level architecture.json already exists from an earlier story. For every E2E AC, ALSO pass consort-annotate-ac --ui-seams with the ia.md data-testid seam token(s) the AC's UI drives — the column / cell / region / empty-state / form field it asserts (e.g. "stock-cell-tracking,stock-not-tracked"). A deterministic gate checks each seam is named in <PROJECT_ROOT>/.consort/design/ia.md; if the AC needs a surface the IA does not yet name, declare the seam you need anyway and the gate routes the ux-designer to add it to ia.md. In architecture.json, make an EXPLICIT service_backed call (required): set service_backed:true if the feature persists data (a DB table/migration) or carries business logic, and then you MUST declare boundary, service, and repository layers (plus a "models" PACKAGE app/models/, one module per domain object, NOT a flat app/models.py, when it persists entities); set false ONLY for a trivial static/read-through endpoint. An Infra-layer AC or a migration/schema/storage NFR while service_backed is false hard-blocks the gate. When service_backed:true you MUST also declare architecture.json persistence_invariants[]: the DB-level guarantees the schema enforces (each with id, type one of unique|foreign_key|cascade|not_null|check|transactional|migration_reversible, table, and a one-line brief), covering unique/composite keys, foreign keys + cascade rules, NOT NULL / CHECK constraints, any transactional-atomicity boundary, and migration reversibility. The test-strategist must cover each with a real-branch test; a service_backed feature with no persistence_invariants hard-blocks the gate. When THIS story introduces a NEW read/query shape over the schema — a filter, lookup, sort, or join an AC drives (e.g. "list stock filtered by location", "look up a SKU's rows by sku") that is an INDEX candidate — declare it in architecture.json access_patterns[] (id, a one-line brief, and the table + column(s) it reads) so the DBA is pulled back in to reconsider the physical design for it; this is how a later READ story evolves the schema, since a story that adds no persistence_invariant would otherwise skip the DBA. Declare an access_pattern ONLY for a genuinely new query shape (not for every read, and not for one already covered by a prior story's pattern or a primary-key lookup). REUSE the established project architecture conventions (set by F1-stock-visibility): boundary=app/routes (react), service=app/services, repository=app/repositories, models=app/models, infrastructure=app/database.py. Declare the SAME role -> module paths in architecture.json, do NOT remap or rename an established layer; a divergent layout hard-blocks the spec gate and mismatches the inherited code. Author this story's architectural_notes FRESH from the ACs + IA on disk this turn. On a reflect-revise re-run the self-heal clears this story's prior architectural_notes BY DESIGN, so an absent/empty prior is the expected revise state — re-derive against the corrected ACs + IA. Write every artifact under the ABSOLUTE artifact root <PROJECT_ROOT>/.consort (this feature: <PROJECT_ROOT>/.consort/features/F1-stock-visibility/; this story: <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/); use that absolute path and never resolve or guess the project root yourself.CROSS-STORY CONTEXT — this story is NOT reviewed in isolation. The feature's OTHER stories' ACs, the architecture's open_decisions, and its mandated (not-null) persistence fields are below; use them for the cross-story checks (an AC contradicting a gated sibling AC; a mandated field no sibling submit AC supplies). This IS that context — do not run `consort-cross-story-context` yourself.

{
  "current_story": "S3-sku-detail-view",
  "sibling_stories": [
    {
      "story": "S1-file-stock-record",
      "acs": [
        {
          "ac_id": "AC1-file-record-appears",
          "status": "draft",
          "layer": "E2E",
          "given": "the stock-filing form is open and no record exists for SKU 'SKU-100' at location 'A-01'",
          "when": "the operator enters SKU 'SKU-100', location 'A-01', quantity 12 and a tracking code, and submits the form",
          "then": "the submission succeeds without a full-page reload, the operator sees confirmation, and a stock record for (SKU-100, A-01) with quantity 12 and that tracking code exists",
          "architectural_notes": "E2E client-server round trip: the React /stock/new form posts asynchronously to the app/routes stock-records endpoint and renders the success status from the real response, with no page reload. The boundary hands off to app/services/stock_record_service, which validates and calls app/repositories/stock_record_repository to persist an app/models/stock_record row in the paired branch DB. The tests assert the success region and the stored (SKU-100, A-01, 12, tracking code) row."
        },
        {
          "ac_id": "AC2-missing-required-field-inline",
          "status": "draft",
          "layer": "E2E",
          "given": "the stock-filing form is open",
          "when": "the operator submits with a required field (SKU, location, or quantity) left blank",
          "then": "an inline error naming that field is shown next to it (never a generic 'bad request'), the entered values are kept, and no record is created",
          "architectural_notes": "E2E: the required-field rule is decided by the server, and the client renders the real 422 errors[] next to each named field. Required-field validation belongs to the service layer, with request parsing at the boundary; the client maps errors by field and keeps the entered values. A real Playwright test against the live API asserts the field-named inline error and that no stock_records row was created."
        },
        {
          "ac_id": "AC3-invalid-quantity-inline",
          "status": "draft",
          "layer": "E2E",
          "given": "the stock-filing form is open with SKU and location filled",
          "when": "the operator submits a quantity that is not a non-negative whole number (e.g. -3 or 2.5 or 'abc')",
          "then": "an inline error naming the quantity field and stating the rule is shown next to it, and no record is created",
          "architectural_notes": "E2E: the service enforces that quantity is a non-negative whole number (R2/R7), and the PI5 CHECK constraint backs it up. The client renders the server's quantity error, which states the rule, inline next to the quantity field. Tests cover -3, 2.5 and 'abc' against the live API and confirm no row is stored."
        },
        {
          "ac_id": "AC4-same-key-updates-in-place",
          "status": "draft",
          "layer": "API",
          "given": "a stock record exists for (SKU-100, A-01) with quantity 12",
          "when": "the operator files a record for the same SKU-100 at A-01 with quantity 20",
          "then": "the existing record is updated in place to quantity 20, exactly one record exists for (SKU-100, A-01), and no duplicate is created",
          "architectural_notes": "API: this is the backend write rule for an R3 collision. app/services/stock_record_service calls an upsert in app/repositories/stock_record_repository (INSERT ... ON CONFLICT (sku, location) DO UPDATE), relying on PI1's unique constraint. The upsert returns 200 with created=false. A pytest-bdd test on the paired branch asserts quantity 20 and exactly one row."
        },
        {
          "ac_id": "AC5-optional-tracking-code-omitted",
          "status": "draft",
          "layer": "API",
          "given": "the stock-filing form is open",
          "when": "the operator submits a valid SKU, location and quantity with the tracking code left empty",
          "then": "the submission is accepted and the record is stored with no tracking code (null), so downstream views can show it as 'not tracked'",
          "architectural_notes": "API: the service turns a blank or absent tracking_code into NULL before the repository writes it, and stock_records.tracking_code stays nullable (no not_null invariant). A pytest-bdd test on the paired branch asserts the request is accepted and the stored tracking_code IS NULL, never an empty string. This is the null state that S2/S3 render as 'not tracked' (R9)."
        }
      ]
    },
    {
      "story": "S2-stock-by-location-table",
      "acs": [
        {
          "ac_id": "AC1-table-lists-stock-by-location",
          "status": "draft",
          "layer": "E2E",
          "given": "stock records exist for (SKU-100, A-01, qty 12), (SKU-200, A-01, qty 5) and (SKU-100, B-02, qty 7)",
          "when": "the operator opens the home view",
          "then": "a stock-by-location table shows exactly one row per record, each with its SKU, location, quantity and tracking code, ordered by location so all A-01 rows appear together before B-02",
          "architectural_notes": "E2E: the React home view fetches the real EP2 list on mount and renders one stock-row per record in server order. app/routes delegates to app/services/stock_record_service.list_records, which calls app/repositories/stock_record_repository.list_ordered (ORDER BY location, sku; AP1). Playwright seeds three rows on the paired branch and asserts row count, cell values and A-01 rows grouped before B-02."
        },
        {
          "ac_id": "AC2-untracked-fields-show-not-tracked",
          "status": "draft",
          "layer": "E2E",
          "given": "a record (SKU-100, A-01) filed with a tracking code and a record (SKU-200, A-01) filed without one (tracking code null), and no par level filed for either",
          "when": "the operator opens the home view",
          "then": "each optional field is judged independently per row: SKU-200's tracking-code cell reads 'not tracked' while SKU-100's shows its code, and the par-level cell reads 'not tracked' on both rows; no cell is blank and the row still renders cleanly",
          "architectural_notes": "E2E: the server returns tracking_code null for the untracked row and par_level null for all rows (OD1); the client stock-table component renders each null field as its own not-tracked pill per row, independently. Realizes NFR9's rendering clause (R9). Playwright against the live API asserts the per-cell pill vs code and that no cell is blank or 'null'."
        },
        {
          "ac_id": "AC3-empty-state-when-no-stock",
          "status": "draft",
          "layer": "E2E",
          "given": "no stock records exist",
          "when": "the operator opens the home view",
          "then": "an explicit teaching empty state is shown in place of the table (stating there is no stock yet and pointing the operator to file a stock record), never a blank page or an empty table header",
          "architectural_notes": "E2E: the empty state is decided by the real server response (an empty list), so the client must parse the actual EP2 body; the home page renders empty-state with a CTA to /stock/new and does not render stock-table. Playwright on a branch with stock_records cleared asserts the empty state and no table header."
        },
        {
          "ac_id": "AC4-filed-record-appears-without-reload",
          "status": "draft",
          "layer": "E2E",
          "given": "the home table shows existing stock and no record exists for (SKU-300, C-03)",
          "when": "the operator files a record for (SKU-300, C-03, qty 4) through the form and returns to the home view within the app",
          "then": "the new (SKU-300, C-03) row appears in the table with quantity 4, reached by in-app navigation with no full-page reload",
          "architectural_notes": "E2E client-server round trip across EP1 then EP2: the form posts asynchronously, and SPA navigation back to / remounts the home route, whose useStockRecords hook re-fetches EP2 (no client-side stale cache, R5). Playwright files (SKU-300, C-03, 4) via the live API and asserts the new row appears with no full-page navigation event."
        },
        {
          "ac_id": "AC5-zero-quantity-record-listed",
          "status": "draft",
          "layer": "E2E",
          "given": "a record exists for (SKU-400, A-01) with quantity 0",
          "when": "the operator opens the home view",
          "then": "the (SKU-400, A-01) row is listed with quantity shown as 0, per the feature data semantic that a quantity-0 record is a valid filed record",
          "architectural_notes": "E2E: the repository listing applies no quantity filter, so a quantity-0 row (valid under PI5 CHECK >= 0) is returned by EP2 and the client renders 0 rather than hiding the row or treating 0 as falsy/blank. Playwright seeds (SKU-400, A-01, 0) on the paired branch and asserts the row with quantity 0."
        }
      ]
    }
  ],
  "open_decisions": [
    {
      "id": "OD1-optional-columns-in-f1",
      "question": "Does F1 persist batch, serial and par_level columns, or only tracking_code (batch/serial split is F6)?",
      "decision_status": "resolved",
      "resolved_by_story": "S1-file-stock-record",
      "resolution": "F1 stock_records persists sku, location, quantity, tracking_code (nullable) only. batch/serial/par_level are not stored in F1; the API returns them as null and S2/S3 render 'not tracked'. F6 adds batch/serial additively."
    },
    {
      "id": "OD-scope-batch-serial-inputs",
      "question": "The IA names stock-record-batch / stock-record-serial form inputs on /stock/new, but no S1 AC files batch or serial and F6 owns the split — should those inputs be dropped from the F1 form?",
      "decision_status": "open"
    },
    {
      "id": "OD2-filing-audit-trail",
      "question": "Does an update-in-place filing (AC4) count as an inventory adjustment requiring an immutable timestamp + actor (R1), given no auth in V1 and no actor field on the form?",
      "decision_status": "deferred"
    }
  ],
  "required_persistence_fields": [
    {
      "invariant_id": "PI2-sku-not-null",
      "table": "stock_records",
      "brief": "stock_records.sku is NOT NULL; inserting a row with a null sku is rejected by the database."
    },
    {
      "invariant_id": "PI3-location-not-null",
      "table": "stock_records",
      "brief": "stock_records.location is NOT NULL; inserting a row with a null location is rejected by the database."
    },
    {
      "invariant_id": "PI4-quantity-not-null",
      "table": "stock_records",
      "brief": "stock_records.quantity is NOT NULL; inserting a row with a null quantity is rejected by the database."
    }
  ]
} Be terse: produce ONLY the required artifact file(s) on disk, then stop with at most a one-line confirmation. Do NOT print a plan, a summary of what you did, rationale, tables, or restate the artifacts to stdout, that output is wasted latency. The files on disk are the deliverable, not your prose.
```

## Tools used

- Bash {"command":"cd <PROJECT_ROOT>/.consort/features/F1-stock-visibility; for f in stories/S3-sku-detail-view/acs/*.json; do echo \"== $f\"; cat $f; done; cat architecture.json; grep -n -i \"detail\\|not-found\\|sku-detail\" ../../design/ia.md | head -60","description":"Read S3 ACs, architecture.json, and IA detail seams"}
- Bash {"command":"cd <PROJECT_ROOT>/.consort; grep -o 'stock-row[a-z<>-]*' design/ia.md | sort -u; grep -n '^#' features/F1-stock-visibility/architecture.md","description":"Check stock-row seam and architecture.md sections"}
- Read {"file_path":"<PROJECT_ROOT>/.consort/features/F1-stock-visibility/architecture.md"}
- Bash {"command":"cd <PROJECT_ROOT>/.consort/features/F1-stock-visibility && python3 -I - <<'EOF'\nimport json\np='architecture.json'; a=json.load(open(p))\na['api_contracts']=[c for c in a['api_contracts'] if c['id']!='EP3-get-stock-record']+[{\n \"id\":\"EP3-get-stock-record\",\"method\":\"GET\",\"path\":\"/api/stock-records/lookup\",\"query\":[\"sku\",\"location\"],\"shape\":\"object\",\n \"item\":\"200 body: the single record matching the exact (sku, location) pair {id: int, sku: string, location: string, quantity: int >= 0, tracking_code: string|null, batch: null, serial: null, par_level: null} (batch/serial/par_level always null in F1 per OD1). 404 when no record matches, body {error: 'not_found', sku: string, location: string} echoing the requested pair. Lookup served by the PI1 (sku, location) unique index.\",\n \"story\":\"S3-sku-detail-view\"}]\nfor n in a['nfrs']:\n  if n['id']=='NFR9-graceful-rendering':\n    fs=[f for f in n['fitness_functions'] if not (isinstance(f,dict) and 'AC2-detail-untracked-fields-show-not-tracked' in f.get('realized_by',[]))]\n    fs.append({\"clause\":\"In the record detail view each absent optional field (tracking_code null, par_level null) renders its own 'not tracked' pill independently of the other fields; no field is blank, 'null' or crashes the view.\",\"realized_by\":[\"AC2-detail-untracked-fields-show-not-tracked\"]})\n    n['fitness_functions']=fs\n    n['requirement']=n['requirement'].rstrip('.')+'. S3 renders each null optional field on the record detail view as \\'not tracked\\'.'\njson.dump(a,open(p,'w'),indent=2); open(p,'a').write('\\n')\nEOF","description":"Add EP3 detail contract and NFR9 detail clause"}
- Bash {"command":"cd <PROJECT_ROOT> && F=F1-stock-visibility S=S3-sku-detail-view && \\\n./scripts/lk consort-annotate-ac --feature $F --story $S --ac AC1-row-opens-detail --layer E2E --api-contract EP3-get-stock-record --ui-seams \"stock-row,sku-detail,detail-sku,detail-location,detail-quantity,detail-tracking-code\" --notes \"E2E: the home stock-row is a SPA link to /sku/:sku?location=, and the React detail page fetches EP3 for exactly that (sku, location) pair with no full-page reload. app/routes delegates to app/services/stock_record_service.get_record, which calls app/repositories/stock_record_repository.get_by_key (equality on the PI1 unique pair, so no other record's values can leak). Playwright seeds both SKU-100 rows on the paired branch and asserts the A-01 values only.\" && \\\n./scripts/lk consort-annotate-ac --feature $F --story $S --ac AC2-detail-untracked-fields-show-not-tracked --layer E2E --api-contract EP3-get-stock-record --ui-seams \"sku-detail,detail-sku,detail-location,detail-quantity,detail-tracking-code,detail-par-level,detail-not-tracked\" --notes \"E2E: the server returns tracking_code null (stored NULL per S1 AC5) and par_level null (OD1) for the record, and the client detail component renders each null field as its own not-tracked pill, independently. This realizes the NFR9 detail clause (R9). Playwright against the live API asserts both pills, the SKU/location/quantity values, and that no field is blank or 'null'.\" && \\\n./scripts/lk consort-annotate-ac --feature $F --story $S --ac AC3-unknown-record-not-found --layer E2E --api-contract EP3-get-stock-record --ui-seams \"detail-not-found,detail-not-found-back\" --notes \"E2E: not-found is decided by the server. The service returns no record, and the boundary maps that to a 404 echoing the requested sku and location. The client must parse that real response and render the detail-not-found state naming SKU-999 at Z-99 with a back link to /, never an empty detail view or a generic error. Playwright opens /sku/SKU-999?location=Z-99 directly against the live API on a branch with no such row.\" && \\\n./scripts/lk consort-response-formatter --role architect-reviewer --feature $F --story $S","description":"Annotate S3 ACs and run self-check"}
- Bash {"command":"cd <PROJECT_ROOT>/.consort/features/F1-stock-visibility && python3 -I - <<'EOF'\np='architecture.md'; s=open(p).read()\ns=s.replace(\"## Architectural Concerns Mapping\",\"## Layer assignments (S3-sku-detail-view)\\n\\n- AC1–AC3 are all tagged **E2E**. The detail view (`/sku/:sku?location=`) renders the real `EP3-get-stock-record` response: an object for the exact (sku, location) pair, or a 404 echoing the pair, which the client renders as `detail-not-found`. Read-only story: no new persistence invariant. The lookup is an equality on the PI1 `(sku, location)` unique index, so it needs no new access pattern.\\n\\n## Architectural Concerns Mapping\",1)\ns=s.replace(\"| Auth/authz | N/A\",\"| Single-record lookup + not-found | Service / Boundary | `stock_record_service.get_record()` → `stock_record_repository.get_by_key()` | The service returns None; the boundary maps that to a 404 `{error, sku, location}` (EP3) |\\n| Auth/authz | N/A\",1)\ns=s.replace(\"## Risks\",\"- S3: `StockRecordService.get_record(sku, location)` → `StockRecordRepository.get_by_key()`. A `useStockRecord(sku, location)` hook backs the detail page, which renders 'not tracked' pills per null field and a not-found state on a 404.\\n\\n## Risks\",1)\ns=s.replace(\"## Decisions\",\"- S3: location is passed as a query param (`?location=`) instead of a path segment, so a location containing `/` still resolves. A SKU containing `/` would need URL-encoding in `/sku/:sku`.\\n\\n## Decisions\",1)\ns=s.replace(\"- Cleanup uses\",\"- **S3:** Playwright runs AC1–AC3 against the live API. pytest-bdd covers the server half of EP3: an exact-pair match, nulls, and the 404 echoing the pair.\\n- Cleanup uses\",1)\nopen(p,'w').write(s)\nEOF","description":"Add S3 sections to architecture.md"}

## Final reasoning

I annotated S3's three ACs as E2E with notes, `ia.md` seams and the new `EP3-get-stock-record` contract; the self-check passes. I also added EP3 and an NFR9 detail-view clause to `architecture.json` and S3 sections to `architecture.md`. No new access pattern, since the lookup uses the existing (sku, location) unique index.
