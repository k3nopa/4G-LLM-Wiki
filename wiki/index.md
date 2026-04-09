# Wiki Index
_Last updated: 2026-04-09 — 29 pages total_

## Overview
- [overview.md](overview.md) — Domain synthesis: 4G EPC and IMS architecture, current understanding

## Entities

### EPC
- [entities/MME.md](entities/MME.md) — Mobility Management Entity: NAS, auth, mobility, bearer signaling
- [entities/SGW.md](entities/SGW.md) — Serving Gateway: user plane mobility anchor, ECM-IDLE buffering
- [entities/PGW.md](entities/PGW.md) — PDN Gateway: IP anchor, PCEF, rate enforcement, SGi edge
- [entities/PCRF.md](entities/PCRF.md) — Policy and Charging Rules Function: Gx/Rx, VoLTE bearer trigger
- [entities/HSS.md](entities/HSS.md) — Home Subscriber Server: EPC S6a data + full IMS Cx/Sh data

### IMS
- [entities/P-CSCF.md](entities/P-CSCF.md) — Proxy-CSCF: UE SIP entry, IPsec security, AF on Rx
- [entities/I-CSCF.md](entities/I-CSCF.md) — Interrogating-CSCF: home IMS entry, S-CSCF assignment via Cx, THIG
- [entities/S-CSCF.md](entities/S-CSCF.md) — Serving-CSCF: registrar, session routing, iFC evaluation, ISC to ASes
- [entities/TAS.md](entities/TAS.md) — Telephony Application Server: MMTEL services via ISC, Sh, Mr
- [entities/BGCF.md](entities/BGCF.md) — Breakout Gateway Control Function: PSTN routing (Mj/Mk)
- [entities/MRF.md](entities/MRF.md) — MRFC + MRFP: conference/media resources (SIP Mr + H.248 Mp)

## Concepts
- [concepts/EPS-bearer.md](concepts/EPS-bearer.md) — EPS bearer model: default/dedicated, TFT, QCI, GBR/Non-GBR
- [concepts/EMM-ECM-states.md](concepts/EMM-ECM-states.md) — EMM/ECM state machines: REGISTERED/DEREGISTERED, IDLE/CONNECTED
- [concepts/IMS-identity-model.md](concepts/IMS-identity-model.md) — IMPI, IMPU, GRUU, Implicit Registration Set, service profile, iFC

## Procedures
- [procedures/EPS-attach.md](procedures/EPS-attach.md) — E-UTRAN Initial Attach: 26-step procedure establishing EMM registration, default EPS bearer, and PDN address (TS 23.401 §5.3.2.1)
- [procedures/TAU.md](procedures/TAU.md) — Tracking Area Update: triggers, S-GW change and no-change variants, ISR, bearer context transfer, HSS location update (TS 23.401 §5.3.3)
- [procedures/service-request.md](procedures/service-request.md) — Service Request (UE/network triggered, DDN, paging, extended buffering) and S1 Release (bearer preservation rules) (TS 23.401 §5.3.4–5.3.5)
- [procedures/detach.md](procedures/detach.md) — EPS Detach: UE-, MME-, SGSN-, HSS-initiated variants, ISR teardown, Switch Off handling (TS 23.401 §5.3.8)
- [procedures/dedicated-bearer.md](procedures/dedicated-bearer.md) — Bearer management: dedicated bearer activation, QoS modification, deactivation, UE-requested resource modification (TS 23.401 §5.4.1–5.4.5)
- [procedures/X2-handover.md](procedures/X2-handover.md) — X2-based intra-E-UTRAN handover: without and with SGW relocation, path switch, end-marker, bearer failure handling (TS 23.401 §5.5.1.1)
- [procedures/S1-handover.md](procedures/S1-handover.md) — S1-based intra-E-UTRAN handover: normal (21-step), reject, cancel; MME/SGW relocation, indirect forwarding, Operation Indication flag (TS 23.401 §5.5.1.2)

## Protocols
_No pages yet._

## Interfaces
- [interfaces/reference-points.md](interfaces/reference-points.md) — All EPC reference points: S1, S5, S6a, S11, Gx, Rx, SGi and more
- [interfaces/IMS-reference-points.md](interfaces/IMS-reference-points.md) — All IMS reference points: Gm, Mw, ISC, Cx, Sh, Rx, Mi, Mj, Mr and more

## Sources
- [sources/ts23401-section4.md](sources/ts23401-section4.md) — 3GPP TS 23.401 §4: EPC architecture, network elements, EMM/ECM, bearer model
- [sources/ts23228-section4.md](sources/ts23228-section4.md) — 3GPP TS 23.228 §4: IMS architecture, CSCF roles, identity model, reference points

## Analyses
_No analyses filed yet._
