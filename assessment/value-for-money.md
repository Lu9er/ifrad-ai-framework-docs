# Value for Money Assessment

**Quantified Value for Money Assessment** **AI-Optimized Health Supply Chain Framework - Uganda Pilot Implementation**

**Version 1.0 | November 2025**

***

## Document Information

| Field                 | Value                                                          |
| --------------------- | -------------------------------------------------------------- |
| **Project**           | Optimizing Aid Supply Chain with Local Insights in Uganda      |
| **Lead Organization** | International Foundation for Recovery and Development (IFRAD)  |
| **Funding Source**    | Elrha Humanitarian Innovation Fund (UK FCDO)                   |
| **Status**            | Phase 1 Complete - Awaiting MOH/NMS Financial Data for Phase 2 |
| **Next Version**      | Version 2.0 (with quantified calculations) expected May 2026   |

***

## Executive Summary

This document provides the quantified value for money (VfM) assessment framework for the offline-first AI supply chain optimization system piloted across 10 health facilities in Karamoja and Southwestern Uganda.

The assessment follows the **4E Model**:

* **Economy** (Spending Less)
* **Efficiency** (Spending Well)
* **Effectiveness** (Spending Wisely)
* **Equity** (Spending Fairly)

### Data Availability Status

The baseline assessment captured operational performance metrics but encountered significant financial data gaps:

| Data Type                | Status                                     |
| ------------------------ | ------------------------------------------ |
| Budget utilization       | Unavailable for 5 of 10 facilities         |
| Commodity expiry costs   | Not tracked at facility level              |
| Emergency delivery costs | District level only, no facility breakdown |
| Health worker wages      | Held centrally by MOH                      |

### This Document Provides

* Complete VfM calculation methodology with worked formulas
* Verified baseline operational metrics from pilot facilities
* Clearly marked placeholders for MOH/NMS financial data
* Data collection guide for government partners
* Sensitivity analysis framework for scenario modeling

***

## Key Findings from Available Data

Baseline operational metrics demonstrate substantial improvement potential:

| Finding                                       | Value                                     |
| --------------------------------------------- | ----------------------------------------- |
| Facilities experiencing stockouts             | **100%** (10 of 10)                       |
| Average stockout duration (remote facilities) | **Up to 120 days** (Nakivale)             |
| Connectivity reliability                      | Unreliable at **89%** of facilities       |
| Storage capacity-stockout correlation         | **r = -0.695** (strong negative)          |
| Emergency procurement frequency               | Quarterly to monthly at HC III facilities |

***

## Baseline Performance Data

Data collected from 10 pilot facilities (2 regional referral hospitals, 8 spoke health centers) across four districts: Moroto, Amudat, Mbarara, and Isingiro.

**Data Collection Period:** August 25 - September 6, 2025 **Source:** IFRAD Baseline Assessment Report (October 2025), 33 quantitative surveys, 31 key informant interviews

***

## Projected Improvements

Based on the offline-first AI framework design with three-tier forecasting architecture:

| Performance Area              | Target Improvement                 | Source                           |
| ----------------------------- | ---------------------------------- | -------------------------------- |
| Stockout duration reduction   | 40% reduction                      | Needs Assessment Report Dec 2024 |
| Delivery delay reduction      | 30% reduction (rainy season)       | Needs Assessment Report Dec 2024 |
| Forecast accuracy (Tiers 2-3) | ≥85% prediction accuracy           | Needs Assessment Report Dec 2024 |
| Ordering cycle time           | From 2 weeks to 2 days             | Technical Validation Report      |
| Equity target                 | ≤5% accuracy disparity urban/rural | Needs Assessment Report Dec 2024 |

***

## Financial ROI Calculation

Financial ROI measures cash-releasable savings through reduced waste (expired commodities) and avoided emergency logistics costs.

### Formula

```
ROI = (Total Savings - Operating Cost) / Operating Cost × 100%
```

### Variable Definitions and Data Status

| Variable              | Definition                                                        | Data Source/Status        |
| --------------------- | ----------------------------------------------------------------- | ------------------------- |
| **Expiry Savings**    | (Baseline Expiry Rate - Current Rate) × Unit Cost × Annual Volume | MOH/NMS REQUIRED          |
| **Logistics Savings** | (Baseline Emergency Runs - Current Runs) × Cost per Run           | MOH/NMS REQUIRED          |
| **Operating Cost**    | Annual server hosting + maintenance + technical support           | PROJECT DATA: £50,000 GBP |

### Worked Example

| Component                                  | Value (UGX)                   |
| ------------------------------------------ | ----------------------------- |
| Baseline annual expiries (value)           | \[PLACEHOLDER - MOH/NMS DATA] |
| Projected expiries with AI (40% reduction) | \[AUTO-CALCULATED]            |
| **Expiry Savings**                         | \[AUTO-CALCULATED]            |
| Baseline emergency runs/year               | \[PLACEHOLDER - MOH/NMS DATA] |
| Cost per emergency run                     | \[PLACEHOLDER - MOH/NMS DATA] |
| **Logistics Savings**                      | \[AUTO-CALCULATED]            |
| Annual operating cost                      | \[UGX equivalent of £50,000]  |
| **FINANCIAL ROI**                          | \[CALCULATED UPON DATA INPUT] |

