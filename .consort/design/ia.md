# StockFlow Information Architecture

## Screens
| Screen | Route | Purpose | Key seams |
|---|---|---|---|
| Home: stock by location | `/` | Calm stock-by-location table (SKU, location, quantity, state pill, tracking code / batch / serial), with the search surface above it | `home-title`, `stage-label`, `search-input`, `stock-table`, `stock-row`, `integrity-count`, `migration-overlay` |
| SKU detail | `/sku/:sku` | Narrow single-column view of one SKU: stock per location, tracking code / batch / serial, "not tracked" where empty | `sku-detail`, `stage-label`, `stock-state-pill` |
| Receive | `/receive` | Inbound receipt form (supplier, SKU, quantity, location) | `receive-form`, field errors |
| Pick | `/pick` | Outbound pick form; refuses overcommit inline | `pick-form` |
| Adjust | `/adjust` | Level correction, never negative; inline green flash | `adjust-form` |
| Count | `/count` | Cycle count with variance, then reconcile | `count-form` |
| File stock record (F1-S1) | `/stock/new` | `page--narrow` `card` form (`field` inputs, `btn--primary` submit) to file or update one stock record keyed by (SKU, location). Controls: SKU text input (required), Location text input (required), Quantity numeric text input (required, non-negative whole number), optional Tracking code / Batch / Serial text inputs (blank submits as null, never placeholder). Submit is async (no full-page reload); success shows an inline `alert` success region (`role="status"`, aria-live) saying "Record filed" or "Record updated" when the (SKU, location) already existed (updated in place, no duplicate). Each blank required field / invalid quantity shows an inline field-named `field__error` next to it; never a generic "bad request"; nothing stored on error | `stock-record-form`, `stock-record-sku`, `stock-record-location`, `stock-record-quantity`, `stock-record-tracking-code`, `stock-record-batch`, `stock-record-serial`, `stock-record-submit`, `stock-record-success`, `stock-record-sku-error`, `stock-record-location-error`, `stock-record-quantity-error` |
| Home stock-by-location table (F1-S2) | `/` | Refines Home: `stock-table` rows ordered by location then SKU so each location's stock reads together, with a `location-group` header row per location; columns SKU, Location, Quantity (mono, right-aligned), Tracking code, Batch, Serial, Par level. Each absent optional field shows its own `pill` "not tracked" (`not-tracked`), each present field its value; rows are never blank. Data is fetched client-side on route mount, so returning from `/stock/new` via SPA navigation shows the new row with no full-page reload. No records: the table grid is NOT rendered; an `empty-state` (icon, "No stock filed yet", teaching copy, `btn--primary` CTA to `/stock/new`) shows instead. No interactive filter is introduced by this story | `stock-table`, `stock-row`, `location-group`, `stock-cell-sku`, `stock-cell-location`, `stock-cell-quantity`, `stock-cell-tracking-code`, `stock-cell-batch`, `stock-cell-serial`, `stock-cell-par-level`, `not-tracked`, `empty-state`, `empty-state-cta` |
| Record SKU detail (F1-S3) | `/sku/:sku` (`?location=`) | Refines SKU detail for one (SKU, location) record, opened by clicking a home `stock-row` (whole row is a link, SPA navigation, no full-page reload). `page--narrow` `card` with a definition list: SKU, Location, Quantity (mono), Tracking code, Batch, Serial, Par level; shows ONLY the record matching `:sku` + `?location=`, nothing from other records. Each absent optional field shows its own `pill` "not tracked" (`detail-not-tracked`), each present field its value; never blank. Back link to `/`. No interactive filter introduced | `sku-detail`, `detail-sku`, `detail-location`, `detail-quantity`, `detail-tracking-code`, `detail-batch`, `detail-serial`, `detail-par-level`, `detail-not-tracked`, `detail-back` |
| Confirmation | `/confirmation/:id` | Landing view after a successful form submit (navigation is the feedback) | `confirmation` |

Forms and detail use the `page--narrow` variant; home uses the 960px page.

## Navigation
Navbar on every screen: brand (warehouse icon + "StockFlow") links to `/`. Links: Stock (`/`), Receive (`/receive`), Pick (`/pick`), Adjust (`/adjust`), Count (`/count`), File record (`/stock/new`). Active link in brand red. Home rows link to `/sku/:sku?location=<location>` (the record's own key). Empty-state CTAs link to `/receive` (the F1 no-stock empty state's `empty-state-cta` links to `/stock/new`); home also carries a "File stock record" `btn--secondary` linking to `/stock/new`. Every route is declared in `App.tsx` `<Routes>` and reachable from the navbar or a row link. The search lives on home (F9, read-only), not a separate route; an empty result shows the empty state "No stock matches that SKU or location".

## F6 / F10 surfacing
- Stage label sits beside the home title and the detail title. F6 states: "Today" (before) and "Transition - to verify" (after expand + backfill). F10 shows the contracted end state with no combined-code column.
- F6 home and detail: Batch and Serial columns carry the new-to-verify pill; the combined tracking-code column carries the to-be-retired pill. NULL batch/serial shows the "not tracked" pill, never blank.
- The integrity-probe nonconforming-row count is shown in the app on home (and detail where relevant) for PO sign-off.
- The migration overlay covers the screen while a stepwise step (reset, expand + backfill, verify) runs and clears when it finishes.
- F10: the combined tracking-code column and its pill disappear; Batch and Serial show without the new-to-verify pill once signed off, "not tracked" where empty.

## User flows
1. View stock: open `/`, scan the table, search by SKU or location, open a SKU detail.
2. Receive: `/receive`, submit, land on confirmation; invalid SKU shows an inline field-named error.
3. Pick: `/pick`, submit; an overcommit shows an inline quantity error.
4. Adjust: `/adjust`, submit, inline green flash and success toast.
5. Count: `/count`, enter counted quantity, see variance, reconcile.
6. Scan (F8): scan a barcode on any write form; success flashes the zone green and the row updates in place, failure flashes red with a persistent toast.
7. F6 demo walk: reset, expand + backfill (overlay), verify (stage label, pills, probe count) on the isolated branch.
8. File stock record (F1-S1, AC1-AC5): open `/stock/new` from the navbar or home CTA, enter SKU, location, quantity and optional tracking code / batch / serial, submit; the inline success status confirms without reload (AC1); blank optional detail is stored as null (AC4); blank SKU / location / quantity each shows its field-named inline error and nothing is stored (AC2); a quantity like -3 or 'abc' shows the quantity inline error (AC3); re-filing the same (SKU, location) with new values shows "Record updated" and the home table shows one row with the new values (AC5).
9. Browse stock by location (F1-S2, AC1-AC4): open `/` and read rows grouped by location (AC1); absent optional fields read "not tracked" per field (AC3); with no records see the teaching empty state and follow its CTA to `/stock/new` (AC4); file a record there, return to `/` via the navbar Stock link and see the new row with no full-page reload (AC2).
10. Open record detail (F1-S3, AC1-AC2): on `/` click the row for (S, L); `/sku/S?location=L` opens without a full-page reload and shows only that record's SKU, location, quantity and tracking code (AC1); absent optional fields read "not tracked" per field, present ones show values (AC2); `detail-back` returns to `/`.
