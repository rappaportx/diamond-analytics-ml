# MLOps & Monitoring Documentation

## Overview

Production-grade model monitoring infrastructure including performance tracking, data drift detection, and automated alerting. This phase demonstrates MLOps skills critical for the certification exam.

---

## MLOps Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           MLOPS INFRASTRUCTURE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                      PERFORMANCE TRACKING                            │   │
│   │                                                                      │   │
│   │   fare_predictions ──► performance_tracking ──► model_health_alerts │   │
│   │                                                                      │   │
│   │   Metrics: Daily MAE, RMSE, Correlation, % within thresholds        │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                        DRIFT MONITORING                              │   │
│   │                                                                      │   │
│   │   train_set (baseline) ──┐                                          │   │
│   │                          ├──► drift_monitoring ──► model_health_    │   │
│   │   test_set (current)  ───┘                          alerts          │   │
│   │                                                                      │   │
│   │   Metrics: Z-scores for miles, fare, duration, distance             │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                         ALERTING SYSTEM                              │   │
│   │                                                                      │   │
│   │   model_health_alerts (VIEW)                                        │   │
│   │   - Combines performance and drift alerts                           │   │
│   │   - Severity levels: OK, WARNING, CRITICAL, DRIFT_WARNING, DRIFT_   │   │
│   │                       ALERT                                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Performance Tracking

### Purpose
Monitor model prediction accuracy over time to detect performance degradation.

### Implementation

```sql
CREATE OR REPLACE TABLE performance_tracking AS
SELECT
  DATE(trip_start_timestamp) as prediction_date,
  COUNT(*) as num_predictions,

  -- Error metrics
  ROUND(AVG(absolute_error), 3) as daily_mae,
  ROUND(AVG(POW(actual_fare - predicted_fare, 2)), 3) as daily_mse,
  ROUND(SQRT(AVG(POW(actual_fare - predicted_fare, 2))), 3) as daily_rmse,

  -- Correlation
  ROUND(CORR(actual_fare, predicted_fare), 4) as daily_correlation,

  -- Threshold accuracy
  ROUND(COUNTIF(absolute_error < 2) / COUNT(*) * 100, 1) as within_2_dollars_pct,
  ROUND(COUNTIF(absolute_error < 5) / COUNT(*) * 100, 1) as within_5_dollars_pct,
  ROUND(COUNTIF(absolute_error < 10) / COUNT(*) * 100, 1) as within_10_dollars_pct

FROM fare_predictions
GROUP BY 1
ORDER BY 1;
```

### Schema

| Column | Type | Description |
|--------|------|-------------|
| prediction_date | DATE | Day of predictions |
| num_predictions | INT64 | Prediction count |
| daily_mae | FLOAT64 | Mean Absolute Error |
| daily_mse | FLOAT64 | Mean Squared Error |
| daily_rmse | FLOAT64 | Root Mean Squared Error |
| daily_correlation | FLOAT64 | Actual vs Predicted correlation |
| within_2_dollars_pct | FLOAT64 | % predictions within $2 |
| within_5_dollars_pct | FLOAT64 | % predictions within $5 |
| within_10_dollars_pct | FLOAT64 | % predictions within $10 |

### Sample Results

| Date | Predictions | MAE | RMSE | Correlation | Within $5 |
|------|-------------|-----|------|-------------|-----------|
| 2023-09-30 | 12,156 | $3.52 | $6.45 | 0.954 | 83.2% |
| 2023-09-29 | 17,876 | $3.68 | $6.78 | 0.952 | 82.1% |
| 2023-09-28 | 19,071 | $3.75 | $6.89 | 0.951 | 81.8% |

---

## Data Drift Detection

### Concept

**Certification Concept**: Data drift occurs when the statistical properties of input data change over time, potentially degrading model performance.

### Z-Score Method

Z-scores measure how many standard deviations the current mean is from the baseline mean:

```
Z-score = |current_mean - baseline_mean| / baseline_std
```

### Interpretation

