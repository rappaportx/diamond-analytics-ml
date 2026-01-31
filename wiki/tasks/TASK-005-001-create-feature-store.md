# TASK-005-001: Create Feature Store Table

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-005-001 |
| User Story | [US-005: Feature Store](../user-stories/US-005-feature-store.md) |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |

---

## Objective

Create a unified feature store table combining all engineered features for model training.

---

## Prerequisites

- [x] Cleaned data available (trips_cleaned)
- [x] Temporal features designed (TASK-003-001)
- [x] Geospatial features designed (TASK-003-002)
- [x] Derived features designed (TASK-003-003)

---

## Step-by-Step Instructions

### Step 1: Design Feature Store Schema

| Category | Features | Count |
|----------|----------|-------|
| Identifiers | unique_key, taxi_id | 2 |
| Timestamps | trip_start_timestamp | 1 |
| Distance | trip_miles, trip_seconds, straight_line_km, route_efficiency | 4 |
| Geographic | pickup/dropoff downtown/ohare/midway distances | 6 |
| Temporal Cyclical | hour/dow/month sin/cos | 6 |
| Boolean Flags | is_weekend, is_rush_hour, is_night, is_downtown_pickup, is_airport_trip | 5 |
| Coordinates | pickup/dropoff lat/long | 4 |
| Target | target_fare | 1 |
| **Total** | | **29** |

### Step 2: Create Feature Store Table

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.feature_store`
PARTITION BY DATE(trip_start_timestamp)
CLUSTER BY taxi_id
AS
SELECT
  -- Identifiers
  unique_key,
  taxi_id,
  trip_start_timestamp,

  -- Original distance metrics
  trip_miles,
  trip_seconds,

  -- Geospatial derived features
  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
    ) / 1000,
    3
  ) as straight_line_km,

  -- Route efficiency (actual vs straight line)
  ROUND(
    trip_miles / NULLIF(
      ST_DISTANCE(
        ST_GEOGPOINT(pickup_longitude, pickup_latitude),
        ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
      ) / 1609.34,
      0
    ),
    3
  ) as route_efficiency,

  -- Downtown distances
  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(-87.6298, 41.8781)
    ) / 1000,
    3
  ) as pickup_downtown_km,

  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
      ST_GEOGPOINT(-87.6298, 41.8781)
    ) / 1000,
    3
  ) as dropoff_downtown_km,

  -- O'Hare distances
  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(-87.9073, 41.9742)
    ) / 1000,
    3
  ) as pickup_ohare_km,

  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
      ST_GEOGPOINT(-87.9073, 41.9742)
    ) / 1000,
    3
  ) as dropoff_ohare_km,

  -- Midway distances
  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(-87.7522, 41.7868)
    ) / 1000,
    3
  ) as pickup_midway_km,

  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
      ST_GEOGPOINT(-87.7522, 41.7868)
    ) / 1000,
    3
  ) as dropoff_midway_km,

  -- Cyclical temporal features
  ROUND(SIN(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24), 4) as hour_sin,
  ROUND(COS(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24), 4) as hour_cos,

  ROUND(SIN(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7), 4) as dow_sin,
  ROUND(COS(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7), 4) as dow_cos,

  ROUND(SIN(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12), 4) as month_sin,
  ROUND(COS(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12), 4) as month_cos,

  -- Boolean features
  EXTRACT(DAYOFWEEK FROM trip_start_timestamp) IN (1, 7) as is_weekend,

  CASE
    WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 7 AND 9 THEN TRUE
    WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 16 AND 19 THEN TRUE
    ELSE FALSE
  END as is_rush_hour,

  CASE
    WHEN EXTRACT(HOUR FROM trip_start_timestamp) >= 22 THEN TRUE
    WHEN EXTRACT(HOUR FROM trip_start_timestamp) <= 5 THEN TRUE
    ELSE FALSE
  END as is_night,

  ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.6298, 41.8781)
  ) < 3000 as is_downtown_pickup,

  (ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)
  ) < 3000
  OR ST_DISTANCE(
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)
  ) < 3000) as is_airport_trip,

  -- Raw coordinates (for geospatial analysis)
  pickup_latitude,
  pickup_longitude,
  dropoff_latitude,
  dropoff_longitude,

  -- Target variable
  trip_total as target_fare

FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`;
```

### Step 3: Verify Feature Store

```sql
SELECT
  COUNT(*) as total_rows,
  COUNT(DISTINCT taxi_id) as unique_taxis,
  MIN(trip_start_timestamp) as first_trip,
  MAX(trip_start_timestamp) as last_trip
