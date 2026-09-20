# Autonomous Revenue Engine

**Nine production-grade systems that take a company from a dirty CSV row to a cited, budget-bounded sales conversation — and the AI infrastructure that keeps them honest.**

Most GTM tooling demos well and fails in production, because the hard parts are not the happy path. They are the fabricated claim, the stale signal, the prompt injection, the silent model downgrade, the context window that quietly truncates the one constraint that mattered. This suite is built around those failure modes.

Every repository below is independently installable, strictly typed, and covered by its own test suite. **1,583 tests across the nine.**

---

## The system

Five services form a revenue pipeline. Four more sit underneath as shared AI infrastructure — the pipeline calls into them rather than reimplementing inference, safety, evaluation and memory nine times over.

```
                            ══ GTM / REVENUE PIPELINE ══

   dirty CSV            inbound lead          company              outbound
      │                      │                   │                    │
      ▼                      ▼                   ▼                    ▼
┌───────────┐         ┌──────────────┐    ┌───────────┐       ┌─────────────┐
│rev-cleanse│────────▶│ speed-router │───▶│    lea    │──────▶│ hook-smith  │
│           │         │              │    │           │signals│             │
│ canonical │ clean   │ sub-ms SLA   │    │autonomous │       │ grounded    │
│ accounts  │ records │ routing +    │    │ research  │       │ outbound    │
│ DuckDB    │         │ Slack Block  │    │ + ICP fit │       │ copy        │
└───────────┘         └──────────────┘    └───────────┘       └─────────────┘
                                                 │                    │
                                                 │  enriched record   │
                                                 ▼                    ▼
                                          ┌────────────────────────────────┐
                                          │        deal-intel-rag          │
                                          │  cited battlecard / objection  │
                                          │  copilot, refuses when unsure  │
                                          └────────────────────────────────┘
                                                         │
  ─────────────────────────────────────────────────────────────────────────────
                            ══ AI INFRASTRUCTURE ══
  ─────────────────────────────────────────────────────────────────────────────
                                                         │
   ┌──────────────┬───────────────────┬──────────────────┴──┬──────────────────┐
   │              │                   │                     │                  │
   ▼              ▼                   ▼                     ▼                  │
┌───────────┐ ┌────────────┐   ┌─────────────┐      ┌─────────────┐            │
│cache-flow │ │ guard-rail │   │ eval-forge  │      │  mem-core   │            │
│           │ │            │   │             │      │             │            │
│ semantic  │ │ injection  │   │ debiased    │      │ 3-tier      │            │
│ cache +   │ │ firewall   │   │ LLM judge   │      │ memory +    │            │
│ cost      │ │ <0.15ms    │   │ + CI gate   │      │ budget      │            │
│ routing   │ │ fast path  │   │             │      │ compiler    │            │
└───────────┘ └────────────┘   └─────────────┘      └─────────────┘            │
   cheaper       safer            measurable           bounded                 │
      └──────────────┴───────────────┴────────────────────┴───────────────────┘
                     every LLM call in the pipeline flows through these
```

**Read it as:** `rev-cleanse` decides what an account *is*. `speed-router` decides who gets it and how fast. `lea` decides whether it is worth pursuing. `hook-smith` decides what to say. `deal-intel-rag` answers what comes back. The infrastructure layer decides what it costs, whether it is safe, whether it actually works, and what it remembers.

---

## The nine

### Part 1 — GTM & Revenue Operations

