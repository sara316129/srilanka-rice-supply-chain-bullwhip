# Bullwhip Effect in Sri Lanka's Rice Supply Chain — An Agent-Based Model

**Group 05** — Agent-Based Modeling (AM 4086 / AM 4039 / FM 4054), University of Colombo, Semester I 2026
**Project advisor:** Dr. S. C. Surasinghe

An agent-based simulation of the Bullwhip Effect in the North Central Province (Anuradhapura and
Polonnaruwa districts) rice supply chain during Sri Lanka's 2022 economic crisis, built in NetLogo 6.4 and
visualized in AnyLogic (Personal Learning Edition).

---

## 1. Overview

This project models how a small, real-world demand/supply shock (fertilizer ban, fuel shortage, consumer
panic buying) gets amplified into much larger order swings as it moves upstream through a four-tier supply
chain: **Consumer → Retailer → Wholesaler → Farmer**, with an adaptive **Government** agent that can
intervene once food security drops below a threshold.

The base ordering model draws on:

> Chatfield, D. C., & Pritchard, A. M. (2013). Returns and the Bullwhip Effect. *Transportation Research Part E*, 49(1), 159–175.

## 2. Research Questions

1. How do the fertilizer ban, fuel shortage, and consumer panic signal interact to amplify demand variance
   across the chain?
2. Which government interventions most effectively reduce bullwhip amplification during a crisis?
3. How does agent-level behavioural heterogeneity (and parameter uncertainty) affect the magnitude and
   duration of the cascade?

## 3. Repository Structure

```
├── model/
│   ├── model.nlogox                  # Primary NetLogo 6.4 model
│   └── anylogic_visualization/       # AnyLogic PLE model (visualization only, see §6)
├── experiments/
│   ├── E0_baseline_validation.csv        # add once exported from BehaviorSpace
│   ├── E1_shock_decomposition.csv        # add once exported from BehaviorSpace
│   ├── E2_intervention_returns_panic.csv # add once exported from BehaviorSpace
│   ├── E3_sensitivity_sweeps.csv         # add once exported from BehaviorSpace
│   └── GovtOnOff_FSI_50runs-table.csv    # ✅ confirmed (FSI-threshold sensitivity check, 300 runs)
├── analysis/
│   ├── data_analysis.py              # pandas / scipy / Plotly analysis scripts
│   └── figures/                      # exported charts (time series, boxplots, histograms, heatmap)
├── docs/
│   ├── Experimental_Design_and_Data_Collection.pdf
│   ├── Section3_Data_Analysis_Report.pdf
│   └── Group_05_Assignment_08.pdf    # final written report
└── README.md
```

## 4. Model Summary

| Element | Description |
|---|---|
| Agents | Consumer, Retailer, Wholesaler, Farmer, Government (5 agent types) |
| Ordering policy | Order-Up-To, with lead time pipelines of 2 / 3 / 4 weeks per tier |
| Return allowance | ρ = 0.10 (baseline, from Chatfield & Pritchard 2013) |
| Government policy | Adaptive order-amplification cap, activates when Food Security Index (Φ) < threshold Φ* |
| Time horizon | 156 weekly steps (2021–2023, three calendar years) |
| Crisis shocks | Fertilizer ban (week 17), fuel shortage (week 65), consumer panic buying (weeks 74–90) |

## 5. Experiments

All experiments were run in NetLogo BehaviorSpace, table output mode (one CSV row per run/step pair).

| Experiment | Design | Runs | Research Question |
|---|---|---|---|
| **E0** — Baseline validation | All shocks off, government off | 10 | Validation (confirms stable low-variance equilibrium absent any shock) |
| **E1** — Shock decomposition | 2×2×2 factorial (fertilizer × fuel × panic), government off | 400 | RQ1 |
| **E2** — Intervention × returns × panic | 2×3×3 factorial (government × ρ × panic-rate), primary experiment | 900 | RQ2, RQ3 |
| **E3** — One-factor-at-a-time sensitivity | α, ρ, panic-rate, fsi-threshold each ±20% around baseline | 360 | RQ3 |
| **FSI-threshold sensitivity** *(separate check)* | 2×3 factorial (government × fsi-threshold ∈ {0.2, 0.4, 0.6}) | 300 | RQ2, RQ3 (supplementary) |

**Note on headline results — two related but distinct experiments:**

- The **primary E2 result**, at the calibrated default `fsi-threshold = 0.4`, found a **62.6% reduction**
  in the farmer-tier Bullwhip Ratio (67.04 → 25.08), based on 100 runs (50 government-ON, 50 government-OFF).
