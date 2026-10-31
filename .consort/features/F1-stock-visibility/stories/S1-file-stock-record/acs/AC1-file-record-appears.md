# AC1-file-record-appears

- **Given** the stock-filing form is open and no record exists for SKU 'SKU-100' at location 'A-01'
- **When** the operator enters SKU 'SKU-100', location 'A-01', quantity 12 and a tracking code, and submits the form
- **Then** the submission succeeds without a full-page reload, the operator sees confirmation, and a stock record for (SKU-100, A-01) with quantity 12 and that tracking code exists
