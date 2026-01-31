# TASK-009-003: Train K-Means Model

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-009-003 |
| User Story | [US-009: K-Means Clustering](../user-stories/US-009-kmeans-clustering.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Train the K-Means clustering model with k=5 clusters to segment taxi drivers.

---

## Prerequisites

- [x] Driver features prepared (TASK-009-001)
- [x] Optimal k determined (TASK-009-002): k=5
- [x] Scaled features available

---

## Step-by-Step Instructions

### Step 1: Create K-Means Model

```sql
CREATE OR REPLACE MODEL `sonorous-key-320714.diamond_analytics.taxi_segments_kmeans`
OPTIONS(
  model_type = 'KMEANS',
  num_clusters = 5,
  kmeans_init_method = 'KMEANS++',
  max_iterations = 50,
  min_rel_progress = 0.001,
  standardize_features = TRUE
) AS
SELECT
  -- Features for clustering (excluding taxi_id)
  avg_fare,
  avg_miles,
  night_trip_pct,
  downtown_pct,
  airport_trip_pct,
  CAST(active_days AS FLOAT64) as active_days
FROM `sonorous-key-320714.diamond_analytics.driver_profiles`;
```

**Option Explanation**:

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | KMEANS | Clustering algorithm |
| num_clusters | 5 | Number of segments |
| kmeans_init_method | KMEANS++ | Smart initialization |
| max_iterations | 50 | Convergence limit |
| min_rel_progress | 0.001 | Early stopping |
| standardize_features | TRUE | Auto-scale features |

### Step 2: Verify Model Creation

```sql
SELECT
  iteration,
  loss,
  duration_ms
FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.taxi_segments_kmeans`)
ORDER BY iteration;
```

**Expected Output**:

| iteration | loss | duration_ms |
|-----------|------|-------------|
| 1 | 25456.78 | 3500 |
| 5 | 18234.56 | 2800 |
| 10 | 16789.12 | 2600 |
| 15 | 16543.45 | 2500 |
| 20 | 16498.23 | 2400 |

### Step 3: Evaluate Clustering Quality

```sql
SELECT *
FROM ML.EVALUATE(MODEL `sonorous-key-320714.diamond_analytics.taxi_segments_kmeans`);
```

**Expected Output**:

| davies_bouldin_index | mean_squared_distance |
|---------------------|----------------------|
| 1.23 | 3.45 |

**Metric Interpretation**:

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Davies-Bouldin Index | 1.23 | Lower = better separation (< 2 is good) |
| Mean Squared Distance | 3.45 | Average distance to centroid |

### Step 4: Get Cluster Centroids

```sql
SELECT
  centroid_id,
  feature,
  ROUND(numerical_value, 2) as centroid_value
FROM ML.CENTROIDS(MODEL `sonorous-key-320714.diamond_analytics.taxi_segments_kmeans`)
ORDER BY centroid_id, feature;
```

**Expected Output** (partial):

| centroid_id | feature | centroid_value |
|-------------|---------|----------------|
| 1 | active_days | 245.67 |
| 1 | airport_trip_pct | 5.23 |
| 1 | avg_fare | 16.78 |
| 1 | avg_miles | 3.12 |
| 1 | downtown_pct | 45.67 |
| 1 | night_trip_pct | 8.34 |
| 2 | active_days | 312.45 |
| 2 | airport_trip_pct | 2.12 |
| ... | ... | ... |

### Step 5: Assign Drivers to Clusters

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.driver_segments` AS
SELECT
  taxi_id,
  CENTROID_ID as cluster_id,
  avg_fare,
  avg_miles,
  night_trip_pct,
  downtown_pct,
  airport_trip_pct,
  active_days
FROM ML.PREDICT(
  MODEL `sonorous-key-320714.diamond_analytics.taxi_segments_kmeans`,
  (SELECT * FROM `sonorous-key-320714.diamond_analytics.driver_profiles`)
);
```

### Step 6: Analyze Cluster Sizes

```sql
SELECT
  cluster_id,
  COUNT(*) as driver_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) as pct
FROM `sonorous-key-320714.diamond_analytics.driver_segments`
GROUP BY cluster_id
ORDER BY cluster_id;
```

**Expected Output**:

| cluster_id | driver_count | pct |
|------------|--------------|-----|
| 1 | 1,234 | 27.3% |
| 2 | 987 | 21.8% |
| 3 | 856 | 18.9% |
| 4 | 823 | 18.2% |
| 5 | 623 | 13.8% |

---

## Cluster Profiles

### Step 7: Generate Cluster Profiles

```sql
SELECT
  cluster_id,
  COUNT(*) as drivers,
  ROUND(AVG(avg_fare), 2) as avg_fare,
  ROUND(AVG(avg_miles), 2) as avg_miles,
  ROUND(AVG(night_trip_pct), 1) as night_pct,
  ROUND(AVG(downtown_pct), 1) as downtown_pct,
  ROUND(AVG(airport_trip_pct), 1) as airport_pct,
  ROUND(AVG(active_days), 0) as avg_active_days
FROM `sonorous-key-320714.diamond_analytics.driver_segments`
GROUP BY cluster_id
ORDER BY cluster_id;
```

**Expected Output**:

| cluster_id | drivers | avg_fare | avg_miles | night_pct | downtown_pct | airport_pct | avg_active_days |
|------------|---------|----------|-----------|-----------|--------------|-------------|-----------------|
| 1 | 1,234 | $15.23 | 2.8 | 32.5% | 25.6% | 3.2% | 178 |
| 2 | 987 | $17.89 | 3.5 | 8.4% | 52.3% | 4.5% | 312 |
| 3 | 856 | $16.45 | 3.2 | 12.3% | 38.7% | 5.1% | 256 |
| 4 | 823 | $18.67 | 4.1 | 15.6% | 42.1% | 6.8% | 289 |
| 5 | 623 | $28.45 | 8.7 | 18.2% | 15.3% | 35.6% | 234 |

---

## Cluster Naming

| Cluster | Profile | Suggested Name |
|---------|---------|----------------|
| 1 | High night %, lower fares | Night Owls |
| 2 | High downtown %, most active | Downtown Regulars |
| 3 | Balanced metrics, moderate activity | Balanced Operators |
| 4 | Above avg fares, high downtown | Downtown Focus |
| 5 | High airport %, highest fares | Airport Specialists |

---

## Model Summary

| Metric | Value |
|--------|-------|
| Clusters | 5 |
| Drivers Clustered | 4,523 |
| Davies-Bouldin Index | 1.23 |
| Iterations to Converge | 20 |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| K-Means Training | CREATE MODEL with KMEANS |
| Cluster Evaluation | Davies-Bouldin Index |
| Centroid Analysis | ML.CENTROIDS function |
| Cluster Assignment | ML.PREDICT for segments |

---

## Next Task

[TASK-009-004: Profile Clusters](./TASK-009-004-profile-clusters.md)
