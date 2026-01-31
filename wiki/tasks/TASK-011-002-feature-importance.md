# TASK-011-002: Feature Importance Analysis

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-011-002 |
| User Story | [US-011: Model Evaluation](../user-stories/US-011-model-evaluation.md) |
| EPIC | [EPIC-004: Model Evaluation](../epics/EPIC-004-model-evaluation.md) |
| Status | Complete |

---

## Objective

Analyze feature importance from the XGBoost model to understand which features drive fare predictions.

---

## Prerequisites

- [x] XGBoost model trained with enable_global_explain = TRUE
- [x] Model evaluation complete (TASK-011-001)

---

## Step-by-Step Instructions

### Step 1: Get Global Feature Importance

```sql
SELECT *
FROM ML.FEATURE_IMPORTANCE(
  MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`
)
ORDER BY importance_weight DESC;
```

**Expected Output**:

| feature | importance_weight | importance_gain | importance_cover |
|---------|-------------------|-----------------|------------------|
| trip_miles | 0.2845 | 0.4523 | 0.1876 |
| trip_seconds | 0.1923 | 0.2134 | 0.1567 |
| straight_line_km | 0.1456 | 0.1234 | 0.1234 |
| dropoff_ohare_km | 0.0734 | 0.0456 | 0.0876 |
| pickup_ohare_km | 0.0678 | 0.0412 | 0.0812 |
| route_efficiency | 0.0534 | 0.0298 | 0.0567 |
| pickup_downtown_km | 0.0423 | 0.0234 | 0.0456 |
| hour_sin | 0.0312 | 0.0156 | 0.0345 |
| ... | ... | ... | ... |

### Step 2: Understand Importance Metrics

| Metric | Description | Best For |
|--------|-------------|----------|
| importance_weight | Frequency of splits | Identifying "busy" features |
| importance_gain | Avg improvement when feature used | Identifying powerful features |
| importance_cover | Avg samples affected | Identifying broad-impact features |

**Recommendation**: Use `importance_gain` for XGBoost.

### Step 3: Create Feature Importance Summary

```sql
SELECT
  feature,
  ROUND(importance_gain, 4) as gain,
  ROUND(importance_gain / SUM(importance_gain) OVER() * 100, 2) as gain_pct,
  RANK() OVER(ORDER BY importance_gain DESC) as rank
FROM ML.FEATURE_IMPORTANCE(
  MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`
)
ORDER BY importance_gain DESC;
```

**Expected Output**:

| feature | gain | gain_pct | rank |
|---------|------|----------|------|
| trip_miles | 0.4523 | 45.2% | 1 |
| trip_seconds | 0.2134 | 21.3% | 2 |
| straight_line_km | 0.1234 | 12.3% | 3 |
| dropoff_ohare_km | 0.0456 | 4.6% | 4 |
| pickup_ohare_km | 0.0412 | 4.1% | 5 |

### Step 4: Visualize Top Features

```
Feature Importance (% of Total Gain)
=====================================

trip_miles        ████████████████████████████████████████████░ 45.2%
trip_seconds      █████████████████████░░░░░░░░░░░░░░░░░░░░░░░░ 21.3%
straight_line_km  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 12.3%
dropoff_ohare_km  ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  4.6%
pickup_ohare_km   ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  4.1%
route_efficiency  ███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  3.0%
Others            ████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  9.5%
```

### Step 5: Group Features by Category

```sql
WITH importance AS (
  SELECT
    feature,
    importance_gain,
    CASE
      WHEN feature IN ('trip_miles', 'trip_seconds', 'straight_line_km', 'route_efficiency')
        THEN 'Distance'
      WHEN feature LIKE '%ohare%' OR feature LIKE '%midway%' OR feature LIKE '%downtown%'
        THEN 'Geographic'
      WHEN feature LIKE 'hour%' OR feature LIKE 'dow%' OR feature LIKE 'month%'
        THEN 'Temporal_Cyclical'
      WHEN feature IN ('is_weekend', 'is_rush_hour', 'is_night', 'is_downtown_pickup', 'is_airport_trip')
        THEN 'Boolean_Flags'
      ELSE 'Other'
    END as category
  FROM ML.FEATURE_IMPORTANCE(
    MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`
  )
)
SELECT
  category,
  COUNT(*) as feature_count,
  ROUND(SUM(importance_gain), 4) as total_gain,
  ROUND(SUM(importance_gain) / (SELECT SUM(importance_gain) FROM importance) * 100, 1) as pct_contribution
FROM importance
GROUP BY category
ORDER BY total_gain DESC;
```

**Expected Output**:

| category | feature_count | total_gain | pct_contribution |
|----------|---------------|------------|------------------|
| Distance | 4 | 0.818 | 81.8% |
| Geographic | 6 | 0.098 | 9.8% |
| Boolean_Flags | 5 | 0.052 | 5.2% |
| Temporal_Cyclical | 6 | 0.032 | 3.2% |

### Step 6: Interpret Results

#### Top 5 Most Important Features

| Rank | Feature | Importance | Interpretation |
|------|---------|------------|----------------|
| 1 | trip_miles | 45.2% | Primary fare determinant |
| 2 | trip_seconds | 21.3% | Duration matters for traffic |
| 3 | straight_line_km | 12.3% | Direct distance baseline |
| 4 | dropoff_ohare_km | 4.6% | Airport premium for drops |
| 5 | pickup_ohare_km | 4.1% | Airport premium for pickups |

#### Feature Category Analysis

| Category | % Contribution | Business Insight |
|----------|----------------|------------------|
| Distance | 81.8% | Fare primarily distance-based |
| Geographic | 9.8% | Airport/downtown adds premium |
| Boolean Flags | 5.2% | Time context moderately important |
| Temporal | 3.2% | Hour/day has minor effect |

### Step 7: Document Feature Insights

Key findings for certification:

1. **Distance Dominates**: 82% of predictive power from distance features
2. **Airport Premium**: O'Hare proximity adds significant value
3. **Time Features Less Critical**: Cyclical encoding provides marginal lift
4. **Boolean Flags Useful**: Simple indicators add predictive value

---

## Feature Importance Table

| Rank | Feature | Gain | Category | Insight |
|------|---------|------|----------|---------|
| 1 | trip_miles | 0.452 | Distance | Core predictor |
| 2 | trip_seconds | 0.213 | Distance | Traffic impact |
| 3 | straight_line_km | 0.123 | Distance | Route efficiency |
| 4 | dropoff_ohare_km | 0.046 | Geographic | Airport premium |
| 5 | pickup_ohare_km | 0.041 | Geographic | Airport premium |
| 6 | route_efficiency | 0.030 | Distance | Indirect routes |
| 7 | pickup_downtown_km | 0.023 | Geographic | Downtown demand |
| 8 | is_airport_trip | 0.018 | Boolean | Airport flag |
| 9 | dropoff_downtown_km | 0.015 | Geographic | Destination value |
| 10 | hour_sin | 0.012 | Temporal | Peak hours |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| ML.FEATURE_IMPORTANCE | BigQuery ML explainability |
| Importance Metrics | Weight, Gain, Cover |
| Model Interpretability | Understanding predictions |
| Feature Engineering Validation | Did features help? |

---

## Next Task

[TASK-011-003: Evaluate K-Means Clusters](./TASK-011-003-evaluate-kmeans.md)
