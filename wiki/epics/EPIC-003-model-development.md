# EPIC-003: Model Development

## Overview

| Field | Value |
|-------|-------|
| EPIC ID | EPIC-003 |
| Title | Model Development |
| Status | Complete |
| User Stories | 5 |
| Total Tasks | 21 |

**Goal**: Train four distinct ML models demonstrating supervised, unsupervised, time series, and neural network approaches using BigQuery ML.

**Business Value**: Creates production-ready models for fare prediction, demand forecasting, driver segmentation, and anomaly detection.

---

## Scope

- Time-based train/test/holdout split
- XGBoost regression for fare prediction
- ARIMA_PLUS for demand forecasting
- K-Means for driver segmentation
- Autoencoder for anomaly detection

## Out of Scope

- Ensemble models
- Deep learning beyond autoencoder
- Real-time model training

---

## User Stories

| ID | Title | Tasks | Status |
|----|-------|-------|--------|
| [US-006](../user-stories/US-006-train-test-split.md) | Train/Test/Holdout Split | 5 | Complete |
| [US-007](../user-stories/US-007-xgboost-training.md) | XGBoost Fare Predictor | 5 | Complete |
| [US-008](../user-stories/US-008-arima-training.md) | ARIMA Demand Forecaster | 5 | Complete |
| [US-009](../user-stories/US-009-kmeans-training.md) | K-Means Driver Segmentation | 4 | Complete |
| [US-010](../user-stories/US-010-autoencoder-training.md) | Autoencoder Anomaly Detector | 4 | Complete |

---

## Acceptance Criteria

- [x] No date overlap in train/test/holdout splits
- [x] XGBoost R² > 0.75
- [x] ARIMA model trained for 6 community areas
- [x] K-Means produces 5 interpretable clusters
- [x] Autoencoder produces reconstruction errors

---

## Technical Specifications

### Data Splits

| Split | Records | Date Range | Percentage |
|-------|---------|------------|------------|
| Train | 7,785,679 | 2022-01-01 to 2023-06-30 | 73% |
| Test | 1,427,550 | 2023-07-01 to 2023-09-30 | 13% |
| Holdout | 1,385,212 | 2023-10-01 to 2023-12-31 | 13% |

### Models Created

| Model | Type | Algorithm | Key Config |
|-------|------|-----------|------------|
| fare_predictor_xgb | Regression | BOOSTED_TREE_REGRESSOR | max_depth=8, lr=0.1 |
| demand_forecast_arima | Time Series | ARIMA_PLUS | auto_arima=TRUE |
| taxi_segments_kmeans | Clustering | KMEANS | k=5, KMEANS++ |
| anomaly_detector | Neural Network | AUTOENCODER | [32,16,8,16,32] |

---

## Deliverables

| Deliverable | Type | Location |
|-------------|------|----------|
| train_set | Table | `diamond_analytics.train_set` |
| test_set | Table | `diamond_analytics.test_set` |
| holdout_set | Table | `diamond_analytics.holdout_set` |
| hourly_demand | Table | `diamond_analytics.hourly_demand` |
| taxi_profiles | Table | `diamond_analytics.taxi_profiles` |
| fare_predictor_xgb | Model | `diamond_analytics.fare_predictor_xgb` |
| demand_forecast_arima | Model | `diamond_analytics.demand_forecast_arima` |
| taxi_segments_kmeans | Model | `diamond_analytics.taxi_segments_kmeans` |
| anomaly_detector | Model | `diamond_analytics.anomaly_detector` |

---

## Metrics Achieved

| Model | Metric | Target | Actual | Status |
|-------|--------|--------|--------|--------|
| XGBoost | R² | >0.75 | 0.913 | **Exceeded** |
| XGBoost | MAE | <$5 | $3.56 | Met |
| K-Means | Davies-Bouldin | <2.0 | 1.59 | Met |
| ARIMA | Trained Areas | 6 | 6 | Met |
| Autoencoder | Functional | Yes | Yes | Met |

---

## Related Certification Topics

| Topic | Relevance |
|-------|-----------|
| Supervised Learning | XGBoost regression |
| Unsupervised Learning | K-Means clustering |
| Time Series | ARIMA_PLUS forecasting |
| Neural Networks | Autoencoder architecture |
| Data Leakage Prevention | Time-based splits |
| Hyperparameter Tuning | Model OPTIONS |

---

## Dependencies

### Upstream
- [EPIC-002: Feature Engineering](./EPIC-002-feature-engineering.md)
- feature_store table

### Downstream
- [EPIC-004: Model Evaluation](./EPIC-004-model-evaluation.md)

---

## Navigation

- **Previous EPIC**: [EPIC-002: Feature Engineering](./EPIC-002-feature-engineering.md)
- **Next EPIC**: [EPIC-004: Model Evaluation](./EPIC-004-model-evaluation.md)
- **Back to**: [Wiki Home](../README.md)
