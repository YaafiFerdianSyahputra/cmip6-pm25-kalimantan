# Statistical Calibration of CMIP6-Based Daily PM2.5 Ensemble Simulations over Fire-Prone Kalimantan, Indonesia

This repository holds the analysis notebook and processed data for the article

> D. Y. Faidah and Y. F. Syahputra, "Statistical Calibration of CMIP6-Based Daily PM2.5 Ensemble Simulations over Fire-Prone Kalimantan, Indonesia," *Jurnal RESTI (Rekayasa Sistem dan Teknologi Informasi)*, submitted.

The notebook `analysis_calibration.ipynb` runs the full analysis from the processed provincial data. It evaluates daily surface PM2.5 from the CMIP6 model MPI-ESM-1-2-HAM (SSP3-7.0, three members) against the MERRA-2 aerosol reanalysis for the five provinces of Indonesian Kalimantan. It then calibrates the ensemble with two distribution-based methods, validates them out of sample, and applies the selected method to September 2026 – January 2027. Every table and figure in the article (Tables 2–4, Figures 1–9) comes from this notebook.

## Repository structure

```
├── analysis_calibration.ipynb    analysis notebook (executed, outputs saved)
├── data/
│   ├── pm25_daily_province.csv
│   └── merra2_pm25_components_daily_province.csv
├── outputs/
│   ├── tables/                   result tables (CSV)
│   └── figures/                  Figure1.png – Figure9.png, as printed in the article
├── requirements.txt
└── README.md
```

## Running the notebook

```bash
pip install -r requirements.txt
jupyter notebook analysis_calibration.ipynb
```

**Google Colab.** Open the notebook in Colab and run all cells. The first cell opens an upload dialog; select the two CSV files from `data/` and, if you have them, the GADM boundary files. The files are moved into a `data/` folder in the Colab session, so Google Drive is not needed. The CSV files have to be uploaded again whenever a new Colab session starts.

**Local Jupyter.** Run all cells from the repository root. A full run takes a few minutes, and all outputs are written to `outputs/`.

Figures 1 and 8 need the GADM 4.1 provincial boundaries of Indonesia (level 1). On the first run the notebook downloads them as GeoJSON (`data/gadm41_IDN_1.json`). If the download fails, get the level-1 file for Indonesia from https://gadm.org and put it in `data/`, either as GeoJSON (`gadm41_IDN_1.json`) or as a shapefile (`gadm41_IDN_1.shp` with its `.shx`, `.dbf`, `.prj` and `.cpg` files). The boundaries are not included in the repository because the GADM licence does not allow redistribution, and `.gitignore` keeps them out of version control.

The figures use Arial, or Liberation Sans where Arial is not installed. The two fonts have the same metrics, so the layout matches the article; with Liberation Sans, the GeoJSON boundaries and the package versions in `requirements.txt`, the figures in `outputs/figures/` are reproduced pixel for pixel. The shapefile gives the same maps with slight differences in how the coastline is drawn.

## Notebook sections

| Section | Content | Article |
|---|---|---|
| 1. Setup | Packages, paths, constants, figure style | – |
| 2. Data | Data loading, completeness checks, seasons, calibration and projection periods | Section 2.1 |
| 3. Study Area and Workflow | Study area map with the MERRA-2 and model grids; workflow diagram | Figures 1–2 |
| 4. Descriptive Statistics | Statistics per province and season; daily series; monthly climatology | Table 2, Figures 3–4 |
| 5. Evaluation of the Raw Ensemble | Synchrony test (daily anomaly correlation); bias, spread, rank histogram, coverage, CRPS | Section 3.2 |
| 6. Calibration Methods | Variance scaling in log space (VS) and multiplicative quantile delta mapping (QDM) | Section 2.4 |
| 7. Leave-One-Month-Out Validation | Out-of-sample scores, method selection, time-split check, member diagnostics | Table 3, Figures 5–6 |
| 8. Calibrated Projection | Selected method refitted on the whole calibration period and applied to all members | Table 4, Figure 7 |
| 9. Air Quality Categories | ISPU categories and exceedance of the 24-hour standard | Table 4 |
| 10. Monthly Maps of the Projection | Monthly calibrated ensemble means per province | Figure 8 |
| 11. Composition of MERRA-2 PM2.5 | Seasonal contribution of the aerosol components | Figure 9 |
| 12. Summary | Key numbers reported in the article | – |

## Method in brief

