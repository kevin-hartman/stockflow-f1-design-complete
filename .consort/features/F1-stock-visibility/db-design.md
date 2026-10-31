# F1 db-design

Table `stock_records` (mirrors `app/models/stock_record.py`): uuid PK, sku/location/quantity NOT NULL, tracking_code nullable, timestamps.

## Migration plan
- S1-file-stock-record: SC1-create-stock-records (create_table; reversible via drop/recreate).
- S2-stock-by-location-table: SC2-add-location-sku-index (add_index; downgrade drops it).

## Invariants
- PI1: UNIQUE (sku, location) `uq_stock_records_sku_location`; upsert uses ON CONFLICT (sku, location) DO UPDATE.
- PI2/PI3/PI4: NOT NULL on sku, location, quantity.
- PI5: CHECK `ck_stock_records_quantity_non_negative` (quantity >= 0).
- PI6: downgrade drops table; upgrade head recreates columns, constraints, check.

## Access patterns
- AP1-list-ordered-by-location: index `ix_stock_records_location_sku` on (location, sku) serves the ordered listing; the (sku, location) unique index leads with sku and can't.
