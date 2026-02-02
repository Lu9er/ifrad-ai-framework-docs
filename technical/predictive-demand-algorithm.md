# Predictive Demand Algorithm

**Tiered Forecasting Specification for Uganda's Health Supply Chain**

---

## Introduction

This document specifies a pragmatic forecasting framework for Uganda's health supply chain, informed by the 2025 Baseline Assessment conducted by IFRAD. The assessment revealed that **89% of facilities experience unreliable connectivity**, widespread system fragmentation persists, and infrastructure constraints are primary drivers of stockouts.

{% hint style="info" %}
**Key Finding:** Storage capacity demonstrated a strong negative correlation with stockout frequency (r = -0.695), confirming that physical infrastructure limitations must be treated as **hard constraints** rather than variables in any forecasting system.
{% endhint %}

This framework abandons a one-size-fits-all complex AI model in favor of a **Tiered Forecasting Approach** ensuring forecasting remains functional, understandable, and actionable at every level of the health system.

---

## Framework Overview

### Core Principles from Baseline Evidence

| Principle | Implementation |
|-----------|----------------|
| **Infrastructure as Binding Constraint** | Storage capacity hard-coded as a limit in all algorithms, not an input variable |
| **Connectivity is Intermittent** | Forecasting must function primarily offline (89% unreliable connectivity) |
| **Human Oversight is Critical** | Overrides are not failures—they represent contextual knowledge algorithms cannot capture |

---

## Tiered Forecasting Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                         TIER 3                                      │
│              Machine Learning Forecasting                           │
│         (District/Central Level - Random Forest/XGBoost)            │
│                                                                     │
│   Purpose: Stockout risk prediction, district planning              │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│                         TIER 2                                      │
│         Hierarchical Exponential Smoothing (HES)                    │
│              (District Server - Weekly Generation)                  │
│                                                                     │
│   Purpose: Second opinion for facility comparison                   │
└───────────────────────────────┬────────────────────────────────────┘
                                │
                                ▼
