# AC2-untracked-fields-show-not-tracked

- **Given** a record (SKU-100, A-01) filed with a tracking code and a record (SKU-200, A-01) filed without one (tracking code null), and no par level filed for either
- **When** the operator opens the home view
- **Then** each optional field is judged independently per row: SKU-200's tracking-code cell reads 'not tracked' while SKU-100's shows its code, and the par-level cell reads 'not tracked' on both rows; no cell is blank and the row still renders cleanly
