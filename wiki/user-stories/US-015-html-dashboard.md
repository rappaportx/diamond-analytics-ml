# US-015: HTML Dashboard

## User Story

**As a** ML Engineer,
**I want to** have a visual dashboard for model metrics,
**So that** I can quickly assess model health and performance.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-015 |
| EPIC | [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md) |
| Status | Complete |
| Tasks | 2 |

---

## Acceptance Criteria

- [x] Self-contained HTML dashboard created
- [x] Interactive charts with Chart.js
- [x] KPI cards for key metrics
- [x] Drift monitoring gauges
- [x] Cluster profile table
- [x] Auto-refresh capability
- [x] Mobile responsive design

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-015-001 | Create Dashboard HTML | Complete |
| TASK-015-002 | Create Data Export Script | Complete |

---

## Dashboard Features

### KPI Cards

| Card | Metric | Source |
|------|--------|--------|
| Model R2 Score | 0.923 | ML.EVALUATE |
| Mean Absolute Error | $3.17 | ML.EVALUATE |
| Prediction Accuracy | 82.8% | fare_predictions |
| Total Predictions | 1.4M | fare_predictions |

### Charts

| Chart | Type | Data Source |
|-------|------|-------------|
| MAE Trend | Line | performance_tracking |
| Feature Importance | Horizontal Bar | ML.FEATURE_IMPORTANCE |
| Prediction Quality | Doughnut | fare_predictions |

### Drift Gauges

| Gauge | Metric | Threshold |
|-------|--------|-----------|
| Miles Z-Score | Current vs Baseline | 1.5 (warning), 2.0 (alert) |
| Fare Z-Score | Current vs Baseline | 1.5 (warning), 2.0 (alert) |
| Duration Z-Score | Current vs Baseline | 1.5 (warning), 2.0 (alert) |

---

## Technical Details

### Technology Stack

| Component | Technology |
|-----------|------------|
| Charts | Chart.js 4.x |
| Styling | CSS Grid, Flexbox |
| Data Loading | Fetch API |
| Hosting | Cloud Storage |

### Dashboard Files

| File | Size | Purpose |
|------|------|---------|
| index.html | 25 KB | Dashboard UI |
| dashboard_data.json | 14 KB | Data feed |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Model Visualization | Dashboard design |
| JavaScript Integration | Chart.js |
| Responsive Design | Mobile-first |

---

## Navigation

- **EPIC**: [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md)
- **Next**: [US-016: Cloud Storage Deployment](./US-016-cloud-storage-deployment.md)
