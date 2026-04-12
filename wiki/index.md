# Wiki Index
_Last updated: 2026-04-12 — 63 pages total (5b-3 complete)_

## Overview
- [overview.md](overview.md) — Domain synthesis: 4G EPC and IMS architecture, current understanding

## Entities

### EPC
- [entities/MME.md](entities/MME.md) — Mobility Management Entity: NAS, auth, mobility, bearer signaling
- [entities/MME-deepdive.md](entities/MME-deepdive.md) — MME deep-dive synthesis: all interfaces, messages, state machines, procedure participation, UE context model, failure/overload behavior
- [entities/SGW.md](entities/SGW.md) — Serving Gateway: user plane mobility anchor, ECM-IDLE buffering
- [entities/SGW-deepdive.md](entities/SGW-deepdive.md) — SGW deep-dive synthesis: all interfaces, GTPv2-C/PMIPv6/Gxc messages, ECM-IDLE buffering/DDN, mobility anchor behavior, BBERF role in PMIP mode, bearer context model, procedure participation
- [entities/PGW.md](entities/PGW.md) — PDN Gateway: IP anchor, PCEF, rate enforcement, SGi edge
- [entities/PGW-deepdive.md](entities/PGW-deepdive.md) — PGW deep-dive synthesis: all interfaces, GTPv2-C/PMIPv6/Diameter messages, PCEF bearer binding, IP allocation, procedure participation across GTP and PMIP modes, charging, failure behavior
- [entities/PCRF.md](entities/PCRF.md) — Policy and Charging Rules Function: Gx/Rx, VoLTE bearer trigger
- [entities/PCRF-deepdive.md](entities/PCRF-deepdive.md) — PCRF deep-dive synthesis: all interfaces (Gx/Gxc/Gxa/Gxb/Rx/S9/Sp/Np), full Diameter message tables, PCC rule generation, VoLTE gate control sequence, session correlation, roaming S9, non-3GPP access modes, failure behavior
- [entities/HSS.md](entities/HSS.md) — Home Subscriber Server: EPC S6a data + full IMS Cx/Sh data
- [entities/HSS-deepdive.md](entities/HSS-deepdive.md) — HSS deep-dive synthesis: all interfaces (S6a/Cx/Sh/SWx/Dh), full Diameter message tables, EPC and IMS subscriber data models, AV generation, IMS registration state machine, EPC location state machine, multi-HSS/SLF, procedure participation
- [entities/ePDG.md](entities/ePDG.md) — Evolved Packet Data Gateway: untrusted non-3GPP access gateway, IKEv2/IPsec termination, S2b MAG (TS 23.402 §4.3.4)

