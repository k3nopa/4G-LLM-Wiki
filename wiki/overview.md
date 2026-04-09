---
title: "4G EPC & IMS — Domain Overview"
type: overview
tags: [EPC, IMS, LTE, 4G, architecture]
sources: [ts_123401v150400p.pdf, ts_123228v150600p.pdf]
updated: 2026-04-08
---

# 4G EPC & IMS — Domain Overview

> This page is the living synthesis of the wiki. It evolves with every source ingested.

## Current Thesis

4G LTE separates concerns cleanly into two complementary subsystems:

- **EPC (Evolved Packet Core)** handles all IP connectivity — attaching devices, managing mobility, routing data, enforcing policy, and billing
- **IMS (IP Multimedia Subsystem)** rides on top of EPC to deliver voice, video, and messaging as SIP-based services — most critically VoLTE

Together they form the **EPS (Evolved Packet System)**: EPS = E-UTRAN + EPC + IMS.

---

## Architecture Sketch

```
UE ─── eNodeB ─── [EPC] ─────────────────── Internet
   LTE-Uu   S1-MME/S1-U                        │
                    │                           SGi
              ┌─────┴──────┐                    │
              MME          SGW ──S5──── PGW ────┤
              │S6a          S11     Gx  │        │
              HSS          MME      PCRF         IMS APN
                                    │Rx          │
                                   P-CSCF ──Gm─ UE
                                    │Mw
                                   I-CSCF ──Cx── HSS
                                    │Mw
                                   S-CSCF ──ISC── TAS
                                    │Mi
                                   BGCF ──Mj── MGCF ── PSTN
```

**EPC key nodes:**
| Node | Role |
|---|---|
| MME | Control plane: authentication, attach, mobility, bearer management |
| SGW | User plane anchor in the RAN/core boundary; local mobility anchor |
| PGW | PDN gateway; UE IP allocation, PCEF, policy enforcement (Gx), SGi edge |
| HSS | Subscriber database: EPS auth vectors, subscription profiles, IMS service profiles |
| PCRF | Policy engine: Gx (bearer rules to PGW), Rx (session info from P-CSCF) |

**IMS key nodes:**
| Node | Role |
|---|---|
| P-CSCF | First SIP contact point for UE; IPsec security; AF on Rx for QoS |
| I-CSCF | Home IMS entry; queries HSS (Cx) for S-CSCF assignment; THIG |
| S-CSCF | Registrar + routing engine + iFC evaluator; ISC to ASes |
| TAS | Telephony AS: MMTEL features (call forwarding, hold, conferencing, barring) |
| BGCF | PSTN breakout selection: routes to MGCF (Mj) or peer BGCF (Mk) |
| MRFC/MRFP | Media resources: conference mixing, tones (SIP Mr + H.248 Mp) |
| HSS (shared) | Serves both EPC (S6a) and IMS (Cx for S-CSCF; Sh for ASes) |

---

## Key Interfaces

**EPC interfaces:** → see [EPC Reference Points](interfaces/reference-points.md)

| Interface | Between | Protocol |
|---|---|---|
| S1-MME | eNodeB ↔ MME | S1-AP (SCTP) |
| S1-U | eNodeB ↔ SGW | GTP-U (UDP) |
| S11 | MME ↔ SGW | GTPv2-C |
| S5/S8 | SGW ↔ PGW | GTPv2-C + GTP-U |
| S6a | MME ↔ HSS | Diameter |
| Gx | PCRF ↔ PGW (PCEF) | Diameter |
| Rx | P-CSCF (AF) ↔ PCRF | Diameter |

**IMS interfaces:** → see [IMS Reference Points](interfaces/IMS-reference-points.md)

| Interface | Between | Protocol |
|---|---|---|
| Gm | UE ↔ P-CSCF | SIP over IPsec |
| Mw | P-CSCF ↔ I-CSCF, I-CSCF ↔ S-CSCF | SIP |
| ISC | S-CSCF ↔ AS (TAS) | SIP |
| Cx | S-CSCF/I-CSCF ↔ HSS | Diameter |
| Sh | AS ↔ HSS | Diameter |
| Mi/Mj | S-CSCF/BGCF ↔ BGCF/MGCF | SIP |

---

## Key Procedures (to be expanded in Phase 2)

**EPC procedures (TS 23.401 §5):**
1. **EPS Attach** — UE→eNB→MME; S6a auth; S11 default bearer; EMM→REGISTERED
2. **Default Bearer Activation** — PGW allocates IP; Gx session establishment; SGW/eNB S1-U tunnels
3. **Dedicated Bearer Activation** — PCRF-triggered (Rx→Gx→RAR) for VoLTE GBR bearer; S11 bearer creation
4. **TAU (Tracking Area Update)** — UE moves to new TA; may trigger SGW relocation
5. **X2 Handover** — inter-eNB HO without MME; SGW path switch
6. **S1 Handover** — source eNB→MME→target eNB; path switch; possible SGW relocation

**IMS procedures (TS 23.228 §5):**
7. **IMS Registration** — REGISTER: UE→P-CSCF→I-CSCF→(Cx UAR)→S-CSCF→(Cx SAR, MAR)→200 OK
8. **VoLTE MO Call** — INVITE→P-CSCF (Rx AAR)→S-CSCF (iFC→TAS)→term. side; GBR bearer established
9. **VoLTE MT Call** — INVITE arrives at I-CSCF→(Cx LIR)→S-CSCF→pages UE→GBR bearer
10. **IMS De-registration** — UE sends REGISTER(Expires:0); S-CSCF→Cx SAR(UNREGISTRATION)→HSS

---

## Open Questions / Gaps

- Detailed NAS message sequences during EPS Attach (EMM/ESM state machines) — Phase 2
- Exact Diameter AVPs for S6a (ULR fields, subscription data format) and Gx PCC rules
- P-CSCF discovery: PCO mechanism detail (what PGW provides in PCO IE)
- Emergency call handling: IMS emergency registration flow; E-CSCF; location retrieval
- T-GRUU re-allocation: what changes on re-REGISTER? Does `+sip.instance` change?
- S-CSCF re-assignment on failure: how does HSS detect S-CSCF failure vs planned relocation?
- IP-SM-GW interworking with SMS-SC: SGd vs S6c Diameter interface detail
- Exact S9 signaling for roaming QoS: what Rx parameters cross V-PCRF→H-PCRF?
- Interworking with 5G: N26 interface, EPS fallback, IMS continuity during handover

---

## Sources Contributing to This Page

- **TS 23.401 §4** — EPC architecture, all network elements, EMM/ECM states, EPS bearer model, reference points
- **TS 23.228 §4** — IMS architecture, all CSCF roles, IMS identity model (IMPI/IMPU/GRUU), iFC/service control, roaming architectures, all IMS reference points
