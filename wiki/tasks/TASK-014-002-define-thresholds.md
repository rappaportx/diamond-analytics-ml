# TASK-014-002: Define Alert Thresholds

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-014-002 |
| User Story | [US-014: Performance Tracking & Alerting](../user-stories/US-014-alerting-system.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Define performance and drift alert thresholds for automated model health monitoring.

---

## Prerequisites

- [x] Performance tracking table created (TASK-014-001)
- [x] Drift monitoring table created (TASK-013-005)
- [x] Understanding of baseline performance

---

## Step-by-Step Instructions

### Step 1: Analyze Historical Performance

```sql
SELECT
  ROUND(AVG(daily_mae), 3) as avg_mae,
  ROUND(STDDEV(daily_mae), 3) as std_mae,
  ROUND(APPROX_QUANTILES(daily_mae, 100)[OFFSET(50)], 3) as median_mae,
  ROUND(APPROX_QUANTILES(daily_mae, 100)[OFFSET(75)], 3) as p75_mae,
  ROUND(APPROX_QUANTILES(daily_mae, 100)[OFFSET(90)], 3) as p90_mae,
  ROUND(MAX(daily_mae), 3) as max_mae
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`;
```

**Expected Output**:

| avg_mae | std_mae | median_mae | p75_mae | p90_mae | max_mae |
|---------|---------|------------|---------|---------|---------|
| 3.12 | 0.45 | 3.05 | 3.45 | 3.78 | 4.23 |

### Step 2: Define Performance Thresholds

Based on historical analysis:

| Severity | MAE Range | Color | Action |
|----------|-----------|-------|--------|
| OK | <= $3.50 | Green | None required |
| WARNING | $3.50 - $5.00 | Yellow | Monitor closely |
| CRITICAL | > $5.00 | Red | Immediate investigation |

**Rationale**:
- $3.50 ≈ avg + 1 std dev (83rd percentile)
- $5.00 ≈ avg + 4 std dev (business threshold)

### Step 3: Define Drift Thresholds

| Severity | Z-Score Range | Action |
|----------|---------------|--------|
| OK | < 1.5 | Normal variation |
| DRIFT_WARNING | 1.5 - 2.0 | Investigate pattern changes |
| DRIFT_ALERT | > 2.0 | Consider model retraining |

**Statistical Basis**:
- Z = 1.5: ~93% of normal variation
- Z = 2.0: ~95% of normal variation

### Step 4: Create Threshold Configuration Table

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.alert_thresholds` AS
SELECT * FROM UNNEST([
  STRUCT(
    'PERFORMANCE_MAE' as metric,
    'OK' as severity_low,
    'WARNING' as severity_high,
    3.50 as threshold_value,
    'Daily MAE between OK and WARNING' as description
  ),
  STRUCT(
    'PERFORMANCE_MAE',
    'WARNING',
    'CRITICAL',
    5.00,
    'Daily MAE between WARNING and CRITICAL'
  ),
  STRUCT(
    'DRIFT_ZSCORE',
    'OK',
    'DRIFT_WARNING',
    1.5,
    'Z-score between OK and WARNING'
  ),
  STRUCT(
    'DRIFT_ZSCORE',
    'DRIFT_WARNING',
    'DRIFT_ALERT',
    2.0,
    'Z-score between WARNING and ALERT'
  ),
  STRUCT(
    'ACCURACY_5',
    'CRITICAL',
    'WARNING',
    75.0,
    'Within $5 accuracy below 75%'
  ),
  STRUCT(
    'ACCURACY_5',
    'WARNING',
    'OK',
    80.0,
    'Within $5 accuracy above 80%'
  ),
  STRUCT(
    'BIAS',
    'OK',
    'WARNING',
    1.0,
    'Mean bias magnitude exceeds $1'
  ),
  STRUCT(
    'BIAS',
    'WARNING',
    'CRITICAL',
    2.0,
    'Mean bias magnitude exceeds $2'
  )
]);
```

### Step 5: Verify Thresholds

```sql
SELECT
  metric,
  severity_low,
  severity_high,
  threshold_value,
  description
FROM `sonorous-key-320714.diamond_analytics.alert_thresholds`
ORDER BY metric, threshold_value;
```

### Step 6: Test Threshold Application

```sql
SELECT
  prediction_date,
  daily_mae,
  within_5_dollars_pct,
  mean_bias,
  CASE
    WHEN daily_mae > 5.0 THEN 'CRITICAL'
    WHEN daily_mae > 3.5 THEN 'WARNING'
    ELSE 'OK'
  END as performance_status,
  CASE
    WHEN within_5_dollars_pct < 75 THEN 'CRITICAL'
    WHEN within_5_dollars_pct < 80 THEN 'WARNING'
    ELSE 'OK'
  END as accuracy_status,
  CASE
    WHEN ABS(mean_bias) > 2.0 THEN 'CRITICAL'
    WHEN ABS(mean_bias) > 1.0 THEN 'WARNING'
    ELSE 'OK'
  END as bias_status
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
ORDER BY prediction_date DESC
LIMIT 10;
```

---

## Complete Threshold Reference

### Performance Metrics

| Metric | OK | WARNING | CRITICAL |
|--------|-----|---------|----------|
| Daily MAE | <= $3.50 | $3.50 - $5.00 | > $5.00 |
| Daily RMSE | <= $4.50 | $4.50 - $6.50 | > $6.50 |
| Within $5 | >= 80% | 75% - 80% | < 75% |
| Within $10 | >= 90% | 85% - 90% | < 85% |
| Mean Bias | < $1.00 | $1.00 - $2.00 | > $2.00 |
| Correlation | >= 0.90 | 0.80 - 0.90 | < 0.80 |

### Drift Metrics

| Metric | OK | DRIFT_WARNING | DRIFT_ALERT |
|--------|-----|---------------|-------------|
| Miles Z-Score | < 1.5 | 1.5 - 2.0 | > 2.0 |
| Fare Z-Score | < 1.5 | 1.5 - 2.0 | > 2.0 |
| Max Z-Score | < 1.5 | 1.5 - 2.0 | > 2.0 |

---

## Threshold Justification

### Why These Values?

| Threshold | Value | Justification |
|-----------|-------|---------------|
| MAE OK | $3.50 | ~P85 of historical performance |
| MAE CRITICAL | $5.00 | Business requirement |
| Z-Score WARNING | 1.5 | ~93% confidence |
| Z-Score ALERT | 2.0 | ~95% confidence |
| Accuracy OK | 80% | Business requirement |

### Trade-offs

| Stricter Thresholds | Looser Thresholds |
|--------------------|-------------------|
| More false alarms | Miss real issues |
| Faster detection | Delayed response |
| Higher operational cost | Higher risk |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Alert Thresholds | Defining trigger points |
| Statistical Basis | Z-scores, percentiles |
| Multi-Level Severity | Graduated response |
| Threshold Configuration | Centralized management |

---

## Next Task

[TASK-014-003: Create Model Health Alerts View](./TASK-014-003-create-alert-view.md)
