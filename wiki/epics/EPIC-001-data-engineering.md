# EPIC-001: Data Engineering

## Overview

| Field | Value |
|-------|-------|
| EPIC ID | EPIC-001 |
| Title | Data Engineering |
| Status | Complete |
| User Stories | 2 |
| Total Tasks | 9 |

**Goal**: Transform raw Chicago Taxi data into a clean, partitioned, and clustered base table optimized for ML training.

**Business Value**: Establishes data quality foundation and cost-optimized storage for all downstream ML workflows.

---

## Scope

- Data profiling and quality assessment
- Data cleaning with business rules
- Table partitioning and clustering
- Data quality logging

## Out of Scope

- Real-time data ingestion
- External data sources
- Data encryption at rest

---

## User Stories

| ID | Title | Tasks | Status |
|----|-------|-------|--------|
| [US-001](../user-stories/US-001-data-profiling.md) | Data Profiling & Quality Assessment | 5 | Complete |
| [US-002](../user-stories/US-002-data-cleaning.md) | Create Cleaned Base Table | 4 | Complete |

---

## Acceptance Criteria

- [x] Data quality score > 95%
- [x] Table partitioned by date
- [x] Table clustered by company, payment_type
- [x] All records have valid lat/long within Chicago bounds
- [x] Date range: 2022-01-01 to 2023-12-31

---

## Technical Specifications

### Source Data
| Property | Value |
|----------|-------|
| Source Table | `bigquery-public-data.chicago_taxi_trips.taxi_trips` |
| Source Records | 20,713,994 (2020+) |
| Source Schema | 23 columns |

### Target Data
| Property | Value |
|----------|-------|
| Target Table | `diamond_analytics.trips_cleaned` |
| Target Records | 10,598,441 |
| Target Schema | 26 columns (23 + 3 derived) |
| Partition Column | `DATE(trip_start_timestamp)` |
| Cluster Columns | `company`, `payment_type` |

### Cleaning Rules
| Rule | Purpose |
|------|---------|
| trip_total > 0 AND < 500 | Valid fare amounts |
| trip_miles > 0 AND < 100 | Valid distances |
| trip_seconds > 60 AND < 14400 | Valid durations |
| pickup_lat BETWEEN 41.6 AND 42.1 | Chicago bounds |
| pickup_lng BETWEEN -88.0 AND -87.5 | Chicago bounds |

---

## Deliverables

| Deliverable | Type | Location |
|-------------|------|----------|
| trips_cleaned | Table | `diamond_analytics.trips_cleaned` |
| data_quality_log | Table | `diamond_analytics.data_quality_log` |

---

## Metrics Achieved

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Data Quality Score | >95% | 99.81% | Exceeded |
| Partitions Created | >300 | 730 | Exceeded |
| Records Cleaned | >1M | 10.6M | Exceeded |

---

## Related Certification Topics

| Topic | Relevance |
|-------|-----------|
| Data Pipelines | ETL from raw to clean |
| BigQuery Optimization | Partitioning, clustering |
| Data Quality | Profiling, validation |
| SQL Best Practices | SAFE_DIVIDE, EXTRACT |

---

## Dependencies

### Upstream
- GCP Project authenticated
- BigQuery API enabled
- Public dataset accessible

### Downstream
- [EPIC-002: Feature Engineering](./EPIC-002-feature-engineering.md)

---

## Navigation

- **Back to**: [Wiki Home](../README.md)
- **Next EPIC**: [EPIC-002: Feature Engineering](./EPIC-002-feature-engineering.md)
