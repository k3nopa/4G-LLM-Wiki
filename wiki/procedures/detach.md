---
title: "EPS Detach Procedures"
type: procedure
tags: [detach, MME, SGW, PGW, HSS, SGSN, ISR, bearer-teardown, NAS]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-09
---

# EPS Detach Procedures

Detach removes a UE from EPS service: all EPS bearers are released, MM context deleted, and S1 signalling connection torn down. TS 23.401 §5.3.8 defines four initiator variants.

---

## Overview of Variants

| Clause | Initiator | Trigger |
|---|---|---|
| §5.3.8.2.1 | UE | UE powers off or explicitly detaches from E-UTRAN |
| §5.3.8.2.2 | UE (via SGSN) | UE detaches from GERAN/UTRAN with ISR active |
| §5.3.8.3 | MME | O&M, inactivity timeout, SIPTO GW relocation |
| §5.3.8.3A | SGSN | SGSN detach with ISR active |
| §5.3.8.4 | HSS | Subscription withdrawal, operator command |

---

## UE-Initiated Detach — E-UTRAN (§5.3.8.2.1)

```mermaid
sequenceDiagram
    participant UE
    participant eNB as eNodeB
    participant MME
    participant SGSN
    participant SGW as Serving GW
    participant PGW as PDN GW
    participant PCRF

    UE->>eNB: 1. NAS Detach Request (GUTI, Switch Off)
    eNB->>MME: 1. Initial UE Message / Uplink NAS Transport

    MME->>SGW: 2. Delete Session Request (LBI, ULI, SecRAT usage) per PDN
    Note over SGW: ISR active? Deactivate ISR, return Cause.<br/>ISR inactive? Forward to PGW (step 6).

    SGW-->>MME: 3. Delete Session Response [ISR active path]

    MME-->>SGSN: 4. Detach Indication (Cause = complete detach) [if ISR]
    SGSN->>SGW: 5. Delete Session Request (LBI, CGI/SAI) per PDN [if ISR]

    SGW->>PGW: 6. Delete Session Request (LBI, ULI, SecRAT usage)
    PGW-->>SGW: 7. Delete Session Response (Cause, APN Rate Control Status)
    PGW-)PCRF: 8. PCEF IP-CAN Session Termination (TS 23.203)

    SGW-->>MME: 9. Delete Session Response
    SGSN-->>MME: 10. Detach Acknowledge [if ISR]

    MME-->>UE: 11. Detach Accept [if Switch Off = false]
    MME->>eNB: 12. S1 Release Command (Cause = Detach)
```

### Step-by-step notes

| Step | Action | Key detail |
|---|---|---|
| 1 | UE→MME: NAS Detach Request | Switch Off = true → power-off (no step 11). If UE was ECM-IDLE, S1 connection is established to deliver the message |
| 2 | MME→SGW: Delete Session Request | One message per PDN connection. Carries LBI identifying the default bearer. UE Time Zone IE included if TZ changed. If ISR activated, MME waits to receive step 5 Delete Session Request before releasing CP-TEID |
| 3 | SGW ISR active path | S-GW deactivates ISR, releases EPS Bearer contexts, returns Delete Session Response with Cause. Does **not** forward to PGW yet |
| 4–5 | MME notifies SGSN | Detach Indication triggers SGSN to send Delete Session Request to SGW per PDN |
| 6 | SGW→PGW: Delete Session Request | Carries ULI, UE TZ, Secondary RAT usage (taking least-age data from MME and SGSN). Indicates all bearers of the PDN connection shall be released |
| 8 | PGW→PCRF: IP-CAN Session Termination | PCEF-initiated; not executed if no PCRF deployed |
| 11 | Detach Accept | Omitted when Switch Off = true (device is powering off) |
| 12 | S1 Release | Follows §5.3.5; Cause = Detach |

> **Note (PMIP S5/S8):** Steps 3, 4, 5 (GTP-based) are replaced by TS 23.402 procedures for PMIP-based S5/S8.

---

## MME-Initiated Detach (§5.3.8.3)

