# US-013: Drift Monitoring

## User Story

**As a** ML Engineer preparing for certification,
**I want to** implement Z-score based data drift monitoring,
**So that** I can detect when input data distribution changes.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-013 |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Baseline statistics calculated from training data
- [x] Daily statistics calculated from test/production data
- [x] Z-scores computed for key features
- [x] Drift status logic implemented
- [x] Drift monitoring table created

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-013-001 | Calculate Baseline Statistics | Complete |
| TASK-013-002 | Calculate Daily Statistics | Complete |
| TASK-013-003 | Compute Z-Scores | Complete |
| TASK-013-004 | Create Drift Status Logic | Complete |
| TASK-013-005 | Create Drift Monitoring Table | Complete |

---

## Z-Score Methodology

### Formula
```
Z-score = |current_mean - baseline_mean| / baseline_std_dev
```

### Interpretation

| Z-Score | Status | Interpretation | Action |
|---------|--------|----------------|--------|
| < 1.0 | OK | Normal variation | None |
| 1.0 - 1.5 | OK | Slight shift | Monitor |
| 1.5 - 2.0 | DRIFT_WARNING | Notable shift | Investigate |
| > 2.0 | DRIFT_ALERT | Significant drift | Consider retraining |

---

## Features Monitored

| Feature | Baseline Mean | Baseline Std |
|---------|---------------|--------------|
| trip_miles | From train_set | From train_set |
| target_fare | From train_set | From train_set |
| trip_seconds | From train_set | From train_set |
| straight_line_km | From train_set | From train_set |

---

## Results

### Sample Drift Monitoring Output

| Date | Sample Size | Miles Z | Fare Z | Status |
|------|-------------|---------|--------|--------|
| 2023-09-30 | 12,156 | 0.114 | 0.102 | **OK** |
| 2023-09-29 | 17,876 | 0.133 | 0.152 | **OK** |
| 2023-09-28 | 19,071 | 0.120 | 0.158 | **OK** |
| 2023-09-27 | 18,288 | 0.057 | 0.081 | **OK** |
| 2023-09-26 | 18,371 | 0.086 | 0.110 | **OK** |

### Summary

| Metric | Value |
|--------|-------|
| Days Monitored | 92 |
| Max Z-Score | 0.37 |
| Drift Warnings | 0 |
| Drift Alerts | 0 |

**Key Finding**: **No significant drift detected** - data distribution stable.

---

## SQL Implementation

```sql
CREATE TABLE drift_monitoring AS
WITH baseline AS (
  SELECT
    AVG(trip_miles) as base_avg_miles,
    STDDEV(trip_miles) as base_std_miles,
    AVG(target_fare) as base_avg_fare,
    STDDEV(target_fare) as base_std_fare
  FROM train_set
),
daily_stats AS (
  SELECT
    DATE(trip_start_timestamp) as stat_date,
    AVG(trip_miles) as curr_avg_miles,
    AVG(target_fare) as curr_avg_fare,
    COUNT(*) as sample_size
  FROM test_set
  GROUP BY 1
)
SELECT
  stat_date as window_date,
  sample_size,
  ABS(curr_avg_miles - base_avg_miles) / base_std_miles as miles_zscore,
  ABS(curr_avg_fare - base_avg_fare) / base_std_fare as fare_zscore,
  CASE
    WHEN miles_zscore > 2 OR fare_zscore > 2 THEN 'DRIFT_ALERT'
    WHEN miles_zscore > 1.5 OR fare_zscore > 1.5 THEN 'DRIFT_WARNING'
    ELSE 'OK'
  END as drift_status
FROM daily_stats
CROSS JOIN baseline;
```

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Data Drift | Distribution change detection |
| Statistical Monitoring | Z-score methodology |
| Baseline Comparison | Train vs inference |
| Threshold Design | Multi-level alerts |

---

## Navigation

- **EPIC**: [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md)
- **Previous**: [US-012: Batch Predictions](./US-012-batch-predictions.md)
- **Next**: [US-014: Alerting System](./US-014-alerting-system.md)
