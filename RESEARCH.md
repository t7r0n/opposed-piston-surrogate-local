# Research And Plan Review

Company: Achates Power
Project: `OPSurrogate`

## Refined Thesis

Opposed-piston licensing needs fast, credible what-if exploration across fuel, load, emissions, and thermal constraints.

The implementation is intentionally local and synthetic, but the test harness is shaped around the real operating question: can the proposed artifact create evidence a founder, CTO, or product leader would immediately recognize as useful?

## Fresh Sources Checked

- https://achatespower.com/
- https://achatespower.com/approach/
- https://achatespower.com/achates-power-continues-development-of-opposed-piston-technology-for-us-army-vehicles-green-car-congress/

## Plan Excerpt Used

## The Gap

The single biggest constraint on opposed-piston design progress at Achates was never the *idea* — it was the **wall-clock cost of a 3D combustion CFD sweep in CONVERGE**. A meaningful piston-bowl geometry × injector-spray × scavenging DoE runs into hundreds of multi-hour CONVERGE jobs on a cluster, even with the ML-GGA loop Redon described in 2022 ([AMR PDF](https://www1.eere.energy.gov/vehiclesandfuels/downloads/2022_AMR/ace166_redon_2022_o_5-1_1203pm_ML.pdf)). Now that the IP sits inside GA-ASI's UAS program — which has fundamentally different operating conditions (high-altitude, intermittent throttle, hydrogen) — the cycle of *test a new operating envelope → re-run CFD sweeps → train a surrogate* needs to be **an order of magnitude faster** to be worth the airframe-integration runway. The gap is a **physics-informed neural surrogate, trained on the Achates CFD corpus, that predicts in-cylinder pressure trace, NOx and unburned-fuel emission, and scavenging efficiency in <100ms per operating point**, with quantified uncertainty so the optimizer knows when to escalate to a real CONVERGE run. This is precisely the *post-2022* version of what Redon's ML-GGA pipeline was reaching for, built with techniques that didn't exist on the prior project.

## The Project — `OPSurrogate`

> A physics-informed neural surrogate for opposed-piston engine combustion: predicts pressure trace, NOx, unburned-fuel and scavenging efficiency in under 100ms per operating point — with calibrated uncertainty so the design loop knows when to spend a CONVERGE job.

**What it is.** A PyTorch-based surrogate model, a thin Python design-loop runner (`opsurr`), and a tiny Streamlit cockpit. Inputs are the 14–18 parameters Achates already varies in DoEs (piston-bowl shape parameters, injector spray angle, injection timing, intake/exhaust port phasing, lambda, compression ratio, scavenging-blower set-point, EGR fraction, fuel composition incl. hydrogen). Outputs are the targets that matter for both the road-vehicle thesis (BSFC, NOx, soot) and the GA-ASI UAS thesis (altitude-adjusted scavenging efficiency, hydrogen-knock margin).

**Why it solves the gap.** Achates' own published pipeline ([DOE AMR](https://www1.eere.energy.gov/vehiclesandfuels/downloads/2022_AMR/ace166_redon_2022_o_5-1_1203pm_ML.pdf)) bottlenecks on CONVERGE job cost. A surrogate with <100ms inference and calibrated uncertainty cuts the iteration loop from days to seconds for 95% of the search and reserves real CONVERGE runs for the high-uncertainty 5%. For the GA-ASI hydrogen-UAS path — where the operating envelope is *new* and the prior CONVERGE corpus is *largely irrelevant* in the high-altitude regime — the uncertainty signal becomes the active-learning compass for which experiments and CFD sweeps to fund first.

**The wow moment.** A 30-second sweep across 5,000 candidate piston-bowl geometries on Lemke's laptop, color-mapped by NOx-vs-BSFC Pareto front, with the top-10 candidates' pressure traces overlaid against a real CONVERGE reference trace and a confidence band. Then a single click flags three points the surrogate is *not confident in* and queues real CONVERGE jobs for them. The total wall-clock from "what if we tried a wider piston bowl" to "here's the design and here's where we're guessing" is under one minute.

## Prototype Plan (the shippable demo)

**Surface:**
```bash
pip install opsurrogate
opsurr ingest --converge-dir ./cases --target imep,no_x,bsfc,scav_eff
opsurr train --backbone fno --epochs 200 --calibrate
opsurr sweep --param-file ranges.yaml --budget 5000
opsurr serve  # Streamlit cockpit
```

**Five demo inputs.** Synthesizing a public-domain proxy CONVERGE-style corpus (we'll use the published OP scavenging+combustion papers + a CONVERGE-tutorial-like Diesel sector geometry):
1. Baseline 10.6L heavy-duty OP at a standard road operating point — reproduces the public 20% efficiency claim ([Power Progress](https://www.powerprogress.com/news/act-expo-2024-achates-op-engine-shows-up-to-20-efficiency-gain/8037397.article)).
2. A piston-bowl-geometry sweep at fixed lambda — show the Pareto front.
3. A scavenging-blower set-point sweep at altitude — for the UAS path.
4. A hydrogen-fuel-fraction sweep at idle vs. cruise — for the Argonne hydrogen demo storyline.
5. An *out-of-distribution* point the surrogate flags and refuses to predict confidently — proving the uncertainty head is real.

**Expected output.** A Pareto front plot (BSFC vs. NOx) with 5000 points colored by confidence; the top 10 candidates with overlaid pressure traces + ground-truth CONVERGE reference; a JSON `next_jobs.json` listing the 3 highest-information CONVERGE jobs to run next.

**Proof metrics:**
- Inference latency: **<100ms** per operating point on laptop CPU.
- Pressure-trace RMSE: **<3% of peak in-cylinder pressure** on a held-out CONVERGE slice.
- Calibration: **80% of true CONVERGE outputs inside the predicted 80% interval** on the held-out slice.
- Active-learning sample efficiency: matches expert-greedy with **~40% fewer CONVERGE jobs** on a back-test.


## Build Acceptance Criteria

- Deterministic local fixtures.
- Domain-specific metrics and failure modes.
- Passing unit tests.
- Passing CLI verifier.
- Static dashboard generated locally.
- Benchmark output under the project `outputs/` folder.
- Public-safe README: no founder emails, no private outreach text, no credentials.
