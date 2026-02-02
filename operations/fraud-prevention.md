# Fraud Prevention Handbook

**Fraud Prevention Protocol Handbook & Basic Deployment Roadmaps**
**Version 1.3 - Optimized for Solo-Staff Operations**

---

## Document Control

| Field | Value |
|-------|-------|
| **Version** | 1.3 (Optimized for Solo-Staff Operations) |
| **Date** | December 2025 |
| **Author** | IFRAD Technical Team |
| **Validation** | Kyambogo University, Ministry of Health, District Health Teams |
| **Classification** | Public - Open Access |

---

## Operational Context Notice

{% hint style="warning" %}
**Low-Resource Setting Constraints**

This handbook is designed for low-resource humanitarian settings (Karamoja/Refugee Settlements) where facilities operate with severe constraints:
- Staff shortages (below 65% capacity)
- Unreliable power
- Intermittent connectivity

**Design principle:** Controls must be usable by a single nurse in a solar-powered clinic.
{% endhint %}

**Primary legal record:** During Phase 1, the physical Stock Card remains the official legal record. The AI system functions as a decision-support layer. However, significant variances (>5%) trigger physical audits to prevent paper-based manipulation.

---

## Part 1: Fraud Prevention Protocols

### Purpose and Scope

This handbook establishes fraud prevention protocols for Uganda's AI Supply Chain Optimization Framework. It addresses risks identified through baseline assessments where **68% of facilities experience regular stockouts and inventory discrepancies**.

### Fraud Risk Categories

The framework targets four primary risk categories identified in the baseline:

| Category | Description | Examples |
|----------|-------------|----------|
| **Category A** | Inventory Discrepancies | Phantom stock entries, over-reporting consumption to hide theft, manipulation of expiry data |
| **Category B** | Data Quality Manipulation | Backdating records, falsifying patient numbers to justify higher allocations |
| **Category C** | Procurement Manipulation | Ghost delivery documentation, collusion enabled by lack of budget visibility |
| **Category D** | Stock Diversion | Unauthorized redistribution or private sale, often justified by "emergency" needs |

---

### Automated Detection Mechanisms

The AI framework uses automated alerts calibrated to **minimize false positives** in volatile contexts. The system cross-references morbidity data (from DHIS2) before flagging consumption anomalies.

{% hint style="info" %}
**Context-Aware Detection**

A spike in malaria drugs is not fraud if malaria cases have also spiked.
{% endhint %}

### Alert Thresholds

| Anomaly Type | Threshold | Context Check | Response |
|--------------|-----------|---------------|----------|
| **Consumption spike** | >30% above 3-month avg | Correlate with DHIS2 disease cases | Flag only if morbidity stable |
| **Stock count variance** | >5% system vs physical | None required | Mandatory physical audit |
| **Order frequency** | >2 emergency orders/30 days | Check for documented outbreaks | DHO review at weekly meeting |
| **Expiry losses** | >5% batch-level losses | Review storage conditions | Storage/distribution audit |

### Alert Response Timelines

Deadlines are tied to **workflow touchpoints**, not arbitrary hours:

| Stage | Timing |
|-------|--------|
| **System Detection** | Immediate |
| **Local Verification** | Before next order submission (prevents ignoring alerts) |
| **District Escalation** | Weekly DHO coordination meeting |

---

### Audit Trail & Data Security

#### Data Captured Automatically

| Element | Description |
|---------|-------------|
| **User ID** | Authenticated user for every transaction |
| **Timestamp** | Date/time of all entries |
| **Pre/Post Values** | Record of all data changes |

#### Location Verification (Battery-Optimized)

To preserve battery life on solar-dependent devices:

| Event Type | GPS Capture |
|------------|-------------|
| **High-Risk Events** | Active GPS (stock delivery receipt, physical counts, emergency orders) |
| **Routine Dispensing** | Static Facility ID only |

#### Data Security & Storage

| Aspect | Implementation |
|--------|----------------|
| **Encryption** | AES-256 via Android Keystore |
| **Storage Management** | Auto-archive logs >24 months to compressed storage |

### Paper-Digital Reconciliation (Anti-Loophole Clause)

| Rule | Description |
|------|-------------|
| **Primacy Rule** | Paper is the legal record |
| **The Check** | If variance between Digital and Paper exceeds **5%**, system triggers **Mandatory Physical Audit** by District Health Team within 14 days |

{% hint style="warning" %}
The paper record is not automatically accepted as truth if the digital variance is significant.
{% endhint %}

---

### Human Override and Emergency Protocols

Many HCIIs are run by a single staff member. "Two-person authorization" is impossible. We use a **Break-Glass Protocol**.

#### Break-Glass Protocol for Solo Staff

| Component | Protocol |
|-----------|----------|
| **Trigger** | Single staff member needs to perform high-risk action (emergency order, stock adjustment) |
| **Action** | Single-person override permitted |
| **System Flag** | Transaction automatically flagged for **Retroactive DHO Review** |
| **Documentation** | Drop-down menu selection (e.g., "Outbreak," "Transport Failure," "Staff Shortage") + optional text box |
| **Review** | Flagged transactions reviewed at next weekly DHO meeting |

