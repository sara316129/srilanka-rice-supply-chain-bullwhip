# Bullwhip Effect in Sri Lanka's Rice Supply Chain — An Agent-Based Model

**Group 05** — Agent-Based Modeling (AM 4086 / AM 4039 / FM 4054), University of Colombo, Semester I 2026
**Project advisor:** Dr. S. C. Surasinghe

An agent-based simulation of the Bullwhip Effect in the North Central Province (Anuradhapura and
Polonnaruwa districts) rice supply chain during Sri Lanka's 2022 economic crisis, built in NetLogo.

---

## 1. Overview

This project models how a small, real-world demand/supply shock (fertilizer ban, fuel shortage,
consumer panic buying) gets amplified into much larger order swings as it moves upstream through a
four-tier supply chain: **Consumer → Retailer → Wholesaler → Farmer**, with an adaptive
**Government** agent that can intervene once food security drops below a threshold.

The base ordering model draws on:

> Chatfield, D. C., & Pritchard, A. M. (2013). Returns and the Bullwhip Effect. *Transportation Research Part E*, 49(1), 159–175.

## 2. Research Questions

1. How do the fertilizer ban, fuel shortage, and consumer panic signal interact to amplify demand
   variance across the chain?
2. Which government interventions most effectively reduce bullwhip amplification during a crisis?
3. How does agent-level behavioural heterogeneity (and parameter uncertainty) affect the magnitude
   and duration of the cascade?

## 3. Repository Structure

```
├── model.nlogox                      # NetLogo model (final, corrected version — see §8)
├── main.tex                          # Final report (LaTeX, SIURO template)
├── references.bib
├── figures/
│   ├── e1_shock_decomposition.png
│   ├── rho_effect.png
│   ├── heatmap_mean_bwr.png
│   ├── boxplot_final_distribution.png
│   └── bullwhip_by_threshold.png
├── experiments/
│   ├── E0-baseline-validation-table.csv
│   ├── E1-shock-decomposition-table.csv
│   ├── E2-intervention-returns-panic-table.csv       # post-fix (see §8)
│   ├── FSI-threshold-check-table.csv                 # post-fix (see §8)
│   ├── E3-sensitivity-alpha-table.csv
│   ├── E3-sensitivity-panic-rate-table.csv
│   └── E3-sensitivity-fsi-threshold-table.csv
│       # Note: an E3-sensitivity-rho sweep exists but predates the returns-mechanism
│       # correction (§8) and is NOT included; a corrected re-run is listed as future work.
├── analysis/
│   └── data_analysis.py              # pandas / scipy / matplotlib analysis scripts
└── README.md
```

## 4. Model Summary

| Element | Description |
|---|---|
| Agents | Consumer, Retailer, Wholesaler, Farmer, Government (5 agent types) |
| Ordering policy | Order-Up-To, with lead time pipelines of 2 / 3 / 4 weeks per tier |
| Return allowance | ρ ∈ {0, 0.10, 0.20}; modeled as a stochastic shock on the order quantity (see §8) |
| Government policy | Adaptive order-amplification cap, activates when Food Security Index (Φ) < threshold Φ* |
| Time horizon | 156 weekly steps (2021–2023, three calendar years) |
| Crisis shocks | Fertilizer ban (week 17), fuel shortage (week 65), consumer panic buying (weeks 74–90) |

## 5. Experiments

All experiments were run in NetLogo BehaviorSpace, table output mode (one CSV row per run/step pair).

| Experiment | Design | Runs | Research Question |
|---|---|---|---|
| **E0** — Baseline validation | All shocks off, government off | 10 | Validation |
| **E1** — Shock decomposition | 2×2×2 factorial (fertilizer × fuel × panic), government off | 400 | RQ1 |
| **E2** — Intervention × returns × panic | 2×3×3 factorial (government × ρ × panic-rate) | 900 | RQ2, RQ3 |
| **E3** — One-factor-at-a-time sensitivity | α, panic-rate, fsi-threshold each ±20% around baseline | 270 | RQ3 |
| **FSI-threshold check** | 2×3 factorial (government × fsi-threshold ∈ {0.2, 0.4, 0.6}) | 300 | RQ2, RQ3 |

**Note:** an E3 sweep for ρ was originally run but predates the returns-mechanism correction
described in §8; it has been excluded rather than reported, and a corrected re-run is listed as
future work in the final report.

### Key results

- **Government intervention** reduces the farmer-tier Bullwhip Ratio by **61.3%** at calibrated
  default settings (ρ=0.1, panic-rate=1.02): 74.53 (off) → 28.85 (on), Welch's t-test
  p = 9.9×10⁻²⁹.
- **Threshold sensitivity**: intervention is effective at fsi-threshold ≥ 0.4 (65–67% reduction,
  p < 10⁻²⁴) but statistically ineffective at fsi-threshold = 0.2 (p = 0.39) — activating too late
  in the crisis timeline provides no measurable benefit.
- **Return allowance (ρ)**: increasing ρ increases the farmer-tier Bullwhip Ratio (72.8 → 74.0 →
  86.5 as ρ rises from 0 to 0.2), consistent in direction with Chatfield & Pritchard (2013),
  though only clearly significant at ρ=0.2.
