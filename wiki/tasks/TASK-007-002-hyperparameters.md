# TASK-007-002: Configure XGBoost Hyperparameters

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-007-002 |
| User Story | [US-007: XGBoost Training](../user-stories/US-007-xgboost-training.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Configure optimal hyperparameters for the XGBoost regression model to maximize predictive performance while preventing overfitting.

---

## Prerequisites

- [x] Feature selection completed (TASK-007-001)
- [x] Training data available in train_set
- [x] Understanding of XGBoost hyperparameters

---

## Hyperparameter Reference

### Tree Structure Parameters

| Parameter | Default | Our Value | Purpose |
|-----------|---------|-----------|---------|
| `max_tree_depth` | 6 | **8** | Maximum depth of each tree |
| `min_split_loss` | 0 | **0.1** | Minimum loss reduction for split |
| `num_parallel_tree` | 1 | 1 | Trees per boosting round |

**Explanation**:
- `max_tree_depth = 8`: Allows trees to capture complex interactions without overfitting
- `min_split_loss = 0.1`: Requires meaningful improvement for each split (regularization)

### Learning Parameters

| Parameter | Default | Our Value | Purpose |
|-----------|---------|-----------|---------|
| `max_iterations` | 20 | **100** | Maximum boosting rounds |
| `learn_rate` | 0.3 | **0.1** | Step size for updates |
| `early_stop` | FALSE | **TRUE** | Stop when no improvement |
| `min_rel_progress` | 0.01 | **0.001** | Minimum progress threshold |

**Explanation**:
- `max_iterations = 100`: Enough rounds for convergence
- `learn_rate = 0.1`: Lower rate = more stable learning
- `early_stop = TRUE`: Prevents overfitting automatically
- `min_rel_progress = 0.001`: Sensitive stopping criterion

### Regularization Parameters

| Parameter | Default | Our Value | Purpose |
|-----------|---------|-----------|---------|
| `l1_reg` | 0 | **0.1** | L1 regularization (Lasso) |
| `l2_reg` | 1 | **1.0** | L2 regularization (Ridge) |

**Explanation**:
- `l1_reg = 0.1`: Encourages sparsity (feature selection)
- `l2_reg = 1.0`: Smooths feature weights (default is good)

### Sampling Parameters

| Parameter | Default | Our Value | Purpose |
|-----------|---------|-----------|---------|
| `subsample` | 1.0 | **0.8** | Row sampling per tree |
| `colsample_bytree` | 1.0 | **0.8** | Column sampling per tree |
| `colsample_bylevel` | 1.0 | **0.8** | Column sampling per level |

**Explanation**:
- All set to 0.8: 80% sampling introduces randomness, reduces overfitting
- Acts like "dropout" for tree ensembles

### Validation Parameters

| Parameter | Default | Our Value | Purpose |
|-----------|---------|-----------|---------|
| `data_split_method` | AUTO_SPLIT | **CUSTOM** | How to create eval set |
| `data_split_col` | - | **is_eval** | Column for custom split |

**Explanation**:
- `CUSTOM` split: We control exactly which rows are for evaluation
- 80% training, 20% evaluation within training set

---

## Complete OPTIONS Block

```sql
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
)
```

---

## Hyperparameter Tuning Rationale

### Why max_tree_depth = 8?

| Depth | Pros | Cons |
|-------|------|------|
| 4-6 | Fast, low overfitting | May underfit |
| 8 | **Good balance** | Moderate training time |
| 10+ | Captures complexity | High overfitting risk |

**Decision**: 8 balances complexity and generalization for fare prediction.

### Why learn_rate = 0.1?

| Rate | Pros | Cons |
|------|------|------|
| 0.3 (default) | Fast training | Prone to overshoot |
| 0.1 | **Stable convergence** | More iterations needed |
| 0.01 | Very stable | Very slow training |

**Decision**: 0.1 with 100 iterations provides good convergence.

### Why subsample = 0.8?

| Value | Behavior |
|-------|----------|
| 1.0 | Use all rows (deterministic) |
| 0.8 | **Use 80% random rows** |
| 0.5 | Use 50% random rows |

**Decision**: 0.8 introduces beneficial randomness without losing too much data.

---

## Validation Strategy

### Custom Split Column

```sql
-- Create is_eval column in training query
CASE WHEN RAND() < 0.2 THEN TRUE ELSE FALSE END as is_eval
```

This creates:
- **80%** of rows with `is_eval = FALSE` → Training
- **20%** of rows with `is_eval = TRUE` → Evaluation

### Why Custom Split?

| Method | Description | Use Case |
|--------|-------------|----------|
| NO_SPLIT | All data for training | Not recommended |
| AUTO_SPLIT | Random 80/20 | Simple cases |
| **CUSTOM** | User-defined split | Control needed |
| SEQ | Sequential split | Time series |

**Decision**: CUSTOM gives us explicit control over validation.

---

## Expected Behavior

### Training Progression

```
Iteration 1:  Training Loss = 450.2, Eval Loss = 455.1
Iteration 10: Training Loss = 125.3, Eval Loss = 130.5
Iteration 20: Training Loss = 45.2,  Eval Loss = 48.7
Iteration 30: Training Loss = 42.1,  Eval Loss = 44.2
...
Iteration 50: Training Loss = 41.5,  Eval Loss = 43.8  ← Early stop triggered
```

### Signs of Good Configuration

| Indicator | Good Sign |
|-----------|-----------|
| Training loss decreases | Learning happening |
| Eval loss follows train loss | Not overfitting |
| Gap between train/eval is small | Good generalization |
| Early stopping triggers | Automatic optimization |

---

## Certification Concepts

| Concept | How Demonstrated |
|---------|------------------|
| Hyperparameter Tuning | All OPTIONS configured |
| Regularization | L1, L2, sampling |
| Early Stopping | Prevent overfitting |
| Validation Strategy | Custom train/eval split |
| Ensemble Methods | Boosted trees |

---

## Next Task

[TASK-007-003: Create Training Query](./TASK-007-003-training-query.md)
