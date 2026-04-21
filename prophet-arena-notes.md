# Prophet Arena — Research Notes

_Compiled: 2026-04-16_

Working notes on **Prophet Arena**, its **Agent Leaderboard**, and associated resources (ProphetHacks hackathon, `ai-prophet` SDK). Drawn from the public ai-prophet GitHub repo, web coverage of the platform, and an arXiv paper from the team ("LLM-as-a-Prophet", 2510.17638). Direct fetches of `prophetarena.co` and `prophethacks.com` were blocked by this environment's egress proxy, so those sections rely on search-surfaced summaries and cached descriptions — treat specifics there as second-hand until verified on the live sites.

---

## 1. What is Prophet Arena?

- A **live benchmark for predictive intelligence** of LLMs. Instead of static Q&A over known answers, models forecast *future, unresolved events* drawn from real prediction markets.
- Built by the **SIGMA Lab at the University of Chicago**, powered by the regulated prediction market **Kalshi** (and also pulling markets from **Polymarket**).
- Authors include Qingchuan Yang, Jibang Wu, and Haifeng Xu (UChicago). Contact is `contact@prophetarena.co`.
- Positioning: sibling / competitor framing to "Prediction Arena" (arXiv 2604.07355) which runs longitudinal live-trading of frontier models with real capital.

### Why it's interesting
- Benchmarks that rely on historical ground truth are contaminable — models may have memorized outcomes. Live, future-resolving events are **uncontaminatable** by design.
- Forces models to express **calibrated probability distributions**, not yes/no labels → unlocks proper scoring rules.
- Ties academic evaluation to a **market-derived utility metric** (Average Return from simulated betting), not just accuracy.

---

## 2. The Two Leaderboards

Prophet Arena runs **two parallel leaderboards** with different affordances. This split is the key design idea to remember.

### Model Leaderboard (`/leaderboard`)
- Evaluates **raw model inference**.
- Fixed, centrally curated prediction context — every model gets the **same retrieved docs + same market snapshot** (latest Yes/No contract prices and trading volumes).
- Models **cannot** perform independent web search or tool use.
- Purpose: isolates reasoning / priors from retrieval skill.

### Agent Leaderboard (`/agent-leaderboard`, rules at `/research/agent-leaderboard-rules`)
- Evaluates **end-to-end agent capability**.
- Agents may freely use web search, external data sources, APIs, other compute tools.
- A submission is a **self-contained prediction agent** that produces probabilistic forecasts over a standardized event slate.
- Input to an agent is a structured **JSON payload** containing:
  - `event_id`
  - `title` (e.g., _"Will country X hold an election by March 2026?"_)
  - `markets` (e.g., `Yes` / `No`)
  - `rules` for resolution
- Once an agent's performance crosses a threshold it is **auto-promoted** to the public leaderboard. Pre-promotion status visible via an "onboarding models" page.
- Prophet Arena reserves the right to flag / remove agents that violate **reproducibility, fairness, or submission integrity**.

### Scoring metrics (both boards)
- **Brier score** — proper scoring rule for calibrated probabilistic forecasts. Early reporting has **GPT-5 at ~82.21% Brier** (leading).
- **Average Return** — profitability of trading the market using the model's forecasts. Reported that **OpenAI's o3-mini leads this metric** — i.e., accuracy and profitability diverge, which is itself a finding.
- Dashboard aggregates across time, domain, and scoring metric.

---

## 3. Benchmark Design (from the "LLM-as-a-Prophet" paper)

Each event is decomposed into a **pipeline of stages** so experiments can be controlled stage-by-stage:

1. **Context construction** — an LLM-based **search agent** retrieves information for the event.
2. **Market snapshot** — latest Yes/No contract prices + volumes are attached.
3. **Forecasting** — models emit a probability distribution, not a label.
4. **Scoring** — Brier score on resolution + Average Return on simulated bets.

Decomposing the pipeline this way lets researchers swap in / out components (e.g., same retrieval, different forecasting LLM) to attribute performance.

---

## 4. ai-prophet SDK & CLI

Repo: **https://github.com/ai-prophet/ai-prophet**
License: **MIT**. Language: **Python 3.11+**. Currently small (~42 commits, 6 forks at time of review), with active PRs.