- **Shock decomposition (E1)**: no single crisis shock (fertilizer, fuel, or panic) produces a
  statistically significant main effect in isolation (all p > 0.05); amplification appears to
  emerge from interaction with the ordering policy and government-response mechanism rather than
  from any one shock alone. See the final report's Discussion for interpretation.

## 6. Tooling

- **NetLogo** — primary modeling and BehaviorSpace tool; source of all statistical results and
  figures reported.
- **Python** (pandas, scipy, matplotlib) — post-processing of BehaviorSpace CSV exports,
  significance testing (Welch's t-test), and all chart generation.
- An earlier supplementary AnyLogic PLE model was also built for cross-checking purposes only; it
  is not included in this repository and no statistical claim in the final report is drawn from it.

## 7. Data Sources

| Source | Used for |
|---|---|
| Central Bank of Sri Lanka, Annual Report 2022 | Crisis shock timing calibration |
| Department of Census and Statistics (DCS), Paddy Statistics 2021/22 Maha Season | Farmer-tier capacity shock calibration (`fertilizer-shock` = 0.627) |
| World Food Programme (WFP), weekly rice price data | Qualitative validation of shock timing; FSI threshold convention |

The `fertilizer-shock` multiplier is **0.627**, derived from the DCS-reported Maha season yield
decline of 4,492 → 2,819 kg/ha (≈37%). This value is used consistently across the model default,
all BehaviorSpace experiments, and the final report.

## 8. Model Correction History

Two implementation issues were identified and corrected during development; both are disclosed
here for transparency and reproducibility.

1. **Consumer panic-rate overwrite (fixed):** in an earlier version, the panic-rate multiplier was
   applied to consumer demand, but demand was then immediately overwritten by a fresh random draw
   in the next line, silently discarding the multiplier's effect. Fixed by combining both into a
   single assignment.
2. **Returns mechanism direction (fixed):** in an earlier version, returns were added directly to
   on-hand inventory (`inventory + rho * orders_placed`), which — combined with the Order-Up-To
   formula's subtraction of inventory — caused higher ρ to mechanically *reduce* order variance,
   the opposite of Chatfield & Pritchard's (2013) reported effect. The mechanism was corrected to
   apply returns as a stochastic shock directly on the order quantity itself
   (`order_qty = base_order + rho * base_order * N(0,1)`), which at ρ=0 is identical to the
   no-returns case and, as ρ increases, injects unpredictable order-quantity volatility rather than
   a predictable inventory buffer. This is reported in the final report's Model Implementation
   section (Eq. for returns) and confirmed to reproduce the correct effect direction (§5).

All experiment CSVs in `experiments/` reflect the model **after** both corrections, with the
exception noted in §5 regarding the E3 ρ-sweep.

## 9. How to Reproduce

1. Open `model.nlogox` in NetLogo.
2. Tools → BehaviorSpace → select an experiment (E0, E1, E2, E3, or FSI-threshold-check) → Run.
3. In the run dialog, enable **Table output** (not Spreadsheet) and export to `experiments/`.
4. Run `analysis/data_analysis.py` to regenerate summary statistics and figures.

## 10. Group Members & Contributions

| Member | Responsibility |
|---|---|
| Member 1 | Chatfield & Pritchard (2013) — base ordering model, return allowance mechanism, NetLogo implementation and bug diagnosis/correction (§8) |
| Member 2 | Dominguez, Cannella & Framinan (2015) — supply chain structure literature |
| Member 3 | Python data analysis, statistical testing, visualization |
| Member 4 | Qu & Raff (2017) — centralized vs. decentralized inventory control literature |
| Member 5 | Moyaux, Chaib-draa & D'Amours (2007) — information sharing literature |

*(Full individual contribution statements, including each member's student index number and
personal understanding-of-project summary, are in the final report, Section "Individual
Contribution Statement".)*

## 11. References

- Chatfield, D. C., & Pritchard, A. M. (2013). Returns and the Bullwhip Effect. *Transportation Research Part E*, 49(1), 159–175.
- Lee, H. L., Padmanabhan, V., & Whang, S. (1997). Information Distortion in a Supply Chain: The Bullwhip Effect. *Management Science*, 43(4), 546–558.
- Dominguez, R., Cannella, S., & Framinan, J. M. (2015). The impact of the supply chain structure on bullwhip effect. *Applied Mathematical Modelling*, 39, 7309–7325.
- Moyaux, T., Chaib-draa, B., & D'Amours, S. (2007). Information Sharing as a Coordination Mechanism for Reducing the Bullwhip Effect in a Supply Chain. *IEEE Transactions on Systems, Man, and Cybernetics—Part C*, 37(3), 396–409.
- Qu, Z., & Raff, H. (2017). Centralized versus decentralized inventory control in supply chains and the bullwhip effect. CEPIE Working Paper No. 17/17.
- Central Bank of Sri Lanka. (2022). *Annual Report 2022*.
- Department of Census and Statistics, Sri Lanka. *Paddy Statistics, 2021/22 Maha Season*.
- World Food Programme. *Sri Lanka weekly food price monitoring data*.
