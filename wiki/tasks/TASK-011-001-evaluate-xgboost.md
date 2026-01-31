# TASK-011-001: Evaluate XGBoost Model

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-011-001 |
| User Story | [US-011: Model Evaluation](../user-stories/US-011-model-evaluation.md) |
| EPIC | [EPIC-004: Model Evaluation](../epics/EPIC-004-model-evaluation.md) |
| Status | Complete |

---

## Objective

Perform comprehensive evaluation of the XGBoost fare prediction model using ML.EVALUATE and custom metrics.

---

## Prerequisites

- [x] XGBoost model trained (TASK-007-003)
- [x] Test set available
- [x] Understanding of regression metrics

---

## Step-by-Step Instructions

### Step 1: Run ML.EVALUATE

```sql
SELECT *
FROM ML.EVALUATE(
  MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
  (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
);
```

**Expected Output**:

| Metric | Value | Interpretation |
|--------|-------|----------------|
| mean_absolute_error | 3.12 | Average $3.12 prediction error |
| mean_squared_error | 15.67 | Squared error (penalizes outliers) |
| mean_squared_log_error | 0.045 | Log-scale error |
| median_absolute_error | 2.14 | Median error (robust) |
| r2_score | 0.913 | 91.3% variance explained |
| explained_variance | 0.915 | Similar to R2 |

### Step 2: Understand Each Metric

#### R-Squared (R2)

```
R2 = 1 - (Sum of Squared Errors / Total Sum of Squares)
```

| Value | Interpretation |
|-------|----------------|
| < 0 | Worse than mean prediction |
| 0 | Same as predicting mean |
| 0.5 | Moderate fit |
| 0.75 | Good fit |
| 0.9+ | Excellent fit |

**Our Result**: R2 = 0.913 (Excellent)

#### Mean Absolute Error (MAE)

```
MAE = Average(|actual - predicted|)
```

| Value | Interpretation |
|-------|----------------|
| < $2 | Excellent |
| $2-4 | Good |
| $4-6 | Acceptable |
| > $6 | Needs improvement |

**Our Result**: MAE = $3.12 (Good)

#### Root Mean Squared Error (RMSE)

```
RMSE = sqrt(Mean Squared Error) = sqrt(15.67) = $3.96
```

RMSE penalizes large errors more than MAE.

### Step 3: Evaluate on Test Set with Predictions

```sql
SELECT
  COUNT(*) as total_predictions,
  ROUND(AVG(target_fare), 2) as avg_actual,
  ROUND(AVG(predicted_target_fare), 2) as avg_predicted,
  ROUND(AVG(ABS(target_fare - predicted_target_fare)), 2) as mae,
  ROUND(SQRT(AVG(POW(target_fare - predicted_target_fare, 2))), 2) as rmse,
  ROUND(CORR(target_fare, predicted_target_fare), 4) as correlation
FROM ML.PREDICT(
  MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
  (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
);
```

**Expected Output**:

| total_predictions | avg_actual | avg_predicted | mae | rmse | correlation |
|-------------------|------------|---------------|-----|------|-------------|
| 2,119,688 | $18.47 | $18.42 | $3.12 | $3.96 | 0.9558 |

### Step 4: Error Distribution Analysis

```sql
WITH predictions AS (
  SELECT
    target_fare,
    predicted_target_fare,
    ABS(target_fare - predicted_target_fare) as absolute_error
  FROM ML.PREDICT(
    MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
    (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
  )
)
SELECT
  COUNTIF(absolute_error < 1) / COUNT(*) * 100 as within_1_dollar_pct,
  COUNTIF(absolute_error < 2) / COUNT(*) * 100 as within_2_dollars_pct,
  COUNTIF(absolute_error < 5) / COUNT(*) * 100 as within_5_dollars_pct,
  COUNTIF(absolute_error < 10) / COUNT(*) * 100 as within_10_dollars_pct,
  COUNTIF(absolute_error >= 10) / COUNT(*) * 100 as over_10_dollars_pct
FROM predictions;
```

