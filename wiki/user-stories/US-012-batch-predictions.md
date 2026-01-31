# US-012: Batch Predictions

## User Story

**As a** ML Engineer preparing for certification,
**I want to** generate batch predictions from all trained models,
**So that** I can validate model performance and create production outputs.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-012 |
| EPIC | [EPIC-004: Model Evaluation](../epics/EPIC-004-model-evaluation.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Fare predictions generated for test set
- [x] Prediction quality categories assigned
- [x] Demand forecasts generated for 1 week
- [x] Anomaly scores calculated
- [x] Prediction distribution analyzed

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-012-001 | Generate Fare Predictions | Complete |
| TASK-012-002 | Calculate Prediction Quality | Complete |
| TASK-012-003 | Generate Demand Forecasts | Complete |
| TASK-012-004 | Score Anomalies | Complete |
| TASK-012-005 | Summarize Prediction Distribution | Complete |

---

## Fare Predictions

### Quality Distribution

| Quality | Definition | Count | % |
|---------|------------|-------|---|
| Excellent | Error < $2 | 681,439 | **47.8%** |
| Good | Error < $5 | 498,798 | **35.0%** |
| Fair | Error < $10 | 154,952 | 10.9% |
| Poor | Error ≥ $10 | 89,858 | 6.3% |

**Key Finding**: **82.8%** of predictions within $5 of actual fare.

---

## Demand Forecasts

### 1-Week Forecast Summary

| Area | Name | Predicted Trips | Avg/Hour |
|------|------|-----------------|----------|
| 8 | Near North | 29,614 | 176 |
| 32 | Loop | 27,076 | 161 |
| 76 | O'Hare | 22,411 | 133 |
| 28 | Near West | 10,445 | 62 |
| 6 | Lake View | 3,050 | 18 |
| 7 | Lincoln Park | 2,600 | 15 |

**Forecast Horizon**: 168 hours (1 week)
**Confidence Level**: 95%

---

## Anomaly Scores

| Classification | Count | % | Avg MSE |
|----------------|-------|---|---------|
| HIGH_RISK | 28,583 | 57.17% | 0.941 |
| MEDIUM_RISK | 18,972 | 37.94% | 0.351 |
| LOW_RISK | 2,388 | 4.78% | 0.161 |
| NORMAL | 57 | 0.11% | 0.097 |

---

## Output Tables Created

| Table | Records | Purpose |
|-------|---------|---------|
| fare_predictions | 1,425,047 | XGBoost predictions |
| demand_forecast | 1,008 | ARIMA forecasts |
| anomaly_scores | 50,000 | Autoencoder scores |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| ML.PREDICT | Batch prediction function |
| ML.FORECAST | Time series forecasting |
| ML.DETECT_ANOMALIES | Anomaly scoring |
| Prediction Analysis | Quality categorization |

---

## Navigation

- **EPIC**: [EPIC-004: Model Evaluation](../epics/EPIC-004-model-evaluation.md)
- **Previous**: [US-011: Model Evaluation](./US-011-model-evaluation.md)
- **Next**: [US-013: Drift Monitoring](./US-013-drift-monitoring.md)
