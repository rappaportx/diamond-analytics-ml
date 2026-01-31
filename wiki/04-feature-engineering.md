# Feature Engineering Documentation

## Overview

Feature Engineering transforms raw trip data into ML-ready features. This phase demonstrates advanced feature engineering techniques including cyclical encoding, geospatial calculations, and feature store design.

---

## Feature Categories

| Category | Features | Purpose |
|----------|----------|---------|
| Temporal | 14 | Capture time patterns |
| Geospatial | 7 | Capture location patterns |
| Trip Characteristics | 10 | Basic trip attributes |
| **Total** | **31** | Complete feature set |

---

## Temporal Features

### Cyclical Encoding

**Certification Concept**: Cyclical encoding prevents discontinuity at time boundaries. For example, hour 23 should be "close" to hour 0, but raw values (23 vs 0) suggest they're far apart.

**Mathematical Formula**:
```
sin_value = SIN(2 * π * value / period)
cos_value = COS(2 * π * value / period)
```

### Implementation

```sql
-- Hour encoding (period = 24)
SIN(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24) as hour_sin,
COS(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24) as hour_cos,

-- Day of week encoding (period = 7)
SIN(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7) as dow_sin,
COS(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7) as dow_cos,

-- Month encoding (period = 12)
SIN(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12) as month_sin,
COS(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12) as month_cos,

-- Day of year encoding (period = 365)
SIN(2 * ACOS(-1) * EXTRACT(DAYOFYEAR FROM trip_start_timestamp) / 365) as doy_sin,
COS(2 * ACOS(-1) * EXTRACT(DAYOFYEAR FROM trip_start_timestamp) / 365) as doy_cos
```

### Why SIN and COS Together?
Using both sine and cosine creates a unique (x, y) coordinate for each time value:
- Midnight (hour 0): sin=0, cos=1
- 6 AM (hour 6): sin=1, cos=0
- Noon (hour 12): sin=0, cos=-1
- 6 PM (hour 18): sin=-1, cos=0

This ensures that:
- 11 PM and 1 AM are close (both near hour 0)
- 6 AM and 6 PM are opposite
- Each hour has a unique combination

### Binary Time Flags

```sql
-- Weekend indicator
CASE WHEN EXTRACT(DAYOFWEEK FROM trip_start_timestamp) IN (1, 7)
     THEN 1 ELSE 0 END as is_weekend,

-- Rush hour indicators
CASE WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 7 AND 9
     THEN 1 ELSE 0 END as is_morning_rush,
CASE WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 16 AND 19
     THEN 1 ELSE 0 END as is_evening_rush,

-- Time of day indicators
CASE WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 0 AND 5
     THEN 1 ELSE 0 END as is_late_night,
CASE WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 10 AND 15
     THEN 1 ELSE 0 END as is_midday,

-- Season indicators
CASE WHEN EXTRACT(MONTH FROM trip_start_timestamp) IN (6, 7, 8)
     THEN 1 ELSE 0 END as is_summer,
CASE WHEN EXTRACT(MONTH FROM trip_start_timestamp) IN (12, 1, 2)
     THEN 1 ELSE 0 END as is_winter
```

---

## Geospatial Features

### Haversine Distance (Straight Line)

**Certification Concept**: Haversine formula calculates great-circle distance between two points on a sphere (Earth).

```sql
-- Using BigQuery GIS functions
ST_DISTANCE(
  ST_GEOGPOINT(pickup_longitude, pickup_latitude),
  ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
) / 1000 as straight_line_km
```

### Route Circuity

**Certification Concept**: Circuity ratio measures how much longer the actual route is compared to straight-line distance. Higher values indicate more indirect routes.

```sql
SAFE_DIVIDE(
  trip_miles * 1.60934,  -- Convert miles to km
  ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
  ) / 1000
) as route_circuity
```

**Interpretation**:
- Circuity = 1.0: Direct route (impossible in practice)
- Circuity = 1.2-1.4: Typical urban route
- Circuity > 2.0: Very indirect route (possible anomaly)

### Grid-Based Location Encoding

**Certification Concept**: Similar to H3 or Geohash, grid cells group nearby locations.

```sql
CONCAT(
  CAST(FLOOR(pickup_latitude * 100) AS STRING), '_',
  CAST(FLOOR(pickup_longitude * 100) AS STRING)
) as pickup_grid_cell
```

This creates ~0.01 degree cells (approximately 1.1 km x 0.9 km in Chicago).

### Location Category Flags

