# US-007: XGBoost Fare Predictor

## User Story

**As a** ML Engineer preparing for certification,
**I want to** train an XGBoost regression model for fare prediction,
**So that** I demonstrate supervised learning with hyperparameter tuning.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-007 |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Features selected for training
- [x] Hyperparameters configured
- [x] 50% sampling for training efficiency
- [x] 80/20 train/eval split within training data
- [x] Model trained with early stopping
- [x] R² > 0.75 on test set

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-007-001](../tasks/TASK-007-001-feature-selection.md) | Select Features for Training | Complete |
| [TASK-007-002](../tasks/TASK-007-002-hyperparameters.md) | Configure Hyperparameters | Complete |
| [TASK-007-003](../tasks/TASK-007-003-training-query.md) | Create Training Query | Complete |
| [TASK-007-004](../tasks/TASK-007-004-execute-training.md) | Execute Model Training | Complete |
| [TASK-007-005](../tasks/TASK-007-005-monitor-training.md) | Monitor Training Progress | Complete |

---

## Model Configuration

### Hyperparameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| model_type | BOOSTED_TREE_REGRESSOR | XGBoost regression |
| booster_type | GBTREE | Gradient boosted trees |
| max_iterations | 100 | Maximum boosting rounds |
| learn_rate | 0.1 | Step size |
| max_tree_depth | 8 | Tree complexity |
| min_split_loss | 0.1 | Minimum loss reduction |
| subsample | 0.8 | Row sampling |
| colsample_bytree | 0.8 | Column sampling per tree |
| colsample_bylevel | 0.8 | Column sampling per level |
| l1_reg | 0.1 | L1 regularization |
| l2_reg | 1.0 | L2 regularization |
| early_stop | TRUE | Stop when no improvement |
| min_rel_progress | 0.001 | Minimum progress threshold |

### Features Used (18)

| Category | Features |
|----------|----------|
| Distance | trip_miles, trip_seconds, straight_line_km |
| Temporal | hour_sin, hour_cos, dow_sin, dow_cos, month_sin, month_cos |
| Binary | is_weekend, is_morning_rush, is_evening_rush, is_late_night |
| Location | is_airport_pickup, is_airport_dropoff, is_downtown_pickup, is_downtown_dropoff, same_area_trip |

---

## Actual Results

| Metric | Value |
|--------|-------|
| R² | **0.913** |
| MAE | $3.56 |
| RMSE | $6.48 |
| Explained Variance | 0.9134 |
| Training Time | ~40 minutes |

### Target Achievement

| Target | Required | Actual | Status |
|--------|----------|--------|--------|
| R² | > 0.75 | 0.913 | **Exceeded by 21%** |

---

## Feature Importance

| Rank | Feature | Weight |
|------|---------|--------|
| 1 | trip_miles | 2,090 |
| 2 | trip_seconds | 1,991 |
| 3 | straight_line_km | 1,876 |
| 4 | hour_cos | 927 |
| 5 | hour_sin | 800 |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Supervised Learning | Regression with labeled data |
| Hyperparameter Tuning | OPTIONS configuration |
| Regularization | L1, L2, dropout equivalents |
| Early Stopping | Prevent overfitting |
| Feature Selection | Domain-driven selection |

---

## Navigation

- **EPIC**: [EPIC-003: Model Development](../epics/EPIC-003-model-development.md)
- **Previous**: [US-006: Train/Test Split](./US-006-train-test-split.md)
- **Next**: [US-008: ARIMA Training](./US-008-arima-training.md)
