# StockFlow – F1 design-complete snapshot

## What this is

A **ready-to-build snapshot** of StockFlow, a warehouse-management app for a mid-market
operation that has outgrown spreadsheets. **V1** lets the team see and adjust stock at one
warehouse – file a SKU's stock at a location, see it in a scannable by-location table, and drill
into a SKU's detail – as a React single-page client on a JSON API.

It's built with **[Consort](https://github.com/databricks-solutions/consort)**, a spec-first,
test-driven agent framework for apps on **Lakebase** (Databricks serverless Postgres). Consort
normally walks a feature through two lanes: a *design* lane (plan → spec → UX → architecture →
tests) and a *build* lane (TDD: RED → GREEN → REFACTOR).

**The purpose of this repo is to skip the design lane and go straight to build.** The planning
and the entire F1 design lane are already authored and frozen at their gates – so you can
stand this up as your own project and start the TDD build loop immediately, without redoing
any of the up-front design work.

## Where we are

Design is complete; no build code has been written yet.

- **Planning:** complete. Sprint `v1` is locked, with the F1–F10 backlog proposed and sized.
- **Feature F1 – "Stock visibility":** design complete – feature-spec, UX design-guide,
  architecture, db-design, and the ordered test-list are all authored and gated (in `.consort/`).
  All three stories are at an approved spec gate, ready to build:
  - `S1-file-stock-record` – file a stock record (SKU, location, quantity, tracking code) with inline fields
  - `S2-stock-by-location-table` – see a scannable stock-by-location table on the home view
  - `S3-sku-detail-view` – click a row to see that record's SKU detail across its locations

## How to use it – jump straight to build

**Prerequisite:** Consort is installed, your environment is prepped, and access to a Lakebase
profile is ready to go. If you haven't set that up yet, run:

```bash
bash <(curl -sL https://tinyurl.com/consort-bootstrap)
```

If you need Lakebase, you can try it on [Free Edition](https://login.databricks.com/signup?intent=SIGN_UP&provider=DB_FREE_TIER).

### 1. Clone it down

```bash
git clone https://github.com/kevin-hartman/stockflow-f1-design-complete.git
cd stockflow-f1-design-complete
git checkout feature-f1-stock-visibility
(cd client && npm install)
```

`feature-f1-stock-visibility` is the branch carrying the F1 design state; the last line
installs the UI client dependencies.

### 2. Stand it up as *your* project

Run the setup script, replacing `my-stockflow` with your own project name (kebab-case):

```bash
./setup.sh my-stockflow
```

It creates a fresh Lakebase project, re-attaches git to a new repo of your name, regenerates
`.env` against your workspace, and leaves you on the F1 feature branch with its paired Lakebase
branch ready. If you hit a snag, read the [runbook](RUNBOOK-rehydrate-as-new-project.md) and
share it with a Claude session to resolve.

### 3. Build the first feature

In your Claude session, resume the workflow:

```
/consort:start
```

F1's design is approved, so `/consort:start` picks up at the **build** lane. Then build it:

```
/build F1-stock-visibility
```

It runs the Driver/Navigator TDD loop (RED → GREEN → REFACTOR) against F1's Lakebase branch —
each cycle writes a failing test, greens it, and refactors, against a real database.
