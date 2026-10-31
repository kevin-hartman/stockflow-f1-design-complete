# AC4-same-key-updates-in-place

- **Given** a stock record exists for (SKU-100, A-01) with quantity 12
- **When** the operator files a record for the same SKU-100 at A-01 with quantity 20
- **Then** the existing record is updated in place to quantity 20, exactly one record exists for (SKU-100, A-01), and no duplicate is created
