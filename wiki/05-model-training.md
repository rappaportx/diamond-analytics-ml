# Model Training Documentation

## Overview

This phase covers the training of four distinct ML models using BigQuery ML:
1. XGBoost Regressor (supervised)
2. ARIMA_PLUS (time series)
3. K-Means (unsupervised clustering)
4. Autoencoder (neural network for anomaly detection)

---

## Train/Test/Holdout Split Strategy

### Time-Based Split (Critical for Certification)

**Certification Concept**: Time-based splits prevent data leakage in temporal data. Random splits would allow the model to "see" future data during training.

```
Timeline: ─────────────────────────────────────────────────────────────────►
          │          TRAIN          │    TEST    │   HOLDOUT   │
          │      2022-01-01         │ 2023-07-01 │  2023-10-01 │
          │         to              │     to     │     to      │
          │      2023-06-30         │ 2023-09-30 │  2023-12-31 │
          │        (73%)            │   (13%)    │    (13%)    │
          │       7.8M rows         │  1.4M rows │   1.4M rows │
```

### Split Verification

```sql
-- Verify NO date overlap between splits
SELECT
  'train' as split_name,
  MIN(trip_start_timestamp) as min_date,
  MAX(trip_start_timestamp) as max_date
FROM train_set
UNION ALL
SELECT 'test', MIN(trip_start_timestamp), MAX(trip_start_timestamp)
FROM test_set
UNION ALL
SELECT 'holdout', MIN(trip_start_timestamp), MAX(trip_start_timestamp)
FROM holdout_set;
```

**Results**:
| Split | Min Date | Max Date | Gap |
|-------|----------|----------|-----|
| train | 2022-01-01 | 2023-06-30 23:45 | - |
| test | 2023-07-01 00:00 | 2023-09-30 23:45 | Clean |
| holdout | 2023-10-01 00:00 | 2023-12-31 23:45 | Clean |

---

## Model 1: XGBoost Fare Predictor

### Model Type
BOOSTED_TREE_REGRESSOR

### Purpose
Predict taxi trip fare based on trip characteristics.

### Feature Selection

| Feature | Type | Rationale |
|---------|------|-----------|
| trip_miles | Numeric | Primary fare driver |
| trip_seconds | Numeric | Duration affects fare |
| hour_sin, hour_cos | Numeric | Time of day pricing |
| dow_sin, dow_cos | Numeric | Day of week patterns |
| month_sin, month_cos | Numeric | Seasonal patterns |
| is_weekend | Binary | Weekend pricing |
| is_morning_rush | Binary | Rush hour surcharge |
| is_evening_rush | Binary | Rush hour surcharge |
| is_late_night | Binary | Late night surcharge |
| straight_line_km | Numeric | Trip distance proxy |
| is_airport_pickup | Binary | Airport trips |
| is_airport_dropoff | Binary | Airport trips |
| is_downtown_pickup | Binary | Downtown trips |
| is_downtown_dropoff | Binary | Downtown trips |
| same_area_trip | Binary | Short trips |

### Hyperparameters

```sql
CREATE OR REPLACE MODEL fare_predictor_xgb
OPTIONS(
  model_type = 'BOOSTED_TREE_REGRESSOR',
  input_label_cols = ['target_fare'],

  -- Tree configuration
  booster_type = 'GBTREE',
  max_tree_depth = 8,
  num_parallel_tree = 1,

  -- Learning parameters
  max_iterations = 100,
  learn_rate = 0.1,
  min_split_loss = 0.1,

  -- Regularization
  l1_reg = 0.1,
  l2_reg = 1.0,

  -- Sampling
  subsample = 0.8,
  colsample_bytree = 0.8,
  colsample_bylevel = 0.8,

  -- Early stopping
  early_stop = TRUE,
  min_rel_progress = 0.001,

  -- Validation
  data_split_method = 'CUSTOM',
  data_split_col = 'is_eval',

  -- Explainability
  enable_global_explain = TRUE
)
```

