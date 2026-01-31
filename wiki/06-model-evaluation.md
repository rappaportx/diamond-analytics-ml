# Model Evaluation Documentation

## Overview

Comprehensive evaluation of all four models using appropriate metrics for each model type. This phase demonstrates proper model evaluation techniques critical for the ML certification exam.

---

## XGBoost Fare Predictor Evaluation

### Regression Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| **R²** | **0.913** | 91.3% of variance explained |
| MAE | $3.56 | Average error magnitude |
| RMSE | $6.48 | Root mean squared error |
| Explained Variance | 0.9134 | Nearly same as R² |
| MSLE | 0.0423 | Log-scale error |

### Evaluation Query

```sql
SELECT
  'XGBoost Fare Predictor' as model_name,
  ROUND(mean_absolute_error, 3) as MAE,
  ROUND(SQRT(mean_squared_error), 3) as RMSE,
  ROUND(r2_score, 4) as R2,
  ROUND(explained_variance, 4) as Explained_Variance,
  ROUND(mean_squared_log_error, 4) as MSLE
FROM ML.EVALUATE(MODEL fare_predictor_xgb,
  (SELECT * FROM test_set
   WHERE target_fare BETWEEN 2.5 AND 150)
);
```

### Target Achievement

| Target | Requirement | Actual | Status |
|--------|-------------|--------|--------|
| R² | > 0.75 | 0.913 | **Exceeded by 21%** |

### Feature Importance

```sql
SELECT
  feature,
  importance_weight,
  importance_gain,
  importance_cover
FROM ML.FEATURE_IMPORTANCE(MODEL fare_predictor_xgb)
ORDER BY importance_weight DESC
LIMIT 10;
```

**Results**:
| Rank | Feature | Weight | Gain | Cover |
|------|---------|--------|------|-------|
| 1 | trip_miles | 2,090 | 977,744 | 134,919 |
| 2 | trip_seconds | 1,991 | 222,978 | 85,880 |
| 3 | straight_line_km | 1,876 | 1,371,769 | 114,073 |
| 4 | hour_cos | 927 | 6,454 | 46,935 |
| 5 | hour_sin | 800 | 5,083 | 16,760 |
| 6 | is_downtown_pickup | 492 | 41,266 | 57,099 |
| 7 | is_airport_pickup | 454 | 134,920 | 108,788 |
| 8 | is_airport_dropoff | 445 | 40,376 | 46,382 |
| 9 | month_sin | 434 | 2,556 | 54,155 |
| 10 | is_downtown_dropoff | 418 | 38,034 | 66,041 |

### Feature Importance Interpretation

- **trip_miles** and **straight_line_km**: Distance is the primary fare driver
- **trip_seconds**: Duration contributes to fare calculation
- **hour_sin/cos**: Time of day affects pricing
- **airport/downtown flags**: Location-based surcharges

### Prediction Quality Distribution

| Quality | Definition | Count | Percentage |
|---------|------------|-------|------------|
| Excellent | Error < $2 | 681,439 | **47.8%** |
| Good | Error < $5 | 498,798 | **35.0%** |
| Fair | Error < $10 | 154,952 | 10.9% |
| Poor | Error >= $10 | 89,858 | 6.3% |

**Key Finding**: **82.8%** of predictions are within $5 of actual fare.

---

## ARIMA_PLUS Demand Forecast Evaluation

### Time Series Metrics

```sql
SELECT
  pickup_community_area,
  ROUND(AIC, 2) as AIC,
  ROUND(variance, 2) as variance,
  non_seasonal_p, non_seasonal_d, non_seasonal_q,
  has_holiday_effect,
  has_spikes_and_dips,
  has_step_changes
FROM ML.ARIMA_EVALUATE(MODEL demand_forecast_arima);
```

**Results**:
| Area | AIC | Variance | Order (p,d,q) | Anomalies |
|------|-----|----------|---------------|-----------|
| 7 | 43,839 | 13.89 | (1,1,1) | Spikes, Steps |
| 6 | 50,319 | 31.19 | (1,1,1) | Spikes, Steps |
| 28 | 62,959 | 150.67 | (1,0,1) | Spikes |
| 32 | 66,807 | 243.87 | (1,1,1) | None |
| 8 | 67,510 | 266.22 | (1,1,1) | None |
| 76 | 72,693 | 508.25 | (1,1,1) | None |

### Metric Interpretation

| Metric | Meaning |
|--------|---------|
| AIC | Akaike Information Criterion (lower is better) |
| Variance | Forecast uncertainty |
| (p,d,q) | ARIMA order parameters |
| has_holiday_effect | Holiday seasonality detected |
| has_spikes_and_dips | Anomalies cleaned |
| has_step_changes | Level shifts adjusted |

### Forecast Validation

