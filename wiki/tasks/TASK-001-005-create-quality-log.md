# TASK-001-005: Create Data Quality Log

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-001-005 |
| User Story | [US-001: Data Profiling](../user-stories/US-001-data-profiling.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Create a data quality log table to track data profiling metrics over time for audit and monitoring purposes.

---

## Prerequisites

- [x] Dataset created (TASK-001-003)
- [x] Source data profiled (TASK-001-004)
- [x] Understanding of data quality metrics

---

## Step-by-Step Instructions

### Step 1: Create Quality Log Table

Execute in BigQuery Console:

```sql
CREATE TABLE IF NOT EXISTS `sonorous-key-320714.diamond_analytics.data_quality_log` (
  log_id STRING,
  log_timestamp TIMESTAMP,
  source_table STRING,
  metric_name STRING,
  metric_value FLOAT64,
  metric_unit STRING,
  notes STRING
);
```

**Column Definitions**:

| Column | Type | Purpose |
|--------|------|---------|
| log_id | STRING | Unique identifier (UUID) |
| log_timestamp | TIMESTAMP | When metric was recorded |
| source_table | STRING | Table being profiled |
| metric_name | STRING | Name of quality metric |
| metric_value | FLOAT64 | Numeric value |
| metric_unit | STRING | Unit of measurement |
| notes | STRING | Additional context |

### Step 2: Insert Profiling Results

Log the metrics from TASK-001-004:

```sql
INSERT INTO `sonorous-key-320714.diamond_analytics.data_quality_log`
SELECT
  GENERATE_UUID() as log_id,
  CURRENT_TIMESTAMP() as log_timestamp,
  'bigquery-public-data.chicago_taxi_trips.taxi_trips' as source_table,
  metric_name,
  metric_value,
  metric_unit,
  notes
FROM UNNEST([
  STRUCT('total_trips' as metric_name, 20713994.0 as metric_value, 'count' as metric_unit, 'Total records since 2020' as notes),
  STRUCT('unique_taxis', 5727.0, 'count', 'Distinct taxi_id values'),
  STRUCT('unique_companies', 64.0, 'count', 'Distinct company values'),
  STRUCT('avg_fare', 25.19, 'USD', 'Mean trip_total'),
  STRUCT('median_fare', 15.04, 'USD', 'P50 trip_total'),
  STRUCT('median_miles', 2.2, 'miles', 'P50 trip_miles'),
  STRUCT('median_duration', 13.3, 'minutes', 'P50 trip_seconds/60'),
  STRUCT('null_fare_pct', 0.08, 'percent', 'Missing trip_total'),
  STRUCT('invalid_miles_pct', 13.0, 'percent', 'trip_miles <= 0'),
  STRUCT('null_location_pct', 6.4, 'percent', 'Missing pickup coordinates')
]);
```

### Step 3: Verify Log Entries

```sql
SELECT
  metric_name,
  metric_value,
  metric_unit,
  notes
FROM `sonorous-key-320714.diamond_analytics.data_quality_log`
ORDER BY metric_name;
```

**Expected Output**:

| metric_name | metric_value | metric_unit | notes |
|-------------|--------------|-------------|-------|
| avg_fare | 25.19 | USD | Mean trip_total |
| invalid_miles_pct | 13.0 | percent | trip_miles <= 0 |
| median_duration | 13.3 | minutes | P50 trip_seconds/60 |
| median_fare | 15.04 | USD | P50 trip_total |
| median_miles | 2.2 | miles | P50 trip_miles |
| null_fare_pct | 0.08 | percent | Missing trip_total |
| null_location_pct | 6.4 | percent | Missing pickup coordinates |
| total_trips | 20713994.0 | count | Total records since 2020 |
| unique_companies | 64.0 | count | Distinct company values |
| unique_taxis | 5727.0 | count | Distinct taxi_id values |

### Step 4: Create Quality Summary View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.data_quality_summary` AS
SELECT
  source_table,
  MAX(log_timestamp) as last_profiled,
  COUNT(*) as metrics_logged,
  MAX(CASE WHEN metric_name = 'total_trips' THEN metric_value END) as total_records,
  MAX(CASE WHEN metric_name = 'null_fare_pct' THEN metric_value END) as null_fare_pct,
  MAX(CASE WHEN metric_name = 'invalid_miles_pct' THEN metric_value END) as invalid_miles_pct
FROM `sonorous-key-320714.diamond_analytics.data_quality_log`
GROUP BY source_table;
```

---

## Quality Metrics Reference

### Completeness Metrics

| Metric | Formula | Target |
|--------|---------|--------|
| Null Rate | COUNT(NULL) / COUNT(*) * 100 | < 5% |
| Completeness | 100 - Null Rate | > 95% |

### Validity Metrics

| Metric | Formula | Target |
|--------|---------|--------|
| Invalid Rate | COUNT(invalid) / COUNT(*) * 100 | < 10% |
| Valid Rate | 100 - Invalid Rate | > 90% |

### Consistency Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| Uniqueness | Distinct values / Total values | Varies |
| Referential Integrity | Foreign key matches | 100% |

---

## Data Quality Thresholds

| Category | Metric | Threshold | Status |
|----------|--------|-----------|--------|
| Fare | null_fare_pct | < 1% | PASS (0.08%) |
| Location | null_location_pct | < 10% | PASS (6.4%) |
| Distance | invalid_miles_pct | < 15% | PASS (13%) |

---

## Scheduled Profiling Query

For ongoing monitoring, schedule this query:

```sql
-- Run weekly to track data quality over time
INSERT INTO `sonorous-key-320714.diamond_analytics.data_quality_log`
WITH latest_stats AS (
  SELECT
    COUNT(*) as total_trips,
    COUNTIF(trip_total IS NULL) / COUNT(*) * 100 as null_fare_pct,
    COUNTIF(trip_miles <= 0) / COUNT(*) * 100 as invalid_miles_pct,
    COUNTIF(pickup_latitude IS NULL) / COUNT(*) * 100 as null_location_pct
  FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
  WHERE trip_start_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
)
SELECT
  GENERATE_UUID(),
  CURRENT_TIMESTAMP(),
  'bigquery-public-data.chicago_taxi_trips.taxi_trips',
  metric_name,
  metric_value,
  'percent',
  'Weekly automated profile'
FROM latest_stats
UNPIVOT(metric_value FOR metric_name IN (null_fare_pct, invalid_miles_pct, null_location_pct));
```

---

## Verification Checklist

| Check | Expected | Status |
|-------|----------|--------|
| Table created | No error | Complete |
| 10 metrics logged | COUNT(*) = 10 | Complete |
| Summary view works | Returns 1 row | Complete |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Duplicate log entries | Query run multiple times | Add DISTINCT or check before insert |
| Missing metrics | Insert failed | Check for NULL values in source |
| View error | Table not found | Verify table name spelling |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Data Quality Dimensions | Completeness, validity, consistency |
| Audit Logging | Tracking metrics over time |
| Views | Simplified data access |
| UNPIVOT | Row transformation |

---

## Next Task

[TASK-002-001: Define Cleaning Rules](./TASK-002-001-define-cleaning-rules.md)
