# Offline Workflow

**Offline-First System Specification & Interface Design**
**Version 2.0 - Post-Validation Revision**

---

## Introduction

This document details the offline-first workflow and user interface for a health supply chain system designed for the realities of Uganda's rural and humanitarian settings. It is a direct response to the 2025 Baseline Assessment, which found:

- **89% of facilities** have unreliable internet and power
- Staff have **low to medium digital literacy**
- Health workers **revert to paper** as the authoritative record when digital systems fail

{% hint style="success" %}
**Design Philosophy**

The design prioritizes **resilience over real-time connectivity**, **human judgment over rigid automation**, and **seamless integration with (not replacement of) existing paper-based workflows**.
{% endhint %}

---

## Design Philosophy & Core Principles

| Principle | Description |
|-----------|-------------|
| **Offline-first is non-negotiable** | Every critical function must be available without an internet connection |
| **Human-in-the-Loop** | The system provides recommendations; health workers make final decisions. Override is a feature, not a bug |
| **Paper-digital harmony** | System generates paper outputs matching existing MoH registers (e.g., HMIS 105) |
| **Minimalist and guided UI** | Simple interfaces with clear language and visual cues for varying tech comfort levels |

---

## Detailed Offline Operation Specification

### Local Database Architecture

| Specification | Implementation |
|--------------|----------------|
| **Database Engine** | SQLite configured in Write-Ahead Logging (WAL) mode |
| **WAL Configuration** | `PRAGMA journal_mode=WAL; PRAGMA synchronous=NORMAL;` |
| **Target Size** | <50MB per facility for entry-level Android device performance |

{% hint style="info" %}
**Rationale for WAL**

Standard SQLite journal mode blocks all reads during write operations. WAL mode prevents database locks during background sync, allowing facility staff to continue data entry while uploads occur.
{% endhint %}

### Stored Data

- Facility profile and storage capacity rating
- Master list of essential medicines and commodities
- Last 12 months of facility-specific consumption data (active operational data)
- Current stock levels and pending orders
- User credentials for offline authentication (hashed using bcrypt)

### Data Encryption & Security

| Aspect | Implementation |
|--------|----------------|
| **Encryption Standard** | AES-256 encryption for entire SQLite database file |
| **Key Management** | Android Keystore System with hardware-backed secure enclaves |
| **Key Lifecycle** | Generated at first app installation, stored exclusively in Android Keystore |
| **Compliance** | Uganda Data Protection and Privacy Act (2019) |

### Data Persistence & Auto-Save

- All data is saved **instantly** to the local database
- Auto-save is **continuous and invisible** to the user
- **No explicit 'Save' buttons** - reduces cognitive load
- Prevents data loss if app closes unexpectedly or device loses power

### Data Archiving & Purging Policy

| Policy | Implementation |
|--------|----------------|
| **Active Retention** | Last 12 months of consumption data and stock counts |
| **Archive Trigger** | Automated monthly job moves data >12 months to compressed .zip files |
| **Archive Upload** | Uploaded to district server during sync, then deleted after confirmation |
| **Emergency Purge** | Triggered if database exceeds 45MB (90% of target) |

**Emergency Purge Process:**
1. User receives notification: "Storage limit approaching. Archiving old data."
2. Oldest archived data removed first
3. If still over 45MB, oldest active data beyond 12-month window archived
4. Archive upload occurs before deletion to prevent data loss

---

## Tier 1 Offline Forecasting Integration

The Tier 1 Rule-Based Forecast runs entirely locally. This is the **primary forecasting mechanism** available offline.

**Example Logic Displayed to User:**

```
┌────────────────────────────────────────────────────────┐
│ Forecast: 1,000 Coartem tablets                        │
│                                                        │
│ Calculated from:                                       │
│   • Your 3-month average (800 tablets)                 │
│   • Rainy season adjustment (+200)                     │
│   • Capped at 1,000 due to 'Inadequate' storage        │
│                                                        │
│ [Slider to adjust quantity]                            │
└────────────────────────────────────────────────────────┘
```

### Tier 2 Statistical Forecasts (Informational Only)

When facilities sync, they receive Tier 2 hierarchical statistical forecasts from district models.