### Hyperparameter Explanation

| Parameter | Value | Purpose |
|-----------|-------|---------|
| max_tree_depth | 8 | Controls tree complexity |
| learn_rate | 0.1 | Step size for updates |
| subsample | 0.8 | Row sampling per tree |
| colsample_bytree | 0.8 | Feature sampling per tree |
| l1_reg | 0.1 | L1 regularization (sparsity) |
| l2_reg | 1.0 | L2 regularization (smoothness) |
| early_stop | TRUE | Prevent overfitting |

### Training Query

```sql
SELECT
  target_fare,
  trip_miles,
  trip_seconds,
  hour_sin, hour_cos,
  dow_sin, dow_cos,
  month_sin, month_cos,
  is_weekend,
  is_morning_rush,
  is_evening_rush,
  is_late_night,
  straight_line_km,
  is_airport_pickup,
  is_airport_dropoff,
  is_downtown_pickup,
  is_downtown_dropoff,
  same_area_trip,
  CASE WHEN RAND() < 0.2 THEN TRUE ELSE FALSE END as is_eval
FROM train_set
WHERE target_fare IS NOT NULL
  AND target_fare BETWEEN 2.5 AND 150
  AND RAND() < 0.5;  -- 50% sample for faster training
```

---

## Model 2: ARIMA_PLUS Demand Forecaster

### Model Type
ARIMA_PLUS (AutoML Time Series)

### Purpose
Forecast hourly trip demand by community area.

### Data Preparation

```sql
-- Create hourly aggregation
CREATE TABLE hourly_demand AS
SELECT
  TIMESTAMP_TRUNC(trip_start_timestamp, HOUR) as hour_timestamp,
  CAST(pickup_community_area AS STRING) as pickup_community_area,
  COUNT(*) as trip_count,
  ROUND(AVG(target_fare), 2) as avg_fare
FROM feature_store
WHERE pickup_community_area IS NOT NULL
GROUP BY 1, 2
HAVING COUNT(*) >= 5;  -- Minimum for signal
```

### Configuration

```sql
CREATE OR REPLACE MODEL demand_forecast_arima
OPTIONS(
  model_type = 'ARIMA_PLUS',

  -- Time series specification
  time_series_timestamp_col = 'hour_timestamp',
  time_series_data_col = 'trip_count',
  time_series_id_col = 'pickup_community_area',

  -- AutoML settings
  auto_arima = TRUE,
  data_frequency = 'HOURLY',

  -- Decomposition
  decompose_time_series = TRUE,

  -- Holiday effects
  holiday_region = 'US',

  -- Data cleaning
  clean_spikes_and_dips = TRUE,
  adjust_step_changes = TRUE
)
```

### Areas Modeled
| Area ID | Name | Rationale |
|---------|------|-----------|
| 8 | Near North Side | High volume |
| 32 | Loop | Downtown core |
| 28 | Near West Side | West Loop |
| 6 | Lake View | Popular area |
| 7 | Lincoln Park | Popular area |
| 76 | O'Hare | Airport traffic |

---

## Model 3: K-Means Driver Segmentation

### Model Type
KMEANS (Unsupervised Clustering)

### Purpose
Segment taxi drivers by behavior patterns for business insights.

### Profile Aggregation

```sql
CREATE TABLE taxi_profiles AS
SELECT
  taxi_id,
  COUNT(*) as total_trips,
  ROUND(AVG(target_fare), 2) as avg_fare,
  ROUND(AVG(trip_miles), 2) as avg_miles,
  ROUND(AVG(trip_seconds) / 60, 1) as avg_duration_mins,
  ROUND(AVG(CAST(is_weekend AS FLOAT64)), 3) as weekend_ratio,
  ROUND(AVG(CAST(is_morning_rush AS FLOAT64)), 3) as morning_rush_ratio,
  ROUND(AVG(CAST(is_evening_rush AS FLOAT64)), 3) as evening_rush_ratio,
  ROUND(AVG(CAST(is_late_night AS FLOAT64)), 3) as late_night_ratio,
  ROUND(AVG(CAST(is_airport_pickup AS FLOAT64)), 3) as airport_ratio,
  ROUND(AVG(CAST(is_downtown_pickup AS FLOAT64)), 3) as downtown_ratio
FROM trips_cleaned t
JOIN feature_store f ON t.unique_key = f.unique_key
WHERE taxi_id IS NOT NULL
GROUP BY taxi_id
HAVING COUNT(*) >= 50;  -- Minimum trips for reliable profile
```