┌────────────────────────────────────────────────────────────────────┐
│                         TIER 1                                      │
│              Rule-Based Forecasting (Offline)                       │
│                 (Mobile App - Always Available)                     │
│                                                                     │
│   Purpose: Primary operational forecasting                          │
└────────────────────────────────────────────────────────────────────┘
```

---

## Tier 1: Rule-Based Forecasting

**Deployment:** Embedded within the offline-first mobile application
**Connectivity:** None required
**Target:** All facilities, especially those with zero connectivity

### Core Algorithm Logic

#### Base Calculation

**Simple Moving Average** of the past 3 months of consumption data.

#### Seasonal Adjustment

Pre-loaded fixed multipliers derived from multi-year DHIS2 national consumption patterns:

| Commodity | Season | Adjustment |
|-----------|--------|------------|
| Antimalarials | April-June, October-December (rainy seasons) | +30% |
| Oral Rehydration Salts | Dry season months | +25% |
| Anti-snake venom | Planting and harvest seasons | +40% |

#### Infrastructure Constraint Application

The final recommended order quantity is **capped** based on storage capacity:

| Storage Rating | Cap Applied |
|---------------|-------------|
| Very Inadequate | 50% of calculated demand |
| Inadequate | 70% of calculated demand |
| Adequate | No cap |

{% hint style="warning" %}
**Rationale:** This directly addresses the baseline finding that facilities with inadequate storage capacity experienced the highest stockout frequency (r = -0.695). The algorithm prevents recommendations for quantities that cannot physically be stored.
{% endhint %}

### Cold Start Protocol

For new facilities or those with insufficient historical data (<3 months), the system matches against similar facilities using:

1. **Facility Level:** HC II, HC III, HC IV, or Hospital
2. **Geographic Region:** Karamoja, West Nile, Central, Eastern, etc.
3. **Patient Volume Bracket:** Low (<500/month), Medium (500-1500/month), High (>1500/month)
4. **Commodity Class:** HIV facilities vs. non-HIV, etc.

The system calculates **median consumption** from matched facilities, then applies seasonal adjustments and storage constraints.

> **Example:** A new HC III in Karamoja with 800 monthly patients would use the median consumption from other HC III facilities in Karamoja serving 600-1000 patients per month.

### Data Synchronization Protocol

**Upload to District Server:**
- Final approved order quantities
- Override logs (quantity changed, reason selected, user ID)
- Current stock levels
- Storage capacity status

**Download from District Server:**
- Updated regional baseline data (refreshed median consumption values)
- Revised seasonal multipliers
- Tier 2 forecasts for comparison
- Alerts or directives from district supply officers

---

## Tier 2: Hierarchical Exponential Smoothing (HES)

**Deployment:** District-level servers
**Schedule:** Weekly generation, pushed to facilities during sync
**Target:** Facilities with intermittent connectivity

### Model Selection

**Hierarchical Exponential Smoothing (HES)** using bottom-up reconciliation was selected for:

| Advantage | Description |
|-----------|-------------|
| **Scalability** | Single model handles all facilities vs. 200+ separate ETS models |
| **Resource Efficiency** | 80% reduction in computational overhead |
| **Coherence** | Bottom-up reconciliation resolves inconsistencies |
| **Open Source** | Available via Python (hierarchicalforecast) and R (hts, fable) |
| **Interpretability** | Clear hierarchical structure aids government stakeholder explanation |

### Hierarchical Structure

```
District Total
├── HC II Group (sum of all HC II facilities)
├── HC III Group (sum of all HC III facilities)
├── HC IV Group (sum of all HC IV facilities)
└── Hospital Group (sum of hospital facilities)
```

### Data Sources

- **DHIS2:** Historical consumption patterns (12-24 months)
- **e-LMIS:** Stock level history and delivery schedules
- **Tier 1 Uploads:** Actual facility orders and consumption
- **eAFYA:** Patient service volumes (demand proxy)

### Output and Use Case

{% hint style="info" %}
**Important:** Tier 2 forecasts are **not orders**. They are a second opinion for facility staff to compare against their Tier 1 calculation.
{% endhint %}

**User Interface Display:**

```
┌──────────────────────────────────────────────────┐
│ ACT Tablets Forecast Comparison                   │
├──────────────────────────────────────────────────┤
│ Your Facility Forecast (Tier 1):    650 tablets  │
│ District Data Forecast (Tier 2):    720 tablets  │
│ Difference:                   +70 tablets (+10.8%)│
│                                                   │
│ Explanation: District data shows increased       │
│ malaria cases in nearby facilities this month.   │
└──────────────────────────────────────────────────┘
```

**Purpose:** Tier 2 identifies trends not visible from single-facility data:
- Emerging disease patterns across the district
- Seasonal patterns not yet reflected in 3-month windows
- Supply chain disruptions affecting multiple facilities

---

## Tier 3: Machine Learning Forecasting

**Deployment:** Central cloud or district servers with reliable power/connectivity
**Target:** District supply officers (not deployed to facilities)

### Core Algorithm

**Random Forest** or **XGBoost**, selected for:
- Robust handling of missing data
- Ability to capture non-linear relationships
- Feature importance transparency

### Feature Set

**From Baseline Data:**
- Storage capacity rating
- Average delivery lead time (days)
- Facility type and level
- Inventory count frequency

**From National Systems:**
- Budget utilization rates
- Historical stockout frequency
- Supplier delivery performance
- Disease surveillance alerts (DHIS2)

**Derived Features:**
- Days since last stockout
- Consumption velocity (units per day)
- Seasonality indicators
- Geographic risk factors (road accessibility, distance from hub)

### Purpose and Output

{% hint style="success" %}
**Note:** Tier 3 does **not** generate facility-level orders. Its outputs serve district supply officers for planning.
{% endhint %}

**Use Cases:**
- District-level demand forecasting for bulk procurement
- Stockout risk prediction (>80% probability in next 2 weeks)
- Proactive redistribution suggestions

**Example Output:**

```
HIGH RISK FACILITIES (Next 14 days)
───────────────────────────────────────────────────
Moroto HC III: 94% stockout risk for ACT
  → Recommendation: Emergency redistribution from
    Moroto Hospital (current stock: 180% of monthly need)

Abim HC II: 87% stockout risk for Amoxicillin
  → Recommendation: Expedite pending delivery
    (currently 3 days overdue)
