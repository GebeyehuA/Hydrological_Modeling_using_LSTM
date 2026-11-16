# Hydrological_Modeling_using_LSTM
Long Short-Term Memory (LSTM) deep learning framework for daily rainfall–runoff modeling and river streamflow prediction across catchments in the Abay (Upper Blue Nile) Basin, Ethiopia.

# Deep Learning for Hydrological Modeling: LSTM-Based Multi-Step Streamflow Prediction in the Abay (Upper Blue Nile) Basin, Ethiopia

## Overview
This repository provides a reproducible, scientifically rigorous deep learning workflow for **daily multi-step streamflow prediction (1- to 7-day lead times)** across catchments in the **Abay (Upper Blue Nile) Basin, Ethiopia**, implemented in **`Hydrological_Modeling_using_LSTM.ipynb`**.

Built for hydrological modeling in complex Ethiopian highland watersheds, the notebook integrates daily gridded meteorological forcing from **ERA5-Land** (and optionally **CHIRPS** precipitation) extracted over **HydroSHEDS/HydroBASINS** catchment polygons via Google Earth Engine (GEE), paired with observed river discharge records. Beyond training a **Long Short-Term Memory (LSTM)** network, the workflow enforces strict hydrological data hygiene—preventing temporal and scaler data leakage, prohibiting target interpolation, benchmarking against naive and machine learning baselines, and evaluating high-flow vs. low-flow regimes separately.

---