| Aspect | Implementation |
|--------|----------------|
| **Storage Location** | Separate read-only JSON files (not in SQLite database) |
| **User Interface** | Read-only 'District Insights' section of the app |
| **Purpose** | Context for comparison, cannot be used directly for order generation |

{% hint style="info" %}
**Rationale:** Separating informational forecast data from operational transaction data prevents potential conflicts and keeps the sync process simple and reliable.
{% endhint %}

---

## Synchronization Logic

### Automatic Sync Triggers

| Condition | Description |
|-----------|-------------|
| **Network Detection** | Background service checks for connectivity every 15 minutes |
| **Battery Threshold** | Automatic sync only when device battery >20% |
| **Device Activity** | Sync runs only when device not in active use (screen off or app in background) |
| **Manual Override** | Users can trigger 'Sync Now' at any time, bypassing other checks |

### Upload Process

| Aspect | Specification |
|--------|---------------|
| **Payload** | Batched JSON packets: stock counts, orders with overrides, consumption data, sync metadata |
| **Compression** | GZIP to minimize data transfer costs |
| **Retry Logic** | Exponential backoff (initial 30s, maximum 15min) |

### Download Process

| Aspect | Specification |
|--------|---------------|
| **Payload** | Delivery schedules, budget notifications, Tier 2 forecasts (JSON), system messages, credential tokens |
| **Processing** | Downloaded data validated for schema compliance and plausibility before writing |

### Conflict Resolution Principle

{% hint style="warning" %}
**Local User Edits Always Win**

The system trusts the health worker on the ground. If a user adjusted an order offline and the central system also modified it, the user's version is preserved. Conflicts are logged to the override audit trail for district officer review.
{% endhint %}

---

## Offline Authentication & Credential Management

| Aspect | Implementation |
|--------|----------------|
| **Credential Storage** | Cached locally (hashed using bcrypt with salt) |
| **Credential Sync** | Updates checked during each successful server connection |
| **Offline Validity** | **90 days** of offline operation |
| **Expiry Warning** | Appears at 75 days (15 days before expiry) |
| **Permission Changes** | Sync immediately upon next connection |

---

## Paper Backup Integration

{% hint style="success" %}
**Paper-digital harmony is a core design principle. The system does not replace paper—it augments it.**
{% endhint %}

- **Printable Forms:** Every data entry and order screen includes a 'Print' button
- **Output Format:** Pre-populated paper form matching official MoH registers (e.g., HMIS 105)

**Benefits:**
- Data can be transported physically if needed
- Staff have a familiar trusted record
- Continuity of operations during prolonged power failure or device issues

---

## User Roles & Pragmatic Workflows

| Role | Primary Offline Tasks | Key Interface Features |
|------|----------------------|------------------------|
| **Facility Staff** | Weekly stock counts, place orders, report stockouts | Home dashboard with stock alerts, one-tap order adjustment, offline data entry forms |
| **Store Manager** | Manage stock ledger, review expiry dates, approve facility orders | Expiry alert screen, stock ledger view, order approval queue |
| **District Supply Officer** | Review facility status, approve bulk orders, plan redistributions | District dashboard (requires sync), facility risk list, override pattern reports |

---

## UI Wireframe Descriptions

### Login Screen

**Features:**
- Clear, large **'Work Offline'** button prominently displayed
- Persistent visual indicator showing 'Offline Mode' or 'Last Synced: 2 days ago'

> **Justification:** Addresses the baseline quote: *"Right now, unless you buy MBs, you can't open anything."* Users should never feel blocked by connectivity.

### Home Dashboard (Facility Staff)

**Key Elements:**

| Element | Description |
|---------|-------------|
| **Color-coded Stock Alerts** | Red = stockout, Amber = low stock, Green = adequate |
| **This Week's Forecast** | Editable box showing Tier 1 forecast for top 5 commodities with adjustment sliders |
| **Quick Actions** | Large buttons for 'Stock Count', 'Place Order', 'View Pending Syncs' |
| **Sync Status Bar** | Always visible: pending uploads/downloads, last sync time |

### Order Adjustment Screen

**Key Elements:**