### Monorepo layout
```
ai-prophet/
├── packages/
│   ├── core/          # ai-prophet-core — typed SDK (API models + client)
│   └── cli/           # ai-prophet — benchmark runner, `prophet` CLI
├── .github/workflows/ # CI
├── .pre-commit-config.yaml
├── CONTRIBUTING.md
├── LICENSE (MIT)
└── README.md
```

Both packages are on PyPI as `ai-prophet-core` (SDK) and `ai-prophet` (CLI).

### Install (dev)
```bash
python -m pip install -e packages/core
python -m pip install -e "packages/cli[dev]"
pre-commit install
```

### Lint / tests
```bash
ruff check --config packages/cli/pyproject.toml packages/core packages/cli
pytest packages/core/tests
pytest packages/cli/tests
```

### Required env vars
| Var | Use |
|---|---|
| `PA_SERVER_API_KEY` | Prophet Arena API auth |
| `ANTHROPIC_API_KEY` | Claude models |
| `OPENAI_API_KEY` | GPT models |
| `GEMINI_API_KEY` | Google models |
| `XAI_API_KEY` | xAI models |
| `KALSHI_API_KEY_ID`, `KALSHI_PRIVATE_KEY_B64` | Live (non-paper) Kalshi trading |
| `PA_SERVER_URL`, `KALSHI_BASE_URL`, `LIVE_BETTING_ENABLED` | Optional overrides |
| (Brave API key) | For the SEARCH stage of the pipeline |

### CLI surface (`prophet …`)
Forecast workflow:
- `prophet forecast events` — pull the server's current event slate
- `prophet forecast predict` — run the local pipeline to generate predictions
- `prophet forecast submit` — upload predictions
- `prophet forecast leaderboard` — view rankings

Trading benchmark:
```
prophet trade eval run -m <provider:model> \
  --replicates <n> --slug <name> --max-ticks <n>
```
Options: `--starting-cash`, betting-strategy selection, trace directory.

Configuration: drops a **`config.local.yaml`** in cwd to customize the pipeline, search behavior, and LLM behavior.

### Per-tick, per-participant pipeline (4 stages)
1. **REVIEW** — pick which markets to act on
2. **SEARCH** — gather web evidence (Brave Search)
3. **FORECAST** — emit calibrated probabilities
4. **ACTION** — place trade intents

**Privacy / locality note**: All LLM calls happen **locally** on the participant's infra — only final **trade intents** are transmitted to the Prophet Arena server. This is important for submissions where teams don't want prompts / chains exposed.

### Core SDK (`ai-prophet-core`)
Main classes:
- `ServerAPIClient(base_url=..., api_key="prophet_...")` — authenticated API client, context-manager friendly (`with ServerAPIClient(...) as c:`).
- `BenchmarkSession` — tick-based experiment driver. Methods: `create_experiment()`, `claim_tick()`, `load_candidates()`, `submit_intents()`, `finalize()`, `complete_tick()`.
- `BettingEngine(paper=True|False)` — simulated-or-live trade execution. Methods: `make_trade()`, `trade_from_forecast()`.

Data models:
- `Market` — `market_id`, `question`, `quote.best_bid`, `quote.best_ask`
- `MarketSnapshot.markets: list[Market]`
- `Forecast` — `market_ticker`, `p_yes`, `rationale`
- `Lease` — `available`, `reason`, `retry_after_sec` (tick-claim response; back-pressures concurrent participants)

Top-level API endpoints:
- `get_market_snapshot()` — curated liquid markets
- `submit_forecast()`
- Experiment endpoints via `BenchmarkSession`
- Trade endpoints via `BettingEngine`

Provider coverage: Anthropic, OpenAI, Google Gemini, xAI, plus **any OpenAI-compatible endpoint** (Together, local vLLM, etc.).

---

## 5. ProphetHacks — hackathon

Site: https://www.prophethacks.com/ (not directly reachable from this environment; no cached details surfaced in search).

What can reasonably be inferred given the naming and ecosystem:
- Almost certainly a **hackathon organized by / alongside the Prophet Arena team** — the `ProphetHacks` branding mirrors `PromptHacks` / `PropTechHacks` naming conventions on Devpost.
- Likely **tracks built around the Agent Leaderboard**: build a forecasting agent, submit via the `ai-prophet` CLI, compete on Brier score and/or Average Return.
- Prize structure, dates, and sponsors were not retrievable. **Action item**: open the URL directly in a browser to capture:
  - Event dates + location (virtual vs in-person, e.g., UChicago)
  - Sponsors (Kalshi? Polymarket? model-provider credits?)
  - Tracks (agent track vs tool/dataset track)
  - Prize pool and judging rubric
  - Registration / Devpost link

