---
title: "IMS Reference Points (Interfaces)"
type: interface
tags: [IMS, interfaces, Gm, Mw, ISC, Cx, Sh, Mr, Mp, Mi, Mj, Mk, Rx, Ut, Ici, Izi, Mx, II-NNI, SIP, Diameter]
sources: [ts_123228v150600p.pdf, ts_129165v160600p.pdf, ts_123237v180000p.pdf]
updated: 2026-04-18
---

# IMS Reference Points

**Spec reference:** 3GPP TS 23.228 §4.2

## Complete Reference Point Table

| Interface | Between | Protocol | Plane | Purpose |
|---|---|---|---|---|
| **Gm** | UE ↔ P-CSCF | SIP (over IPsec) | Control | UE registration; originating/terminating SIP sessions |
| **Mw** | P-CSCF ↔ I-CSCF, P-CSCF ↔ S-CSCF, I-CSCF ↔ S-CSCF | SIP | Control | Intra-IMS SIP forwarding between CSCFs |
| **ISC** | S-CSCF ↔ AS (TAS, MMTEL, etc.) | SIP | Control | Service trigger; iFC-driven AS invocation |
| **Cx** | S-CSCF ↔ HSS, I-CSCF ↔ HSS | Diameter | Control | Auth (MAR/MAA), profile download (SAR/SAA), S-CSCF assign (UAR/UAA), location query (LIR/LIA), de-reg (RTR/RTA), profile push (PPR/PPA) |
| **Sh** | AS ↔ HSS | Diameter | Control | AS read/write subscriber service data; change notification subscription |
| **Rx** | P-CSCF ↔ PCRF | Diameter | Control | P-CSCF (as AF) sends session info to PCRF for QoS bearer authorization |
| **Mg** | MGCF → I-CSCF | SIP | Control | PSTN-originated call enters IMS via MGCF→I-CSCF |
| **Mi** | S-CSCF → BGCF | SIP | Control | Route IMS-originated session toward PSTN |
| **Mj** | BGCF → MGCF | SIP | Control | Local PSTN breakout: BGCF routes to MGCF |
| **Mk** | BGCF ↔ BGCF | SIP | Control | Inter-network BGCF routing for remote PSTN breakout |
| **Mx** | BGCF / CSCF / ATCF / MSC Server enhanced for ICS/SRVCC/dual radio ↔ [IBCF](../entities/IBCF.md) | SIP | Control | Internal IMS network signalling toward/from IBCF (the II-NNI border element) |
| **Ici** | [IBCF](../entities/IBCF.md) ↔ [IBCF](../entities/IBCF.md) (peer network) | SIP | Control | **II-NNI control plane** — SIP signalling between two IM CN subsystems |
| **Izi** | [TrGW](../entities/TrGW.md) ↔ [TrGW](../entities/TrGW.md) (peer network) | RTP/UDP | User | **II-NNI user plane** — media streams between two IM CN subsystems |
| **Mr** | S-CSCF or AS ↔ MRFC | SIP | Control | Request conference/announcement media resources |
| **Mp** | MRFC → MRFP | H.248 (MEGACO) | Control | MRFC controls MRFP: create/modify/delete media connections |
| **Ma** | I-CSCF → AS | SIP | Control | Direct termination to AS (certain scenarios, bypasses S-CSCF assignment) |
| **Mm** | IMS ↔ external IP network | SIP | Control | External SIP network interworking (e.g. between operators) |
| **Ut** | UE ↔ AS | HTTP(S)/XCAP | Control | Subscriber-managed AS configuration (call forwarding targets, barring) |
| **Iq** | [ATCF](../entities/ATCF.md) ↔ [ATGW](../entities/ATGW.md) | H.248 / proprietary | Control | ATCF controls ATGW when ATCF is co-located with P-CSCF (per TS 23.334) |
| **Ix** | [ATCF](../entities/ATCF.md) ↔ [ATGW](../entities/ATGW.md) | H.248 / proprietary | Control | ATCF controls ATGW when ATCF is co-located with IBCF (per TS 29.162) |
| **I2** | [ATCF](../entities/ATCF.md) ↔ MSC Server (ICS-enhanced) | SIP | Control | ATCF ↔ ICS-capable MSC Server for SRVCC and CS-to-PS SRVCC; also used for CS-to-PS direction |

---

## Cx Interface — Diameter Commands