### IMS
- [entities/P-CSCF.md](entities/P-CSCF.md) — Proxy-CSCF: UE SIP entry, IPsec security, AF on Rx
- [entities/P-CSCF-deepdive.md](entities/P-CSCF-deepdive.md) — P-CSCF deep-dive synthesis: Gm/Mw/Rx interfaces, IPsec SA negotiation, Rx session lifecycle, SDP→AAR mapping, P-header insertion, procedure participation, emergency call handling, SigComp
- [entities/I-CSCF.md](entities/I-CSCF.md) — Interrogating-CSCF: home IMS entry, S-CSCF assignment via Cx, THIG
- [entities/S-CSCF.md](entities/S-CSCF.md) — Serving-CSCF: registrar, session routing, iFC evaluation, ISC to ASes
- [entities/S-CSCF-deepdive.md](entities/S-CSCF-deepdive.md) — S-CSCF deep-dive synthesis: all interfaces, Cx messages, iFC evaluation algorithm, ODI chain, session routing logic, 7 procedure flows, AS interaction modes, registration state model, capabilities, failure behavior
- [entities/TAS.md](entities/TAS.md) — Telephony Application Server: MMTEL services via ISC, Sh, Mr
- [entities/TAS-deepdive.md](entities/TAS-deepdive.md) — TAS deep-dive synthesis: all interfaces (ISC/Sh/Mr/Mr'/Ut/Dh/Cr), MMTEL service logic (call forwarding tree, barring, CLIR, hold, conference, MWI), all 5 AS modes, voicemail flows, transcoding, Ut/XCAP self-service, data model, procedure participation
- [entities/BGCF.md](entities/BGCF.md) — Breakout Gateway Control Function: PSTN routing (Mj/Mk)
- [entities/MRF.md](entities/MRF.md) — MRFC + MRFP: conference/media resources (SIP Mr + H.248 Mp); tones, transcoding, Cr/Mr' interfaces (TS 23.218 §8)
- [entities/MRB.md](entities/MRB.md) — Media Resource Broker: Query mode (Rc), In-Line mode (Mr'), MRF pool management (TS 23.218 §13)

### IMS Emergency
- [entities/E-CSCF.md](entities/E-CSCF.md) — Emergency-CSCF: routes emergency sessions to PSAP via LRF; always in serving network; interfaces Mw/Ml/Mi/Mg/Mm/Mx/I4/I6 (TS 23.167 §6.2.2)
- [entities/LRF.md](entities/LRF.md) — Location Retrieval Function: resolves UE location → PSAP routing (ESQK/ESRN/URI); consists of RDF + Location Server; Ml to E-CSCF, Sh to HSS, Le to PSAP (TS 23.167 §6.2.3)

## Concepts
- [concepts/EPS-bearer.md](concepts/EPS-bearer.md) — EPS bearer model: default/dedicated, TFT, QCI, GBR/Non-GBR
- [concepts/EMM-ECM-states.md](concepts/EMM-ECM-states.md) — EMM/ECM state machines: REGISTERED/DEREGISTERED, IDLE/CONNECTED
- [concepts/IMS-identity-model.md](concepts/IMS-identity-model.md) — IMPI, IMPU, GRUU, Implicit Registration Set, service profile, iFC
- [concepts/information-storage.md](concepts/information-storage.md) — Per-node data models: HSS subscription, MME MM context, SGW/PGW bearer context, UE context; charging and Secondary RAT reporting (TS 23.401 §5.7)
- [concepts/IM-call-model.md](concepts/IM-call-model.md) — IM Call Model: S-CSCF functional architecture, SPTs, iFC evaluation algorithm, ODI tracking, Transit Function, ICSI/IARI, per-direction handling, session release, charging/ICID/IOI, subscriber data detail (TS 23.218 §4–§6.9)
- [concepts/AS-interaction-modes.md](concepts/AS-interaction-modes.md) — Application Server interaction modes: 5 modes (terminating UA, originating UA, SIP proxy, B2BUA, not involved), AS functional model (AS-ILCM/OLCM/Logic), interfaces (ISC/Sh/Dh/Cr/Rc), session handling procedures (TS 23.218 §9)
- [concepts/non-3GPP-access-architecture.md](concepts/non-3GPP-access-architecture.md) — Non-3GPP access: trusted/untrusted, IPMS, reference models/points, ANDSF policies, QoS/IPsec SA model, identities, IP allocation, charging, detach principles (TS 23.402 §4)
- [concepts/IMS-emergency-architecture.md](concepts/IMS-emergency-architecture.md) — IMS emergency framework: 30 architectural principles, E-CSCF/LRF reference model, UE-detectable vs non-detectable flows, location principles, IP-CAN expectations, NG-eCall introduction (TS 23.167 §4–§6)
- [concepts/SRVCC.md](concepts/SRVCC.md) — Single Radio Voice Call Continuity: all 6 variants (E-UTRAN→1xCS/GERAN, vSRVCC, UTRAN(HSPA), CS-to-PS, 5G-SRVCC), architecture principles, new entities (1xCS IWS, MSC Server enhanced, MME_SRVCC), reference points (S102, Sv, N26), PCC role, emergency SRVCC (TS 23.216)

## Procedures
- [procedures/EPS-attach.md](procedures/EPS-attach.md) — E-UTRAN Initial Attach: 26-step procedure establishing EMM registration, default EPS bearer, and PDN address (TS 23.401 §5.3.2.1)
- [procedures/TAU.md](procedures/TAU.md) — Tracking Area Update: triggers, S-GW change and no-change variants, ISR, bearer context transfer, HSS location update (TS 23.401 §5.3.3)
- [procedures/service-request.md](procedures/service-request.md) — Service Request (UE/network triggered, DDN, paging, extended buffering) and S1 Release (bearer preservation rules) (TS 23.401 §5.3.4–5.3.5)
- [procedures/detach.md](procedures/detach.md) — EPS Detach: UE-, MME-, SGSN-, HSS-initiated variants, ISR teardown, Switch Off handling (TS 23.401 §5.3.8)
- [procedures/dedicated-bearer.md](procedures/dedicated-bearer.md) — Bearer management: dedicated bearer activation, QoS modification, deactivation, UE-requested resource modification (TS 23.401 §5.4.1–5.4.5)
- [procedures/X2-handover.md](procedures/X2-handover.md) — X2-based intra-E-UTRAN handover: without and with SGW relocation, path switch, end-marker, bearer failure handling (TS 23.401 §5.5.1.1)
- [procedures/S1-handover.md](procedures/S1-handover.md) — S1-based intra-E-UTRAN handover: normal (21-step), reject, cancel; MME/SGW relocation, indirect forwarding, Operation Indication flag (TS 23.401 §5.5.1.2)
- [procedures/PDN-connectivity.md](procedures/PDN-connectivity.md) — UE requested PDN connectivity (16-step), PDN disconnection, MME-triggered SGW relocation, Location Change Reporting / PRA (TS 23.401 §5.10)
- [procedures/IMS-registration.md](procedures/IMS-registration.md) — P-CSCF discovery, S-CSCF assignment, IMS initial registration (11-step), re-registration, implicit registration, mobile/network-initiated de-registration (TS 23.228 §5.1–5.3)
- [procedures/IMS-QoS-bearer.md](procedures/IMS-QoS-bearer.md) — IMS QoS/PCC interaction: Rx authorization, gate control, preconditions, resource sharing, PSI routing, S-S#1 session flow (TS 23.228 §5.4.5–5.4.9, §5.4.12, §5.5.1)
- [procedures/VoLTE-MO-call.md](procedures/VoLTE-MO-call.md) — VoLTE mobile-originated call: S-S#2/3/4 routing variants, MO#1/MO#2 origination flows (33-step), PSTN-O, NI-O, AS-O origination procedures (TS 23.228 §5.5–5.6)
- [procedures/VoLTE-MT-call.md](procedures/VoLTE-MT-call.md) — VoLTE mobile-terminated call: MT#1/MT#2 (33-step), MT#3 CS-domain, PSTN-T, NI-T, AS-T#1–4 PSI termination variants, SLF HSS resolution, mid-session signalling, sessions without preconditions (TS 23.228 §5.7–5.8)
- [procedures/session-release.md](procedures/session-release.md) — IMS session release: terminal-initiated (18-step), PSTN-initiated, network-initiated P-CSCF and S-CSCF variants, signalling bearer loss handling; session hold and resume (TS 23.228 §5.10–5.11)
- [procedures/S2b-attach.md](procedures/S2b-attach.md) — Untrusted non-3GPP attach/detach via ePDG: PMIPv6 and GTP S2b variants (9-step), S2c/DSMIPv6, emergency attach, additional PDN connectivity, dedicated bearer activation/modification, PGW-initiated deactivation, QoS IPsec SA models (TS 23.402 §7)
- [procedures/non3GPP-handover.md](procedures/non3GPP-handover.md) — Non-3GPP → 3GPP handover without optimization: 18-step non-3GPP→E-UTRAN GTP flow, PMIP S5/S8 Alt A/B variants, UTRAN/GERAN variant, PGW reuse via HSS, IP continuity mechanism (TS 23.402 §8)
- [procedures/trusted-non3GPP-attach.md](procedures/trusted-non3GPP-attach.md) — Trusted non-3GPP (TWAN/S2a) attach/detach: PMIPv6 S2a 11-step, MIPv4 FACoA 14-step, DSMIPv6 S2c, chained PMIP S8-S2a roaming, detach variants (UE/TNAN/HSS/PGW-initiated), dynamic PCC, additional PDN connectivity (TS 23.402 §6)
- [procedures/SRVCC-from-E-UTRAN.md](procedures/SRVCC-from-E-UTRAN.md) — SRVCC from E-UTRAN: 1xCS 19-step flow (§6.1) + all 5 (v)SRVCC call flows to GERAN/UTRAN with/without PS HO and vSRVCC (§6.2) (TS 23.216)
- [procedures/SRVCC-from-UTRAN-HSPA.md](procedures/SRVCC-from-UTRAN-HSPA.md) — SRVCC from UTRAN(HSPA): 3 SGSN-based flows to GERAN/UTRAN including SRVCC CS KEYS exchange and Gn/Gp vs S4 bearer handling (§6.3) (TS 23.216)
- [procedures/PMIP-S5S8-procedures.md](procedures/PMIP-S5S8-procedures.md) — PMIP-based S5/S8 for 3GPP access: SGW-as-BBERF model, initial attach PMIP steps C.1–C.5, detach, dedicated bearer activation/modification/deactivation (SGW drives TFT), PDN connectivity, handover/TAU with SGW relocation, GTP vs PMIP differences (TS 23.402 §5)
- [procedures/IMS-emergency-session.md](procedures/IMS-emergency-session.md) — IMS emergency session: 3 paths (credentialed/anonymous/non-UE-detectable), emergency registration, 13-step location retrieval, PSAP interworking, NG-eCall MSD transfer (TS 23.167 §7.2–§7.7, Annex C)
- [procedures/IMS-emergency-access-variants.md](procedures/IMS-emergency-access-variants.md) — Access-specific emergency procedures: IP-CAN support matrix, E-UTRAN/LTE domain selection tables H.1/H.2, eCall domain rules, WLAN-to-EPC, roaming-without-NNI GIBA flow, non-3GPP-to-5GC constraints (TS 23.167 Annexes E/H/J/K/L)

## Protocols
_No pages yet._

## Interfaces
- [interfaces/reference-points.md](interfaces/reference-points.md) — All EPC reference points: S1, S5, S6a, S11, Gx, Rx, SGi and more
- [interfaces/IMS-reference-points.md](interfaces/IMS-reference-points.md) — All IMS reference points: Gm, Mw, ISC, Cx, Sh, Rx, Mi, Mj, Mr and more

## Sources
- [sources/ts23401-section4.md](sources/ts23401-section4.md) — 3GPP TS 23.401 §4: EPC architecture, network elements, EMM/ECM, bearer model
- [sources/ts23228-section4.md](sources/ts23228-section4.md) — 3GPP TS 23.228 §4: IMS architecture, CSCF roles, identity model, reference points
- [sources/ts23218-section5-6.md](sources/ts23218-section5-6.md) — 3GPP TS 23.218 §4–§6.5A: IM Call Model, SPTs, iFC evaluation, S-CSCF functional model, Transit Function
- [sources/ts23402-section4.md](sources/ts23402-section4.md) — 3GPP TS 23.402 §4–§8: Non-3GPP architecture (§4), PMIP S5/S8 3GPP procedures (§5), trusted S2a/S2c (§6), untrusted S2b/S2c + bearer management (§7), handovers (§8.1–§8.2); §9+ remaining (IN PROGRESS)
- [sources/ts23167.md](sources/ts23167.md) — 3GPP TS 23.167 v17.2.0: IMS emergency sessions — architecture, E-CSCF/LRF entities, §3–§7 procedures, NG-eCall, Annexes E/H/J/K/L access variants (COMPLETE)
- [sources/ts23216.md](sources/ts23216.md) — 3GPP TS 23.216 v16.4.0: SRVCC — §1–§5 concepts/architecture + §6.1–§6.3 E-UTRAN/HSPA procedures done; §6.4–§9 CS-to-PS/5G-SRVCC/failure pending (IN PROGRESS)

## Analyses
- [analyses/iFC-worked-examples.md](analyses/iFC-worked-examples.md) — iFC triggering chain, call forwarding (redirect/proxy), announcement/conference/transcoding B2BUA flows, voicemail deposit/playback; AS mode patterns (TS 23.218 Annexes B/C)
