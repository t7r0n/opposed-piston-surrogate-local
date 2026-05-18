# Opposed-Piston Surrogate Local

Opposed-piston licensing needs fast, credible what-if exploration across fuel, load, emissions, and thermal constraints.

The implementation is a laptop-scale proof of the workflow behind that claim, with `engine point` fixtures and falsifiable gates.

## Engineering read

Offline engine-cycle surrogate for opposed-piston efficiency, emissions, and calibration tradeoffs.

## What is measured

- Compiles 180 replayable `engine point` fixtures that make the `OPSurrogate` assumptions observable.
- Treats `nox_reduction`, `bsfc_delta`, `thermal_margin`, and `calibration_stability` as release gates, not dashboard decoration.
- Plants degraded cases for `nox_spike`, `thermal_limit`, `fuel_map_hole`, and `transient_smoke` and checks whether the harness catches them.
- Exports the `Opposed-Piston Surrogate Local` run as structured reports, static HTML, benchmark numbers, and a shareable package.

## One-pass run

```bash
uv sync --extra dev
uv run op-surrogate init-demo --force
uv run op-surrogate run-suite
uv run op-surrogate verify
uv run op-surrogate dashboard
uv run op-surrogate benchmark --iterations 100
uv run op-surrogate export-demo-pack
```

## Evidence packet

- `data/scenarios.json`
- `outputs/summary.json`
- `outputs/reports.json`
- `outputs/evidence_pack.md`
- `outputs/dashboard.html`
- `outputs/benchmark.json`
- `outputs/demo-pack.zip`

## Checks

```bash
uv run ruff check .
uv run pytest -q
uv run op-surrogate run-suite
uv run op-surrogate verify
uv run op-surrogate benchmark --iterations 100
```

## No-secrets boundary

The `opposed-piston-surrogate-local` public surface is source, tests, lockfile, and docs. It does not need credentials, browser state, customer records, or hosted services.
