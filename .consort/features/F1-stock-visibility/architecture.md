# F1-stock-visibility — Architecture

Layering (uses the project conventions): boundary `app/routes` (JSON only, React SPA under `client/`) → service `app/services` → repository `app/repositories` (the only layer that touches the ORM; `app/repositories/session.py` `get_session` is re-exported by the service) → models package `app/models/` (`stock_record.py`) → infrastructure `app/database.py`. `service_backed: true`, because F1 persists `stock_records`.

## Layer assignments (S1-file-stock-record)

- AC1, AC2, AC3 are tagged **E2E**. Each one is a client↔server round trip on `/stock/new`: the success status or the field-named inline error is produced by the real `EP1-file-stock-record` response.
- AC4 is tagged **API**. It is the R3 collision write rule: an upsert on `(sku, location)`.
- AC5 is tagged **API**. A blank tracking code is stored as NULL. This is the data contract S2/S3 rely on.

## Layer assignments (S2-stock-by-location-table)

- AC1–AC5 are all tagged **E2E**. Each table/empty-state outcome is the React home view rendering the real `EP2-list-stock-records` response (bare list, ordered by location then sku). Read-only story: no new persistence invariant; AP1 `(location, sku)` ordering pulls the DBA back for an index decision.

## Layer assignments (S3-sku-detail-view)

- AC1–AC3 are all tagged **E2E**. The detail view (`/sku/:sku?location=`) renders the real `EP3-get-stock-record` response: an object for the exact (sku, location) pair, or a 404 echoing the pair, which the client renders as `detail-not-found`. Read-only story: no new persistence invariant. The lookup is an equality on the PI1 `(sku, location)` unique index, so it needs no new access pattern.

## Architectural Concerns Mapping

| Concern | Owner layer | Module | Notes |
|---|---|---|---|
| Request parsing / shape | Boundary | `app/routes/stock_records.py` | Returns JSON. 422 carries `errors[{field,message}]` |
| Validation (required fields, non-negative whole quantity) | Service | `app/services/stock_record_service.py` | Field-named messages (R7) |
| Collision resolution (upsert) | Repository | `app/repositories/stock_record_repository.py` | `ON CONFLICT (sku, location) DO UPDATE` (R3) |
| Transactions | Service | `stock_record_service` | One unit of work per filing |
| Blank → NULL normalization | Service | `stock_record_service` | `tracking_code` is nullable (R9) |
| Data integrity backstop | Schema | migration `stock_records` | PI1–PI6 |
| Error display | Client | `client/src/api` + `describeError()` | Field-keyed inline errors |
| Listing order (location, sku) | Repository | `stock_record_repository.list_ordered()` | `ORDER BY location, sku` (AP1) |
| 'not tracked' rendering of nulls | Client | `client/src/components` stock table | Per-field, per-row (R9) |
| Single-record lookup + not-found | Service / Boundary | `stock_record_service.get_record()` → `stock_record_repository.get_by_key()` | The service returns None; the boundary maps that to a 404 `{error, sku, location}` (EP3) |
| Auth/authz | N/A | — | Out of bounds for V1 |
| Logging/observability | Cross-cutting | scaffold logging | No F1-specific budget |
| Config | Infrastructure | `app/database.py` | `DATABASE_URL` from env. Scaffold defaults are left unchanged |

## Pattern proposals

- One `StockRecordService` that owns the filing use case: it validates, normalizes and upserts. The boundary stays thin.
- `StockRecordRepository.upsert(sku, location, quantity, tracking_code) -> (record, created)`. It does the collision resolution in a single atomic statement.
- One model module per aggregate: `app/models/stock_record.py`.
- On the client, a `useFileStockRecord` hook in `client/src/hooks` calls `client/src/api/stockRecords.ts`. The form component renders field errors from the server's `errors[]`.

- S2: `StockRecordService.list_records()` → `StockRecordRepository.list_ordered()`; a `useStockRecords` hook fetches on route mount so SPA navigation back from `/stock/new` re-fetches (R5).

- S3: `StockRecordService.get_record(sku, location)` → `StockRecordRepository.get_by_key()`. A `useStockRecord(sku, location)` hook backs the detail page, which renders 'not tracked' pills per null field and a not-found state on a 404.

## Risks

- The IA names batch and serial inputs, but F1 stores no batch or serial columns (OD1, OD-scope-batch-serial-inputs). If the PO keeps those inputs, they would collect data that is never persisted.
- R1 audit: an update-in-place filing (AC4) changes a quantity without an audit trail. This is deferred as OD2. Adding an adjustment log later is additive.
- The PI5 CHECK duplicates the service validation on purpose, as defense in depth. Any later change to the quantity rule has to update both.

- S2 lists every row unpaginated; a large warehouse may need pagination later (additive query param).

- S3: location is passed as a query param (`?location=`) instead of a path segment, so a location containing `/` still resolves. A SKU containing `/` would need URL-encoding in `/sku/:sku`.

## Decisions

- **OD1 (resolved):** F1 persists only `sku`, `location`, `quantity`, `tracking_code` (nullable). The API returns `batch`, `serial` and `par_level` as null, and the UI renders them as "not tracked". F6 adds batch and serial additively. *Recommend: accept.*
- **OD-scope-batch-serial-inputs (open, for the PO):** *Recommend:* drop the `stock-record-batch` and `stock-record-serial` inputs from the F1 form until F6 adds them.
- **OD2 (deferred):** the immutable timestamp + actor audit (R1) applies to the inventory-adjustment features (receive/pick/adjust/count), with the actor typed in by the operator because there is no auth. *Recommend:* treat F1 filing as establishing a record, not as an adjustment.
- Par level is not filed in F1, as the spec recommends.

## Test strategy

All tests run against the paired Lakebase branch `DATABASE_URL`, with Alembic migrations applied first. Nothing is mocked, stubbed or in-memory (R4).
- **pytest-bdd API suite** (`tests/features/*.feature` + `tests/step_defs/test_*.py`): AC4 and AC5, plus the server half of AC2 and AC3 (422 field errors, no row stored).
- **Playwright E2E against the live API:** S1 AC1, AC2, AC3; S2 AC1–AC5 (seeded via real rows on the branch).
- **pytest-bdd API suite (S2):** server half of EP2 — ordering, nulls, quantity 0, empty list.
- **DB-integration tests:** PI1 through PI6. These cover the unique pair, the NOT NULLs, the quantity CHECK, and schema recreation after a downgrade followed by `upgrade head`.
- **S3:** Playwright runs AC1–AC3 against the live API. pytest-bdd covers the server half of EP3: an exact-pair match, nulls, and the 404 echoing the pair.
- Cleanup uses targeted DELETEs on `stock_records`.

## Sign-off

Recommendation: **proceed**. OD-scope-batch-serial-inputs goes to the PO at the design gate. — architect-reviewer
