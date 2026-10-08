# Problem Analysis Workshop A: Statistical Testing on Autonomous Vehicle Incidents

**PROG8431: Data Analysis, Mathematics, Modeling and Algorithms** · Team 3

| Name | Student ID |
| :--- | :--- |
| Sultan Atanda | 9114837 |
| Ricardo Mohammed | 7500382 |
| Alamir Ibrahim | 9104960 |

---

## Overview

This project applies hypothesis testing to real-world RoboTaxi (Autonomous Vehicle) crash data from the **NHTSA Standing General Order (SGO) 2021-01**. We ask one question:

> **Does a RoboTaxi's speed just before a crash differ depending on what it crashes into?**

- **Continuous variable:** `SV Precrash Speed (MPH)` (the Subject Vehicle's speed immediately before the crash)
- **Grouping variable:** `Crash_Interaction_Type` — **Motor Vehicle** vs. **Fixed Object**
- **Significance level:** α = 0.05

All analysis lives in a single reusable `SpeedAnalysis` class, and every chart is built with **Plotly** for interactive, presentation-ready visuals.

---

## Dataset

`autonomous_vehicle_incidents_cleaned.csv` — the cleaned dataset prepared in our previous analysis.

- **1,257 records, 14 features** (make, model year, automation system, city/state, roadway type, crash partner, pre-crash movements, pre-crash speed, ODD status, roadway and weather conditions, crash interaction type)
- Groups used in this workshop:
  - Motor Vehicle crashes: **n = 1,154**
  - Fixed Object crashes: **n = 70**
- Rows in the third interaction category are excluded from the group comparison but kept in the "All crashes" histogram and normality check.

---

## Hypotheses

| Test | H₀ | Hₐ |
| :--- | :--- | :--- |
| **Shapiro-Wilk** (normality) | Speed is normally distributed | Speed is not normally distributed |
| **F-test** (variances) | σ²<sub>Motor Vehicle</sub> = σ²<sub>Fixed Object</sub> | σ²<sub>Motor Vehicle</sub> ≠ σ²<sub>Fixed Object</sub> |
| **t-score** | The Motor Vehicle mean is typical of Fixed Object crashes | It is unusual |
| **t-test** (means) | μ<sub>Motor Vehicle</sub> = μ<sub>Fixed Object</sub> | μ<sub>Motor Vehicle</sub> ≠ μ<sub>Fixed Object</sub> |

---

## Analysis Pipeline

| Step | Method | Output |
| :--- | :--- | :--- |
| Summary | `summary()` | n, mean, median, SD, min, max, % at 0 mph per group |
| ① Graphical summary | `plot_histogram()` | Overall histogram + overlaid per-group histograms with mean lines |
| ② Normality | `normality_test()` | Q-Q plots (all / per group) + Shapiro-Wilk |
| ③ Equal variances | `f_test()` | Box plot + two-tailed F-test; picks Student's or Welch's t-test |
| ④ t-score | `t_score()` | T = (X − x̄) / (s/√n), plotted on the t-distribution |
| ⑤ Equal means | `t_test()` | Welch's t-test (manual + SciPy), t-distribution plot, means ± 95% CI |
| ⑥ Results | `results_table()` | Colour-coded table of every test and decision |

Each test is logged through `decide()`, so the final table always reflects exactly what was run.

---

## Results

### Summary statistics (pre-crash speed, mph)

| Group | n | Mean | Median | SD | Min | Max | % at 0 mph |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Motor Vehicle | 1,154 | 4.9 | 0.0 | 9.2 | 0 | 65 | 58.7 |
| Fixed Object | 70 | 9.7 | 8.5 | 7.6 | 1 | 29 | 0.0 |

### Hypothesis tests

| Test | Statistic | p-value | Decision |
| :--- | :--- | :--- | :--- |
| Shapiro-Wilk (All crashes) | W = 0.647 | p < 0.001 | Reject H₀ |
| Shapiro-Wilk (Motor Vehicle) | W = 0.615 | p < 0.001 | Reject H₀ |
| Shapiro-Wilk (Fixed Object) | W = 0.890 | p < 0.001 | Reject H₀ |
| F-test | F = 1.46 (df 1153, 69) | p = 0.048 | Reject H₀ → use Welch's |
| t-score | T = −5.26 (df 69) | p < 0.001 | Reject H₀ |
| Welch's t-test | t = −5.04 (df ≈ 82) | p < 0.001 | Reject H₀ |

**Mean difference:** −4.8 mph (95% CI −6.7 to −2.9)

### Interpretation

- **Speed is heavily right-skewed** with a large spike at 0 mph. Normality is rejected for every group, but with large samples the Central Limit Theorem makes the group means approximately normal, so a t-test is still reasonable.
- **Variances differ** (p = 0.048), so we use **Welch's t-test** instead of Student's.
- **The means differ significantly.** Fixed Object crashes happen at roughly twice the speed of Motor Vehicle crashes.

---

## 🎯 Key Takeaway

**RoboTaxis hit fixed objects at about twice the speed (9.7 vs 4.9 mph) of their crashes with other vehicles, and 59% of car-on-RoboTaxi crashes happen while the RoboTaxi is stopped.**

Most vehicle-on-RoboTaxi crashes are low-energy, stationary incidents (e.g., being rear-ended at a light), while fixed-object crashes involve the RoboTaxi itself moving.

---

## How to Run

1. Open `ProblemAnalysisWorkshopA.ipynb` in Google Colab or Jupyter.
2. Make sure `autonomous_vehicle_incidents_cleaned.csv` is in the same directory as the notebook.
3. Install dependencies (first code cell):
   ```bash
   pip install pandas numpy scipy plotly
   ```
4. Run all cells top to bottom.

To reuse the analysis on another variable or grouping:

```python
analysis = SpeedAnalysis(df, value_col="SV Precrash Speed (MPH)",
                         group_col="Crash_Interaction_Type",
                         groups=["Motor Vehicle", "Fixed Object"], alpha=0.05)
analysis.summary()
analysis.plot_histogram()
analysis.normality_test()
analysis.f_test()
analysis.t_score()
analysis.t_test()
analysis.results_table()
```

---

## Project Structure

```
.
├── ProblemAnalysisWorkshopA.ipynb           # Main analysis notebook
├── autonomous_vehicle_incidents_cleaned.csv # Cleaned NHTSA SGO dataset
└── README.md
```

---

## Tech Stack

- **Python 3**
- **pandas / NumPy** — data handling
- **SciPy** — Shapiro-Wilk, F-distribution, t-tests, Q-Q plots
- **Plotly** — interactive charts

---

## Limitations

- Group sizes are very unbalanced (1,154 vs 70), which reduces the power of the variance test; the F-test result (p = 0.048) sits close to the threshold.
- The F-test is sensitive to non-normal data; a Levene's test would be a more robust check.
- Given the strong skew, a non-parametric test (Mann-Whitney U) would be a useful confirmation.
- SGO reports are submitted by manufacturers and some fields are redacted, so reporting quality varies.

---

## Data Source

U.S. National Highway Traffic Safety Administration (NHTSA), *Standing General Order 2021-01: Incident Reporting for Automated Driving Systems (ADS) and Level 2 Advanced Driver Assistance Systems (ADAS)*.
