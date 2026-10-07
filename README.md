# weather-prediction-RDU

Hourly temperature forecasts for Raleigh-Durham International Airport (RDU), built for AIPI 520 Project 1.

**Team:** Jothi Gupta, Konur Nordberg, Daniel Yaari

## Objective

Predict the hourly temperature measured at RDU for the 336 hours from **12am September 17 to 11pm September 30, 2026**, using only data available before September 17.

Requires one linear regression model and at least one other model.

## Approach

Every model in this repo starts from the same idea: NOAA's Global Forecast System (GFS) already forecasts temperature, so the models learn to correct GFS by comparing past forecasts with what RDU actually recorded.

The team worked in parallel and built three independent pipelines.


### What we learned

- **GFS is too extreme.** The best linear regression in Pipeline A used only the GFS temperature, scaled down and shifted toward the mean. Pipeline B reached the same conclusion: the weight it puts on GFS falls from 0.92 at one day ahead to 0.10 at fifteen days.
- **GFS loses to climatology after about a week.** In the Pipeline B backtest, raw GFS error passes the error of simply predicting the normal temperature at around day 7.
- **More features did not help the linear models.** In Pipeline A, regressions with 9 to 15 features validated worse than the one-feature model.
- **The target window included a forecast bust.** From September 21 to 26, GFS missed a pattern change and every model's error rose to 9–13°F. Most of each model's overall error comes from those days.

## Repository structure

```
data/
  raw/                  Downloaded forecasts and observations
    ghcnh_cache/        NOAA GHCNh station files, 2016–2026
  processed/            Modeling tables, predictions, metrics, model configs
notebooks/
  data_sourcing/        Download GFS forecasts and RDU observations
  data_preprocessing/   EDA and construction of the master modeling dataset
  models/
    linear_regression/  Linear regression notebooks
    xgboost/            XGBoost notebook
    RF/                 Pipeline B: per-lead-day regression and random forest
  final_summary/        Final evaluation on the 336-hour target window
reports/figures/        Evaluation charts
```

### Key files

| File | What it is |
|---|---|
| `data/processed/modeling_historical.csv` | Pipeline A training table: 13,237 labeled forecast hours, 2021–2025 |
| `data/processed/modeling_future.csv` | Pipeline A features for the 336 target hours |
| `data/processed/modeling_feature_manifest.json` | Feature list, split definition and leakage notes |
| `data/processed/xgboost_selected_config.json` | Chosen XGBoost features and hyperparameters |
| `data/processed/final_2026_evaluation_hourly_FULL336.csv` | Hour-by-hour predictions and actuals for the target window |
| `reports/figures/final_2026_full336/` | Eight evaluation charts for the target window |

## Running the notebooks

### Requirements

Python 3 with `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `requests` and `xgboost`. The Herbie-based download notebooks also need `herbie-data` and `xarray`.

```
pip install pandas numpy scikit-learn matplotlib requests xgboost herbie-data xarray
```

## Guarding against data leakage

The team's rule from day one: **don't use any future data.**

- **Hard cutoff.** No observation from September 17, 2026 onward is used to train or tune a model.
- **Forecast-time features only.** Pipeline A's recent-observation features use only observations strictly before each simulated forecast cutoff. Pipeline B's climatology is built from 2018–2025 only.
- **Time-based splits.** Pipeline A holds out whole years. Pipeline B trains on earlier runs and validates on later ones, with a 15-day gap so no training forecast overlaps a validation period.
- **Test data used last.** Model selection was done on validation data. The target-window actuals were downloaded only to score frozen predictions.

## Notes and limitations

- Pipeline A interpolates GFS linearly to hourly values after forecast hour 120, where GFS output is 3-hourly.
- Pipeline B trains on April to September 2026 only, so it has never seen a late September. The climatological baseline is what carries the seasonal signal.
- The random forest's leaf size was tuned on the same backtest folds it is scored on, so its backtest RMSE is slightly optimistic.
- One Open-Meteo run (June 10, 2026) failed to download and is missing from Pipeline B's training data.
- `notebooks/models/linear_regression/linear_regression.ipynb` is an early exploratory notebook, kept for reference. It predates the leakage fixes in Pipeline B.
- Ridge regression work is on the `new-jothi-branch` branch and has not been merged into `main`.

## Project plan

The team worked individually between meetings and compared progress at each checkpoint.

| Phase | When | Format | Focus |
|---|---|---|---|
| 1 | Friday | Meet after class, 11:30 | Data collection, everything saved as CSV |
| 2 | Sunday | Async | Processing data |
| 3 | Friday | Meet after class, 3:30 | Linear regression modeling; candidates for the second model (logistic regression, time series, KNN classifier, Prophet) |
| 4 | Sunday | To be decided after Phase 3 | Iteration and improvement |
| 5 | Tuesday | Meet after class, 2:30 | Deliverables |

**Timeline**

1. One to two days individually finding datasets
2. Meet
3. One to two days individually working on linear regression

## Data sources

- [Meteostat, station 72306](https://meteostat.net/en/station/72306?t=2026-09-01/2026-09-16): RDU observations for September 1–16, 2026, used in early exploration
- [NOAA Global Forecast System](https://www.ncei.noaa.gov/products/weather-climate-models/global-forecast): forecast runs
- [dynamical.org](https://dynamical.org/): GFS point forecasts
- [Open-Meteo](https://open-meteo.com/): archived single GFS runs
- [NOAA GHCNh](https://www.ncei.noaa.gov/products/global-historical-climatology-network-hourly): hourly observations, station USW00013722
- [Iowa Environmental Mesonet](https://mesonet.agron.iastate.edu/request/download.phtml?network=NC_ASOS): ASOS observations, station RDU
- [Aviation Weather Center](https://aviationweather.gov/): KRDU METARs used to fill gaps in the final evaluation6