```mermaid
sequenceDiagram
    participant UE
    participant eNB as eNodeB
    participant MME
    participant SGSN
    participant SGW as Serving GW
    participant PGW as PDN GW
    participant PCRF

    Note over MME: Trigger: O&M, inactivity, SIPTO relocation
    MME-->>UE: 1. Detach Request (Detach Type) [explicit only]
    Note over MME: Implicit detach: no NAS message to UE.<br/>UE in ECM-IDLE → MME pages UE first.

    MME->>SGW: 2. Delete Session Request per PDN
    SGW-->>MME: 3. Delete Session Response [ISR path]
    MME-->>SGSN: 4. Detach Notification
    SGSN->>SGW: 5. Delete Session Request [ISR + complete detach]
    SGW->>PGW: 6. Delete Session Request
    PGW-->>SGW: 7. Delete Session Response
    PGW-)PCRF: 8. PCEF IP-CAN Session Termination
    SGW-->>MME: 9. Delete Session Response
    SGSN-->>MME: 10. Detach Acknowledge
    UE-->>MME: 11. Detach Accept (via eNB)
    MME->>eNB: 12. S1 Release Command (Cause = Detach)
```

**Implicit vs explicit:**
- **Implicit**: MME removes the UE context locally (no Detach Request to UE). An SGSN registration is preserved.
- **Explicit**: MME sends Detach Request with optional `Detach Type = re-attach required` (SIPTO GW relocation). UE re-attaches after RRC release.

**SIPTO GW relocation**: MME sets Detach Type = "explicit detach with reattach required" when GW relocation is desirable for all SIPTO APNs. UE should re-establish those PDN connections immediately after reattach.

---

## HSS-Initiated Detach (§5.3.8.4)

```mermaid
sequenceDiagram
    participant UE
    participant eNB as eNodeB
    participant RNC
    participant MME
    participant SGSN
    participant SGW as Serving GW
    participant PGW as PDN GW
    participant PCRF
    participant HSS

    HSS->>MME: 1a. Cancel Location (IMSI, Cancellation Type = Subscription Withdrawn)
    MME-->>HSS: 1a. Cancel Location ACK
    HSS->>SGSN: 1b. Cancel Location (if SGSN also registered)
    SGSN-->>HSS: 1b. Cancel Location ACK

    MME-->>UE: 2a. Detach Request (Detach Type) [if ECM-CONNECTED]
    SGSN-->>UE: 2b. Detach Request [if registered]

    MME->>SGW: 3a. Delete Session Request per PDN
    SGSN->>SGW: 3b. Delete Session Request per PDN

    SGW->>PGW: 4. Delete Session Request
    PGW-->>SGW: 5. Delete Session Response
    PGW-)PCRF: 6. PCEF IP-CAN Session Termination

    SGW-->>MME: 7a. Delete Session Response
    SGW-->>SGSN: 7b. Delete Session Response

    UE-->>MME: 8a. Detach Accept
    UE-->>SGSN: 8b. Detach Accept

    MME->>eNB: 10a. S1 Release Command
    SGSN->>RNC: 10b. PS Signalling Connection Release
```

**Triggers**: Operator command, subscription change (for subscription-change RAT restrictions use Insert Subscriber Data instead), subscription withdrawal.

**Emergency bearers**: MME/SGSN shall NOT initiate detach for emergency-attached UEs; only non-emergency PDN connections are deactivated.

---

## ISR Behaviour During Detach

| Scenario | SGW action |
|---|---|
| ISR active, receives **first** Delete Session Request (from MME or SGSN) | Deactivate ISR; release related EPS Bearer contexts; respond with Delete Session Response. Do **not** release CP-TEID until both nodes have sent their Delete Session Requests |
| ISR active, receives **second** Delete Session Request | Release remaining contexts; proceed with Delete Session Request to PGW |
| ISR **not** active | On receiving Delete Session Request, immediately send Delete Session Request to PGW |

---

## "Switch Off" Parameter

| Value | Meaning | Step 11 (Detach Accept) |
|---|---|---|
| true | UE is powering off | Omitted — UE cannot receive it |
| false | Explicit detach (e.g., airplane mode) | Sent to UE |

---

## Post-Detach State

- **MM context**: deleted from MME (may be retained briefly for optimisation)
- **EPS Bearer contexts**: all released in SGW and PGW
- **PDN address**: PGW may retain for implementation-defined period
- **PCRF**: IP-CAN session terminated via PCEF-initiated procedure (TS 23.203)
- **EMM state** → EMM-DEREGISTERED; **ECM state** → ECM-IDLE

---

## Related Pages

- [MME](../entities/MME.md) — initiates or processes all detach variants
- [SGW](../entities/SGW.md) — releases bearer contexts, ISR deactivation
- [PGW](../entities/PGW.md) — IP-CAN session termination with PCRF
- [EPS Attach](EPS-attach.md) — inverse procedure
- [Service Request](service-request.md) — ECM-IDLE paging used during MME-initiated explicit detach
- [EMM/ECM States](../concepts/EMM-ECM-states.md) — state transitions on detach
