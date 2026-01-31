# TASK-002-001: Define Cleaning Rules

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-002-001 |
| User Story | [US-002: Data Cleaning](../user-stories/US-002-data-cleaning.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Define comprehensive data cleaning rules based on profiling results to prepare high-quality training data.

---

## Prerequisites

- [x] Data profiling complete (TASK-001-004)
- [x] Quality metrics logged (TASK-001-005)
- [x] Understanding of ML data requirements

---

## Step-by-Step Instructions

### Step 1: Analyze Profiling Results

Review the data quality issues identified:

| Issue | Percentage | Impact | Decision |
|-------|------------|--------|----------|
| Null fares | 0.08% | Low | Exclude |
| Invalid miles (<=0) | 13% | High | Exclude |
| Null locations | 6.4% | Medium | Exclude |
| Extreme outliers | ~1% | Medium | Exclude |

### Step 2: Define Cleaning Rules

Document each cleaning rule with rationale:

#### Rule 1: Valid Fare Amount

```sql
trip_total > 0 AND trip_total < 500
```

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| Minimum fare | > $0 | Exclude free rides, errors |
| Maximum fare | < $500 | Exclude outliers (99.9th percentile) |

**Records Affected**: ~0.1% excluded

#### Rule 2: Valid Trip Distance

```sql
trip_miles > 0 AND trip_miles < 100
```

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| Minimum distance | > 0 miles | Exclude cancelled trips |
| Maximum distance | < 100 miles | Chicago metro area limit |

**Records Affected**: ~13% excluded

#### Rule 3: Valid Trip Duration

```sql
trip_seconds > 60 AND trip_seconds < 14400
```

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| Minimum duration | > 60 seconds | Exclude immediate cancellations |
| Maximum duration | < 4 hours | Exclude abandoned meters |

**Records Affected**: ~2% excluded

#### Rule 4: Valid Coordinates

```sql
pickup_latitude IS NOT NULL
AND pickup_longitude IS NOT NULL
AND dropoff_latitude IS NOT NULL
AND dropoff_longitude IS NOT NULL
```

**Records Affected**: ~6.4% excluded

#### Rule 5: Valid Taxi ID

```sql
taxi_id IS NOT NULL
```

**Records Affected**: < 0.1% excluded

#### Rule 6: Date Range Filter

```sql
trip_start_timestamp >= '2020-01-01'
AND trip_start_timestamp < '2024-01-01'
```

| Criterion | Value | Rationale |
|-----------|-------|-----------|
| Start date | 2020-01-01 | Recent data, post-ride-share impact |
| End date | 2024-01-01 | Data availability limit |

### Step 3: Create Rules Documentation Table

```sql
CREATE TABLE IF NOT EXISTS `sonorous-key-320714.diamond_analytics.cleaning_rules` (
  rule_id STRING,
  rule_name STRING,
  sql_condition STRING,
  rationale STRING,
  estimated_exclusion_pct FLOAT64,
  created_date DATE
);

INSERT INTO `sonorous-key-320714.diamond_analytics.cleaning_rules`
VALUES
  ('R001', 'Valid Fare', 'trip_total > 0 AND trip_total < 500', 'Exclude zero/negative and extreme outliers', 0.1, CURRENT_DATE()),
  ('R002', 'Valid Distance', 'trip_miles > 0 AND trip_miles < 100', 'Exclude cancelled and unrealistic distances', 13.0, CURRENT_DATE()),
  ('R003', 'Valid Duration', 'trip_seconds > 60 AND trip_seconds < 14400', 'Exclude immediate cancellations and stuck meters', 2.0, CURRENT_DATE()),
  ('R004', 'Valid Coordinates', 'pickup_latitude IS NOT NULL AND pickup_longitude IS NOT NULL AND dropoff_latitude IS NOT NULL AND dropoff_longitude IS NOT NULL', 'Require complete geospatial data', 6.4, CURRENT_DATE()),
  ('R005', 'Valid Taxi ID', 'taxi_id IS NOT NULL', 'Require vehicle identification', 0.05, CURRENT_DATE()),
  ('R006', 'Date Range', 'trip_start_timestamp >= 2020-01-01 AND trip_start_timestamp < 2024-01-01', 'Recent data window', 0.0, CURRENT_DATE());
```

### Step 4: Calculate Combined Impact

```sql
SELECT
  COUNT(*) as total_records,
  COUNTIF(
    trip_total > 0 AND trip_total < 500
    AND trip_miles > 0 AND trip_miles < 100
    AND trip_seconds > 60 AND trip_seconds < 14400
    AND pickup_latitude IS NOT NULL
    AND pickup_longitude IS NOT NULL
    AND dropoff_latitude IS NOT NULL
    AND dropoff_longitude IS NOT NULL
    AND taxi_id IS NOT NULL
  ) as valid_records,
  ROUND(COUNTIF(
    trip_total > 0 AND trip_total < 500
    AND trip_miles > 0 AND trip_miles < 100
    AND trip_seconds > 60 AND trip_seconds < 14400
    AND pickup_latitude IS NOT NULL
    AND pickup_longitude IS NOT NULL
    AND dropoff_latitude IS NOT NULL
    AND dropoff_longitude IS NOT NULL
    AND taxi_id IS NOT NULL
  ) / COUNT(*) * 100, 2) as retention_pct
FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`
WHERE trip_start_timestamp >= '2020-01-01'
  AND trip_start_timestamp < '2024-01-01';
```

**Expected Results**:

| Metric | Value |
|--------|-------|
| Total Records | ~20.7M |
| Valid Records | ~10.6M |
| Retention Rate | ~51% |

---

## Cleaning Rules Summary

| Rule | Condition | Priority |
|------|-----------|----------|
| R001 | Valid fare: $0 < fare < $500 | High |
| R002 | Valid distance: 0 < miles < 100 | High |
| R003 | Valid duration: 60s < time < 4hrs | Medium |
| R004 | Complete coordinates | High |
| R005 | Valid taxi ID | Medium |
| R006 | Date range: 2020-2023 | High |

---

## Combined WHERE Clause

Use this in the cleaning query:

```sql
WHERE
  -- R001: Valid Fare
  trip_total > 0 AND trip_total < 500
  -- R002: Valid Distance
  AND trip_miles > 0 AND trip_miles < 100
  -- R003: Valid Duration
  AND trip_seconds > 60 AND trip_seconds < 14400
  -- R004: Valid Coordinates
  AND pickup_latitude IS NOT NULL
  AND pickup_longitude IS NOT NULL
  AND dropoff_latitude IS NOT NULL
  AND dropoff_longitude IS NOT NULL
  -- R005: Valid Taxi ID
  AND taxi_id IS NOT NULL
  -- R006: Date Range
  AND trip_start_timestamp >= '2020-01-01'
  AND trip_start_timestamp < '2024-01-01'
```

---

## Quality Trade-offs

| Decision | Trade-off |
|----------|-----------|
| Strict filtering | Fewer records but higher quality |
| Include edge cases | More records but noisy data |
| Our approach | Balance: 51% retention with clean data |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Data Cleaning | Rule-based filtering |
| Outlier Detection | Threshold-based exclusion |
| Data Quality | Completeness requirements |
| Documentation | Audit trail for decisions |

---

## Next Task

[TASK-002-002: Create Cleaned Table](./TASK-002-002-create-cleaned-table.md)
