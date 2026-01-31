# Diamond Analytics - ML Certification Portfolio

A production-ready machine learning analytics platform built with BigQuery ML, demonstrating end-to-end ML engineering capabilities.

## Live Dashboards

| Dashboard | Description | Link |
|-----------|-------------|------|
| **Technical** | ML model metrics, drift monitoring, alerts | [View Dashboard](https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/index.html) |
| **Executive** | Revenue impact, ROI analysis, business insights | [View Dashboard](https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/executive.html) |

## Project Overview

| Metric | Value |
|--------|-------|
| **Dataset** | Chicago Taxi Trips (2022-2023) |
| **Records Analyzed** | 10,598,441 cleaned trips |
| **Revenue Under Management** | $26.3 Million |
| **Predictions Generated** | 1.4 Million |
| **Operational Cost** | ~$35/month |

## ML Models

### 1. Fare Predictor (XGBoost)
- **R² Score**: 0.913 (92.3% accuracy)
- **MAE**: $3.56
- **82.8%** of predictions within $5 of actual

### 2. Demand Forecaster (ARIMA_PLUS)
- Hourly demand predictions
- Seasonal pattern detection
- Loop/Downtown focus

### 3. Driver Segmentation (K-Means)
| Segment | Drivers | Avg Fare | Strategy |
|---------|---------|----------|----------|
| Airport Specialists | 670 | $43.91 | Highest value - expand |
| Balanced Operators | 1,041 | $31.03 | Cross-train for airports |
| Night Owls | 204 | $30.88 | Target late-night demand |
| Downtown Regulars | 477 | $22.20 | Optimize short trips |
| Downtown Focus | 768 | $20.59 | Volume-based |

### 4. Anomaly Detector (Autoencoder)
- Fraud detection
- Unusual trip patterns
- Reconstruction error analysis

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Chicago Taxi   │────▶│   BigQuery ML    │────▶│  Cloud Storage  │
│  Public Data    │     │   (4 Models)     │     │  (Dashboards)   │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │                         │
                               ▼                         ▼
                        ┌──────────────┐          ┌─────────────┐
                        │ Cloud Function│◀────────│  Scheduler  │
                        │ (Daily Refresh)│         │  (6 AM CT)  │
                        └──────────────┘          └─────────────┘
```

## Project Structure

```
diamond-analytics-ml/
├── index.html                 # Technical ML dashboard
├── executive_dashboard.html   # Business executive dashboard
├── dashboard_data.json        # Auto-refreshed data
├── export_dashboard_data.py   # Local data export script
├── COST_ANALYSIS.md           # Detailed cost breakdown
├── cloud_function/            # Serverless refresh function
│   ├── main.py
│   └── requirements.txt
└── wiki/                      # Complete project documentation
    ├── README.md              # Wiki home
    ├── epics/                 # 6 EPICs
    ├── user-stories/          # 17 User Stories
    └── tasks/                 # 50+ Tasks
```

## Cost Analysis

| Component | Monthly | Annual |
|-----------|---------|--------|
| BigQuery Storage | $0.25 | $3.00 |
| BigQuery Queries | ~$5.00 | ~$60.00 |
| Cloud Function | ~$0.10 | ~$1.20 |
| Cloud Scheduler | $0.00 | $0.00 |
| **Total** | **~$35** | **~$420** |

**Cloud vs On-Premise**: 97% cost reduction compared to equivalent on-premise infrastructure.

## Certification Topics Covered

- Data Pipelines & ETL
- Feature Engineering & Feature Stores
- Supervised Learning (XGBoost)
- Unsupervised Learning (K-Means)
- Time Series Forecasting (ARIMA_PLUS)
- Anomaly Detection (Autoencoder)
- Model Evaluation & Metrics
- Hyperparameter Tuning
- Data Leakage Prevention
- Model Monitoring & Drift Detection
- MLOps Automation

## Quick Start

### View Live Dashboard
Simply visit the [Technical Dashboard](https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/index.html) or [Executive Dashboard](https://storage.googleapis.com/sonorous-key-320714-ml-dashboard/executive.html).

### Manual Refresh
```bash
curl https://us-central1-sonorous-key-320714.cloudfunctions.net/refresh-ml-dashboard
```

### Export Data Locally
```bash
python export_dashboard_data.py
```

## Documentation

Complete documentation is available in the [wiki/](./wiki/) directory:

1. [Project Overview](./wiki/01-project-overview.md)
2. [Architecture](./wiki/02-architecture.md)
3. [Data Engineering](./wiki/03-data-engineering.md)
4. [Feature Engineering](./wiki/04-feature-engineering.md)
5. [Model Training](./wiki/05-model-training.md)
6. [Model Evaluation](./wiki/06-model-evaluation.md)
7. [MLOps & Monitoring](./wiki/07-mlops-monitoring.md)
8. [Certification Mapping](./wiki/08-certification-mapping.md)

## License

MIT License - See LICENSE for details.

---

Built with BigQuery ML | Hosted on Google Cloud Storage | Automated with Cloud Functions
