---
name: cortex-eval
description: >
  This skill should be used when the user wants to evaluate cortex search quality, run the golden
  query set, compare results against a saved baseline, or verify that a chunker/indexing change
  improved hit@k or MRR metrics. Triggers on "run eval", "check search quality", "did the fix help",
  "compare baseline", "what are the eval results", or after any reindex following a chunker change.
---

# /cortex-eval

To evaluate cortex search quality: run the golden query set against the current index and interpret hit@k + MRR results.

## Prereqs

- Must be in `/root/projects/cortex`
- Requires a full reindex before meaningful results (stale index → stale metrics)
- Qdrant must be reachable at `http://192.168.68.103:6333`

## Commands

```bash
cd /root/projects/cortex

# Run against current index (direct Qdrant on portainer-lxc)
python3 eval/run.py

# Save results as a named baseline
python3 eval/run.py --save <name>

# Compare current results against a saved baseline
python3 eval/run.py --compare <name>

# Run via HTTP API (deployed server)
python3 eval/run.py --url https://cortex.hyvitech.org/api --token <token>
```

## Metrics

| Metric | Meaning | Good |
|--------|---------|------|
| hit@1 | Top result matches expected file | >70% |
| hit@3 | Expected file in top 3 | >90% |
| hit@5 | Expected file in top 5 | >95% |
| MRR | Mean reciprocal rank | >0.80 |

Golden set: `eval/golden.json` — 21 queries, each with expected file match(es).
Baselines: `eval/baselines.json` — named snapshots for comparison.

## Baseline at v2.1.6 (pre-reindex, pre-framework-tokens)

hit@1 61.9%, hit@3 85.7%, hit@5 95.2%, MRR 0.7516

## Workflow: Verifying a Chunker Change

1. Ensure full reindex completed after deploying the new image
2. Run `python3 eval/run.py` — compare vs baseline above
3. If improved, save new baseline: `python3 eval/run.py --save v<VERSION>`
4. Update `notes/projects/cortex.md` with new baseline numbers

## Interpreting Results

- hit@1 is the most sensitive metric — most affected by chunk header quality
- MRR captures rank position, not just presence — a result appearing 3rd vs 1st matters
- If hit@5 drops, something regressed badly (embedding or collection issue)
- If hit@1 improves but MRR stays flat, top result improved but mid-results didn't

## Adding Queries to the Golden Set

Edit `eval/golden.json`. Each entry:
```json
{
  "query": "server side hooks SvelteKit",
  "expected": ["cortex/server/mcp/hooks.server.ts"]
}
```

Use `file` paths relative to repo root. Run eval after adding to confirm the query is actually findable.
