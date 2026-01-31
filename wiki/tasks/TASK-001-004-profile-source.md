# TASK-001-004: Profile Source Data

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-001-004 |
| User Story | [US-001: Data Profiling](../user-stories/US-001-data-profiling.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Execute a comprehensive profiling query against the Chicago Taxi public dataset to understand data volume, date range, statistical distributions, and data quality issues.

---

## Prerequisites

- [x] GCP authentication configured (TASK-001-001)
- [x] BigQuery API enabled (TASK-001-002)
- [x] Project dataset created (TASK-001-003)
- [x] Project ID: `sonorous-key-320714`

---

## Step-by-Step Instructions

### Step 1: Open BigQuery Console

**Option A: Web Console**
1. Navigate to: https://console.cloud.google.com/bigquery
2. Ensure project `sonorous-key-320714` is selected in the dropdown
3. Click "Compose New Query" button

**Option B: Command Line**
1. Open terminal
2. Use `bq query` command (shown in Step 2)

### Step 2: Execute Profiling Query

Copy and paste the following SQL into the query editor:

```sql
SELECT
  COUNT(*) as total_trips,
  COUNT(DISTINCT taxi_id) as unique_taxis,
  COUNT(DISTINCT company) as unique_companies,
  MIN(trip_start_timestamp) as earliest_trip,
  MAX(trip_start_timestamp) as latest_trip,
  ROUND(AVG(trip_total), 2) as avg_fare,
  ROUND(APPROX_QUANTILES(trip_miles, 100)[OFFSET(50)], 2) as median_miles,
  ROUND(APPROX_QUANTILES(trip_total, 100)[OFFSET(50)], 2) as median_fare,
  ROUND(APPROX_QUANTILES(trip_seconds, 100)[OFFSET(50)] / 60, 1) as median_duration_mins,
  COUNTIF(trip_total IS NULL) as null_fares,
  COUNTIF(trip_miles <= 0) as invalid_miles,
  COUNTIF(pickup_latitude IS NULL) as null_locations
FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
WHERE trip_start_timestamp >= '2020-01-01';
```

**Command Line Execution**:
```bash
bq query --use_legacy_sql=false --project_id=sonorous-key-320714 '
SELECT
  COUNT(*) as total_trips,
  COUNT(DISTINCT taxi_id) as unique_taxis,
  COUNT(DISTINCT company) as unique_companies,
  MIN(trip_start_timestamp) as earliest_trip,
  MAX(trip_start_timestamp) as latest_trip,
  ROUND(AVG(trip_total), 2) as avg_fare,
  ROUND(APPROX_QUANTILES(trip_miles, 100)[OFFSET(50)], 2) as median_miles,
  ROUND(APPROX_QUANTILES(trip_total, 100)[OFFSET(50)], 2) as median_fare,
  ROUND(APPROX_QUANTILES(trip_seconds, 100)[OFFSET(50)] / 60, 1) as median_duration_mins,
  COUNTIF(trip_total IS NULL) as null_fares,
  COUNTIF(trip_miles <= 0) as invalid_miles,
  COUNTIF(pickup_latitude IS NULL) as null_locations
FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
WHERE trip_start_timestamp >= "2020-01-01";
'
```

### Step 3: Understand the Query

| Line | Function | Purpose |
|------|----------|---------|
| `COUNT(*)` | Aggregate | Total trip count |
| `COUNT(DISTINCT taxi_id)` | Aggregate | Unique vehicles |
| `COUNT(DISTINCT company)` | Aggregate | Unique taxi companies |
| `MIN/MAX(trip_start_timestamp)` | Aggregate | Date range |
| `AVG(trip_total)` | Statistical | Mean fare amount |
| `APPROX_QUANTILES(...)[OFFSET(50)]` | Statistical | Median calculation |
| `COUNTIF(condition)` | Quality | Count matching condition |
| `WHERE trip_start_timestamp >= '2020-01-01'` | Filter | Recent data only |

### Step 4: Verify Results Structure

Expected output columns:
| Column | Type | Example |
|--------|------|---------|
| total_trips | INTEGER | 20,713,994 |
| unique_taxis | INTEGER | 5,727 |
| unique_companies | INTEGER | 64 |
| earliest_trip | TIMESTAMP | 2020-01-01 00:00:00 |
| latest_trip | TIMESTAMP | 2023-12-31 23:59:00 |
| avg_fare | FLOAT | 25.19 |
| median_miles | FLOAT | 2.2 |
| median_fare | FLOAT | 15.04 |
| median_duration_mins | FLOAT | 13.3 |
| null_fares | INTEGER | 16,609 |
| invalid_miles | INTEGER | 2,686,438 |
| null_locations | INTEGER | 1,336,117 |

### Step 5: Document Findings

Record the following in your notes:

**Volume Metrics**:
- Total Records: ___________
- Unique Taxis: ___________
- Unique Companies: ___________

**Date Range**:
- Earliest: ___________
- Latest: ___________

**Statistical Metrics**:
- Average Fare: $___________
- Median Fare: $___________
- Median Miles: ___________
- Median Duration: ___________ minutes

**Data Quality Issues**:
- Null Fares: ___________ (_____%)
- Invalid Miles (<=0): ___________ (_____%)
- Null Locations: ___________ (_____%)

---

## Expected Output

```
+-------------+--------------+------------------+---------------------+---------------------+----------+--------------+-------------+----------------------+------------+---------------+----------------+
| total_trips | unique_taxis | unique_companies |    earliest_trip    |     latest_trip     | avg_fare | median_miles | median_fare | median_duration_mins | null_fares | invalid_miles | null_locations |
+-------------+--------------+------------------+---------------------+---------------------+----------+--------------+-------------+----------------------+------------+---------------+----------------+
|    20713994 |         5727 |               64 | 2020-01-01 00:00:00 | 2023-12-31 23:45:00 |    25.19 |          2.2 |       15.04 |                 13.3 |      16609 |       2686438 |        1336117 |
+-------------+--------------+------------------+---------------------+---------------------+----------+--------------+-------------+----------------------+------------+---------------+----------------+
```

---

## Actual Results Recorded

| Metric | Value |
|--------|-------|
| Total Trips | 20,713,994 |
| Unique Taxis | 5,727 |
| Unique Companies | 64 |
| Date Range | 2020-01-01 to 2023-12-31 |
| Average Fare | $25.19 |
| Median Fare | $15.04 |
| Median Miles | 2.2 |
| Median Duration | 13.3 minutes |
| Null Fares | 16,609 (0.08%) |
| Invalid Miles (<=0) | 2,686,438 (13%) |
| Null Locations | 1,336,117 (6.4%) |

---

## Data Quality Observations

### Quality Assessment

| Dimension | Score | Notes |
|-----------|-------|-------|
| Fare Completeness | **99.92%** | Excellent |
| Location Completeness | **93.6%** | Good |
| Distance Validity | **87%** | 13% invalid/zero miles |
| Timestamp Validity | ~100% | Very few issues |

### Issues to Address in Cleaning

1. **Invalid Miles (13%)**: Filter out trips with `trip_miles <= 0`
2. **Null Locations (6.4%)**: Exclude from geospatial features
3. **Outliers**: Filter extreme fare amounts

---

## Query Cost

| Metric | Value |
|--------|-------|
| Bytes Processed | ~15 GB |
| Estimated Cost | ~$0.09 (at $6.25/TB) |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Access Denied" | No permission | Verify BigQuery API enabled |
| Query timeout | Large dataset | This query should complete in ~30 seconds |
| Unexpected NULLs | Date filter issue | Check date format matches |
| Zero results | Filter too restrictive | Remove WHERE clause to test |

---

## SQL Function Reference

| Function | Purpose | Example |
|----------|---------|---------|
| `COUNT(*)` | Row count | Total trips |
| `COUNT(DISTINCT col)` | Unique values | Unique taxis |
| `APPROX_QUANTILES(col, 100)[OFFSET(50)]` | Approximate median | Median fare |
| `COUNTIF(condition)` | Conditional count | Null fares |
| `ROUND(value, decimals)` | Round numeric | $25.19 |

---

## Certification Concepts

| Concept | How Demonstrated |
|---------|------------------|
| Data Profiling | Understanding dataset before modeling |
| SQL Aggregations | COUNT, AVG, MIN, MAX |
| Approximate Functions | APPROX_QUANTILES for efficiency |
| Data Quality Metrics | Completeness, validity |
| Public Datasets | Accessing bigquery-public-data |

---

## Next Task

[TASK-001-005: Create Data Quality Log](./TASK-001-005-create-quality-log.md)
