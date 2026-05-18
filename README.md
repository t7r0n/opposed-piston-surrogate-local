# Opposed-Piston Surrogate Local

Offline engine-cycle surrogate for opposed-piston efficiency, emissions, and calibration tradeoffs.

This is a local-first, synthetic-data prototype inspired by a company-specific project plan for **Achates Power**. It is built to demonstrate the engineering shape of `OPSurrogate` without private data, credentials, external APIs, or hosted services.

## Why it matters

Opposed-piston licensing needs fast, credible what-if exploration across fuel, load, emissions, and thermal constraints.

## What it does

- Generates deterministic synthetic `engine point` scenarios.
- Scores each scenario against domain-specific quality gates.
- Produces evidence-backed findings for realistic failure modes.
- Writes a static dashboard, JSON reports, benchmark output, and a portable demo pack.
- Exposes a JSONL tool loop for local agent integration.

## Metrics

- `nox_reduction`
- `bsfc_delta`
- `thermal_margin`
- `calibration_stability`

## Failure modes

- `nox_spike`
- `thermal_limit`
- `fuel_map_hole`
- `transient_smoke`

## Quickstart

```bash
uv sync --extra dev
uv run op-surrogate init-demo --force
uv run op-surrogate run-suite
uv run op-surrogate verify
uv run op-surrogate dashboard
uv run op-surrogate benchmark --iterations 100
uv run op-surrogate export-demo-pack
```

## Expected outputs

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Validation

```bash
uv run ruff check .
uv run pytest -q
uv run op-surrogate run-suite
uv run op-surrogate verify
uv run op-surrogate benchmark --iterations 100
```

## Demo hook

A calibration map reveals the Pareto frontier between NOx, brake-specific fuel consumption, and thermal margin.