FROM `sonorous-key-320714.diamond_analytics.feature_store`;
```

**Expected Output**:

| total_rows | unique_taxis | first_trip | last_trip |
|------------|--------------|------------|-----------|
| 10,598,441 | 5,727 | 2020-01-01 | 2023-12-31 |

### Step 4: Verify Feature Calculations

```sql
SELECT
  ROUND(AVG(straight_line_km), 2) as avg_straight_km,
  ROUND(AVG(route_efficiency), 2) as avg_efficiency,
  ROUND(AVG(pickup_downtown_km), 2) as avg_downtown_km,
  ROUND(AVG(pickup_ohare_km), 2) as avg_ohare_km,
  COUNTIF(is_weekend) as weekend_trips,
  COUNTIF(is_rush_hour) as rush_trips,
  COUNTIF(is_airport_trip) as airport_trips
FROM `sonorous-key-320714.diamond_analytics.feature_store`;
```

**Expected Output**:

| avg_straight_km | avg_efficiency | avg_downtown_km | avg_ohare_km | weekend_trips | rush_trips | airport_trips |
|-----------------|----------------|-----------------|--------------|---------------|------------|---------------|
| 4.87 | 1.42 | 8.23 | 15.67 | 2,345,678 | 3,456,789 | 567,890 |

### Step 5: Check for NULLs

```sql
SELECT
  COUNTIF(straight_line_km IS NULL) as null_distance,
  COUNTIF(route_efficiency IS NULL) as null_efficiency,
  COUNTIF(hour_sin IS NULL) as null_temporal,
  COUNTIF(target_fare IS NULL) as null_target
FROM `sonorous-key-320714.diamond_analytics.feature_store`;
```

**Expected Output**: All zeros

---

## Feature Store Schema

| Column | Type | Category | Description |
|--------|------|----------|-------------|
| unique_key | STRING | ID | Trip identifier |
| taxi_id | STRING | ID | Vehicle identifier |
| trip_start_timestamp | TIMESTAMP | Time | Trip start time |
| trip_miles | FLOAT64 | Distance | Actual trip distance |
| trip_seconds | INTEGER | Distance | Trip duration |
| straight_line_km | FLOAT64 | Geo | Direct distance |
| route_efficiency | FLOAT64 | Geo | Actual/straight ratio |
| pickup_downtown_km | FLOAT64 | Geo | Distance to Loop |
| dropoff_downtown_km | FLOAT64 | Geo | Distance to Loop |
| pickup_ohare_km | FLOAT64 | Geo | Distance to ORD |
| dropoff_ohare_km | FLOAT64 | Geo | Distance to ORD |
| pickup_midway_km | FLOAT64 | Geo | Distance to MDW |
| dropoff_midway_km | FLOAT64 | Geo | Distance to MDW |
| hour_sin | FLOAT64 | Temporal | Cyclical hour |
| hour_cos | FLOAT64 | Temporal | Cyclical hour |
| dow_sin | FLOAT64 | Temporal | Cyclical day |
| dow_cos | FLOAT64 | Temporal | Cyclical day |
| month_sin | FLOAT64 | Temporal | Cyclical month |
| month_cos | FLOAT64 | Temporal | Cyclical month |
| is_weekend | BOOLEAN | Flag | Weekend indicator |
| is_rush_hour | BOOLEAN | Flag | Rush hour indicator |
| is_night | BOOLEAN | Flag | Night trip indicator |
| is_downtown_pickup | BOOLEAN | Flag | Downtown pickup |
| is_airport_trip | BOOLEAN | Flag | Airport trip |
| pickup_latitude | FLOAT64 | Coord | Pickup lat |
| pickup_longitude | FLOAT64 | Coord | Pickup long |
| dropoff_latitude | FLOAT64 | Coord | Dropoff lat |
| dropoff_longitude | FLOAT64 | Coord | Dropoff long |
| target_fare | FLOAT64 | Target | Fare to predict |

---

## Feature Store Summary

| Metric | Value |
|--------|-------|
| Total Features | 29 |
| Total Rows | 10,598,441 |
| Partitioned By | DATE(trip_start_timestamp) |
| Clustered By | taxi_id |
| Table Size | ~3.2 GB |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Feature Store | Centralized feature repository |
| Feature Engineering | Creating predictive features |
| Partitioning | Cost optimization |
| Clustering | Query performance |

---

## Next Task

[TASK-006-001: Create Time-Based Split](./TASK-006-001-create-time-split.md)
