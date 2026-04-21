# VIS Arena — Design Notes

_Draft v0.1 · compiled 2026-04-20_

Self-contained working notes on **VIS Arena**: a live, agent-driven arena for visualization storytelling that extends the VIS × GenAI workshop challenge. Pairs with the pitch deck at `vis-arena-deck.html` and the Prophet Arena research notes at `prophet-arena-notes.md`.

## TL;DR

Prophet Arena proved live arenas work as benchmarks for LLM forecasting. We do the same for visualization storytelling — but with a twist. In vis storytelling nobody agrees on what makes a good report; the rubric itself is an open research question. VIS Arena doesn't impose one. Instead, each participant submits a single agentic design that both **generates** reports and **peer-reviews** others' reports from the same underlying philosophy. One agent, two commands, one taste. The platform archives every artifact and every cross-review, surfaces consensus, and — later — anchors results with human preference. The arena becomes a data-generating apparatus for the evaluation question itself, not just another leaderboard.

## 1. Context — what the 2025 challenge looked like

At VIS × GenAI 2025, participants submitted agent designs that generated visualization reports (PDF / HTML) on a fixed dataset. Humans reviewed the outputs. That format was the right starting point: it pulled real agent designs onto the table and surfaced what was technically possible at the time. A year in, the seams are visible.

## 2. Three pain points the 2025 format exposed

**Manual review doesn't scale.** Every new submission, every new dataset, every iteration costs reviewer hours. The bottleneck is not the agents — it's the humans downstream. This caps how many agents can compete, how often the challenge can run, and how quickly the field accumulates evidence.

**Datasets stay narrow.** A single fixed dataset privileges agents tuned to its shape. You learn little about generalization. Agents that happen to fit the chosen data structure look stronger than they are.

**Rubrics drift.** "What makes a good report?" is genuinely open. Different reviewers applied different implicit standards — one weights narrative coherence, another weights chart correctness, another weights design polish. Without an auditable rubric, scores don't aggregate and the field doesn't converge.

## 3. Three tailwinds — what changed in a year

The agentic design surface expanded dramatically since the 2025 challenge.

- **Capability layer.** SKILL.md, MCP servers, Claude Agent SDK, OpenAI Agents, Claude Code — reusable skill modules, typed tool contracts, long-context planning loops that were bespoke in 2025 are now standard scaffolding.
- **Tool-routing layer.** Composio, Arcade, Nango, Tessl — single-endpoint routing over many domain tools. A CLI or data-viz library becomes one function call instead of a custom integration.
- **Reader layer.** Browserbase, Chrome DevTools MCP, Cloudflare Browser Run, WebMCP — agents can now read interactive reports the way a human would: scroll, click, hover, inspect.

Generation gets much richer. More importantly, *evaluation* does too — the same scaffolding lets an agent act as a reader/reviewer, not just a producer. That's the piece that unlocks the VIS Arena design.

## 4. The proposal

VIS Arena is a live platform where each submission is a single agentic design exposing two commands:

- `vis-arena generate` — runs the agent's generation pipeline on a supplied dataset + task, produces an interactive HTML report, a static export, and an **insights manifest** (a JSON list of claims with pointers into the data).
- `vis-arena review` — runs the agent's review pipeline on a peer's report bundle, produces structured critique along the agent's own dimensions with free-form rationale tied to specific regions of the report.

Both commands route through the *same* underlying agent. The philosophy that drives generation also drives review. Reviewing behavior becomes a **fingerprint** of the agent's evaluation philosophy — observable, not declared.

Per arena tick: a fresh dataset + task is released; every registered agent generates; every agent reviews every other agent's output; the platform aggregates the N×N review matrix into a consensus ranking; everything is archived with full provenance. Human preference is collected asynchronously on the archived bundles and layered in later as an external anchor.

## 5. Why this resolves the three pain points

Direct mapping, one line each.

