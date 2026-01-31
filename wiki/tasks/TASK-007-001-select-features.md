# TASK-007-001: Select Features for XGBoost Model

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-007-001 |
| User Story | [US-007: XGBoost Training](../user-stories/US-007-xgboost-training.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Select the optimal set of features from the feature store for training the XGBoost fare prediction model.

---

## Prerequisites

- [x] Feature store created with all features
- [x] Training data available
- [x] Understanding of feature importance

---

## Step-by-Step Instructions

### Step 1: Review Available Features

List all features in the feature store:

```sql
SELECT column_name, data_type
FROM `sonorous-key-320714.diamond_analytics.INFORMATION_SCHEMA.COLUMNS`
WHERE table_name = 'feature_store'
ORDER BY ordinal_position;
```

### Step 2: Categorize Features

#### Numeric Features (Direct Use)

| Feature | Type | Expected Importance |
|---------|------|---------------------|
| trip_miles | FLOAT64 | Very High |
| trip_seconds | INTEGER | High |
| straight_line_km | FLOAT64 | High |
| pickup_downtown_km | FLOAT64 | Medium |
| dropoff_downtown_km | FLOAT64 | Medium |
| pickup_ohare_km | FLOAT64 | Medium |
| dropoff_ohare_km | FLOAT64 | Medium |
| route_efficiency | FLOAT64 | Medium |

#### Cyclical Features (Temporal)

| Feature | Type | Expected Importance |
|---------|------|---------------------|
| hour_sin | FLOAT64 | Medium |
| hour_cos | FLOAT64 | Medium |
| dow_sin | FLOAT64 | Low-Medium |
| dow_cos | FLOAT64 | Low-Medium |
| month_sin | FLOAT64 | Low |
| month_cos | FLOAT64 | Low |

#### Boolean Features

| Feature | Type | Expected Importance |
|---------|------|---------------------|
| is_weekend | BOOLEAN | Medium |
| is_rush_hour | BOOLEAN | Medium |
| is_night | BOOLEAN | Low-Medium |
| is_downtown_pickup | BOOLEAN | Medium |
| is_airport_trip | BOOLEAN | High |

#### Target Variable

| Feature | Type | Purpose |
|---------|------|---------|
| target_fare | FLOAT64 | Label for training |

### Step 3: Features to Exclude

| Feature | Reason for Exclusion |
|---------|---------------------|
| unique_key | Identifier, not predictive |
| taxi_id | Identifier, causes overfitting |
| trip_start_timestamp | Used for splitting only |
| pickup_latitude | Raw coords not useful |
| pickup_longitude | Raw coords not useful |
| dropoff_latitude | Raw coords not useful |
| dropoff_longitude | Raw coords not useful |
| is_eval | Split indicator only |
| hour_of_day | Redundant with cyclical |
| day_of_week | Redundant with cyclical |
| month | Redundant with cyclical |

### Step 4: Create Feature Selection View

```sql
CREATE OR REPLACE VIEW `sonorous-key-320714.diamond_analytics.xgb_training_features` AS
SELECT
  -- Distance features
  trip_miles,
  trip_seconds,
  straight_line_km,
  route_efficiency,

  -- Geographic features
  pickup_downtown_km,
  dropoff_downtown_km,
  pickup_ohare_km,
  dropoff_ohare_km,
  pickup_midway_km,
  dropoff_midway_km,

  -- Cyclical temporal features
  hour_sin,
  hour_cos,
  dow_sin,
  dow_cos,
  month_sin,
  month_cos,

  -- Boolean features
  is_weekend,
  is_rush_hour,
  is_night,
  is_downtown_pickup,
  is_airport_trip,

  -- Target variable
  target_fare,

  -- Split indicator
  is_eval

FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

### Step 5: Verify Feature Selection

```sql
SELECT
  COUNT(*) as row_count,
  COUNT(DISTINCT is_eval) as split_values,
  COUNTIF(is_eval = TRUE) as eval_rows,
  COUNTIF(is_eval = FALSE) as train_rows
FROM `sonorous-key-320714.diamond_analytics.xgb_training_features`;
```

**Expected Output**:

| row_count | split_values | eval_rows | train_rows |
|-----------|--------------|-----------|------------|
| ~5,300,000 | 2 | ~1,060,000 | ~4,240,000 |

### Step 6: Check for NULL Values

```sql
SELECT
  COUNTIF(trip_miles IS NULL) as null_miles,
  COUNTIF(trip_seconds IS NULL) as null_seconds,
  COUNTIF(straight_line_km IS NULL) as null_distance,
  COUNTIF(target_fare IS NULL) as null_target
FROM `sonorous-key-320714.diamond_analytics.xgb_training_features`;
```

**Expected**: All zeros (no NULLs in selected features)

---

## Selected Features Summary

### Final Feature Count: 21

| Category | Count | Features |
|----------|-------|----------|
| Distance | 4 | trip_miles, trip_seconds, straight_line_km, route_efficiency |
| Geographic | 6 | pickup/dropoff downtown/ohare/midway distances |
| Temporal | 6 | hour/dow/month sin/cos |
| Boolean | 5 | is_weekend, is_rush_hour, is_night, is_downtown_pickup, is_airport_trip |
| **Total** | **21** | |

### Feature Correlation with Target

```sql
SELECT
  ROUND(CORR(trip_miles, target_fare), 3) as miles_corr,
  ROUND(CORR(trip_seconds, target_fare), 3) as seconds_corr,
  ROUND(CORR(straight_line_km, target_fare), 3) as distance_corr,
  ROUND(CORR(pickup_downtown_km, target_fare), 3) as downtown_corr
FROM `sonorous-key-320714.diamond_analytics.xgb_training_features`;
```

**Expected Output**:

| miles_corr | seconds_corr | distance_corr | downtown_corr |
|------------|--------------|---------------|---------------|
| 0.85 | 0.72 | 0.78 | -0.15 |

---

## Feature Engineering Decisions

| Decision | Rationale |
|----------|-----------|
| Use cyclical encoding | Preserves time continuity |
| Include airport distances | Strong fare predictor |
| Exclude raw timestamps | Causes data leakage |
| Exclude taxi_id | Prevents overfitting |
| Keep all boolean flags | Low cardinality, useful |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Feature Selection | Choosing predictive features |
| Data Leakage Prevention | Excluding future information |
| Correlation Analysis | Feature-target relationships |
| Encoding Strategies | Cyclical, boolean representations |

---

## Next Task

[TASK-007-002: Configure Hyperparameters](./TASK-007-002-hyperparameters.md)
