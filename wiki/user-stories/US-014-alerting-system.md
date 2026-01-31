# US-014: Performance Tracking & Alerting

## User Story

**As a** ML Engineer preparing for certification,
**I want to** implement performance tracking and automated alerting,
**So that** I can monitor model health and detect degradation.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-014 |
| EPIC | [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md) |
| Status | Complete |
| Tasks | 4 |

---

## Acceptance Criteria

- [x] Daily performance metrics tracked
- [x] Alert thresholds defined
- [x] Unified alert view created
- [x] Recent alerts queryable

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-014-001 | Create Performance Tracking Table | Complete |
| TASK-014-002 | Define Alert Thresholds | Complete |
| TASK-014-003 | Create Model Health Alerts View | Complete |
| TASK-014-004 | Query Recent Alerts | Complete |

---

## Performance Metrics Tracked

| Metric | Calculation | Purpose |
|--------|-------------|---------|
| daily_mae | AVG(absolute_error) | Error magnitude |
| daily_mse | AVG(squared_error) | Squared error |
| daily_rmse | SQRT(daily_mse) | Error with outlier sensitivity |
| daily_correlation | CORR(actual, predicted) | Prediction quality |
| within_2_dollars_pct | % with error < $2 | Excellent accuracy |
| within_5_dollars_pct | % with error < $5 | Good accuracy |
| within_10_dollars_pct | % with error < $10 | Acceptable accuracy |

---

## Alert Thresholds

### Performance Alerts

| Severity | MAE Range | Color | Action |
|----------|-----------|-------|--------|
| OK | ≤ $3.50 | Green | None |
| WARNING | $3.50 - $5.00 | Yellow | Monitor |
| CRITICAL | > $5.00 | Red | Investigate |

### Drift Alerts

| Severity | Z-Score Range | Action |
|----------|---------------|--------|
| OK | < 1.5 | None |
| DRIFT_WARNING | 1.5 - 2.0 | Investigate |
| DRIFT_ALERT | > 2.0 | Consider retraining |

---

## Unified Alert View

```sql
CREATE VIEW model_health_alerts AS
-- Performance alerts
SELECT
  CURRENT_TIMESTAMP() as alert_time,
  'PERFORMANCE' as alert_type,
  prediction_date as alert_date,
  CASE
    WHEN daily_mae > 5 THEN 'CRITICAL'
    WHEN daily_mae > 3.5 THEN 'WARNING'
    ELSE 'OK'
  END as severity,
  CONCAT('Daily MAE: ', CAST(daily_mae AS STRING)) as message
FROM performance_tracking

UNION ALL

-- Drift alerts
SELECT
  CURRENT_TIMESTAMP(),
  'DATA_DRIFT',
  window_date,
  drift_status,
  CONCAT('Z-scores: miles=', miles_zscore, ', fare=', fare_zscore)
FROM drift_monitoring
WHERE drift_status != 'OK';
```

---

## Sample Alert Output

| Alert Type | Date | Severity | Message |
|------------|------|----------|---------|
| PERFORMANCE | 2023-09-30 | WARNING | Daily MAE: 3.52 |
| PERFORMANCE | 2023-09-29 | WARNING | Daily MAE: 3.68 |
| PERFORMANCE | 2023-09-28 | WARNING | Daily MAE: 3.75 |
| PERFORMANCE | 2023-09-27 | OK | Daily MAE: 3.37 |
| PERFORMANCE | 2023-09-26 | OK | Daily MAE: 3.37 |

---

## Alert Summary

| Type | Total Days | OK | WARNING | CRITICAL |
|------|------------|-----|---------|----------|
| PERFORMANCE | 92 | 35 | 57 | 0 |
| DATA_DRIFT | 92 | 92 | 0 | 0 |

**Key Finding**: Model performance stable with occasional warnings, no critical issues or drift detected.

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Model Monitoring | Performance tracking |
| Alert Systems | Threshold-based alerting |
| Unified Views | Combined alert sources |
| MLOps Principles | Proactive monitoring |

---

## Navigation

- **EPIC**: [EPIC-005: MLOps Infrastructure](../epics/EPIC-005-mlops-infrastructure.md)
- **Previous**: [US-013: Drift Monitoring](./US-013-drift-monitoring.md)
- **Back to**: [Wiki Home](../README.md)
