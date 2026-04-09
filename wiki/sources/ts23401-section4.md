---
title: "3GPP TS 23.401 §4 — EPC Architecture & Network Elements"
type: source
tags: [EPC, architecture, MME, SGW, PGW, PCRF, EMM, ECM, bearer, reference-points]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# Source: 3GPP TS 23.401 v15.4.0 §4

**Spec:** 3GPP TS 23.401 Release 15 (2018-07)
**Section ingested:** §4 (Architecture model and concepts), pages 19–94
**File:** `raw/papers/ts_123401v150400p.pdf`

## Abstract

Section 4 of TS 23.401 defines the complete EPC architecture model: the reference architecture diagrams (non-roaming, roaming, local breakout), normative definitions of all reference points, high-level functional groupings, network element specifications, EPS mobility/connection state machines, and the EPS bearer model.

## Key Facts Extracted

### Architecture
- EPS = E-UTRAN + EPC. EPC = MME + SGW + PGW + HSS + PCRF.
- Two gateway deployment options: SGW and PGW as separate nodes (S5 between them) or collocated (Single Gateway option).
- Roaming: home-routed (PGW in HPLMN, S8 between SGW in VPLMN and PGW in HPLMN) or local breakout (PGW in VPLMN, V-PCRF + H-PCRF linked via S9).

### Reference Points (§4.2.3)
| Interface | Between | Protocol | Purpose |
|---|---|---|---|
| S1-MME | E-UTRAN ↔ MME | S1-AP | Control plane (NAS relay, S1 procedures) |
| S1-U | E-UTRAN ↔ SGW | GTP-U | User plane bearer tunnels |
| S3 | MME ↔ SGSN | GTP | Inter-3GPP access mobility (idle + active) |
| S4 | SGSN ↔ SGW | GTP | 2G/3G anchor + user plane if no Direct Tunnel |
| S5 | SGW ↔ PGW | GTPv2-C + GTP-U | Intra-PLMN SGW–PGW (PMIP variant in TS 23.402) |
| S6a | MME ↔ HSS | Diameter | Subscription data, authentication vectors, location update |
| S8 | SGW (VPLMN) ↔ PGW (HPLMN) | GTP | Inter-PLMN variant of S5 |
| S9 | H-PCRF ↔ V-PCRF | Diameter | Policy/charging transfer for local breakout |
| S10 | MME ↔ MME | GTPv2 | MME relocation, UE context transfer |
| S11 | MME ↔ SGW | GTPv2-C | Bearer management between MME and SGW |
| S12 | UTRAN ↔ SGW | GTP-U | Direct Tunnel (operator config option) |
| S13 | MME ↔ EIR | — | UE identity check |
| SGi | PGW ↔ PDN | IP | Interface to external packet data networks (IMS, Internet) |
| Rx | PCRF ↔ AF (P-CSCF) | Diameter | QoS/charging policy from application to PCRF |
| Gx | PCRF ↔ PGW (PCEF) | Diameter | Policy and charging rules delivery |
| S7a | MME ↔ CSS | — | CSG subscription data for roaming |
| Nq | MME ↔ RCAF | — | RAN congestion info from RCAF to MME |
| Np | RCAF ↔ PCRF | — | RAN User Plane Congestion Information (RUCI) |

### Network Elements (§4.4)

**MME** — control plane only, no user plane.
Functions: NAS signaling + security, S3 inter-CN, ECM-IDLE UE reachability + paging, TAI list management, TAI→timezone, PGW/SGW/MME selection, S6a (HSS auth + subscription), authentication, authorization, bearer management (dedicated bearer establishment), lawful interception (signaling), warning message distribution, RFSP Index provision to eNB, CIoT CP optimisation (user data transport + header compression).

**Serving GW (SGW)** — user plane anchor at RAN/core boundary.
Functions: local mobility anchor for inter-eNB HO, "end marker" sending on SGW switch, inter-3GPP mobility anchor (S4), ECM-IDLE DL packet buffering + DDN triggering, lawful interception, packet routing/forwarding, DSCP/QCI marking, per-UE/bearer accounting, OFCS interface (TS 32.240).

**PDN GW (PGW)** — edge to external networks.
Functions: SGi termination, per-user DPI packet filtering, lawful interception, UE IP allocation, UL/DL DSCP marking, per-bearer charging + OFCS, UL/DL gating, UL/DL rate enforcement (SDF level per TS 23.203), APN-AMBR enforcement, DHCPv4/v6 (server+client), PCC features (PCRF, OCS), UL+DL bearer binding verification, packet screening, "end marker" on SGW change.

