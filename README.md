# AI Framework for Health Supply Chain Optimization

**Technical Documentation for Humanitarian AI Systems in Uganda**

---

## Overview

This framework provides a comprehensive approach to optimizing health supply chains in low-resource humanitarian settings. Developed by the International Foundation for Recovery and Development (IFRAD) under the Elrha Humanitarian Innovation Fund, it addresses the unique challenges of Uganda's health system where **89% of facilities experience unreliable connectivity** and infrastructure constraints directly impact medicine availability.

### Core Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Offline-First** | All critical functions work without internet connectivity |
| **Human-in-the-Loop** | AI provides recommendations; health workers make final decisions |
| **Infrastructure-Aware** | Storage capacity treated as a hard constraint, not a variable |
| **Paper-Digital Harmony** | System augments, not replaces, existing paper workflows |

### Three-Tier Forecasting Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    CENTRAL DATA HUB                         │
│         National reporting & system integration             │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ Periodic ETL
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   DISTRICT SERVICES                         │
│    Tier 2: Hierarchical Statistical Forecasting (HES)       │
│    Tier 3: Machine Learning Models (Random Forest/XGBoost)  │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ Opportunistic Sync
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   FACILITY LEVEL                            │
│         Tier 1: Rule-Based Forecasting (Offline)            │
│              SQLite + AES-256 Encryption                    │
└─────────────────────────────────────────────────────────────┘
```

### Key Baseline Findings

The framework directly addresses findings from the 2025 Baseline Assessment:

- **Storage-Stockout Correlation**: r = -0.695 (strong negative)
- **Connectivity Failure Rate**: 89% of facilities
- **Stockout Prevalence**: 100% of pilot facilities experienced stockouts
- **Maximum Stockout Duration**: 120 days at refugee-serving facilities

---

## Documentation Structure

### Technical Specifications
- [System Architecture](technical/system-architecture.md) - Distributed system design for offline-first operations
- [Predictive Demand Algorithm](technical/predictive-demand-algorithm.md) - Three-tier forecasting methodology
- [Implementation Manual](technical/implementation-manual.md) - Android-specific development guide
- [Offline Workflow](technical/offline-workflow.md) - Interface design and data flow

### Integration & Standards
- [Integration Standards](integration/integration-standards.md) - Maturity model and system interoperability

### Operations
- [Decision-Making Library](operations/decision-making-library.md) - Override protocols and use cases
- [Fraud Prevention Handbook](operations/fraud-prevention.md) - Detection mechanisms and audit trails

### Ethics & Compliance
- [Ethical Guidelines](ethics/ethical-guidelines.md) - Humanitarian AI principles

### Assessment
- [Value for Money](assessment/value-for-money.md) - Quantified impact assessment

---

## Project Information

| Field | Details |
|-------|---------|
| **Lead Organization** | International Foundation for Recovery and Development (IFRAD) |
| **Funding** | Elrha Humanitarian Innovation Fund (UK FCDO) |
| **Pilot Region** | Karamoja and Southwestern Uganda |
| **Pilot Facilities** | 10 (2 Regional Referral Hospitals, 8 Health Centers) |
| **Phase** | Phase 1 Pilot (July - December 2025) |

---

*Version 2.0 - Post-Validation Revision | December 2025*
