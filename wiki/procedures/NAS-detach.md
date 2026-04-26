---
title: "NAS Detach Procedure (Stage 3)"
type: procedure
tags: [NAS, EMM, detach, EPS, MME, UE, T3421, T3422, stage-3]
sources: [ts24301v170600p.pdf]
updated: 2026-04-19
---

# NAS Detach Procedure (Stage 3)

**Spec reference:** 3GPP TS 24.301 §5.5.2  
**Stage-2 counterpart:** [EPS Detach](detach.md) (TS 23.401 §5.3.8)

---

## 1. Overview (§5.5.2.1)

The detach procedure is used to:
- UE detach from EPS services only
- UE detach from both EPS and non-EPS services (combined EPS/IMSI detach)
- UE detach from non-EPS services only (IMSI detach; UE stays EPS attached)
- Network: inform UE it is detached for EPS services and/or non-EPS services
- Network: force re-attach (with or without re-attach required indication)
- Network: disconnect UE from last remaining PDN when EMM-REGISTERED without PDN connection not supported

EPS bearer contexts are deactivated **locally** (no peer-to-peer ESM signalling between UE and MME), unless a separate EPS bearer context deactivation procedure is initiated.

The UE is allowed to initiate detach even if T3346 is running.

---

## 2. Detach Types

| Type | Direction | Detach Type IE |
|---|---|---|
| EPS detach | UE→network | EPS detach (switch-off or not) |
| Combined EPS/IMSI detach | UE→network | Combined EPS/IMSI detach |
| IMSI detach | UE→network | IMSI detach |
| Re-attach required | Network→UE | Re-attach required |
| Re-attach not required | Network→UE | Re-attach not required |
| IMSI detach | Network→UE | IMSI detach |

---

## 3. UE-Initiated Detach Procedure (§5.5.2.2)

```mermaid
sequenceDiagram
    participant UE
    participant MME

    alt Non-switch-off
        UE->>MME: DETACH REQUEST [start T3421]
        Note over UE: → EMM-DEREGISTERED-INITIATED
        MME->>UE: DETACH ACCEPT
        Note over UE: stop T3421 → EMM-DEREGISTERED (or EMM-REGISTERED if IMSI only)
    else Switch-off
        UE->>MME: DETACH REQUEST
        Note over UE: procedure complete after sending; may power off
    end
```

### 3.1 Initiation (§5.5.2.2.1)

- UE sends DETACH REQUEST → start T3421 → EMM-DEREGISTERED-INITIATED (for EPS detach) or EMM-REGISTERED.IMSI-DETACH-INITIATED (for IMSI-only detach)
- **Detach type IE:** set "switch off" bit if USIM removed / UE switching off / eCall inactivity; set EPS detach / combined EPS+IMSI / IMSI-only type
- **Security context flag:** "mapped security context" if current context is mapped; otherwise "native security context"
- **Identity:** GUTI (if valid) else IMSI else IMEI
- If switch-off: UE may power off as soon as DETACH REQUEST is sent; try for at least 5s (standard), 85s (NB-S1), 14s (WB-S1 CE mode)
- After last DETACH REQUEST sent (switch-off): store current native EPS security context as valid (or store non-current full native context and delete mapped/partial native)

### 3.2 Completion for EPS Services Only (§5.5.2.2.2)

- If detach type does not indicate "switch off": network sends DETACH ACCEPT; bearers deactivated locally; network → EMM-DEREGISTERED
- UE on receiving DETACH ACCEPT: stop T3421
  - If EPS service disabled: → EMM-NULL state
  - Otherwise: → EMM-DEREGISTERED state

### 3.3 Completion for Combined Detach (§5.5.2.2.3)

- Network sends DETACH ACCEPT (if not switch-off)
- Combined EPS/IMSI detach: UE deactivates bearers locally; → EMM-DEREGISTERED + MM-NULL
- IMSI detach only: UE non-EPS inactive; → MM-NULL + EMM-REGISTERED

### 3.4 Abnormal Cases (§5.5.2.2.4)

| Case | UE Action |
|---|---|
| (a) Access barred (EAB/ACB) | WB-S1: postpone until access granted; NB-S1: implementation-specific; may perform local detach |
| (b) Lower layer failure / NAS connection release before DETACH ACCEPT | Depends on detach type: EPS disable → EMM-NULL; EPS detach → EMM-DEREGISTERED; IMSI detach → EMM-REGISTERED.NORMAL-SERVICE + MM-NULL; combined → EMM-DEREGISTERED + MM-NULL |
| (c) T3421 timeout | Retransmit DETACH REQUEST, restart T3421 (repeat max 4 times); on 5th expiry: abort, same state transitions as (b) |
| (d) DETACH REQUEST collision (switch-off) | Ignore network message; continue UE detach |
| (d) DETACH REQUEST collision (non-switch-off) | Process network message per §5.5.2.3.2 with collision modifications |
| (e) Detach and EMM common procedure collision | "switch off": ignore common procedure message; non-switch-off + IMSI detach: both proceed; non-switch-off + EPS detach: ignore common procedure |
| (f) Cell change to new TA (non-switch-off, non-switch-off) | Abort, re-initiate after TAU (if not switch-off + not removing USIM) |
| (i) CS SERVICE NOTIFICATION collision | Ignore; continue combined detach |

