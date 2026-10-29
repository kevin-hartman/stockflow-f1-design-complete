# test-strategist (test-strategist) – sonnet

## Prompt

```
REVISE (reflection gate): blocking smell "reflect-testlist-defect": reflection gate: T51/T52/T53/T54 require a genuinely empty stock_records table ('no records on the live branch'), but the feature branch DB is shared with S1/S2 specs that seed randomUUID-keyed rows and the build forbids DELETE teardown. The empty state can never be reached reliably on a correct app (passes alone, fails at full-suite verify). Re-author so the empty response is obtained without depending on a globally empty table (e.g. Playwright page.route scoped by URL pathname === '/api/stock-records' returning [] for the empty-state specs, and a component test that supplies an empty list), keeping the real-DB path for populated tests.

This is a DIRECTED test-list fix — apply the SPECIFIC change named above to the flagged item(s) in the existing ordered list (your directive has the exact edit steps); do NOT re-run the fan-out, and keep every OTHER item + its T-id byte-identical. The named fix is REQUIRED: make the exact change described – do NOT omit it, weaken it, or defer it to an open question. If it genuinely cannot be made as written, name the concrete blocker; do not punt.

DIRECTED test-list revise — apply ONLY the specific fix named in your revise brief (the flagged AC(s): AC4-empty-state-when-no-stock). Do NOT run the fan-out (consort-fan-out-analysts): the test list is already authored and the fix is targeted, so a re-derive would re-roll the whole scope nondeterministically and lose coverage. The current ordered list is on disk — EDIT it in place.
1. Open <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S2-stock-by-location-table/analyst-ordered.json and apply the brief's named change to ONLY the item(s) it calls out: change/re-slice/relabel the exact test(s) named, or ADD the single test the brief requires. Keep every OTHER item and its T-id byte-identical.
2. APPEND the edited <PROJECT_ROOT>/.consort/features/F1-stock-visibility/stories/S2-stock-by-location-table/analyst-ordered.json items to the feature master <PROJECT_ROOT>/.consort/features/F1-stock-visibility/test-list.json, keeping every OTHER story's items (the master's prior copy of THIS story's items was cleared for the revise); never author a test-list-per-story.json (the orchestration regenerates the per-story + per-AC views from the master). (Story S2-stock-by-location-table's ACs are AC1-table-lists-records-by-location, AC2-new-record-appears-without-reload, AC3-untracked-fields-show-not-tracked, AC4-empty-state-when-no-stock; every appended item's ac_id must be one of these.)
3. Self-check: ./scripts/lk consort-response-formatter --role test-strategist --feature F1-stock-visibility --story S2-stock-by-location-table — fix + re-run until it passes. Be terse: produce ONLY the required artifact file(s) on disk, then stop with at most a one-line confirmation. Do NOT print a plan, a summary of what you did, rationale, tables, or restate the artifacts to stdout, that output is wasted latency. The files on disk are the deliverable, not your prose.
```

## Tools used

- Bash {"command":"python3 -I -c \"\nimport json\nb='.consort/features/F1-stock-visibility/'\np=b+'stories/S2-stock-by-location-table/analyst-ordered.json'\nn=json.load(open(p))\nD={\n'T51':'Home stock view rendered alone with an empty record list supplied directly (no globally empty table required) shows the empty-state with its empty-state-cta pointing to /stock/new',\n'T52':'Home stock view rendered alone with an empty record list supplied directly renders no stock-table grid',\n'T53':'Playwright e2e opens route / with page.route scoped to url.pathname === \\'/api/stock-records\\' fulfilling an empty JSON array [] (the shared branch DB is never assumed empty) and sees the empty-state teaching message with no stock-table grid rendered',\n'T54':'Playwright e2e with page.route scoped to url.pathname === \\'/api/stock-records\\' fulfilling [] clicks the empty-state-cta on the empty home view and lands on the /stock/new filing form (unrouted /stock/new filing form still hits the real backend)'}\nfor i in n['items']:\n    if i['id'] in D: i['description']=D[i['id']]\njson.dump(n,open(p,'w'),indent=2)\nm=json.load(open(b+'test-list.json'))\nids={i['id'] for i in n['items']}\nm['items']=[i for i in m['items'] if i['id'] not in ids]+n['items']\njson.dump(m,open(b+'test-list.json','w'),indent=2)\n\"; ./scripts/lk consort-response-formatter --role test-strategist --feature F1-stock-visibility --story S2-stock-by-location-table 2>&1 | tail -5","description":"Re-author empty-state items and re-append"}

## Final reasoning

I rewrote T51–T54 so none of them needs an empty `stock_records` table, and the self-check passes. S2's items are re-appended to the master.

- **T51 and T52:** the component tests now supply an empty record list directly.
- **T53 and T54:** the Playwright specs route `/api/stock-records` (matched on `url.pathname`) to return `[]`.

Every other item and T-id is unchanged.

T50 has the same problem and I did not touch it. It asserts that `GET /api/stock-records` returns `[]` when no records exist, which also needs a globally empty table. Your brief named only T51–T54, so I left it alone. It will likely fail at full-suite verify on the shared branch DB and probably needs the same kind of fix.