## 6. Open questions / things to verify on the live sites

- **Market categories** on `/markets`: elections, sports, macroeconomic indicators, tech/AI timelines, etc. — confirm breakdown and volume.
- **Current leaderboard ordering** beyond GPT-5 and o3-mini — Claude, Gemini, Grok, open-weights?
- **Agent leaderboard participants** — are third-party agents (non-model-lab) actually listed?
- **Rate limits / quotas** on `PA_SERVER_API_KEY` for agent submissions.
- **Resolution latency** — how long after Kalshi resolves does a forecast score lock in?
- **Data licensing** — can researchers redistribute the event slate + outcomes as a static benchmark?
- **ProphetHacks details** — dates, prizes, tracks (all currently unknown).

---

## 7. Related ideas & adjacent work

- **Prediction Arena** (arXiv 2604.07355) — different group; ran a 57-day longitudinal eval (Jan 12 – Mar 9, 2026) of 6 frontier models in live trading + 4 in paper trading. First live-capital autonomous-agent eval on prediction markets. Good comparison point for Prophet Arena's agent leaderboard.
- **Manifold market** "LLM reaches >90% Brier on Prophet Arena by 2026" — useful signal for community expectations about saturation.
- **`awesome-foundation-model-leaderboards`** (SAILResearch) — Prophet Arena is listed; worth skimming for sibling benchmarks (Arena-Hard, LMSYS Chatbot Arena, AgentBench, etc.) to position a research narrative.
- **Awesome-Prediction-Market-Tools** (aarora4) — curated list of agent/analytics tooling across Kalshi/Polymarket; useful for finding data sources / reference implementations for a hackathon submission.

## 8. Ideas this unlocks for an agent submission

Rough brainstorm, not vetted:

1. **Retrieval-ablation agent** — dominate the Agent board specifically by out-retrieving the Model board's curated context (e.g., primary-source scraping, structured source ranking).
2. **Calibration layer** — wrap any frontier model with an explicit Platt / isotonic calibrator fit on resolved Prophet Arena events → likely improves Brier without touching the base model.
3. **Market-aware forecasting** — incorporate market price as a prior, then diverge from it only when the evidence is strong → should boost Average Return even if Brier moves little.
4. **Multi-model ensemble with disagreement as signal** — Prophet Arena's own finding that models have "personalities" suggests ensembles over diverse personas may be undervalued.
5. **Event decomposition** — use an LLM to break "Will X happen by date D?" into sub-questions, forecast each, combine probabilistically (noisy-OR / Bayesian network). Maps cleanly onto the pipeline-stage design of the benchmark.
6. **Temporal self-consistency** — have the agent forecast the same event at multiple horizons and enforce martingale-like consistency; use violations as confidence signal.

---

## 9. The "For Humans → For Agents" Shift — Products, Tools, Ideas

Prophet Arena's agent leaderboard is one instance of a much broader reconfiguration: products, interfaces, and protocols that used to target human users are being re-cut to target **AI agents as the primary consumer**. Below is a working catalog of that shift — useful as context, and as a jumping-off point for "what would an agent-first version of X look like?" ideation.

### 9.1 The core reframing

The same product can be evaluated along two axes now:
- **Human-optimized**: visual hierarchy, copy that persuades, SEO for search engines, checkout UX designed to reduce friction for a pair of eyeballs.
- **Agent-optimized**: machine-readable capability declarations, deterministic APIs, proper scoring rules / structured outputs, auth flows that don't assume a browser, pay-per-call pricing, reproducibility guarantees.

The interesting question for any product category is: *which parts need to be duplicated for agents vs. which parts should be retired entirely for humans too?*

### 9.2 Category map — human product → agent-first analog

