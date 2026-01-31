# EPIC-005: MLOps Infrastructure

## Overview

| Field | Value |
|-------|-------|
| EPIC ID | EPIC-005 |
| Title | MLOps Infrastructure |
| Status | Complete |
| User Stories | 2 |
| Total Tasks | 9 |

**Goal**: Implement production-grade monitoring infrastructure including performance tracking, data drift detection, and automated alerting.

**Business Value**: Ensures model quality in production through proactive monitoring and early issue detection.

---

## Scope

- Daily performance tracking
- Z-score based drift detection
- Automated alert thresholds
- Unified monitoring view

## Out of Scope

- Real-time monitoring
- Email/SMS notifications
- Auto-retraining triggers
- Dashboard UI

---

## User Stories

| ID | Title | Tasks | Status |
|----|-------|-------|--------|
| [US-013](../user-stories/US-013-drift-monitoring.md) | Drift Monitoring | 5 | Complete |
| [US-014](../user-stories/US-014-alerting-system.md) | Performance Tracking & Alerting | 4 | Complete |

---

## Acceptance Criteria

- [x] Daily MAE, RMSE, correlation tracked
- [x] Z-scores calculated for key features
- [x] Drift status (OK/WARNING/ALERT) determined
- [x] Performance thresholds defined
- [x] Unified alert view created

---

## Technical Specifications

### Performance Tracking Metrics

| Metric | Calculation | Purpose |
|--------|-------------|---------|
| daily_mae | AVG(absolute_error) | Error magnitude |
| daily_rmse | SQRT(AVG(squared_error)) | Error sensitivity |
| daily_correlation | CORR(actual, predicted) | Prediction quality |
| within_2_dollars_pct | % error < $2 | Threshold accuracy |
| within_5_dollars_pct | % error < $5 | Threshold accuracy |
| within_10_dollars_pct | % error < $10 | Threshold accuracy |

### Drift Monitoring Features

| Feature | Baseline Source | Z-Score Calculation |
|---------|-----------------|---------------------|
| trip_miles | train_set mean/std | \|current - baseline\| / std |
| target_fare | train_set mean/std | \|current - baseline\| / std |
| trip_seconds | train_set mean/std | \|current - baseline\| / std |
| straight_line_km | train_set mean/std | \|current - baseline\| / std |

### Alert Thresholds

| Alert Type | Severity | Condition |
|------------|----------|-----------|
| PERFORMANCE | OK | MAE <= $3.50 |
| PERFORMANCE | WARNING | MAE $3.50 - $5.00 |
| PERFORMANCE | CRITICAL | MAE > $5.00 |
| DATA_DRIFT | OK | All Z-scores < 1.5 |
| DATA_DRIFT | DRIFT_WARNING | Any Z-score 1.5 - 2.0 |
| DATA_DRIFT | DRIFT_ALERT | Any Z-score > 2.0 |

---

## Deliverables

| Deliverable | Type | Location |
|-------------|------|----------|
| performance_tracking | Table | `diamond_analytics.performance_tracking` |
| drift_monitoring | Table | `diamond_analytics.drift_monitoring` |
| model_health_alerts | View | `diamond_analytics.model_health_alerts` |

---

## Metrics Achieved

| Metric | Target | Actual | Status |
|--------|--------|--------|--------|
| Tracking Days | 90+ | 92 | Met |
| Drift Alerts | Functional | 0 (stable) | Met |
| Performance Alerts | Functional | Working | Met |

---

## Monitoring Results

### Performance Summary

| Period | Avg MAE | Within $5 | Status |
|--------|---------|-----------|--------|
| Sep 2023 | $3.62 | 82.5% | WARNING (most days) |

### Drift Summary

| Period | Max Z-Score | Drift Status |
|--------|-------------|--------------|
| Sep 2023 | 0.37 | **OK (no drift)** |

---

## Related Certification Topics

| Topic | Relevance |
|-------|-----------|
| Model Monitoring | Performance tracking |
| Drift Detection | Statistical Z-scores |
| Alerting Systems | Threshold-based alerts |
| MLOps Principles | Continuous monitoring |
| Production ML | Operational awareness |

---

## Dependencies

### Upstream
- [EPIC-004: Model Evaluation](./EPIC-004-model-evaluation.md)
- fare_predictions table
- train_set, test_set tables

### Downstream
- None (terminal EPIC)

---

## Future Enhancements

| Enhancement | Technology | Benefit |
|-------------|------------|---------|
| Scheduled Refresh | Cloud Scheduler | Automated updates |
| Push Notifications | Cloud Functions | Proactive alerts |
| Dashboard UI | Looker Studio | Visualization |
| Auto-Retraining | Vertex AI Pipelines | Self-healing |

---

## Navigation

- **Previous EPIC**: [EPIC-004: Model Evaluation](./EPIC-004-model-evaluation.md)
- **Back to**: [Wiki Home](../README.md)