| Area | Avg Predicted/Hour | Total (1 Week) |
|------|-------------------|----------------|
| 8 (Near North) | 176 | 29,614 |
| 32 (Loop) | 161 | 27,076 |
| 76 (O'Hare) | 133 | 22,411 |
| 28 (Near West) | 62 | 10,445 |
| 6 (Lake View) | 18 | 3,050 |
| 7 (Lincoln Park) | 15 | 2,600 |

---

## K-Means Clustering Evaluation

### Clustering Metrics

```sql
SELECT
  davies_bouldin_index,
  mean_squared_distance
FROM ML.EVALUATE(MODEL taxi_segments_kmeans);
```

**Results**:
| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Davies-Bouldin Index | 1.59 | < 2.0 | **Good** |
| Mean Squared Distance | 4.50 | - | - |

### Davies-Bouldin Index Interpretation

- **< 1.0**: Excellent cluster separation
- **1.0 - 2.0**: Good cluster separation
- **> 2.0**: Poor cluster separation

### Cluster Profiles

```sql
SELECT
  CENTROID_ID as cluster_id,
  COUNT(*) as cluster_size,
  ROUND(AVG(avg_fare), 2) as avg_fare,
  ROUND(AVG(airport_ratio) * 100, 1) as airport_pct,
  ROUND(AVG(downtown_ratio) * 100, 1) as downtown_pct,
  ROUND(AVG(late_night_ratio) * 100, 1) as late_night_pct
FROM ML.PREDICT(MODEL taxi_segments_kmeans,
  (SELECT * FROM taxi_profiles))
GROUP BY CENTROID_ID
ORDER BY cluster_id;
```

**Results**:
| Cluster | Name | Size | Avg Fare | Airport% | Downtown% | Late Night% |
|---------|------|------|----------|----------|-----------|-------------|
| 1 | **Night Owls** | 204 | $30.88 | 16.7% | 35.8% | **32.3%** |
| 2 | **Downtown Regulars** | 477 | $22.20 | 8.5% | **49.8%** | 3.2% |
| 3 | **Downtown Focus** | 768 | $20.59 | 7.5% | **55.6%** | 3.5% |
| 4 | **Balanced Operators** | 1,041 | $31.03 | 15.9% | 39.0% | 4.3% |
| 5 | **Airport Specialists** | 670 | **$43.91** | **30.8%** | 30.5% | 6.0% |

### Cluster Naming Rationale

| Cluster | Primary Characteristic | Business Insight |
|---------|------------------------|------------------|
| Night Owls | High late_night_ratio (32.3%) | Target for overnight scheduling |
| Downtown Regulars | High downtown, low weekend | Weekday business travelers |
| Downtown Focus | Highest downtown (55.6%) | Core urban service |
| Balanced Operators | Largest group, mixed | Flexible scheduling |
| Airport Specialists | Highest airport, highest fare | Premium segment |

---

## Autoencoder Anomaly Detection Evaluation

### Anomaly Classification

| Classification | Threshold | Count | Percentage |
|----------------|-----------|-------|------------|
| HIGH_RISK | MSE > 0.5 | 28,583 | 57.17% |
| MEDIUM_RISK | MSE > 0.2 | 18,972 | 37.94% |
| LOW_RISK | MSE > 0.1 | 2,388 | 4.78% |
| NORMAL | MSE <= 0.1 | 57 | 0.11% |

### Interpretation

The high proportion of HIGH_RISK classifications indicates:
1. The autoencoder learned "typical" patterns from training data
2. Test data has significant variation from learned patterns
3. Thresholds may need calibration for production use

### Reconstruction Error Analysis

```sql
SELECT
  anomaly_classification,
  ROUND(AVG(mean_squared_error), 4) as avg_mse,
  ROUND(MIN(mean_squared_error), 4) as min_mse,
  ROUND(MAX(mean_squared_error), 4) as max_mse
FROM anomaly_scores
GROUP BY anomaly_classification
ORDER BY avg_mse DESC;
```

---

## Certification Topics Demonstrated

| Topic | Metric Type | How Demonstrated |
|-------|-------------|------------------|
| Regression Evaluation | R², MAE, RMSE | XGBoost evaluation |
| Classification Concepts | Threshold tuning | Anomaly classification |
| Time Series Evaluation | AIC, Variance | ARIMA evaluation |
| Clustering Evaluation | Davies-Bouldin | K-Means evaluation |
| Feature Importance | Weight, Gain, Cover | XGBoost explainability |
| Model Comparison | Multiple metrics | Cross-model analysis |

---

## Key Takeaways

1. **XGBoost Fare Predictor**: Excellent performance (R² = 0.913), distance features most important
2. **ARIMA Demand Forecast**: Successfully models seasonality across 6 areas
3. **K-Means Clustering**: 5 distinct, interpretable driver segments
4. **Autoencoder**: Functional anomaly detection, needs threshold tuning

---

## Navigation

- **Previous**: [Model Training](./05-model-training.md)
- **Next**: [MLOps & Monitoring](./07-mlops-monitoring.md)
