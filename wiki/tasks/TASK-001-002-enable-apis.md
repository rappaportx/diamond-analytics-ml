# TASK-001-002: Enable Required APIs

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-001-002 |
| User Story | [US-001: Data Profiling](../user-stories/US-001-data-profiling.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Enable the BigQuery API and BigQuery Connection API in the GCP project to allow ML model training and data operations.

---

## Prerequisites

- [x] GCP authentication verified (TASK-001-001)
- [x] Project owner or editor role
- [x] Access to GCP Console or gcloud CLI

---

## Step-by-Step Instructions

### Step 1: Check Current API Status

List currently enabled APIs:

```bash
gcloud services list --enabled --project=sonorous-key-320714 | grep -i bigquery
```

**Expected Output** (if already enabled):
```
bigquery.googleapis.com                BigQuery API
bigqueryconnection.googleapis.com      BigQuery Connection API
```

### Step 2: Enable BigQuery API

If not already enabled, run:

```bash
gcloud services enable bigquery.googleapis.com --project=sonorous-key-320714
```

**Expected Output**:
```
Operation "operations/acf.p2-PROJECT_NUMBER-..." finished successfully.
```

### Step 3: Enable BigQuery Connection API

Required for external connections and some ML features:

```bash
gcloud services enable bigqueryconnection.googleapis.com --project=sonorous-key-320714
```

### Step 4: Enable BigQuery Storage API (Optional)

For faster data reads:

```bash
gcloud services enable bigquerystorage.googleapis.com --project=sonorous-key-320714
```

### Step 5: Verify All APIs Enabled

```bash
gcloud services list --enabled --project=sonorous-key-320714 --filter="name:bigquery"
```

**Expected Output**:
```
NAME                                   TITLE
bigquery.googleapis.com                BigQuery API
bigqueryconnection.googleapis.com      BigQuery Connection API
bigquerystorage.googleapis.com         BigQuery Storage API
```

---

## Alternative: Enable via Console

### Step 1: Navigate to APIs & Services

1. Go to: https://console.cloud.google.com/apis/library
2. Ensure project `sonorous-key-320714` is selected

### Step 2: Search and Enable

1. Search for "BigQuery API"
2. Click on the result
3. Click "ENABLE" button
4. Repeat for "BigQuery Connection API"

---

## API Reference

| API | Purpose | Required |
|-----|---------|----------|
| BigQuery API | Core query and ML operations | Yes |
| BigQuery Connection API | External data sources | Recommended |
| BigQuery Storage API | Fast data reads | Optional |
| BigQuery Data Transfer API | Scheduled transfers | Optional |

---

## Verification Checklist

| Check | Command | Expected |
|-------|---------|----------|
| BigQuery API | `gcloud services list --enabled \| grep bigquery.googleapis.com` | Listed |
| Test query | `bq query "SELECT 1"` | Returns 1 |
| ML access | `bq query "SELECT * FROM ML.EVALUATE(...)"` | No API error |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Permission denied" | Not project owner/editor | Request IAM role upgrade |
| "Billing not enabled" | No billing account | Link billing account to project |
| "Quota exceeded" | Too many API calls | Wait or request quota increase |
| API already enabled | No action needed | Proceed to next task |

---

## Cost Implications

| Item | Cost |
|------|------|
| Enabling APIs | Free |
| BigQuery queries | $6.25 per TB processed |
| BigQuery ML | $250 per TB for training |
| First 1 TB/month | Free tier |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| GCP APIs | Service enablement |
| Project Setup | Required configuration |
| IAM Prerequisites | Role requirements |

---

## Next Task

[TASK-001-003: Create Dataset](./TASK-001-003-create-dataset.md)
