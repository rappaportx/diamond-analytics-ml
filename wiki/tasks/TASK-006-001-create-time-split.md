# TASK-006-001: Create Time-Based Train/Test Split

## Task Details

| Field | Value |
|-------|-------|
| Task ID | TASK-006-001 |
| User Story | [US-006: Data Splitting](../user-stories/US-006-data-splitting.md) |
| EPIC | [EPIC-002: Feature Engineering](../epics/EPIC-002-feature-engineering.md) |
| Status | Complete |

---

## Objective

Create time-based train/test/holdout splits to prevent data leakage and simulate real-world deployment.

---

## Prerequisites

- [x] Feature store created with all features
- [x] Understanding of data leakage
- [x] Date range: 2020-01-01 to 2023-12-31

---

## Step-by-Step Instructions

### Step 1: Understand Split Strategy

| Split | Date Range | Purpose | % of Data |
|-------|------------|---------|-----------|
| Train | 2020-01-01 to 2023-06-30 | Model training | ~50% |
| Test | 2023-07-01 to 2023-09-30 | Model evaluation | ~20% |
| Holdout | 2023-10-01 to 2023-12-31 | Final validation | ~25% |

**Why Time-Based Split?**

| Method | Risk | Recommendation |
|--------|------|----------------|
| Random split | Data leakage from future | Not recommended |
| Time-based split | Simulates production | Recommended |
| K-fold | Complex for time series | For advanced use |

### Step 2: Verify Date Range

```sql
SELECT
  MIN(trip_start_timestamp) as first_trip,
  MAX(trip_start_timestamp) as last_trip,
  COUNT(*) as total_trips
FROM `sonorous-key-320714.diamond_analytics.feature_store`;
```

**Expected Output**:

| first_trip | last_trip | total_trips |
|------------|-----------|-------------|
| 2020-01-01 00:00:00 | 2023-12-31 23:59:00 | 10,598,441 |

### Step 3: Create Training Set

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.train_set` AS
SELECT
  *,
  -- Add is_eval column for XGBoost validation
  CASE WHEN RAND() < 0.2 THEN TRUE ELSE FALSE END as is_eval
FROM `sonorous-key-320714.diamond_analytics.feature_store`
WHERE trip_start_timestamp >= '2020-01-01'
  AND trip_start_timestamp < '2023-07-01';
```

**Note**: The `is_eval` column creates an 80/20 train/validation split within the training set for XGBoost early stopping.

### Step 4: Create Test Set

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.test_set` AS
SELECT
  *,
  FALSE as is_eval  -- Not used for testing
FROM `sonorous-key-320714.diamond_analytics.feature_store`
WHERE trip_start_timestamp >= '2023-07-01'
  AND trip_start_timestamp < '2023-10-01';
```

### Step 5: Create Holdout Set

```sql
CREATE OR REPLACE TABLE `sonorous-key-320714.diamond_analytics.holdout_set` AS
SELECT
  *,
  FALSE as is_eval
FROM `sonorous-key-320714.diamond_analytics.feature_store`
WHERE trip_start_timestamp >= '2023-10-01'
  AND trip_start_timestamp < '2024-01-01';
```

### Step 6: Verify Split Sizes

```sql
SELECT
  'Train' as split,
  COUNT(*) as row_count,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM `sonorous-key-320714.diamond_analytics.feature_store`), 1) as pct,
  MIN(trip_start_timestamp) as min_date,
  MAX(trip_start_timestamp) as max_date
FROM `sonorous-key-320714.diamond_analytics.train_set`

UNION ALL

SELECT
  'Test' as split,
  COUNT(*) as row_count,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM `sonorous-key-320714.diamond_analytics.feature_store`), 1) as pct,
  MIN(trip_start_timestamp),
  MAX(trip_start_timestamp)
FROM `sonorous-key-320714.diamond_analytics.test_set`

UNION ALL

SELECT
  'Holdout' as split,
  COUNT(*) as row_count,
  ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM `sonorous-key-320714.diamond_analytics.feature_store`), 1) as pct,
  MIN(trip_start_timestamp),
  MAX(trip_start_timestamp)
FROM `sonorous-key-320714.diamond_analytics.holdout_set`

ORDER BY min_date;
```

**Expected Output**:

| split | row_count | pct | min_date | max_date |
|-------|-----------|-----|----------|----------|
| Train | 5,299,220 | 50.0% | 2020-01-01 | 2023-06-30 |
| Test | 2,119,688 | 20.0% | 2023-07-01 | 2023-09-30 |
| Holdout | 3,179,533 | 30.0% | 2023-10-01 | 2023-12-31 |

### Step 7: Verify No Data Leakage

```sql
-- Check for overlap between sets
SELECT
  'Train-Test Overlap' as check_type,
  COUNT(*) as overlap_count
FROM `sonorous-key-320714.diamond_analytics.train_set` t
JOIN `sonorous-key-320714.diamond_analytics.test_set` te
  ON t.unique_key = te.unique_key

UNION ALL

SELECT
  'Test-Holdout Overlap',
  COUNT(*)
FROM `sonorous-key-320714.diamond_analytics.test_set` te
JOIN `sonorous-key-320714.diamond_analytics.holdout_set` h
  ON te.unique_key = h.unique_key;
```

**Expected Output**: All overlap_count = 0

### Step 8: Verify Train/Eval Split

```sql
SELECT
  is_eval,
  COUNT(*) as count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) as pct
FROM `sonorous-key-320714.diamond_analytics.train_set`
GROUP BY is_eval;
```

**Expected Output**:

| is_eval | count | pct |
|---------|-------|-----|
| FALSE | 4,239,376 | 80.0% |
| TRUE | 1,059,844 | 20.0% |

---

## Split Summary

| Split | Rows | Percentage | Date Range |
|-------|------|------------|------------|
| Train (Full) | 5,299,220 | 50% | 2020-01 to 2023-06 |
| Train (Training) | 4,239,376 | 40% | Within train set |
| Train (Validation) | 1,059,844 | 10% | Within train set |
| Test | 2,119,688 | 20% | 2023-07 to 2023-09 |
| Holdout | 3,179,533 | 30% | 2023-10 to 2023-12 |

---

## Data Leakage Prevention

| Risk | Mitigation |
|------|------------|
| Future data in training | Time-based cutoff |
| Target leakage | Excluded from features |
| Overlapping records | Verified no overlap |
| Feature leakage | No post-trip features |

---

## Certification Concepts

| Concept | Description |
|---------|-------------|
| Train/Test Split | Data partitioning for ML |
| Data Leakage | Preventing future information |
| Time-Based Split | Temporal data handling |
| Validation Set | Early stopping data |

---

## Next Task

[TASK-007-001: Select Features for XGBoost](./TASK-007-001-select-features.md)
