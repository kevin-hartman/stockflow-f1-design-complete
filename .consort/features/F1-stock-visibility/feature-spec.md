# F1 — File a stock record and see it by SKU and location

## Summary
Operators file stock records (SKU, location, quantity, combined tracking code) and see them in a home stock-by-location table and a SKU detail view, inside a single-page app. Each (sku, location) pair is uniquely addressable.

## Data semantics
- One record per (sku, location); re-filing the same pair updates it in place (recommended resolution, see open questions).
- The tracking code is optional; an omitted code is stored null and is shown as "not tracked".
- A quantity-0 record is still a valid filed record (recommended resolution, see open questions).

## Stories
- S1-file-stock-record — file a record via the form with inline field-named validation; one record per (sku, location).
- S2-stock-by-location-table — home table of stock by location with "not tracked" fields and an empty state.
- S3-sku-detail-view — detail view for a single record, opened from a table row.

### S1-file-stock-record
ACs: AC1 valid submission creates the record (no full reload); AC2 blank required field → inline field-named error; AC3 invalid quantity → inline error; AC4 same (sku, location) updates in place, no duplicate; AC5 tracking code omitted → stored null.

## Out of scope
- Adjusting, receiving, picking, counting stock (F2–F5).
- Searching or filtering the table (F9).
- Splitting the tracking code into batch and serial columns (F6).

## Open questions
- Re-filing an existing (sku, location): update in place, reject, or add to quantity? Recommended: update in place (replace quantity/tracking code).
- Is the tracking code required? Recommended: optional (stored null when omitted), since the request expects "not tracked" rendering.
- Par level is named as an optional detail but not as a form field — is it filed in F1? Recommended: not filed in F1; shown as "not tracked".
- Is quantity 0 allowed? Recommended: yes (non-negative whole number).

### S2-stock-by-location-table
ACs: AC1 table lists one row per record ordered by location; AC2 absent optional fields (null tracking code, par level) show "not tracked" per field independently; AC3 teaching empty state when no stock; AC4 newly filed record appears via in-app navigation, no reload; AC5 quantity-0 record is listed.
Open questions (S2):
- "An empty location shows an explicit empty state": locations exist only through records in F1, so is this the whole-table empty state? Recommended: yes, a home-view empty state when no records exist; no per-location empty rows.
- Batch and serial live inside the combined tracking code until F6, so they are not separate columns here. Recommended: the not-tracked fields in F1 are the tracking code and par level.

### S3-sku-detail-view
ACs: AC1 selecting a table row opens that record's detail view in-app (no reload) with its SKU, location, quantity and tracking code; AC2 null tracking code and absent par level each show "not tracked" independently on the detail view; AC3 a detail address for a non-existent (sku, location) shows an explicit not-found state with a way back.
Open questions (S3):
- Should the detail view be directly addressable (deep link / tablet refresh)? Recommended: yes, keyed by (sku, location), which is what makes the AC3 not-found state reachable.
- Does the detail view offer edit actions? Recommended: no, read-only in F1 (adjusting stock is F2).