- **Seasons.** Dry season is June–October; wet season is November–May.
- **Calibration period.** 1 January 2025 – 31 August 2026, the days with both MERRA-2 and model data (608 days, 20 months). The projection period starts the day after the last MERRA-2 day and runs to 31 January 2027.
- **Calibration.** Each method is fitted separately for every province–season stratum (10 strata). The model distribution is estimated from the pooled members, and the same correction is applied to each member. QDM corrects one calendar month at a time, with quantiles bounded to 0.01–0.99.
- **Validation.** Leave-one-month-out: each of the 20 months is held out in turn, the methods are fitted on the remaining months and applied to the held-out month, and the scores are computed on the pooled daily out-of-sample series. Holding out whole months rather than single days keeps strongly autocorrelated neighbouring days out of the training data. A second check trains on 2025 and tests on January–August 2026.
- **Method selection.** Because a free-running projection is not synchronised with observed weather, the method is chosen by distributional agreement: the sum of its ranks on the mean Kolmogorov–Smirnov statistic across provinces and on the pooled rank-histogram χ². CRPS is reported as a consistency check. Quantile delta mapping was selected.
- **Air quality categories.** The 24-hour PM2.5 bands of the Indonesian ISPU (Ministry of Environment and Forestry Regulation P.14/2020). The 24-hour standard of 55 µg m⁻³ follows Government Regulation 22/2021.

## Data

All concentrations are in µg m⁻³ and refer to UTC days. Provincial values are area-weighted means of the grid cells in each province (cell fraction inside the province × cos latitude), with boundaries from GADM 4.1.

**`data/pm25_daily_province.csv`**: daily provincial PM2.5, 1 January 2025 – 31 January 2027.

| Column | Description |
|---|---|
| `date` | Date (UTC day) |
| `province` | West, Central, South, East or North Kalimantan |
| `r1i1p1f1`, `r2i1p1f1`, `r3i1p1f1` | MPI-ESM-1-2-HAM members, SSP3-7.0, after unit conversion and remapping to the MERRA-2 grid |
| `merra2` | MERRA-2 PM2.5; empty after 31 August 2026 |

**`data/merra2_pm25_components_daily_province.csv`**: daily MERRA-2 PM2.5 components, 1 January 2025 – 31 August 2026.

| Column | Description |
|---|---|
| `date`, `province` | As above |
| `dust` | DUSMASS25 |
| `sea_salt` | SSSMASS25 |
| `black_carbon` | BCSMASS |
| `organic_matter` | OCSMASS |
| `sulfate` | SO4SMASS × 132.14/96.06 (as ammonium sulfate) |
| `pm25_total` | Sum of the five components |

MERRA-2 does not include nitrate, and it includes ammonium only as part of ammonium sulfate.

## Outputs

| File in `outputs/tables/` | Content | Article |
|---|---|---|
| `descriptive_statistics.csv` | Statistics per province, season and source (rows with `season = All`) | Table 2 |
| `annual_means.csv`, `member_means.csv` | Annual means and means of each member | Section 3.1 |
| `synchrony_test.csv` | Raw and anomaly correlations between the ensemble mean and MERRA-2 | Section 3.2 |
| `raw_ensemble_evaluation.csv` | Scores of the raw ensemble per province and season | Section 3.2 |
| `vs_parameters.csv` | Variance-scaling parameters fitted on the whole calibration period | – |
| `validation_leave_one_month_out.csv` | Out-of-sample scores per province, season and method | Table 3 |
| `validation_summary.csv` | Scores over all provinces, used for method selection | Table 3 |
| `validation_time_split.csv` | Train 2025 / test 2026 check | Section 3.3 |
| `member_diagnostics.csv` | Relative bias, SD ratio and KS statistic of each member | Figure 6 |
| `calibrated_daily_pm25.csv` | Raw and calibrated daily values of each member, ensemble mean and range | Table 4, Figures 7–8 |
| `daily_category_probability.csv` | Daily probability of each ISPU category across members | – |
| `projection_category_days.csv` | Days per ISPU category per month in the projection period | Table 4 |
| `merra2_category_frequency.csv` | Frequency of ISPU categories in MERRA-2 | – |
| `merra2_components.csv` | Seasonal mean components and their shares on the 10 highest days | Figure 9 |

Abbreviations in the tables: `Raw` raw ensemble, `VS` variance scaling in log space, `QDM` quantile delta mapping, `ks_d` Kolmogorov–Smirnov statistic, `crps` fair CRPS, `bs_p90_*` Brier score and its decomposition for days above the provincial MERRA-2 90th percentile, `dP95` difference in the 95th percentile (model minus MERRA-2).

## Data sources

- MERRA-2 M2T1NXAER v5.12.4, Global Modeling and Assimilation Office (GMAO), NASA GES DISC, doi:[10.5067/KLICLTZ8EM9D](https://doi.org/10.5067/KLICLTZ8EM9D).
- MPI-ESM-1-2-HAM, SSP3-7.0, AERhr output (`sfpm25`, `ps`, `tas`), Earth System Grid Federation.
- GADM version 4.1, https://gadm.org.

The derived files in this repository are subject to the terms of use of the original data providers.