| Human category | Agent-first analog / shift | Example products |
|---|---|---|
| SEO (Search Engine Optimization) | **AEO / GEO** (Agent / Generative Engine Optimization) — structured, crawler-friendly content that LLMs can cite | Webflow AEO, Frase, Agent Engine Optimization playbooks |
| `robots.txt` / `sitemap.xml` | **`llms.txt`** — markdown at site root declaring canonical content and how LLMs should use it | Emerging standard; adopted by Anthropic, Stripe docs, others |
| GUI / web apps for humans | **MCP servers**, **WebMCP**, typed tool schemas | Model Context Protocol, WebMCP proposal, Composio, Arcade, Nango, Tessl |
| Chrome / browser UX | **Agent-driven browsers** | Chrome DevTools MCP, Browser MCP, Browserbase + Stagehand, Cloudflare Browser Run, AgentDesk BrowserTools |
| Credit card checkout | **Agent payment rails** | Stripe + OpenAI **Agentic Commerce Protocol (ACP)**, Google **Agent Payments Protocol (AP2)**, Google+Shopify **Universal Commerce Protocol (UCP)**, **x402** (pay-per-API-call in USDC on Base), Visa **Agentic Ready**, Nevermined, Sapiom |
| SaaS seat-based pricing | **Usage-based / per-call / per-outcome** pricing for agent buyers | Stripe x402, Nevermined, "MAIO" (Machine Agent Interaction Optimization) |
| Customer support CX | **Agent-to-agent negotiation & action** | Google **A2A (Agent2Agent)** protocol; Entra Agent ID-gated transactions |
| Evaluations (MMLU, etc., static) | **Live, uncontaminatable arenas** | **Prophet Arena**, Prediction Arena, LMSYS Chatbot Arena, PR Arena, Agent Arena |
| Static benchmark datasets | **Live benchmarks with streaming ground truth** | Prophet Arena (Kalshi/Polymarket resolutions), Prediction Arena |
| Dashboards / BI for analysts | **Data agents** with APIs | Google Gemini Data Agents, Vertex AI Agent Builder, MCP Toolbox for Databases |
| Docs sites for developers | **Machine-readable API descriptions + tool skills** | Tessl (turns APIs/libraries/conventions into agent-usable skills), Composio Tool Router |
| Identity (SSO for humans) | **Non-human identity / agent IDs** | Microsoft Entra Agent ID, Sapiom agent wallets, delegated-auth schemes |
| Operating system UX | **Agent-native OS** concepts | Emerging; "who's building it" discourse on Medium, agent-browser-as-OS framings |

### 9.3 Infrastructure layer — protocols to watch

- **MCP (Model Context Protocol)** — Anthropic-originated, now de facto standard for wiring agents into tools/data. Production-ready ecosystem; most of the items above either ship MCP servers or consume them.
- **A2A (Agent2Agent)** — Google's framework-agnostic protocol for agents in different ecosystems to talk to each other.
- **ACP (Agentic Commerce Protocol)** — Stripe + OpenAI; merchants can turn on with ~1 line of code if already on Stripe. Powers ChatGPT Instant Checkout with Etsy + Shopify merchants.
- **AP2 (Agent Payments Protocol)** — Google-led, 60+ partners (Adyen, AmEx, Ant, Coinbase, Mastercard, PayPal, Salesforce, Worldpay). Supports cards and crypto (x402 extension).
- **UCP (Universal Commerce Protocol)** — Google + Shopify, Jan 2026.
- **x402** — HTTP-native pay-per-request in stablecoin; "vending machine for autonomous software."
- **WebMCP** — proposal for web apps to declare their capabilities directly, so agents skip DOM scraping.
- **llms.txt** — content-side manifest.

### 9.4 Interface ideas — building GUIs *for agents*

This is a genuinely open design space. A few threads:

1. **Agent-Computer Interface (ACI) research** — SWE-agent, Agent S, OSWorld etc. Humans score 70–75% on standard desktop tasks; agents are 15–40%. The gap implies room for redesigning the *interface*, not just the model. Dual-input (screenshot + accessibility tree), constrained action spaces, immediate structured feedback.
2. **Terminal-as-universal-agent-UI** — arXiv 2603.10664 "Terminal Is All You Need" — argues stdout/stderr over rich GUIs for human-AI collaboration.
3. **Tool Search** (Anthropic) — dynamic tool discovery instead of dumping all schemas in context; shifts API design toward self-describing, searchable surface area.
4. **Capability declaration over UI interpretation** — WebMCP, OpenAPI++, tool manifests that say *what the app can do* in agent-digestible form.

### 9.5 Emerging "built for agents" product patterns

Patterns to note when scouting or building:

