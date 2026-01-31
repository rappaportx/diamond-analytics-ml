# TASK-009-001: Select Features for K-Means Clustering

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-009-001 |
| User Story | [US-009: K-Means Clustering](../user-stories/US-009-kmeans-clustering.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Select and prepare features for K-Means clustering to segment taxi drivers based on their operating patterns.

---

## Prerequisites

- [x] Feature store with driver-level metrics
- [x] Understanding of K-Means requirements
- [x] Cleaned data available

---

## Step-by-Step Instructions

### Step 1: Understand K-Means Requirements

| Requirement | Description |
|-------------|-------------|
| Numeric features | All features must be numeric |
| Scaled features | Features should be normalized |
| No target variable | Unsupervised learning |
| Meaningful features | Features that differentiate segments |

### Step 2: Create Driver-Level Aggregations

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.driver_profiles` AS
SELECT
  taxi_id,

  -- Volume metrics
  COUNT(*) as total_trips,
  COUNT(DISTINCT DATE(trip_start_timestamp)) as active_days,

  -- Revenue metrics
  ROUND(SUM(trip_total), 2) as total_revenue,
  ROUND(AVG(trip_total), 2) as avg_fare,
  ROUND(STDDEV(trip_total), 2) as fare_std,

  -- Distance metrics
  ROUND(SUM(trip_miles), 2) as total_miles,
  ROUND(AVG(trip_miles), 2) as avg_miles,

  -- Time patterns
  ROUND(AVG(EXTRACT(HOUR FROM trip_start_timestamp)), 1) as avg_hour,
  ROUND(COUNTIF(EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 22 AND 23
             OR EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 0 AND 5)
        / COUNT(*) * 100, 1) as night_trip_pct,

  -- Geographic patterns
  ROUND(COUNTIF(pickup_community_area IN (8, 32, 33))  -- Downtown areas
        / COUNT(*) * 100, 1) as downtown_pct,

  -- Airport trips
  ROUND(COUNTIF(
    ST_DISTANCE(
      ST_GEOGPOINT(pickup_longitude, pickup_latitude),
      ST_GEOGPOINT(-87.9073, 41.9742)
    ) < 3000
    OR ST_DISTANCE(
      ST_GEOGPOINT(dropoff_longitude, dropoff_latitude),
      ST_GEOGPOINT(-87.9073, 41.9742)
    ) < 3000
  ) / COUNT(*) * 100, 1) as airport_trip_pct

FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
GROUP BY taxi_id
HAVING COUNT(*) >= 100;  -- Minimum trips for meaningful profile
```

### Step 3: Verify Driver Profiles

```sql
SELECT
  COUNT(*) as total_drivers,
  ROUND(AVG(total_trips), 0) as avg_trips,
  ROUND(AVG(total_revenue), 0) as avg_revenue,
  ROUND(AVG(avg_fare), 2) as avg_fare,
  ROUND(AVG(night_trip_pct), 1) as avg_night_pct,
  ROUND(AVG(airport_trip_pct), 1) as avg_airport_pct
FROM `sonorous-key-320714.diamond_analytics.driver_profiles`;
```

**Expected Output**:

| total_drivers | avg_trips | avg_revenue | avg_fare | avg_night_pct | avg_airport_pct |
|---------------|-----------|-------------|----------|---------------|-----------------|
| 4,523 | 2,341 | $43,256 | $18.47 | 12.3% | 8.7% |

### Step 4: Select Final Clustering Features

| Feature | Description | Why Include |
|---------|-------------|-------------|
| avg_fare | Average fare per trip | Revenue behavior |
| avg_miles | Average distance per trip | Trip type |
| night_trip_pct | % trips at night | Time preference |
| downtown_pct | % downtown pickups | Location preference |
| airport_trip_pct | % airport trips | Specialty focus |
| active_days | Days worked in period | Engagement level |

### Step 5: Create Scaled Feature Table

K-Means is sensitive to scale, so normalize features:

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.driver_features_scaled` AS
WITH stats AS (
  SELECT
    AVG(avg_fare) as fare_mean, STDDEV(avg_fare) as fare_std,
    AVG(avg_miles) as miles_mean, STDDEV(avg_miles) as miles_std,
    AVG(night_trip_pct) as night_mean, STDDEV(night_trip_pct) as night_std,
    AVG(downtown_pct) as downtown_mean, STDDEV(downtown_pct) as downtown_std,
    AVG(airport_trip_pct) as airport_mean, STDDEV(airport_trip_pct) as airport_std,
    AVG(active_days) as days_mean, STDDEV(active_days) as days_std
  FROM `sonorous-key-320714.diamond_analytics.driver_profiles`
)
SELECT
  d.taxi_id,

  -- Z-score normalized features
  (d.avg_fare - s.fare_mean) / s.fare_std as avg_fare_scaled,
  (d.avg_miles - s.miles_mean) / s.miles_std as avg_miles_scaled,
  (d.night_trip_pct - s.night_mean) / s.night_std as night_pct_scaled,
  (d.downtown_pct - s.downtown_mean) / s.downtown_std as downtown_pct_scaled,
  (d.airport_trip_pct - s.airport_mean) / s.airport_std as airport_pct_scaled,
  (d.active_days - s.days_mean) / s.days_std as active_days_scaled

FROM `sonorous-key-320714.diamond_analytics.driver_profiles` d
CROSS JOIN stats s;
```

### Step 6: Verify Scaling

```sql
SELECT
  ROUND(AVG(avg_fare_scaled), 4) as fare_mean,
  ROUND(STDDEV(avg_fare_scaled), 4) as fare_std,
  ROUND(AVG(avg_miles_scaled), 4) as miles_mean,
  ROUND(STDDEV(avg_miles_scaled), 4) as miles_std
FROM `sonorous-key-320714.diamond_analytics.driver_features_scaled`;
```

**Expected Output** (should be ~0 mean, ~1 std):

| fare_mean | fare_std | miles_mean | miles_std |
|-----------|----------|------------|-----------|
| 0.0000 | 1.0000 | 0.0000 | 1.0000 |

---

## Selected Features Summary

| Feature | Original Range | Scaled Range | Purpose |
|---------|----------------|--------------|---------|
| avg_fare | $5-80 | -2 to +4 | Revenue pattern |
| avg_miles | 0.5-25 | -2 to +3 | Distance pattern |
| night_trip_pct | 0-60% | -1 to +4 | Time preference |
| downtown_pct | 0-90% | -2 to +3 | Location focus |
| airport_trip_pct | 0-40% | -1 to +5 | Specialty trips |
| active_days | 10-500 | -2 to +3 | Engagement |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Feature Selection | Choosing relevant features |
| Feature Scaling | Z-score normalization |
| Aggregation | Creating entity-level features |
| Unsupervised Learning | No target variable |

---

## Next Task

[TASK-009-002: Determine Optimal K](./TASK-009-002-determine-k.md)
