# Research And Plan Review

Project: Opposed-Piston Surrogate Local
Project: `OPSurrogate`

## Refined Thesis

Opposed-piston licensing needs fast, credible what-if exploration across fuel, load, emissions, and thermal constraints.

The implementation is intentionally local and synthetic, but the test harness is shaped around the real operating question: can the proposed artifact create evidence a founder, CTO, or product leader would immediately recognize as useful?

## Plan Excerpt Used

## The Gap

The single biggest constraint on opposed-piston design progress is not the core engine idea; it is the wall-clock cost of a 3D combustion CFD sweep in CONVERGE. A meaningful piston-bowl geometry, injector-spray, and scavenging design of experiments can run into hundreds of multi-hour jobs on a cluster. When the operating envelope changes, the cycle of testing a new envelope, re-running CFD sweeps, and training a surrogate needs to be much faster to be useful. The gap is a physics-informed surrogate that predicts in-cylinder pressure trace, NOx, unburned-fuel emission, and scavenging efficiency in under 100ms per operating point, with quantified uncertainty so the optimizer knows when to escalate to a real CONVERGE run.

## The Project — `OPSurrogate`

> A physics-informed neural surrogate for opposed-piston engine combustion: predicts pressure trace, NOx, unburned-fuel and scavenging efficiency in under 100ms per operating point — with calibrated uncertainty so the design loop knows when to spend a CONVERGE job.

**What it is.** A PyTorch-based surrogate model, a thin Python design-loop runner (`opsurr`), and a tiny Streamlit cockpit. Inputs are the 14-18 parameters an opposed-piston team would vary in design studies: piston-bowl shape parameters, injector spray angle, injection timing, intake/exhaust port phasing, lambda, compression ratio, scavenging-blower set-point, EGR fraction, and fuel composition. Outputs are the targets that matter for road and aerospace-style operating envelopes: BSFC, NOx, soot, altitude-adjusted scavenging efficiency, and knock margin.

**Why it solves the gap.** A surrogate with under-100ms inference and calibrated uncertainty cuts the iteration loop from days to seconds for most of the search and reserves real CONVERGE runs for the high-uncertainty points. When the operating envelope is new, the uncertainty signal becomes the active-learning compass for which experiments and CFD sweeps to fund first.

**The wow moment.** A 30-second sweep across 5,000 candidate piston-bowl geometries on a laptop, color-mapped by NOx-vs-BSFC Pareto front, with the top-10 candidates' pressure traces overlaid against a reference trace and a confidence band. Then a single click flags three points the surrogate is not confident in and queues real CFD jobs for them. The total wall-clock from "what if we tried a wider piston bowl" to "here is the design and here is where we are guessing" is under one minute.

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
1. Baseline heavy-duty OP at a standard road operating point.
2. A piston-bowl-geometry sweep at fixed lambda — show the Pareto front.
3. A scavenging-blower set-point sweep at altitude — for the UAS path.
4. A hydrogen-fuel-fraction sweep at idle vs. cruise.
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
