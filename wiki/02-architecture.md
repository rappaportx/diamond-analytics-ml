# System Architecture & Data Flow

## Overview

The Diamond Analytics platform follows a modular architecture designed for ML certification demonstration. Each component is independently testable and maps to specific certification exam topics.

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              GCP PROJECT                                         │
│                         sonorous-key-320714                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │                         BigQuery Dataset                                 │   │
│   │                      diamond_analytics (US)                              │   │
│   │                                                                          │   │
│   │   ┌─────────────────────────────────────────────────────────────────┐   │   │
│   │   │                    DATA LAYER (15 Tables)                        │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │data_quality_ │  │trips_cleaned │  │features_     │          │   │   │
│   │   │  │log           │  │(Partitioned) │  │temporal      │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │features_     │  │feature_store │  │train_set     │          │   │   │
│   │   │  │geospatial    │  │(Partitioned) │  │              │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │test_set      │  │holdout_set   │  │hourly_demand │          │   │   │
│   │   │  │              │  │              │  │              │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │taxi_profiles │  │fare_         │  │demand_       │          │   │   │
│   │   │  │              │  │predictions   │  │forecast      │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │anomaly_      │  │performance_  │  │drift_        │          │   │   │
│   │   │  │scores        │  │tracking      │  │monitoring    │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   └─────────────────────────────────────────────────────────────────┘   │   │
│   │                                                                          │   │
│   │   ┌─────────────────────────────────────────────────────────────────┐   │   │
│   │   │                    MODEL LAYER (4 Models)                        │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │   │   │
│   │   │  │fare_         │  │demand_       │  │taxi_segments_│          │   │   │
│   │   │  │predictor_xgb │  │forecast_     │  │kmeans        │          │   │   │
│   │   │  │(XGBoost)     │  │arima         │  │(K-Means)     │          │   │   │
│   │   │  └──────────────┘  └──────────────┘  └──────────────┘          │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐                                               │   │   │
│   │   │  │anomaly_      │                                               │   │   │
│   │   │  │detector      │                                               │   │   │
│   │   │  │(Autoencoder) │                                               │   │   │
│   │   │  └──────────────┘                                               │   │   │
│   │   └─────────────────────────────────────────────────────────────────┘   │   │
│   │                                                                          │   │
│   │   ┌─────────────────────────────────────────────────────────────────┐   │   │
│   │   │                    VIEW LAYER (1 View)                           │   │   │
│   │   │                                                                  │   │   │
│   │   │  ┌──────────────┐                                               │   │   │
│   │   │  │model_health_ │                                               │   │   │
│   │   │  │alerts        │                                               │   │   │
│   │   │  └──────────────┘                                               │   │   │
│   │   └─────────────────────────────────────────────────────────────────┘   │   │
│   │                                                                          │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagram

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                              DATA FLOW                                            │
└──────────────────────────────────────────────────────────────────────────────────┘

                    PHASE 1: DATA PROFILING
                    ┌─────────────────────┐
                    │  bigquery-public-   │
                    │  data.chicago_taxi_ │
                    │  trips.taxi_trips   │
                    │     (20.7M rows)    │
                    └─────────┬───────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │  data_quality_log   │
                    │  (Quality Metrics)  │
                    └─────────────────────┘

                    PHASE 2: DATA CLEANING
                              │
                              ▼
                    ┌─────────────────────┐
                    │   trips_cleaned     │◀── Partitioned by DATE
                    │    (10.6M rows)     │◀── Clustered by company, payment_type
                    │    + derived cols   │
                    └─────────┬───────────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
    PHASE 3: FEATURE ENGINEERING
    ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
    │ features_   │   │ features_   │   │   (Other    │
    │ temporal    │   │ geospatial  │   │  Features)  │
    │ (Cyclical)  │   │ (Haversine) │   │             │
    └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
           │                 │                 │
           └─────────────────┼─────────────────┘
                             │
                             ▼
                    ┌─────────────────────┐
                    │    feature_store    │◀── Unified Feature Table
                    │    (31 features)    │◀── Partitioned & Clustered
                    └─────────┬───────────┘
                              │
    PHASE 4: TRAIN/TEST SPLIT │ (TIME-BASED - NO OVERLAP)
            ┌─────────────────┼─────────────────┐
            │                 │                 │
            ▼                 ▼                 ▼
    ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
    │  train_set  │   │  test_set   │   │holdout_set  │
    │   (7.8M)    │   │   (1.4M)    │   │   (1.4M)    │
    │ 2022-06/23  │   │ 07-09/2023  │   │ 10-12/2023  │
    └──────┬──────┘   └──────┬──────┘   └─────────────┘
           │                 │
    PHASE 5: MODEL TRAINING  │
           │                 │
           ▼                 │
    ┌─────────────────┐      │
    │     MODELS      │      │
    │ ┌─────────────┐ │      │
    │ │   XGBoost   │ │      │
    │ │  (fare_     │ │      │
    │ │ predictor)  │ │      │
    │ └─────────────┘ │      │
    │ ┌─────────────┐ │      │
    │ │   ARIMA+    │ │      │
    │ │  (demand_   │ │      │
    │ │  forecast)  │ │      │
    │ └─────────────┘ │      │
    │ ┌─────────────┐ │      │
    │ │   K-Means   │ │      │
    │ │   (taxi_    │ │      │
    │ │  segments)  │ │      │
    │ └─────────────┘ │      │
    │ ┌─────────────┐ │      │
    │ │ Autoencoder │ │      │
    │ │  (anomaly_  │ │      │
    │ │  detector)  │ │      │
    │ └─────────────┘ │      │
    └────────┬────────┘      │
             │               │
    PHASE 6: EVALUATION      │
             │               │
             ▼               ▼
    ┌─────────────────────────────┐
    │       ML.EVALUATE()         │
    │       ML.PREDICT()          │
    │   ML.FEATURE_IMPORTANCE()   │
    └─────────────┬───────────────┘
                  │
    PHASE 7: PREDICTIONS
                  │
    ┌─────────────┼─────────────────┐
    │             │                 │
    ▼             ▼                 ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│  fare_   │ │ demand_  │ │ anomaly_ │
