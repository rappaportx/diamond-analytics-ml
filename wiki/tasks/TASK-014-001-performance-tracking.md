# TASK-014-001: Create Performance Tracking Table

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-014-001 |
| User Story | [US-014: Performance Tracking & Alerting](../user-stories/US-014-alerting-system.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Create a performance tracking table to monitor daily model metrics for ongoing model health assessment.

---

## Prerequisites

- [x] Test predictions generated (TASK-012-001)
- [x] Understanding of performance metrics
- [x] Model evaluation complete

---

## Step-by-Step Instructions

### Step 1: Design Performance Schema

| Metric | Type | Purpose |
|--------|------|---------|
| prediction_date | DATE | Tracking date |
| prediction_count | INTEGER | Volume |
| daily_mae | FLOAT64 | Mean Absolute Error |
| daily_mse | FLOAT64 | Mean Squared Error |
| daily_rmse | FLOAT64 | Root Mean Squared Error |
| daily_correlation | FLOAT64 | Actual vs Predicted |
| within_2_dollars_pct | FLOAT64 | Excellent accuracy |
| within_5_dollars_pct | FLOAT64 | Good accuracy |
| within_10_dollars_pct | FLOAT64 | Acceptable accuracy |

### Step 2: Create Performance Tracking Table

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.performance_tracking` AS
SELECT
  DATE(trip_start_timestamp) as prediction_date,
  COUNT(*) as prediction_count,

  -- Error metrics
  ROUND(AVG(absolute_error), 4) as daily_mae,
  ROUND(AVG(squared_error), 4) as daily_mse,
  ROUND(SQRT(AVG(squared_error)), 4) as daily_rmse,

  -- Correlation
  ROUND(CORR(actual_fare, predicted_fare), 4) as daily_correlation,

  -- Accuracy buckets
  ROUND(COUNTIF(absolute_error < 2) * 100.0 / COUNT(*), 2) as within_2_dollars_pct,
  ROUND(COUNTIF(absolute_error < 5) * 100.0 / COUNT(*), 2) as within_5_dollars_pct,
  ROUND(COUNTIF(absolute_error < 10) * 100.0 / COUNT(*), 2) as within_10_dollars_pct,

  -- Average values
  ROUND(AVG(actual_fare), 2) as avg_actual_fare,
  ROUND(AVG(predicted_fare), 2) as avg_predicted_fare,

  -- Bias detection
  ROUND(AVG(error), 4) as mean_bias

FROM `sonorous-key-320714.diamond_analytics.test_predictions`
GROUP BY DATE(trip_start_timestamp)
ORDER BY prediction_date;
```

### Step 3: Verify Performance Table

```sql
SELECT
  COUNT(*) as total_days,
  MIN(prediction_date) as first_date,
  MAX(prediction_date) as last_date,
  ROUND(AVG(daily_mae), 2) as overall_mae,
  ROUND(AVG(daily_rmse), 2) as overall_rmse,
  ROUND(AVG(within_5_dollars_pct), 1) as avg_within_5_pct
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`;
```

**Expected Output**:

| total_days | first_date | last_date | overall_mae | overall_rmse | avg_within_5_pct |
|------------|------------|-----------|-------------|--------------|------------------|
| 92 | 2023-07-01 | 2023-09-30 | 3.12 | 3.96 | 82.8 |

### Step 4: View Daily Performance

```sql
SELECT
  prediction_date,
  prediction_count,
  daily_mae,
  daily_rmse,
  within_5_dollars_pct,
  mean_bias
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
ORDER BY prediction_date DESC
LIMIT 10;
```

**Expected Output**:

| prediction_date | prediction_count | daily_mae | daily_rmse | within_5_dollars_pct | mean_bias |
|-----------------|------------------|-----------|------------|---------------------|-----------|
| 2023-09-30 | 12,156 | 3.52 | 4.47 | 82.1 | 0.08 |
| 2023-09-29 | 17,876 | 3.68 | 4.65 | 81.3 | 0.12 |
| 2023-09-28 | 19,071 | 3.75 | 4.72 | 80.9 | 0.15 |
| 2023-09-27 | 18,288 | 3.37 | 4.28 | 83.4 | 0.05 |
| 2023-09-26 | 18,371 | 3.37 | 4.28 | 83.4 | 0.05 |

### Step 5: Weekly Aggregation View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.performance_weekly` AS
SELECT
  DATE_TRUNC(prediction_date, WEEK) as week_start,
  SUM(prediction_count) as total_predictions,
  ROUND(AVG(daily_mae), 3) as weekly_mae,
  ROUND(AVG(daily_rmse), 3) as weekly_rmse,
  ROUND(AVG(within_5_dollars_pct), 1) as weekly_accuracy_5,
  ROUND(AVG(mean_bias), 4) as weekly_bias
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
GROUP BY DATE_TRUNC(prediction_date, WEEK)
ORDER BY week_start DESC;
```

### Step 6: Performance Trend Analysis

```sql
SELECT
  prediction_date,
  daily_mae,
  ROUND(AVG(daily_mae) OVER (
    ORDER BY prediction_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ), 3) as mae_7day_avg,
  CASE
    WHEN daily_mae > AVG(daily_mae) OVER () + STDDEV(daily_mae) OVER () THEN 'HIGH'
    WHEN daily_mae < AVG(daily_mae) OVER () - STDDEV(daily_mae) OVER () THEN 'LOW'
    ELSE 'NORMAL'
  END as performance_status
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
ORDER BY prediction_date DESC
LIMIT 14;
```

---

## Performance Tracking Schema

| Column | Type | Description |
|--------|------|-------------|
| prediction_date | DATE | Date of predictions |
| prediction_count | INTEGER | Number of predictions |
| daily_mae | FLOAT64 | Mean Absolute Error |
| daily_mse | FLOAT64 | Mean Squared Error |
| daily_rmse | FLOAT64 | Root Mean Squared Error |
| daily_correlation | FLOAT64 | Prediction correlation |
| within_2_dollars_pct | FLOAT64 | % within $2 |
| within_5_dollars_pct | FLOAT64 | % within $5 |
| within_10_dollars_pct | FLOAT64 | % within $10 |
| avg_actual_fare | FLOAT64 | Average true fare |
| avg_predicted_fare | FLOAT64 | Average predicted fare |
| mean_bias | FLOAT64 | Average signed error |

---

## Performance Summary

### Overall Metrics (92 Days)

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Average MAE | $3.12 | < $5.00 | PASS |
| Average RMSE | $3.96 | < $6.00 | PASS |
| Within $5 | 82.8% | > 80% | PASS |
| Mean Bias | 0.08 | ~0 | PASS |

### Day-of-Week Pattern

```sql
SELECT
  EXTRACT(DAYOFWEEK FROM prediction_date) as dow,
  ROUND(AVG(daily_mae), 3) as avg_mae,
  ROUND(AVG(within_5_dollars_pct), 1) as avg_accuracy
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
GROUP BY dow
ORDER BY dow;
```

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Performance Monitoring | Tracking model metrics |
| Daily Aggregation | Time-based analysis |
| Trend Detection | Moving averages |
| Bias Monitoring | Systematic errors |

---

## Next Task

[TASK-014-002: Define Alert Thresholds](./TASK-014-002-define-thresholds.md)
