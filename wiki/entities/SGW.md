---
title: "SGW — Serving Gateway"
type: entity
tags: [SGW, EPC, user-plane, mobility-anchor, GTP, bearer]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# SGW — Serving Gateway

**Spec reference:** 3GPP TS 23.401 §4.4.3.2
**Plane:** User plane (data forwarding); control plane signaling via GTPv2-C
**Layer:** EPC core — RAN/core boundary anchor

## Role

The SGW is the user plane gateway that terminates the S1-U interface toward E-UTRAN. It is the local mobility anchor for intra-LTE handovers — when the UE moves between eNodeBs, the SGW stays constant and only the S1-U tunnel endpoint at the eNodeB changes. There is exactly one SGW per UE at any point in time.

> The SGW and PGW may be collocated in a single physical node (Single Gateway option), with S5 internal.

## Functions (from TS 23.401 §4.4.3.2)

| Function | Description |
|---|---|
| Local mobility anchor (intra-LTE) | Anchors user plane during inter-eNB handover; path switching happens at SGW |
| "End marker" sending | Immediately sends end-marker packets to source eNB after path switch to help reordering at target |
| Inter-3GPP mobility anchor | Terminates S4; relays traffic between 2G/3G SGSN and PGW during inter-RAT HO |
| ECM-IDLE DL buffering | Buffers downlink packets when UE is in ECM-IDLE; triggers DDN (Downlink Data Notification) to MME to initiate paging |
| Packet routing and forwarding | Routes GTP-U tunneled packets between eNodeB and PGW |
| Lawful Interception | LI of user plane traffic |
| DSCP/QCI marking | Sets DiffServ Code Point on uplink and downlink packets based on QCI and ARP |
| Per-UE/bearer accounting | Generates CDRs for inter-operator charging (GTP-based S5/S8) |
| OFCS interface | Interfaces Offline Charging System per TS 32.240 |
| Paging Policy Differentiation | Optional; differentiates paging based on traffic type |

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| S1-U | eNodeB | GTP-U | User plane bearer tunnels to/from RAN |
| S5 | PGW | GTPv2-C + GTP-U | Bearer management + user plane (intra-PLMN) |
| S8 | PGW (HPLMN) | GTP | Inter-PLMN variant of S5 (roaming, home-routed) |
| S11 | MME | GTPv2-C | Session/bearer control from MME |
| S4 | SGSN | GTP | 2G/3G inter-RAT mobility anchor |
| S12 | UTRAN (RNC) | GTP-U | Direct Tunnel (bypasses SGW user plane, operator option) |

## Mobility Anchor Behavior

**Intra-eNB handover:** No SGW involvement (X2 handover).

**Inter-eNB handover (X2):** Target eNB sends Path Switch Request to MME → MME sends Modify Bearer Request to SGW → SGW switches S1-U endpoint from source to target eNB → SGW sends end-marker to source eNB.

**Inter-eNB handover (S1):** Source MME/SGW involved; full bearer re-establishment or context transfer.

**Inter-SGW handover:** SGW change triggers Create Session (new SGW) and Delete Session (old SGW) via MME.

## ECM-IDLE Buffering

When UE enters ECM-IDLE:
1. SGW receives downlink packet for UE
2. SGW has no active S1-U tunnel → buffers packet
3. SGW sends **Downlink Data Notification (DDN)** to MME
4. MME initiates paging procedure
5. When UE responds → Service Request → ECM-CONNECTED → SGW delivers buffered packets

## Key Context Stored per Bearer
- EPS Bearer Identity (EBI)
- UE IP address (or TEID on PGW side)
- eNodeB TEID (uplink) + SGW TEID (downlink)
- QCI, ARP, GBR, MBR
- Charging ID

## Related Pages
- [MME](MME.md) — control plane master via S11
- [PGW](PGW.md) — user plane peer via S5/S8
- [EPS bearer model](../concepts/EPS-bearer.md)
- [Reference points](../interfaces/reference-points.md)