#### Standard Review Triggers (Multiple Staff Available)

| Condition | Action |
|-----------|--------|
| AI Confidence <80% | Manual review by facility in-charge |
| High-Volume Redistribution | DHO approval required for moves affecting >500 patients |

---

### Verification Procedures

#### Stock Count Verification (Cycle Counting)

To reduce staff burden while maintaining accuracy:

| Aspect | Specification |
|--------|---------------|
| **Protocol** | Weekly Cycle Count: 3-5 items on rotating basis (not all 15 tracer items) |
| **Goal** | Every tracer item counted at least once per month |
| **Labor Reduction** | ~70% reduction in weekly counting burden |
| **Full Inventory** | Monthly count of all items per HMIS 105(6) requirements |

#### Delivery Verification

| Check | Method |
|-------|--------|
| **Documentation** | Delivery note vs. System Order matched at receipt |
| **Location** | GPS verified at point of delivery |

#### Whistleblowing

Confidential reporting channels aligned with safeguarding policies:

| Channel | Description |
|---------|-------------|
| **Internal** | Anonymous reporting function within mobile application |
| **District** | Direct reporting to DHO or integrity officer |
| **National** | Ministry of Health Inspectorate Division |

---

## Part 2: Deployment Roadmaps

### Deployment Philosophy

- **Offline-First:** Facilities with the poorest connectivity piloted first to validate architecture where most needed
- **Integration:** Framework is an interoperability layer, not a replacement for DHIS2 or NMS systems

### Phase 1: Pilot Validation (FUNDED)

| Aspect | Detail |
|--------|--------|
| **Timeline** | July - December 2025 (6 months) |
| **Funding** | Elrha HIF (£50,000) |
| **Target** | 10 baseline facilities (Karamoja & Southwestern Uganda) |
| **Objectives** | Validate technical specs, refine offline sync, test fraud thresholds, gather user feedback |

**Success Metrics:**
- 80% data capture rate
- Successful offline operation for 14+ days
- Staff satisfaction >70%

### Phase 2: District Expansion (SUBJECT TO FUTURE FUNDING)

| Aspect | Detail |
|--------|--------|
| **Status** | Unfunded - No commitment without new grants |
| **Target** | ~50 facilities in Karamoja/Nakivale |
| **Focus** | Validate cross-facility redistribution algorithms |

### Phase 3: Regional Scale-Up (SUBJECT TO FUTURE FUNDING)

| Aspect | Detail |
|--------|--------|
| **Status** | Unfunded |
| **Target** | ~200 facilities in Northern Uganda |
| **Focus** | Integration with WFP/UNHCR systems |

### Phase 4: National Adoption (SUBJECT TO FUTURE FUNDING)

| Aspect | Detail |
|--------|--------|
| **Status** | Unfunded |
| **Target** | National Health Information System integration |

---

### Training Sequence

Training is task-based and visual to accommodate varying digital literacy:

| Tier | Audience | Duration | Content |
|------|----------|----------|---------|
| **Tier 1** | Facility Staff | 4 hrs | Basic data entry, offline sync, handling AI alerts, Break-Glass protocol |
| **Tier 2** | In-Charges | 8 hrs | Dashboard interpretation, data quality monitoring |
| **Tier 3** | District Teams | 16 hrs | Analytics, fraud investigation, break-glass review |

### Integration Timelines

| Phase | Systems |
|-------|---------|
| **Phase 1 (Funded)** | DHIS2 (Morbidity data), eLMIS (Sync) |
| **Phase 2+ (Unfunded)** | CSSP (NMS Orders), eAFYA (Patient Data), WFP LESS (Refugee Supply) |

### Sustainability Mechanisms

| Type | Mechanism |
|------|-----------|
| **Technical** | Open-source codebase (GitHub), local hosting (MoH servers), standard APIs |
| **Institutional** | MoH ownership post-pilot, "Train-the-Trainer" model |
| **Financial** | Cost recovery via reduced procurement waste (projected 25% savings) |

---

## Annex A: Summary of Operational Adaptations

| Issue | Standard Approach | Adapted Protocol (v1.3) |
|-------|-------------------|-------------------------|
| Staff Shortages | 2-person authorization | Break-glass protocol with retroactive DHO audit |
| Data Entry Burden | Typed explanations | Drop-down menus for common override reasons |
| Inventory Labor | Weekly full counts | Weekly Cycle Counts (3-5 items/week) |
| Unreliable Power | GPS on every transaction | GPS on high-risk events only |
| Alert Fatigue | 24-hour response deadline | Response tied to Next Order Submission |
| Volatile Demand | Fixed fraud thresholds | Thresholds correlated with DHIS2 morbidity |
| Legal Ambiguity | Paper vs. Digital conflict | >5% variance triggers mandatory physical audit |
| Storage Limits | Unlimited log retention | Auto-archive logs >24 months |

---

## Document References

- Uganda Essential Medicines and Health Supplies Management Manual 2023
- IFRAD Baseline Assessment Report, October 2025
- AI Supply Chain Framework Technical Specifications v0.1
- Kyambogo University Technical Validation Report
- Elrha Incident Reporting and Safeguarding Policy
