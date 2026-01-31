# US-002: Create Cleaned Base Table

## User Story

**As a** ML Engineer preparing for certification,
**I want to** create a cleaned, partitioned, and clustered base table,
**So that** I have high-quality data optimized for ML model training.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-002 |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |
| Tasks | 4 |

---

## Acceptance Criteria

- [x] Cleaning rules designed based on profiling
- [x] Table created with partitioning on date
- [x] Table created with clustering on company, payment_type
- [x] Derived columns added (pickup_hour, day_type, avg_speed)
- [x] Row count verified
- [x] Partition count verified

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-002-001](../tasks/TASK-002-001-design-rules.md) | Design Cleaning Rules | Complete |
| [TASK-002-002](../tasks/TASK-002-002-create-ddl.md) | Create Partitioned Table DDL | Complete |
| [TASK-002-003](../tasks/TASK-002-003-execute-creation.md) | Execute Table Creation | Complete |
| [TASK-002-004](../tasks/TASK-002-004-verify-table.md) | Verify Row Counts and Partitions | Complete |

---

## Cleaning Rules Applied

| Rule | Purpose | Rationale |
|------|---------|-----------|
| trip_total > 0 | Valid fares | Remove cancelled trips |
| trip_total < 500 | Reasonable fares | Remove data errors |
| trip_miles > 0 | Valid distance | Remove zero-distance trips |
| trip_miles < 100 | Reasonable distance | Remove implausible values |
| trip_seconds > 60 | Valid duration | Remove instant trips |
| trip_seconds < 14400 | Reasonable duration | Remove >4 hour trips |
| pickup_lat 41.6-42.1 | Chicago bounds | Geographic filtering |
| pickup_lng -88.0 to -87.5 | Chicago bounds | Geographic filtering |
| Date 2022-2024 | Recent data | Focus period |

---

## Actual Results

| Metric | Value |
|--------|-------|
| Input Records | 20,713,994 |
| Output Records | 10,598,441 |
| Records Removed | 10,115,553 (49%) |
| Date Range | 2022-01-01 to 2023-12-31 |
| Partitions Created | 730 |
| Total Revenue | $287.7M |

---

## Derived Columns

| Column | Calculation |
|--------|-------------|
| pickup_hour | EXTRACT(HOUR FROM trip_start_timestamp) |
| pickup_dow | EXTRACT(DAYOFWEEK FROM trip_start_timestamp) |
| pickup_month | EXTRACT(MONTH FROM trip_start_timestamp) |
| pickup_year | EXTRACT(YEAR FROM trip_start_timestamp) |
| day_type | CASE WHEN dow IN (1,7) THEN 'weekend' ELSE 'weekday' |
| avg_speed_mph | SAFE_DIVIDE(trip_miles, trip_seconds/3600) |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Table Partitioning | DATE partition for cost optimization |
| Table Clustering | Multi-column clustering for query performance |
| Data Cleaning | Business rule-based filtering |
| Derived Features | SQL transformations for new columns |

---

## Navigation

- **EPIC**: [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md)
- **Previous**: [US-001: Data Profiling](./US-001-data-profiling.md)
- **Next**: [US-003: Temporal Features](./US-003-temporal-features.md)
