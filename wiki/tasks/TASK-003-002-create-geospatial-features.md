# TASK-003-002: Create Geospatial Features

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-003-002 |
| User Story | [US-004: Geospatial Features](../user-stories/US-004-geospatial-features.md) |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |

---

## Objective

Create geospatial features including straight-line distance, geographic regions, and airport proximity indicators.

---

## Prerequisites

- [x] Cleaned table with coordinates (TASK-002-002)
- [x] Understanding of BigQuery GEOGRAPHY functions
- [x] Knowledge of Chicago geography

---

## Step-by-Step Instructions

### Step 1: Understand ST_DISTANCE Function

BigQuery's `ST_DISTANCE` calculates the geodesic distance between two points in meters.

```sql
ST_DISTANCE(
  ST_GEOGPOINT(longitude1, latitude1),
  ST_GEOGPOINT(longitude2, latitude2)
)
```

**Note**: Order is (longitude, latitude), not (latitude, longitude)!

### Step 2: Calculate Straight-Line Distance

```sql
SELECT
  trip_miles,
  ROUND(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
    ) / 1000,  -- Convert meters to kilometers
    2
  ) as straight_line_km
FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
LIMIT 5;
```

**Expected Output**:

| trip_miles | straight_line_km |
|------------|------------------|
| 2.5 | 3.12 |
| 5.8 | 7.45 |
| 1.2 | 1.56 |

### Step 3: Define Chicago Airport Locations

| Airport | Code | Latitude | Longitude |
|---------|------|----------|-----------|
| O'Hare | ORD | 41.9742 | -87.9073 |
| Midway | MDW | 41.7868 | -87.7522 |

### Step 4: Calculate Airport Proximity

```sql
-- O'Hare Airport
ROUND(
  ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)  -- O'Hare
  ) / 1000,
  2
) as pickup_ohare_km,

ROUND(
  ST_DISTANCE(
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)  -- O'Hare
  ) / 1000,
  2
) as dropoff_ohare_km,

-- Midway Airport
ROUND(
  ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.7522, 41.7868)  -- Midway
  ) / 1000,
  2
) as pickup_midway_km,

ROUND(
  ST_DISTANCE(
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
    ST_GEOGPOINT(-87.7522, 41.7868)  -- Midway
  ) / 1000,
  2
) as dropoff_midway_km
```

### Step 5: Create Airport Trip Flags

```sql
-- Flag for airport pickups (within 2km of airport)
CASE
  WHEN ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)
  ) < 2000 THEN 'ORD'
  WHEN ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.7522, 41.7868)
  ) < 2000 THEN 'MDW'
  ELSE 'CITY'
END as pickup_airport,

-- Flag for airport dropoffs
CASE
  WHEN ST_DISTANCE(
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
    ST_GEOGPOINT(-87.9073, 41.9742)
  ) < 2000 THEN 'ORD'
  WHEN ST_DISTANCE(
    ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
    ST_GEOGPOINT(-87.7522, 41.7868)
  ) < 2000 THEN 'MDW'
  ELSE 'CITY'
END as dropoff_airport
```

### Step 6: Define Downtown Chicago

Downtown (Loop) center: (41.8781, -87.6298)

```sql
-- Distance from downtown
ROUND(
  ST_DISTANCE(
    ST_GEOGPOINT(pickup_longitude, pickup_latitude),
    ST_GEOGPOINT(-87.6298, 41.8781)  -- Downtown Chicago
  ) / 1000,
  2
) as pickup_downtown_km,

-- Is downtown pickup (within 3km)
ST_DISTANCE(
  ST_GEOGPOINT(pickup_longitude, pickup_latitude),
  ST_GEOGPOINT(-87.6298, 41.8781)
) < 3000 as is_downtown_pickup
```

### Step 7: Calculate Route Efficiency

```sql
-- Route efficiency: actual miles / straight-line distance
-- Higher = more indirect route
ROUND(
  trip_miles / NULLIF(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude)
    ) / 1609.34,  -- Convert meters to miles
    0
  ),
  2
) as route_efficiency
```

**Interpretation**:

| Efficiency | Meaning |
|------------|---------|
| 1.0 | Perfect straight line |
| 1.2-1.5 | Normal city driving |
| > 2.0 | Very indirect route |

---

## Complete Geospatial Feature Set

| Feature | Type | Formula | Purpose |
|---------|------|---------|---------|
| straight_line_km | FLOAT64 | ST_DISTANCE / 1000 | Direct distance |
| pickup_ohare_km | FLOAT64 | Distance to ORD | Airport proximity |
| dropoff_ohare_km | FLOAT64 | Distance to ORD | Airport proximity |
| pickup_midway_km | FLOAT64 | Distance to MDW | Airport proximity |
| dropoff_midway_km | FLOAT64 | Distance to MDW | Airport proximity |
| pickup_airport | STRING | CASE statement | Airport category |
| dropoff_airport | STRING | CASE statement | Airport category |
| pickup_downtown_km | FLOAT64 | Distance to Loop | Downtown proximity |
| is_downtown_pickup | BOOLEAN | distance < 3km | Downtown flag |
| route_efficiency | FLOAT64 | actual / straight | Route directness |

---

## Verify Geospatial Features

```sql
SELECT
  pickup_airport,
  dropoff_airport,
  COUNT(*) as trip_count,
  ROUND(AVG(trip_total), 2) as avg_fare
FROM (
  SELECT
    trip_total,
    CASE
      WHEN ST_DISTANCE(ST_GEOGPOINT(pickup_longitude, pickup_latitude),
                       ST_GEOGPOINT(-87.9073, 41.9742)) < 2000 THEN 'ORD'
      WHEN ST_DISTANCE(ST_GEOGPOINT(pickup_longitude, pickup_latitude),
                       ST_GEOGPOINT(-87.7522, 41.7868)) < 2000 THEN 'MDW'
      ELSE 'CITY'
    END as pickup_airport,
    CASE
      WHEN ST_DISTANCE(ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
                       ST_GEOGPOINT(-87.9073, 41.9742)) < 2000 THEN 'ORD'
      WHEN ST_DISTANCE(ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
                       ST_GEOGPOINT(-87.7522, 41.7868)) < 2000 THEN 'MDW'
      ELSE 'CITY'
    END as dropoff_airport
  FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
)
GROUP BY pickup_airport, dropoff_airport
ORDER BY trip_count DESC;
```

**Expected Insight**: Airport trips have significantly higher average fares.

---

## Chicago Geography Reference

| Location | Lat | Long | Description |
|----------|-----|------|-------------|
| O'Hare (ORD) | 41.9742 | -87.9073 | Major international airport |
| Midway (MDW) | 41.7868 | -87.7522 | Secondary airport |
| Downtown/Loop | 41.8781 | -87.6298 | Business district center |
| Navy Pier | 41.8917 | -87.6086 | Tourist destination |
| Wrigley Field | 41.9484 | -87.6553 | Cubs stadium |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Geospatial Functions | ST_DISTANCE, ST_GEOGPOINT |
| Feature Engineering | Creating predictive features |
| Domain Knowledge | Airport, downtown patterns |
| Distance Calculations | Geodesic vs Euclidean |

---

## Next Task

[TASK-003-003: Create Derived Features](./TASK-003-003-create-derived-features.md)