## Data Sources & Citations
* **Meteorological Forcing (Primary):** [ERA5-Land Daily Aggregated](https://developers.google.com/earth-engine/datasets/catalog/ECMWF_ERA5_LAND_DAILY_AGGR) (`ECMWF/ERA5_LAND/DAILY_AGGR` in GEE; ~11,132 m spatial resolution).
  * *Variables:* Precipitation, temperature, evapotranspiration, soil moisture, and model-derived runoff (evaluated both with and without runoff predictors).
  * *License & Citation:* Copernicus Climate Change Service (C3S) information; cite ECMWF ERA5-Land per Copernicus license requirements.
* **Precipitation Comparison (Optional):** [CHIRPS Daily](https://developers.google.com/earth-engine/datasets/catalog/UCSB-CHG/CHIRPS/DAILY) (`UCSB-CHG/CHIRPS/DAILY` in GEE; `precipitation` band in mm/day) used as an alternative precipitation predictor after GEE extraction and date merging.
* **Catchment Boundaries:** [WWF HydroSHEDS / HydroBASINS](https://developers.google.com/earth-engine/datasets/catalog/WWF_HydroSHEDS_v1_Basins_hybas_6) (`WWF/HydroSHEDS/v1/Basins/hybas_6` and other Pfafstetter levels in GEE). Selected polygons are explicitly validated against gauge metadata rather than assuming a level automatically matches a catchment.
* **Observed Streamflow Targets:** Daily river discharge observations ($m^3/s$) from hydrometric station records in the Abay Basin.

---

## Notebook Workflow (`Hydrological_Modeling_using_LSTM.ipynb`)

The end-to-end 46-cell notebook is structured into six methodological stages:

1. **Catchment Delineation & GEE Extraction:**
   * Verify gauge identity, coordinates, datum, units, and quality flags.
   * Validate that the selected HydroBASINS polygon represents the upstream contributing catchment and outlet location.
   * Extract daily basin-averaged ERA5-Land (and optional CHIRPS) forcing time series from Google Earth Engine and check export dates, missing pixels, and physical units.
2. **Discharge Quality Control & Audit:**
   * Identify and document duplicate dates and missing discharge gaps across months and seasons.
   * **Zero Target Fabrication:** Missing observed discharge values are **never** interpolated or synthetically filled to construct training targets.
3. **Chronological Splitting & Leakage-Free Preprocessing:**
   * Partition time series into strictly chronological **Training**, **Validation**, and **Test** periods (reported explicitly).
   * Fit feature and target scalers **exclusively on the training split** before transforming validation and test sequences.
   * Construct lookback input windows ($L$ days) mapped to 7-day direct forecast horizons ($t+1$ through $t+7$) using only complete, valid target sequences.
4. **LSTM Modeling & Baseline Benchmarking:**
   * Train a multi-step **LSTM** model alongside benchmark baselines: **Persistence**, **Climatology**, and tree-based models (**Random Forest** / **Gradient Boosting**).
5. **Lead-Time, Flow-Regime & Sensitivity Evaluation:**
   * Report forecast skill individually across every lead time ($t+1$ to $t+7$) using **NSE**, **KGE**, **RMSE**, **MAE**, and **PBIAS**.
   * Evaluate **high-flow (flood)** and **low-flow (baseflow)** performance separately.
   * Run sensitivity experiments across lookback window length, feature sets (**with vs. without ERA5-Land runoff**), basin averaging, and missing-data handling.
6. **Scientific Quality-Control Verification (Cells 45–46):**
   * Final audit against the 13-point hydrological quality-control checklist and interpretation guidelines before reporting results.

---

## Scientific Quality-Control Checklist
Before interpreting or publishing outputs from `Hydrological_Modeling_using_LSTM.ipynb`, verify every item below:

- [ ] **Gauge Metadata:** Gauge identity, coordinates, datum, units, and quality flags verified with the data provider.
- [ ] **Catchment Topology:** Selected polygon is the upstream contributing catchment for that gauge.
- [ ] **Area & Outlet Check:** Catchment area and outlet location checked against independent basin/station information.
- [ ] **GEE Export Integrity:** GEE export dates, missing pixels, and units checked.
- [ ] **Discharge Audit:** Observed discharge gaps and duplicate dates documented.
- [ ] **No Target Fabrication:** No discharge values were fabricated or interpolated to create training targets.
- [ ] **Chronological Splits:** Training/validation/test periods are chronological and reported explicitly.
- [ ] **Leakage-Free Scaling:** Feature and target scalers were fitted on training data only.
- [ ] **Lead-Time Granularity:** Forecast skill is reported for each lead time ($t+1$ to $t+7$).
- [ ] **Baseline Comparisons:** Baseline models (e.g., persistence, climatology, Random Forest/gradient boosting) are compared.
- [ ] **Flow Regime Separation:** High-flow and low-flow performance are assessed separately.
- [ ] **Forecast Framing:** Results are not described as operational forecasts unless future meteorological forecasts are supplied.
- [ ] **Sensitivity Analysis:** Sensitivity to lookback length, feature set, basin averaging, and missing-data treatment is assessed.

---

## Important Interpretation Notes
* **Partial Discharge Records:** LSTM performance may be biased if missingness is concentrated in flood (*Kiremt*) seasons or particular years. Always report coverage by month/season alongside the number of valid target sequences.
* **Spatial Representativeness:** ERA5-Land has a grid spacing of about **11.1 km** (`11,132 m`). Small or mountainous catchments in the Abay basin may be poorly represented by a simple basin mean; compare against CHIRPS or elevation-weighted extraction where needed.
* **Runoff Predictor Dependency:** ERA5-Land `runoff` is a model-derived variable. Always test models **both with and without it** to understand whether apparent skill gains depend on a reanalysis hydrological product.
* **Forecast Definition:** A 7-day direct forecast based only on past forcing does not have access to future rainfall. For real operational forecasting, add forecast precipitation and temperature from a Numerical Weather Prediction (NWP) system.
* **Multi-Basin Generalization:** When extending across multiple Abay sub-basins, use a separate **spatial holdout (leave-one-catchment-out)** experiment—do not randomly mix samples from the same basin across training and test splits.

---

## Repository Structure
```text
Abay_Ethiopian_Watershed_LSTM/
├── Hydrological_Modeling_using_LSTM.ipynb   # Main end-to-end 46-cell workflow & QC notebook
├── data/
│   ├── raw/                                 # GEE ERA5-Land / CHIRPS CSV exports & station discharge
│   ├── processed/                           # QC-audited, date-aligned time series (no target interpolation)
│   └── shapefiles/                          # Validated HydroBASINS / gauge watershed polygons
├── outputs/
│   ├── figures/                             # Seasonal coverage plots, hydrographs, lead-time skill curves
│   └── tables/                              # Lead-time (t+1..t+7) & high/low-flow benchmark metrics
├── environment.yml                          # Conda / Python dependencies
└── README.md
