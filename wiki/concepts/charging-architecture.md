---
title: "3GPP Charging Architecture — Offline and Online"
type: concept
tags: [charging, offline-charging, online-charging, Rf, Ro, CTF, CDF, OCS, IEC, ECUR, SCUR, Diameter]
sources: [ts_132299v160200p.pdf]
updated: 2026-04-12
---

# 3GPP Charging Architecture — Offline and Online

Source: 3GPP TS 32.299 v16.2.0 (Release 16), §4–§5

---

## 1. Architecture Overview

3GPP charging has two orthogonal paths: **offline** (post-payment, record-keeping) and **online** (real-time credit control). Both paths originate at the same logical function in each network element.

### Logical Functions

| Function | Abbreviation | Role |
|---|---|---|
| Charging Trigger Function | CTF | Resides in each NE; detects chargeable events; sends charging data |
| Charging Data Function | CDF | Collects offline charging data from CTFs; constructs CDRs |
| Online Charging Function | OCF | Real-time credit/unit management for online charging |
| Charging Gateway Function | CGF | Aggregates CDRs from CDF/OCF; passes to Billing Domain |
| Billing Domain | BD | Post-processing, billing mediation |

### Reference Points

| Point | From → To | Purpose |
|---|---|---|
| **Rf** | CTF → CDF | Offline charging (Diameter Accounting) |
| **Ro** | CTF → OCF | Online charging (Diameter Credit-Control) |
| **Gy** | PCEF → OCS | Online charging for PS domain (Ro-based, roaming context) |
| **Gyn** | TDF → OCS | Online charging for TDF (Ro-based) |
| **Ga** | CDF/OCF → CGF | CDR transfer |
| **Bx** / **Bo** | CGF → Billing Domain | CDR delivery to billing system |

```mermaid
graph LR
    subgraph NE ["Network Element"]
        CTF["CTF\n(Charging Trigger Function)"]
    end
    subgraph Offline ["Offline Path"]
        CDF["CDF\n(Charging Data Function)"]
    end
    subgraph Online ["Online Path"]
        OCF["OCF\n(Online Charging Function)"]
    end
    CGF["CGF\n(Charging Gateway Function)"]
    BD["Billing Domain"]

    CTF -->|"Rf (ACR/ACA)"| CDF
    CTF -->|"Ro (CCR/CCA)"| OCF
    CDF -->|"Ga"| CGF
    OCF -->|"Ga"| CGF
    CGF -->|"Bx/Bo"| BD
```

### Key Architectural Principles (§4.1.1)

- Each CTF has a **priority-ordered CDF/OCF address list**; if the primary charging function is unavailable, the CTF falls over to the secondary, etc.
- Within a Release, each NE sends charging information only to charging entities in the **same PLMN** — cross-PLMN charging is not done directly.
- **Single IMSI architecture** (EU roaming unbundling Regulation III): a specific Service-NE (Proxy Function) uses Ro to forward charging to an OCF in another network; details in TS 32.240 Annex B.
- For PS domain online charging in roaming: PCEF in VPLMN uses **Gy**, TDF uses **Gyn** (both Ro-based), sending to an OCS in the HPLMN (TS 32.251).
- Each CDF knows other CDFs' network addresses (configurable list) for Redirect-Request redundancy.

---

## 2. Offline Charging (§5.1)

### Scenarios

Two basic scenarios:

| Type | Trigger | ACR types used |
|---|---|---|
| **Event-based** | Single discrete event (e.g. SMS, MMS delivery) | ACR[Event] only |
| **Session-based** | Ongoing session (e.g. data bearer, IMS call) | ACR[Start] + ACR[Interim]* + ACR[Stop] |

*Interim triggered by timer expiry or significant session changes.

### Event-Based Charging Flow

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant NE as Network Element
    participant CTF as CTF
    participant CDF as CDF

    UE->>NE: 1. Request for resource usage
    NE-->>UE: 2. Content/Service Delivery
    Note over CTF: 3. Charging Data Generation
    CTF->>CDF: 4. ACR[Event] — Event Related Charging Data
    Note over CDF: 5. Process Request (CDR generation config-dependent)
    CDF-->>CTF: 6. ACA — Charging Data Response
```

### Session-Based Charging Flow

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant CDF as CDF

    UE->>CTF: 1. Request for resource usage
    CTF-->>UE: 2. Session established
    Note over CTF: 3. Charging Data Generation (session start)
    CTF->>CDF: 4. ACR[Start]
    CDF-->>CTF: 5. ACA
    Note over CTF: 7. Charging Data Generation (interim timer/event)
    CTF->>CDF: 8. ACR[Interim]
    CDF-->>CTF: 10. ACA
    UE->>CTF: 11. Session released
    Note over CTF: 12. Charging Data Generation (session stop)
    CTF->>CDF: 13. ACR[Stop]
    CDF-->>CTF: 15. ACA
```

