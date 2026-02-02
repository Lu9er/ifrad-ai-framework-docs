# Ethical Guidelines

**Ethical Guidelines for Humanitarian AI Systems**
**Principles for Ethical Design, Development, and Deployment in Health Supply Chain Optimization**

**Version 3.0 | November 2025**

---

## Executive Summary

Artificial Intelligence is reshaping how humanitarian health supply chains anticipate, procure, and deliver medicines in Uganda and across East Africa. This transformation introduces both technological promise and deep ethical responsibility.

These Ethical Guidelines establish the framework for all design, development, and deployment of AI tools for supply chain optimization in humanitarian and fragile settings. Developed by IFRAD under the Elrha Humanitarian Innovation Fund, they govern the AI Framework for Health Supply Chain Optimization currently being implemented across ten facilities in Karamoja and Southwestern Uganda.

{% hint style="success" %}
**First of Its Kind**

This is the first comprehensive ethical framework for humanitarian AI supply chain systems **developed in Africa for African contexts**.
{% endhint %}

### Alignment

These guidelines align with:
- Uganda's National Development Plan IV
- Sphere Humanitarian Charter
- Uganda's Data Protection and Privacy Act (2019)
- WHO Ethics and Governance of AI for Health (2021)

---

## Purpose and Scope

These guidelines accompany the AI Framework for Health Supply Chain Optimization, forming its ethical foundation. They define **minimum ethical conditions** under which AI may be used to inform decisions affecting medicine availability, nutrition commodities, and lifesaving supplies.

**Applies to:** Governments, NGOs, software developers, donors, and research institutions creating or funding AI systems for humanitarian supply chains.

**Governs:** The three-tier forecasting system:
- **Tier 1:** Rule-based forecasting (offline, facility level)
- **Tier 2:** Hierarchical statistical forecasting (district servers)
- **Tier 3:** Machine learning forecasting (central level)

---

## Humanitarian Ethical Mandate

{% hint style="warning" %}
**Humanitarian technology must begin from humanity, not efficiency.**
{% endhint %}

Every AI system deployed under humanitarian mandate must uphold the four humanitarian protection principles from the **Sphere Handbook (2018)**:

1. Avoid exposing people to further harm
2. Provide access to impartial assistance
3. Protect people from physical and psychological abuse
4. Assist people to claim rights and recover dignity

For AI systems, these translate into: **transparency** in automated decisions, **equity** in data representation, and the **right to human oversight**.

---

## Guiding Principles

### 1. Do No Harm

All AI activities must undergo pre-deployment ethical risk assessments evaluating potential harm to individuals, communities, or institutions.

{% hint style="danger" %}
The system must **never autonomously execute** a recommendation without validated human approval at decision points appropriate to the tier and context.
{% endhint %}

**Tier-Specific Safeguards:**

| Tier | Safeguard |
|------|-----------|
| **Tier 1 (Facility)** | Recommendations are advisory only. Staff must manually approve all orders through weekly inventory review |
| **Tier 2 (District)** | Statistical forecasts supplement but cannot override facility decisions |
| **Tier 3 (Central)** | ML predictions require human validation before redistribution decisions |

---

### 2. Equity and Inclusion

Models must be trained and tested on data reflecting facility types, refugee settlements, and low-connectivity regions.

**Concrete Equity Mechanisms Implemented:**

| Mechanism | Implementation |
|-----------|----------------|
| **Storage Capacity Adjustment** | Very inadequate storage: 50% cap. Inadequate: 70% cap. Addresses r = -0.695 correlation |
| **Refugee Facility Prioritization** | 1.2x weighting in allocation algorithms for refugee settlements |
| **Nutrition Program Integration** | Integration with WHO/WFP/UNICEF/UNHCR Nutrition Appointment Platform |
| **Inventory Frequency** | Auto-triggers weekly count recommendations for facilities with >4 stockouts annually |
| **Cold Start Protocols** | New facilities use stratified baselines matched by type, region, and refugee-serving status. Never initialize rural facilities using urban averages |

---

### 3. Transparency and Explainability

Every AI recommendation must be intelligible to human operators.

**Tier-Specific Requirements:**

| Tier | Transparency Requirement |
|------|-------------------------|
| **Tier 1** | Plain language: *"Based on 3-month average (800 tablets) + Malaria Season adjustment (+200) = 1,000 calculated. Capped at 700 due to Inadequate storage rating."* |
| **Tier 2** | Model confidence intervals and data quality flags when records incomplete |
| **Tier 3** | Feature importance rankings showing which variables most influenced predictions |

---

### 4. Accountability and Oversight

Responsibility for outcomes rests with **human actors and institutions**, not algorithms.

