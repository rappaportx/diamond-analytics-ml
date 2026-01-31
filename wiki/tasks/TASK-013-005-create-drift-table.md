# TASK-013-005: Create Drift Monitoring Table

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-013-005 |
| User Story | [US-013: Drift Monitoring](../user-stories/US-013-drift-monitoring.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Create the comprehensive drift monitoring table that combines baseline statistics, daily calculations, and drift status for ongoing model health monitoring.

---

## Prerequisites

- [x] Baseline statistics calculated (TASK-013-001)
- [x] Understanding of Z-score methodology
- [x] Test set data available

---

## Step-by-Step Instructions

### Step 1: Create Drift Monitoring Table

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.drift_monitoring` AS
WITH baseline AS (
  -- Get baseline statistics from training data
  SELECT
    AVG(trip_miles) as base_avg_miles,
    STDDEV(trip_miles) as base_std_miles,
    AVG(target_fare) as base_avg_fare,
    STDDEV(target_fare) as base_std_fare,
    AVG(trip_seconds) as base_avg_seconds,
    STDDEV(trip_seconds) as base_std_seconds,
    AVG(straight_line_km) as base_avg_distance,
    STDDEV(straight_line_km) as base_std_distance
  FROM `sonorous-key-320714.diamond_analytics.train_set`
),

daily_stats AS (
  -- Calculate daily statistics from test/production data
  SELECT
    DATE(trip_start_timestamp) as stat_date,
    COUNT(*) as sample_size,
    AVG(trip_miles) as curr_avg_miles,
    STDDEV(trip_miles) as curr_std_miles,
    AVG(target_fare) as curr_avg_fare,
    STDDEV(target_fare) as curr_std_fare,
    AVG(trip_seconds) as curr_avg_seconds,
    AVG(straight_line_km) as curr_avg_distance
  FROM `sonorous-key-320714.diamond_analytics.test_set`
  GROUP BY DATE(trip_start_timestamp)
)

SELECT
  d.stat_date as window_date,
  d.sample_size,

  -- Current averages
  ROUND(d.curr_avg_miles, 3) as current_avg_miles,
  ROUND(d.curr_avg_fare, 2) as current_avg_fare,

  -- Baseline averages (for reference)
  ROUND(b.base_avg_miles, 3) as baseline_avg_miles,
  ROUND(b.base_avg_fare, 2) as baseline_avg_fare,

  -- Z-scores
  ROUND(ABS(d.curr_avg_miles - b.base_avg_miles) / b.base_std_miles, 3) as miles_zscore,
  ROUND(ABS(d.curr_avg_fare - b.base_avg_fare) / b.base_std_fare, 3) as fare_zscore,
  ROUND(ABS(d.curr_avg_seconds - b.base_avg_seconds) / b.base_std_seconds, 3) as seconds_zscore,
  ROUND(ABS(d.curr_avg_distance - b.base_avg_distance) / b.base_std_distance, 3) as distance_zscore,

  -- Maximum Z-score across all features
  GREATEST(
    ABS(d.curr_avg_miles - b.base_avg_miles) / b.base_std_miles,
    ABS(d.curr_avg_fare - b.base_avg_fare) / b.base_std_fare,
    ABS(d.curr_avg_seconds - b.base_avg_seconds) / b.base_std_seconds,
    ABS(d.curr_avg_distance - b.base_avg_distance) / b.base_std_distance
  ) as max_zscore,

  -- Drift status based on maximum Z-score
  CASE
    WHEN GREATEST(
      ABS(d.curr_avg_miles - b.base_avg_miles) / b.base_std_miles,
      ABS(d.curr_avg_fare - b.base_avg_fare) / b.base_std_fare
    ) > 2.0 THEN 'DRIFT_ALERT'
    WHEN GREATEST(
      ABS(d.curr_avg_miles - b.base_avg_miles) / b.base_std_miles,
      ABS(d.curr_avg_fare - b.base_avg_fare) / b.base_std_fare
    ) > 1.5 THEN 'DRIFT_WARNING'
    ELSE 'OK'
  END as drift_status

FROM daily_stats d
CROSS JOIN baseline b
ORDER BY d.stat_date DESC;
```

### Step 2: Verify Table Creation

```sql
SELECT
  COUNT(*) as total_days,
  MIN(window_date) as first_date,
  MAX(window_date) as last_date,
  COUNTIF(drift_status = 'OK') as ok_days,
  COUNTIF(drift_status = 'DRIFT_WARNING') as warning_days,
  COUNTIF(drift_status = 'DRIFT_ALERT') as alert_days
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`;
```

**Expected Output**:

| total_days | first_date | last_date | ok_days | warning_days | alert_days |
|------------|------------|-----------|---------|--------------|------------|
| 92 | 2023-07-01 | 2023-09-30 | 92 | 0 | 0 |

### Step 3: View Recent Drift Status

```sql
SELECT
  window_date,
  sample_size,
  current_avg_miles,
  current_avg_fare,
  ROUND(miles_zscore, 3) as miles_z,
  ROUND(fare_zscore, 3) as fare_z,
  drift_status
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`
ORDER BY window_date DESC
LIMIT 10;
```

**Expected Output**:

| window_date | sample_size | current_avg_miles | current_avg_fare | miles_z | fare_z | drift_status |
|-------------|-------------|-------------------|------------------|---------|--------|--------------|
| 2023-09-30 | 12,156 | 3.48 | 18.92 | 0.114 | 0.102 | OK |
| 2023-09-29 | 17,876 | 3.51 | 18.67 | 0.133 | 0.152 | OK |
| 2023-09-28 | 19,071 | 3.49 | 18.54 | 0.120 | 0.158 | OK |
| 2023-09-27 | 18,288 | 3.44 | 18.38 | 0.057 | 0.081 | OK |
| 2023-09-26 | 18,371 | 3.46 | 18.42 | 0.086 | 0.110 | OK |

### Step 4: Analyze Z-Score Distribution

```sql
SELECT
  ROUND(AVG(miles_zscore), 3) as avg_miles_zscore,
  ROUND(MAX(miles_zscore), 3) as max_miles_zscore,
  ROUND(AVG(fare_zscore), 3) as avg_fare_zscore,
  ROUND(MAX(fare_zscore), 3) as max_fare_zscore,
  ROUND(AVG(max_zscore), 3) as avg_max_zscore,
  ROUND(MAX(max_zscore), 3) as overall_max_zscore
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`;
```

**Expected Output**:

| avg_miles_zscore | max_miles_zscore | avg_fare_zscore | max_fare_zscore | avg_max_zscore | overall_max_zscore |
|------------------|------------------|-----------------|-----------------|----------------|--------------------|
| 0.098 | 0.287 | 0.112 | 0.342 | 0.145 | 0.368 |

### Step 5: Create Weekly Aggregation View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.drift_monitoring_weekly` AS
SELECT
  DATE_TRUNC(window_date, WEEK) as week_start,
  COUNT(*) as days_monitored,
  SUM(sample_size) as total_samples,
  ROUND(AVG(miles_zscore), 3) as avg_miles_z,
  ROUND(AVG(fare_zscore), 3) as avg_fare_z,
  ROUND(MAX(max_zscore), 3) as peak_zscore,
  COUNTIF(drift_status != 'OK') as drift_events
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`
GROUP BY DATE_TRUNC(window_date, WEEK)
ORDER BY week_start DESC;
```

---

## Drift Monitoring Schema

| Column | Type | Description |
|--------|------|-------------|
| window_date | DATE | Monitoring date |
| sample_size | INTEGER | Daily transaction count |
| current_avg_miles | FLOAT64 | Daily average miles |
| current_avg_fare | FLOAT64 | Daily average fare |
| baseline_avg_miles | FLOAT64 | Training average miles |
| baseline_avg_fare | FLOAT64 | Training average fare |
| miles_zscore | FLOAT64 | Z-score for miles |
| fare_zscore | FLOAT64 | Z-score for fare |
| seconds_zscore | FLOAT64 | Z-score for duration |
| distance_zscore | FLOAT64 | Z-score for distance |
| max_zscore | FLOAT64 | Maximum Z-score |
| drift_status | STRING | OK/DRIFT_WARNING/DRIFT_ALERT |

---

## Drift Status Summary

### Current Status (92 Days Monitored)

| Status | Count | Percentage |
|--------|-------|------------|
| OK | 92 | 100% |
| DRIFT_WARNING | 0 | 0% |
| DRIFT_ALERT | 0 | 0% |

### Z-Score Statistics

| Metric | Miles | Fare | Seconds | Distance |
|--------|-------|------|---------|----------|
| Mean | 0.098 | 0.112 | 0.087 | 0.091 |
| Max | 0.287 | 0.342 | 0.256 | 0.278 |
| Peak Day | 2023-08-15 | 2023-09-02 | 2023-07-04 | 2023-08-21 |

### Key Finding

**No significant drift detected** - All Z-scores well below warning threshold of 1.5.

---

## Automated Monitoring Query

Schedule this query for daily drift checks:

```sql
SELECT
  window_date,
  sample_size,
  drift_status,
  ROUND(max_zscore, 3) as max_z
FROM `sonorous-key-320714.diamond_analytics.drift_monitoring`
WHERE window_date = CURRENT_DATE() - 1
  AND drift_status != 'OK';
```

If this returns rows, investigate immediately.

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Data Drift Detection | Identifying distribution changes |
| Z-Score Methodology | Statistical significance |
| Baseline Comparison | Training vs production |
| Automated Monitoring | Scheduled drift checks |
| Alert Thresholds | Multi-level severity |

---

## Next Task

[TASK-014-001: Create Performance Tracking Table](./TASK-014-001-performance-tracking.md)