| Repo | Does | Production rigor | Stack | Tests |
|---|---|---|---|---|
| **[lea](https://github.com/yashsabbavarapu/lead-enrichment-agent)** | Autonomous company research → deterministic ICP score | **Non-raising tool contracts** — a failing tool degrades the report, never the run. Gap-driven stopping conditions instead of fixed loops; every claim carries provenance back to a fetched URL | pydantic, httpx, typer, selectolax, anthropic | 884 |
| **[deal-intel-rag](https://github.com/yashsabbavarapu/deal-intel-rag)** | RAG over battlecards, security docs, pricing | **Empirical calibration harness** that measures the actual overlap between relevant and irrelevant vector distances, rather than assuming a threshold. Strict citation binding and an explicit refusal mode | chromadb, pydantic, typer | 23 |
| **[hook-smith](https://github.com/yashsabbavarapu/hook-smith)** | Signals → targeted outbound copy | **Claim-to-signal binding (`SignalScope`)** — every generated claim must bind to a specific ingested signal. Redlines unverified claims and catches cross-signal misattribution, the failure that makes AI outbound embarrassing | pydantic (zero-dep core) | 72 |
| **[speed-router](https://github.com/yashsabbavarapu/speed-router)** | Inbound webhook → qualified, routed, Slack-dispatched | **Sub-millisecond routing** with deterministic qualification tiers, weighted-capacity rep assignment, and single-process safety guards against double-dispatch | fastapi, pydantic, uvicorn | 57 |
| **[rev-cleanse](https://github.com/yashsabbavarapu/rev-cleanse)** | Dirty CRM CSV → canonical accounts | **Strict idempotency with a full merge audit log** — re-running never double-merges, and every field override is explained (`500 -> 850, newer timestamp wins`). Fuzzy resolution over canonicalized domains and stripped legal suffixes | duckdb, rapidfuzz, pydantic | 105 |

### Part 2 — Core AI Engineering

| Repo | Does | Production rigor | Stack | Tests |
|---|---|---|---|---|
| **[cache-flow](https://github.com/yashsabbavarapu/cache-flow)** | Semantic cache + cost-aware model router | Cache hits at **cosine ≥ 0.94** return in sub-millisecond memoized time; misses route to cheap vs. heavy models on query-complexity heuristics, so spend tracks difficulty rather than volume | fastapi, numpy, pydantic | 76 |
| **[guard-rail](https://github.com/yashsabbavarapu/guard-rail)** | Prompt-injection & adversarial firewall | **Dual-path**: a `<0.15ms` deterministic gate (entropy, delimiter escape, canary exfiltration, homoglyph evasion) backed by an adversarial vector gate. 50-prompt red-team fuzzer, plus explicit guards against *silent backend degradation* | numpy, pydantic | 66 |
| **[eval-forge](https://github.com/yashsabbavarapu/eval-forge)** | Synthetic benchmarks + debiased LLM judge | **Positional-swap debiasing** with Cohen's Kappa and bootstrap *p*-values, and a CI exit gate that emits `JUDGE UNRELIABLE` rather than reporting a confident number from an unreliable judge | pydantic, typer, rich | 126 |
| **[mem-core](https://github.com/yashsabbavarapu/mem-core)** | 3-tier agent memory + context budget compiler | **Line-atomic extractive compression** — a truncated `budget = 5000` would read as `500`, so facts drop whole. Budget guarantee fuzzed over 300 randomized trials: zero breaches, zero partial facts | numpy, pydantic, SQLite | 174 |

---

## What "production rigor" means here

Four commitments hold across all nine repositories:

1. **Strict typing.** `mypy --strict` passes on every typed package — no `Any` escapes, no untyped defs.
2. **Determinism where it matters.** Scoring, extraction, routing and budget packing are deterministic and unit-tested. Model calls sit behind interfaces with offline stubs, so the suites run with no network and no keys.
3. **Explicit failure modes.** Each system states what it refuses to do — `deal-intel-rag` refuses to answer uncited, `eval-forge` refuses to report an unreliable verdict, `lea` refuses to score a company it learned nothing about, `mem-core` refuses to emit a partial fact.
4. **Stated limits.** Every README documents where the system stops being good. A memory engine that oversells its recall is worse than one that publishes its floor.

---

## Quickstart

Each repo installs and runs standalone:

```bash
git clone https://github.com/yashsabbavarapu/mem-core && cd mem-core
python3.11 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest -v
```

End to end, the pipeline composes like this:

```bash
# 1. Clean the inbound list into canonical accounts
rev-cleanse ingest --input leads.csv --db crm.duckdb --audit merge_audit.json

# 2. Research a company and score it against the ICP
lea enrich "Vanta" --json > vanta.json

# 3. Turn verified signals into outbound copy (claims bound to signals)
hooksmith generate --input vanta.json --strict

# 4. Answer the objection that comes back, with citations
dealintel ask "how do you compare on SOC 2 automation?"
```

And in Python, the infrastructure layer composes independently of the pipeline:

```python
from guardrail import Gateway          # 1. is the input safe?
from cacheflow import SemanticCache    # 2. have we answered this already?
from memcore import ContextCompiler, ContextBudget   # 3. what context fits?

verdict = Gateway().inspect(user_input)
if verdict.blocked:
    raise ValueError(verdict.reason)

hit = SemanticCache().get(user_input)          # cosine >= 0.94 -> sub-ms
prompt = ContextCompiler(ContextBudget(total_token_budget=1500)).compile(
    user_input, working, episodic, semantic
)
assert prompt.total_tokens <= 1500              # guaranteed, not hoped for
```

---

## Engineering standards

| | |
|---|---|
| **Language** | Python 3.11+ (CI matrix: 3.11, 3.12, 3.13) |
| **Typing** | `mypy --strict` across all typed packages |
| **Lint** | `ruff` with an explicit rule set (`E, F, I, UP, B, SIM, RUF`) |
| **Tests** | 1,583 across nine repositories; offline by default |
| **CI** | GitHub Actions on every repo — lint, types, tests, on three Python versions |
| **License** | MIT throughout |

---

*Built by [Yash Sabbavarapu](https://github.com/yashsabbavarapu).*
