# System Architecture

**Technical Specification v2.0 - Post-Validation Revision**

---

## Architectural Philosophy & Overview

This architecture is designed for the constraints identified in the 2025 Baseline Assessment: 89% unreliable connectivity, low to medium digital literacy, and system fragmentation across multiple platforms (DHIS2, e-LMIS, eAFYA, paper registers). Instead of a single complex AI system, we implement a distributed, tiered architecture that places core functionality where it's needed most: **on the mobile device at the facility level**.

### Three Architectural Layers

| Layer | Function | Connectivity |
|-------|----------|--------------|
| **Offline-first mobile application** | Tier 1 rule-based forecasting, local SQLite storage (WAL mode), encrypted offline authentication | No internet required |
| **District-level services** | Tier 2 hierarchical time series forecasting, Tier 3 ML models, data aggregation, district dashboard | Periodic connectivity |
| **Central data hub** | National data warehouse, long-term trend analysis, integration with DHIS2/e-LMIS/eAFYA | Always connected |

---

## Core Architectural Principle

> **Data Flow Follows Connectivity**

The system does not assume real-time connectivity. Data moves opportunistically, following the Integration Maturity Model (Level 2: Opportunistic Sync).

{% hint style="warning" %}
**Critical Constraint:** Every core function required for daily facility operations must work without internet connectivity. Synchronization enhances capabilities but is never blocking.
{% endhint %}

---

## Detailed Data Flow by Layer

### Facility Level (Always Available - No Internet Required)

- Health workers use the mobile application for daily operational tasks: stock counts, order generation (using Tier 1 rule-based forecast), stockout reporting and consumption tracking
- All data is stored immediately in the local SQLite database configured in **Write-Ahead Logging (WAL) mode** to enable concurrent read operations during background sync processes
- The application generates paper backup forms matching Ministry of Health registers (e.g., HMIS 105) to ensure continuity during device failure or prolonged power outages
- Tier 1 forecasting engine runs entirely locally using preloaded seasonal adjustment rules and the facility's historical consumption data (last 12 months)

### Synchronization Layer (Opportunistic - Triggered by Connectivity)

| Trigger | Condition |
|---------|-----------|
| **Automatic** | Network detected AND battery > 20% AND device not in active use |
| **Manual** | User initiates 'Sync Now' (bypasses battery checks) |

**Upload Payload:**
- Batched JSON packets
- New stock counts
- Human-adjusted orders with override justifications
- Consumption data
- Sync metadata (last sync timestamp, facility ID, user ID)

**Download Payload:**
- Tier 2 statistical forecasts (stored separately as read-only)
- Updated delivery schedules
- Budget status notifications
- System messages

{% hint style="info" %}
**Conflict Resolution Principle:** Local user edits always win. The health worker on the ground has ground truth. Conflicts are logged to the override audit trail for district review.
{% endhint %}

### District Level (Periodic Connectivity Required)

- District server aggregates data from multiple facilities within its jurisdiction as mobile applications successfully sync
- **Tier 2** hierarchical time series forecasting service runs on aggregated facility data using Hierarchical Exponential Smoothing (HES)
- **Tier 3** machine learning models (Random Forest or XGBoost) identify facilities at high risk for stockouts
- District supply officers access a web-based dashboard to view insights, review override audit logs, and plan supply redistributions

### Central Level (National Reporting & Integration)

- Central data hub periodically pulls aggregated data from district servers using scheduled ETL jobs orchestrated by Apache Airflow
- Integration with national health information systems (DHIS2, e-LMIS, eAFYA) occurs at this level through RESTful APIs and FHIR-compliant protocols
- Supports Ministry of Health officials with national-level dashboards showing aggregated trends, regional stockout patterns, and forecast accuracy metrics (MAPE)

{% hint style="success" %}
**Design Note:** The central hub is not required for day-to-day facility operations. Districts can operate semi-autonomously if central connectivity is interrupted.
{% endhint %}

---

## Technology Stack & Key Components

### Mobile Client (Android-First Application)