```
┌────────────────────────────────────────────────────────────────┐
│ SYSTEM RECOMMENDATION                                          │
│ Displays Tier 1 forecast with reasoning:                       │
│   • 3-month average                                            │
│   • Seasonal adjustment                                        │
│   • Storage constraint                                         │
├────────────────────────────────────────────────────────────────┤
│ ADJUSTMENT SLIDER                                              │
│ Min/max bounds based on storage capacity                       │
├────────────────────────────────────────────────────────────────┤
│ OVERRIDE REASON DROPDOWN (Mandatory)                           │
│   • Disease Outbreak                                           │
│   • Delivery Delay                                             │
│   • Budget Change                                              │
│   • Clinical Judgment                                          │
│   • Other (free text)                                          │
├────────────────────────────────────────────────────────────────┤
│ DISTRICT INSIGHTS (Optional, collapsible)                      │
│ Tier 2 statistical forecast from last sync                     │
├────────────────────────────────────────────────────────────────┤
│ [Print Paper Order] - Generates HMIS 105 format                │
└────────────────────────────────────────────────────────────────┘
```

### Sync & Conflict Resolution Screen

**Key Elements:**
- Simple list showing items as: Pending (yellow), Successful (green), Failed (red)
- For conflicts: Clear message - *"Your local change was saved. The district suggestion was logged for review."*
- **Retry All Button** for failed syncs

### District Dashboard (Pharmacists)

**Key Elements:**
- **Cached Map View:** Facility status using pre-downloaded map tiles, color-coded by stock status
- **Trend Graphs:** Line charts for stock levels and consumption patterns
- **Override Log:** Accessible to managers to understand the 'why' behind order changes

---

## Technical Specifications

| Function | Implementation Detail |
|----------|----------------------|
| **Development Framework** | Native Android (Kotlin/Java). Minimum SDK: Android 8.0 (API level 26) |
| **Offline Storage** | SQLite in WAL mode. Schema mirrors central PostgreSQL for seamless sync |
| **Data Sync Protocol** | RESTful JSON over HTTPS (TLS 1.3). Batched, GZIP compressed. Exponential backoff |
| **Security** | AES-256 via Android Keystore. TLS 1.3 in transit. Bcrypt for passwords |
| **Battery Optimization** | Background sync only when battery >20% AND device not in active use |
| **UI Responsiveness** | Single-column layouts, 5-inch minimum (480x800px), 48dp touch targets |
| **Tested Devices** | Samsung Galaxy A series, Tecno Spark series, Infinix Hot series |
| **Offline Credential Expiry** | 90-day validity, warning at 75 days |

---

## Change Management & Capacity Building

### Training Focus
- How to use the app **when the internet is down**
- Scenario-based training on overrides (e.g., what to do during a malaria outbreak)

### Digital Literacy Support
- Embedded video guides and interactive tutorials within the app
- Text instructions use simple, jargon-free language

### Phased Rollout
- Start with facilities that conduct **weekly counts** (baseline showed they are more successful)
- Build early wins and champions

### Addressing Stakeholder Concerns

| Concern | Solution |
|---------|----------|
| **Frontline Worker Burden** | Auto-save, single-tap adjustments, paper backup |
| **System Integration** | Opportunistic sync eliminates duplicate entry; facility staff only use this app |
| **Trust Building** | Local edits always win; override logs foster learning, not punishment |

---

## Conclusion

{% hint style="success" %}
**This offline-first design is not a fallback or compromise. It is the primary architecture, deliberately chosen to match Uganda's operational reality.**
{% endhint %}

By placing core functionality on the device, encrypting data with hardware-backed security, maintaining paper-digital harmony, and trusting health workers to override system recommendations, we create a system that works **with the grain** of existing workflows rather than against them.

**The system operates at full capacity with zero connectivity, and gains additional insights when connectivity permits. This is resilience by design.**

---

## Document Information

| Field | Value |
|-------|-------|
| **Version** | 2.0 (Post-Technical Validation) |
| **Date** | November 2025 |
| **Authors** | Gideon Abako, Timothy Kavuma (IFRAD) |
| **Validator** | Ojok Ivan, Kyambogo University |
| **License** | CC BY 4.0 |
