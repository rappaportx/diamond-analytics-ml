# TASK-003-001: Create Temporal Features

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-003-001 |
| User Story | [US-003: Temporal Features](../user-stories/US-003-temporal-features.md) |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |

---

## Objective

Extract and create temporal features from trip timestamps including hour, day of week, month, and cyclical encodings.

---

## Prerequisites

- [x] Cleaned table created (TASK-002-002)
- [x] Understanding of cyclical encoding
- [x] Knowledge of time-based patterns in taxi data

---

## Step-by-Step Instructions

### Step 1: Understand Temporal Patterns

Taxi demand varies by:

| Pattern | Example | Impact on Fare |
|---------|---------|----------------|
| Hour of day | Rush hour vs midnight | High |
| Day of week | Weekday vs weekend | Medium |
| Month | Summer vs winter | Low |
| Holidays | Special events | High |

### Step 2: Extract Basic Temporal Features

```sql
SELECT
  trip_start_timestamp,

  -- Basic extractions
  EXTRACT(HOUR FROM trip_start_timestamp) as hour_of_day,
  EXTRACT(DAYOFWEEK FROM trip_start_timestamp) as day_of_week,
  EXTRACT(MONTH FROM trip_start_timestamp) as month,
  EXTRACT(YEAR FROM trip_start_timestamp) as year,

  -- Derived features
  CASE EXTRACT(DAYOFWEEK FROM trip_start_timestamp)
    WHEN 1 THEN TRUE  -- Sunday
    WHEN 7 THEN TRUE  -- Saturday
    ELSE FALSE
  END as is_weekend

FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
LIMIT 5;
```

**Expected Output**:

| trip_start_timestamp | hour_of_day | day_of_week | month | year | is_weekend |
|---------------------|-------------|-------------|-------|------|------------|
| 2023-09-15 08:30:00 | 8 | 6 | 9 | 2023 | FALSE |
| 2023-09-16 22:15:00 | 22 | 7 | 9 | 2023 | TRUE |

### Step 3: Understand Cyclical Encoding

**Problem**: Hour 23 and Hour 0 are adjacent, but numerically far apart (23 vs 0).

**Solution**: Encode cyclical features using sine and cosine.

```
hour_sin = SIN(2 * PI * hour / 24)
hour_cos = COS(2 * PI * hour / 24)
```

**Visual Representation**:

```
Hour 0:  sin=0.00, cos=1.00  (midnight)
Hour 6:  sin=1.00, cos=0.00  (morning)
Hour 12: sin=0.00, cos=-1.00 (noon)
Hour 18: sin=-1.00, cos=0.00 (evening)
Hour 23: sin=-0.26, cos=0.97 (close to midnight!)
```

### Step 4: Create Cyclical Encoding Functions

```sql
-- Hour cyclical encoding (24-hour cycle)
SIN(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24) as hour_sin,
COS(2 * ACOS(-1) * EXTRACT(HOUR FROM trip_start_timestamp) / 24) as hour_cos,

-- Day of week cyclical encoding (7-day cycle)
SIN(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7) as dow_sin,
COS(2 * ACOS(-1) * EXTRACT(DAYOFWEEK FROM trip_start_timestamp) / 7) as dow_cos,

-- Month cyclical encoding (12-month cycle)
SIN(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12) as month_sin,
COS(2 * ACOS(-1) * EXTRACT(MONTH FROM trip_start_timestamp) / 12) as month_cos
```

**Note**: `ACOS(-1)` returns PI in BigQuery.

### Step 5: Verify Cyclical Encoding

```sql
WITH sample AS (
  SELECT DISTINCT EXTRACT(HOUR FROM trip_start_timestamp) as hour
  FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
)
SELECT
  hour,
  ROUND(SIN(2 * ACOS(-1) * hour / 24), 3) as hour_sin,
  ROUND(COS(2 * ACOS(-1) * hour / 24), 3) as hour_cos
FROM sample
ORDER BY hour;
```

**Expected Output** (partial):

| hour | hour_sin | hour_cos |
|------|----------|----------|
| 0 | 0.000 | 1.000 |
| 6 | 1.000 | 0.000 |
| 12 | 0.000 | -1.000 |
| 18 | -1.000 | 0.000 |
| 23 | -0.259 | 0.966 |

### Step 6: Create Rush Hour Feature

```sql
CASE
  WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 7 AND 9 THEN TRUE
  WHEN EXTRACT(HOUR FROM trip_start_timestamp) BETWEEN 16 AND 19 THEN TRUE
  ELSE FALSE
END as is_rush_hour
```

### Step 7: Create Night Trip Feature

```sql
CASE
  WHEN EXTRACT(HOUR FROM trip_start_timestamp) >= 22 THEN TRUE
  WHEN EXTRACT(HOUR FROM trip_start_timestamp) <= 5 THEN TRUE
  ELSE FALSE
END as is_night
```

---

## Complete Temporal Feature Set

| Feature | Type | Formula | Purpose |
|---------|------|---------|---------|
| hour_of_day | INTEGER | EXTRACT(HOUR) | Raw hour |
| hour_sin | FLOAT64 | SIN(2*PI*hour/24) | Cyclical hour |
| hour_cos | FLOAT64 | COS(2*PI*hour/24) | Cyclical hour |
| day_of_week | INTEGER | EXTRACT(DAYOFWEEK) | Raw day |
| dow_sin | FLOAT64 | SIN(2*PI*dow/7) | Cyclical day |
| dow_cos | FLOAT64 | COS(2*PI*dow/7) | Cyclical day |
| month | INTEGER | EXTRACT(MONTH) | Raw month |
| month_sin | FLOAT64 | SIN(2*PI*month/12) | Cyclical month |
| month_cos | FLOAT64 | COS(2*PI*month/12) | Cyclical month |
| is_weekend | BOOLEAN | dow IN (1,7) | Weekend flag |
| is_rush_hour | BOOLEAN | hour IN (7-9, 16-19) | Rush hour flag |
| is_night | BOOLEAN | hour >= 22 OR hour <= 5 | Night flag |

---

## Why Cyclical Encoding Matters

### Without Cyclical Encoding

| Hour | Value | Distance 0→23 |
|------|-------|---------------|
| 0 | 0 | 23 units |
| 23 | 23 | 23 units |

The model sees 0 and 23 as very different!

### With Cyclical Encoding

| Hour | (sin, cos) | Distance 0→23 |
|------|------------|---------------|
| 0 | (0, 1) | 0.26 units |
| 23 | (-0.26, 0.97) | 0.26 units |

The model sees 0 and 23 as adjacent!

---

## Certification Exam Tip

**Question Pattern**: "How would you encode hour of day to preserve cyclical relationship?"

**Answer**: Use sine and cosine transformations:
- `sin(2 * pi * hour / 24)` and `cos(2 * pi * hour / 24)`
- This preserves the circular nature where hour 23 is close to hour 0

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Feature Engineering | Creating predictive features |
| Cyclical Encoding | Preserving circular relationships |
| Temporal Features | Time-based predictors |
| Domain Knowledge | Rush hour, weekend patterns |

---

## Next Task

[TASK-003-002: Create Geospatial Features](./TASK-003-002-create-geospatial-features.md)
