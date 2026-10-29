# architect-reviewer-estimate-committed (architect-reviewer) – opus

## Prompt

```
Estimate the sprint's COMMITTED feature(s) with a t-shirt size (XS/S/M/L/XL). Read each committed feature's request at <PROJECT_ROOT>/.consort/features/<F>/feature-request.md, then ADD one entry per committed feature to <PROJECT_ROOT>/.consort/planning/estimates.json keyed by its REAL feature id (e.g. "F1-stock-visibility", not a "FP" candidate id), each {"feature_id":"<F>","size":"<XS|S|M|L|XL>","rationale":"<why>"}. KEEP every existing estimate already in the file (merge, do not overwrite the candidate sizes). This is the size sync-backlog stamps into the per-sprint backlog, so the committed backlog shows real sizing. Be terse: produce ONLY the required artifact file(s) on disk, then stop with at most a one-line confirmation. Do NOT print a plan, a summary of what you did, rationale, tables, or restate the artifacts to stdout, that output is wasted latency. The files on disk are the deliverable, not your prose.
```

## Tools used

- Bash {"command":"cd <PROJECT_ROOT>/.consort && ls features planning; cat planning/estimates.json; cat planning/backlog*.json 2>/dev/null | head -50; ls planning/*","description":"Survey features, estimates, and backlog"}
- Bash {"command":"ls; ls sprints 2>/dev/null; find . -name \"backlog*.json\" -maxdepth 4 | head; head -40 features/F6-split-tracking-code/feature-request.md","description":"Locate committed backlog and read F6 request"}
- Bash {"command":"ls sprints/; grep -h -o '\"id\": *\"F[^\"]*\"\\|\"feature_id\": *\"F[^\"]*\"' sprints/*/backlog.json | sort -u; cat next.json | head -30","description":"Identify committed features across sprints"}
- Bash {"command":"python3 -I -c '\nimport json;p=\"planning/estimates.json\";d=json.load(open(p))\nids={e[\"feature_id\"] for e in d[\"estimates\"]}\nif \"F6-split-tracking-code\" not in ids:\n  d[\"estimates\"].append({\"feature_id\":\"F6-split-tracking-code\",\"size\":\"L\",\"rationale\":\"Additive expand migration with delimiter backfill (NULL on nonconforming), integrity probe surfaced in app, stage pills + migration overlay, parent-aware branch schema diff; live-data preservation risk.\"})\njson.dump(d,open(p,\"w\"),indent=2);print(len(d[\"estimates\"]))'","description":"Merge F6 committed estimate into estimates file"}

## Final reasoning

I added a size L estimate for `F6-split-tracking-code` to `.consort/planning/estimates.json`. That was the only feature without an entry under its real id. The other committed features already had one, and I kept all 10 existing estimates.
