---
title: "Sx Interface Family (Sxa / Sxb / Sxc)"
type: interface
tags: [CUPS, Sx, Sxa, Sxb, Sxc, SGW-C, SGW-U, PGW-C, PGW-U, TDF-C, TDF-U, PFCP]
sources: [ts23214]
updated: 2026-04-19
---

# Sx Interface Family (Sxa / Sxb / Sxc)

**Source:** 3GPP TS 23.214 v16.1.0; wire protocol: 3GPP TS 29.244 (PFCP)

The Sx interfaces are the control channel between the **CP function** and the corresponding **UP function** in a CUPS-split EPC node. They carry session establishment, modification, termination, and reporting messages.

---

## Interface Summary

| Interface | Between | Node context |
|---|---|---|
| **Sxa** | SGW-C ↔ SGW-U | Serving Gateway CP/UP split |
| **Sxb** | PGW-C ↔ PGW-U | PDN Gateway CP/UP split |
| **Sxc** | TDF-C ↔ TDF-U | Traffic Detection Function CP/UP split |
| **Sxa/Sxb** (combined) | Combined SGW/PGW-C ↔ Combined SGW/PGW-U | Combined node deployment |

---

## Protocol

The Sx interfaces are based on **PFCP (Packet Forwarding Control Protocol)**, defined in 3GPP TS 29.244. PFCP runs over UDP. The Sx interface can be thought of as the 3GPP-defined use of PFCP in the EPC context.

The **Sx-u tunnel** is a GTP-U tunnel used to forward user-plane packets between UP and CP when the CP function is the target (e.g. for SGW-C buffering, HTTP redirect, RADIUS/DHCP relay). Sx-u tunnel parameters are defined in TS 29.244. Tunnel granularity: per bearer of a PDN connection, per PDN, or per UP function.

---

## Session Context Types

| Sx session type | Context content |
|---|---|
| At SGW-U | Default bearer + all dedicated bearer parameters for the PDN connection |
| At PGW-U | Default bearer + all dedicated bearers + IP-CAN session parameters |
| At TDF-U (solicited) | TDF session related parameters |
| At TDF-U (unsolicited) | Application detection and reporting instructions |

---

## Message Types (Stage 2)

| Message | Direction | Purpose |
|---|---|---|
| Sx Session Establishment Request | CP → UP | Create new session context; carries PDRs, FARs, URRs, QERs |
| Sx Session Establishment Response | UP → CP | Confirms creation; includes allocated F-TEIDu(s) |
| Sx Session Modification Request | CP → UP | Update session context parameters |
| Sx Session Modification Response | UP → CP | Confirms update |
| Sx Session Termination Request | CP → UP | Remove session context; release F-TEIDu(s) |
| Sx Session Termination Response | UP → CP | Confirms removal; may include final usage reports |
| Sx Report | UP → CP | Report usage (threshold/periodic/on-demand), first DL packet, drop threshold |
| Sx Report Ack | CP → UP | Acknowledges report; may carry extended buffering parameters |

---

## Key Rules

- **All Sx session management is CP-initiated.** The UP function never initiates session establishment, modification, or termination.
- **UP reports are UP-initiated.** The UP function sends Sx reports when usage thresholds are crossed, on first DL packet arrival, or when drop thresholds are reached.
- **F-TEIDu allocation is mandatory at UP.** The UP function manages its own F-TEIDu namespace; the CP requests allocation and distributes results to peer nodes.
- **Session ID lifetime.** Assigned by CP at establishment; used by both ends to identify the session context throughout its lifetime.

---

## Parameter Types Carried on Sx

| Rule type | Abbreviation | Purpose | Defined in |
|---|---|---|---|
| Packet Detection Rule | PDR | Traffic detection criteria (F-TEIDu, UE IP, SDF filter, App ID) + references to FAR/URR/QER | TS 23.214 §7.3; TS 29.244 |
| Forwarding Action Rule | FAR | Forwarding operation and target (GTP-U encap, to CP, traffic steering) | TS 23.214 §7.4; TS 29.244 |
| Usage Reporting Rule | URR | Usage measurement criteria (thresholds, periods, triggers) and reporting granularity | TS 23.214 §7.4; TS 29.244 |
| QoS Enforcement Rule | QER | QoS enforcement parameters (MBR, GBR, APN-AMBR correlation) | TS 23.214 §7.6; TS 29.244 |

---

## Cross-References

- [CUPS Architecture](../concepts/CUPS-architecture.md) — CP/UP split model, PDR/FAR/URR functional description
- [Sx Session Management procedures](../procedures/Sx-session-management.md) — all Sx procedure flows
- TS 29.244 (PFCP) — wire-level protocol definition (not yet ingested)
