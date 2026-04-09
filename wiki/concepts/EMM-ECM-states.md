---
title: "EMM and ECM States"
type: concept
tags: [EMM, ECM, mobility-management, connection-management, MME, NAS, state-machine]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
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

## Related Pages
- [MME](../entities/MME.md) — state machine owner
- [EPS bearer model](EPS-bearer.md)
