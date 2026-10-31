# AC1-row-opens-detail

- **Given** the home table lists records (SKU-100, A-01, qty 12, tracking code A01-B7-S0042) and (SKU-100, B-02, qty 7)
- **When** the operator selects the (SKU-100, A-01) row
- **Then** a detail view for exactly that record opens by in-app navigation with no full-page reload, showing SKU-100, location A-01, quantity 12 and tracking code A01-B7-S0042, and not the values of the (SKU-100, B-02) record
