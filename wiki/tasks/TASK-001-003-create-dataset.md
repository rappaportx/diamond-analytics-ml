# TASK-001-003: Create Project Dataset

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-001-003 |
| User Story | [US-001: Data Profiling](../user-stories/US-001-data-profiling.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Create the `diamond_analytics` dataset in BigQuery to store all project tables, models, and views.

---

## Prerequisites

- [x] GCP authentication verified (TASK-001-001)
- [x] BigQuery API enabled (TASK-001-002)
- [x] Project ID: `sonorous-key-320714`

---

## Step-by-Step Instructions

### Step 1: Create Dataset via CLI

Execute the following command:

```bash
bq mk --dataset \
  --location=US \
  --description="Diamond Analytics ML Certification Project - Chicago Taxi Fare Prediction" \
  sonorous-key-320714:diamond_analytics
```

**Parameter Explanation**:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `--dataset` | flag | Indicates dataset creation |
| `--location` | US | Multi-region for public data access |
| `--description` | "Diamond Analytics..." | Human-readable description |
| `project:dataset` | sonorous-key-320714:diamond_analytics | Fully qualified name |

**Expected Output**:
```
Dataset 'sonorous-key-320714:diamond_analytics' successfully created.
```

### Step 2: Verify Dataset Creation

```bash
bq ls --project_id=sonorous-key-320714
```

**Expected Output**:
```
     datasetId
 ------------------
  diamond_analytics
```

### Step 3: Check Dataset Details

```bash
bq show sonorous-key-320714:diamond_analytics
```

**Expected Output**:
```
Dataset sonorous-key-320714:diamond_analytics

   Last modified                  ACLs                   Labels
 ----------------- ------------------------------------ --------
  30 Jan 12:00:00   Owners: projectOwners
                    Writers: projectWriters
                    Readers: projectReaders

Location: US
```

---

## Alternative: Create via SQL

Execute in BigQuery Console:

```sql
CREATE SCHEMA IF NOT EXISTS `sonorous-key-320714.diamond_analytics`
OPTIONS(
  location = 'US',
  description = 'Diamond Analytics ML Certification Project - Chicago Taxi Fare Prediction'
);
```

---

## Alternative: Create via Console

### Step 1: Navigate to BigQuery

1. Go to: https://console.cloud.google.com/bigquery
2. Select project `sonorous-key-320714`

### Step 2: Create Dataset

1. Click the three dots next to project name
2. Select "Create dataset"
3. Fill in:
   - Dataset ID: `diamond_analytics`
   - Data location: `US (multiple regions)`
   - Default table expiration: Leave empty
4. Click "CREATE DATASET"

---

## Dataset Configuration

| Setting | Value | Rationale |
|---------|-------|-----------|
| Location | US | Same region as public taxi data |
| Encryption | Google-managed | Default, sufficient for training |
| Table Expiration | None | Keep tables indefinitely |
| Partition Expiration | None | Configure per-table |

---

## Location Selection Guide

| Location | Use Case | Cost |
|----------|----------|------|
| US (multi-region) | Public datasets, high availability | Standard |
| us-central1 | Single region, lower latency | Standard |
| EU (multi-region) | European data residency | Standard |

**Why US?**: The `bigquery-public-data.chicago_taxi_trips` dataset is in US multi-region.

---

## Verification Checklist

| Check | Expected Result | Status |
|-------|-----------------|--------|
| Dataset appears in `bq ls` | diamond_analytics listed | Complete |
| Dataset location is US | Location: US | Complete |
| Can query from dataset | No permission error | Complete |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Dataset already exists" | Previously created | Safe to proceed |
| "Permission denied" | Missing roles | Need BigQuery Admin or Data Editor |
| "Invalid dataset ID" | Special characters | Use only letters, numbers, underscores |
| "Location conflict" | Region mismatch | Ensure US for public data |

---

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Dataset | lowercase_underscores | diamond_analytics |
| Tables | lowercase_underscores | trips_cleaned |
| Models | lowercase_underscores | fare_predictor_xgb |
| Views | prefix with v_ or _view | model_health_alerts |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Dataset Creation | BigQuery resource organization |
| Location Selection | Multi-region vs single region |
| Naming Standards | Best practices for resources |

---

## Next Task

[TASK-001-004: Profile Source Data](./TASK-001-004-profile-source.md)
