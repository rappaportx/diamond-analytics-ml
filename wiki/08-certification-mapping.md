# Certification Exam Topic Mapping

## Overview

This document maps the Diamond Analytics project work to Google Cloud Professional Machine Learning Engineer certification exam topics.

---

## Exam Domain Coverage

### Domain 1: Architecting low-code ML solutions (12%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Developing ML models with BigQuery ML | ✅ Complete | All 4 models trained in BQML |
| Building and using Vertex AI AutoML models | ⬜ Not Covered | Out of scope for this project |
| Building and using pre-built ML APIs | ⬜ Not Covered | Out of scope for this project |

**Project Demonstration**:
- Created XGBoost regression model
- Created ARIMA_PLUS time series model
- Created K-Means clustering model
- Created Autoencoder neural network

---

### Domain 2: Collaborating within and across teams (16%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Methods of data exploration | ✅ Complete | Phase 1 - Data Profiling |
| Feature engineering | ✅ Complete | Phase 3 - Feature Engineering |
| Developing ML models | ✅ Complete | Phase 5 - Model Training |
| Debugging and troubleshooting ML pipelines | ✅ Partial | Error handling in queries |

**Project Demonstration**:
- Comprehensive data profiling
- Cyclical encoding for temporal features
- Haversine distance calculation
- Feature store design

---

### Domain 3: Scaling prototypes into ML models (18%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Building models | ✅ Complete | All 4 models |
| Training models | ✅ Complete | Phase 5 |
| Performing hyperparameter tuning | ✅ Complete | XGBoost configuration |
| Tracking and running ML experiments | ✅ Partial | Evaluation tracking |

**Project Demonstration**:
- 18 hyperparameters configured for XGBoost
- ARIMA AutoML for automatic parameter selection
- K-Means++ initialization
- Early stopping for neural network

---

### Domain 4: Serving and scaling models (19%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Deploying models | ✅ Complete | Models deployed in BigQuery |
| Serving predictions | ✅ Complete | ML.PREDICT and ML.FORECAST |
| Scaling prediction serving | ✅ Partial | BigQuery handles scaling |
| Implementing model versioning | ⬜ Not Covered | Out of scope |

**Project Demonstration**:
- Batch predictions for 1.4M test records
- Demand forecasting for 168 hours
- Anomaly scoring pipeline

---

### Domain 5: Automating and orchestrating ML pipelines (21%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Designing ML pipelines | ✅ Complete | End-to-end data flow |
| Implementing training pipelines | ✅ Complete | Phase 5 |
| Implementing serving pipelines | ✅ Complete | Phase 7 |
| Implementing pipeline automation | ⬜ Partial | Manual execution |

**Project Demonstration**:
- Complete ETL pipeline
- Feature engineering pipeline
- Model training pipeline
- Prediction pipeline

---

### Domain 6: Monitoring ML solutions (14%)

| Topic | Coverage | Project Reference |
|-------|----------|-------------------|
| Identifying risks to ML solutions | ✅ Complete | Drift detection |
| Monitoring, testing, and troubleshooting | ✅ Complete | Phase 8 |
| Monitoring data and model quality | ✅ Complete | Performance tracking |
| Setting up alerts and notifications | ✅ Complete | Alert view |

**Project Demonstration**:
- Z-score based drift monitoring
- Daily performance tracking
- Automated alert thresholds
- Quality classification

---

## Detailed Topic Mapping

### BigQuery ML Functions Used

| Function | Purpose | Phase |
|----------|---------|-------|
| CREATE MODEL | Model training | 5 |
| ML.EVALUATE | Model evaluation | 6 |
| ML.PREDICT | Batch predictions | 7 |
| ML.FORECAST | Time series forecasting | 7 |
| ML.DETECT_ANOMALIES | Anomaly detection | 7 |
| ML.FEATURE_IMPORTANCE | Explainability | 6 |
| ML.ARIMA_EVALUATE | Time series evaluation | 6 |

### Model Types Demonstrated

