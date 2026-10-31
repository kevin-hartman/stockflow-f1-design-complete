# navigator-reconcile (navigator) – sonnet

## Prompt

```
FEATURE-COMPLETE RECONCILIATION for F1-stock-visibility: every story is now designed + gated. Independently check the WHOLE set of stories against ITSELF — symmetrically, every story vs every other, INCLUDING the first-designed story (which was never checked against its later siblings). The cross-story context (all stories' ACs + the architecture's open_decisions + its mandated not-null fields) is injected below. Look ONLY for CROSS-STORY defects: an AC that contradicts another story's gated AC; a mandated (not-null) field no story's submit AC supplies; two stories' ACs overlapping on the same observable behavior; a persistence invariant anchored to the wrong story. Do NOT re-review a single story in isolation — each already passed its own reflect + gate. Write the verdict to <PROJECT_ROOT>/.consort/features/F1-stock-visibility/reconcile-verdict.json: {"version":1,"passed":<bool>,"findings":[{"owner":"spec-author"|"test-strategist"|"architect-reviewer"|"ux-designer","story":"<the OWNING story to change>","detail":"<the cross-story defect>","ac_ids":["<exact AC id(s)>"],"severity":"blocking"|"advisory"}]}. Use passed:true with findings:[] when the stories are mutually consistent. Each finding names the OWNING story (the one reopened + changed) and its author. Produce ONLY the verdict file, then stop.FEATURE-COMPLETE CROSS-STORY RECONCILIATION — every story below is now designed + gated. Check the WHOLE set against ITSELF, symmetrically (every story vs every other, including the first-designed one): an AC that contradicts another story's gated AC; a mandated (not-null) field no story's submit AC supplies; two stories' ACs that overlap on the same observable behavior; an invariant anchored to the wrong story. For each defect, name the OWNING story (the one to change) + its author (owner). This IS the context — do not run `consort-cross-story-context` yourself.

{
  "stories": [
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
    },
    {
      "story": "S3-sku-detail-view",
      "acs": [
        {
          "ac_id": "AC1-row-opens-detail",
          "status": "draft",
          "layer": "E2E",
          "given": "the home table lists records (SKU-100, A-01, qty 12, tracking code A01-B7-S0042) and (SKU-100, B-02, qty 7)",
          "when": "the operator selects the (SKU-100, A-01) row",
          "then": "a detail view for exactly that record opens by in-app navigation with no full-page reload, showing SKU-100, location A-01, quantity 12 and tracking code A01-B7-S0042, and not the values of the (SKU-100, B-02) record",
          "architectural_notes": "E2E: the home stock-row is a SPA link to /sku/:sku?location=, and the React detail page fetches EP3 for exactly that (sku, location) pair with no full-page reload. app/routes delegates to app/services/stock_record_service.get_record, which calls app/repositories/stock_record_repository.get_by_key (equality on the PI1 unique pair, so no other record's values can leak). Playwright seeds both SKU-100 rows on the paired branch and asserts the A-01 values only."
        },
        {
          "ac_id": "AC2-detail-untracked-fields-show-not-tracked",
          "status": "draft",
          "layer": "E2E",
          "given": "a record (SKU-200, A-01, qty 5) filed without a tracking code (stored null) and with no par level",
          "when": "the operator opens that record's detail view",
          "then": "the tracking-code field and the par-level field each read 'not tracked', judged independently, while SKU, location and quantity are shown; no field is blank or reads 'null' and the view renders cleanly",
          "architectural_notes": "E2E: the server returns tracking_code null (stored NULL per S1 AC5) and par_level null (OD1) for the record, and the client detail component renders each null field as its own not-tracked pill, independently. This realizes the NFR9 detail clause (R9). Playwright against the live API asserts both pills, the SKU/location/quantity values, and that no field is blank or 'null'."
        },
        {
          "ac_id": "AC3-unknown-record-not-found",
          "status": "draft",
          "layer": "E2E",
          "given": "no stock record exists for (SKU-999, Z-99)",
          "when": "the operator opens the detail view address for (SKU-999, Z-99) directly",
          "then": "an explicit 'record not found' state naming SKU-999 at Z-99 is shown with a way back to the stock table, never a blank page, an empty detail view or a generic error",
          "architectural_notes": "E2E: not-found is decided by the server. The service returns no record, and the boundary maps that to a 404 echoing the requested sku and location. The client must parse that real response and render the detail-not-found state naming SKU-999 at Z-99 with a back link to /, never an empty detail view or a generic error. Playwright opens /sku/SKU-999?location=Z-99 directly against the live API on a branch with no such row."
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

- Write {"file_path":"<PROJECT_ROOT>/.consort/features/F1-stock-visibility/reconcile-verdict.json","content":"{\"version\":1,\"passed\":true,\"findings\":[{\"owner\":\"ux-designer\",\"story\":\"S1-file-stock-record\",\"detail\":\"Open decision OD-scope-batch-serial-inputs: the IA names stock-record-batch/stock-record-serial inputs on /stock/new, but no S1 AC files batch or serial (OD1 says F1 does not persist them; F6 owns the split). Drop the inputs from the F1 form IA, or the form would show fields that are silently discarded.\",\"ac_ids\":[\"AC1-file-record-appears\"],\"severity\":\"advisory\"}]}\n"}

## Final reasoning

I wrote the reconcile verdict. It passes, with one advisory finding and no blocking ones.

The advisory is on S1 (`AC1-file-record-appears`), owned by the ux-designer. The IA puts batch and serial inputs on `/stock/new`, but no S1 AC files them and OD1 says F1 doesn't store them. Those inputs should come off the F1 form.

I found no other cross-story defects. The not-null fields (sku, location, quantity) are all supplied by the S1 form.
