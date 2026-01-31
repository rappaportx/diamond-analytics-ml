# EPIC-002: Feature Engineering

## Overview

| Field | Value |
|-------|-------|
| EPIC ID | EPIC-002 |
| Title | Feature Engineering |
| Status | Complete |
| User Stories | 3 |
| Total Tasks | 14 |

**Goal**: Transform cleaned trip data into ML-ready features using advanced engineering techniques including cyclical encoding and geospatial calculations.

**Business Value**: Creates high-quality features that capture temporal and spatial patterns essential for accurate fare prediction.

---

## Scope

- Temporal feature engineering with cyclical encoding
- Geospatial feature engineering with Haversine distance
- Unified feature store design
- NULL handling and imputation

## Out of Scope

- Real-time feature computation
- External feature sources (weather, events)
- Feature versioning

---

## User Stories

| ID | Title | Tasks | Status |
|----|-------|-------|--------|
| [US-003](../user-stories/US-003-temporal-features.md) | Temporal Feature Engineering | 5 | Complete |
| [US-004](../user-stories/US-004-geospatial-features.md) | Geospatial Feature Engineering | 5 | Complete |
| [US-005](../user-stories/US-005-feature-store.md) | Unified Feature Store | 4 | Complete |

---

## Acceptance Criteria

- [x] Cyclical encoding for hour, day of week, month
- [x] Binary flags for rush hours, weekends, seasons
- [x] Haversine distance calculation
- [x] Airport and downtown location flags
- [x] Feature store with 100% feature completeness
- [x] No NULL values in final features

---

## Technical Specifications

### Temporal Features (14)

| Feature | Type | Encoding |
|---------|------|----------|
| hour_sin | FLOAT64 | SIN(2π × hour/24) |
| hour_cos | FLOAT64 | COS(2π × hour/24) |
| dow_sin | FLOAT64 | SIN(2π × dow/7) |
| dow_cos | FLOAT64 | COS(2π × dow/7) |
| month_sin | FLOAT64 | SIN(2π × month/12) |
| month_cos | FLOAT64 | COS(2π × month/12) |
| doy_sin | FLOAT64 | SIN(2π × doy/365) |
| doy_cos | FLOAT64 | COS(2π × doy/365) |
| is_weekend | INT64 | Binary (0/1) |
| is_morning_rush | INT64 | Binary (0/1) |
| is_evening_rush | INT64 | Binary (0/1) |
| is_late_night | INT64 | Binary (0/1) |
| is_summer | INT64 | Binary (0/1) |
| is_winter | INT64 | Binary (0/1) |

### Geospatial Features (7)

| Feature | Type | Calculation |
|---------|------|-------------|
| straight_line_km | FLOAT64 | ST_DISTANCE / 1000 |
| route_circuity | FLOAT64 | actual_miles / straight_line |
| pickup_grid_cell | STRING | lat_floor × lng_floor |
| dropoff_grid_cell | STRING | lat_floor × lng_floor |
| is_airport_pickup | INT64 | area IN (76, 56) |
| is_airport_dropoff | INT64 | area IN (76, 56) |
| is_downtown_pickup | INT64 | area IN (32, 8, 33) |
| is_downtown_dropoff | INT64 | area IN (32, 8, 33) |
| same_area_trip | INT64 | pickup = dropoff |

---

## Deliverables

| Deliverable | Type | Location |
|-------------|------|----------|
| features_temporal | Table | `diamond_analytics.features_temporal` |
| features_geospatial | Table | `diamond_analytics.features_geospatial` |
| feature_store | Table | `diamond_analytics.feature_store` |

---

## Metrics Achieved

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Total Features | >20 | 31 | Exceeded |
| Feature Completeness | 100% | 100% | Met |
| NULL Values | 0 | 0 | Met |

---

## Related Certification Topics

| Topic | Relevance |
|-------|-----------|
| Feature Engineering | Cyclical, geospatial encoding |
| Feature Stores | Unified feature table |
| Data Transformation | SQL-based transformations |
| Domain Knowledge | Chicago geography |

---

## Dependencies

### Upstream
- [EPIC-001: Data Engineering](./EPIC-001-data-engineering.md)
- trips_cleaned table

### Downstream
- [EPIC-003: Model Development](./EPIC-003-model-development.md)

---

## Navigation

- **Previous EPIC**: [EPIC-001: Data Engineering](./EPIC-001-data-engineering.md)
- **Next EPIC**: [EPIC-003: Model Development](./EPIC-003-model-development.md)
- **Back to**: [Wiki Home](../README.md)