| Model Type | BigQuery ML Type | Exam Relevance |
|------------|------------------|----------------|
| Regression | BOOSTED_TREE_REGRESSOR | Supervised learning |
| Time Series | ARIMA_PLUS | Forecasting |
| Clustering | KMEANS | Unsupervised learning |
| Anomaly Detection | AUTOENCODER | Neural networks |

### Feature Engineering Techniques

| Technique | Implementation | Exam Topic |
|-----------|----------------|------------|
| Cyclical Encoding | SIN/COS transformation | Feature preprocessing |
| Binary Flags | CASE statements | Categorical features |
| Geospatial Distance | ST_DISTANCE | Domain features |
| Aggregations | AVG, STDDEV | Feature creation |
| NULL Handling | COALESCE | Data quality |

### Data Processing Techniques

| Technique | Implementation | Exam Topic |
|-----------|----------------|------------|
| Partitioning | DATE partition | Cost optimization |
| Clustering | Column clustering | Query performance |
| Data Cleaning | WHERE filters | Data quality |
| Data Validation | Quality log | Data pipelines |

### MLOps Concepts

| Concept | Implementation | Exam Topic |
|---------|----------------|------------|
| Performance Monitoring | Daily metrics table | Model observability |
| Drift Detection | Z-score calculation | Data quality |
| Alerting | Threshold-based view | Operations |
| Baseline Comparison | Train vs inference stats | Monitoring |

---

## Exam Study Guide

### Key Concepts to Review

Based on this project, review these concepts for the exam:

1. **Data Leakage Prevention**
   - Time-based splits for temporal data
   - Proper feature engineering timing

2. **Feature Engineering**
   - Cyclical encoding for time features
   - Geospatial distance calculations
   - Feature store design patterns

3. **Model Selection**
   - When to use regression vs classification
   - Clustering for segmentation
   - Time series for forecasting

4. **Model Evaluation**
   - R², MAE, RMSE for regression
   - Davies-Bouldin for clustering
   - AIC for time series

5. **MLOps**
   - Drift detection methods
   - Performance monitoring metrics
   - Alert threshold design

### Practice Questions

**Q1**: You have hourly sales data and want to predict next week's sales. Which BigQuery ML model type should you use?
- **Answer**: ARIMA_PLUS with data_frequency='HOURLY'

**Q2**: Your model's R² dropped from 0.90 to 0.75 over two weeks. What should you check first?
- **Answer**: Check for data drift using statistical comparison between training and recent data

**Q3**: You have time-based data. How should you split train/test?
- **Answer**: Time-based split where test data is strictly after training data

**Q4**: What's the purpose of using both SIN and COS for encoding hours?
- **Answer**: Creates unique (x,y) coordinates and maintains proximity relationships (hour 23 is close to hour 0)

**Q5**: How do you prevent overfitting in XGBoost?
- **Answer**: Use regularization (l1_reg, l2_reg), subsample, early_stop, and max_tree_depth limits

---

## Coverage Summary

| Domain | Weight | Coverage |
|--------|--------|----------|
| Low-code ML | 12% | ~60% |
| Collaboration | 16% | ~80% |
| Scaling Prototypes | 18% | ~85% |
| Serving Models | 19% | ~70% |
| ML Pipelines | 21% | ~75% |
| Monitoring | 14% | ~90% |
| **Overall** | **100%** | **~76%** |

---

## Recommended Next Steps

### For Certification Preparation

1. **Vertex AI AutoML**: Practice building models with Vertex AI
2. **Kubeflow Pipelines**: Learn pipeline orchestration
3. **TensorFlow Extended**: Understand TFX components
4. **Cloud Functions**: Implement real-time serving
5. **Cloud Monitoring**: Set up production dashboards

### Additional Projects

1. **Image Classification**: Use Vertex AI Vision
2. **NLP Pipeline**: Text classification with AutoML
3. **Real-time Serving**: Deploy model to Vertex AI Endpoints
4. **MLOps Pipeline**: Implement with Vertex AI Pipelines

---

## Navigation

- **Previous**: [MLOps & Monitoring](./07-mlops-monitoring.md)
- **Back to**: [Wiki Home](./README.md)