- **Sells to agents directly** — headless browsers (Browserbase), search-for-agents (Exa, Tavily, Brave API), sandboxed code-exec (E2B, Modal), vector DBs pivoting from human RAG to agent memory (Mem0, Letta).
- **Exposes only a programmatic surface** — no dashboard-first metaphor; MCP server + CLI (the `ai-prophet` repo itself is a good example: CLI-first, agent-first).
- **Pay-per-call micro-billing** — x402, Nevermined; assumes the "user" is a short-lived process, not a monthly-seat human.
- **Agent-verified / agent-signed interactions** — agent IDs + wallets + audit trails; regulation follows.
- **Live / uncontaminatable evaluation** — Prophet Arena's core move; applies anywhere you can stream ground truth.

### 9.6 Ideas to chase (agent-first versions of human things)

Concrete "what if X were rebuilt for agents?" seeds:

1. **Agent-first research repository** — papers published as `.mcp.json` capability manifests describing what the code does, benchmarks it beats, datasets it needs, sibling to arXiv.
2. **Market research / CRM for agents** — instead of humans reading 10-Ks, agents call structured endpoints with pay-per-query pricing (x402). Basically "Bloomberg for agents."
3. **Agent-first design systems** — Figma plugins that emit both human comps and `ui.mcp.json` so an agent can reconstruct the UI behaviorally.
4. **Prediction-market-as-oracle** — Prophet Arena style, but the *product* is a calibrated probability endpoint for any agent's downstream planner. Sell forecasts as a service.
5. **Agent provenance / attribution layer** — when multiple agents collaborate on an outcome, who gets paid? A2A + AP2 + a Shapley-value backend.
6. **Agent-first wiki** — MCP endpoints over curated domain knowledge, with Brier-scored corrections and bounties (marries Prophet-Arena-style live scoring with Wikipedia).
7. **"Chrome DevTools for agents"** — a full-trace debugger that replays an agent run with tool-call timeline, counterfactual re-runs, cost attribution.
8. **Benchmark-as-a-service** — the Prophet Arena model generalized: any domain (medical coding, legal review) gets a live, streaming ground-truth eval that's uncontaminatable.

### 9.7 Why this matters for Prophet Arena specifically

- Prophet Arena's **Agent Leaderboard** is literally the "built for agents" version of a benchmark — submission is a program, not a prose answer. Every design decision (JSON payloads, locally-run LLMs, only trade intents on the wire, pay-per-tick implicit via API keys) is recognizably agent-first.
- A compelling research narrative would tie Prophet Arena to the **broader agent-economy protocols**: a forecaster agent using AP2 to monetize its calibrated-probability endpoint, purchased by a downstream trading agent, resolved against Kalshi, scored via Prophet Arena. That's a full loop from A2A → MCP → AP2 → Prophet Arena → Kalshi.
- It also suggests where a hackathon like **ProphetHacks** could focus: not just "best Brier score" but "best agent-to-agent forecasting service that other agents actually buy."

### Extra sources for section 9

