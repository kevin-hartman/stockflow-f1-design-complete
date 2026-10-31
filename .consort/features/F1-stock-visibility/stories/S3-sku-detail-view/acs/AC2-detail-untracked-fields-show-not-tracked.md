# AC2-detail-untracked-fields-show-not-tracked

- **Given** a record (SKU-200, A-01, qty 5) filed without a tracking code (stored null) and with no par level
- **When** the operator opens that record's detail view
- **Then** the tracking-code field and the par-level field each read 'not tracked', judged independently, while SKU, location and quantity are shown; no field is blank or reads 'null' and the view renders cleanly
