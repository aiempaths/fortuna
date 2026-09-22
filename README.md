[README.md](https://github.com/user-attachments/files/32540739/README.md)
# Fortuna
**Fortuna** is a local engine for univariate telemetry: you send a series as JSON, it scores recent trajectory geometry against similar past windows, and it returns a JSON receipt. This repository is the **public contract** (docs + schemas only).
Contact operator for a POST a request that matches these schemas and get a receipt back. Using an AI model is optional and entirely on your side; Fortuna itself has no AI integration.
## Audience: research engineers and technical PMs on sensor/telemetry stacks (space weather, industrial SCADA, env/ops monitoring).
They feed a univariate series, read `d_rel` / `purity` / persistence comparison from JSON, and keep compute local or on-prem. They want receipts and "show me vs a dumb baseline" — not a SaaS demo or built-in AI.

Full definition: [docs/audience.md](docs/audience.md).
## What Fortuna does
1. Take a univariate numeric series (oldest → newest).
2. Build short window features that describe the shape of the recent trajectory.
3. Find past windows in the same volatility regime with similar geometry.
4. Form a neighbour vote for the forward excursion label, plus reliability metrics (`d_rel`, `purity`).
5. Optionally compare those decisions to horizon-matched persistence and return a receipt.

### Default engine knobs
| Parameter | Default | Meaning |
|-----------|---------|---------|
| `window` | `40` | Length of the lookback window |
| `horizon` | `8` | Steps ahead for the excursion label / forecast |
| `warmup` | `0.4` | Fraction of the series skipped before scoring |
| `k` | `5` | Neighbour count |
| `vote_threshold` | `0.35` | Predict positive if mean neighbour vote ≥ this |
| `theiler` | `0` | Exclude history closer than this many steps (`0` = off) |
### Reliability gate defaults
Provisional (recalibrate per domain):
| Metric | Default rule |
|--------|----------------|
| `purity` | ≥ `0.75` |
| `d_rel` | ≤ `0.180` |

## Modes
- **`diagnostics`** — run the engine; return aggregate `d_rel` / `purity`, gate pass/fail, and optional per-step rows. Does **not** claim predictive edge by itself.
- **`predict_vs_persistence`** — same decisions vs horizon-matched persistence (`p[t] = a[t - horizon]`). Returns accuracy / MCC-style summary fields when both classes exist.
## How to call (once the local server exists)
```bash
curl -s -X POST "$FORTUNA_BASE_URL/v0/jobs" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $FORTUNA_API_KEY" \
  -d @examples/request_diagnostics.json
FORTUNA_BASE_URL is not published in this repo. The operator provides it (often http://127.0.0.1:8787 on their machine).

Files in this repo
Path	Role
schemas/request.schema.json	Request contract
schemas/response.schema.json	Response contract
docs/field-meanings.md	Plain-language meaning of every field
docs/audience.md	Who this is for
examples/request_diagnostics.json	Copy-paste diagnostics job
examples/response_pass.json	Example successful receipt
What this is not
Not a cloud runner and not GitHub Actions compute
Not an LLM product
Not a private research vault dump
Not a guarantee that gate pass equals forecasting skill

Version
api_version: 0.1.0 (Fortuna contract)
engine_id: fortuna
probe_profile: fortuna-engine-defaults-v1 (window=40, horizon=8, warmup=0.4, k=5)
