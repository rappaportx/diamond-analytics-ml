# Data Engineering Documentation

## Overview

The Data Engineering phase transforms raw Chicago Taxi trip data into a clean, optimized base table suitable for ML model training. This phase demonstrates data pipeline skills critical for the ML certification exam.

---

## Phase Objectives

1. Profile source data to understand characteristics
2. Assess data quality and document issues
3. Apply business rules for data cleaning
4. Create partitioned and clustered table for cost optimization

---

## Source Data Profile

### Dataset Information
| Field | Value |
|-------|-------|
| Source Table | `bigquery-public-data.chicago_taxi_trips.taxi_trips` |
| Total Records (2020+) | 20,713,994 |
| Unique Taxis | 5,727 |
| Unique Companies | 64 |
| Date Range | 2020-01-01 to 2023-12-31 |

### Statistical Summary
| Metric | Value |
|--------|-------|
| Average Fare | $25.19 |
| Median Fare | $15.04 |
| Median Miles | 2.2 |
| Median Duration | 13.3 minutes |

### Data Quality Issues Identified
| Issue | Count | Percentage |
|-------|-------|------------|
| Null Fares | 16,609 | 0.08% |
| Invalid Miles (<=0) | 2,686,438 | 13% |
| Null Locations | 1,336,117 | 6.4% |
| Invalid Timestamps | 86 | <0.01% |

---

## Cleaning Rules

### Business Logic Applied

```sql
WHERE
  -- Date range filter
  trip_start_timestamp >= '2022-01-01'
  AND trip_start_timestamp < '2024-01-01'

  -- Valid fare amounts
  AND trip_total > 0
  AND trip_total < 500

  -- Valid trip distances
  AND trip_miles > 0
  AND trip_miles < 100

  -- Valid trip durations (1 min to 4 hours)
  AND trip_seconds > 60
  AND trip_seconds < 14400

  -- Chicago geographic bounds
  AND pickup_latitude BETWEEN 41.6 AND 42.1
  AND pickup_longitude BETWEEN -88.0 AND -87.5
```

### Rationale for Each Rule

| Rule | Rationale |
|------|-----------|
| trip_total > 0 | Remove cancelled/refunded trips |
| trip_total < 500 | Remove data entry errors |
| trip_miles > 0 | Remove trips with no distance |
| trip_miles < 100 | Remove implausible distances |
| trip_seconds > 60 | Remove instantaneous trips |
| trip_seconds < 14400 | Remove trips over 4 hours |
| pickup_lat/lng bounds | Chicago metro area only |

---

## Output Table: trips_cleaned

### Table Properties
| Property | Value |
|----------|-------|
| Table Name | `diamond_analytics.trips_cleaned` |
| Row Count | 10,598,441 |
| Partition Column | `DATE(trip_start_timestamp)` |
| Cluster Columns | `company`, `payment_type` |
| Number of Partitions | 730 |

### Schema
| Column | Type | Description |
|--------|------|-------------|
| unique_key | STRING | Trip identifier |
| taxi_id | STRING | Vehicle identifier |
| trip_start_timestamp | TIMESTAMP | Pickup time |
| trip_end_timestamp | TIMESTAMP | Dropoff time |
| trip_seconds | INT64 | Trip duration |
| trip_miles | FLOAT64 | Trip distance |
| pickup_community_area | INT64 | Pickup neighborhood |
| dropoff_community_area | INT64 | Dropoff neighborhood |
| fare | FLOAT64 | Base fare |
| tips | FLOAT64 | Tip amount |
| tolls | FLOAT64 | Toll charges |
| extras | FLOAT64 | Extra charges |
| trip_total | FLOAT64 | Total amount |
| payment_type | STRING | Payment method |
| company | STRING | Taxi company |
| pickup_latitude | FLOAT64 | Pickup lat |
| pickup_longitude | FLOAT64 | Pickup lng |
| dropoff_latitude | FLOAT64 | Dropoff lat |
| dropoff_longitude | FLOAT64 | Dropoff lng |
| pickup_hour | INT64 | Hour of pickup (0-23) |
| pickup_dow | INT64 | Day of week (1-7) |
| pickup_month | INT64 | Month (1-12) |
| pickup_year | INT64 | Year |
| day_type | STRING | 'weekend' or 'weekday' |
| avg_speed_mph | FLOAT64 | Average trip speed |

### Derived Column Calculations

```sql
-- Pickup time components
EXTRACT(HOUR FROM trip_start_timestamp) as pickup_hour,
EXTRACT(DAYOFWEEK FROM trip_start_timestamp) as pickup_dow,
EXTRACT(MONTH FROM trip_start_timestamp) as pickup_month,
EXTRACT(YEAR FROM trip_start_timestamp) as pickup_year,

-- Day type categorization
CASE
  WHEN EXTRACT(DAYOFWEEK FROM trip_start_timestamp) IN (1, 7)
  THEN 'weekend'
  ELSE 'weekday'
END as day_type,

-- Average speed calculation
SAFE_DIVIDE(trip_miles, SAFE_DIVIDE(trip_seconds, 3600)) as avg_speed_mph
```

---

## Data Quality Log

### Log Table Structure
| Column | Type | Description |
|--------|------|-------------|
| check_timestamp | TIMESTAMP | When check was run |
| table_name | STRING | Source table name |
| total_records | INT64 | Total row count |
| null_fares | INT64 | Count of null fares |
| negative_miles | INT64 | Count of negative miles |
| negative_duration | INT64 | Count of negative durations |
| invalid_timestamps | INT64 | Count of end < start |
| invalid_pickup_lat | INT64 | Out of bounds latitude |
| invalid_pickup_lng | INT64 | Out of bounds longitude |
| fare_completeness_pct | FLOAT64 | % of non-null fares |
| location_validity_pct | FLOAT64 | % valid locations |

### Quality Scores Recorded
| Metric | Value |
|--------|-------|
| Fare Completeness | 99.81% |
| Location Validity | 100% |
| Total Records | 6,495,415 (2023) |

---

## Optimization Techniques

### Partitioning Benefits
- **Cost Reduction**: Queries only scan relevant date partitions
- **Example**: A query for January 2023 scans ~1/24th of the data
- **Partition Pruning**: BigQuery automatically excludes irrelevant partitions

### Clustering Benefits
- **Query Speed**: Queries filtering by company or payment_type are faster
- **Storage Optimization**: Related data stored together
- **Example**: `WHERE company = 'Flash Cab'` benefits from clustering

### Best Practices Applied
1. **Partitioning on date column** - Most common filter dimension
2. **Clustering on categorical columns** - Frequently used in WHERE/GROUP BY
3. **Appropriate partition granularity** - Daily partitions for 2 years of data

---

## Certification Topics Demonstrated

| Topic | How Demonstrated |
|-------|------------------|
| Data Pipelines | ETL from public dataset to cleaned table |
| Data Profiling | Statistical analysis of source data |
| Data Quality | Quality log with metrics |
| BigQuery Optimization | Partitioning and clustering |
| SQL Best Practices | SAFE_DIVIDE, EXTRACT functions |

---

## Related Artifacts

### EPICs
- [EPIC-001: Data Engineering](./epics/EPIC-001-data-engineering.md)

### User Stories
- [US-001: Data Profiling & Quality Assessment](./user-stories/US-001-data-profiling.md)
- [US-002: Create Cleaned Base Table](./user-stories/US-002-data-cleaning.md)

### Tasks
- TASK-001-001 through TASK-002-004

---

## Navigation

- **Previous**: [Architecture](./02-architecture.md)
- **Next**: [Feature Engineering](./04-feature-engineering.md)
