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
reports green, clone this starter and stand it up as your own project:

```bash
git clone https://github.com/kevin-hartman/stockflow-starter-project.git
cd stockflow-starter-project
git checkout staging
(cd client && npm install)
./setup.sh my-stockflow          # your project name (kebab-case)
```

`./setup.sh` creates a fresh Lakebase project (prod + staging), re-attaches git to a new repo of
your name, regenerates `.env` against your workspace, and **leaves you on `staging` at the plan
gate** — no feature branch, nothing to claim. (It handles no-feature starters, single-feature, and
parallel multi-feature snapshots alike; see
[`RUNBOOK-rehydrate-as-new-project.md`](RUNBOOK-rehydrate-as-new-project.md) for the details and
gotchas.)

Then resume and design the first feature — the backlog is already authored, so there's no sprint to
plan:

```
/consort:start            # resumes at the plan gate (the F1–F10 backlog is already authored)
/design F1-stock-visibility
```

`F1-stock-visibility` is the foundation feature (record + view stock by SKU and location) and the
natural place to start; `/design` reads its `feature-request.md` and never overwrites it.
