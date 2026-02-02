# Integration Standards & Maturity Model

**For Ugandan Health Supply Chain Systems**
**Version 2.0 - Post-Validation Revision**

---

## Introduction

This document provides a realistic framework for data exchange across Uganda's health supply chain systems. It is grounded in the empirical findings of the 2025 Baseline Assessment, which revealed that **89% of facilities face unreliable internet**, and that system fragmentation and parallel paper-based workflows are the norm, not the exception.

{% hint style="info" %}
**Objective**

The objective is not to mandate an unattainable real-time integration, but to define a **pragmatic maturity model** that guides facilities from their current state of digital disconnection toward incremental levels of interoperability.
{% endhint %}

---

## Foundational Principles

### Grounding in Baseline Reality

The baseline assessment found:

| Finding | Implication |
|---------|-------------|
| **System Fragmentation** | Facilities operate multiple disconnected digital systems (DHIS2, eAFYA, CSSP, eLMIS) alongside paper records |
| **Infrastructure Gaps** | Unreliable power and internet prevent consistent use of API-dependent systems |

**Conclusion:** Current integration efforts must prioritize practical, incremental data exchange over complex, real-time API orchestration.

---

## Integration Maturity Model

This model defines a graduated pathway. The **current project focus is Level 2 (Opportunistic Sync)** as the primary target.

| Level | Description | Data Exchange Method | Target |
|-------|-------------|---------------------|--------|
| **Level 0: Paper-Centric** | All records on paper registers. No digital entry. | Physical transport of paper reports to district for manual entry | ~40% of HCIIs. Focus: Digitization at source |
| **Level 1: Single-System** | One digital system used (e.g., DHIS2 mobile), reducing duplicate entry but isolated | Weekly export of CSV/Excel files, emailed or physically transported | ~35% of facilities. Focus: Standardizing on offline-first app |
| **Level 2: Opportunistic Sync** | **PRIMARY TARGET.** Offline-first app used. Data synced when connectivity allows | App syncs JSON packets to central server when online. Pulls reports and forecasts | **80% of facilities by project end** |
| **Level 3: Real-Time** | Full API-based, real-time data exchange between national systems | RESTful APIs, FHIR resources. Assumes stable connectivity | Regional Referral Hospitals & District HQs only. Future state |

---

## System-Specific Integration Standards

### DHIS2 Integration

**Level 2 (Opportunistic Sync):**
- Central server periodically pulls aggregated consumption and stock data from DHIS2's API
- Pushes back: Tier 2/3 forecast recommendations, stockout alerts, override logs as analytics outputs

**Level 3 (Real-Time):**
- RESTful API and FHIR standards for bidirectional data exchange with sub-minute latency

### e-LMIS Integration

**Level 2 (Opportunistic Sync):**
- Facility-level orders (Tier 1 rule-based + human overrides) and stock counts collected via offline-first app
- During sync, data pushed to central queue → processed in batch jobs (every 4-6 hours) to update e-LMIS
- **Eliminates need for facility staff to use multiple systems**
- Delivery information pulled from e-LMIS and pushed to facilities during next sync

**Level 3 (Real-Time):**
- Direct RESTful API integration for real-time stock visibility and automated order submission

### eAFYA Integration

**Level 2 (Opportunistic Sync):**
- Patient attendance and service utilization data used at district level as Tier 3 forecasting feature
- District IT officers export CSV files weekly using built-in export functionality
- CSV files uploaded to district server via secure file transfer (SFTP or web interface)
- Data informs allocation but does not trigger real-time deductions at facility level

**Level 3 (Real-Time):**
- Direct integration for automatic deduction of dispensed commodities using FHIR MedicationRequest resources

### WFP LESS Integration

**Level 2 (Opportunistic Sync):**
- Delivery status updates pulled by central server via API calls (every 6-12 hours)
- Pushed to relevant facilities during next sync
- Provides near-real-time visibility into last-mile logistics
- Supports proactive redistribution planning when delivery delays detected

**Level 3 (Real-Time):**
- API endpoints for live shipment tracking and automated receipt confirmations

