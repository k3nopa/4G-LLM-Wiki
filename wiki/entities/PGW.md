---
title: "PGW — PDN Gateway"
type: entity
tags: [PGW, EPC, user-plane, SGi, IP-allocation, PCEF, charging, DPI]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# PGW — PDN Gateway

**Spec reference:** 3GPP TS 23.401 §4.4.3.3
**Plane:** User plane (data path) + control plane (GTPv2-C, Diameter Gx)
**Layer:** EPC edge — boundary between EPC and external PDNs

## Role

The PGW is the anchor point between the EPS and external packet data networks (Internet, IMS, enterprise PDNs). It is the UE's IP address anchor — the UE's IP address is allocated by (or delegated via) the PGW and persists as long as the PDN connection exists. The PGW also hosts the **PCEF (Policy and Charging Enforcement Function)** — it is where PCRF-derived policy rules are enforced.

> A UE accessing multiple APNs may have multiple PGWs. The SGW and PGW may be collocated (Single Gateway option).

## Functions (from TS 23.401 §4.4.3.3)

| Function | Description |
|---|---|
| SGi termination | Terminates the interface to external PDNs |
| UE IP address allocation | Allocates IPv4/IPv6/dual-stack addresses; also supports DHCPv4 (server+client) and DHCPv6 |
| Per-user packet filtering | Deep Packet Inspection (DPI) for service data flow detection |
| Lawful Interception | LI of user plane traffic |
| UL/DL DSCP marking | Sets DiffServ Code Point based on QCI and ARP |
| Per-bearer charging | Collects uplink/downlink data volume per EPS bearer; reports to OFCS |
| UL/DL gating | Gate open/close per SDF as instructed by PCRF via Gx |
| UL/DL rate enforcement (SDF) | Rate policing/shaping per Service Data Flow per TS 23.203 |
| APN-AMBR enforcement | Aggregate rate enforcement across all Non-GBR SDFs for same APN |
| GBR rate enforcement | DL rate enforcement on accumulated MBRs of SDFs with same GBR QCI |
| Bearer binding | Maps SDFs to EPS bearers (UL bearer binding verification, DL TDF binding) |
| Packet screening | Verifies UE uses assigned IPv4 address/IPv6 prefix |
| "End marker" on SGW change | Sends end-marker to old SGW when SGW changes |
| PCC features | Interfaces PCRF (Gx) and OCS for online charging; enforces policy rules |
| Non-IP data | Supports Non-IP PDN type for CIoT EPS Optimisations |

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| S5 | SGW | GTPv2-C + GTP-U | Bearer management + user plane (intra-PLMN) |
| S8 | SGW (VPLMN) | GTP | Inter-PLMN variant of S5 (home-routed roaming) |
| SGi | External PDN | IP | Data path to Internet, IMS, enterprise networks |
| Gx | PCRF | Diameter | Receive policy/charging rules from PCRF; report events |
| Gy | OCS | Diameter | Online charging (credit control) |
| Gz/Rf | OFCS | Diameter | Offline charging records |
| S2a/S2b | ePDG / Trusted non-3GPP | GTP/PMIP | Non-3GPP access (see TS 23.402) |

## PCEF Role

The PGW hosts the PCEF, which:
1. Detects service data flows (SDFs) via DPI or static configuration
2. Applies Policy and Charging Control (PCC) rules received from PCRF via Gx
3. Enforces QoS (gating, rate limits) per SDF and per bearer
4. Reports usage/events back to PCRF

**Gx session lifecycle:**
- PGW initiates Gx session to PCRF on PDN connection creation (CCR-Initial)
- PCRF installs default PCC rules (e.g. for IMS signaling bearer)
- PCRF can push new rules via RAR (e.g. for VoLTE dedicated bearer)
- PGW reports events (session start/stop, usage thresholds) via CCR-Update/Terminate

## IP Address Allocation

- **IPv4**: assigned from pool configured per APN; PGW acts as DHCPv4 server or allocates directly
- **IPv6**: PGW assigns IPv6 prefix via DHCPv6 or Router Advertisement
- **Dual-stack**: both IPv4 and IPv6 allocated for same PDN connection
- Address persists for lifetime of PDN connection

## Bearer Binding

- PGW receives SDF packet filters from PCRF (via PCC rules)
- PGW binds SDFs to EPS bearers based on QCI/ARP matching
- Default bearer: catch-all for traffic not matching dedicated bearer filters
- Dedicated bearer: specific SDFs (e.g. VoLTE RTP media → GBR bearer with QCI=1)

## Related Pages
- [SGW](SGW.md) — user plane peer via S5/S8
- [PCRF](PCRF.md) — policy master via Gx
- [EPS bearer model](../concepts/EPS-bearer.md)
- [Reference points](../interfaces/reference-points.md)
