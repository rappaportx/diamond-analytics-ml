# EPIC-006: Dashboard Deployment

## EPIC Details

| Field | Value |
|-------|-------|
| EPIC ID | EPIC-006 |
| Title | Dashboard Deployment |
| Status | Complete |
| User Stories | 3 |
| Total Tasks | 7 |

---

## Description

Deploy a production-ready, auto-refreshing HTML dashboard for the Diamond Analytics ML project. The dashboard provides real-time visibility into model performance, data drift, and prediction quality.

---

## User Stories

| ID | Title | Status |
|----|-------|--------|
| [US-015](../user-stories/US-015-html-dashboard.md) | HTML Dashboard | Complete |
| [US-016](../user-stories/US-016-cloud-storage-deployment.md) | Cloud Storage Deployment | Complete |
| [US-017](../user-stories/US-017-automated-refresh.md) | Automated Refresh | Complete |

---

## Architecture

```
+------------------+     +-------------------+     +------------------+
|   BigQuery ML    | --> | Cloud Function    | --> | Cloud Storage    |
| - Models         |     | - Data Export     |     | - index.html     |
| - Predictions    |     | - JSON Generation |     | - dashboard_data |
| - Monitoring     |     |                   |     |   .json          |
+------------------+     +-------------------+     +------------------+
                               ^                          |
                               |                          v
                     +-------------------+        +------------------+
                     | Cloud Scheduler   |        | Public Web       |
                     | - Daily 6 AM      |        | - Auto-refresh   |
                     +-------------------+        | - Interactive    |
                                                  +------------------+
```

---

## Dashboard URL

**Live Dashboard**: https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/index.html

---

## Components

### 1. HTML Dashboard (index.html)

| Feature | Description |
|---------|-------------|
| KPI Cards | R2, MAE, Accuracy, Total Predictions |
| MAE Trend Chart | 30-day performance history |
| Feature Importance | Top 10 features bar chart |
| Prediction Quality | Doughnut chart distribution |
| Drift Monitoring | Real-time Z-score gauges |
| Cluster Profiles | K-Means segment analysis |
| Alert Table | Recent model health alerts |

### 2. Data Export (export_dashboard_data.py)

| Query | Source Table | Purpose |
|-------|--------------|---------|
| Model Metrics | ML.EVALUATE | R2, MAE, RMSE |
| Feature Importance | ML.FEATURE_IMPORTANCE | Top features |
| Prediction Quality | fare_predictions | Quality buckets |
| Performance History | performance_tracking | Daily metrics |
| Drift Monitoring | drift_monitoring | Z-scores |
| Cluster Profiles | ML.PREDICT + taxi_profiles | Segments |
| Alerts | model_health_alerts | Health status |

### 3. Cloud Function

| Setting | Value |
|---------|-------|
| Runtime | Python 3.11 |
| Memory | 512 MB |
| Timeout | 300 seconds |
| Trigger | HTTP |
| Region | us-central1 |

### 4. Cloud Scheduler

| Setting | Value |
|---------|-------|
| Schedule | 0 6 * * * (6 AM daily) |
| Timezone | America/Chicago |
| Target | Cloud Function HTTP |

---

## Key Metrics Displayed

| Metric | Current Value | Target |
|--------|---------------|--------|
| R2 Score | 0.923 | > 0.75 |
| MAE | $3.17 | < $5.00 |
| Within $5 Accuracy | 82.8% | > 80% |
| Drift Status | OK | No alerts |
| Predictions | 1,425,047 | - |

---

## Cost Estimate

| Resource | Monthly Cost |
|----------|-------------|
| Cloud Storage | ~$0.10 |
| Cloud Function | ~$0.50 |
| Cloud Scheduler | Free tier |
| **Total** | ~$0.60/month |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| MLOps Visualization | Dashboard for model monitoring |
| Serverless Deployment | Cloud Functions |
| Scheduled Jobs | Cloud Scheduler |
| Static Hosting | Cloud Storage websites |

---

## Navigation

- **Previous**: [EPIC-005: MLOps Infrastructure](./EPIC-005-mlops-infrastructure.md)
- **Back to**: [Wiki Home](../README.md)
