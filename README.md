# bench:primitives

A public, reproducible benchmark for code-intelligence MCP servers and code-search baselines.

**90 hand-verified tasks** across **3 OSS codebases** (sverklo, express 4.21.1, lodash 4.17.21), **4 task categories** (definition lookup, reference finding, file dependencies, dead code), **5 baselines** (naive grep, smart grep, [sverklo](https://github.com/sverklo/sverklo), [jcodemunch-mcp](https://github.com/jgravelle/jcodemunch-mcp), [GitNexus](https://github.com/abhigyanpatwari/GitNexus)).

Reproducible from a fresh clone with one npm script.

## Why this repo exists

When you publish a benchmark and your competitor responds by shipping fixes within hours, that's the bench working as intended. This repo is the eval surface that loop runs on. It's split out from [github.com/sverklo/sverklo](https://github.com/sverklo/sverklo) (the primary code-intelligence MCP server) so the methodology gets its own audit trail, its own contributor PRs, and its own credibility signal independent of the tool that wrote it.

If you build a code-intelligence MCP server, code-search tool, or retrieval system in this neighborhood, **you can submit a baseline implementation here** and we'll run it on the same task suite. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Headline results (May 2026, sverklo v0.20.2)

| baseline | n | F1 | P1 (def lookup) | P2 (ref finding) | P4 (file deps) | P5 (dead code) | avg input tokens | tools/task |
|---|---|---|---|---|---|---|---|---|
| naive-grep | 90 | 0.29 | 0.10 | 0.18 | 0.53 | 0.67 | 20,278 | 6.5 |
| smart-grep | 90 | 0.49 | 0.43 | **0.40** | 0.59 | 0.67 | 1,220 | 4.9 |
| **sverklo** | 90 | **0.56** | 0.73 | 0.25 | **0.71** | 0.67 | 469 | **1.0** |
| jcodemunch | 90 | 0.32 | **0.73** | 0.00 | 0.46 | 0.00 | 1,267 | 1.2 |
| gitnexus | 90 | 0.25 | 0.27 | 0.00 | 0.30 | 0.67 | **372** | 1.2 |

**Headline:** Sverklo leads overall F1 (0.56 vs smart-grep 0.49). Sverklo and jcodemunch tie on definition lookup (0.73). Smart-grep wins reference finding (0.40, with sverklo at 0.25 — the only category where a tuned ripgrep beats a hybrid retriever). Sverklo wins file-dependency reasoning (0.71). All tools tie at 0.67 on dead code on the empty-expected pattern.

[Full per-category breakdown and the slice where each baseline loses →](https://sverklo.com/bench/)

## Methodology

- Tasks are **hand-verified** at ground-truth construction time. Each P1, P2, P4 task points at exact `(file, line)` tuples, validated by reading the file. P5 tasks list expected dead-code names, validated against the actual codebase.
- Tolerances: P1 uses **±3-line** tolerance (parsers disagree on "def line"). P2 uses **±2 lines**. P4/P5 use **set membership**.
- Token cost is measured as **input tokens to the consuming agent**, including all tool-call results. The metric that matters in production is "how many tokens does the agent eat to get an answer," not "how big is the index file."
- The "honesty section" of the writeup lists every task where sverklo loses to a baseline. Same for every baseline. No cherry-picking.
- Cold-start is reported separately from warm-call latency. Both are loud.

Full methodology document: [METHODOLOGY.md](./METHODOLOGY.md).

## Reproducing the bench

The runtime lives in the main sverklo monorepo:

```bash
git clone https://github.com/sverklo/sverklo.git
cd sverklo
npm install
npm run build
npm run bench:quick
```

Results land in `benchmark/results/<timestamp>/` as `raw.jsonl` (per-task records), `summary.json` (aggregate per baseline + category), and `report.md` (human-readable).

To run a single baseline: `BASELINES=sverklo,jcodemunch npm run bench:quick`.

To re-validate a competitor's published claim, point the harness at their fork and re-run. The task definitions in this repo are authoritative; the harness is a reference implementation.

## Task definitions (copies for reference)

The authoritative ground truth lives at [github.com/sverklo/sverklo/tree/main/benchmark/src/ground-truth/seed](https://github.com/sverklo/sverklo/tree/main/benchmark/src/ground-truth/seed). Mirrored here as a stable read-only reference:

- [tasks/sverklo.jsonl](./tasks/sverklo.jsonl) — 30 tasks against the sverklo monorepo (TS)
- [tasks/express.gen.ts](./tasks/express.gen.ts) — 30-task generator against express 4.21.1 (CommonJS, modular)
- [tasks/lodash.gen.ts](./tasks/lodash.gen.ts) — 30-task generator against lodash 4.17.21 (single-file UMD/IIFE — different shape)

Datasets covered:
- **sverklo monorepo** — TypeScript, modular, the most representative real-world repo (it's the project's own codebase)
- **express 4.21.1** — JavaScript, modular CommonJS, well-known structure
- **lodash 4.17.21** — JavaScript, single 17K-line UMD/IIFE wrapper. Pathological for parsers — exposed blind spots in both jcodemunch (size cap, IIFE call-graph fallback) and sverklo (regex brace counter mis-counting inside string literals) within 36 hours of being added to the bench.

## How to add a baseline

If you maintain a code-intelligence tool and want it benchmarked here, see [CONTRIBUTING.md](./CONTRIBUTING.md). The TL;DR: implement the `Baseline` interface in the main sverklo repo (one ~150-line file mapping task categories to tool calls), open a PR, and we'll cross-link it here.

## License

MIT. Same as the [main sverklo repo](https://github.com/sverklo/sverklo).

The bench task files are CC-BY-4.0 — reuse them in academic work, in your own evaluations, or to argue with our methodology. Cite the [Zenodo paper](https://doi.org/10.5281/zenodo.19802051) where applicable.

---

**Related:**
- [github.com/sverklo/sverklo](https://github.com/sverklo/sverklo) — the MCP server being benchmarked (the "sverklo" baseline above)
- [sverklo.com/bench/](https://sverklo.com/bench/) — rendered results page with the per-category breakdowns
- [Issue #25](https://github.com/sverklo/sverklo/issues/25) — original "compare against jcodemunch and GitNexus" thread that drove the May 2026 expansion
- [Zenodo paper, CC-BY 4.0](https://doi.org/10.5281/zenodo.19802051) — peer-reviewable methodology writeup
