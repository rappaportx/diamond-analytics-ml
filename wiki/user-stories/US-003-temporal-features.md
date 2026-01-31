# US-003: Temporal Feature Engineering

## User Story

**As a** ML Engineer preparing for certification,
**I want to** create temporal features using cyclical encoding and binary flags,
**So that** my models can capture time-based patterns in taxi demand and pricing.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-003 |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Cyclical encoding for hour (sin/cos)
- [x] Cyclical encoding for day of week (sin/cos)
- [x] Cyclical encoding for month (sin/cos)
- [x] Cyclical encoding for day of year (sin/cos)
- [x] Binary flags for weekend, rush hours, seasons
- [x] Features table created with all temporal features

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-003-001](../tasks/TASK-003-001-hour-encoding.md) | Implement Cyclical Hour Encoding | Complete |
| [TASK-003-002](../tasks/TASK-003-002-dow-encoding.md) | Implement Cyclical Day-of-Week Encoding | Complete |
| [TASK-003-003](../tasks/TASK-003-003-month-encoding.md) | Implement Cyclical Month Encoding | Complete |
| [TASK-003-004](../tasks/TASK-003-004-binary-flags.md) | Create Binary Time Flags | Complete |
| [TASK-003-005](../tasks/TASK-003-005-create-table.md) | Create Temporal Features Table | Complete |

---

## Features Created

### Cyclical Features (8)

| Feature | Formula | Period |
|---------|---------|--------|
| hour_sin | SIN(2π × hour/24) | 24 hours |
| hour_cos | COS(2π × hour/24) | 24 hours |
| dow_sin | SIN(2π × dow/7) | 7 days |
| dow_cos | COS(2π × dow/7) | 7 days |
| month_sin | SIN(2π × month/12) | 12 months |
| month_cos | COS(2π × month/12) | 12 months |
| doy_sin | SIN(2π × doy/365) | 365 days |
| doy_cos | COS(2π × doy/365) | 365 days |

### Binary Flags (6)

| Feature | Definition | Purpose |
|---------|------------|---------|
| is_weekend | dow IN (1, 7) | Weekend pattern |
| is_morning_rush | hour 7-9 | Morning surge |
| is_evening_rush | hour 16-19 | Evening surge |
| is_late_night | hour 0-5 | Late night pattern |
| is_summer | month IN (6, 7, 8) | Seasonal pattern |
| is_winter | month IN (12, 1, 2) | Seasonal pattern |

---

## Why Cyclical Encoding?

### Problem with Raw Values
Raw hour values (0-23) create artificial discontinuity:
- Hour 23 and hour 0 are numerically far apart (23 vs 0)
- But temporally they're adjacent (11 PM to midnight)

### Solution: SIN/COS Encoding
Using both sine and cosine creates a circular representation:
- Each hour maps to a unique (x, y) point on a circle
- Adjacent hours are geometrically close
- Hour 23 is close to hour 0

### Visual Representation
```
         12:00
           |
    9:00 --+-- 15:00
           |
         18:00/6:00
           |
    3:00 --+-- 21:00
           |
         0:00/24:00
```

---

## Actual Results

| Metric | Value |
|--------|-------|
| Total Records | 10,598,441 |
| Features Created | 14 |
| NULL Values | 0 |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Cyclical Encoding | SIN/COS transformation for periodic features |
| Feature Engineering | Creating derived features from raw data |
| Trigonometric Functions | ACOS(-1) for π, SIN, COS |
| Binary Encoding | CASE statements for categorical features |

---

## Navigation

- **EPIC**: [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md)
- **Previous**: [US-002: Data Cleaning](./US-002-data-cleaning.md)
- **Next**: [US-004: Geospatial Features](./US-004-geospatial-features.md)