***

## Social Return on Investment (SROI)

SROI monetizes efficiency gains by valuing staff time freed from manual ordering processes.

### Formula

```
Value of Time Released = Hours Saved × Cycles per Year × Average Hourly Wage
```

### Time Calculation Basis

| Metric                  | Current          | With AI System  | Savings      |
| ----------------------- | ---------------- | --------------- | ------------ |
| Time per ordering cycle | 2 weeks (80 hrs) | 2 days (16 hrs) | **64 hours** |

**Rationale:** Staff currently spend \~1 week collecting data from paper records, preparing orders, and entering into multiple systems (DHIS2, eAFYA, CSSP). AI system reduces this to 2 days for data review and order approval only.

### Worked Example (10 Pilot Facilities)

| Component                             | Value                          |
| ------------------------------------- | ------------------------------ |
| Hours saved per ordering cycle        | 64 hours                       |
| Ordering cycles per facility per year | \[PLACEHOLDER - assume 12]     |
| Number of pilot facilities            | 10                             |
| **Total hours saved annually**        | 64 × 12 × 10 = **7,680 hours** |
| Average hourly wage (UGX)             | \[PLACEHOLDER - MOH PAYROLL]   |
| **SROI (Monetized Time Value)**       | \[CALCULATED UPON DATA INPUT]  |

{% hint style="info" %}
**Interpretation:** This monetized value represents health system efficiency gains. Staff can redirect freed time to direct patient care.

This is classified as **'time-releasable'** rather than **'cash-releasable'** value.
{% endhint %}

***

## Data Collection Guide for MOH/NMS

### National Medical Stores (NMS) Data Requirements

| Data Element                   | Specification                                                                     | Collection Method                               |
| ------------------------------ | --------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Commodity Expiry Value**     | Total UGX value of expired medicines at 10 pilot facilities (Aug 2024 - Aug 2025) | NMS invoices matched to facility expiry reports |
| **Top 10 Expired Commodities** | Most frequently expired items with unit costs                                     | NMS procurement database                        |
| **Emergency Delivery Costs**   | Average cost per emergency delivery (fuel, per diem, vehicle)                     | District fuel logs and vehicle manifests        |

### Ministry of Health Data Requirements

| Data Element                 | Specification                                                           | Collection Method                         |
| ---------------------------- | ----------------------------------------------------------------------- | ----------------------------------------- |
| **Health Worker Wages**      | Average gross hourly wage for pharmacists/clinical officers in ordering | MOH payroll (annual salary / 2,080 hours) |
| **Ordering Cycle Frequency** | Number of ordering cycles per facility per year                         | District procurement schedules            |
| **System Operating Costs**   | Annual hosting, maintenance, support in UGX                             | MOH budget office                         |

### Data Validation Protocol

1. Verify expiry data against facility-level stock cards (avoid double-counting)
2. Cross-reference emergency delivery costs against district fuel logs
3. Calculate blended hourly wage including benefits
4. Document assumptions for any estimated values
5. Use **Bank of Uganda midpoint exchange rate (Dec 31, 2025)** for GBP→UGX
6. Obtain district health office sign-off before finalizing

***

## Sensitivity Analysis Framework

Given uncertainty in baseline financial data, sensitivity analysis models how VfM outcomes vary under different scenarios.

### Key Variables for Scenario Modeling

| Variable                     | Conservative | Optimistic |
| ---------------------------- | ------------ | ---------- |
| Stockout reduction           | 20%          | 50%        |
| Expiry reduction             | 15%          | 40%        |
| Emergency delivery reduction | 20% fewer    | 40% fewer  |
| Time savings per cycle       | 40 hours     | 80 hours   |

### Break-Even Analysis

A neutral Financial ROI (0%) indicates the system **pays for itself** through waste reduction without generating additional cash savings.

```
Break-Even Condition: Minimum Required Savings = Annual Operating Cost
```

***

## Glossary

| Term                       | Definition                                                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **4E Model**               | Framework: Economy (spending less), Efficiency (spending well), Effectiveness (spending wisely), Equity (spending fairly) |
| **Cash-releasable**        | Savings that could be returned to budget (e.g., reduced expiries, fuel costs)                                             |
| **Time-releasable**        | Staff time freed from admin tasks, redirected to patient care                                                             |
| **MAPE**                   | Mean Absolute Percentage Error - forecast accuracy metric                                                                 |
| **Offline-first**          | Systems function fully without internet, syncing when available                                                           |
| **Three-tier forecasting** | Tier 1: Rule-based (offline), Tier 2: Statistical (intermittent), Tier 3: ML (district/central)                           |
| **Tracer commodities**     | Key essential medicines monitored for stockout tracking                                                                   |

***

## Document Information

| Field        | Value                                                       |
| ------------ | ----------------------------------------------------------- |
| **Version**  | 1.0 (Methodology Complete)                                  |
| **Date**     | November 2025                                               |
| **Author**   | IFRAD                                                       |
| **Contact**  | ifraduganda@gmail.com                                       |
| **Audience** | Elrha HIF Program, Uganda MOH, NMS, District Health Offices |