### Charging Data Request / Response (Logical Fields)

**Charging Data Request** (CTF → CDF):

| Field | Category | Description |
|---|---|---|
| Session Identifier | M | Identifies the operation session |
| Originator Host | M | Source NE identity and realm |
| Originator Domain | M | Realm of operation originator |
| Destination Domain | M | Realm of operation destination |
| Operation Type | M | Event / Start / Interim / Stop |
| Operation Number | M | Sequence number |
| Operation Identifier | Om | Unique operation ID |
| User Name | Oc | Service user identity |
| Service information | Om | Service-specific parameters (per middle-tier TS) |

**Charging Data Response** (CDF → CTF):

| Field | Category | Description |
|---|---|---|
| Session Identifier | M | Identifies the operation session |
| Operation Result | M | Result of the operation |
| Operation Type | M | Echoes the transfer type |
| Operation Number | M | Sequence number |
| Error Reporting Host | Oc | Identity of proxy that sent non-2001 result |

---

## 3. Online Charging (§5.2)

### Core Concepts

Online charging operates via the **Ro** reference point between CTF and OCF. Two sub-functions:

| Sub-function | Description | Where performed |
|---|---|---|
| **Unit Determination** | Calculates non-monetary units (time, volume, events) needed | CTF (decentralized) or OCF (centralized) |
| **Rating** | Converts non-monetary units to monetary cost | CTF (decentralized) or OCF (centralized) |

> **Note:** Centralized Unit Determination + Decentralized Rating is **not possible** (the combination is undefined).

### Three Online Charging Cases

| Case | Abbreviation | Reservation? | Debit timing | RFC 4006 mechanism |
|---|---|---|---|---|
| Immediate Event Charging | **IEC** | No | Immediate (before/during/after service) | Direct Debit One-Time Event (§6.3.3) |
| Event Charging with Unit Reservation | **ECUR** | Yes | After service delivery | Reserve + Debit Units |
| Session Charging with Unit Reservation | **SCUR** | Yes | After session (ongoing reservation) | Session-Based Credit-Control (§6.3.5) |

SCUR and ECUR use both **Debit Units** and **Reserve Units** operations; when both are needed in SCUR they are **combined in one message** (CCR with both used-unit and requested-unit containers).

For SCUR/ECUR: reserved units ≠ consumed units — CTF can modify the reservation during the session, including returning unused reserved units.

### IEC — Immediate Event Charging

Three variants depending on where rating/unit-determination resides:

#### IEC-a: Decentralized UD + Centralized Rating

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant OCF as OCF

    UE->>CTF: 1. Request for resource usage
    Note over CTF: 2. Units Determination (CTF calculates units)
    CTF->>OCF: 3. Debit Units Request (Non-monetary Units)
    Note over OCF: 4. Rating Control (OCF → monetary)
    Note over OCF: 5. Account Control (deduct from balance)
    OCF-->>CTF: 6. Debit Units Response (Non-monetary Units)
    CTF-->>UE: 7. Content/Service Delivery
```

#### IEC-b: Centralized UD + Centralized Rating

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant OCF as OCF

    UE->>CTF: 1. Request for resource usage
    CTF->>OCF: 2. Debit Units Request (Service Key)
    Note over OCF: 3. Units Determination
    Note over OCF: 4. Rating Control
    Note over OCF: 5. Account Control
    OCF-->>CTF: 6. Debit Units Response (Non-monetary Units)
    CTF-->>UE: 7. Content/Service Delivery
```

#### IEC-c: Decentralized UD + Decentralized Rating

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant OCF as OCF

    UE->>CTF: 1. Request for resource usage
    Note over CTF: 2. Units Determination
    Note over CTF: 3. Rating Control (CTF → monetary)
    CTF->>OCF: 4. Debit Units Request (Monetary Units)
    Note over OCF: 5. Account Control
    OCF-->>CTF: 6. Debit Units Response (Monetary Units)
    CTF-->>UE: 7. Content/Service Delivery