**T3421 retry:** First 4 expiries → retransmit + restart T3421. Fifth expiry → abort, state transition per (b).

---

## 4. Network-Initiated Detach Procedure (§5.5.2.3)

```mermaid
sequenceDiagram
    participant UE
    participant MME

    MME->>UE: DETACH REQUEST [start T3422]
    Note over MME: detach type: re-attach required / not required / IMSI detach

    UE->>MME: DETACH ACCEPT
    Note over MME: stop T3422

    alt Re-attach required
        Note over UE: deactivate bearers → EMM-DEREGISTERED → initiate attach
    else Re-attach not required
        Note over UE: deactivate bearers → EMM-DEREGISTERED
    else IMSI detach
        Note over UE: EPS bearers remain → MM U2 NOT UPDATED; may re-attach non-EPS via TAU
    end
```

### 4.1 Initiation (§5.5.2.3.1)

- MME sends DETACH REQUEST → start T3422 (if type ≠ "IMSI detach")
- May include EMM cause IE
- If MME performs local detach (no UE reachable): inform UE via next EMM message with cause #10 "implicitly detached"
- In NB-S1/WB-S1 satellite access: if UE at location not permitted → EMM cause #78

### 4.2 Completion by the UE (§5.5.2.3.2)

**"Re-attach required":**
- UE: deactivate all EPS bearer contexts locally, stop T3346/T3396 if running, send DETACH ACCEPT
- → EMM-DEREGISTERED; initiate attach (or combined attach) to re-establish PDN connections
- Enable N1 mode if previously disabled (per NAS config MO "Re_enable_N1_upon_reattach")

**"Re-attach not required" (no EMM cause or unrelated cause):**
- UE: deactivate all EPS bearer contexts, send DETACH ACCEPT → EMM-DEREGISTERED
- No automatic re-attach

**"Re-attach not required" + EMM cause #2 (IMSI unknown in HSS):**
- USIM invalid for non-EPS services; UE still EPS-attached

**"IMSI detach":**
- UE does NOT deactivate EPS bearer contexts; MM update = U2 NOT UPDATED
- May send DETACH ACCEPT; may re-attach to non-EPS via combined TAU (TRACKING AREA UPDATE REQUEST with "combined TA/LA updating with IMSI attach")

### 4.3 EMM Cause Handling on Network-Initiated Detach

| Cause | UE Action |
|---|---|
| #2 IMSI unknown in HSS | USIM invalid for non-EPS services; UE stays EPS-attached |
| #3 Illegal UE / #6 Illegal ME | EU3, delete GUTI/TAI/eKSI → EMM-DEREGISTERED.NO-IMSI |
| #7 EPS services not allowed | EU3 → EMM-DEREGISTERED |
| #8 EPS+non-EPS not allowed | EU3 → EMM-DEREGISTERED.NO-IMSI |
| #11 PLMN not allowed | EU3, add to forbidden PLMN list → EMM-DEREGISTERED.PLMN-SEARCH |
| #12 Tracking area not allowed | EU3, add TA to "forbidden for regional provision" → LIMITED-SERVICE |
| #13 Roaming not allowed in TA | EU3, add TA to "forbidden for roaming" → PLMN-SEARCH |
| #14 EPS services not allowed in PLMN | EU3, add to "forbidden PLMNs for GPRS service" → PLMN-SEARCH |
| #15 No suitable cells in TA | EU3, add TA to "forbidden TAs for roaming" → LIMITED-SERVICE |
| #25 Not authorized for CSG | EU3, remove CSG from Allowed CSG list → LIMITED-SERVICE |
| #78 PLMN not allowed at UE location | EU3 → PLMN-SEARCH |

### 4.4 Network Abnormal Cases (§5.5.2.3.5)

| Case | Network Action |
|---|---|
| T3422 timeout (first 4 times) | Retransmit DETACH REQUEST, restart T3422 |
| T3422 5th expiry | Abort; if type = "IMSI detach" or "re-attach not required" + cause #2 → don't change EMM state; otherwise → EMM-DEREGISTERED |
| Lower layer failure | Same logic as T3422 5th expiry |
| DETACH REQUEST (switch-off) collision | Consider both detach procedures complete |
| DETACH REQUEST (non-switch-off) collision | Send DETACH ACCEPT |
| TAU REQUEST received before detach complete | Progress both or abort detach (depends on cause values) |
| SERVICE REQUEST / EXTENDED SERVICE REQUEST collision | Progress detach procedure; or progress both if CS fallback emergency |

---

## 5. Timers

| Timer | Started by | Stopped by | Purpose |
|---|---|---|---|
| T3421 | UE, on DETACH REQUEST | DETACH ACCEPT | Supervise UE-initiated detach |
| T3422 | MME, on DETACH REQUEST | DETACH ACCEPT | Supervise network-initiated detach |

---

## Related Pages

- [EPS Detach (stage-2)](detach.md) — TS 23.401 §5.3.8 flow
- [NAS Attach](NAS-attach.md) — Re-attach after detach
- [NAS TAU](NAS-TAU.md) — Used for IMSI re-attach after IMSI detach
- [NAS EMM Protocol](../protocols/NAS-EMM-protocol.md) — EMM state machine
- [EMM/ECM States](../concepts/EMM-ECM-states.md)
- [MME](../entities/MME.md)
