# TASK-008-001: Prepare Time Series Data for ARIMA

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-008-001 |
| User Story | [US-008: ARIMA Forecasting](../user-stories/US-008-arima-forecasting.md) |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |

---

## Objective

Prepare aggregated daily time series data for ARIMA demand forecasting model.

---

## Prerequisites

- [x] Cleaned data available (trips_cleaned)
- [x] Understanding of time series requirements
- [x] BigQuery ARIMA_PLUS model knowledge

---

## Step-by-Step Instructions

### Step 1: Understand ARIMA Data Requirements

| Requirement | Description |
|-------------|-------------|
| Time column | Regular intervals (daily, hourly) |
| Value column | Numeric metric to forecast |
| No gaps | Continuous time series |
| Sufficient history | At least 2 full cycles |

### Step 2: Aggregate to Daily Level

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.daily_demand` AS
SELECT
  DATE(trip_start_timestamp) as trip_date,
  COUNT(*) as daily_trips,
  ROUND(SUM(trip_total), 2) as daily_revenue,
  ROUND(AVG(trip_total), 2) as avg_fare,
  ROUND(AVG(trip_miles), 2) as avg_miles,
  COUNT(DISTINCT taxi_id) as active_taxis
FROM `sonorous-key-320714.diamond_analytics.trips_cleaned`
GROUP BY DATE(trip_start_timestamp)
ORDER BY trip_date;
```

### Step 3: Verify Data Quality

```sql
SELECT
  COUNT(*) as total_days,
  MIN(trip_date) as first_date,
  MAX(trip_date) as last_date,
  DATE_DIFF(MAX(trip_date), MIN(trip_date), DAY) + 1 as expected_days,
  ROUND(AVG(daily_trips), 0) as avg_daily_trips,
  ROUND(AVG(daily_revenue), 0) as avg_daily_revenue
FROM `sonorous-key-320714.diamond_analytics.daily_demand`;
```

**Expected Output**:

| total_days | first_date | last_date | expected_days | avg_daily_trips | avg_daily_revenue |
|------------|------------|-----------|---------------|-----------------|-------------------|
| 1461 | 2020-01-01 | 2023-12-31 | 1461 | 7,256 | $134,125 |

### Step 4: Check for Missing Days

```sql
WITH date_spine AS (
  SELECT date
  FROM UNNEST(GENERATE_DATE_ARRAY('2020-01-01', '2023-12-31')) as date
)
SELECT
  COUNT(*) as missing_days
FROM date_spine d
LEFT JOIN `sonorous-key-320714.diamond_analytics.daily_demand` dd
  ON d.date = dd.trip_date
WHERE dd.trip_date IS NULL;
```

**Expected Output**: `missing_days = 0`

### Step 5: Visualize Time Series

```sql
SELECT
  trip_date,
  daily_trips,
  ROUND(AVG(daily_trips) OVER (
    ORDER BY trip_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ), 0) as moving_avg_7d
FROM `sonorous-key-320714.diamond_analytics.daily_demand`
WHERE trip_date >= '2023-01-01'
ORDER BY trip_date
LIMIT 30;
```

### Step 6: Identify Seasonality Patterns

```sql
SELECT
  EXTRACT(DAYOFWEEK FROM trip_date) as day_of_week,
  ROUND(AVG(daily_trips), 0) as avg_trips,
  ROUND(STDDEV(daily_trips), 0) as std_trips
FROM `sonorous-key-320714.diamond_analytics.daily_demand`
GROUP BY day_of_week
ORDER BY day_of_week;
```

**Expected Output**:

| day_of_week | avg_trips | std_trips |
|-------------|-----------|-----------|
| 1 (Sun) | 5,234 | 1,456 |
| 2 (Mon) | 6,892 | 1,678 |
| 3 (Tue) | 7,345 | 1,723 |
| 4 (Wed) | 7,567 | 1,789 |
| 5 (Thu) | 8,123 | 1,845 |
| 6 (Fri) | 9,456 | 2,012 |
| 7 (Sat) | 6,178 | 1,567 |

**Insight**: Friday is highest demand, Sunday lowest.

---

## Time Series Schema

| Column | Type | Description |
|--------|------|-------------|
| trip_date | DATE | Daily date |
| daily_trips | INTEGER | Count of trips |
| daily_revenue | FLOAT64 | Total fare revenue |
| avg_fare | FLOAT64 | Average fare per trip |
| avg_miles | FLOAT64 | Average miles per trip |
| active_taxis | INTEGER | Unique taxis operating |

---

## Data Quality Summary

| Check | Result | Status |
|-------|--------|--------|
| Total days | 1,461 | PASS |
| Missing days | 0 | PASS |
| Continuous dates | Yes | PASS |
| Min daily trips | > 100 | PASS |
| Seasonality detected | Weekly | PASS |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Time Series Preparation | Aggregation for forecasting |
| Data Completeness | No missing time periods |
| Seasonality Analysis | Weekly patterns |
| Moving Averages | Trend smoothing |

---

## Next Task

[TASK-008-002: Train ARIMA Model](./TASK-008-002-train-arima.md)
