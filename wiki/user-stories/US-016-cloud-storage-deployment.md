# US-016: Cloud Storage Deployment

## User Story

**As a** ML Engineer,
**I want to** deploy the dashboard to Cloud Storage,
**So that** it is publicly accessible without running servers.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-016 |
| EPIC | [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md) |
| Status | Complete |
| Tasks | 2 |

---

## Acceptance Criteria

- [x] Cloud Storage bucket created
- [x] Dashboard files uploaded
- [x] Public access configured
- [x] Website hosting enabled
- [x] Content types set correctly

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-016-001 | Create Storage Bucket | Complete |
| TASK-016-002 | Configure Public Access | Complete |

---

## Deployment Details

### Bucket Configuration

| Setting | Value |
|---------|-------|
| Bucket Name | sonorous-key-320714-ml-dashboard |
| Location | US (multi-region) |
| Storage Class | Standard |
| Access Control | Uniform (IAM) |

### Public Access

```bash
# Grant public read access
gsutil iam ch allUsers:objectViewer gs://sonorous-key-320714-ml-dashboard

# Enable website hosting
gsutil web set -m index.html gs://sonorous-key-320714-ml-dashboard
```

### URLs

| Resource | URL |
|----------|-----|
| Dashboard | https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/index.html |
| Data JSON | https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/dashboard_data.json |

---

## Content Types

| File | Content-Type | Cache-Control |
|------|--------------|---------------|
| index.html | text/html | default |
| dashboard_data.json | application/json | no-cache, max-age=300 |

---

## Cost Analysis

| Resource | Monthly Estimate |
|----------|-----------------|
| Storage (40 KB) | $0.001 |
| Requests (~10K) | $0.01 |
| Egress (~1 GB) | $0.12 |
| **Total** | ~$0.13/month |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Cloud Storage | Object storage |
| Static Website Hosting | Serverless hosting |
| IAM Policies | Access control |
| Content Delivery | Global edge caching |

---

## Navigation

- **EPIC**: [EPIC-006: Dashboard Deployment](../epics/EPIC-006-dashboard-deployment.md)
- **Previous**: [US-015: HTML Dashboard](./US-015-html-dashboard.md)
- **Next**: [US-017: Automated Refresh](./US-017-automated-refresh.md)