│predictions│ │ forecast │ │  scores  │
└──────────┘ └──────────┘ └──────────┘
                  │
    PHASE 8: MLOPS
                  │
    ┌─────────────┼─────────────────┐
    │             │                 │
    ▼             ▼                 ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│performance│ │  drift_  │ │model_    │
│ _tracking │ │monitoring│ │health_   │
│           │ │          │ │alerts    │
└──────────┘ └──────────┘ └──────────┘
```

---

## Table Schemas

### trips_cleaned
```sql
CREATE TABLE trips_cleaned (
  unique_key STRING,
  taxi_id STRING,
  trip_start_timestamp TIMESTAMP,
  trip_end_timestamp TIMESTAMP,
  trip_seconds INT64,
  trip_miles FLOAT64,
  pickup_census_tract STRING,
  dropoff_census_tract STRING,
  pickup_community_area INT64,
  dropoff_community_area INT64,
  fare FLOAT64,
  tips FLOAT64,
  tolls FLOAT64,
  extras FLOAT64,
  trip_total FLOAT64,
  payment_type STRING,
  company STRING,
  pickup_latitude FLOAT64,
  pickup_longitude FLOAT64,
  dropoff_latitude FLOAT64,
  dropoff_longitude FLOAT64,
  -- Derived columns
  pickup_hour INT64,
  pickup_dow INT64,
  pickup_month INT64,
  pickup_year INT64,
  day_type STRING,
  avg_speed_mph FLOAT64
)
PARTITION BY DATE(trip_start_timestamp)
CLUSTER BY company, payment_type;
```

### feature_store
```sql
CREATE TABLE feature_store (
  unique_key STRING,
  trip_start_timestamp TIMESTAMP,
  target_fare FLOAT64,
  trip_miles FLOAT64,
  trip_seconds INT64,
  fare FLOAT64,
  tips FLOAT64,
  payment_type STRING,
  company STRING,
  pickup_community_area INT64,
  dropoff_community_area INT64,
  avg_speed_mph FLOAT64,
  -- Temporal features (14)
  hour_sin FLOAT64,
  hour_cos FLOAT64,
  dow_sin FLOAT64,
  dow_cos FLOAT64,
  month_sin FLOAT64,
  month_cos FLOAT64,
  is_weekend INT64,
  is_morning_rush INT64,
  is_evening_rush INT64,
  is_late_night INT64,
  is_summer INT64,
  is_winter INT64,
  -- Geospatial features (7)
  straight_line_km FLOAT64,
  route_circuity FLOAT64,
  is_airport_pickup INT64,
  is_airport_dropoff INT64,
  is_downtown_pickup INT64,
  is_downtown_dropoff INT64,
  same_area_trip INT64,
  -- Metadata
  feature_created_at TIMESTAMP
)
PARTITION BY DATE(trip_start_timestamp)
CLUSTER BY pickup_community_area;
```

---

## API Dependencies

| API | Purpose | Status |
|-----|---------|--------|
| bigquery.googleapis.com | Core BigQuery operations | Enabled |
| bigquerydatatransfer.googleapis.com | Data transfer service | Enabled |
| storage.googleapis.com | GCS for exports | Enabled |

---

## Security Configuration

| Component | Setting |
|-----------|---------|
| Dataset Location | US (multi-region) |
| Default Table Expiration | None |
| Access Control | IAM-based |
| Encryption | Google-managed keys |

---

## Navigation

- **Previous**: [Project Overview](./01-project-overview.md)
- **Next**: [Data Engineering](./03-data-engineering.md)