- [Webflow — Introducing Webflow AEO](https://webflow.com/blog/introducing-webflow-aeo)
- [Search Engine Land — Agentic engine optimization playbook](https://searchengineland.com/agentic-engine-optimization-google-ai-director-474358)
- [Hyperleap — llms.txt Explained: Should Your SaaS Site Have One in 2026?](https://hyperleap.ai/blog/llms-txt-explained-saas-generator)
- [Stripe — Introducing our agentic commerce solutions](https://stripe.com/blog/introducing-our-agentic-commerce-solutions)
- [Stripe newsroom — Instant Checkout in ChatGPT + Agentic Commerce Protocol](https://stripe.com/newsroom/news/stripe-openai-instant-checkout)
- [OpenAI — Buy it in ChatGPT: Instant Checkout and the Agentic Commerce Protocol](https://openai.com/index/buy-it-in-chatgpt/)
- [Google Cloud — Announcing Agent Payments Protocol (AP2)](https://cloud.google.com/blog/products/ai-machine-learning/announcing-agents-to-payments-ap2-protocol)
- [Google Developers — Under the Hood: Universal Commerce Protocol (UCP)](https://developers.googleblog.com/under-the-hood-universal-commerce-protocol-ucp/)
- [Crossmint — Agentic payments protocols compared (MPP, ACP, AP2, x402)](https://www.crossmint.com/learn/agentic-payments-protocols-compared)
- [BlockEden — Sapiom's $15.75M Bet: Why AI Agents Need Their Own Wallets](https://blockeden.xyz/blog/2026/03/13/sapiom-ai-agent-wallets-payment-rails-autonomous-commerce/)
- [Chrome for Developers — Chrome DevTools MCP for your AI agent](https://developer.chrome.com/blog/chrome-devtools-mcp)
- [Browser MCP](https://browsermcp.io/)
- [Cloudflare — Browser Run for AI agents](https://blog.cloudflare.com/browser-run-for-ai-agents/)
- [Scalekit — WebMCP: how browser agents can call web tools without scraping the DOM](https://www.scalekit.com/blog/webmcp-the-missing-bridge-between-ai-agents-and-the-web)
- [Anthropic — Advanced tool use (Tool Search)](https://www.anthropic.com/engineering/advanced-tool-use)
- [Anthropic docs — What is the Model Context Protocol (MCP)?](https://docs.anthropic.com/en/docs/build-with-claude/mcp)
- [Composio — AI agent integration platforms comparison (2026)](https://composio.dev/content/ai-agent-integration-platforms)
- [Nango — Best AI agent integration platforms 2026](https://nango.dev/blog/best-ai-agent-integration-platforms/)
- [Tessl — Agent Enablement Platform](https://tessl.io/)
- [Agent Native — Agentic SaaS Playbook in 2026](https://www.agentnative.dev/agentic-saas)
- [Deloitte — SaaS meets AI agents](https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/saas-ai-agents.html)
- [Medium — Who Is Building the Agent-Native Operating System?](https://medium.com/@marc.bara.iniesta/who-is-building-the-agent-native-operating-system-c6bae5a5a3f5)
- [Cobus Greyling — The Advent of Open Agentic Frameworks & ACI](https://cobusgreyling.medium.com/the-advent-of-open-agentic-frameworks-agent-computer-interfaces-aci-30414c3e8e16)
- [Agent S: An Open Agentic Framework that Uses Computers Like a Human (arXiv 2410.08164)](https://arxiv.org/html/2410.08164v1)
- [SWE-agent: ACIs Enable Automated Software Engineering (arXiv 2405.15793)](https://arxiv.org/pdf/2405.15793)
- [Terminal Is All You Need (arXiv 2603.10664)](https://arxiv.org/html/2603.10664)
- [o-mega — 2025–2026 guide to AI computer-use benchmarks](https://o-mega.ai/articles/the-2025-2026-guide-to-ai-computer-use-benchmarks-and-top-ai-agents)

---

## Sources

- [ai-prophet GitHub repo](https://github.com/ai-prophet/ai-prophet) — primary source for SDK/CLI details
- [Prophet Arena — Agent Leaderboard Rules](https://www.prophetarena.co/research/agent-leaderboard-rules) (not directly fetched; search snippet)
- [Prophet Arena — Research / Welcome](https://www.prophetarena.co/research/welcome) (search snippet)
- [Prophet Arena — Leaderboard](https://www.prophetarena.co/leaderboard) (search snippet)
- [Prophet Arena — Markets](https://prophetarena.co/markets)
- [LLM-as-a-Prophet (arXiv 2510.17638)](https://arxiv.org/abs/2510.17638)
- [Prediction Arena: Benchmarking AI Models on Real-World Prediction Markets (arXiv 2604.07355)](https://arxiv.org/abs/2604.07355)
- [UChicago DSI — Breaking New Ground in ML/AI: Prophet Arena](https://datascience.uchicago.edu/insights/breaking-new-ground-in-machine-learning-and-ai-new-platform-prophet-arena-redefines-how-we-evaluate-ais-intelligence/)
- [Zoonop — Prophet Arena Launches, Powered by Kalshi](https://zoonop.com/articles/prophet-arena-launches-as-ai-benchmark-for-real-world-predictive-intelligence-powered-by-kalshi)
- [ReadWrite — Prediction Pulse: Kalshi, Polymarket, Prophet Arena](https://readwrite.com/prediction-markets-kalshi-polymarket-ai-prophet-arena-ncaa-powells-speech/)
- [Manifold — "LLM reaches >90% Brier score on Prophet Arena by 2026?"](https://manifold.markets/kiudee/llm-reaches-90-brier-score-on-proph)
- [ProphetHacks](https://www.prophethacks.com/) — site not directly reachable from this environment; details TBD
