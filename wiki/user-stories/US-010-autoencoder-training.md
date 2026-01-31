# US-010: Autoencoder Anomaly Detector

## User Story

**As a** ML Engineer preparing for certification,
**I want to** train an Autoencoder neural network for anomaly detection,
**So that** I demonstrate deep learning for unsupervised anomaly scoring.

---

## Linked Artifacts

| Field | Value |
|-------|-------|
| User Story ID | US-010 |
| EPIC | [EPIC-003: Model Development](../epics/EPIC-003-model-development.md) |
| Status | Complete |
| Tasks | 4 |

---

## Acceptance Criteria

- [x] Reconstruction features selected
- [x] Neural network architecture designed
- [x] Autoencoder model trained
- [x] Reconstruction errors produced
- [x] Anomaly thresholds defined

---

## Tasks

| ID | Title | Status |
|----|-------|--------|
| TASK-010-001 | Select Reconstruction Features | Complete |
| TASK-010-002 | Configure Neural Network Architecture | Complete |
| TASK-010-003 | Execute Model Training | Complete |
| TASK-010-004 | Validate Reconstruction Error | Complete |

---

## Model Architecture

```
Input Layer (11 features)
         │
         ▼
┌─────────────────────┐
│   Dense(32, ReLU)   │  ◄─── Encoder
│     Dropout(0.2)    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Dense(16, ReLU)   │
│     Dropout(0.2)    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Dense(8, ReLU)    │  ◄─── Bottleneck
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Dense(16, ReLU)   │
│     Dropout(0.2)    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│   Dense(32, ReLU)   │  ◄─── Decoder
│     Dropout(0.2)    │
└─────────┬───────────┘
          │
          ▼
Output Layer (11 features)
```

---

## Model Configuration

| Option | Value | Purpose |
|--------|-------|---------|
| model_type | AUTOENCODER | Self-supervised learning |
| hidden_units | [32, 16, 8, 16, 32] | Symmetric architecture |
| activation_fn | RELU | Non-linearity |
| dropout | 0.2 | Regularization |
| batch_size | 256 | Mini-batch size |
| learn_rate | 0.001 | Learning rate |
| max_iterations | 50 | Training epochs |
| optimizer | ADAM | Optimization algorithm |
| early_stop | TRUE | Prevent overfitting |
| min_rel_progress | 0.001 | Convergence threshold |

---

## Features Used

| Feature | Type | Range |
|---------|------|-------|
| trip_miles | Numeric | 0-100 |
| trip_minutes | Numeric | 1-240 |
| target_fare | Numeric | 2.5-150 |
| straight_line_km | Numeric | 0-100 |
| route_circuity | Numeric | 1-10 |
| hour_sin | Numeric | -1 to 1 |
| hour_cos | Numeric | -1 to 1 |
| dow_sin | Numeric | -1 to 1 |
| dow_cos | Numeric | -1 to 1 |
| is_airport | Binary | 0-1 |
| is_downtown | Binary | 0-1 |

---

## Anomaly Classification

| Classification | MSE Threshold | Interpretation |
|----------------|---------------|----------------|
| NORMAL | MSE ≤ 0.1 | Typical trip pattern |
| LOW_RISK | 0.1 < MSE ≤ 0.2 | Slight deviation |
| MEDIUM_RISK | 0.2 < MSE ≤ 0.5 | Moderate deviation |
| HIGH_RISK | MSE > 0.5 | Significant anomaly |

---

## Results Summary

| Classification | Count | Percentage |
|----------------|-------|------------|
| HIGH_RISK | 28,583 | 57.17% |
| MEDIUM_RISK | 18,972 | 37.94% |
| LOW_RISK | 2,388 | 4.78% |
| NORMAL | 57 | 0.11% |

---

## Certification Concepts Demonstrated

| Concept | Description |
|---------|-------------|
| Neural Networks | Deep learning architecture |
| Autoencoders | Self-supervised learning |
| Anomaly Detection | Reconstruction error method |
| Regularization | Dropout technique |
| Architecture Design | Symmetric encoder-decoder |

---

## Navigation

- **EPIC**: [EPIC-003: Model Development](../epics/EPIC-003-model-development.md)
- **Previous**: [US-009: K-Means Training](./US-009-kmeans-training.md)
- **Next**: [US-011: Model Evaluation](./US-011-model-evaluation.md)
