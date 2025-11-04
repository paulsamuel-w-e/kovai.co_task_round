# Kovai.co Task Round

1. Strong yearly seasonality across most series

- Evidence: seasonal_decompose with period=365 and visual plots show a clear seasonal pattern; Darts NaiveSeasonal(K=365) worked sensibly.

- treated data as seasonal and used seasonal baseline models (NaiveSeasonal).

- use seasonal-aware models

2. Most series follow a very similar trend — except school and a spike in other

- Evidence: the log-transformed series “overlap very well” except school (more variation) and other shows a sudden spike.

- visual inspection, log1p transform to improve comparability.

- investigate other spike dates  and treat school separately (different seasonal amplitude or additional exogenous factors).

3. Log transform (log1p) improved visibility and made distributions more regular

- Evidence: scale_data = copy_data.transform(lambda x: np.log1p(x)) and the histograms/KDEs showed clearer, multimodal but less skewed shapes.

- transformed before decomposition and modeling.

- remember to inverse-transform predictions (expm1) when reporting back in original units; evaluate errors on original scale too.

4. Missing-data handling mattered — you dropped NA for modeling, then later interpolated & back-filled for production forecasts

- Evidence: scale_clean.dropna(inplace=True) before train/test, then later scale_data_filled['other'].interpolate(); bfill() and re-trained on filled series.

- conservative drop for initial experiments, then interpolation to produce 7-day forecasts for all columns.

- adopt a principled missing-value strategy (time-aware interpolation, seasonal forward-fill, or model-based imputation) and compare downstream forecast accuracy.

5. Naive seasonal baselines gave reasonable but variable error by series

- Evidence (MAE per series from Darts NaiveSeasonal):

- local_route: 0.5961

- light_rail: 0.3020 (best)

- rapid_route: 0.3874

- other: 0.5799

- peak_service: 1.8058

- school: 2.3285 (worst)

- trained NaiveSeasonal per series, logged MAE, and produced 7-day forecasts saved to 7_day_forecasts.csv.

6. Multiple modeling paths attempted — Prophet, sktime Naive, and Darts — but Prophet wasn’t evaluated yet

- Evidence: Prophet model was instantiated and fit (model = Prophet(yearly_seasonality=20)), but no test evaluation printed. sktime NaiveForecaster was trained and MAPE computed (value not shown), while Darts workflow reached full evaluation and forecasts.

- explored different toolkits (Prophet, sktime, Darts) — good for robustness/experiment tracking.



---
# 7 Days Forecast

| date       | local_route | light_rail | peak_service | rapid_route | school   | other     |
|------------|------------:|-----------:|-------------:|------------:|---------:|----------:|
| 2024-09-30 |   7.903227  |  8.635509  |    0.000000  |  8.909235  | 0.000000 |  3.091042 |
| 2024-10-01 |   7.828835  |  8.454892  |    0.000000  |  8.771990  | 0.000000 |  3.044522 |
| 2024-10-02 |   9.411320  |  9.230633  |    5.730100  |  9.797071  | 0.000000 |  4.158883 |
| 2024-10-03 |   9.333177  |  9.119759  |    5.808142  |  9.681905  | 0.000000 |  4.174387 |
| 2024-10-04 |   9.452031  |  9.235813  |    5.746203  |  9.786392  | 0.000000 |  4.007333 |
| 2024-10-05 |   9.418898  |  9.263123  |    5.564520  |  9.762327  | 0.000000 |  4.127134 |
| 2024-10-06 |   8.313362  |  8.829226  |    0.000000  |  9.074062  | 0.000000 |  2.708050 |

---
