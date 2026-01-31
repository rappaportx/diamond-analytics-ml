# US-004: Geospatial Feature Engineering

## User Story

**As a** ML Engineer preparing for certification,
**I want to** create geospatial features using distance calculations and location categorization,
**So that** my models can capture spatial patterns in taxi trips.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-004 |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Haversine (straight-line) distance calculated
- [x] Route circuity ratio calculated
- [x] Grid-based location encoding implemented
- [x] Airport pickup/dropoff flags created
- [x] Downtown pickup/dropoff flags created
- [x] Same-area trip flag created
- [x] Features table created

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-004-001](../tasks/TASK-004-001-haversine.md) | Calculate Haversine Distance | Complete |
| [TASK-004-002](../tasks/TASK-004-002-circuity.md) | Calculate Route Circuity | Complete |
| [TASK-004-003](../tasks/TASK-004-003-grid-encoding.md) | Create Grid-Based Location Encoding | Complete |
| [TASK-004-004](../tasks/TASK-004-004-location-flags.md) | Create Airport/Downtown Flags | Complete |
| [TASK-004-005](../tasks/TASK-004-005-create-table.md) | Create Geospatial Features Table | Complete |

---

## Features Created

### Distance Features (2)

| Feature | Calculation | Purpose |
|---------|-------------|---------|
| straight_line_km | ST_DISTANCE(pickup, dropoff) / 1000 | True distance |
| route_circuity | actual_miles / straight_line | Route efficiency |

### Location Encoding (2)

| Feature | Calculation | Granularity |
|---------|-------------|-------------|
| pickup_grid_cell | FLOOR(lat×100)_FLOOR(lng×100) | ~1km cells |
| dropoff_grid_cell | FLOOR(lat×100)_FLOOR(lng×100) | ~1km cells |

### Location Category Flags (5)

| Feature | Definition | Area IDs |
|---------|------------|----------|
| is_airport_pickup | pickup_area IN (76, 56) | O'Hare, Midway |
| is_airport_dropoff | dropoff_area IN (76, 56) | O'Hare, Midway |
| is_downtown_pickup | pickup_area IN (32, 8, 33) | Loop, Near North/South |
| is_downtown_dropoff | dropoff_area IN (32, 8, 33) | Loop, Near North/South |
| same_area_trip | pickup_area = dropoff_area | Local trips |

---

## Chicago Community Areas Reference

| Area ID | Name | Type |
|---------|------|------|
| 8 | Near North Side | Downtown |
| 32 | Loop | Downtown Core |
| 33 | Near South Side | Downtown |
| 56 | Garfield Ridge (Midway) | Airport |
| 76 | O'Hare | Airport |

---

## Haversine Distance Formula

The Haversine formula calculates great-circle distance between two points on a sphere:

```
a = sin²(Δlat/2) + cos(lat1) × cos(lat2) × sin²(Δlng/2)
c = 2 × atan2(√a, √(1-a))
d = R × c
```

Where:
- R = Earth's radius (6,371 km)
- lat1, lat2 = latitudes in radians
- Δlat, Δlng = differences in radians

BigQuery simplifies this with `ST_DISTANCE()`.

---

## Route Circuity Interpretation

| Circuity | Interpretation |
|----------|----------------|
| 1.0 | Direct route (theoretical minimum) |
| 1.2 - 1.4 | Typical urban route |
| 1.5 - 2.0 | Indirect route |
| > 2.0 | Very indirect (potential anomaly) |

Average circuity in urban areas is typically 1.2-1.3.

---

## Actual Results

| Metric | Value |
|--------|-------|
| Total Records | 10,598,441 |
| Features Created | 9 |
| NULL Values | 0 |
| Airport Trips | ~5% |
| Downtown Trips | ~45% |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Geospatial Functions | ST_GEOGPOINT, ST_DISTANCE |
| Feature Engineering | Domain-specific features |
| Categorical Encoding | Location-based flags |
| Data Validation | Geographic bounds checking |

---

## Navigation

- **EPIC**: [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md)
- **Previous**: [US-003: Temporal Features](./US-003-temporal-features.md)
- **Next**: [US-005: Feature Store](./US-005-feature-store.md)
