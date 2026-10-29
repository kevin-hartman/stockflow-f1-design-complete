# StockFlow starter project

A ready-to-run **starter snapshot** of StockFlow — a warehouse stock-management app for
mid-market operations. The whole up-front product intent is already authored and the backlog
is proposed and sized, **parked at the plan gate**: no feature has been designed or built yet.
Clone it, stand it up as your own Lakebase-paired project, and start by designing the first
feature — instead of writing intake from a blank page.

It's built with **[Consort](https://github.com/databricks-solutions/consort)**, a spec-first,
test-driven agent framework for apps on **Lakebase** (Databricks serverless Postgres).

## What's in it

- **Intake** — `product-overview.md`, `nfrs.md`, and the UX design system (`design/design-brief.md`,
  `design/design-guide.*`, `design/ia.md`, `client/src/styles/*`).
- **Planning** — a proposed, t-shirt-sized backlog (`planning/feature-proposals.md` +
  `estimates.json`), parked at the **plan gate**.
- **Ten feature-request folders** (`features/F1-stock-visibility` … `F10-retire-tracking-code`) —
  the committed backlog, one open-ended ask per feature, that `/design` reads.
- **Two branches:** `main` (the trunk) and `staging` (the working tier). There is **no feature
  branch yet** — the project lives on `staging` until you design the first feature.

## Stand it up as your own project

**Prerequisite:** Consort installed and a Lakebase-enabled Databricks workspace. If you haven't set
that up, run:

```bash
bash <(curl -sL https://tinyurl.com/consort-bootstrap)
```

That installs the Consort plugin, pre-downloads the toolkit, and runs the environment doctor. Once it
reports green, clone this starter:

```bash
git clone https://github.com/kevin-hartman/stockflow-starter-project.git
cd stockflow-starter-project
git checkout staging
(cd client && npm install)
```

Then stand it up as your own project — replace `my-stockflow` with your project name (kebab-case):

```bash
./setup.sh my-stockflow
```

`./setup.sh` creates a fresh Lakebase project (prod + staging), re-attaches git to a new repo of
your name, regenerates `.env` against your workspace, and **leaves you on `staging` at the plan
gate** — no feature branch, nothing to claim. (It handles no-feature starters, single-feature, and
parallel multi-feature snapshots alike; see
[`RUNBOOK-rehydrate-as-new-project.md`](RUNBOOK-rehydrate-as-new-project.md) for the details and
gotchas.)

Then start the workflow. First, resume the project:

```
/consort:start
```

It picks up where the starter leaves off and is **waiting for your approval** at the plan gate. Once
you approve, begin building — either one feature at a time:

```
/design F1-stock-visibility
```

or the whole sprint at once:

```
/sprint
```

`F1-stock-visibility` is the foundation feature (record + view stock by SKU and location) and the
natural place to start; `/design` reads its `feature-request.md` and never overwrites it.
