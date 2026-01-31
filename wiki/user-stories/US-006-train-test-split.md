# US-006: Train/Test/Holdout Split

## User Story

**As a** ML Engineer preparing for certification,
**I want to** create time-based train/test/holdout splits,
**So that** I prevent data leakage and have proper validation sets.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-006 |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |
| Tasks | 5 |

---

## Acceptance Criteria

- [x] Time-based split strategy defined
- [x] Training set created (historical data)
- [x] Test set created (recent data)
- [x] Holdout set created (most recent data)
- [x] No date overlap between splits verified
- [x] Split proportions documented

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| [TASK-006-001](../tasks/TASK-006-001-split-strategy.md) | Define Time-Based Split Strategy | Complete |
| [TASK-006-002](../tasks/TASK-006-002-train-set.md) | Create Training Set | Complete |
| [TASK-006-003](../tasks/TASK-006-003-test-set.md) | Create Test Set | Complete |
| [TASK-006-004](../tasks/TASK-006-004-holdout-set.md) | Create Holdout Set | Complete |
| [TASK-006-005](../tasks/TASK-006-005-verify-overlap.md) | Verify No Date Overlap | Complete |

---

## Why Time-Based Splits?

### Problem with Random Splits
For temporal data, random splits cause **data leakage**:
- Model sees "future" data during training
- Creates unrealistic performance estimates
- Model memorizes patterns it wouldn't know in production

### Solution: Temporal Splits
Time-based splits ensure:
- Training data is strictly before test data
- Model only learns from past patterns
- Evaluation reflects real-world deployment

---

## Split Strategy

```
Timeline:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━►
│                    TRAIN                    │   TEST   │  HOLDOUT  │
│               2022-01-01                    │ 2023-07  │  2023-10  │
│                   to                        │    to    │    to     │
│               2023-06-30                    │ 2023-09  │  2023-12  │
│                  (73%)                      │  (13%)   │   (13%)   │
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━►
```

---

## Actual Results

| Split | Records | Percentage | Min Date | Max Date |
|-------|---------|------------|----------|----------|
| Train | 7,785,679 | 73% | 2022-01-01 00:00 | 2023-06-30 23:45 |
| Test | 1,427,550 | 13% | 2023-07-01 00:00 | 2023-09-30 23:45 |
| Holdout | 1,385,212 | 13% | 2023-10-01 00:00 | 2023-12-31 23:45 |

### Date Gap Verification

| Transition | Train Max | Next Min | Gap |
|------------|-----------|----------|-----|
| Train → Test | 2023-06-30 23:45 | 2023-07-01 00:00 | 15 min |
| Test → Holdout | 2023-09-30 23:45 | 2023-10-01 00:00 | 15 min |

**Result**: Clean temporal separation, no overlap.

---

## SQL Implementation

```sql
-- Training Set
CREATE TABLE train_set AS
SELECT * FROM feature_store
WHERE trip_start_timestamp < '2023-07-01';

-- Test Set
CREATE TABLE test_set AS
SELECT * FROM feature_store
WHERE trip_start_timestamp >= '2023-07-01'
  AND trip_start_timestamp < '2023-10-01';

-- Holdout Set
CREATE TABLE holdout_set AS
SELECT * FROM feature_store
WHERE trip_start_timestamp >= '2023-10-01';
```

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Data Leakage Prevention | Time-based splits for temporal data |
| Train/Test/Holdout | Three-way split methodology |
| Validation Strategy | Proper evaluation framework |
| SQL Date Filtering | Timestamp comparisons |

---

## Navigation

- **EPIC**: [EPIC-003: Model Development](../epics/EPIC-003-model-development.md)
- **Previous**: [US-005: Feature Store](./US-005-feature-store.md)
- **Next**: [US-007: XGBoost Training](./US-007-xgboost-training.md)