```

---

## Algorithm Workflow with Human-in-the-Loop

### Facility-Level Workflow (Offline)

#### Step 1: Automated Forecast Generation

User opens the mobile app. The system automatically runs the Tier 1 Rule-Based Forecast for all tracked commodities.

#### Step 2: Forecast Review During Inventory

During weekly physical inventory count, the app displays:

```
┌────────────────────────────────────────────────────────┐
│ COARTEM (Artemether-Lumefantrine) 20mg Tablets         │
├────────────────────────────────────────────────────────┤
│ Current Stock:        420 tablets                      │
│ Recommended Order:    700 tablets                      │
│                                                        │
│ Calculation:                                           │
│   • 3-month average consumption: 800 tablets/month     │
│   • Malaria season adjustment:   +200 tablets (+25%)   │
│   • Calculated need:             1,000 tablets         │
│   • Storage limit applied:       700 tablets           │
│     (Inadequate storage = 70% cap)                     │
│                                                        │
│ [Accept]   [Adjust Quantity]   [Mark as Override]      │
└────────────────────────────────────────────────────────┘
```

#### Step 3: Human Override Capability

If user selects "Adjust Quantity":
- Enter new quantity
- Select reason from dropdown:
  - Disease outbreak
  - Delivery delay expected
  - Storage issue resolved
  - Expiry concern
  - Community health campaign
  - Other (free text)

Override is logged with timestamp and user ID.

#### Step 4: Local Order Approval

Facility in-charge reviews and approves. Order saved locally in SQLite database.

### District-Level Workflow (Online)

| Step | Action |
|------|--------|
| **1. Model Execution** | District server runs Tier 3 ML model weekly |
| **2. Officer Review** | Supply officers review high-risk facilities, plan redistribution |
| **3. Intervention** | Send alerts, authorize redistribution, update delivery schedules |

---

## Model Evaluation and Validation

### Primary Success Metric: Stockout Frequency and Duration

The ultimate measure is **improved supply availability**, not forecast accuracy:

- Percentage of facilities experiencing stockouts per month
- Average stockout duration (days)
- Commodity availability rate

### Forecast Accuracy Metrics (Secondary)

| Performance Level | MAPE Range | Action |
|-------------------|------------|--------|
| Target | <20% | Routine monitoring |
| Acceptable | 20-30% | Monitor closely |
| Review | >30% | Algorithm investigation triggered |

### Override Analysis

{% hint style="info" %}
**High override rates are signals, not failures:**
- **>50% override rate:** Investigate forecasting assumptions
- **Clustered overrides by reason:** Systemic supply chain issue
- **<5% override rate:** Users may not be engaging with system
{% endhint %}

---

## Technology Stack (Open Source)

| Component | Technology |
|-----------|------------|
| Tier 1 Mobile App | React Native, SQLite |
| Tier 2 Forecasting | Python, `hierarchicalforecast` library, pandas |
| Tier 3 ML | Python, scikit-learn (Random Forest), XGBoost |
| Data Sync | RESTful APIs over HTTPS with TLS 1.2+ |
| Database | PostgreSQL (district/central), SQLite (mobile) |

---

## Transparency and Explainability

Every forecast must be explainable in simple terms:

**Tier 1 Example:**
> "Based on your last 3 months of usage, adjusted for malaria season (+30%), limited by your storage capacity."

**Tier 3 Example:**
> "High stockout risk due to: delivery delays (40% contribution), low recent stock levels (35%), nearby facility outbreaks (25%)."

---

## Risks and Mitigation

| Risk | Mitigation |
|------|------------|
| **Model Drift** | Monthly automated retraining, quarterly governance review, automated MAPE alerts |
| **Data Quality Issues** | Quality dashboards, validation rules in app, facility feedback loop |
| **User Trust** | Transparent explanations, override capability, participatory design |
| **Infrastructure Failure** | Tier 1 fully offline, district server redundancy, CSV export fallback |
| **Misuse/Gaming** | Audit trails, anomaly detection, supervisory dashboards |

---

## Success Criteria

After 12 months of deployment:

| Metric | Target |
|--------|--------|
| Stockout Reduction | 30% reduction vs. baseline |
| Duration Improvement | 40% reduction in average stockout duration |
| User Adoption | >80% of facilities actively using forecasts |
| Override Appropriateness | <20% of overrides flagged as anomalous |
| System Reliability | >95% uptime for Tier 1 offline functionality |
| Data Quality | >70% of facility data passing quality thresholds |

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 2.0 (Post-Technical Validation) |
| **Date** | November 2025 |
| **Authors** | Gideon Abako, Timothy Kavuma (IFRAD) |
| **Validator** | Ojok Ivan, Kyambogo University |
| **License** | CC BY 4.0 |

### Key Changes in v2.0

- **Tier 2 Model:** Replaced standard ETS with Hierarchical Exponential Smoothing (HES)
- **Cold Start Protocol:** Added detailed facility matching criteria
- **Data Sync Protocol:** Specified bidirectional workflow
- **Model Selection Rationale:** Added comprehensive comparison section
- **Open Source Implementation:** Specified Python libraries for replication
