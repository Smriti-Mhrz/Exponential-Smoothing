# Exponential-Smoothing

# Overview
This assignment analyzes monthly retail sales of Australian Sparkling Wine (1980–1994) and applies the Exponential Smoothing (ETS) framework to compare multiple forecasting models, including Holt–Winters Additive, Multiplicative, and Damped variants, against a Seasonal Naïve benchmark.

# Dataset
PropertyDetailsSourceAustralianWines.csvWine SelectedSparklingMeasureMonthly retail sales (thousands of litres)PeriodJanuary 1980 – December 1994FrequencyMonthlyTotal Records180 observations
Summary Statistics:
Overall MeanDecember MeanJune MeanMaxMin2,4315,8131,4597,2421,170

Sparkling wine was selected for its dominant December spike — averaging 3.6× the off-peak monthly mean — making it an ideal test case for extreme seasonal patterns.


# Key Observations from EDA

Level: Typical monthly sales range ~1,200–2,700 thousand litres outside the holiday peak.
Trend: Mild upward drift from 1980 to 1993 (not aggressive growth).
Seasonality: Very strong 12-month cycle with a sharp December peak and post-holiday drop in January–February.
Noise: Moderate and mostly pattern-free remainder component.
Seasonality Type: The December-to-annual-mean ratio increases over time → supports multiplicative over additive seasonality.
ACF: Large positive spikes at lags 12, 24, and 36 confirm a classic 12-month seasonal cycle.


# Methodology
Data Split
SetPeriodObservationsTrainingJanuary 1980 – December 1993168ValidationJanuary 1994 – December 199412
Models Fitted
ModelETS NotationDescriptionSeasonal NaïveSNAIVEBaseline benchmark — repeats last year's valuesHolt–Winters AdditiveETS(A, A, A)Assumes constant seasonal amplitudeHolt–Winters MultiplicativeETS(A, A, M)Assumes seasonal effect scales with levelHolt–Winters DampedETS(A, Ad, M)Multiplicative seasonality with dampened trend

# Results
Validation Accuracy (1994 Holdout), Sorted by MASE
ModelMERMSEMAEMAPE (%)MASEhw_mult-81.07432.52337.7416.951.120 ✅hw_add-127.66421.19339.6416.901.126hw_damped-90.13452.14357.5417.971.186snaive-117.25583.92425.0820.751.410
Best Model: Holt–Winters Multiplicative (hw_mult)

MASE: 1.120 — lowest among all candidates
MAE improvement vs SNAIVE: ~20.5%
Bias (ME): -81.07 → slight under-forecasting, within acceptable tolerance


Note: All models have MASE > 1 because the MASE denominator uses an in-sample Naïve scaler in fable, not the SNAIVE itself.


Residual Diagnostics (Best Model: hw_mult)
DiagnosticResultResiduals over timeFluctuate around zero; no obvious clusters or long runsResidual histogramApproximately centered at zero; roughly bell-shapedACF of residualsOne significant spike at lag 12 (acf ≈ 0.185)Ljung–Box test (lag = 24)Q = 21.25, p = 0.6241 → consistent with white noise ✅

# Deployment Checklist
GateMetricPassGate 1 — Beats SNAIVE on MAEMAE(best) = 337.7 vs MAE(snaive) = 425.1✅Gate 2 — Ljung–Box p > 0.05p = 0.6241✅Gate 3 — |ME| < 10% of training mean|ME| = 81.1 vs tolerance = 242.9✅
✅ Final Decision: DEPLOY
The hw_mult model passes all three deployment gates — it beats the benchmark, has white-noise residuals, and its bias is well within the operational tolerance band.

# Suggested Next Steps

ARIMA vs ETS bake-off — Fit ARIMA(Sales) and compare holdout accuracy against ETS(A,A,M).
Outlier adjustment — December spikes and occasional shocks may distort parameter estimates; treat outliers or use robust ETS variants.
External regressors — Incorporate promotions, price index, holiday timing, or macroeconomic indicators for further accuracy gains.


# Tools & Libraries

Language: R
Key Packages: fpp3, knitr, kableExtra, scales
Functions used: ETS(), SNAIVE(), forecast(), accuracy(), gg_tsresiduals(), STL(), ACF(), ljung_box()


# References

Hyndman, R.J. & Athanasopoulos, G. (2021). Forecasting: Principles and Practice, 3rd ed. https://otexts.com/fpp3/
fpp3 CRAN package: https://cran.r-project.org/web/packages/fpp3/index.html