```

### ECUR — Event Charging with Unit Reservation

Reserve before service; debit after. Shown for Decentralized UD + Centralized Rating:

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant OCF as OCF

    UE->>CTF: 1. Request for resource usage
    Note over CTF: 2. Units Determination
    CTF->>OCF: 3. Reserve Units Request (Non-monetary Units)
    Note over OCF: 4. Rating Control
    Note over OCF: 5. Account Control (check balance)
    Note over OCF: 6. Reservation Control (hold units)
    OCF-->>CTF: 7. Reserve Units Response (Non-monetary Units)
    Note over CTF: 8. Reserved Units Supervision (monitor consumption)
    CTF-->>UE: 9. Content/Service Delivery
    CTF->>OCF: 10. Debit Units Request (Non-monetary Units consumed)
    Note over OCF: 11. Rating Control
    Note over OCF: 12. Account Control (deduct)
    OCF-->>CTF: 13. Debit Units Response
    UE-->>CTF: 14. Session released
```

### SCUR — Session Charging with Unit Reservation

Like ECUR but operates over a **session** with reservation maintained during session; multiple debit+reserve cycles possible while session is ongoing.

```mermaid
sequenceDiagram
    participant UE as UE-A
    participant CTF as CTF
    participant OCF as OCF

    UE->>CTF: 1. Request for session
    Note over CTF: 2. Units Determination
    CTF->>OCF: 3. Reserve Units Request
    Note over OCF: 4-6. Rating + Account + Reservation
    OCF-->>CTF: 7. Reserve Units Response
    Note over CTF: 8. Reserved Units Supervision
    Note over CTF: 9. Session ongoing (multiple reserve/debit cycles possible)
    UE->>CTF: 10. Session released
    CTF->>OCF: 11. Debit Units Request
    Note over OCF: 12-13. Rating + Account Control
    OCF-->>CTF: 14. Debit Units Response
```

### Debit / Reserve Units Message Content (§5.2.3)

**Debit/Reserve Units Request** (CTF → OCF):

| Field | Category | Description |
|---|---|---|
| Session Identifier | M | Operation session ID |
| Originator Host/Domain | M | Source NE identity |
| Destination Domain | M | OCF realm |
| Operation Identifier | M | Unique operation ID |
| Operation Token | M | Service identifier |
| Operation Type | M | Event / Start / Interim / Stop |
| Operation Number | M | Sequence number |
| Subscriber Identifier | Om | MSISDN or fixed device identity |
| Multiple Unit Operation | Oc | Quota management parameters |
| Service Information | Om | Per-service parameters (middle-tier TS) |

**Debit/Reserve Units Response** (OCF → CTF):

| Field | Category | Description |
|---|---|---|
| Session Identifier | M | Operation session ID |
| Operation Result | M | Success / error code |
| Operation Type | M | Echoes request type |
| Multiple Unit Operation | Oc | Granted quota and quota management parameters |
| Low Balance Indication | Oc | Account balance fell below threshold |
| Remaining Balance | Oc | Current subscriber account balance |
| Operation Failure Action | Oc | What CTF should do if CCR sending is prevented (SCUR) |
| Operation Event Failure Action | Oc | What CTF should do if CCR sending is prevented (IEC) |

---

## 4. Other Online Charging Requirements (§5.3)

| Requirement | Description |
|---|---|
| **Re-authorization** | OCF sets idle timeout with quota; expiry or mid-session events trigger re-auth; CTF reports quota usage with reason code |
| **Threshold-based re-auth** | OCF can include remaining-quota threshold at which CTF shall trigger re-auth |
| **Termination action** | OCF specifies CTF behavior when last granted units are consumed (e.g. terminate, redirect) |
| **Account expiration** | OCF can provide account expiration date/time to CTF |

---

## 5. Offline vs. Online Comparison

| Dimension | Offline | Online |
|---|---|---|
| Interface | Rf (Diameter Accounting) | Ro (Diameter Credit-Control) |
| Protocol base | RFC 6733 Accounting | RFC 4006 Credit-Control |
| Timing | Post-event/post-session | Real-time (before/during service) |
| Server-side function | CDF | OCF |
| Messages | ACR/ACA | CCR/CCA |
| Service allowed? | Yes, unconditionally | Conditional on credit availability |
| Credit reservation | No | Yes (ECUR/SCUR) |
| Use case | Data bearer volume, IMS CDRs | Prepaid, real-time credit depletion |

---

## Related Pages

- [interfaces/reference-points.md](../interfaces/reference-points.md) — Rf, Ro reference point details
- [entities/PGW.md](../entities/PGW.md) — PCEF/CTF in PGW; Gy interface
- [entities/PCRF.md](../entities/PCRF.md) — Gx/Rx policy (interacts with charging on Gy)
- [entities/P-CSCF.md](../entities/P-CSCF.md) — IMS CTF; generates Rf charging records for IMS sessions
- [entities/S-CSCF.md](../entities/S-CSCF.md) — IMS CTF; charging at S-CSCF
