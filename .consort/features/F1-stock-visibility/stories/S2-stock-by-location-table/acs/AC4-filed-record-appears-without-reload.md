# AC4-filed-record-appears-without-reload

- **Given** the home table shows existing stock and no record exists for (SKU-300, C-03)
- **When** the operator files a record for (SKU-300, C-03, qty 4) through the form and returns to the home view within the app
- **Then** the new (SKU-300, C-03) row appears in the table with quantity 4, reached by in-app navigation with no full-page reload
