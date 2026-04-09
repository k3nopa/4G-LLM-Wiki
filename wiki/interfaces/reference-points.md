---
title: "EPC Reference Points (Interfaces)"
type: interface
tags: [interfaces, S1, S5, S6a, S11, Gx, Rx, SGi, GTP, Diameter, reference-points]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# EPC Reference Points

**Spec reference:** 3GPP TS 23.401 §4.2.3

## Complete Reference Point Table

| Interface | Between | Protocol | Plane | Purpose |
|---|---|---|---|---|
| **S1-MME** | eNodeB ↔ MME | S1-AP (SCTP) | Control | NAS relay, E-RAB management, paging, handover signaling |
| **S1-U** | eNodeB ↔ SGW | GTP-U (UDP) | User | Per-bearer GTP-U tunnels for UE data traffic |
| **S3** | MME ↔ SGSN | GTPv2-C | Control | Inter-3GPP access mobility (idle + active); intra- or inter-PLMN |
| **S4** | SGSN ↔ SGW | GTPv2-C + GTP-U | Both | 2G/3G anchor; user plane if no Direct Tunnel |
| **S5** | SGW ↔ PGW | GTPv2-C + GTP-U | Both | Intra-PLMN SGW–PGW bearer management + data |
| **S6a** | MME ↔ HSS | Diameter | Control | Authentication vectors, subscription download, location update |
| **S6d** | SGSN ↔ HSS | Diameter | Control | Same as S6a toward SGSN |
| **S8** | SGW (VPLMN) ↔ PGW (HPLMN) | GTP | Both | Inter-PLMN variant of S5 (home-routed roaming) |
| **S9** | H-PCRF ↔ V-PCRF | Diameter | Control | Policy/charging transfer for roaming local breakout |
| **S10** | MME ↔ MME | GTPv2 | Control | UE context transfer during inter-MME HO or relocation |
| **S11** | MME ↔ SGW | GTPv2-C | Control | Session/bearer lifecycle (Create/Modify/Delete Session, bearer) |
| **S12** | UTRAN (RNC) ↔ SGW | GTP-U | User | Direct Tunnel; bypasses SGW user plane (operator option) |
| **S13** | MME ↔ EIR | — | Control | UE identity (IMEI) check |
| **SGi** | PGW ↔ PDN | IP | User | Data path to external PDNs (Internet, IMS, enterprise) |
| **Gx** | PCRF ↔ PGW | Diameter | Control | PCC rules (QoS + charging) from PCRF to PCEF |
| **Rx** | PCRF ↔ AF (P-CSCF) | Diameter | Control | Session info from IMS to PCRF for QoS authorization |
| **Gy** | PGW ↔ OCS | Diameter | Control | Online credit control |
| **Gz/Rf** | PGW/SGW ↔ OFCS | Diameter | Control | Offline charging records |
| **SGs** | MME ↔ MSC/VLR | SGs-AP | Control | CS Fallback, SMS over SGs |
| **S7a** | MME ↔ CSS | — | Control | CSG subscription data for roaming UEs |
| **Nq** | MME ↔ RCAF | — | Control | RAN congestion info: RCAF → MME |
| **Np** | RCAF ↔ PCRF | — | Control | RAN User Plane Congestion Info: RCAF → PCRF |
| **X2** | eNodeB ↔ eNodeB | X2-AP | Control | Inter-eNB handover (direct, no MME involvement) |
| **LTE-Uu** | UE ↔ eNodeB | LTE-Uu (RRC+PDCP) | Both | Air interface |

## Key Protocol Notes

- **GTPv2-C**: GTP version 2 control plane; used on S11, S5, S8, S10, S3, S4
- **GTP-U**: GTP user plane (UDP encapsulated); used on S1-U, S5, S8, S12
- **PMIP variant**: S5/S8 can use Proxy Mobile IP instead of GTP (see TS 23.402); not common in commercial deployments
- **S1-AP**: runs over SCTP (Stream Control Transmission Protocol)
- **Diameter**: all Gx, Rx, S6a, S9 interfaces; application-layer AAA/policy protocol

## Architecture Diagrams

### Non-Roaming (Standard)
```
UE ─LTE-Uu─ eNodeB ─S1-MME─ MME ─S6a─ HSS
                  └──S1-U──  SGW ─S11─ MME
                              │S5
                             PGW ─Gx─ PCRF ─Rx─ AF/P-CSCF
                              └──SGi──► IMS / Internet
```

### Roaming — Home Routed
```
VPLMN: UE ─ eNodeB ─ MME ─ SGW
                              │S8
HPLMN:                       PGW ─Gx─ H-PCRF ─Rx─ AF
                               └──SGi──► IMS
HSS in HPLMN; MME in VPLMN queries HSS via S6a
```

### Roaming — Local Breakout
```
VPLMN: UE ─ eNodeB ─ MME ─ SGW ─S5─ PGW ─Gx─ V-PCRF ─S9─ H-PCRF
                                        └──SGi──► Visited PDN
```

## Related Pages
- [MME](../entities/MME.md)
- [SGW](../entities/SGW.md)
- [PGW](../entities/PGW.md)
- [PCRF](../entities/PCRF.md)
- [HSS](../entities/HSS.md)
