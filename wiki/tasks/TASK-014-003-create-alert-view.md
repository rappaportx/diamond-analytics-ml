# TASK-014-003: Create Model Health Alerts View

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-014-003 |
| User Story | [US-014: Performance Tracking & Alerting](../user-stories/US-014-alerting-system.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Create a unified alert view that combines performance alerts and drift alerts for comprehensive model health monitoring.

---

## Prerequisites

- [x] Performance tracking table created (TASK-014-001)
- [x] Drift monitoring table created (TASK-013-005)
- [x] Alert thresholds defined (TASK-014-002)

---

## Step-by-Step Instructions

### Step 1: Create Unified Alert View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.model_health_alerts` AS

-- Performance Alerts
SELECT
  CURRENT_TIMESTAMP() as alert_timestamp,
  'PERFORMANCE' as alert_type,
  prediction_date as alert_date,
  CASE
    WHEN daily_mae > 5 THEN 'CRITICAL'
    WHEN daily_mae > 3.5 THEN 'WARNING'
    ELSE 'OK'
  END as severity,
  CONCAT(
    'Daily MAE: $', CAST(ROUND(daily_mae, 2) AS STRING),
    ' | RMSE: $', CAST(ROUND(daily_rmse, 2) AS STRING),
    ' | Within $5: ', CAST(ROUND(within_5_dollars_pct, 1) AS STRING), '%'
  ) as alert_message,
  daily_mae as metric_value,
  prediction_count as sample_size
FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
WHERE daily_mae > 3.5  -- Only show warnings and above

UNION ALL

-- Drift Alerts
SELECT
  CURRENT_TIMESTAMP() as alert_timestamp,
  'DATA_DRIFT' as alert_type,
  window_date as alert_date,
  drift_status as severity,
  CONCAT(
    'Z-scores: miles=', CAST(ROUND(miles_zscore, 2) AS STRING),
    ', fare=', CAST(ROUND(fare_zscore, 2) AS STRING),
    ' | Max Z: ', CAST(ROUND(max_zscore, 2) AS STRING)
  ) as alert_message,
  max_zscore as metric_value,
  sample_size
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`
WHERE drift_status != 'OK'

ORDER BY alert_date DESC;
```

### Step 2: Create Complete Health Dashboard View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.model_health_dashboard` AS
WITH performance_summary AS (
  SELECT
    COUNT(*) as total_days,
    COUNTIF(daily_mae <= 3.5) as ok_days,
    COUNTIF(daily_mae > 3.5 AND daily_mae <= 5) as warning_days,
    COUNTIF(daily_mae > 5) as critical_days,
    ROUND(AVG(daily_mae), 2) as avg_mae,
    ROUND(AVG(within_5_dollars_pct), 1) as avg_accuracy_5
  FROM `sonorous-key-320714.diamond_analytics.performance_tracking`
),
drift_summary AS (
  SELECT
    COUNT(*) as monitored_days,
    COUNTIF(drift_status = 'OK') as stable_days,
    COUNTIF(drift_status = 'DRIFT_WARNING') as drift_warnings,
    COUNTIF(drift_status = 'DRIFT_ALERT') as drift_alerts,
    ROUND(AVG(max_zscore), 3) as avg_max_zscore
  FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`
),
recent_alerts AS (
  SELECT COUNT(*) as alerts_last_7_days
  FROM `sonorous-key-320714.diamond_analytics.model_health_alerts`
  WHERE alert_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
)
SELECT
  -- Performance metrics
  p.total_days as performance_days_tracked,
  p.ok_days as performance_ok_days,
  p.warning_days as performance_warning_days,
  p.critical_days as performance_critical_days,
  p.avg_mae as average_mae,
  p.avg_accuracy_5 as avg_within_5_dollars_pct,

  -- Drift metrics
  d.monitored_days as drift_days_monitored,
  d.stable_days as drift_stable_days,
  d.drift_warnings,
  d.drift_alerts,
  d.avg_max_zscore,

  -- Recent activity
  r.alerts_last_7_days,

  -- Overall health score (0-100)
  ROUND(
    (p.ok_days / NULLIF(p.total_days, 0) * 50) +
    (d.stable_days / NULLIF(d.monitored_days, 0) * 50),
    1
  ) as overall_health_score,

  -- Health status
  CASE
    WHEN p.critical_days > 0 OR d.drift_alerts > 0 THEN 'CRITICAL'
    WHEN p.warning_days > 5 OR d.drift_warnings > 3 THEN 'WARNING'
    ELSE 'HEALTHY'
  END as overall_status

FROM performance_summary p
CROSS JOIN drift_summary d
CROSS JOIN recent_alerts r;
```

### Step 3: Query Current Health Status

```sql
SELECT * FROM `sonorous-key-320714.diamond_analytics.model_health_dashboard`;
```

**Expected Output**:

| Metric | Value |
|--------|-------|
| performance_days_tracked | 92 |
| performance_ok_days | 35 |
| performance_warning_days | 57 |
| performance_critical_days | 0 |
| average_mae | 3.52 |
| avg_within_5_dollars_pct | 82.8 |
| drift_days_monitored | 92 |
| drift_stable_days | 92 |
| drift_warnings | 0 |
| drift_alerts | 0 |
| alerts_last_7_days | 5 |
| overall_health_score | 69.0 |
| overall_status | WARNING |

### Step 4: Query Recent Alerts

```sql
SELECT
  alert_date,
  alert_type,
  severity,
  alert_message
FROM `sonorous-key-320714.diamond_analytics.model_health_alerts`
ORDER BY alert_date DESC
LIMIT 10;
```

**Expected Output**:

| alert_date | alert_type | severity | alert_message |
|------------|------------|----------|---------------|
| 2023-09-30 | PERFORMANCE | WARNING | Daily MAE: $3.52 \| RMSE: $4.47 \| Within $5: 82.1% |
| 2023-09-29 | PERFORMANCE | WARNING | Daily MAE: $3.68 \| RMSE: $4.65 \| Within $5: 81.3% |
| 2023-09-28 | PERFORMANCE | WARNING | Daily MAE: $3.75 \| RMSE: $4.72 \| Within $5: 80.9% |
| 2023-09-27 | PERFORMANCE | OK | Daily MAE: $3.37 \| RMSE: $4.28 \| Within $5: 83.4% |
| 2023-09-26 | PERFORMANCE | OK | Daily MAE: $3.37 \| RMSE: $4.28 \| Within $5: 83.4% |

### Step 5: Create Alert Summary by Type

```sql
SELECT
  alert_type,
  severity,
  COUNT(*) as alert_count,
  MIN(alert_date) as first_occurrence,
  MAX(alert_date) as last_occurrence
FROM `sonorous-key-320714.diamond_analytics.model_health_alerts`
GROUP BY alert_type, severity
ORDER BY alert_type, severity;
```

**Expected Output**:

| alert_type | severity | alert_count | first_occurrence | last_occurrence |
|------------|----------|-------------|------------------|-----------------|
| PERFORMANCE | WARNING | 57 | 2023-07-05 | 2023-09-30 |
| PERFORMANCE | CRITICAL | 0 | NULL | NULL |
| DATA_DRIFT | DRIFT_WARNING | 0 | NULL | NULL |
| DATA_DRIFT | DRIFT_ALERT | 0 | NULL | NULL |

---

## Alert View Schema

| Column | Type | Description |
|--------|------|-------------|
| alert_timestamp | TIMESTAMP | When alert was generated |
| alert_type | STRING | PERFORMANCE or DATA_DRIFT |
| alert_date | DATE | Date of the issue |
| severity | STRING | OK/WARNING/CRITICAL or drift status |
| alert_message | STRING | Detailed alert description |
| metric_value | FLOAT64 | Primary metric value |
| sample_size | INTEGER | Number of predictions/samples |

---

## Health Score Calculation

```
Health Score = (Performance OK%) * 50 + (Drift Stable%) * 50
```

| Score Range | Status | Action |
|-------------|--------|--------|
| 90-100 | Excellent | Continue monitoring |
| 70-89 | Good | Review warnings |
| 50-69 | Fair | Investigate issues |
| < 50 | Poor | Immediate action needed |

---

## Alert Summary

### Performance Alerts (92 Days)

| Severity | Count | Percentage |
|----------|-------|------------|
| OK | 35 | 38% |
| WARNING | 57 | 62% |
| CRITICAL | 0 | 0% |

### Drift Alerts (92 Days)

| Status | Count | Percentage |
|--------|-------|------------|
| OK | 92 | 100% |
| DRIFT_WARNING | 0 | 0% |
| DRIFT_ALERT | 0 | 0% |

### Key Findings

1. **Model stable with moderate warnings** - No critical performance issues
2. **No data drift detected** - Input distributions remain stable
3. **MAE trending slightly above target** - Consider threshold adjustment or retraining

---

## Automated Alert Query

Schedule daily:

```sql
-- Morning alert check
SELECT *
FROM `sonorous-key-320714.diamond_analytics.model_health_alerts`
WHERE alert_date = CURRENT_DATE() - 1
  AND severity IN ('CRITICAL', 'DRIFT_ALERT')
ORDER BY alert_type;
```

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Unified Monitoring | Combined performance and drift |
| Alert Views | Real-time health visibility |
| Health Scoring | Quantified model health |
| MLOps Dashboard | Operational monitoring |

---

## Navigation

- **User Story**: [US-014: Alerting System](../user-stories/US-014-alerting-system.md)
- **Previous**: [TASK-014-002: Define Alert Thresholds](./TASK-014-002-define-thresholds.md)
- **Back to**: [Wiki Home](../README.md)
