# Diamond Analytics - ML Certification Project Wiki

## Project Overview

| Field | Value |
|-------|-------|
| **Project Name** | Diamond Analytics - ML Certification Portfolio |
| **Dataset** | `sonorous-key-320714.diamond_analytics` |
| **Source Data** | `bigquery-public-data.chicago_taxi_trips.taxi_trips` |
| **Date Range** | 2022-01-01 to 2023-12-31 |
| **Total Records** | 10,598,441 cleaned trips |
| **Completion Date** | January 30, 2026 |

---

## Live Dashboard

**[View Dashboard](https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/index.html)** - Real-time ML model performance monitoring

---

## Quick Links

| Document | Description |
|----------|-------------|
| [Project Overview](./01-project-overview.md) | Executive summary and key metrics |
| [Architecture](./02-architecture.md) | System architecture and data flow |
| [Data Engineering](./03-data-engineering.md) | Phase 2 - ETL and data cleaning |
| [Feature Engineering](./04-feature-engineering.md) | Phase 3 - Feature creation |
| [Model Training](./05-model-training.md) | Phase 5 - ML model development |
| [Model Evaluation](./06-model-evaluation.md) | Phase 6 - Performance metrics |
| [MLOps & Monitoring](./07-mlops-monitoring.md) | Phase 8 - Production monitoring |
| [Certification Mapping](./08-certification-mapping.md) | Exam topic coverage |

---

## EPICs

| ID | Title | User Stories | Tasks | Status |
|----|-------|--------------|-------|--------|
| [EPIC-001](./epics/EPIC-001-data-engineering.md) | Data Engineering | 2 | 9 | Complete |
| [EPIC-002](./epics/EPIC-002-feature-engineering.md) | Feature Engineering | 3 | 14 | Complete |
| [EPIC-003](./epics/EPIC-003-model-development.md) | Model Development | 5 | 21 | Complete |
| [EPIC-004](./epics/EPIC-004-model-evaluation.md) | Model Evaluation | 2 | 10 | Complete |
| [EPIC-005](./epics/EPIC-005-mlops-infrastructure.md) | MLOps Infrastructure | 2 | 9 | Complete |
| [EPIC-006](./epics/EPIC-006-dashboard-deployment.md) | Dashboard Deployment | 3 | 7 | Complete |

---

## Project Metrics

### Data Pipeline
| Metric | Value |
|--------|-------|
| Raw Records Profiled | 20,713,994 |
| Cleaned Records | 10,598,441 |
| Data Quality Score | 99.81% |
| Feature Count | 31 |

### Models Trained
| Model | Type | Primary Metric | Value |
|-------|------|----------------|-------|
| fare_predictor_xgb | XGBoost Regressor | R² | **0.913** |
| demand_forecast_arima | ARIMA_PLUS | AIC (Loop) | 66,807 |
| taxi_segments_kmeans | K-Means | Davies-Bouldin | 1.59 |
| anomaly_detector | Autoencoder | Reconstruction Error | Variable |

### Prediction Performance
| Metric | Value |
|--------|-------|
| Mean Absolute Error (MAE) | $3.56 |
| Root Mean Squared Error (RMSE) | $6.48 |
| Predictions within $5 | **82.8%** |
| Predictions within $2 | 47.8% |

### Clustering Results
| Cluster | Name | Size | Avg Fare |
|---------|------|------|----------|
| 1 | Night Owls | 204 | $30.88 |
| 2 | Downtown Weekday Regulars | 477 | $22.20 |
| 3 | Downtown Focus | 768 | $20.59 |
| 4 | Balanced Operators | 1,041 | $31.03 |
| 5 | Airport Specialists | 670 | $43.91 |

---

## User Story Index

