# Autonomous Revenue Engine
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) [![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

Nine systems that take a company from a messy CSV row to a cited, budget-bounded sales conversation, plus the AI infrastructure underneath that keeps them honest.

Most GTM tooling demos well and breaks in production, because the hard parts aren't on the happy path. The hard parts are the fabricated claim, the two-year-old signal presented as news, the prompt injection, the model that silently downgrades, and the context window that quietly drops the one constraint that mattered. Each repo here was built around one of those failures.

Every project installs on its own, passes `mypy --strict`, and carries its own suite. 1,583 tests across the nine.

---

## How it fits together

Five services form the revenue pipeline. Four more sit underneath as shared infrastructure, so inference, safety, evaluation and memory are solved once instead of nine times.

```
                            GTM / REVENUE PIPELINE

   messy CSV            inbound lead          company              outbound
      |                      |                   |                    |
      v                      v                   v                    v
+-----------+         +--------------+    +-----------+       +-------------+
|rev-cleanse|-------->| speed-router |--->|    lea    |------>| hook-smith  |
|           | clean   |              |    |           |signals|             |
| canonical | records | sub-ms SLA   |    |autonomous |       | grounded    |
| accounts  |         | routing +    |    | research  |       | outbound    |
| in DuckDB |         | Slack alert  |    | + ICP fit |       | copy        |
+-----------+         +--------------+    +-----------+       +-------------+
                                                 |                    |
                                                 |  enriched record   |
                                                 v                    v
                                          +--------------------------------+
                                          |        deal-intel-rag          |
                                          |  cited battlecard / objection  |
                                          |  copilot, refuses when unsure  |
                                          +--------------------------------+
                                                         |
  ---------------------------------------------------------------------------
                            AI INFRASTRUCTURE
  ---------------------------------------------------------------------------
                                                         |
   +--------------+---------------+---------------------+------------------+
   |              |               |                     |                  |
   v              v               v                     v                  |
+-----------+ +------------+ +-------------+     +-------------+           |
|cache-flow | | guard-rail | | eval-forge  |     |  mem-core   |           |
|           | |            | |             |     |             |           |
| semantic  | | injection  | | debiased    |     | three-tier  |           |
| cache and | | firewall,  | | LLM judge   |     | memory and  |           |
| cost-based| | 0.15ms     | | with a CI   |     | a budget    |           |
| routing   | | fast path  | | exit gate   |     | compiler    |           |
+-----------+ +------------+ +-------------+     +-------------+           |
   cheaper       safer          measurable          bounded                |
      +--------------+---------------+---------------------+---------------+
                  every model call in the pipeline flows through these
```

Read it as a chain of decisions. `rev-cleanse` decides what an account is. `speed-router` decides who owns it and how fast. `lea` decides whether it's worth pursuing. `hook-smith` decides what to say. `deal-intel-rag` handles what comes back. The infrastructure layer decides what all of that costs, whether it's safe, whether it actually works, and what it remembers.

---

## The nine projects

### GTM and revenue operations

| Repo | What it does | The part that took the work | Stack | Tests | CI |
|---|---|---|---|---|---|
| [lea](https://github.com/yashsabbavarapu/lead-enrichment-agent) | Researches a company, scores it against an ICP | Tool contracts never raise, so a dead endpoint degrades the report instead of killing the run. Research stops when the gaps close, not after a fixed number of loops, and every claim traces back to a URL that was actually fetched | pydantic, httpx, typer, selectolax | 884 | [![CI](https://github.com/yashsabbavarapu/lead-enrichment-agent/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/lead-enrichment-agent/actions/workflows/ci.yml?query=branch%3Amain) |
| [deal-intel-rag](https://github.com/yashsabbavarapu/deal-intel-rag) | Answers objections from battlecards, security docs and pricing | Ships a calibration harness that measures how far relevant and irrelevant vector distances actually overlap, instead of picking a threshold and hoping. Answers cite their sources or the system declines to answer | chromadb, pydantic, typer | 23 | [![CI](https://github.com/yashsabbavarapu/deal-intel-rag/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/deal-intel-rag/actions/workflows/ci.yml?query=branch%3Amain) |
| [hook-smith](https://github.com/yashsabbavarapu/hook-smith) | Turns verified signals into outbound copy | Every claim has to bind to a specific ingested signal through `SignalScope`. Unbound claims get redlined, which catches the cross-signal misattribution that makes AI outreach embarrassing | pydantic, no other runtime deps | 72 | [![CI](https://github.com/yashsabbavarapu/hook-smith/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/hook-smith/actions/workflows/ci.yml?query=branch%3Amain) |
| [speed-router](https://github.com/yashsabbavarapu/speed-router) | Routes inbound webhooks to reps and Slack | Qualification tiers are deterministic and rep assignment is weighted by live capacity. Guards against double-dispatch, because sending the same lead twice is worse than sending it slowly | fastapi, pydantic, uvicorn | 57 | [![CI](https://github.com/yashsabbavarapu/speed-router/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/speed-router/actions/workflows/ci.yml?query=branch%3Amain) |
| [rev-cleanse](https://github.com/yashsabbavarapu/rev-cleanse) | Resolves dirty CRM exports into canonical accounts | Re-running the pipeline never double-merges, and every field override explains itself in an audit log (`500 -> 850, newer timestamp wins`). Fuzzy matching runs over canonicalised domains with legal suffixes stripped | duckdb, rapidfuzz, pydantic | 105 | [![CI](https://github.com/yashsabbavarapu/rev-cleanse/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/rev-cleanse/actions/workflows/ci.yml?query=branch%3Amain) |

### Core AI engineering

| Repo | What it does | The part that took the work | Stack | Tests | CI |
|---|---|---|---|---|---|
| [cache-flow](https://github.com/yashsabbavarapu/cache-flow) | Semantic cache in front of a cost-aware model router | Hits above 0.94 cosine return from memoised lookup in under a millisecond. Misses route to a cheap or heavy model on query-complexity heuristics, so spend tracks difficulty instead of traffic | fastapi, numpy, pydantic | 76 | [![CI](https://github.com/yashsabbavarapu/cache-flow/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/cache-flow/actions/workflows/ci.yml?query=branch%3Amain) |
| [guard-rail](https://github.com/yashsabbavarapu/guard-rail) | Prompt injection and adversarial input firewall | A deterministic gate runs in under 0.15ms on entropy, delimiter escapes, canary exfiltration and homoglyph evasion, with a vector gate behind it. A 50-prompt red team runs in CI, and the system fails loudly rather than degrading to a weaker backend in silence | numpy, pydantic | 66 | [![CI](https://github.com/yashsabbavarapu/guard-rail/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/guard-rail/actions/workflows/ci.yml?query=branch%3Amain) |
| [eval-forge](https://github.com/yashsabbavarapu/eval-forge) | Generates benchmarks and runs a debiased LLM judge | Pairwise judging swaps positions to cancel order bias, then reports Cohen's Kappa and bootstrap p-values. When agreement is too low the CI gate prints `JUDGE UNRELIABLE` instead of a confident number nobody should trust | pydantic, typer, rich | 126 | [![CI](https://github.com/yashsabbavarapu/eval-forge/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/eval-forge/actions/workflows/ci.yml?query=branch%3Amain) |
| [mem-core](https://github.com/yashsabbavarapu/mem-core) | Three-tier agent memory behind a context budget compiler | Facts are atomic when the budget gets tight, because truncating `budget = 5000` produces `budget = 500`, and a fabricated constraint is worse than a missing one. The budget guarantee held across 300 randomised fuzz trials with zero breaches | numpy, pydantic, SQLite | 174 | [![CI](https://github.com/yashsabbavarapu/mem-core/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/yashsabbavarapu/mem-core/actions/workflows/ci.yml?query=branch%3Amain) |

---

## What holds across all nine

`mypy --strict` passes on every typed package, with no `Any` escape hatches.

Scoring, extraction, routing and budget packing are deterministic and unit tested. Model calls sit behind interfaces with offline stubs, so every suite runs without network access or API keys.

Each system states what it refuses to do. `deal-intel-rag` won't answer without a citation. `eval-forge` won't report a verdict from a judge it can't trust. `lea` won't score a company it learned nothing about. `mem-core` won't emit half a fact.

Each README also documents where the system stops being good. A memory engine that oversells its recall is worse than one that publishes its floor.

---

## Getting started

Any repo runs standalone:

```bash
git clone https://github.com/yashsabbavarapu/mem-core && cd mem-core
python3.11 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest -v
```

The pipeline composes from the command line:

```bash
# clean the inbound list into canonical accounts
rev-cleanse ingest --input leads.csv --db crm.duckdb --audit merge_audit.json

# research a company and score it against the ICP
lea enrich "Vanta" --json > vanta.json

# turn verified signals into outbound copy
hooksmith generate --input vanta.json --strict

# answer the objection that comes back, with citations
dealintel ask "how do you compare on SOC 2 automation?"
```

What the last mile looks like. A draft written by a human (or another tool) goes through the redliner before it can reach a prospect:

```
$ hooksmith validate --input fixtures/sample_lead.json \
    --draft fixtures/hallucinated_draft.txt --subject "Your Series C"

1/1 redlined.
Linear (linear.app) - validating draft

[Manual draft] FAIL
  Subject: [REDLINED: Unverified claim] Your Series C

  [REDLINED: Unverified claim] Congrats on closing the $50M Series C led by Andreessen Horowitz last quarter.
  ✗ [REDLINED: Unverified claim] funding round not present in lead record: 'Series C'
  ✗ [REDLINED: Unverified claim] dollar figure not present in lead record: '$50M'
  ✗ [REDLINED: Unverified claim] named entity not present in lead record: 'Andreessen Horowitz'
  ...
```

No verified signal, no send. That is the whole thesis in one screen.


The infrastructure layer composes in Python, independently of the pipeline:

```python
from guardrail import Gateway
from cacheflow import SemanticCache
from memcore import ContextBudget, ContextCompiler

verdict = Gateway().inspect(user_input)      # is the input safe?
if verdict.blocked:
    raise ValueError(verdict.reason)

hit = SemanticCache().get(user_input)        # answered this already?

prompt = ContextCompiler(ContextBudget(total_token_budget=1500)).compile(
    user_input, working, episodic, semantic
)
assert prompt.total_tokens <= 1500           # guaranteed, not hoped for
```

---

## Standards

| | |
|---|---|
| Language | Python 3.11+, CI matrix on 3.11, 3.12 and 3.13 |
| Typing | `mypy --strict` on all typed packages |
| Lint | `ruff`, rule set `E, F, I, UP, B, SIM, RUF` |
| Tests | 1,583 across nine repos, offline by default |
| CI | GitHub Actions per repo: lint, types and tests on three Python versions |
| License | MIT |

Built by [Yash Sabbavarapu](https://github.com/yashsabbavarapu).
