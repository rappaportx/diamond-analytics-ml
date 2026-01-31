# US-008: ARIMA Demand Forecaster

## User Story

**As a** ML Engineer preparing for certification,
**I want to** train an ARIMA_PLUS time series model for demand forecasting,
**So that** I demonstrate time series analysis with seasonality and holiday effects.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-008 |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Hourly demand aggregation created
- [x] Top community areas selected
- [x] ARIMA_PLUS model configured
- [x] Holiday effects enabled (US)
- [x] Time series decomposition enabled
- [x] Model trained for 6 areas

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-008-001 | Create Hourly Demand Aggregation | Complete |
| TASK-008-002 | Select Top Community Areas | Complete |
| TASK-008-003 | Configure ARIMA_PLUS Options | Complete |
| TASK-008-004 | Execute Model Training | Complete |
| TASK-008-005 | Verify Time Series Decomposition | Complete |

---

## Model Configuration

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | ARIMA_PLUS | Time series with extensions |
| time_series_timestamp_col | hour_timestamp | Time column |
| time_series_data_col | trip_count | Value to forecast |
| time_series_id_col | pickup_community_area | Multiple series |
| auto_arima | TRUE | Automatic parameter selection |
| data_frequency | HOURLY | Time granularity |
| decompose_time_series | TRUE | Trend/seasonal decomposition |
| holiday_region | US | US holiday calendar |
| clean_spikes_and_dips | TRUE | Outlier handling |
| adjust_step_changes | TRUE | Level shift handling |

---

## Areas Modeled

| Area ID | Name | Avg Trips/Hour | AIC |
|---------|------|----------------|-----|
| 8 | Near North Side | 176 | 67,510 |
| 32 | Loop | 161 | 66,807 |
| 76 | O'Hare Airport | 133 | 72,693 |
| 28 | Near West Side | 62 | 62,959 |
| 6 | Lake View | 18 | 50,319 |
| 7 | Lincoln Park | 15 | 43,839 |

---

## ARIMA Orders Selected

| Area | (p,d,q) | Interpretation |
|------|---------|----------------|
| 28 | (1,0,1) | Stationary, MA(1) |
| All others | (1,1,1) | Differenced, AR(1)+MA(1) |

---

## Actual Results

| Metric | Value |
|--------|-------|
| Models Trained | 6 (one per area) |
| Forecast Horizon | 168 hours (1 week) |
| Holiday Effects | Detected in 0 areas |
| Spikes/Dips | Detected in 4 areas |
| Step Changes | Detected in 2 areas |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Time Series Forecasting | ARIMA_PLUS model |
| Seasonality | Hourly patterns |
| Holiday Effects | US calendar integration |
| Multiple Time Series | One model, multiple areas |
| AutoML | Automatic parameter selection |

---

## Navigation

- **EPIC**: [EPIC-003: Model Development](../epics/EPIC-003-model-development.md)
- **Previous**: [US-007: XGBoost Training](./US-007-xgboost-training.md)
- **Next**: [US-009: K-Means Training](./US-009-kmeans-training.md)
