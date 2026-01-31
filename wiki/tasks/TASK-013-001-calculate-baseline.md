# TASK-013-001: Calculate Baseline Statistics

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-013-001 |
| User Story | [US-013: Drift Monitoring](../user-stories/US-013-drift-monitoring.md) |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |

---

## Objective

Calculate and store baseline statistics from the training data for comparison during drift detection.

---

## Prerequisites

- [x] Training data available (train_set)
- [x] Understanding of statistical measures
- [x] Dataset for storing baselines

---

## Step-by-Step Instructions

### Step 1: Understand Drift Detection

**What is Data Drift?**
- Changes in input data distribution over time
- Can degrade model performance
- Detected by comparing current stats to training baseline

**Z-Score Formula**:
```
Z-score = |current_mean - baseline_mean| / baseline_std_dev
```

### Step 2: Identify Features to Monitor

| Feature | Type | Why Monitor |
|---------|------|-------------|
| trip_miles | FLOAT64 | Core predictor, distribution changes |
| trip_seconds | INTEGER | Duration patterns may shift |
| target_fare | FLOAT64 | Target variable drift |
| straight_line_km | FLOAT64 | Geographic patterns |

### Step 3: Calculate Baseline Statistics

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.baseline_statistics` AS
SELECT
  'train_set' as source_table,
  CURRENT_TIMESTAMP() as calculated_at,

  -- trip_miles statistics
  AVG(trip_miles) as trip_miles_mean,
  STDDEV(trip_miles) as trip_miles_std,
  MIN(trip_miles) as trip_miles_min,
  MAX(trip_miles) as trip_miles_max,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(50)] as trip_miles_median,

  -- trip_seconds statistics
  AVG(trip_seconds) as trip_seconds_mean,
  STDDEV(trip_seconds) as trip_seconds_std,
  MIN(trip_seconds) as trip_seconds_min,
  MAX(trip_seconds) as trip_seconds_max,
  APPROX_QUANTILES(trip_seconds, 100)[OFFSET(50)] as trip_seconds_median,

  -- target_fare statistics
  AVG(target_fare) as target_fare_mean,
  STDDEV(target_fare) as target_fare_std,
  MIN(target_fare) as target_fare_min,
  MAX(target_fare) as target_fare_max,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(50)] as target_fare_median,

  -- straight_line_km statistics
  AVG(straight_line_km) as straight_line_km_mean,
  STDDEV(straight_line_km) as straight_line_km_std,

  -- Sample size
  COUNT(*) as sample_size

FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

### Step 4: Verify Baseline Statistics

```sql
SELECT
  source_table,
  sample_size,
  ROUND(trip_miles_mean, 2) as miles_mean,
  ROUND(trip_miles_std, 2) as miles_std,
  ROUND(target_fare_mean, 2) as fare_mean,
  ROUND(target_fare_std, 2) as fare_std
FROM `sonorous-key-320714.diamond_analytics.baseline_statistics`;
```

**Expected Output**:

| source_table | sample_size | miles_mean | miles_std | fare_mean | fare_std |
|--------------|-------------|------------|-----------|-----------|----------|
| train_set | 5,299,220 | 3.42 | 4.25 | 18.47 | 12.56 |

### Step 5: Document Baseline Reference

| Feature | Mean | Std Dev | Min | Median | Max |
|---------|------|---------|-----|--------|-----|
| trip_miles | 3.42 | 4.25 | 0.01 | 2.10 | 99.9 |
| trip_seconds | 948 | 756 | 61 | 720 | 14399 |
| target_fare | 18.47 | 12.56 | 0.01 | 12.75 | 499.98 |
| straight_line_km | 4.87 | 5.93 | 0.01 | 2.89 | 45.6 |

### Step 6: Create Baseline Percentiles

For additional monitoring:

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.baseline_percentiles` AS
SELECT
  'trip_miles' as feature,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(5)] as p5,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(25)] as p25,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(50)] as p50,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(75)] as p75,
  APPROX_QUANTILES(trip_miles, 100)[OFFSET(95)] as p95
FROM `sonorous-key-320714.diamond_analytics.train_set`

UNION ALL

SELECT
  'target_fare' as feature,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(5)] as p5,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(25)] as p25,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(50)] as p50,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(75)] as p75,
  APPROX_QUANTILES(target_fare, 100)[OFFSET(95)] as p95
FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

**Expected Output**:

| feature | p5 | p25 | p50 | p75 | p95 |
|---------|-----|-----|-----|-----|-----|
| trip_miles | 0.5 | 1.2 | 2.1 | 4.2 | 12.5 |
| target_fare | 6.50 | 9.25 | 12.75 | 21.50 | 48.75 |

---

## Baseline Statistics Summary

### Core Metrics for Z-Score Calculation

| Feature | Mean (mu) | Std Dev (sigma) | Used For |
|---------|-----------|-----------------|----------|
| trip_miles | 3.42 | 4.25 | Distance drift |
| target_fare | 18.47 | 12.56 | Fare drift |
| trip_seconds | 948 | 756 | Duration drift |
| straight_line_km | 4.87 | 5.93 | Route drift |

### Z-Score Thresholds

| Z-Score | Interpretation | Action |
|---------|----------------|--------|
| < 1.0 | Normal variation | None |
| 1.0 - 1.5 | Slight shift | Monitor |
| 1.5 - 2.0 | Notable shift | Investigate |
| > 2.0 | Significant drift | Consider retraining |

---

## Example Z-Score Calculation

If a new day has avg trip_miles = 3.85:

```
Z = |3.85 - 3.42| / 4.25 = 0.43 / 4.25 = 0.10
```

Z = 0.10 < 1.0 → Normal variation, no drift detected.

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Baseline Statistics | Reference for comparison |
| Standard Deviation | Measure of variation |
| Percentiles | Distribution shape |
| Drift Detection | Monitoring data changes |

---

## Next Task

[TASK-013-002: Calculate Daily Statistics](./TASK-013-002-daily-statistics.md)
