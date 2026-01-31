# US-017: Automated Refresh

## User Story

**As a** ML Engineer,
**I want to** have the dashboard data refresh automatically,
**So that** I always see current model performance metrics.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-017 |
| EPIC | [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md) |
| Status | Complete |
| Tasks | 3 |

---

## Acceptance Criteria

- [x] Cloud Function created for data export
- [x] Function queries all BigQuery sources
- [x] Function uploads JSON to Cloud Storage
- [x] Cloud Scheduler job configured
- [x] Daily refresh at 6 AM Central

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-017-001 | Create Cloud Function | Complete |
| TASK-017-002 | Deploy Cloud Function | Complete |
| TASK-017-003 | Configure Cloud Scheduler | Complete |

---

## Cloud Function Details

### Configuration

| Setting | Value |
|---------|-------|
| Name | refresh-ml-dashboard |
| Runtime | Python 3.11 |
| Memory | 512 MB |
| Timeout | 300 seconds |
| Entry Point | refresh_dashboard |
| Trigger | HTTP |
| Region | us-central1 |

### Function URL

```
https://us-central1-sonorous-key-320714.cloudfunctions.net/refresh-ml-dashboard
```

### Data Queries

| Query | Source | Data Exported |
|-------|--------|---------------|
| Model Metrics | ML.EVALUATE | R2, MAE, RMSE |
| Feature Importance | ML.FEATURE_IMPORTANCE | Top 10 features |
| Prediction Quality | fare_predictions | Quality distribution |
| Performance History | performance_tracking | 30-day history |
| Drift Monitoring | drift_monitoring | Z-scores |
| Cluster Profiles | ML.PREDICT | 5 segments |
| Alerts | model_health_alerts | Recent 20 |

---

## Cloud Scheduler Details

### Job Configuration

| Setting | Value |
|---------|-------|
| Name | ml-dashboard-daily-refresh |
| Schedule | 0 6 * * * |
| Timezone | America/Chicago |
| Target | HTTP GET |
| Location | us-central1 |

### Schedule Explanation

```
0 6 * * *
| | | | |
| | | | +-- Day of week (any)
| | | +---- Month (any)
| | +------ Day of month (any)
| +-------- Hour (6 AM)
+---------- Minute (0)
```

---

## Manual Trigger

To manually refresh the dashboard:

```bash
curl https://us-central1-sonorous-key-320714.cloudfunctions.net/refresh-ml-dashboard
```

Expected response:
```
Dashboard refreshed at 2026-01-30T18:53:40.592283 with 1,425,047 predictions
```

---

## Cost Analysis

| Resource | Monthly Estimate |
|----------|-----------------|
| Cloud Function (30 invocations) | ~$0.01 |
| BigQuery queries | ~$0.30 |
| Cloud Scheduler | Free tier |
| **Total** | ~$0.31/month |

---

## Monitoring

### Logs

View function logs:
```bash
gcloud functions logs read refresh-ml-dashboard --region us-central1 --limit 20
```

### Scheduler History

View job runs:
```bash
gcloud scheduler jobs describe ml-dashboard-daily-refresh --location us-central1
```

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Cloud Functions | Serverless compute |
| Cloud Scheduler | Cron-style scheduling |
| Event-Driven Architecture | Automated workflows |
| MLOps Automation | Continuous monitoring |

---

## Navigation

- **EPIC**: [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md)
- **Previous**: [US-016: Cloud Storage Deployment](./US-016-cloud-storage-deployment.md)
- **Back to**: [Wiki Home](../README.md)