### Nutrition Appointment Platform (WHO/WFP/UNICEF/UNHCR)

This platform supports nutrition service delivery for vulnerable populations including refugees and climate-affected communities.

**Level 2 (Opportunistic Sync):**
- Central server pulls nutrition program enrollment data and appointment schedules (weekly)
- Data feeds into Tier 3 district-level forecasting models as demand signals for nutrition commodities (RUTF, micronutrient supplements, etc.)
- Supply adequacy indicators pushed back to Nutrition Appointment Platform to inform program planning

**Level 3 (Real-Time):**
- Real-time bidirectional integration where appointment confirmations automatically trigger commodity allocation adjustments

---

## Data Governance, Security, and Compliance

All data exchange must adhere to **Uganda's Data Protection and Privacy Act (2019)**.

### Security Standards

| Aspect | Standard |
|--------|----------|
| **Encryption (Transit)** | TLS 1.3 for all transmitted data |
| **Encryption (Rest)** | AES-256 for mobile devices via Android Keystore System |
| **Authentication** | Role-based access control (Principle of Least Privilege) |

**Roles:** Facility Staff, Store Manager, District Supply Officer, National Administrator

### Conflict Resolution Protocol

{% hint style="warning" %}
**CRITICAL DESIGN PRINCIPLE**

In case of conflicting data edits from multiple sources, the system applies a **'local user edits always win'** policy.
{% endhint %}

**Rationale:** The health worker physically present at the facility has ground truth. If a facility worker manually adjusts an order quantity after counting physical stock, and a district officer or automated system also modified it, the facility worker's version is preserved.

**Implementation:**
- All conflicts logged to override audit trail for district officer review
- District officer can contact facility to understand reasoning
- Cannot retroactively override without facility consent

**Exception:** During declared emergencies (disease outbreak, humanitarian crisis), district officers can flag orders for mandatory review, but facility's data remains system of record until facility explicitly approves changes.

### Data Validation Rules

The system enforces these plausibility checks at data entry and upon central receipt:

| Rule | Description |
|------|-------------|
| **Positive integers** | Order quantities must be positive integers |
| **Historical cap** | Orders cannot exceed 200% of 6-month historical maximum without override justification |
| **Facility type** | Orders must align with facility type capacity (e.g., HCII cannot order surgical equipment) |
| **Non-negative stock** | Stock levels cannot be negative |
| **Duplicate detection** | Check for identical facility ID, commodity code, and timestamp within 24-hour window |

### Audit Trails

All transactions must be logged with:

- User ID and role
- Timestamp (UTC)
- Action type (data entry, sync, override, forecast generation)
- Forecast tier that generated the recommendation (Tier 1, 2, or 3)
- Manual override justification (free text)

### Tier 2 Forecast Data Management

{% hint style="info" %}
**Architectural Note**

Tier 2 hierarchical statistical forecasts are pushed to facilities as **informational guidance only**. They are stored separately from the operational SQLite database to prevent concurrent write conflicts during background sync.

Facility staff can view Tier 2 forecasts in a read-only dashboard section but the primary Tier 1 rule-based forecast remains the default for order generation.
{% endhint %}

---

## Realistic Integration Scenarios

Based on baseline findings and stakeholder feedback, the following scenarios must be supported at Level 2:

### Disease Outbreaks

1. Facility uses offline app to mark **Outbreak Mode**
2. Justifies large manual order override (e.g., 300% increase in antimalarials)
3. Override synced to district, alerting them to potential redistribution need
4. Override audit log captures justification
5. System flags for district review **without blocking the order**

### Delivery Delays

1. District officer sees LESS update about road washout affecting Karamoja shipments
2. Uses central dashboard to manually reassign shipments to accessible facility
3. Reassignment pushed to facilities during next sync
4. Affected facility receives notification explaining delay and alternative pickup arrangements

### Budget Adjustments

1. Mid-year budget change entered into central system
2. New budget ceilings pushed to facilities during next sync
3. Facilities see updated, budget-aware Tier 2/3 forecast recommendations
4. Tier 1 rule-based forecast continues generating orders within new budget constraints automatically