- A **separate supplementary experiment** (`GovtOnOff_FSI_50runs-table.csv`, 300 runs total) tested three
  `fsi-threshold` values and corroborates this at the same threshold: **60.03% reduction** (63.55 → 25.40,
  Welch's t-test p = 9.6×10⁻³⁰). It additionally shows the effect is **threshold-dependent**: at
  `fsi-threshold = 0.2`, intervention is statistically indistinguishable from doing nothing (p = 0.635),
  because the government activates too late in the crisis timeline to prevent order amplification.
- Both figures are correct for their respective experiments; they are not in conflict. The final report
  cites both explicitly by experiment name to avoid ambiguity.

## 6. Tooling

- **NetLogo 6.4** — primary modeling and BehaviorSpace tool; source of all statistical results reported.
- **AnyLogic PLE** — used for visualization only (supervisor-approved). Due to Personal Learning Edition
  replication limits, AnyLogic output is directionally consistent with NetLogo but does not match its exact
  magnitude, and is not used for any statistical claim in the final report.
- **Python** (pandas, scipy, Plotly) — post-processing of BehaviorSpace CSV exports, significance testing,
  and chart generation.

## 7. Data Sources

| Source | Used for |
|---|---|
| Central Bank of Sri Lanka, Annual Report 2022 (Chapter 4) | Crisis shock timing calibration |
| Department of Census and Statistics (DCS), Paddy Statistics 2021/22 Maha Season | Farmer-tier capacity shock calibration (`fertilizer-shock`) |
| World Food Programme (WFP), weekly rice price data | Qualitative validation of shock timing only |

The `fertilizer-shock` multiplier is **0.627**, derived from the DCS-reported Maha season yield decline of
4,492 → 2,819 kg/ha (≈37%), per the correction documented in *Experimental Design and Data Collection*
(Section 1). This value is used consistently across the model default, all BehaviorSpace experiments, and
the final report.

## 8. Key Parameters

| Parameter | Symbol | Default | Source |
|---|---|---|---|
| Return allowance | ρ | 0.10 | Chatfield & Pritchard (2013) |
| Demand smoothing | α | 0.25 | Chatfield & Pritchard (2013) |
| FSI activation threshold | Φ* | 0.40 | WFP food-security reporting threshold |
| Fertilizer shock capacity | — | 0.627 | DCS Paddy Statistics 2021/22 Maha (4,492→2,819 kg/ha, ≈37% decline) |
| Fuel shock capacity | — | 0.75 | Estimated (2022 fuel crisis logistics impact) |
| Consumer panic rate | — | 1.02/week | Estimated, calibrated to WFP price-spike timing |

Parameters marked "estimated" are acknowledged as such in the Discussion/Limitations section of the final
report, distinct from parameters grounded directly in cited data.

## 9. How to Reproduce

1. Open `model/model.nlogox` in NetLogo 6.4.
2. Tools → BehaviorSpace → select an experiment (E0–E3, or `GovtOnOff_FSI_50runs`) → Run.
3. Export table-format CSV to `experiments/`.
4. Run `analysis/data_analysis.py` to regenerate summary statistics and figures.

Alternatively, experiments can be run headlessly:

```bash
netlogo-headless.sh --model model/model.nlogox --experiment E2 --table experiments/E2_output.csv
```

## 10. Group Members & Contributions

| Member | Responsibility |
|---|---|
| Member 1 | Chatfield & Pritchard (2013) — base ordering model, return allowance (ρ), NetLogo lead implementation |
| Member 2 | *(add paper/contribution)* |
| Member 3 | Python data analysis, statistical testing, visualization |
| Member 4 | *(add paper/contribution)* |
| Member 5 | *(add paper/contribution)* |

*(Full individual contribution statements are in the final report, Component 6.)*

## 11. References

- Chatfield, D. C., & Pritchard, A. M. (2013). Returns and the Bullwhip Effect. *Transportation Research Part E*, 49(1), 159–175.
- Lee, H. L., Padmanabhan, V., & Whang, S. (1997). Information Distortion in a Supply Chain: The Bullwhip Effect. *Management Science*, 43(4), 546–558.
- Central Bank of Sri Lanka. (2022). *Annual Report 2022*.
- Department of Census and Statistics, Sri Lanka. *Paddy Statistics, 2021/22 Maha Season*.
- World Food Programme. *Sri Lanka weekly food price monitoring data*.