| Z-Score | Interpretation | Action |
|---------|----------------|--------|
| < 1.0 | Normal variation | None |
| 1.0 - 1.5 | Slight shift | Monitor |
| 1.5 - 2.0 | Warning | Investigate |
| > 2.0 | Alert | Consider retraining |

### Implementation

```sql
CREATE OR REPLACE TABLE drift_monitoring AS
WITH baseline AS (
  -- Calculate statistics from training data
  SELECT
    AVG(trip_miles) as base_avg_miles,
    STDDEV(trip_miles) as base_std_miles,
    AVG(target_fare) as base_avg_fare,
    STDDEV(target_fare) as base_std_fare,
    AVG(trip_seconds) as base_avg_seconds,
    STDDEV(trip_seconds) as base_std_seconds,
    AVG(straight_line_km) as base_avg_distance,
    STDDEV(straight_line_km) as base_std_distance
  FROM train_set
  WHERE target_fare IS NOT NULL
),
daily_stats AS (
  -- Calculate daily statistics from test data
  SELECT
    DATE(trip_start_timestamp) as stat_date,
    AVG(trip_miles) as curr_avg_miles,
    AVG(target_fare) as curr_avg_fare,
    AVG(trip_seconds) as curr_avg_seconds,
    AVG(straight_line_km) as curr_avg_distance,
    COUNT(*) as sample_size
  FROM test_set
  GROUP BY 1
)
SELECT
  d.stat_date as window_date,
  d.sample_size,

  -- Z-scores for each feature
  ROUND(ABS(d.curr_avg_miles - b.base_avg_miles)
        / NULLIF(b.base_std_miles, 0), 3) as miles_zscore,
  ROUND(ABS(d.curr_avg_fare - b.base_avg_fare)
        / NULLIF(b.base_std_fare, 0), 3) as fare_zscore,
  ROUND(ABS(d.curr_avg_seconds - b.base_avg_seconds)
        / NULLIF(b.base_std_seconds, 0), 3) as duration_zscore,
  ROUND(ABS(d.curr_avg_distance - b.base_avg_distance)
        / NULLIF(b.base_std_distance, 0), 3) as distance_zscore,

  -- Drift status determination
  CASE
    WHEN ABS(d.curr_avg_miles - b.base_avg_miles)
         / NULLIF(b.base_std_miles, 0) > 2
      OR ABS(d.curr_avg_fare - b.base_avg_fare)
         / NULLIF(b.base_std_fare, 0) > 2
    THEN 'DRIFT_ALERT'
    WHEN ABS(d.curr_avg_miles - b.base_avg_miles)
         / NULLIF(b.base_std_miles, 0) > 1.5
      OR ABS(d.curr_avg_fare - b.base_avg_fare)
         / NULLIF(b.base_std_fare, 0) > 1.5
    THEN 'DRIFT_WARNING'
    ELSE 'OK'
  END as drift_status

FROM daily_stats d
CROSS JOIN baseline b
ORDER BY d.stat_date;
```

### Sample Results

| Date | Sample Size | Miles Z | Fare Z | Status |
|------|-------------|---------|--------|--------|
| 2023-09-30 | 12,156 | 0.114 | 0.102 | OK |
| 2023-09-29 | 17,876 | 0.133 | 0.152 | OK |
| 2023-09-28 | 19,071 | 0.120 | 0.158 | OK |

**Key Finding**: All Z-scores < 0.4, indicating **no significant data drift**.

---

## Alerting System

### Purpose
Unified view combining performance and drift alerts for monitoring dashboards.

### Implementation

