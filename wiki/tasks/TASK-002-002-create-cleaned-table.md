# TASK-002-002: Create Cleaned Table

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-002-002 |
| User Story | [US-002: Data Cleaning](../user-stories/US-002-data-cleaning.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Create the `trips_cleaned` table applying all cleaning rules with partitioning and clustering for optimal query performance.

---

## Prerequisites

- [x] Cleaning rules defined (TASK-002-001)
- [x] Dataset created (TASK-001-003)
- [x] Understanding of partitioning and clustering

---

## Step-by-Step Instructions

### Step 1: Understand Table Design

**Partitioning Strategy**:

| Option | Granularity | Partition Count | Decision |
|--------|-------------|-----------------|----------|
| DAY | Daily | ~1,460 | Selected |
| MONTH | Monthly | ~48 | Alternative |
| YEAR | Yearly | ~4 | Too coarse |

**Clustering Columns**:

| Column | Cardinality | Query Pattern | Priority |
|--------|-------------|---------------|----------|
| company | 64 | Filter by company | 1 |
| payment_type | 5 | Filter by payment | 2 |
| pickup_community_area | 77 | Geographic analysis | 3 |

### Step 2: Create the Cleaned Table

Execute in BigQuery Console:

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.trips_cleaned`
PARTITION BY DATE(trip_start_timestamp)
CLUSTER BY company, payment_type, pickup_community_area
AS
SELECT
  -- Identifiers
  unique_key,
  taxi_id,

  -- Timestamps
  trip_start_timestamp,
  trip_end_timestamp,

  -- Trip metrics
  trip_seconds,
  trip_miles,

  -- Location - Pickup
  pickup_community_area,
  pickup_centroid_latitude as pickup_latitude,
  pickup_centroid_longitude as pickup_longitude,

  -- Location - Dropoff
  dropoff_community_area,
  dropoff_centroid_latitude as dropoff_latitude,
  dropoff_centroid_longitude as dropoff_longitude,

  -- Fare components
  fare,
  tips,
  tolls,
  extras,
  trip_total,

  -- Categorical
  payment_type,
  company

FROM `bigquery-public-data.chicago_taxi_trips.taxi_trips`

WHERE
  -- R001: Valid Fare ($0 < fare < $500)
  trip_total > 0 AND trip_total < 500

  -- R002: Valid Distance (0 < miles < 100)
  AND trip_miles > 0 AND trip_miles < 100

  -- R003: Valid Duration (1 min < time < 4 hours)
  AND trip_seconds > 60 AND trip_seconds < 14400

  -- R004: Valid Coordinates (all present)
  AND pickup_centroid_latitude IS NOT NULL
  AND pickup_centroid_longitude IS NOT NULL
  AND dropoff_centroid_latitude IS NOT NULL
  AND dropoff_centroid_longitude IS NOT NULL

  -- R005: Valid Taxi ID
  AND taxi_id IS NOT NULL

  -- R006: Date Range (2020-2023)
  AND trip_start_timestamp >= '2020-01-01'
  AND trip_start_timestamp < '2024-01-01';
```

**Expected Execution**:
- Processing: ~15-20 GB
- Duration: ~30-60 seconds
- Cost: ~$0.10

### Step 3: Verify Row Count

```sql
SELECT
  COUNT(*) as row_count,
  MIN(trip_start_timestamp) as earliest_trip,
  MAX(trip_start_timestamp) as latest_trip
FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`;
```

**Expected Output**:

| row_count | earliest_trip | latest_trip |
|-----------|---------------|-------------|
| 10,598,441 | 2020-01-01 00:00:00 | 2023-12-31 23:59:00 |

### Step 4: Verify Partitioning

```sql
SELECT
  partition_id,
  total_rows,
  total_logical_bytes / 1024 / 1024 as size_mb
FROM `sonorous-key-320714.diamond_analytics.INFORMATION_SCHEMA.PARTITIONS`
WHERE table_name = 'trips_cleaned'
ORDER BY partition_id DESC
LIMIT 10;
```

### Step 5: Verify Clustering

```sql
SELECT
  table_name,
  clustering_ordinal_position,
  column_name
FROM `sonorous-key-320714.diamond_analytics.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'trips_cleaned'
  AND clustering_ordinal_position IS NOT NULL
ORDER BY clustering_ordinal_position;
```

**Expected Output**:

| table_name | clustering_ordinal_position | column_name |
|------------|----------------------------|-------------|
| trips_cleaned | 1 | company |
| trips_cleaned | 2 | payment_type |
| trips_cleaned | 3 | pickup_community_area |

### Step 6: Test Partition Pruning

```sql
SELECT COUNT(*)
FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
WHERE trip_start_timestamp >= '2023-01-01'
  AND trip_start_timestamp < '2023-02-01';
```

Check the "Bytes processed" in query results - should be much less than full table scan.

---

## Table Schema Reference

| Column | Type | Source | Purpose |
|--------|------|--------|---------|
| unique_key | STRING | Original | Primary identifier |
| taxi_id | STRING | Original | Vehicle identifier |
| trip_start_timestamp | TIMESTAMP | Original | Partition key |
| trip_end_timestamp | TIMESTAMP | Original | Trip completion |
| trip_seconds | INTEGER | Original | Duration feature |
| trip_miles | FLOAT64 | Original | Distance feature |
| pickup_community_area | INTEGER | Original | Geographic feature |
| pickup_latitude | FLOAT64 | Renamed | Geospatial |
| pickup_longitude | FLOAT64 | Renamed | Geospatial |
| dropoff_community_area | INTEGER | Original | Geographic feature |
| dropoff_latitude | FLOAT64 | Renamed | Geospatial |
| dropoff_longitude | FLOAT64 | Renamed | Geospatial |
| fare | FLOAT64 | Original | Base fare |
| tips | FLOAT64 | Original | Tip amount |
| tolls | FLOAT64 | Original | Toll charges |
| extras | FLOAT64 | Original | Extra charges |
| trip_total | FLOAT64 | Original | Target variable |
| payment_type | STRING | Original | Categorical |
| company | STRING | Original | Categorical, cluster key |

---

## Partitioning Benefits

| Benefit | Description |
|---------|-------------|
| Query cost reduction | Only scan relevant date ranges |
| Faster queries | Less data to process |
| Easier maintenance | Drop old partitions easily |
| Cost optimization | Partition-level operations |

**Example Savings**:
- Full table scan: 10.5M rows (~2 GB)
- One month partition: ~270K rows (~50 MB)
- Savings: 97% cost reduction for monthly queries

---

## Clustering Benefits

| Benefit | Description |
|---------|-------------|
| Sorted storage | Data physically organized |
| Faster filters | Skip irrelevant blocks |
| No partition limits | Works with any cardinality |
| Automatic maintenance | BigQuery re-clusters as needed |

---

## Verification Checklist

| Check | Expected | Actual |
|-------|----------|--------|
| Row count | ~10.6M | 10,598,441 |
| Date range | 2020-2023 | Verified |
| Partitions exist | Daily partitions | Verified |
| Clustering active | 3 columns | Verified |
| No NULL fares | COUNT(NULL) = 0 | Verified |

---

## Cost Summary

| Metric | Value |
|--------|-------|
| Table size | ~2.1 GB |
| Storage cost | ~$0.04/month |
| Query cost (full scan) | ~$0.01 |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Table already exists" | Previous run | Use CREATE OR REPLACE |
| Low row count | Filters too strict | Review cleaning rules |
| No partitions showing | Metadata lag | Wait 1-2 minutes, refresh |
| Query not using partitions | Missing date filter | Add date predicate to WHERE |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Table Partitioning | Time-based data organization |
| Clustering | Multi-column sort optimization |
| Cost Optimization | Reducing query costs |
| Data Transformation | ETL patterns |

---

## Next Task

[TASK-002-003: Verify Cleaned Data](./TASK-002-003-verify-cleaned-data.md)
