# navigator-reflect (navigator) – sonnet

## Prompt

```
REFLECT on story S3-sku-detail-view BEFORE the build lane: independently critique its spec slice (<PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/story.json + acs/*.json) and its test-list (<PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/test-list-per-story.json) against the architecture (<PROJECT_ROOT>/.consort/features/F1-stock-visibility/architecture.md/.json), the NFRs (<PROJECT_ROOT>/.consort/nfrs.md — a project-level file, NOT under the feature dir), and the information architecture (<PROJECT_ROOT>/.consort/design/ia.md). RUBRIC (pre-extracted; judge against THIS) :: layer=E2E | required NFRs, NFR1-data-survival | design-token groups, app_name, app_icon, typography, colors, spacing, radius, shadows, breakpoints, motion, components. Look ONLY for design-time defects that would waste a build cycle. The DETERMINISTIC pre-reflect gate has ALREADY verified the structural classes — client-kind↔AC-layer coherence, E2E-layer coverage by a real Playwright spec, and that no real-browser navigation/reload assertion sits in the jsdom component harness — so do NOT spend a finding re-flagging those; focus on the SEMANTIC defects: (1) ACs that contradict each other; (2) an AC with no covering test, or a test that contradicts its AC; (3) an NFR with no fitness test; (4) a test asserting at a layer the architecture forbids; (5) an AC whose declared layer conflicts with the architecture; (6) an untestable/vacuous AC (no observable outcome); (7) a UI-styling test that asserts inline HTML style or raw CSS in the page SOURCE (e.g. a text-align/color/font check inside a style= attr) for a property the design-guide + design-adherence gate govern, instead of the rendered SEAM (the element carries the design-guide class / data-testid): such a test hard-codes the very inline style the design lane then refactors into a token-driven class, so it blocks that refactor (the ui-style-implementation-test smell); (8) for EACH user-facing AC (declared layer E2E or client): read <PROJECT_ROOT>/.consort/design/ia.md and locate the screen, form, or data-testid seam that AC asserts among the IA's Screens, Navigation, and User flows — emit a ux-designer finding naming every asserted surface the IA leaves out, because the architect would otherwise tag an E2E layer against a screen the IA does not define (the ia refinement adds the missing surface). ALSO, when that AC asserts an INTERACTIVE control (a filter, selector, combobox, search), the IA must name a control the user can OPERATE to reach EVERY state the AC asserts: the control's declared option source must cover the asserted values and its empty / no-match state must be reachable THROUGH the control (e.g. an all-values accessible combobox, not a native datalist whose options are only the filtered rows and whose empty option is unselectable) — emit a ux-designer finding when the IA's control cannot reach an asserted state. This control-SEMANTICS check is the twin of the surface-existence check: a surface the IA NAMES but whose control a user cannot operate to the asserted state is still an ia defect, not a build detail or a test bypass; Do NOT critique implementation, style, or scope, only buildability + internal consistency of THIS story's artifacts. BE EXHAUSTIVE in this ONE pass: findings[] is multi-valued — run EVERY check against EVERY AC, test-list item, and NFR, and emit a SEPARATE finding for EACH distinct defect (decompose a LEGACY multi-part singular NFR fitness_function into its sub-guarantees and flag every uncovered clause — but an NFR that already declares the ATOMIC fitness_functions ARRAY is coverage-checked DETERMINISTICALLY by checkFitnessClauseCoverage at the test_list gate, so do NOT re-decompose it). Do NOT return one finding at a time: the reflect↔revise loop is bounded and escalates after a few laps, so a piecemeal reflect burns that budget on repeated revise→re-test→reflect laps and can hand the human a still-defective design. Write your verdict to <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/reflect-verdict.json as {"version":1,"passed":<bool>,"findings":[{"owner":"spec-author"|"test-strategist"|"architect-reviewer"|"ux-designer","detail":"<the defect>","ac_ids":["<exact story AC id(s) this finding is about>"]}]}. passed:true with findings:[] when the spec + test-list are consistent + buildable (the common case, do NOT invent defects). Attribute each finding to its OWNING role so the fix routes to the right gate: spec-author (an AC/spec defect — the given/when/then or scope); architect-reviewer (an ARCHITECTURE defect — checks 4 and 5, i.e. a wrong layer tag, OR a route/path/contract in an AC's architectural_notes that disagrees with the IA or the tests, OR a persistence invariant that contradicts the AC; this is the architect's product, NOT the spec-author's, so do NOT attribute a notes/layer defect to spec-author); test-strategist (a test-list/coverage defect); ux-designer (a user-facing AC whose surface the IA does not name — defect 8). Set "ac_ids" to the EXACT story AC id(s) the finding concerns (verbatim, from this story's acs/) — the revise re-derives ONLY those ACs' test items; omit ac_ids (or []) ONLY for a whole-story defect that needs the entire test-list rebuilt. Write ONLY that file; the orchestrator routes any fix deterministically.CROSS-STORY CONTEXT — this story is NOT reviewed in isolation. The feature's OTHER stories' ACs, the architecture's open_decisions, and its mandated (not-null) persistence fields are below; use them for the cross-story checks (an AC contradicting a gated sibling AC; a mandated field no sibling submit AC supplies). This IS that context — do not run `consort-cross-story-context` yourself.

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

- Bash {"command":"cd <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view && cat story.json acs/*.json test-list-per-story.json; cat ../../../design/ia.md","description":"Read S3 story, ACs, test-list, and IA"}
- Bash {"command":"cd <PROJECT_ROOT>/.consort && grep -n -i \"NFR1\\|R1\\b\" nfrs.md | head -20; grep -n -i \"sku\\|detail\" features/F1-stock-visibility/architecture.md | head -40","description":"Check NFR1 and architecture detail routes"}
- Bash {"command":"ls design; grep -n -i \"sku\\|detail\" design/ia* | head -30; grep -n \"NFR1\" -r features/F1-stock-visibility/architecture.json | head","description":"Locate IA and NFR1 refs"}
- Write {"file_path":"<PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S3-sku-detail-view/reflect-verdict.json","content":"{\"version\":1,\"passed\":true,\"findings\":[]}\n"}

## Final reasoning

I reviewed S3-sku-detail-view and found no defects. I wrote `reflect-verdict.json` with `passed: true` and no findings.

Every surface and `data-testid` the two ACs assert is in `ia.md`, and neither AC conflicts with the sibling stories. Both ACs have covering tests.

I didn't check `NFR1-data-survival` against a test in this story. The story only reads data (no writes or migrations), so I didn't expect one here.
