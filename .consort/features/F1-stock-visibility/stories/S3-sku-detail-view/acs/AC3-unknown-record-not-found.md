# AC3-unknown-record-not-found

- **Given** no stock record exists for (SKU-999, Z-99)
- **When** the operator opens the detail view address for (SKU-999, Z-99) directly
- **Then** an explicit 'record not found' state naming SKU-999 at Z-99 is shown with a way back to the stock table, never a blank page, an empty detail view or a generic error