```sql
-- Airport flags (O'Hare = 76, Midway = 56)
CASE WHEN pickup_community_area IN (76, 56) THEN 1 ELSE 0 END as is_airport_pickup,
CASE WHEN dropoff_community_area IN (76, 56) THEN 1 ELSE 0 END as is_airport_dropoff,

-- Downtown flags (Loop = 32, Near North = 8, Near South = 33)
CASE WHEN pickup_community_area IN (32, 8, 33) THEN 1 ELSE 0 END as is_downtown_pickup,
CASE WHEN dropoff_community_area IN (32, 8, 33) THEN 1 ELSE 0 END as is_downtown_dropoff,

-- Same area trip
CASE WHEN pickup_community_area = dropoff_community_area THEN 1 ELSE 0 END as same_area_trip
```

### Chicago Community Areas Reference
| Area ID | Name | Significance |
|---------|------|--------------|
| 8 | Near North Side | High-traffic downtown |
| 32 | Loop | Central business district |
| 33 | Near South Side | Downtown fringe |
| 56 | Midway Airport | Airport traffic |
| 76 | O'Hare Airport | Major airport |

---

## Feature Store Design

### Purpose
A feature store centralizes features for:
- **Consistency**: Same features used in training and serving
- **Reusability**: Features available for multiple models
- **Efficiency**: Pre-computed features avoid repeated calculations

### Schema Design

```sql
CREATE TABLE feature_store
PARTITION BY DATE(trip_start_timestamp)
CLUSTER BY pickup_community_area
AS
SELECT
  -- Identifiers
  unique_key,
  trip_start_timestamp,

  -- Target variable
  trip_total as target_fare,

  -- Raw features
  trip_miles,
  trip_seconds,
  fare,
  tips,
  payment_type,
  company,
  pickup_community_area,
  dropoff_community_area,
  avg_speed_mph,

  -- Temporal features (with COALESCE for NULL handling)
  COALESCE(hour_sin, 0) as hour_sin,
  COALESCE(hour_cos, 1) as hour_cos,
  -- ... more temporal features

  -- Geospatial features (with COALESCE for NULL handling)
  COALESCE(straight_line_km, trip_miles * 1.6) as straight_line_km,
  COALESCE(route_circuity, 1.3) as route_circuity,
  -- ... more geospatial features

  -- Metadata
  CURRENT_TIMESTAMP() as feature_created_at
FROM trips_cleaned t
LEFT JOIN features_temporal tmp ON t.unique_key = tmp.unique_key
LEFT JOIN features_geospatial geo ON t.unique_key = geo.unique_key;
```

### NULL Handling Strategy

| Feature | Default Value | Rationale |
|---------|---------------|-----------|
| hour_sin | 0 | Neutral cyclical value |
| hour_cos | 1 | Neutral cyclical value |
| straight_line_km | trip_miles * 1.6 | Approximate conversion |
| route_circuity | 1.3 | Typical urban circuity |
| is_airport_* | 0 | Assume not airport |
| is_downtown_* | 0 | Assume not downtown |

---

## Feature Validation

### Completeness Check

```sql
SELECT
  COUNT(*) as total_features,
  COUNTIF(hour_sin IS NOT NULL) as temporal_populated,
  COUNTIF(straight_line_km IS NOT NULL) as geo_populated
FROM feature_store;
```

### Results
| Metric | Value |
|--------|-------|
| Total Features | 10,598,441 |
| Temporal Populated | 10,598,441 (100%) |
| Geospatial Populated | 10,598,441 (100%) |

---

## Certification Topics Demonstrated

| Topic | How Demonstrated |
|-------|------------------|
| Feature Engineering | Cyclical encoding, binary flags |
| Geospatial Analysis | Haversine distance, grid encoding |
| Feature Stores | Unified feature table design |
| NULL Handling | COALESCE with appropriate defaults |
| Domain Knowledge | Chicago area codes, rush hour definitions |

---

## Related Artifacts

### EPICs
- [EPIC-002: Feature Engineering](./epics/EPIC-002-feature-engineering.md)

### User Stories
- [US-003: Temporal Feature Engineering](./user-stories/US-003-temporal-features.md)
- [US-004: Geospatial Feature Engineering](./user-stories/US-004-geospatial-features.md)
- [US-005: Unified Feature Store](./user-stories/US-005-feature-store.md)

---

## Navigation

- **Previous**: [Data Engineering](./03-data-engineering.md)
- **Next**: [Model Training](./05-model-training.md)
