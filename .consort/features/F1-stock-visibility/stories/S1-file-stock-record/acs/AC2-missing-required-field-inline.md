# AC2-missing-required-field-inline

- **Given** the stock-filing form is open
- **When** the operator submits with a required field (SKU, location, or quantity) left blank
- **Then** an inline error naming that field is shown next to it (never a generic 'bad request'), the entered values are kept, and no record is created
