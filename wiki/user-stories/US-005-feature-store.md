# US-005: Unified Feature Store

## User Story

**As a** ML Engineer preparing for certification,
**I want to** create a unified feature store combining all features,
**So that** I have a single source of truth for model training and serving.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-005 |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |
| Tasks | 4 |

---

## Acceptance Criteria

- [x] All temporal features included
- [x] All geospatial features included
- [x] Target variable (trip_total) included
- [x] NULL values handled with COALESCE
- [x] Table partitioned by date
- [x] Table clustered by pickup_community_area
- [x] 100% feature completeness verified

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-005-001](../tasks/TASK-005-001-schema-design.md) | Design Feature Store Schema | Complete |
| [TASK-005-002](../tasks/TASK-005-002-join-features.md) | Join Temporal and Geospatial Features | Complete |
| [TASK-005-003](../tasks/TASK-005-003-null-handling.md) | Handle NULL Values with COALESCE | Complete |
| [TASK-005-004](../tasks/TASK-005-004-verify-completeness.md) | Verify Feature Completeness | Complete |

---

## Feature Store Schema

### Identifiers
| Column | Type | Source |
|--------|------|--------|
| unique_key | STRING | trips_cleaned |
| trip_start_timestamp | TIMESTAMP | trips_cleaned |

### Target Variable
| Column | Type | Source |
|--------|------|--------|
| target_fare | FLOAT64 | trip_total |

### Raw Features (10)
| Column | Type | Source |
|--------|------|--------|
| trip_miles | FLOAT64 | trips_cleaned |
| trip_seconds | INT64 | trips_cleaned |
| fare | FLOAT64 | trips_cleaned |
| tips | FLOAT64 | trips_cleaned |
| payment_type | STRING | trips_cleaned |
| company | STRING | trips_cleaned |
| pickup_community_area | INT64 | trips_cleaned |
| dropoff_community_area | INT64 | trips_cleaned |
| avg_speed_mph | FLOAT64 | trips_cleaned |

### Temporal Features (12)
| Column | Default | Source |
|--------|---------|--------|
| hour_sin | 0 | features_temporal |
| hour_cos | 1 | features_temporal |
| dow_sin | 0 | features_temporal |
| dow_cos | 1 | features_temporal |
| month_sin | 0 | features_temporal |
| month_cos | 1 | features_temporal |
| is_weekend | 0 | features_temporal |
| is_morning_rush | 0 | features_temporal |
| is_evening_rush | 0 | features_temporal |
| is_late_night | 0 | features_temporal |
| is_summer | 0 | features_temporal |
| is_winter | 0 | features_temporal |

### Geospatial Features (7)
| Column | Default | Source |
|--------|---------|--------|
| straight_line_km | trip_miles×1.6 | features_geospatial |
| route_circuity | 1.3 | features_geospatial |
| is_airport_pickup | 0 | features_geospatial |
| is_airport_dropoff | 0 | features_geospatial |
| is_downtown_pickup | 0 | features_geospatial |
| is_downtown_dropoff | 0 | features_geospatial |
| same_area_trip | 0 | features_geospatial |

### Metadata (1)
| Column | Type | Source |
|--------|------|--------|
| feature_created_at | TIMESTAMP | CURRENT_TIMESTAMP() |

---

## NULL Handling Strategy

| Feature Type | Default Value | Rationale |
|--------------|---------------|-----------|
| Cyclical sin | 0 | Neutral midpoint |
| Cyclical cos | 1 | Neutral midpoint |
| Binary flags | 0 | Conservative default |
| Distance km | trip_miles × 1.6 | Approximate conversion |
| Circuity | 1.3 | Typical urban value |

---

## Actual Results

| Metric | Value |
|--------|-------|
| Total Rows | 10,598,441 |
| Total Features | 31 |
| Date Coverage | 730 days |
| Temporal Populated | 100% |
| Geospatial Populated | 100% |
| Average Target | $27.15 |
| Std Dev Target | $21.87 |

---

## Feature Store Benefits

| Benefit | Description |
|---------|-------------|
| Single Source | One table for all models |
| Consistency | Same features in train/serve |
| Efficiency | Pre-computed features |
| Maintainability | Centralized updates |
| Versioning | Timestamp metadata |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Feature Stores | Centralized feature management |
| JOIN Operations | Combining multiple feature tables |
| NULL Handling | COALESCE with appropriate defaults |
| Schema Design | Logical column organization |

---

## Navigation

- **EPIC**: [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md)
- **Previous**: [US-004: Geospatial Features](./US-004-geospatial-features.md)
- **Next**: [US-006: Train/Test Split](./US-006-train-test-split.md)
