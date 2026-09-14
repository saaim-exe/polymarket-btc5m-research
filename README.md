# Polymarket BTC five-minute research

Estimate the probability of an official Up outcome and compare predictions with the market's implied probability.

## Start here

- [Revised V0 architecture](docs/v0-architecture-revised.md): planning proposal, audit gates and open decisions.
- [HF data review](docs/hf-data-architecture-review.md): dated evidence and limitations behind the architecture.
- [Experimental notebook](notebooks/v0.ipynb): the working space for V0 logistic-regression experiments.
- [Existing hypotheses](notebooks/hypothesis.md): exploratory notes; consult the revised architecture for current planning constraints.

## Project layout

| Path | Purpose |
|---|---|
| `notebooks/` | Interactive experiments; keep V0 exploration here |
| `docs/` | Architecture, planning and durable decisions |
| `configs/` | Future dataset manifests, experiment settings and sourced settlement rules |
| `data/raw/` | Local immutable source snapshots |
| `data/interim/` | Local cleaned/aligned intermediate data |
| `data/processed/` | Local experiment-ready decision rows |
| `reports/` | Audit and evaluation summaries |
| `reports/figures/` | Exported research figures |
| `src/ingestion/` | Future revision-pinned acquisition |
| `src/cleaning/` | Future schema checks, deduplication and alignment |
| `src/labeling/` | Future canonical labels, availability and verified-rule QA |
| `src/features/` | Future causal decision rows and feature provenance |
| `src/splits/` | Future chronological evaluation boundaries |
| `src/models/` | Future baselines and learned models |
| `src/backtest/` | Future execution-aware economic evaluation |
| `tests/` | Future meaningful temporal and data-contract checks |

Folders are planning placeholders, not implemented Python packages. The existing `src/main.py` is retained. No training, ingestion or test code has been added.

## Tooling

`environment.yml` remains the existing package/environment definition, unchanged. Use the `btc_research` environment as the notebook kernel. VS Code extension suggestions are recorded in `.vscode/extensions.json`; they do not install extensions or change the environment.

Local datasets, Python caches and notebook checkpoints are ignored by Git. Keep small audit summaries and data-revision manifests under version control; do not commit downloaded datasets.

Next planning step: audit a pinned HF revision for BTC/five-minute dates, labels and usable feature coverage. Settlement-rule claims and deployment relevance remain unverified until their architecture gates pass.
