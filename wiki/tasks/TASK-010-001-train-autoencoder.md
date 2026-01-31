# TASK-010-001: Train Autoencoder Model

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-010-001 |
| User Story | [US-010: Anomaly Detection](../user-stories/US-010-anomaly-detection.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Train an Autoencoder neural network for unsupervised anomaly detection in taxi trip data.

---

## Prerequisites

- [x] Feature store with trip-level features
- [x] Understanding of autoencoder architecture
- [x] Training data available

---

## Step-by-Step Instructions

### Step 1: Understand Autoencoders

**What is an Autoencoder?**

```
Input → Encoder → Bottleneck → Decoder → Reconstruction
  [21]     [16]      [8]        [16]         [21]
```

| Component | Purpose |
|-----------|---------|
| Encoder | Compresses input to latent space |
| Bottleneck | Low-dimensional representation |
| Decoder | Reconstructs original input |
| Loss | Measures reconstruction error |

**Anomaly Detection Logic**:
- Normal data: Low reconstruction error
- Anomalies: High reconstruction error

### Step 2: Select Features for Autoencoder

```sql
SELECT
  -- Numeric features for reconstruction
  trip_miles,
  trip_seconds,
  straight_line_km,
  route_efficiency,
  target_fare,
  pickup_downtown_km,
  dropoff_downtown_km,
  pickup_ohare_km,
  dropoff_ohare_km,
  hour_sin,
  hour_cos,
  dow_sin,
  dow_cos
FROM `sonorous-key-320714.diamond_analytics.train_set`
LIMIT 5;
```

### Step 3: Create Autoencoder Model

```sql
CREATE OR REPLACE MODEL `sonorous-key-320714.diamond_analytics.anomaly_detector`
OPTIONS(
  model_type = 'AUTOENCODER',
  activation_fn = 'RELU',
  batch_size = 1024,
  dropout = 0.2,
  hidden_units = [16, 8, 16],
  learn_rate = 0.001,
  max_iterations = 50,
  optimizer = 'ADAM'
) AS
SELECT
  -- All numeric features
  trip_miles,
  trip_seconds,
  straight_line_km,
  route_efficiency,
  target_fare,
  pickup_downtown_km,
  dropoff_downtown_km,
  pickup_ohare_km,
  dropoff_ohare_km,
  pickup_midway_km,
  dropoff_midway_km,
  hour_sin,
  hour_cos,
  dow_sin,
  dow_cos,
  month_sin,
  month_cos
FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

**Option Explanation**:

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | AUTOENCODER | Unsupervised reconstruction |
| activation_fn | RELU | Non-linear activation |
| batch_size | 1024 | Training batch size |
| dropout | 0.2 | Regularization (20%) |
| hidden_units | [16, 8, 16] | Network architecture |
| learn_rate | 0.001 | Learning rate |
| max_iterations | 50 | Training epochs |
| optimizer | ADAM | Adaptive optimizer |

### Step 4: Monitor Training

```sql
SELECT
  iteration,
  loss,
  eval_loss,
  duration_ms
FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.anomaly_detector`)
ORDER BY iteration;
```

**Expected Output**:

| iteration | loss | eval_loss | duration_ms |
|-----------|------|-----------|-------------|
| 1 | 0.892 | 0.901 | 45000 |
| 10 | 0.234 | 0.245 | 38000 |
| 20 | 0.156 | 0.167 | 36000 |
| 30 | 0.134 | 0.148 | 35000 |
| 40 | 0.128 | 0.145 | 35000 |
| 50 | 0.125 | 0.144 | 35000 |

### Step 5: Evaluate Model

```sql
SELECT *
FROM ML.EVALUATE(MODEL `sonorous-key-320714.diamond_analytics.anomaly_detector`);
```

**Expected Output**:

| mean_absolute_error | mean_squared_error | mean_squared_log_error |
|--------------------|-------------------|------------------------|
| 0.125 | 0.032 | 0.018 |

### Step 6: Detect Anomalies

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.trip_anomalies` AS
SELECT
  *,
  CASE
    WHEN mean_squared_error > 0.5 THEN 'HIGH'
    WHEN mean_squared_error > 0.2 THEN 'MEDIUM'
    ELSE 'LOW'
  END as anomaly_level
FROM ML.DETECT_ANOMALIES(
  MODEL `sonorous-key-320714.diamond_analytics.anomaly_detector`,
  STRUCT(0.95 AS contamination),
  (SELECT * FROM `sonorous-key-320714.diamond_analytics.test_set`)
);
```

### Step 7: Analyze Anomalies

```sql
SELECT
  is_anomaly,
  COUNT(*) as trip_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) as pct,
  ROUND(AVG(mean_squared_error), 4) as avg_mse
FROM `sonorous-key-320714.diamond_analytics.trip_anomalies`
GROUP BY is_anomaly;
```

**Expected Output**:

| is_anomaly | trip_count | pct | avg_mse |
|------------|------------|-----|---------|
| FALSE | 2,013,704 | 95.0% | 0.032 |
| TRUE | 105,984 | 5.0% | 0.567 |

---

## Architecture Visualization

```
Input Layer (17 features)
        ↓
Hidden Layer 1 (16 neurons, ReLU, Dropout 0.2)
        ↓
Bottleneck Layer (8 neurons, ReLU)
        ↓
Hidden Layer 2 (16 neurons, ReLU, Dropout 0.2)
        ↓
Output Layer (17 features - reconstruction)
```

---

## Anomaly Analysis

### Step 8: Profile Anomalous Trips

```sql
SELECT
  is_anomaly,
  ROUND(AVG(trip_miles), 2) as avg_miles,
  ROUND(AVG(target_fare), 2) as avg_fare,
  ROUND(AVG(trip_seconds / 60), 1) as avg_minutes,
  ROUND(AVG(route_efficiency), 2) as avg_efficiency
FROM `sonorous-key-320714.diamond_analytics.trip_anomalies`
GROUP BY is_anomaly;
```

**Expected Output**:

| is_anomaly | avg_miles | avg_fare | avg_minutes | avg_efficiency |
|------------|-----------|----------|-------------|----------------|
| FALSE | 3.24 | $17.89 | 14.5 | 1.42 |
| TRUE | 12.67 | $45.23 | 38.2 | 2.15 |

**Insight**: Anomalies tend to be longer trips with higher fares and more indirect routes.

---

## Model Summary

| Metric | Value |
|--------|-------|
| Architecture | 17 → 16 → 8 → 16 → 17 |
| Final Loss | 0.125 |
| Contamination | 5% |
| Anomalies Detected | 105,984 (5%) |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Autoencoder | Neural network for unsupervised learning |
| Reconstruction Error | Measure of abnormality |
| Anomaly Detection | Identifying unusual patterns |
| Architecture Design | Hidden units configuration |

---

## Next Task

[TASK-010-002: Analyze Anomalies](./TASK-010-002-analyze-anomalies.md)
