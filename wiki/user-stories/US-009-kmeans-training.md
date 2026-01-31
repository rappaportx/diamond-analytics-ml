# US-009: K-Means Driver Segmentation

## User Story

**As a** ML Engineer preparing for certification,
**I want to** train a K-Means clustering model for driver segmentation,
**So that** I demonstrate unsupervised learning for customer/driver profiling.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-009 |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |
| Tasks | 4 |

---

## Acceptance Criteria

- [x] Taxi profile aggregations created
- [x] Clustering features selected
- [x] K-Means model configured with 5 clusters
- [x] Feature standardization enabled
- [x] Clusters are interpretable and nameable

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-009-001 | Create Taxi Profile Aggregations | Complete |
| TASK-009-002 | Select Clustering Features | Complete |
| TASK-009-003 | Configure K-Means Options | Complete |
| TASK-009-004 | Execute Clustering | Complete |

---

## Model Configuration

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | KMEANS | Clustering algorithm |
| num_clusters | 5 | Number of segments |
| kmeans_init_method | KMEANS++ | Smart initialization |
| max_iterations | 50 | Convergence limit |
| min_rel_progress | 0.001 | Early stopping |
| standardize_features | TRUE | Feature scaling |
| distance_type | EUCLIDEAN | Distance metric |

---

## Features Used

| Feature | Type | Purpose |
|---------|------|---------|
| avg_fare | Numeric | Trip value |
| avg_miles | Numeric | Trip length |
| avg_duration_mins | Numeric | Time investment |
| weekend_ratio | Numeric | Weekend work pattern |
| morning_rush_ratio | Numeric | Morning preference |
| evening_rush_ratio | Numeric | Evening preference |
| late_night_ratio | Numeric | Night shift |
| airport_ratio | Numeric | Airport specialization |
| downtown_ratio | Numeric | Downtown specialization |

---

## Cluster Results

| Cluster | Name | Size | Avg Fare | Airport% | Downtown% | Late Night% |
|---------|------|------|----------|----------|-----------|-------------|
| 1 | **Night Owls** | 204 | $30.88 | 16.7% | 35.8% | **32.3%** |
| 2 | **Downtown Regulars** | 477 | $22.20 | 8.5% | **49.8%** | 3.2% |
| 3 | **Downtown Focus** | 768 | $20.59 | 7.5% | **55.6%** | 3.5% |
| 4 | **Balanced Operators** | 1,041 | $31.03 | 15.9% | 39.0% | 4.3% |
| 5 | **Airport Specialists** | 670 | **$43.91** | **30.8%** | 30.5% | 6.0% |

---

## Cluster Naming Rationale

| Cluster | Primary Signal | Business Insight |
|---------|----------------|------------------|
| Night Owls | 32.3% late night (highest) | Target for overnight shifts |
| Downtown Regulars | 49.8% downtown, 17.5% weekend | Weekday business focus |
| Downtown Focus | 55.6% downtown (highest) | Core urban service |
| Balanced Operators | Largest group, moderate all | Flexible scheduling |
| Airport Specialists | 30.8% airport, $44 avg | Premium segment |

---

## Evaluation Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Davies-Bouldin Index | 1.59 | Good separation (<2.0) |
| Mean Squared Distance | 4.50 | Moderate compactness |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Unsupervised Learning | Clustering without labels |
| K-Means Algorithm | Centroid-based clustering |
| Feature Scaling | Standardization for distance |
| Cluster Interpretation | Business-meaningful segments |
| Evaluation Metrics | Davies-Bouldin index |

---

## Navigation

- **EPIC**: [EPIC-003: Model Development](../epics/EPIC-003-model-development.md)
- **Previous**: [US-008: ARIMA Training](./US-008-arima-training.md)
- **Next**: [US-010: Autoencoder Training](./US-010-autoencoder-training.md)