```sql
CREATE OR REPLACE VIEW model_health_alerts AS
-- Performance alerts
SELECT
  CURRENT_TIMESTAMP() as alert_time,
  'PERFORMANCE' as alert_type,
  prediction_date as alert_date,
  CASE
    WHEN daily_mae > 5 THEN 'CRITICAL'
    WHEN daily_mae > 3.5 THEN 'WARNING'
    ELSE 'OK'
  END as severity,
  CONCAT('Daily MAE: ', CAST(ROUND(daily_mae, 2) AS STRING)) as message
FROM performance_tracking

UNION ALL

-- Drift alerts
SELECT
  CURRENT_TIMESTAMP(),
  'DATA_DRIFT',
  window_date,
  drift_status,
  CONCAT('Miles Z-score: ', CAST(miles_zscore AS STRING),
         ', Fare Z-score: ', CAST(fare_zscore AS STRING))
FROM drift_monitoring
WHERE drift_status != 'OK';
```

### Alert Thresholds

| Alert Type | Severity | Condition |
|------------|----------|-----------|
| PERFORMANCE | OK | MAE <= $3.50 |
| PERFORMANCE | WARNING | MAE $3.50 - $5.00 |
| PERFORMANCE | CRITICAL | MAE > $5.00 |
| DATA_DRIFT | OK | All Z-scores < 1.5 |
| DATA_DRIFT | DRIFT_WARNING | Any Z-score 1.5 - 2.0 |
| DATA_DRIFT | DRIFT_ALERT | Any Z-score > 2.0 |

### Sample Alerts

| Alert Type | Date | Severity | Message |
|------------|------|----------|---------|
| PERFORMANCE | 2023-09-30 | WARNING | Daily MAE: 3.52 |
| PERFORMANCE | 2023-09-29 | WARNING | Daily MAE: 3.68 |
| PERFORMANCE | 2023-09-27 | OK | Daily MAE: 3.37 |

---

## Monitoring Dashboard Queries

### Daily Performance Summary

```sql
SELECT
  prediction_date,
  daily_mae,
  within_5_dollars_pct,
  CASE
    WHEN daily_mae > 5 THEN '🔴 CRITICAL'
    WHEN daily_mae > 3.5 THEN '🟡 WARNING'
    ELSE '🟢 OK'
  END as status
FROM performance_tracking
ORDER BY prediction_date DESC
LIMIT 7;
```

### Weekly Drift Summary

```sql
SELECT
  window_date,
  miles_zscore,
  fare_zscore,
  drift_status
FROM drift_monitoring
ORDER BY window_date DESC
LIMIT 7;
```

### Active Alerts

```sql
SELECT *
FROM model_health_alerts
WHERE severity NOT IN ('OK')
ORDER BY alert_date DESC
LIMIT 20;
```

---

## MLOps Best Practices Demonstrated

| Practice | Implementation |
|----------|----------------|
| **Continuous Monitoring** | Daily performance tracking |
| **Statistical Drift Detection** | Z-score based drift monitoring |
| **Automated Alerting** | Threshold-based alert view |
| **Baseline Comparison** | Training data as reference |
| **Multiple Metrics** | MAE, RMSE, correlation, thresholds |
| **Actionable Insights** | Severity levels guide response |

---

## Production Considerations

### Not Implemented (Out of Scope)

| Feature | Why Not Included |
|---------|------------------|
| Scheduled Refresh | Requires Cloud Scheduler |
| Email/SMS Alerts | Requires Cloud Functions |
| Dashboard UI | Requires Looker/Data Studio |
| Auto-Retraining | Requires Vertex AI Pipelines |

### Recommended Extensions

1. **Scheduled Queries**: Refresh monitoring tables hourly
2. **Cloud Monitoring Integration**: Export metrics to Cloud Monitoring
3. **Pub/Sub Alerts**: Trigger notifications on threshold breaches
4. **Looker Dashboard**: Visualize monitoring data

---

## Certification Topics Demonstrated

| Topic | How Demonstrated |
|-------|------------------|
| Model Monitoring | Performance tracking table |
| Drift Detection | Z-score based monitoring |
| Alert Systems | Unified alert view |
| Statistical Methods | Mean, std, Z-scores |
| MLOps Principles | Continuous monitoring design |

---

## Navigation

- **Previous**: [Model Evaluation](./06-model-evaluation.md)
- **Next**: [Certification Mapping](./08-certification-mapping.md)