| Command | From → To | Trigger | Key Parameters |
|---|---|---|---|
| **UAR/UAA** (User Auth Request) | I-CSCF → HSS | REGISTER arrives | Public identity, visited network ID → returns S-CSCF name or capabilities |
| **SAR/SAA** (Server Assignment Request) | S-CSCF → HSS | After auth success; de-reg | Assignment type (REGISTER, UNREGISTER, etc.) → returns service profile (iFCs) |
| **MAR/MAA** (Multimedia Auth Request) | S-CSCF → HSS | During IMS AKA | IMPI, auth scheme → returns auth vectors (AUTN, XRES, CK, IK) |
| **LIR/LIA** (Location Info Request) | I-CSCF → HSS | Terminating session | Public identity → returns S-CSCF name serving that IMPU |
| **RTR/RTA** (Registration Termination Request) | HSS → S-CSCF | HSS-initiated de-reg | De-registration reason (subscription change, admin) |
| **PPR/PPA** (Push Profile Request) | HSS → S-CSCF | Profile change at HSS | Updated service profile (iFCs) pushed to S-CSCF without re-registration |

---

## Rx Interface — Key Diameter AVPs

Used by P-CSCF to request QoS policy for IMS sessions:

| Direction | Message | Purpose |
|---|---|---|
| P-CSCF → PCRF | **AAR** (AA-Request) | Session establishment: media components, IP flows, required QoS |
| PCRF → P-CSCF | **AAA** (AA-Answer) | Authorization granted/denied |
| P-CSCF → PCRF | **AAR** (modify) | Session modification: re-INVITE with changed SDP |
| P-CSCF → PCRF | **STR** (Session-Termination-Request) | Session teardown (BYE received) |
| PCRF → P-CSCF | **ASR** (Abort-Session-Request) | PCRF-initiated session termination |

Key AVPs in AAR: `Media-Component-Description`, `Flow-Description`, `AF-Application-Identifier`, `Service-Info-Status`, `Specific-Action`.

---

## Architecture Diagrams

### Standard VoLTE Session Path
```
UE ──Gm──► P-CSCF ──Mw──► I-CSCF ──Mw──► S-CSCF ──ISC──► TAS
                    │                        │
                    Rx                      Mi
                    │                        │
                  PCRF ◄──Gx──► PGW         BGCF ──Mj──► MGCF ──► PSTN
```

### Registration Path
```
UE ──Gm──► P-CSCF ──Mw──► I-CSCF ──Cx UAR──► HSS
                                     ◄──UAA──
                            I-CSCF ──Mw──► S-CSCF ──Cx SAR──► HSS
                                                     ◄──SAA (service profile)──
```

### IMS QoS Bearer Path
```
SDP in INVITE ──► P-CSCF ──Rx AAR──► PCRF ──Gx RAR──► PGW
                                              PGW establishes GBR bearer
                                     PCRF ──Rx AAA──► P-CSCF (granted)
```

---

## Protocol Notes

- **SIP** (RFC 3261) used on all Gm, Mw, ISC, Mi, Mj, Mk, Mx, Mr, Ma, Mm interfaces
- **Diameter** (RFC 6733 + 3GPP extensions) on Cx, Sh, Rx
- **H.248/MEGACO** (ITU-T H.248) on Mp
- **XCAP** (XML Configuration Access Protocol, RFC 4825) on Ut for subscriber configuration
- All SIP within IMS uses **SIP Identity** mechanisms and `P-Asserted-Identity` / `P-Access-Network-Info` headers
- IPsec between UE and P-CSCF on Gm; inter-node SIP within IMS typically protected by TLS

---

## Related Pages
- [P-CSCF](../entities/P-CSCF.md)
- [I-CSCF](../entities/I-CSCF.md)
- [S-CSCF](../entities/S-CSCF.md)
- [TAS](../entities/TAS.md)
- [BGCF](../entities/BGCF.md)
- [MRF](../entities/MRF.md)
- [HSS](../entities/HSS.md)
- [PCRF](../entities/PCRF.md) — Rx peer
- [IBCF](../entities/IBCF.md) — II-NNI control-plane border element
- [TrGW](../entities/TrGW.md) — II-NNI user-plane border element
- [ATCF](../entities/ATCF.md) — Access Transfer Control Function (Iq/Ix/I2 interfaces)
- [ATGW](../entities/ATGW.md) — Access Transfer Gateway (Iq/Ix interfaces)
- [II-NNI Interface](II-NNI.md) — full Ici/Izi specification
- [EPC Reference Points](reference-points.md)