### Configuration

```sql
CREATE OR REPLACE MODEL taxi_segments_kmeans
OPTIONS(
  model_type = 'KMEANS',

  -- Cluster specification
  num_clusters = 5,
  kmeans_init_method = 'KMEANS++',

  -- Training parameters
  max_iterations = 50,
  min_rel_progress = 0.001,

  -- Feature scaling
  standardize_features = TRUE,

  -- Distance metric
  distance_type = 'EUCLIDEAN'
)
```

### Clustering Features
| Feature | Purpose |
|---------|---------|
| avg_fare | Trip value indicator |
| avg_miles | Trip length preference |
| avg_duration_mins | Time investment |
| weekend_ratio | Weekend work pattern |
| morning_rush_ratio | Morning shift preference |
| evening_rush_ratio | Evening shift preference |
| late_night_ratio | Night shift indicator |
| airport_ratio | Airport specialization |
| downtown_ratio | Downtown specialization |

---

## Model 4: Autoencoder Anomaly Detector

### Model Type
AUTOENCODER (Neural Network)

### Purpose
Detect unusual trip patterns via reconstruction error.

### Architecture

```
Input (11 features)
       │
       ▼
    [Dense 32] ◄─ Encoder
       │
       ▼
    [Dense 16]
       │
       ▼
    [Dense 8]  ◄─ Bottleneck (compressed representation)
       │
       ▼
    [Dense 16]
       │
       ▼
    [Dense 32] ◄─ Decoder
       │
       ▼
Output (11 features)
```

### Configuration

```sql
CREATE OR REPLACE MODEL anomaly_detector
OPTIONS(
  model_type = 'AUTOENCODER',

  -- Architecture
  hidden_units = [32, 16, 8, 16, 32],
  activation_fn = 'RELU',

  -- Regularization
  dropout = 0.2,

  -- Training
  batch_size = 256,
  learn_rate = 0.001,
  max_iterations = 50,
  optimizer = 'ADAM',

  -- Early stopping
  early_stop = TRUE,
  min_rel_progress = 0.001
)
```

### Feature Selection
| Feature | Type | Normalization |
|---------|------|---------------|
| trip_miles | Numeric | Standard |
| trip_minutes | Numeric | Standard |
| target_fare | Numeric | Standard |
| straight_line_km | Numeric | Standard |
| route_circuity | Numeric | Standard |
| hour_sin | Numeric | Already normalized |
| hour_cos | Numeric | Already normalized |
| dow_sin | Numeric | Already normalized |
| dow_cos | Numeric | Already normalized |
| is_airport | Binary | 0/1 |
| is_downtown | Binary | 0/1 |

---

## Certification Topics Demonstrated

| Topic | Model | How Demonstrated |
|-------|-------|------------------|
| Supervised Learning | XGBoost | Regression with labeled data |
| Unsupervised Learning | K-Means | Clustering without labels |
| Time Series | ARIMA_PLUS | Forecasting with seasonality |
| Neural Networks | Autoencoder | Deep learning architecture |
| Hyperparameter Tuning | All | OPTIONS configuration |
| Data Leakage Prevention | All | Time-based splits |
| Feature Selection | All | Domain-driven selection |

---

## Navigation

- **Previous**: [Feature Engineering](./04-feature-engineering.md)
- **Next**: [Model Evaluation](./06-model-evaluation.md)
