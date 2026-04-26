---
title: "EMM and ECM States"
type: concept
tags: [EMM, ECM, mobility-management, connection-management, MME, NAS, state-machine]
sources: [ts_123401v150400p.pdf, ts24301v170600p.pdf]
updated: 2026-04-19
---

# EMM and ECM States

**Spec reference:** 3GPP TS 23.401 §4.6

## Overview

Two orthogonal state machines govern a UE's relationship with the EPC:

| State Machine | What it tracks | Maintained by |
|---|---|---|
| **EMM** (EPS Mobility Management) | Whether UE is registered; location known | MME + UE |
| **ECM** (EPS Connection Management) | Whether NAS signaling connection exists | MME + UE |

They are **independent**: a UE can be EMM-REGISTERED + ECM-IDLE (most common idle state) or EMM-REGISTERED + ECM-CONNECTED (active data session).

---

## EMM States

### EMM-DEREGISTERED
- No valid location or routing information for the UE in the MME
- UE is not reachable
- Some UE context may still exist (to avoid re-running AKA on next attach)
- **Enter:** Initial state; or Detach; or Attach Reject; or TAU Reject; or all bearers deactivated (if UE doesn't support "Attach without PDN connectivity"); or implicit detach by MME

### EMM-REGISTERED
- UE has successfully attached (Attach Accept) or updated (TAU Accept)
- Location is known at TAI list granularity (set of Tracking Areas, all served by same MME)
- UE **shall** always have at least one active PDN connection (unless "Attach without PDN connectivity" supported)
- EPS security context is established

**EMM state diagram (UE and MME):**
```
                    ┌─────────────────┐
                    │ EMM-DEREGISTERED│
                    └────────┬────────┘
         Attach Accept ──────┘  ▲─── Detach / All bearers released / Attach Reject / TAU Reject
         TAU Accept (from GERAN)│
                    ┌────────▼────────┐
                    │  EMM-REGISTERED │
                    └─────────────────┘
```

---

## ECM States

### ECM-IDLE
- No NAS signaling connection between UE and network
- No UE context in E-UTRAN (no RRC connection, no S1-MME, no S1-U)
- UE performs cell selection/reselection autonomously
- **UE behavior in ECM-IDLE (if EMM-REGISTERED):**
  - Perform TAU if current TA not in registered TAI list
  - Perform periodic TAU when periodic TAU timer expires
  - Answer paging by performing Service Request
  - Send Service Request when UE has uplink data to send

### ECM-CONNECTED
- NAS signaling connection established (RRC connection + S1-MME connection)
- UE location known to eNodeB granularity in MME
- Mobility handled by handover procedure (except NB-IoT)
- S1-U user plane bearers established

**ECM state diagram (UE):**
```
ECM-IDLE ──RRC connection established──► ECM-CONNECTED
         ◄─RRC connection released──────────────────────
```

**ECM state diagram (MME):**
```
ECM-IDLE ──S1 connection established──► ECM-CONNECTED
         ◄─S1 Release Procedure─────────────────────────
```

---

## State Combinations and Meaning

| EMM | ECM | Typical Scenario |
|---|---|---|
| DEREGISTERED | IDLE | UE powered off or never attached |
| REGISTERED | IDLE | UE idle, camped on cell, no active data |
| REGISTERED | CONNECTED | UE in active data session or call |
| DEREGISTERED | CONNECTED | Transient during detach while S1 still up |

---

## Key Transitions

| Event | EMM Transition | ECM Transition |
|---|---|---|
| Attach Accept | → REGISTERED | → CONNECTED (S1 established) |
| TAU Accept | → REGISTERED (maintained) | — |
| Service Request (paging response) | — | IDLE → CONNECTED |
| S1 Release (inactivity) | — | CONNECTED → IDLE |
| Detach (UE or network) | → DEREGISTERED | → IDLE (S1 released) |
| Implicit detach timer expiry | → DEREGISTERED | — |
| Attach Reject | → DEREGISTERED | — |

---

## Timers Related to EMM/ECM

| Timer | Location | Behavior |
|---|---|---|
| Periodic TAU timer (T3412) | UE | UE performs TAU on expiry to maintain registration |
| Mobile Reachability Timer | MME | Started when UE → ECM-IDLE; value ≈ T3412; expiry → suspect unreachable |
| Implicit Detach Timer | MME | Started when Mobile Reachability Timer expires; expiry → implicit detach |
| Active Time (PSM) | UE + MME | Power Saving Mode active window; MME won't page UE outside this window |

---

## Why This Matters for VoLTE

For an IMS voice call to reach the UE:
1. UE must be **EMM-REGISTERED** (known location at TAI level)
2. Network pages UE (if ECM-IDLE) → UE performs Service Request → **ECM-CONNECTED**
3. Dedicated GBR bearer established for voice media
4. SIP signaling and RTP both flow over ECM-CONNECTED bearers

---

## EMM Stage-3 Detail (TS 24.301)

### Full UE EMM State Set (§5.1.3.2)

Beyond the two main states above, the stage-3 spec defines a rich substate structure:

**EMM-DEREGISTERED substates:**

| Substate | Meaning |
|---|---|
| NORMAL-SERVICE | Suitable cell; PLMN/TA not forbidden → initiate attach |
| LIMITED-SERVICE | Forbidden PLMN/cell; emergency attach only |
| ATTEMPTING-TO-ATTACH | Attach failed; waiting for T3411/T3402 expiry |
| PLMN-SEARCH | Scanning for PLMNs |
| NO-IMSI | No valid USIM |
| ATTACH-NEEDED | Valid USIM; attach ASAP when access class clears |
| NO-CELL-AVAILABLE | No E-UTRAN cell selectable |
| eCALL-INACTIVE | eCall-only UE; awaiting eCall trigger |

**EMM-REGISTERED substates:**

| Substate | Meaning |
|---|---|
| NORMAL-SERVICE | Normal operation; TAU when conditions met |
| ATTEMPTING-TO-UPDATE | TAU failed; no user data; retry on timer expiry |
| LIMITED-SERVICE | Cell cannot provide normal service |
| PLMN-SEARCH | PLMN selection underway |
| UPDATE-NEEDED | TAU needed but access barred |
| NO-CELL-AVAILABLE | Coverage lost or PSM active |
| ATTEMPTING-TO-UPDATE-MM | EPS-only combined attach/TAU (non-EPS part failed) |
| IMSI-DETACH-INITIATED | Detaching from non-EPS only (IMSI detach) |

**Additional transient states (UE):**
- `EMM-REGISTERED-INITIATED` — attach/combined attach in progress
- `EMM-DEREGISTERED-INITIATED` — detach in progress
- `EMM-TRACKING-AREA-UPDATING-INITIATED` — TAU in progress
- `EMM-SERVICE-REQUEST-INITIATED` — service request in progress

### MME EMM States (§5.1.3.4)

| State | Meaning |
|---|---|
| EMM-DEREGISTERED | No EMM context or context marked detached |
| EMM-COMMON-PROCEDURE-INITIATED | Common procedure active, awaiting UE response |
| EMM-REGISTERED | EMM context + default bearer established |
| EMM-DEREGISTERED-INITIATED | Network-initiated detach, awaiting DETACH ACCEPT |

### EPS Update Status (§5.1.3.3)

Stored on USIM (or NV memory), orthogonal to EMM state:

| Value | Meaning |
|---|---|
| EU1: UPDATED | Last attach/TAU succeeded |
| EU2: NOT UPDATED | Last attempt failed procedurally (no response/reject) |
| EU3: ROAMING NOT ALLOWED | Last attempt completed but rejected (roaming restriction) |

---

## Related Pages
- [MME](../entities/MME.md) — state machine owner
- [EPS bearer model](EPS-bearer.md)
- [NAS EMM Protocol](../protocols/NAS-EMM-protocol.md) — stage-3 NAS details: security, UE modes, substates
