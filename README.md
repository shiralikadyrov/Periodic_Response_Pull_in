[README.md](https://github.com/user-attachments/files/34589421/README.md)
# Periodic response and pull-in analysis of a graphene-motivated electrostatic MEMS model — code and parameters

Companion code for O. Eshkobilov and S. Kadyrov, *Periodic Response and Pull-in Analysis of a
Graphene-Motivated Electrostatic MEMS Model*, Communications in Nonlinear Science and Numerical
Simulation (manuscript CNSNS-D-26-01407).

Model (dimensionless):

    u'' + 2 zeta0 u' + u + beta u^3 = lam (1 + rho cos(Omega tau))^2 / (1 - u)^2,   u < 1

## Contents

| Path | Purpose |
|---|---|
| `notebooks/01_master_analysis.ipynb` | Production notebook. Cells 1–17: library (full ODE, shooting/Floquet, exact- and truncated-force HB, pseudo-arclength continuation, exact-rational Sturm construction, basin/DIC). Cells 18–43: every study in the article, regression checks, metadata. Shipped with outputs from the production run. |
| `notebooks/02_figures_4_5_layout.ipynb` | Regenerates Figures 4–5 with the final panel layout. The integrity counts produced by cell 34 of notebook 01 are embedded in the notebook. |
| `scripts/timing_benchmark.py` | Standalone timing comparison (Table 7). Loads library cells 1–17 from the master notebook, so it runs exactly the production functions. |
| `parameters/numerical_protocol.json` | Model defaults, shooting ODE tolerances, basin/DIC protocol, HB gates, random seed, run profile. |
| `parameters/study_parameters.json` | Parameter sets for each figure and table. Exact rationals are given as strings. |
| `results/timing_benchmark_*.csv` | Per-load and summary timings reported in Table 7. |

## Map from article to code

| Article item | Notebook cell(s) / script | Function(s) |
|---|---|---|
| Table 1 (dimensional device example) | cell 38 | `davidovikj_device1_calibration` |
| Section 4.1 exact Sturm example (M = 4) | cell 19 | `build_exact_surrogate`, `exact_sturm_report`, `recover_physical_resultant_roots` |
| Figure 2, Table 2 (near-critical K = 1,2,3,5,7) | cells 27–28 | `multi_harmonic_near_critical_study` |
| Table 3 (numerical protocol) | `parameters/numerical_protocol.json` | `BasinProtocol`, `ODEOptions` |
| Table 4, Figure 3 (cross-level validation) | cells 21–22 | `cross_level_validation` |
| Figures 4–5 (damping sweep, Dynamical Integrity Chart) | cell 34 (set `CNSNS_RUN_FULL_DIC=1`), then notebook 02 | `compute_integrity_chart`, `_plot_integrity_chart_arrays` |
| Table 5 (basin-index robustness) | cells 32–33 | `basin_robustness_sweep`, `classify_basin_branches` |
| Section 5.3 continuation, Figure 6 | cell 27 | `hardening_continuation_study`, `refine_hb_fold` |
| Section 5.3 fold/Jacobian verification | cell 30 | `build_symbolic_surrogate_with_lambda`, `refine_inphase_surrogate_fold` |
| Table 6 (original-force truncation) | cells 24–25 | `truncation_accuracy_study` |
| Figure 7 (branch-resolved sampling) | cells 40–41 | `generate_figure9_samples` (seed 42) |
| Table 7 (timing comparison) | `scripts/timing_benchmark.py` | — |

## Running

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# full production profile (default) or a quick end-to-end check:
export CNSNS_RUN_PROFILE=review      # or: smoke
export CNSNS_RUN_FULL_DIC=1          # optional: recompute Figures 4-5 (most expensive step)
jupyter nbconvert --to notebook --execute notebooks/01_master_analysis.ipynb --output executed.ipynb
python scripts/timing_benchmark.py
```

Outputs are written to `/content/...` on Colab and to `/mnt/data/CNSNS_second_revision_outputs`
otherwise; edit `OUTPUT_DIR` in cell 1 to change this. Cell 43 runs scientific regression checks
against the values reported in the article.

The production run used Python 3.12.13, NumPy 2.0.2, SciPy 1.16.3, SymPy 1.14.0, pandas 2.2.2 and
Matplotlib 3.10.0. The timing benchmark used Python 3.11.18, NumPy 2.4.6, SciPy 1.17.1 and
SymPy 1.14.0 on an arm64 macOS machine. Timings depend on hardware. Exact Sturm counts do not:
they use rational arithmetic throughout.

## Scope

Only the surrogate-polynomial Sturm count is exact. Basin and DIC indices are finite-horizon grid
fractions that depend on the sampling domain and protocol; they are not safety probabilities. The
dimensional device example (beta ≈ 381.7) checks scale consistency and does not calibrate the generic
sweeps (beta ≤ 5).