**PCRF** — policy and charging control.
Non-roaming: single H-PCRF per IP-CAN session; terminates Gx (to PGW/PCEF) and Rx (to AF).
Roaming local breakout: V-PCRF (Gx + S9 + Rx for visited AF), H-PCRF (Rx for home services + S9).

**HSS** — subscriber master database.
Accessed via S6a (from MME, Diameter). Stores: subscription profiles, authentication vectors, RFSP Index, registered MME identity, IMS voice over PS support indication. (Full definition in TS 23.228.)

**SGSN** — 2G/3G core node interfacing EPC.
Added functions vs TS 23.060: inter-EPC signaling for 2G/3G↔E-UTRAN, PGW/SGW selection, timezone, MME selection for HO to E-UTRAN.

**eNodeB (E-UTRAN)** — LTE base station.
Additional EPC functions: UE header compression (S1-U), UE plane ciphering, MME selection when routing unknown, UL/DL bearer rate enforcement (UE-AMBR, MBR), admission control, DSCP marking.

**RCAF** — RAN Congestion Awareness Function.
Collects user plane congestion info from RAN OAM. Sends RUCI to MME via Nq; sends RUCI to PCRF via Np.

**CSS** — CSG Subscriber Server.
Stores VPLMN-specific CSG subscription data for roaming UEs. Accessible from MME via S7a.

### EMM/ECM States (§4.6)

**EMM states (mobility — MME perspective):**
- `EMM-DEREGISTERED`: no valid location/routing info; UE not reachable. Some UE context may persist (avoid re-running AKA).
- `EMM-REGISTERED`: UE registered; location known at TAI list granularity; at least one active PDN connection (unless "Attach without PDN connectivity").

**ECM states (signaling connectivity):**
- `ECM-IDLE`: no NAS signaling connection; no UE context in E-UTRAN; no S1-MME/S1-U.
- `ECM-CONNECTED`: S1-MME established; UE location known to serving eNB granularity; signaling connection = RRC connection + S1-MME connection.

**Transitions:**
- ECM-IDLE → ECM-CONNECTED: Attach Request, TAU Request, Service Request, Detach Request, Connection Resume
- ECM-CONNECTED → ECM-IDLE: S1 release procedure or S1 Connection Suspend (CIoT)

**Important:** EMM and ECM states are independent. EMM-DEREGISTERED can occur in either ECM state (e.g. explicit detach while ECM-CONNECTED).

### EPS Bearer Model (§4.7)

- **EPS bearer**: unit of QoS treatment; uniquely identifies traffic flows with same scheduling/queuing/shaping between UE and PGW (GTP-based) or UE and SGW (PMIP-based).
- **Default bearer**: established at PDN connection setup; persists for lifetime of PDN connection; provides always-on IP connectivity.
- **Dedicated bearer**: additional bearers for same PDN connection (e.g. GBR bearer for VoLTE media).
- **TFT (Traffic Flow Template)**: set of packet filters for a bearer.
  - UL TFT: uplink packet filters (used by UE to map uplink traffic to bearers)
  - DL TFT: downlink packet filters (used by PCEF/BBERF)
- Bearer level QoS params: QCI, ARP, GBR, MBR, APN-AMBR, UE-AMBR.
- MME sets initial default bearer QoS values from HSS subscription data.

## Entities Mentioned
MME, SGW, PGW, PCRF (H-PCRF, V-PCRF), HSS, eNodeB, SGSN, GERAN, UTRAN, UE, RCAF, CSS, HeNB, DeNB, L-GW

## Concepts Introduced/Refined
EPS bearer, default bearer, dedicated bearer, TFT, EMM states, ECM states, TAI list, ISR (Idle mode Signalling Reduction), RFSP Index, Dual Connectivity, MCG/SCG bearer, GTP-C Load Control, GTP-C Overload Control, MME Pool Area, Weight Factor, LIPA, SIPTO, DCN (Dedicated Core Network), GBR bearer, Non-GBR bearer

## Questions Raised
- What Diameter AVPs flow on S6a during Attach (Update-Location-Request/Answer)?
- What triggers the MME to select a specific PGW (APN resolution, DNS, Gx)?
- How does ISR interact with VoLTE call setup?
- What is the complete PCRF Gx session lifecycle?