### Power/Network Downtime

1. Core workflow continues uninterrupted on offline app
2. All data stored locally in encrypted SQLite database (WAL mode)
3. When connectivity restored (automatically detected or manually triggered), data synchronized
4. Automatic conflict resolution favors facility's local edits
5. **No operational delay** even after weeks of offline operation

### Nutrition Program Expansion

1. WFP scales up nutrition program in refugee settlements (e.g., +500 beneficiaries)
2. Nutrition Appointment Platform updates enrollment data
3. Central server pulls data during weekly scheduled sync
4. Tier 3 models automatically adjust predicted demand for RUTF and micronutrients
5. District officers receive alerts and can proactively reallocate stock before shortages occur

---

## Monitoring, Evaluation, and Compliance

### Key Performance Indicators

| Category | Metric | Target |
|----------|--------|--------|
| **Integration Maturity** | Facilities operating at Level 2+ | 80% by project end |
| **Technical Reliability** | Sync success rate | >95% |
| **Technical Reliability** | Data packet completeness | Monitored |
| **Technical Reliability** | Average time since last sync | Monitored per facility |
| **Forecast Accuracy** | MAPE Tier 1 | <30% |
| **Forecast Accuracy** | MAPE Tier 2 | <25% |
| **Forecast Accuracy** | MAPE Tier 3 | <20% |
| **User Engagement** | Daily active users | Monitored |
| **User Engagement** | Override frequency | <15% of orders |
| **Operational Impact** | Stockout frequency/duration | Reduced from baseline |
| **Operational Impact** | Time spent on supply chain tasks | 50% reduction from baseline |

### Compliance Audits (Semi-Annual)

| Area | Verification |
|------|--------------|
| **Technical Integration** | Data flows between systems, API endpoint functionality, error logs |
| **Data Quality** | Completeness, accuracy, timeliness of synced data, conflict resolution logs |
| **Usability** | Frontline worker interviews confirming workload reduction |
| **Privacy Compliance** | Audit trails, encryption implementation, role-based access controls |

---

## Post-Grant Implementation Roadmap

| Phase | Timeframe | Focus |
|-------|-----------|-------|
| **Phase 1** | Months 1-3 | Deploy offline-first mobile app with Tier 1 forecasting. No external system integration. Build user trust |
| **Phase 2** | Months 4-6 | Activate Level 2 integration with e-LMIS and DHIS2. Deploy district servers with Tier 2/3 forecasting |
| **Phase 3** | Months 7-9 | Add eAFYA, WFP LESS, Nutrition Appointment Platform integration at Level 2. Weekly data flows |
| **Phase 4** | Months 10-12 | Pilot Level 3 real-time integration at 2-3 regional referral hospitals. Evaluate before broader rollout |

### Governance Structure

| Body | Role |
|------|------|
| **District Health Officers** | Primary system owners, core partners throughout implementation |
| **MoH Technical Working Groups** | Policy-level review, national digital health strategy alignment |
| **Quarterly Review Committee** | Cross-stakeholder review of integration performance, override patterns, refinements |

---

## Conclusion

{% hint style="success" %}
**This integration framework is designed to meet Uganda's health supply chain where it currently operates, not where we wish it to be.**
{% endhint %}

By prioritizing **Level 2 (Opportunistic Sync)** integration, we provide a pragmatic path that respects infrastructure constraints while delivering tangible benefits:

- Reduced duplicate data entry
- Enhanced supply visibility
- Better-informed decision-making

The framework explicitly rejects integration approaches that would increase frontline worker burden or require always-online connectivity. Instead, it builds incrementally from the offline-first foundation, adding integration touchpoints only where they provide clear value without introducing new points of failure.

**This is integration designed for resilience, not fragility.**

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 2.0 (Post-Technical Validation) |
| **Date** | November 2025 |
| **Authors** | Gideon Abako, Timothy Kavuma (IFRAD) |
| **Validator** | Ojok Ivan, Kyambogo University |
| **License** | CC BY 4.0 |