- Manual review doesn't scale → **agent review does**. N×N cross-reviews cost compute, not reviewer hours. Human preference happens async on archived bundles.
- Narrow dataset → **streaming datasets per tick**, drawn from a rotating pool (open data portals, weekly releases, partner streams).
- Rubric drift → **rubric pluralism as a feature**. Every agent's review behavior reveals its rubric. Disagreement is surfaced, clustered, and studied — not hidden behind a single imposed scheme.

## 6. The research move — rubric pluralism

This is the intellectual core and what makes VIS Arena different from existing agent arenas.

Real academic peer review is rubric-pluralist: Reviewer 1 values rigor, Reviewer 2 values novelty, Reviewer 3 values clarity. Final acceptance aggregates heterogeneous views. VIS Arena mechanizes this for visualization storytelling and at agent-scale. Each submitted agent contributes not just generation capability but also a *point of view* on what counts as good — embedded in its review pipeline.

Consequences:

- The platform's scientific output is not only "Agent X wins." It's also a map of rubric space: where do reviewer agents cluster? Which dimensions correlate with human preference once we collect it? Which disagreements surface scientifically interesting reports?
- A participant can meaningfully submit a **review-philosophy-first agent** — essentially declaring "here's my theory of what makes vis storytelling good, let's see if the arena agrees." That's publishable on its own.
- Ablations previously impossible become trivial: hold the generator fixed, sweep reviewers; hold the reviewer fixed, sweep generators.

Lexara (arXiv 2603.05832) offers a useful **seed vocabulary** — visualization quality (data fidelity, semantic alignment, functional correctness, design clarity) and language quality (factual grounding, analytical reasoning, conversational coherence). Ship it with the SDK as a reference rubric that participants can extend, replace, or reject. Don't mandate it. Note that Lexara targets conversational visual analytics (CVA), so some dimensions translate cleanly to static/interactive reports while others (conversational coherence) need to be replaced with analogs like "cross-chart narrative flow."

## 7. Handling self-preference bias

Because the same agent generates and reviews, it will rate reports that look like its own generations higher. Documented in Zheng et al. (*Self-Preference Bias in LLM-as-a-Judge*) and Chen et al. (*Justice or Prejudice?*).

Design response: don't try to eliminate the bias. Measure it. For each reviewer agent, compute a **self-preference index** — the lift in score the reviewer gives its own generations (and their nearest neighbors in output space) over peer mean. Publish it alongside the leaderboard. This:

- Turns a flaw into a characterization dimension.
- Deters the most obvious gaming: an agent that rates itself and look-alikes too generously surfaces itself in the index.
- Connects the arena to the broader agent-as-judge literature as an empirical contribution.

## 8. Technical design — the submission schema

A submission is a small self-contained repo with a defined shape.

**Entry points** (two CLI commands, same agent code):

```
vis-arena generate \
  --dataset <path_or_url> \
  --task    <task_prompt> \
  --out     <bundle_dir>

vis-arena review \
  --bundle  <peer_bundle_dir> \
  --out     <review_json>
```

**Report bundle** (output of `generate`, input to `review`):

```
<bundle_dir>/
├── report.html              # interactive, self-contained
├── report.pdf               # static export for archival / fallback
├── screenshots/             # auto-captured key views
├── insights.json            # structured manifest of claims (see below)
├── agent.manifest.json      # agent id, version, model(s), tools used
└── run.log                  # provenance: dataset hash, task, timestamp
```

**Insights manifest** — the key contract. Without this, faithfulness isn't automatable.

```json
{
  "report_id": "...",
  "dataset_hash": "...",
  "insights": [
    {
      "id": "i1",
      "claim": "Q4 sales fell 12% YoY",
      "evidence": {
        "columns": ["quarter", "sales", "year"],
        "filter": "quarter == 'Q4'",
        "aggregation": "yoy_pct_change",
        "value": -0.12
      },
      "chart_refs": ["chart-3"],
      "narrative_role": "headline"
    },
    ...
  ]
}
```

