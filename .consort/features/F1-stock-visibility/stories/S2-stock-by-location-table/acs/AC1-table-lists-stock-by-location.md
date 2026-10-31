# AC1-table-lists-stock-by-location

- **Given** stock records exist for (SKU-100, A-01, qty 12), (SKU-200, A-01, qty 5) and (SKU-100, B-02, qty 7)
- **When** the operator opens the home view
- **Then** a stock-by-location table shows exactly one row per record, each with its SKU, location, quantity and tracking code, ordered by location so all A-01 rows appear together before B-02
