# TASK-001-001: Verify GCP Authentication

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-001-001 |
| User Story | [US-001: Data Profiling](../user-stories/US-001-data-profiling.md) |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |

---

## Objective

Verify that Google Cloud Platform authentication is properly configured for BigQuery access.

---

## Prerequisites

- [ ] Google Cloud SDK installed
- [ ] GCP account with project access
- [ ] Terminal/command line access

---

## Step-by-Step Instructions

### Step 1: Open Terminal

1. Open your terminal application
2. Navigate to your working directory

### Step 2: Check Current Authentication

Execute the following command to list authenticated accounts:

```bash
gcloud auth list
```

**Expected Output**:
```
         Credentialed Accounts
ACTIVE  ACCOUNT
*       your-email@domain.com
```

The asterisk (*) indicates the active account.

### Step 3: Verify Project Configuration

Check the currently configured project:

```bash
gcloud config get-value project
```

**Expected Output**:
```
sonorous-key-320714
```

### Step 4: Test BigQuery Access

Verify BigQuery API access by listing datasets:

```bash
bq ls --project_id=sonorous-key-320714
```

**Expected Output**: List of existing datasets (may be empty for new projects)

### Step 5: If Authentication Fails

If you see "ERROR" or "Not authenticated", run:

```bash
# Authenticate with Google Cloud
gcloud auth login

# Set application default credentials
gcloud auth application-default login
```

Follow the browser prompts to complete authentication.

### Step 6: Set Project (If Needed)

If project is not set correctly:

```bash
gcloud config set project sonorous-key-320714
```

---

## Verification Checklist

| Check | Expected Result | Actual |
|-------|-----------------|--------|
| `gcloud auth list` shows account | Active account visible | ✅ |
| `gcloud config get-value project` | sonorous-key-320714 | ✅ |
| `bq ls` runs without error | No error | ✅ |

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Not authenticated" | No credentials | Run `gcloud auth login` |
| "Permission denied" | No project access | Contact project owner |
| "Project not found" | Wrong project ID | Verify project ID in console |
| "API not enabled" | BigQuery not active | See TASK-001-002 |

---

## Actual Results

| Check | Result |
|-------|--------|
| Active Account | max@atlanticcoastautomotive.com |
| Project ID | sonorous-key-320714 |
| BigQuery Access | Verified |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| GCP Authentication | OAuth2 and service accounts |
| Project Configuration | Resource organization |
| CLI Tools | gcloud and bq commands |

---

## Next Task

[TASK-001-002: Enable Required APIs](./TASK-001-002-enable-apis.md)