**Review output** — structured, not prose-only:

```json
{
  "reviewer": "<agent_id>",
  "subject":  "<report_id>",
  "overall_score": 78.4,
  "dimensions": {
    "faithfulness":      { "score": 82, "notes": "claim i4 not supported by data" },
    "coverage":          { "score": 70, "notes": "missed regional breakdown" },
    "narrative_flow":    { "score": 85, "notes": "..." },
    "interactivity_use": { "score": 74, "notes": "..." }
  },
  "region_notes": [
    { "region": "chart-3", "severity": "medium", "text": "truncated y-axis misleads" }
  ]
}
```

The dimension set is **not prescribed**. Each agent picks its own — that's the whole point of rubric pluralism. The only shared contract is `overall_score` (for leaderboard aggregation) and `region_notes` (for region-tied critique, which humans can later validate).

## 9. Evaluation — the N×N cross-review aggregation

Per tick, for N agents, we run N generations + N² reviews. Aggregation:

- **Consensus ranking** — column means weighted by reviewer reliability (Dawid-Skene-style latent aggregation; Auto-Arena's committee aggregation is a good reference).
- **Per-agent quality** — leaderboard based on consensus position averaged across ticks.
- **Per-reviewer reliability** — how well each reviewer predicts the final consensus; low-reliability reviewers get downweighted.
- **Self-preference index** — diagonal lift vs. peer mean.
- **Rubric clusters** — hierarchical clustering of reviewers by score patterns; reveals whether the field splits into "narrative-first," "fidelity-first," "design-first," etc.
- **Disagreement hotspots** — reports with high column variance; surfaced in the Showcase Stage as scientifically interesting cases.

## 10. Dual-track leaderboard — borrowed from Prophet Arena

Mirror Prophet Arena's split to sidestep the "is this evaluating the model or the scaffolding?" question.

- **Constrained track** — every agent gets the same fixed harness: same generation template, same allowed libraries (e.g., Vega-Lite + pandas), same review prompt skeleton. Isolates the *model* contribution.
- **Open track** — agents bring their own pipelines, fine-tunes, retrieval, MCP tool sets, browser-use. Measures end-to-end capability.

Two leaderboards from one dataset, one submission cohort. Both publishable.

## 11. Ground truth — deferred, not absent

Prophet Arena has Kalshi. VIS Arena has no equivalent oracle. The design choice: **collect everything now, anchor later**.

- Every bundle is versioned and pinned. Nothing is overwritten.
- A lightweight pairwise-preference UI sits dormant until we want ground truth; we then solicit a sample of human judgments on archived pairs.
- Ground truth arrives as a *correlation signal*, not a per-report score: "which reviewer agents' rankings track human preference?" That correlation is the external anchor. It doesn't need to be dense — a small, carefully chosen sample is enough.

This is actually a strength relative to most benchmarks, which can't be re-scored with new ground truth after publication. VIS Arena can.

## 12. Streaming datasets — a rotating pool

Candidates for the first-cohort pool (picked for narrative richness + open licensing):

- A weekly Kaggle tabular release (arbitrary domain; forces generalization).
- A slice of NYC Open Data (311 complaints, collisions, permits — narrative-ready).
- IMDb / MovieLens subsets (genre × decade × rating — rich narrative surface).
- World Bank economic indicators (macro-story scale).
- A partner-contributed private dataset (kept embargoed for contamination control).

Rotation matters for two reasons. First, it tests generalization. Second, it mitigates dataset memorization / contamination — if models have seen a dataset in training, they can "discover" memorized findings; fresh data resists this.

## 13. Relationship to Prophet Arena

| Dimension | Prophet Arena | VIS Arena |
|---|---|---|
| Task | Predict live prediction-market events | Generate interactive report on streaming dataset |
| Ground truth | Kalshi / Polymarket resolution | Archived artifacts + deferred human preference |
| Metrics | Brier score · Average Return | Faithfulness · reader-utility · consensus · self-preference |
| Leaderboards | Model (fixed context) · Agent (open tools) | Constrained harness · Open harness |
| Submission | Agent via `ai-prophet` CLI | Agent via `vis-arena` CLI with `generate` + `review` |
| Novel move | Live uncontaminatable eval | **Unified generate + review submission** |

Same design pattern — pipeline-stage decomposition, live streaming tasks, dual leaderboards, CLI-based submission. Differences are shaped by the absence of a natural oracle: we lean harder on pluralist cross-review and deferred human preference.

## 14. Open design questions

Things worth pushing through before the first cohort runs.

- **What exactly does `/review-viz` see?** Just the bundle, or bundle + original dataset + task? The former mirrors a human reader's access; the latter enables rigorous fact-checking. My gut: give reviewers access to both, let their pipeline decide how much to use.
- **How are dimension names reconciled across reviewers?** Each agent names its own dimensions. For meta-analysis (rubric clustering) we need to map e.g. "narrative_flow" to "story_coherence". Options: free-form + post-hoc embedding-based clustering; or require tagging against a shared taxonomy that can be extended.
- **How do we prevent review-prompt theft?** An agent's review prompts leak its aesthetic. If bundles are public, so are reviews — does that close an arbitrage gap? Probably yes, and probably fine; academic peer review has the same property at conferences.
- **Latency/cost tolerance per tick.** N² reviews at cohort size 20 = 400 review runs per tick. At ~$0.50/run that's $200/tick — sustainable. At cohort size 100 = 10,000 runs = $5,000/tick. Need sponsor compute credits before we scale past ~30 agents.
- **Interactive report evaluation via computer-use.** Reviewers who navigate the live report (via Browserbase / Chrome DevTools MCP) get a richer signal than PDF-only reviewers. Do we differentiate their scores? Likely yes — flag the reader modality in the review output.
- **Rubric gaming window.** Once an agent knows the rubric-clustering story will be told, there's an incentive to submit a deliberately "median" rubric to win consensus. Mitigation: publish rubric clusters post-hoc, not as a scoring signal.
- **Name collision.** "Agent Arena" already exists (Berkeley Gorilla project, LLM-agent comparison). "VIS Arena" is different enough, but worth knowing. Auto-Arena is the closer mechanism prior art.

## 15. Pitch narrative — the 6-beat arc

For collaborator pitches; expanded version lives in the deck (`vis-arena-deck.html`).

1. **Last year's setup** — participants submitted an agent; agent produced PDF/HTML; humans manually reviewed. Right starting point.
2. **Three pain points** — manual eval doesn't scale, datasets stay narrow, rubrics drift.
3. **What changed this year** — SKILL.md, MCP, Claude Agent SDK, tool routing, browser-use. The agentic surface opened up.
4. **Proposal** — VIS Arena. One submission = one agent with two commands. Generation and review share code and philosophy.
5. **Map the fixes** — agent review scales; streaming datasets broaden scope; rubric pluralism surfaces disagreement instead of hiding it.
6. **The ask** — co-design the submission schema; contribute a baseline agent; run a first cohort before the next VIS × GenAI workshop.

**One-sentence version:** *Manual review doesn't scale, rubrics don't agree, datasets stay narrow — and the agent tooling we have now lets us fix all three at once.*

## 16. The ask — concrete

- **Co-design.** Lock the submission schema, insights manifest, and I/O contracts.
- **Baselines.** Each collaborator ships one baseline agent. Candidates: a Calliope-style narrative generator; a fidelity-first reviewer; a browser-use reviewer that navigates interactive reports; an interactive-visualization-first generator built on Vega-Lite + Observable.
- **First cohort.** Run 4–6 agents on one streaming dataset before VIS × GenAI. Arrive with real data, not a slide deck.

## 17. Related work

Grouped by how it informs a specific design choice.

**Live-arena / peer-battle prior art** (mechanism, aggregation math):
- [Auto-Arena](https://auto-arena.github.io/blog/) — peer battles + committee aggregation. Closest mechanism precedent.
- [Agent Arena — Berkeley Gorilla](https://gorilla.cs.berkeley.edu/blogs/14_agent_arena.html) — head-to-head LLM-agent comparison platform; name-collision reference.
- [Prophet Arena — Research / Welcome](https://www.prophetarena.co/research/welcome) — pipeline-stage decomposition, live streaming eval, dual leaderboards.
- [LLM-as-a-Prophet (arXiv 2510.17638)](https://arxiv.org/abs/2510.17638) — formal framing of live forecasting benchmarks.
- [Arena-Hard / BenchBuilder (arXiv 2406.11939)](https://arxiv.org/html/2406.11939v1) — crowdsourced benchmark construction.

**LLM-as-judge, self-bias, calibration** (review pipeline design):
- [Self-Preference Bias in LLM-as-a-Judge (arXiv 2410.21819)](https://arxiv.org/html/2410.21819v2) — the direct paper for the self-preference index metric.
- [Justice or Prejudice? Quantifying Biases in LLM-as-a-Judge](https://openreview.net/forum?id=3GTtZFiajM) — systematic bias catalog.
- [Agent-as-a-Judge (arXiv 2508.02994)](https://arxiv.org/html/2508.02994v1) — survey of the agent-judge paradigm; relevant because `review` is an agent inspecting a report, not an LLM scoring a string.
- [LLMs-as-Judges: A Comprehensive Survey (arXiv 2412.05579)](https://arxiv.org/html/2412.05579v2) — one-stop reference.

**Visualization evaluation & storytelling** (seed rubric, baselines):
- [Lexara: User-Centered Toolkit for Evaluating LLMs for Conversational Visual Analytics (arXiv 2603.05832)](https://arxiv.org/abs/2603.05832) — seed rubric to ship with the SDK.
- [Calliope: Automatic Visual Data Story Generation (arXiv 2010.09975)](https://arxiv.org/abs/2010.09975) — canonical baseline generator.
- [DataShot](https://www.semanticscholar.org/) — the earlier fact-sheet automation reference Calliope compares against.
- [CHART-6: Human-Centered Evaluation of Data Viz Understanding in VLMs (arXiv 2505.17202)](https://arxiv.org/abs/2505.17202) — human-anchored eval; useful for the deferred-preference layer.
- [SciVisAgentBench (arXiv 2603.29139)](https://arxiv.org/abs/2603.29139) — sibling viz-agent benchmark.

**Agent tooling / scaffolding** (what enabled the 2026 redesign):
- [Anthropic docs — Model Context Protocol (MCP)](https://docs.anthropic.com/en/docs/build-with-claude/mcp)
- [Anthropic — Advanced tool use (Tool Search)](https://www.anthropic.com/engineering/advanced-tool-use)
- [Chrome DevTools MCP for your AI agent](https://developer.chrome.com/blog/chrome-devtools-mcp)
- [Browserbase MCP server](https://github.com/browserbase/mcp-server-browserbase)
- [Cloudflare — Browser Run for AI agents](https://blog.cloudflare.com/browser-run-for-ai-agents/)
- [WebMCP explained](https://www.scalekit.com/blog/webmcp-the-missing-bridge-between-ai-agents-and-the-web)
- [Composio — AI agent integration platforms (2026)](https://composio.dev/content/ai-agent-integration-platforms)
- [Tessl — Agent Enablement Platform](https://tessl.io/)

**Venue & format**:
- [VIS × GenAI Workshop](https://visxgenai.github.io/)

## Companion artifacts in this folder

- `vis-arena-deck.html` — pitch deck, interactive cross-review heatmap on slide 6.
- `prophet-arena-notes.md` — precursor notes on Prophet Arena + the broader "for humans → for agents" shift that motivates this work.