| Component | Implementation Specification |
|-----------|------------------------------|
| **Development Framework** | Native Android (Kotlin/Java). Minimum SDK: Android 8.0 (API level 26) |
| **Offline Storage** | SQLite with WAL mode. Target size: <50MB per facility. `PRAGMA journal_mode=WAL; PRAGMA synchronous=NORMAL;` |
| **Data Encryption** | AES-256 encryption using Android Keystore System. Keys generated in hardware-backed secure enclaves where available |
| **Forecasting Engine** | Embedded Tier 1 rule-based logic: 3-month SMA + seasonal multipliers + storage constraints. Runs in <100ms |
| **Synchronization** | RESTful JSON over HTTPS (TLS 1.3). GZIP compression. Exponential backoff (30s initial, 15min max) |
| **Data Archiving** | 12-month active retention. Monthly archive job moves older data to compressed files. Emergency purge at 45MB |
| **Offline Authentication** | bcrypt-hashed credentials cached locally. Valid for 90 days offline. Warning at 75 days |
| **UI Responsiveness** | Single-column layouts, 5-inch minimum (480x800px), 48dp touch targets. Tested on Samsung Galaxy A, Tecno Spark, Infinix Hot |

### District Server Services

| Component | Implementation Specification |
|-----------|------------------------------|
| **API Gateway** | FastAPI (Python) for RESTful endpoints. Rate limiting, request logging, automatic retry handling |
| **Database** | PostgreSQL 14+ partitioned by date for performance optimization |
| **Forecasting Engine (Tier 2)** | Hierarchical Exponential Smoothing (HES) using statsforecast library (Python) |
| **ML Engine (Tier 3)** | scikit-learn (Random Forest) or XGBoost. Features: storage capacity, delivery lead times, facility type, budget allocation, override patterns |
| **Dashboard** | Plotly Dash web interface. Features: facility status map, trend graphs, override audit log, stockout risk alerts, redistribution planning |
| **Deployment** | Docker containers, single server per district. Prometheus/Grafana monitoring, GitLab CI/CD |

### Central Data Hub

| Component | Implementation Specification |
|-----------|------------------------------|
| **Data Warehouse** | PostgreSQL with star schema (source_data, analytics, predictions schemas) |
| **ETL Pipeline** | Apache Airflow orchestrating daily district collection, weekly national system pulls |
| **National System Integration** | RESTful APIs to DHIS2, e-LMIS, eAFYA, Nutrition Appointment Platform (WHO/WFP/UNICEF/UNHCR) |
| **National Dashboard** | Web-based reporting for MoH officials: stockout trends, regional comparisons, forecast accuracy, system adoption |

---

## Data Schema Documentation

### Mobile SQLite Database Schema

All timestamps stored in UTC. All tables include `created_at` and `updated_at` fields for sync conflict resolution.

```sql
-- Core Tables
facilities: id, name, type, storage_capacity_rating, district_id,
            latitude, longitude, created_at, updated_at

commodities: id, name, code (UNIQUE), category, unit,
             created_at, updated_at

stock_levels: id, facility_id, commodity_id, quantity, date_counted,
              user_id, sync_status, created_at, updated_at

orders: id, facility_id, commodity_id, system_quantity, final_quantity,
        override_reason, override_justification, date_created,
        sync_status, user_id, created_at, updated_at

consumption_history: id, facility_id, commodity_id, quantity_consumed,
                     month, year, created_at, updated_at

sync_log: id, facility_id, sync_type, records_pushed, records_pulled,
          last_sync_time, status, error_message, created_at

users: id, username (UNIQUE), password_hash, role, facility_id,
       last_login, credential_expiry, created_at, updated_at
```

### District PostgreSQL Extensions

```sql
-- Additional district-level tables
tier2_forecasts: id, facility_id, commodity_id, forecast_quantity,
                 forecast_month, forecast_year, confidence_interval_lower,
                 confidence_interval_upper, model_type, created_at, updated_at

tier3_predictions: id, facility_id, stockout_risk_score, risk_factors (JSONB),
                   prediction_date, model_version, created_at

override_audit_log: id, facility_id, user_id, order_id, system_quantity,
                    final_quantity, override_reason, override_justification,
                    timestamp, reviewed_by, review_notes, created_at
```