| ID | Title | EPIC | Status |
|----|-------|------|--------|
| [US-001](./user-stories/US-001-data-profiling.md) | Data Profiling & Quality Assessment | EPIC-001 | Complete |
| [US-002](./user-stories/US-002-data-cleaning.md) | Create Cleaned Base Table | EPIC-001 | Complete |
| [US-003](./user-stories/US-003-temporal-features.md) | Temporal Feature Engineering | EPIC-002 | Complete |
| [US-004](./user-stories/US-004-geospatial-features.md) | Geospatial Feature Engineering | EPIC-002 | Complete |
| [US-005](./user-stories/US-005-feature-store.md) | Unified Feature Store | EPIC-002 | Complete |
| [US-006](./user-stories/US-006-train-test-split.md) | Train/Test/Holdout Split | EPIC-003 | Complete |
| [US-007](./user-stories/US-007-xgboost-training.md) | XGBoost Fare Predictor | EPIC-003 | Complete |
| [US-008](./user-stories/US-008-arima-training.md) | ARIMA Demand Forecaster | EPIC-003 | Complete |
| [US-009](./user-stories/US-009-kmeans-training.md) | K-Means Driver Segmentation | EPIC-003 | Complete |
| [US-010](./user-stories/US-010-autoencoder-training.md) | Autoencoder Anomaly Detector | EPIC-003 | Complete |
| [US-011](./user-stories/US-011-model-evaluation.md) | Comprehensive Model Evaluation | EPIC-004 | Complete |
| [US-012](./user-stories/US-012-batch-predictions.md) | Batch Predictions | EPIC-004 | Complete |
| [US-013](./user-stories/US-013-drift-monitoring.md) | Drift Monitoring | EPIC-005 | Complete |
| [US-014](./user-stories/US-014-alerting-system.md) | Performance Tracking & Alerting | EPIC-005 | Complete |
| [US-015](./user-stories/US-015-html-dashboard.md) | HTML Dashboard | EPIC-006 | Complete |
| [US-016](./user-stories/US-016-cloud-storage-deployment.md) | Cloud Storage Deployment | EPIC-006 | Complete |
| [US-017](./user-stories/US-017-automated-refresh.md) | Automated Refresh | EPIC-006 | Complete |

---

## Certification Topics Covered

| Topic | Phase | Demonstrated In |
|-------|-------|-----------------|
| Data Pipelines | Phase 2 | EPIC-001, US-001, US-002 |
| Feature Engineering | Phase 3 | EPIC-002, US-003, US-004 |
| Feature Stores | Phase 3 | US-005 |
| Supervised Learning | Phase 5 | US-007 (XGBoost) |
| Unsupervised Learning | Phase 5 | US-009 (K-Means) |
| Time Series | Phase 5 | US-008 (ARIMA_PLUS) |
| Anomaly Detection | Phase 5 | US-010 (Autoencoder) |
| Model Evaluation | Phase 6 | EPIC-004, US-011 |
| Hyperparameter Tuning | Phase 5 | US-007 |
| Data Leakage Prevention | Phase 4 | US-006 |
| Model Monitoring | Phase 8 | EPIC-005, US-013 |
| Drift Detection | Phase 8 | US-013 |

---

## Artifacts Created

### Tables (15)
```
data_quality_log          trips_cleaned            features_temporal
features_geospatial       feature_store            train_set
test_set                  holdout_set              hourly_demand
taxi_profiles             fare_predictions         demand_forecast
anomaly_scores            performance_tracking     drift_monitoring
```

### Views (1)
```
model_health_alerts
```

### Models (4)
```
fare_predictor_xgb        demand_forecast_arima
taxi_segments_kmeans      anomaly_detector
```

---

## How to Use This Wiki

1. **For Learning**: Start with [Project Overview](./01-project-overview.md), then follow each phase in order
2. **For Reference**: Use the User Story Index above to find specific implementations
3. **For Exam Prep**: Review [Certification Mapping](./08-certification-mapping.md) to see topic coverage
4. **For Reproduction**: Follow the Tasks in each User Story for step-by-step instructions

---

## Navigation

- **Next**: [Project Overview](./01-project-overview.md)
