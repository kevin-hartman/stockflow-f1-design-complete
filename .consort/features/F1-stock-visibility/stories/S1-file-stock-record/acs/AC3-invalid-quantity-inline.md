# AC3-invalid-quantity-inline

- **Given** the stock-filing form is open with SKU and location filled
- **When** the operator submits a quantity that is not a non-negative whole number (e.g. -3 or 2.5 or 'abc')
- **Then** an inline error naming the quantity field and stating the rule is shown next to it, and no record is created
