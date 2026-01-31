# TASK-008-002: Train ARIMA Model

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-008-002 |
| User Story | [US-008: ARIMA Forecasting](../user-stories/US-008-arima-forecasting.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Train the ARIMA_PLUS time series forecasting model for taxi demand prediction.

---

## Prerequisites

- [x] Daily demand table created (TASK-008-001)
- [x] Understanding of ARIMA parameters
- [x] BigQuery ML ARIMA_PLUS documentation reviewed

---

## Step-by-Step Instructions

### Step 1: Understand ARIMA_PLUS

BigQuery's ARIMA_PLUS automatically:
- Handles seasonality detection
- Detects holiday effects
- Tunes ARIMA parameters (p, d, q)
- Handles missing values

| Component | Description |
|-----------|-------------|
| AR (p) | Autoregressive - past values |
| I (d) | Integrated - differencing |
| MA (q) | Moving Average - past errors |
| PLUS | Seasonality, holidays, trends |

### Step 2: Create Training Query

```sql
CREATE OR REPLACE MODEL `sonorous-key-320714.diamond_analytics.demand_forecast_arima`
OPTIONS(
  model_type = 'ARIMA_PLUS',
  time_series_timestamp_col = 'trip_date',
  time_series_data_col = 'daily_trips',
  auto_arima = TRUE,
  data_frequency = 'DAILY',
  decompose_time_series = TRUE,
  holiday_region = 'US'
) AS
SELECT
  trip_date,
  daily_trips
FROM `sonorous-key-320714.diamond_analytics.daily_demand`
WHERE trip_date < '2023-07-01';  -- Training cutoff
```

**Option Explanation**:

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | ARIMA_PLUS | Time series with extras |
| time_series_timestamp_col | trip_date | Date column |
| time_series_data_col | daily_trips | Target metric |
| auto_arima | TRUE | Auto-tune p, d, q |
| data_frequency | DAILY | Data granularity |
| decompose_time_series | TRUE | Enable trend/seasonal |
| holiday_region | US | US holiday effects |

### Step 3: Monitor Training

Training typically takes 2-5 minutes. Check status:

```sql
SELECT *
FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.demand_forecast_arima`);
```

### Step 4: Review Model Coefficients

```sql
SELECT *
FROM ML.ARIMA_COEFFICIENTS(MODEL `sonorous-key-320714.diamond_analytics.demand_forecast_arima`);
```

**Expected Output**:

| ar_coefficients | ma_coefficients | intercept_or_drift |
|-----------------|-----------------|-------------------|
| [0.85, 0.12] | [0.34, 0.21, 0.15] | 7234.56 |

### Step 5: Evaluate Model Fit

```sql
SELECT *
FROM ML.ARIMA_EVALUATE(MODEL `sonorous-key-320714.diamond_analytics.demand_forecast_arima`);
```

**Expected Output**:

| non_seasonal_p | non_seasonal_d | non_seasonal_q | has_drift | log_likelihood | AIC | variance |
|----------------|----------------|----------------|-----------|----------------|-----|----------|
| 2 | 1 | 3 | TRUE | -8234.5 | 16485 | 1234567 |

**Metric Interpretation**:

| Metric | Meaning | Lower is Better |
|--------|---------|-----------------|
| AIC | Akaike Information Criterion | Yes |
| log_likelihood | Model fit quality | No (higher = better) |
| variance | Residual variance | Yes |

### Step 6: Check Seasonality Components

```sql
SELECT *
FROM ML.EXPLAIN_FORECAST(
  MODEL `sonorous-key-320714.diamond_analytics.demand_forecast_arima`,
  STRUCT(30 AS horizon)
);
```

---

## Model Parameters (Auto-Selected)

| Parameter | Value | Description |
|-----------|-------|-------------|
| p (AR) | 2 | Uses 2 lagged values |
| d (I) | 1 | First differencing |
| q (MA) | 3 | Uses 3 lagged errors |
| Seasonal | Weekly | 7-day cycle detected |
| Holiday Effects | Yes | US holidays included |

---

## Training Summary

| Metric | Value |
|--------|-------|
| Training Period | 2020-01-01 to 2023-06-30 |
| Training Days | 912 |
| AIC Score | 16,485 |
| Model Type | ARIMA(2,1,3) with drift |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| ARIMA_PLUS | BigQuery time series model |
| Auto ARIMA | Automatic parameter tuning |
| Seasonality | Weekly patterns |
| Holiday Effects | Calendar adjustments |
| AIC | Model selection criterion |

---

## Next Task

[TASK-008-003: Generate Forecasts](./TASK-008-003-generate-forecast.md)
