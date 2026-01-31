# TASK-007-003: Create XGBoost Training Query

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-007-003 |
| User Story | [US-007: XGBoost Training](../user-stories/US-007-xgboost-training.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Execute the complete XGBoost model training query with selected features and optimized hyperparameters.

---

## Prerequisites

- [x] Features selected (TASK-007-001)
- [x] Hyperparameters configured (TASK-007-002)
- [x] Training data with is_eval column

---

## Step-by-Step Instructions

### Step 1: Final Pre-Training Check

Verify training data is ready:

```sql
SELECT
  COUNTIF(is_eval = FALSE) as training_rows,
  COUNTIF(is_eval = TRUE) as validation_rows,
  ROUND(COUNTIF(is_eval = TRUE) * 100.0 / COUNT(*), 1) as validation_pct
FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

**Expected Output**:

| training_rows | validation_rows | validation_pct |
|---------------|-----------------|----------------|
| ~4,240,000 | ~1,060,000 | 20.0% |

### Step 2: Execute Model Training

```sql
CREATE OR REPLACE MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`
OPTIONS(
  -- Model type
  model_type = 'BOOSTED_TREE_REGRESSOR',
  input_label_cols = ['target_fare'],

  -- Tree configuration
  booster_type = 'GBTREE',
  max_tree_depth = 8,
  min_split_loss = 0.1,
  num_parallel_tree = 1,

  -- Learning parameters
  max_iterations = 100,
  learn_rate = 0.1,

  -- Regularization
  l1_reg = 0.1,
  l2_reg = 1.0,

  -- Sampling (stochastic gradient boosting)
  subsample = 0.8,
  colsample_bytree = 0.8,
  colsample_bylevel = 0.8,

  -- Early stopping
  early_stop = TRUE,
  min_rel_progress = 0.001,

  -- Validation split
  data_split_method = 'CUSTOM',
  data_split_col = 'is_eval',

  -- Explainability
  enable_global_explain = TRUE
) AS
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

  -- Boolean features (cast to enable numeric processing)
  CAST(is_weekend AS INT64) as is_weekend,
  CAST(is_rush_hour AS INT64) as is_rush_hour,
  CAST(is_night AS INT64) as is_night,
  CAST(is_downtown_pickup AS INT64) as is_downtown_pickup,
  CAST(is_airport_trip AS INT64) as is_airport_trip,

  -- Target variable
  target_fare,

  -- Split indicator
  is_eval

FROM `sonorous-key-320714.diamond_analytics.train_set`;
```

### Step 3: Monitor Training Progress

Training typically takes 5-15 minutes. Check status:

```sql
SELECT
  training_run,
  iteration,
  loss,
  eval_loss,
  duration_ms
FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`)
ORDER BY iteration;
```

**Expected Output** (sample):

| training_run | iteration | loss | eval_loss | duration_ms |
|--------------|-----------|------|-----------|-------------|
| 0 | 1 | 145.23 | 148.56 | 12000 |
| 0 | 10 | 25.67 | 27.12 | 8500 |
| 0 | 20 | 12.45 | 13.89 | 8200 |
| 0 | 30 | 10.23 | 11.67 | 8100 |
| 0 | 40 | 9.87 | 11.34 | 8000 |
| 0 | 50 | 9.65 | 11.21 | 7900 |

### Step 4: Verify Model Creation

```sql
SELECT
  model_type,
  training_run,
  iteration,
  loss,
  eval_loss
FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`)
WHERE iteration = (
  SELECT MAX(iteration)
  FROM ML.TRAINING_INFO(MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`)
);
```

### Step 5: Check Model Metadata

```sql
SELECT *
FROM ML.MODEL_INFO(MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`);
```

---

## Expected Training Behavior

### Early Stopping

If early stopping triggers:

```
Iteration 52: Early stopping triggered - no improvement for 5 iterations
Final iteration: 52 (of max 100)
```

This is good! It means the model found optimal complexity.

### Convergence Pattern

| Phase | Iterations | Loss Behavior |
|-------|------------|---------------|
| Rapid learning | 1-10 | Loss drops quickly |
| Refinement | 10-30 | Loss decreases slowly |
| Convergence | 30-50 | Loss stabilizes |
| Early stop | 50+ | Training ends |

### Warning Signs

| Issue | Symptom | Action |
|-------|---------|--------|
| Overfitting | eval_loss >> train_loss | Increase regularization |
| Underfitting | Both losses high | Decrease regularization |
| No convergence | Loss fluctuating | Lower learn_rate |

---

## Training Cost Estimate

| Resource | Estimate |
|----------|----------|
| Data processed | ~5.3M rows x 21 features |
| Training iterations | ~50-100 |
| Duration | 5-15 minutes |
| Cost | ~$5-10 (BQML pricing) |

---

## Post-Training Validation

After training completes:

```sql
-- Quick evaluation
SELECT *
FROM ML.EVALUATE(MODEL `sonorous-key-320714.diamond_analytics.fare_predictor_xgb`);
```

**Expected Output**:

| mean_absolute_error | mean_squared_error | r2_score |
|--------------------|-------------------|----------|
| ~$3.12 | ~15.67 | ~0.913 |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Model already exists" | Previous training | Use CREATE OR REPLACE |
| "Out of memory" | Too much data | Increase sampling |
| Training too slow | Complex model | Reduce max_iterations |
| Poor evaluation metrics | Bad features | Review feature selection |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| CREATE MODEL | BigQuery ML syntax |
| OPTIONS | Hyperparameter specification |
| CUSTOM split | User-defined train/eval |
| enable_global_explain | Feature importance |
| Early stopping | Automatic overfitting prevention |

---

## Next Task

[TASK-007-004: Verify Model Training](./TASK-007-004-verify-training.md)
