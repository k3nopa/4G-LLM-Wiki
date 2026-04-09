---
title: "EPS Bearer"
type: concept
tags: [bearer, QoS, TFT, default-bearer, dedicated-bearer, GBR, Non-GBR, QCI]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# EPS Bearer

**Spec reference:** 3GPP TS 23.401 §4.7

## Definition

An EPS bearer is the unit of QoS treatment in EPS. It uniquely identifies a set of traffic flows that receive the same bearer-level packet forwarding treatment (scheduling policy, queue management, rate shaping, RLC configuration) between a UE and a PDN Gateway (GTP-based S5/S8) or between a UE and a Serving Gateway (PMIP-based S5/S8).

All traffic mapped to the same EPS bearer receives identical treatment. Providing different QoS to different flows requires separate EPS bearers.

## Bearer Types

| Type | Description | Lifetime |
|---|---|---|
| **Default bearer** | First bearer established when UE connects to a PDN; provides always-on IP connectivity; catch-all for unmatched traffic | Lifetime of PDN connection |
| **Dedicated bearer** | Additional bearer for same PDN connection; typically used for GBR services like VoLTE media | Established/released per service |

- One PDN connection has exactly **one default bearer** and zero or more dedicated bearers.
- A UE may have **multiple PDN connections** (e.g. IMS APN + Internet APN), each with its own default bearer.

## QoS Parameters

| Parameter | Scope | Description |
|---|---|---|
| QCI | Per bearer | QoS Class Identifier (1–9+); encodes scheduling weight, packet delay budget, packet error rate, GBR/Non-GBR type |
| ARP | Per bearer | Allocation and Retention Priority; used for admission control and preemption |
| GBR | Per GBR bearer | Guaranteed Bit Rate (UL + DL); network reserves bandwidth |
| MBR | Per GBR bearer | Maximum Bit Rate — hard cap on GBR bearer |
| APN-AMBR | Per APN | Aggregate MBR across all Non-GBR bearers for same APN (enforced at PGW) |
| UE-AMBR | Per UE | Aggregate MBR across all Non-GBR bearers for the UE (enforced at eNB) |

**GBR bearers**: QCI 1–4 (voice, video conferencing, gaming, video streaming with GBR).
**Non-GBR bearers**: QCI 5–9 (IMS signaling, video TCP, voice TCP, etc.) — no bandwidth reservation.

**VoLTE example**: QCI=1 (GBR=~24 kbps), ARP=1 (highest, preemption capability).

## Traffic Flow Template (TFT)

A TFT is the set of packet filters associated with an EPS bearer, used to map traffic to the correct bearer.

| TFT Type | Used by | Direction |
|---|---|---|
| UL TFT | UE | UE maps uplink packets to bearers |
| DL TFT | PCEF (PGW) / BBERF (SGW) | Maps downlink packets to bearers |

A TFT packet filter contains: source IP, destination IP, source port range, destination port range, protocol, Type of Service/DSCP.

**UE uplink matching algorithm:**
1. Evaluate all UL TFTs from lowest precedence index first
2. First match → transmit on that bearer
3. No match → transmit on bearer with no UL filter assigned (default bearer)
4. If all bearers have filters and no match → discard

**Key rule**: At most one EPS bearer per PDN may have no uplink packet filter. The default bearer is typically that bearer.

## Bearer Identity

Each EPS bearer has an **EPS Bearer Identity (EBI)**: integer 5–15 per PDN connection. The EBI together with the UE's identity uniquely identifies a bearer in the network.

## Bearer in the Network Path

```
UE ──[Air]── eNodeB ──[S1-U GTP-U]── SGW ──[S5 GTP-U]── PGW ──[SGi]── PDN
                                   (TEID per bearer)   (TEID per bearer)
```

Each segment uses a GTP-U tunnel identified by a TEID (Tunnel Endpoint ID). The bearer is an end-to-end logical pipe; each hop maintains its own TEID pair.

## Related Pages
- [MME](../entities/MME.md) — bearer management signaling
- [SGW](../entities/SGW.md) — user plane segment SGW↔eNB
- [PGW](../entities/PGW.md) — IP anchor, bearer binding, rate enforcement
- [PCRF](../entities/PCRF.md) — policy decisions that trigger bearer creation