Oversight mechanisms include:
- Governance committees
- Audit trails
- Incident-reporting protocols (consistent with Elrha's Incident Prevention and Management Policy)

---

### 5. Local Edits Always Win

{% hint style="info" %}
**FOUNDATIONAL PRINCIPLE**

When conflicts arise between system recommendations and user adjustments, the **facility worker's local edit is preserved**.
{% endhint %}

- System logs conflicts for district review
- Cannot retroactively override without explicit facility consent
- Acknowledges frontline workers have ground truth

**Human-in-the-Loop by Context:**

| Context | Requirement |
|---------|-------------|
| Routine replenishment | Facility staff manual approval during weekly review |
| Emergency redistribution | Explicit district-level approval required (no automation) |
| Outbreak response | Clinical officer must activate outbreak mode |
| Delivery delays | District supply officer reviews redistribution recommendations |

**Override Logging:**
- User ID, timestamp, original recommendation, final decision, mandatory reason
- High override rates flag forecasting errors, not user penalties
- **No-blame principle** applies

---

### 6. Privacy and Data Protection

Data collection and processing must comply with **Uganda's Data Protection and Privacy Act (2019)**.

**Offline Data Protection:**
- AES-256 encryption using Android Keystore System
- Keys generated in hardware-backed secure enclaves
- Never appear unencrypted in memory or file storage
- 12-month active retention, archived after upload confirmation

---

### 7. Frontline Worker Protection

{% hint style="success" %}
**Stakeholder validation identified frontline worker burden as a critical ethical concern.**

Health workers are already overburdened—digital systems must **reduce**, not increase, their workload.
{% endhint %}

**Worker Protection Mechanisms:**

| Mechanism | Implementation |
|-----------|----------------|
| **Administrative Burden Reduction** | Auto-save, single-tap adjustments, auto-generated paper forms matching HMIS 105 |
| **Battery Optimization** | Auto-sync only when battery >20% AND device not in active use |
| **Cognitive Load Reduction** | Single-column layouts, 48dp touch targets, 5-inch minimum device support |
| **Training Minimization** | Embedded video guides and progressive interface complexity |
| **Device Charging** | Offline-first architecture, low computational requirements |

---

## Data Governance

Developers and implementing partners must:

- Collect **only necessary data** (no patient-level health data)
- Obtain **explicit informed consent** for primary data collection
- Use **anonymization** for all storage and transmission
- Maintain **7-year retention schedules** per Elrha requirements
- Conduct **annual data protection audits**
- **Never transfer data to third parties** without written authorization

---

## Algorithmic Integrity

### Documentation Requirements

Each model must document:
- Algorithm type and selection rationale
- Feature engineering and preprocessing
- Known limitations and failure modes
- Validation methodology and metrics
- Override patterns and refinement history

### Quarterly Algorithmic Audits

Audits assess:
- Forecast accuracy by facility type (refugee vs. non-refugee, HC II vs. RRH)
- Stockout reduction patterns across regions
- Override rates and reasons by facility
- Emergency procurement frequency changes
- Storage capacity constraint effectiveness

**Audit results must be transparent to donors and government partners.**

---

## Risk Management

### Ethical Risk Register

Implementers must maintain a register identifying:
- Data leakage risks
- Biased forecasts disadvantaging specific facilities/regions
- Misuse of analytics for political manipulation
- Over-reliance on automation
- System failure during critical decisions
- Increased frontline worker burden

### Incident Response

| Requirement | Specification |
|-------------|---------------|
| **Reporting Timeline** | Within 24 hours of discovery |
| **Offline Adjustment** | Clock starts when facility syncs or reports via phone |
| **Report Contents** | Root cause analysis, affected facilities, immediate mitigation, corrective measures |
| **Serious Incidents** | Require public disclosure to affected communities |

---

## Governance Structure

### Forecasting Governance Committee

**Convenes quarterly to review compliance, approve updates, and manage grievances.**

**Membership:**
- Ministry of Health (Pharmacy Division)
- District Health Officers (rotating)
- Facility users (nominated by peers)
- Technical partner (IFRAD or successor)
- Academic validator (Kyambogo University)

**Responsibilities:**
- Quarterly review of model performance and stockout trends
- Analysis of override patterns
- Approval of algorithm changes
- Recommendations for framework refinement
- Advocacy for addressing infrastructure inequities

---

## Alignment with Frameworks

| Framework | Alignment |
|-----------|-----------|
| Uganda National Development Plan IV | Full alignment |
| Uganda Data Protection and Privacy Act (2019) | Compliant |
| National eHealth Policy Framework (2013) | Aligned |
| African Union Continental AI Strategy (2024) | Aligned |
| WHO Ethics and Governance of AI for Health (2021) | Aligned |
| OECD AI Principles (2019) | Aligned |
| Sphere Humanitarian Charter (2018) | Foundational |

---

## Conclusion

{% hint style="success" %}
**Ethical innovation is not constraint but foundation for sustainable technological progress.**

These guidelines translate ethical principles into operational parameters:
- Storage capacity → algorithmic constraint
- Human oversight → tier-specific approval workflows
- Equity → concrete weighting factors
- Transparency → plain-language explanations
- Worker protection → battery optimization and auto-save
{% endhint %}

By grounding AI in humanitarian ethics informed by African operational realities, these guidelines assert both technical capability and moral leadership in defining how intelligent systems serve human need.

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 3.0 (Post-Validation) |
| **Date** | November 2025 |
| **Organization** | International Foundation for Recovery and Development (IFRAD) |
| **Technical Validator** | Ojok Ivan, Kyambogo University Department of Data Science, Networks & AI |
| **License** | CC BY 4.0 |