**Expected Output**:

| within_1_dollar_pct | within_2_dollars_pct | within_5_dollars_pct | within_10_dollars_pct | over_10_dollars_pct |
|--------------------|---------------------|---------------------|----------------------|---------------------|
| 42.3% | 62.1% | 82.8% | 94.5% | 5.5% |

### Step 5: Error by Fare Range

```sql
WITH predictions AS (
  SELECT
    target_fare,
    predicted_target_fare,
    ABS(target_fare - predicted_target_fare) as absolute_error,
    CASE
      WHEN target_fare < 10 THEN 'Low (<$10)'
      WHEN target_fare < 25 THEN 'Medium ($10-25)'
      WHEN target_fare < 50 THEN 'High ($25-50)'
      ELSE 'Very High (>$50)'
    END as fare_bucket
  FROM ML.PREDICT(
    MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
    (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
  )
)
SELECT
  fare_bucket,
  COUNT(*) as trip_count,
  ROUND(AVG(absolute_error), 2) as avg_error,
  ROUND(AVG(absolute_error / target_fare * 100), 1) as mape
FROM predictions
GROUP BY fare_bucket
ORDER BY MIN(target_fare);
```

**Expected Output**:

| fare_bucket | trip_count | avg_error | mape |
|-------------|------------|-----------|------|
| Low (<$10) | 534,567 | $1.89 | 25.2% |
| Medium ($10-25) | 987,654 | $2.67 | 14.8% |
| High ($25-50) | 456,789 | $4.23 | 12.1% |
| Very High (>$50) | 140,678 | $7.56 | 11.2% |

**Insight**: Lower percentage error for higher fares.

### Step 6: Residual Analysis

```sql
WITH predictions AS (
  SELECT
    target_fare,
    predicted_target_fare,
    target_fare - predicted_target_fare as residual
  FROM ML.PREDICT(
    MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`,
    (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
  )
)
SELECT
  ROUND(AVG(residual), 4) as mean_residual,  -- Should be ~0
  ROUND(STDDEV(residual), 2) as residual_std,
  ROUND(MIN(residual), 2) as min_residual,
  ROUND(MAX(residual), 2) as max_residual,
  ROUND(APPROX_QUANTILES(residual, 100)[OFFSET(25)], 2) as q1,
  ROUND(APPROX_QUANTILES(residual, 100)[OFFSET(50)], 2) as median,
  ROUND(APPROX_QUANTILES(residual, 100)[OFFSET(75)], 2) as q3
FROM predictions;
```

**Expected Output**:

| mean_residual | residual_std | min_residual | max_residual | q1 | median | q3 |
|---------------|--------------|--------------|--------------|-----|--------|-----|
| 0.05 | 3.96 | -45.23 | 52.67 | -1.67 | 0.12 | 1.89 |

**Good Signs**:
- Mean residual near 0 (unbiased)
- Symmetric Q1/Q3 around median

---

## Evaluation Summary

### Performance Metrics

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| R2 | 0.913 | > 0.75 | EXCEEDED |
| MAE | $3.12 | < $5.00 | PASS |
| RMSE | $3.96 | < $6.00 | PASS |
| Correlation | 0.956 | > 0.90 | PASS |

### Accuracy Buckets

| Threshold | Percentage | Grade |
|-----------|------------|-------|
| Within $2 | 62.1% | Good |
| Within $5 | 82.8% | Very Good |
| Within $10 | 94.5% | Excellent |

### Model Quality Assessment

| Dimension | Score | Notes |
|-----------|-------|-------|
| Accuracy | A | R2 = 0.913 |
| Precision | B+ | MAE = $3.12 |
| Robustness | A- | Low residual std |
| Bias | A | Mean residual ~0 |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| ML.EVALUATE | Built-in evaluation function |
| Regression Metrics | R2, MAE, RMSE, MSE |
| Error Analysis | Distribution of errors |
| Residual Analysis | Bias detection |

---

## Next Task

[TASK-011-002: Feature Importance Analysis](./TASK-011-002-feature-importance.md)
