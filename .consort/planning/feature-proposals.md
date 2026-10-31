# Feature proposals — sprint-1

Source: `product-overview.md` (V1 + feature roadmap) and `nfrs.md`. UI track ON
(`ui_track: true`), so every user-facing candidate is framed as a browser
increment and carries an **E2E (UI) story** need.

Sprint scope: **sprint 1 only**. The roadmap fixes F1 as the sprint-1 increment
and the foundation every later feature writes into. Nothing has been delivered
yet. Later roadmap features are listed under *Deferred* with their canonical ids
so the backlog lines up with them; they are **not** proposed for this sprint.

---

## F1-stock-visibility

- **Ask:** As a warehouse operator / inventory manager, record a stock level for
  a SKU at a location and view stock by SKU and location in the StockFlow SPA
  (home stock-by-location table + SKU detail), each unit carrying the combined
  tracking code.
- **Rationale:** V1 "file, retrieve, and adjust the stock level of one SKU at one
  location" + "hold stock for the same SKU at multiple locations"; roadmap item 1
  (foundation, introduces the combined tracking code). NFRs: R3 (unique
  `(sku, location)`, a re-file of the same pair is resolved at write time by
  updating in place, never a duplicate or an error page), R5 (React + TS SPA,
  client-side navigation, JSON API), R7 (inline field-named validation), R9
  ("not tracked" per optional field, explicit empty state), R1 (records survive
  later migrations), R4 (real-branch integration tests).
- **Priority:** P0 (must-have; sprint 1, blocks every other feature).
- **E2E (UI) story needed:** **Yes.** Candidate story slices (the breakdown
  finalizes them):
  - file/record a stock level through the SPA form, see it in the table, and see
    the same `(sku, location)` updated in place when re-filed, with an optional
    tracking code that can be left out;
  - view the home stock-by-location table, including the empty state;
  - open a SKU's detail and see its stock across multiple locations, with "not
    tracked" shown per optional field.
- **Note:** The SPA shell, navbar, and StockFlow brand that the roadmap says F1
  "establishes" come from the design system (`design-guide.json` / `ia.md`) and
  are applied by these capability stories. They are **not** a separate
  app-shell/branding story.

---

## Deferred: later sprints (not proposed for sprint 1)

Listed only so the backlog matches the roadmap. Each one is re-proposed at its
own `/plan` once the previous sprint's learning is folded in.

| id | sprint (roadmap) | one-line ask | E2E (UI) story |
|---|---|---|---|
| F2-stock-adjustment | 2 | Correct a level in place; audited (timestamp + typed actor), never negative (R1, R2, R5) | Yes |
| F3-inbound-receipt | 2 | Receive a known supplier's quantity to a chosen location (R2, R7) | Yes |
| F4-outbound-pick | 2 | Pick for an order; reject an overcommit with a field-named message (R2, R7) | Yes |
| F5-cycle-count | 2 | Count a shelf, show the variance, reconcile | Yes |
| F6-split-tracking-code-expand | 2 | Add batch + serial and backfill them by delimiter; keep the combined code; show the integrity-probe count in the app (R1, R3, R8) | Yes (pills, probe count, stage labels) |
| F9-stock-search | 2 | Read-only SKU/location search on home, with a no-match empty state (R10) | Yes |
| F7-multi-warehouse | 3 | Scope stock to a warehouse; existing rows go under a default warehouse | Yes |
| F8-barcode-scan | 3 | Scan-driven receive/pick/adjust/count with in-place feedback under 200ms (R6) | Yes |
| F10-retire-tracking-code | 3 | Drop the combined code (reversible round-trip); gated on F6 being signed off | Yes |

---

## Open questions (for the PO)

1. **Search timing (R10 vs roadmap):** R10 calls findability "a stated V1 need".
   The roadmap puts F9-stock-search in sprint 2. Keep it in sprint 2 (recommended,
   because the roadmap's sprint numbers are fixed), or pull a minimal home filter
   into sprint 1?
2. **"Adjust" in V1 vs F2:** V1 lists "file, retrieve, and **adjust**". Does
   sprint-1 F1 only re-file a level (update in place on the same
   `(sku, location)`), with audited adjustment left entirely to F2-stock-adjustment?
   Recommended: yes.
3. **Barcode sign-off:** The overview says an operator should "scan a real
   barcode" before sign-off, but scanning is F8 (sprint 3). For sprint 1, is
   typed/keyboard-wedge entry of the tracking code acceptable? Recommended: yes.

---

## PO decisions (recorded at the backlog gate)

- **Backlog scope this round:** Author a `feature-request.md` for **all ten**
  roadmap features now (F1–F10) so the whole backlog is groomed and ready, then
  stop at the plan gate (no design/build this round). Each request keeps its
  **roadmap sprint** in frontmatter: F1 → sprint 1; F2, F3, F4, F5, F6, F9 →
  sprint 2; F7, F8, F10 → sprint 3. F10 remains gated on F6 being deployed and
  signed off.
- **Q1 — Search (R10):** F1 excludes filtering. All findability is delivered by
  **F9-stock-search** in sprint 2 — a read-only SKU/location filter on the home
  table with a no-match empty state.
- **Q2 — Adjust:** F1 **records / re-files a stock level in place** on the same
  `(sku, location)` only (R3, update-in-place). The audited, never-negative
  adjustment write path (immutable timestamp + actor) is entirely
  **F2-stock-adjustment**, not F1.
- **Q3 — Barcode:** **Typed / keyboard-wedge entry** of the combined tracking
  code is acceptable for F1 (sprint 1). True scan-driven flows (green/red
  scan-zone flash, <~200ms perceived, R6) are **F8-barcode-scan** (sprint 3).