---

## Security & Compliance

| Layer | Security Measure |
|-------|------------------|
| **Data in Transit** | TLS 1.3 encryption for all API communications |
| **Data at Rest (Mobile)** | AES-256 encryption via Android Keystore |
| **Data at Rest (Server)** | Full disk encryption for district/central servers |
| **Authentication** | Role-based access control (RBAC): facility staff, store managers, district officers, national administrators |
| **Audit Logging** | All user actions logged with timestamp, user ID, action details. Override audit log maintained indefinitely |
| **Compliance** | Uganda Data Protection and Privacy Act (2019), MoH Digital Health Guidelines |

---

## Integration Architecture

```
┌──────────────────┐
│   Mobile App     │──── Syncs directly with ────▶ ┌──────────────────┐
│ (Facility Level) │                               │  District Server  │
└──────────────────┘                               └────────┬─────────┘
                                                            │
                                                   Pulls from national
                                                   systems & pushes to
                                                   central hub
                                                            │
                                                            ▼
┌──────────────────┐                               ┌──────────────────┐
│  National Systems │◀── Bulk ETL exchange ───────│   Central Hub     │
│ (DHIS2, e-LMIS,  │                              │                   │
│  eAFYA, NAP)     │                              └──────────────────┘
└──────────────────┘
```

**Data Formats:**
- JSON over HTTPS for APIs
- CSV exports as fallback for Level 1 Integration Maturity facilities
- FHIR-compliant resources where supported

---

## Monitoring & Performance Metrics

### Operational Metrics

| Metric | Target |
|--------|--------|
| Sync success rate | >95% |
| Average time since last sync | Monitored per facility |
| Mobile app crash rate | <1% |
| District server uptime | 99%+ |

### Forecasting Metrics

- Mean Absolute Percentage Error (MAPE) for Tier 2/3 forecasts
- Override frequency by facility and commodity
- Stockout frequency and duration

### User Engagement Metrics

- Daily active users by facility
- Paper form generation frequency (indicates trust in digital system)

---

## Stakeholder Governance Structure

| Body | Role |
|------|------|
| **District Health Officers** | Primary system owners, core partners in implementation |
| **MoH Technical Working Groups** | Policy-level review and national strategy alignment |
| **Quarterly Review Committee** | Cross-stakeholder review of model performance, override patterns, stockout trends |
| **Frontline Worker Representatives** | Participation in design reviews and pilot validation |

---

## Architecture Validation Summary

| Component | Status | Feasibility Score |
|-----------|--------|-------------------|
| Offline-first functionality | Fully validated | 5/5 |
| Data security | Fully validated | 5/5 |
| Database performance | Fully validated | 4/5 |
| Data archiving | Fully validated | 5/5 |
| System interoperability | Fully validated | 4/5 |
| UI responsiveness | Fully validated | 5/5 |
| Data governance | Fully validated | 5/5 |
| Tier 2 forecasting | Validated with modification | - |

---

## Post-Grant Implementation Roadmap (2026+)

| Phase | Description | Duration |
|-------|-------------|----------|
| **Phase 1** | Mobile application development with Tier 1 forecasting | 2-3 months |
| **Phase 2** | Pilot deployment in 3-5 facilities per district (Karamoja + Southwest) | 3 months |
| **Phase 3** | District server deployment with Tier 2/3 forecasting and dashboard | 2 months |
| **Phase 4** | Pilot evaluation and framework refinement | 1 month |
| **Phase 5** | Scaled rollout with district ownership and MoH oversight | Ongoing |

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 2.0 (Post-Technical Validation) |
| **Date** | November 2025 |
| **Authors** | Gideon Abako, Timothy Kavuma (IFRAD) |
| **Validator** | Ojok Ivan, Kyambogo University Department of Data Science, Networks & AI |
| **License** | Creative Commons Attribution 4.0 International (CC BY 4.0) |
