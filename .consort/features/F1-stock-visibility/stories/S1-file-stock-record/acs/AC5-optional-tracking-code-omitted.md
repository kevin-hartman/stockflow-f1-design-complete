# AC5-optional-tracking-code-omitted

- **Given** the stock-filing form is open
- **When** the operator submits a valid SKU, location and quantity with the tracking code left empty
- **Then** the submission is accepted and the record is stored with no tracking code (null), so downstream views can show it as 'not tracked'
