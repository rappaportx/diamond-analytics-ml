# Project Overview - Diamond Analytics

## Executive Summary

The Diamond Analytics project demonstrates end-to-end Machine Learning engineering capabilities using BigQuery ML and the Chicago Taxi public dataset. This project was created for Google Cloud Professional Machine Learning Engineer certification preparation.

---

## Project Goals

1. **Data Engineering**: Transform 20+ million raw taxi trip records into a clean, optimized dataset
2. **Feature Engineering**: Create temporal and geospatial features using ML best practices
3. **Model Development**: Train 4 different model types (supervised, unsupervised, time series, neural network)
4. **Model Evaluation**: Comprehensively evaluate model performance with appropriate metrics
5. **MLOps**: Implement production-grade monitoring and drift detection

---

## Key Achievements

### Data Pipeline
| Metric | Value |
|--------|-------|
| Source Records | 20,713,994 |
| Cleaned Records | 10,598,441 |
| Data Quality Score | 99.81% |
| Processing Efficiency | Partitioned & Clustered |

### Model Performance
| Model | Type | Key Metric | Value | Target | Status |
|-------|------|------------|-------|--------|--------|
| XGBoost Fare Predictor | Regression | R² | 0.913 | >0.75 | **Exceeded** |
| ARIMA Demand Forecast | Time Series | Working | Yes | Yes | **Met** |
| K-Means Clustering | Unsupervised | Davies-Bouldin | 1.59 | <2.0 | **Met** |
| Autoencoder | Anomaly Detection | Working | Yes | Yes | **Met** |

### Business Insights
- **82.8%** of fare predictions within $5 of actual
- **5 distinct driver segments** identified
- **Demand forecasting** operational for 6 key areas
- **Zero drift alerts** - stable data distribution

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DIAMOND ANALYTICS ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐    │
│  │   SOURCE DATA    │     │  DATA CLEANING   │     │  FEATURE STORE   │    │
│  │                  │────▶│                  │────▶│                  │    │
│  │ chicago_taxi_    │     │ trips_cleaned    │     │ feature_store    │    │
│  │ trips (20M+)     │     │ (10.6M rows)     │     │ (31 features)    │    │
│  └──────────────────┘     └──────────────────┘     └──────────────────┘    │
│                                                              │               │
│                                                              ▼               │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                         TRAIN / TEST / HOLDOUT                        │  │
│  │  ┌────────────┐     ┌────────────┐     ┌────────────┐                │  │
│  │  │   TRAIN    │     │    TEST    │     │  HOLDOUT   │                │  │
│  │  │  7.8M rows │     │  1.4M rows │     │  1.4M rows │                │  │
│  │  │ 2022-06/23 │     │ 07-09/2023 │     │ 10-12/2023 │                │  │
│  │  └────────────┘     └────────────┘     └────────────┘                │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                              MODELS                                   │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │  │
│  │  │   XGBoost   │  │   ARIMA+    │  │   K-Means   │  │ Autoencoder │  │  │
│  │  │   (R²=.91)  │  │ (Forecast)  │  │ (5 Clusters)│  │  (Anomaly)  │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                             MLOPS                                     │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐       │  │
│  │  │   Performance   │  │     Drift       │  │     Alerts      │       │  │
│  │  │    Tracking     │  │   Monitoring    │  │      View       │       │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘       │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Dataset Details

### Source: Chicago Taxi Trips
- **Provider**: City of Chicago via BigQuery Public Datasets
- **Table**: `bigquery-public-data.chicago_taxi_trips.taxi_trips`
- **Records**: 200+ million total (20M+ since 2020)
- **Fields**: 23 columns including trip details, fares, locations

### Cleaned Dataset
- **Records**: 10,598,441
- **Date Range**: 2022-01-01 to 2023-12-31
- **Partitioning**: `DATE(trip_start_timestamp)`
- **Clustering**: `company`, `payment_type`

### Cleaning Rules Applied
| Rule | Purpose | Records Removed |
|------|---------|-----------------|
| trip_total > 0 AND < 500 | Remove invalid fares | ~500K |
| trip_miles > 0 AND < 100 | Remove invalid distances | ~2.7M |
| trip_seconds > 60 AND < 14400 | Remove <1min or >4hr trips | ~1M |
| pickup_lat BETWEEN 41.6 AND 42.1 | Chicago bounds only | ~1.3M |
| pickup_lng BETWEEN -88.0 AND -87.5 | Chicago bounds only | Included above |

---

## Models Summary

### 1. XGBoost Fare Predictor
**Purpose**: Predict taxi trip fare based on trip characteristics

| Hyperparameter | Value |
|----------------|-------|
| model_type | BOOSTED_TREE_REGRESSOR |
| max_iterations | 100 |
| learn_rate | 0.1 |
| max_tree_depth | 8 |
| subsample | 0.8 |
| early_stop | TRUE |

**Results**:
- R² = 0.913
- MAE = $3.56
- RMSE = $6.48

### 2. ARIMA_PLUS Demand Forecaster
**Purpose**: Forecast hourly trip demand by community area

| Configuration | Value |
|---------------|-------|
| model_type | ARIMA_PLUS |
| data_frequency | HOURLY |
| auto_arima | TRUE |
| holiday_region | US |
| decompose_time_series | TRUE |

**Coverage**: 6 community areas (8, 32, 28, 6, 7, 76)

### 3. K-Means Driver Segmentation
**Purpose**: Segment taxi drivers by behavior patterns

| Configuration | Value |
|---------------|-------|
| num_clusters | 5 |
| kmeans_init_method | KMEANS++ |
| standardize_features | TRUE |
| distance_type | EUCLIDEAN |

**Clusters Identified**:
1. Night Owls (32% late night)
2. Downtown Weekday Regulars (50% downtown)
3. Downtown Focus (56% downtown)
4. Balanced Operators (largest group)
5. Airport Specialists (31% airport, $44 avg)

### 4. Autoencoder Anomaly Detector
**Purpose**: Detect unusual trip patterns via reconstruction error

| Configuration | Value |
|---------------|-------|
| hidden_units | [32, 16, 8, 16, 32] |
| activation_fn | RELU |
| dropout | 0.2 |
| optimizer | ADAM |

---

## Cost Optimization

### Storage Optimization
- **Partitioning**: Queries scan only relevant date partitions
- **Clustering**: Efficient filtering by company/payment_type

### Training Optimization
- **Sampling**: 50% sample for XGBoost, 10% for Autoencoder
- **Early Stopping**: Prevents unnecessary training iterations

### Query Optimization
- **Feature Store**: Unified table prevents repeated joins
- **Materialized Views**: Pre-computed aggregations

---

## Navigation

- **Previous**: [Wiki Home](./README.md)
- **Next**: [Architecture](./02-architecture.md)
