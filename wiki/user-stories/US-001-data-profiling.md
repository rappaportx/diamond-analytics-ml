# US-001: Data Profiling & Quality Assessment

## User Story

**As a** ML Engineer preparing for certification,
**I want to** profile the Chicago Taxi dataset and assess data quality,
**So that** I understand the data characteristics and can make informed cleaning decisions.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-001 |
| EPIC | [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] GCP authentication verified
- [x] Required APIs enabled
- [x] Project dataset created
- [x] Total record count documented
- [x] Date range identified
- [x] Null value percentages calculated
- [x] Invalid records quantified
- [x] Data quality log table created

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-001-001](../tasks/TASK-001-001-verify-gcp-auth.md) | Verify GCP Authentication | Complete |
| [TASK-001-002](../tasks/TASK-001-002-enable-apis.md) | Enable Required APIs | Complete |
| [TASK-001-003](../tasks/TASK-001-003-create-dataset.md) | Create Project Dataset | Complete |
| [TASK-001-004](../tasks/TASK-001-004-profile-source.md) | Profile Source Data | Complete |
| [TASK-001-005](../tasks/TASK-001-005-create-quality-log.md) | Create Data Quality Log | Complete |

---

## Actual Results

### Data Profile

| Metric | Value |
|--------|-------|
| Total Trips (2020+) | 20,713,994 |
| Unique Taxis | 5,727 |
| Unique Companies | 64 |
| Date Range | 2020-01-01 to 2023-12-31 |
| Average Fare | $25.19 |
| Median Fare | $15.04 |
| Median Miles | 2.2 |
| Median Duration | 13.3 minutes |

### Data Quality Issues

| Issue | Count | Percentage |
|-------|-------|------------|
| Null Fares | 16,609 | 0.08% |
| Invalid Miles (<=0) | 2,686,438 | 13% |
| Null Locations | 1,336,117 | 6.4% |
| Invalid Timestamps | 86 | <0.01% |

### Quality Scores

| Metric | Value |
|--------|-------|
| Fare Completeness | 99.81% |
| Location Validity | 100% |
| Overall Quality | **Excellent** |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Data Profiling | Understanding dataset characteristics before modeling |
| SQL Aggregations | COUNT, AVG, APPROX_QUANTILES |
| Data Quality Metrics | Completeness, validity, consistency |
| BigQuery Public Data | Accessing external datasets |

---

## Navigation

- **EPIC**: [EPIC-001: Data Engineering](../epics/EPIC-001-data-engineering.md)
- **Next User Story**: [US-002: Create Cleaned Base Table](./US-002-data-cleaning.md)
