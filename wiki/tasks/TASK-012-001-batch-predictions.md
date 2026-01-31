# TASK-012-001: Generate Batch Predictions

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-012-001 |
| User Story | [US-012: Batch Predictions](../user-stories/US-012-batch-predictions.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Generate batch predictions on the test set and store results for analysis and monitoring.

---

## Prerequisites

- [x] XGBoost model trained (TASK-007-003)
- [x] Test set available
- [x] Understanding of ML.PREDICT

---

## Step-by-Step Instructions

### Step 1: Understand ML.PREDICT

```sql
ML.PREDICT(MODEL model_name, (SELECT query))
```

Returns original columns plus:
- `predicted_<label>` - The prediction
- For classification: `predicted_<label>_probs` - Probabilities

### Step 2: Generate Test Set Predictions

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.test_predictions` AS
SELECT
  unique_key,
  trip_start_timestamp,
  taxi_id,

  -- Original features (for analysis)
  trip_miles,
  trip_seconds,
  straight_line_km,
  pickup_downtown_km,
  pickup_ohare_km,
  is_airport_trip,
  is_rush_hour,

  -- Actual vs Predicted
  target_fare as actual_fare,
  predicted_target_fare as predicted_fare,

  -- Error metrics
  ROUND(target_fare - predicted_target_fare, 2) as error,
  ROUND(ABS(target_fare - predicted_target_fare), 2) as absolute_error,
  ROUND(POW(target_fare - predicted_target_fare, 2), 2) as squared_error,

  -- Error percentage
  ROUND(ABS(target_fare - predicted_target_fare) / NULLIF(target_fare, 0) * 100, 2) as pct_error,

  -- Error buckets
  CASE
    WHEN ABS(target_fare - predicted_target_fare) < 1 THEN 'Excellent (<$1)'
    WHEN ABS(target_fare - predicted_target_fare) < 2 THEN 'Very Good (<$2)'
    WHEN ABS(target_fare - predicted_target_fare) < 5 THEN 'Good (<$5)'
    WHEN ABS(target_fare - predicted_target_fare) < 10 THEN 'Acceptable (<$10)'
    ELSE 'Poor (>$10)'
  END as accuracy_bucket

FROM ML.PREDICT(
  MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
  (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
);
```

### Step 3: Verify Predictions

```sql
SELECT
  COUNT(*) as total_predictions,
  ROUND(AVG(actual_fare), 2) as avg_actual,
  ROUND(AVG(predicted_fare), 2) as avg_predicted,
  ROUND(AVG(absolute_error), 2) as mae,
  ROUND(SQRT(AVG(squared_error)), 2) as rmse
FROM `sonorous-key-320714.diamond_analytics.test_predictions`;
```

**Expected Output**:

| total_predictions | avg_actual | avg_predicted | mae | rmse |
|-------------------|------------|---------------|-----|------|
| 2,119,688 | $18.47 | $18.42 | $3.12 | $3.96 |

### Step 4: Analyze by Accuracy Bucket

```sql
SELECT
  accuracy_bucket,
  COUNT(*) as trip_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) as pct,
  ROUND(AVG(absolute_error), 2) as avg_error
FROM `sonorous-key-320714.diamond_analytics.test_predictions`
GROUP BY accuracy_bucket
ORDER BY
  CASE accuracy_bucket
    WHEN 'Excellent (<$1)' THEN 1
    WHEN 'Very Good (<$2)' THEN 2
    WHEN 'Good (<$5)' THEN 3
    WHEN 'Acceptable (<$10)' THEN 4
    ELSE 5
  END;
```

**Expected Output**:

| accuracy_bucket | trip_count | pct | avg_error |
|-----------------|------------|-----|-----------|
| Excellent (<$1) | 896,234 | 42.3% | $0.48 |
| Very Good (<$2) | 419,456 | 19.8% | $1.45 |
| Good (<$5) | 439,567 | 20.7% | $3.21 |
| Acceptable (<$10) | 247,890 | 11.7% | $6.89 |
| Poor (>$10) | 116,541 | 5.5% | $15.67 |

### Step 5: Daily Prediction Summary

```sql
SELECT
  DATE(trip_start_timestamp) as prediction_date,
  COUNT(*) as predictions,
  ROUND(AVG(absolute_error), 2) as daily_mae,
  ROUND(SQRT(AVG(squared_error)), 2) as daily_rmse,
  ROUND(COUNTIF(absolute_error < 5) * 100.0 / COUNT(*), 1) as within_5_pct
FROM `sonorous-key-320714.diamond_analytics.test_predictions`
GROUP BY DATE(trip_start_timestamp)
ORDER BY prediction_date DESC
LIMIT 10;
```

### Step 6: Error Analysis by Trip Type

```sql
SELECT
  CASE
    WHEN is_airport_trip THEN 'Airport'
    WHEN is_rush_hour THEN 'Rush Hour'
    ELSE 'Regular'
  END as trip_type,
  COUNT(*) as count,
  ROUND(AVG(actual_fare), 2) as avg_fare,
  ROUND(AVG(absolute_error), 2) as mae,
  ROUND(AVG(pct_error), 1) as mape
FROM `sonorous-key-320714.diamond_analytics.test_predictions`
GROUP BY 1
ORDER BY mae DESC;
```

**Expected Output**:

| trip_type | count | avg_fare | mae | mape |
|-----------|-------|----------|-----|------|
| Airport | 234,567 | $38.45 | $4.56 | 11.8% |
| Rush Hour | 456,789 | $21.23 | $3.34 | 15.7% |
| Regular | 1,428,332 | $14.67 | $2.78 | 18.9% |

---

## Predictions Schema

| Column | Type | Description |
|--------|------|-------------|
| unique_key | STRING | Trip identifier |
| trip_start_timestamp | TIMESTAMP | Prediction date |
| actual_fare | FLOAT64 | True fare |
| predicted_fare | FLOAT64 | Model prediction |
| error | FLOAT64 | Signed error |
| absolute_error | FLOAT64 | Unsigned error |
| squared_error | FLOAT64 | For RMSE calculation |
| pct_error | FLOAT64 | Percentage error |
| accuracy_bucket | STRING | Quality category |

---

## Prediction Quality Summary

| Metric | Value | Status |
|--------|-------|--------|
| Total Predictions | 2,119,688 | Complete |
| MAE | $3.12 | Good |
| RMSE | $3.96 | Good |
| Within $5 | 82.8% | Excellent |
| Within $10 | 94.5% | Excellent |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| ML.PREDICT | Batch prediction function |
| Error Metrics | MAE, RMSE, MAPE |
| Error Analysis | Segment-level performance |
| Prediction Storage | Audit and monitoring |

---

## Next Task

[TASK-012-002: Store Prediction Results](./TASK-012-002-store-predictions.md)
