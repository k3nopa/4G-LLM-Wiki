# Wiki Index
_Last updated: 2026-04-20 — 132 pages total (TS 24.301 chunks 1–4 complete)_

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
- [entities/PCRF.md](entities/PCRF.md) — Policy and Charging Rules Function: Gx/Rx, VoLTE bearer trigger; input sources, V-PCRF/H-PCRF roles, multiple BBF, MPS, PS Data Off (TS 23.203 §6.2.1)
- [entities/PCEF.md](entities/PCEF.md) — Policy and Charging Enforcement Function: SDF detection, QoS/gate enforcement, BBF (GTP mode), measurement, ADC, traffic steering (TS 23.203 §6.2.2)
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
- [entities/IBCF.md](entities/IBCF.md) — Interconnection Border Control Function: II-NNI border node, topology hiding, IMS-ALG, IOI insertion (TS 29.165)
- [entities/TrGW.md](entities/TrGW.md) — Transition Gateway: II-NNI media relay + IPv4/IPv6 NA(P)T-PT, controlled by IBCF (TS 29.165)
- [entities/MRF.md](entities/MRF.md) — MRFC + MRFP: conference/media resources (SIP Mr + H.248 Mp); tones, transcoding, Cr/Mr' interfaces (TS 23.218 §8)
- [entities/MRB.md](entities/MRB.md) — Media Resource Broker: Query mode (Rc), In-Line mode (Mr'), MRF pool management (TS 23.218 §13)
- [entities/SCC-AS.md](entities/SCC-AS.md) — Service Centralization and Continuity AS: 3pcc anchor for Access Transfer and IUT; ISC/Sh/OMA DM interfaces (TS 23.237)
- [entities/ATCF.md](entities/ATCF.md) — Access Transfer Control Function: serving-network SRVCC control plane; STN-SR, ATU-STI, ATGW control; Mw/Mx/Iq/Ix/I2 interfaces (TS 23.237)
- [entities/ATGW.md](entities/ATGW.md) — Access Transfer Gateway: media anchor before/after SRVCC; transcoding; Iq/Ix controlled by ATCF (TS 23.237)

### IMS Emergency
- [entities/E-CSCF.md](entities/E-CSCF.md) — Emergency-CSCF: routes emergency sessions to PSAP via LRF; always in serving network; interfaces Mw/Ml/Mi/Mg/Mm/Mx/I4/I6 (TS 23.167 §6.2.2)
- [entities/LRF.md](entities/LRF.md) — Location Retrieval Function: resolves UE location → PSAP routing (ESQK/ESRN/URI); consists of RDF + Location Server; Ml to E-CSCF, Sh to HSS, Le to PSAP (TS 23.167 §6.2.3)

## Concepts
- [concepts/PCC-architecture.md](concepts/PCC-architecture.md) — PCC architecture: PCRF/PCEF/BBERF/AF/TDF entities, 17 reference points, binding mechanism, requirements, roaming (TS 23.203 §4–§5)
- [concepts/QCI-characteristics.md](concepts/QCI-characteristics.md) — QCI characteristics table (Table 6.1.7-A/B): all GBR/Non-GBR/delay-critical QCIs, ARP priority/pre-emption model, VoLTE QCI assignment (TS 23.203 §6.1.7)
- [concepts/PCC-rules-reference.md](concepts/PCC-rules-reference.md) — All three PCC rule types: PCC rule (Table 6.3), QoS rule (Table 6.5), ADC rule (Table 6.8); IP-CAN/TDF/APN session policy info; usage monitoring; spending limits; traffic steering; NBIFOM routing rules (TS 23.203 §6.3–§6.12)
- [concepts/PCC-EPC-access-specifics.md](concepts/PCC-EPC-access-specifics.md) — EPC access-specific PCC: GTP (Annex A.4) vs PMIP (A.5) bearer binding, Default EPS Bearer QoS, EPS event/credit triggers, non-3GPP access (HRPD Gxa, Trusted WLAN S2a, untrusted ePDG/S2b), PCC rule precedence (TS 23.203 Annex A+H)
- [concepts/EPS-bearer.md](concepts/EPS-bearer.md) — EPS bearer model: default/dedicated, TFT, QCI, GBR/Non-GBR
- [concepts/EMM-ECM-states.md](concepts/EMM-ECM-states.md) — EMM/ECM state machines: REGISTERED/DEREGISTERED, IDLE/CONNECTED
- [concepts/IMS-identity-model.md](concepts/IMS-identity-model.md) — IMPI, IMPU, GRUU, Implicit Registration Set, service profile, iFC
- [concepts/information-storage.md](concepts/information-storage.md) — Per-node data models: HSS subscription, MME MM context, SGW/PGW bearer context, UE context; charging and Secondary RAT reporting (TS 23.401 §5.7)
- [concepts/IM-call-model.md](concepts/IM-call-model.md) — IM Call Model: S-CSCF functional architecture, SPTs, iFC evaluation algorithm, ODI tracking, Transit Function, ICSI/IARI, per-direction handling, session release, charging/ICID/IOI, subscriber data detail (TS 23.218 §4–§6.9)
- [concepts/AS-interaction-modes.md](concepts/AS-interaction-modes.md) — Application Server interaction modes: 5 modes (terminating UA, originating UA, SIP proxy, B2BUA, not involved), AS functional model (AS-ILCM/OLCM/Logic), interfaces (ISC/Sh/Dh/Cr/Rc), session handling procedures (TS 23.218 §9)
- [concepts/non-3GPP-access-architecture.md](concepts/non-3GPP-access-architecture.md) — Non-3GPP access: trusted/untrusted, IPMS, reference models/points, ANDSF policies, QoS/IPsec SA model, identities, IP allocation, detach; HSS/AAA interaction procedures (§12) and per-node information storage (§13) for non-3GPP and HRPD contexts (TS 23.402 §4, §12–§13)
- [concepts/IMS-emergency-architecture.md](concepts/IMS-emergency-architecture.md) — IMS emergency framework: 30 architectural principles, E-CSCF/LRF reference model, UE-detectable vs non-detectable flows, location principles, IP-CAN expectations, NG-eCall introduction (TS 23.167 §4–§6)
- [concepts/SRVCC.md](concepts/SRVCC.md) — Single Radio Voice Call Continuity: all 6 variants (E-UTRAN→1xCS/GERAN, vSRVCC, UTRAN(HSPA), CS-to-PS, 5G-SRVCC), architecture principles, new entities (1xCS IWS, MSC Server enhanced, MME_SRVCC), reference points (S102, Sv, N26), PCC role, emergency SRVCC (TS 23.216)
- [concepts/IMS-access-security.md](concepts/IMS-access-security.md) — IMS access security: 5 security associations, IMS AKA, IPsec ESP SA structure, confidentiality/integrity/topology-hiding mechanisms, ISIM/IMC (TS 33.203 §4–§9)
- [concepts/IMS-auth-alternatives.md](concepts/IMS-auth-alternatives.md) — IMS authentication alternatives: SIP Digest, TLS, key expansion (Annex I), RFC 3329 SIP Security header syntax (Annex H), NAT traversal (Annex M), auth scheme coexistence (Annex P) (TS 33.203)
- [concepts/charging-architecture.md](concepts/charging-architecture.md) — 3GPP charging architecture: CTF/CDF/OCF/CGF roles, Rf/Ro reference points, offline (event/session) and online (IEC/ECUR/SCUR) charging scenarios and flows (TS 32.299 §4–§5)
- [concepts/IMS-charging-architecture.md](concepts/IMS-charging-architecture.md) — IMS charging architecture and principles: offline/online/converged paradigms, ICID correlation, IOI, SDP handling, trigger conditions, RTTI, roaming, location (TS 32.260 §4–§5.1)
- [concepts/IMS-charging-information.md](concepts/IMS-charging-information.md) — IMS Information parameter reference: Service Information components, full IMS Information ~50-field table with per-node matrix, offline/online/converged operation-type matrices (TS 32.260 §6.3–§6.4)
- [concepts/Sh-user-profile-data.md](concepts/Sh-user-profile-data.md) — Sh interface data model: complete Table 7.6.1 (35 Data-References), all IE definitions, UML class model (Annex C), XML schema types (Annex D), T-ADS HSS decision algorithm (Annex E flowchart), Repository Data sharing note (Annex H) (TS 29.328 §7 + Annexes A/C/D/E/H)
- [concepts/IMS-service-continuity.md](concepts/IMS-service-continuity.md) — IMS Service Continuity: Access Transfer (PS-CS/PS-PS/CS-PS) + IUT/Collaborative Session framework, 3pcc model, STN/STI/C-MSISDN identifiers, ATCF/ATGW signalling/bearer paths (TS 23.237 §3–§5)
- [concepts/IMS-SC-charging.md](concepts/IMS-SC-charging.md) — IMS SC security and charging: no additional security requirements (§7); cohesive SCC AS CDR strategy, CS/IMS correlation, online charging IMS-only rule, accounting/settlement, ICID roaming (TS 23.237 §7–§8)
- [concepts/CUPS-architecture.md](concepts/CUPS-architecture.md) — CUPS: SGW/PGW/TDF CP+UP split, Sxa/Sxb/Sxc reference points, functional split tables, PDR/FAR/URR model, charging/buffering/policing/PCC/ADC functions, UP selection criteria (TS 23.214 §1–§5)
- [concepts/CUPS-parameter-reference.md](concepts/CUPS-parameter-reference.md) — Normative §7 attribute tables for PDR/URR/FAR/QER/Usage Report; functional description of CP rule construction at PDN/bearer/measurement-key levels; PFD management params (TS 23.214 §7)

## Procedures
- [procedures/NAS-attach.md](procedures/NAS-attach.md) — NAS EPS Attach stage-3: initiation (T3410), identity selection, common procedures, accept/reject/abnormal cases, combined attach (TS 24.301 §5.5.1)
- [procedures/NAS-detach.md](procedures/NAS-detach.md) — NAS Detach stage-3: UE-initiated (T3421, switch-off/EPS/IMSI types), network-initiated (T3422, re-attach-required), EMM cause handling (TS 24.301 §5.5.2)
- [procedures/NAS-TAU.md](procedures/NAS-TAU.md) — NAS TAU stage-3: 33-case trigger list, T3430 supervision, accept/reject/combined TAU, ISR activation (TS 24.301 §5.5.3)
- [procedures/NAS-service-request.md](procedures/NAS-service-request.md) — NAS Service Request stage-3: 17 trigger cases, T3417/T3417ext, SERVICE/CONTROL PLANE SERVICE/EXTENDED SERVICE REQUEST, reject causes, paging, NAS/Generic transport (TS 24.301 §5.6)
- [procedures/NAS-ESM-procedures.md](procedures/NAS-ESM-procedures.md) — NAS ESM stage-3: ESM sublayer states, IP allocation, network-initiated bearer procedures (default/dedicated activation, modification, deactivation), UE-requested procedures (PDN connectivity/disconnect, bearer resource allocation/modification), miscellaneous (ESM info request, notification, ProSe relay, CIoT data transport) (TS 24.301 §6)
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
- [procedures/HRPD-optimized-handover.md](procedures/HRPD-optimized-handover.md) — HRPD optimized handover (E-UTRAN ↔ CDMA2000): S101/S103 reference points, pre-registration 9-step, active HO 19-step (S103 GRE forwarding, all-zero CoA PBU), idle-mode 8-step, S101 tunnel redirection on TAU (TS 23.402 §9–§10)
- [procedures/TWAN-S2a-procedures.md](procedures/TWAN-S2a-procedures.md) — TWAN GTP/PMIPv6 S2a procedures: TWAN functional split (TWAG/TWAP), three connection modes (TSCM/SCM/MCM), WLCP, initial attach GTP/PMIP 15-step, detach, bearer management, MCM PDN connectivity/disconnection, 3GPP↔TWAN handovers (TS 23.402 §16)
- [procedures/SRVCC-from-E-UTRAN.md](procedures/SRVCC-from-E-UTRAN.md) — SRVCC from E-UTRAN: 1xCS 19-step flow (§6.1) + all 5 (v)SRVCC call flows to GERAN/UTRAN with/without PS HO and vSRVCC (§6.2) (TS 23.216)
- [procedures/SRVCC-from-UTRAN-HSPA.md](procedures/SRVCC-from-UTRAN-HSPA.md) — SRVCC from UTRAN(HSPA): 3 SGSN-based flows to GERAN/UTRAN including SRVCC CS KEYS exchange and Gn/Gp vs S4 bearer handling (§6.3) (TS 23.216)
- [procedures/PMIP-S5S8-procedures.md](procedures/PMIP-S5S8-procedures.md) — PMIP-based S5/S8 for 3GPP access: SGW-as-BBERF model, initial attach PMIP steps C.1–C.5, detach, dedicated bearer activation/modification/deactivation (SGW drives TFT), PDN connectivity, handover/TAU with SGW relocation, GTP vs PMIP differences (TS 23.402 §5)
- [procedures/IMS-emergency-session.md](procedures/IMS-emergency-session.md) — IMS emergency session: 3 paths (credentialed/anonymous/non-UE-detectable), emergency registration, 13-step location retrieval, PSAP interworking, NG-eCall MSD transfer (TS 23.167 §7.2–§7.7, Annex C)
- [procedures/IMS-emergency-access-variants.md](procedures/IMS-emergency-access-variants.md) — Access-specific emergency procedures: IP-CAN support matrix, E-UTRAN/LTE domain selection tables H.1/H.2, eCall domain rules, WLAN-to-EPC, roaming-without-NNI GIBA flow, non-3GPP-to-5GC constraints (TS 23.167 Annexes E/H/J/K/L)
- [procedures/IMS-AKA-registration.md](procedures/IMS-AKA-registration.md) — IMS AKA + IPsec SA setup during IMS registration: 3-phase flow, SA parameters, 4-port model, error cases, re-registration SA transition (TS 33.203 §6–§7)
- [procedures/IMS-SIP-registration.md](procedures/IMS-SIP-registration.md) — IMS SIP registration stage-3: full IMS AKA 2-phase REGISTER (UE/P-CSCF/S-CSCF headers, SA state transitions, 200 OK construction, deregistration) (TS 24.229 §5.1.1+§5.2.2+§5.4.1)
- [procedures/Sh-signalling-flows.md](procedures/Sh-signalling-flows.md) — Sh interface signalling flows: all 4 procedures (Sh-Pull/Update/Subs-Notif/Notif) with IE tables, HSS processing steps, AS Permissions List, SLF/Dh resolution (TS 29.328 §4–§6)
- [procedures/IMS-offline-charging-flows.md](procedures/IMS-offline-charging-flows.md) — IMS offline charging flows: CDR trigger tables, MO/MT session establishment (3 SDP scenarios), mid-session, session release, PSTN-interworking (MGCF/BGCF), multi-party (MRFC), AS redirect/voicemail, SCC AS (B2BUA/SRVCC), ATCF, IBCF, RTTI, OMR, LBO roaming, error cases (TS 32.260 §5.2)
- [procedures/IMS-online-charging-flows.md](procedures/IMS-online-charging-flows.md) — IMS online charging flows: IEC/ECUR/SCUR principles, triggering SIP methods (Tables 5.3.1.1–5.3.1.2), SCUR session establishment/mid-session/release scenarios with Mermaid flows, ECUR session-unrelated, error cases, OCS-initiated service termination; Annex B: all 9 OCS termination sequence diagrams (TS 32.260 §5.3 + Annex B)
- [procedures/IMS-converged-charging-flows.md](procedures/IMS-converged-charging-flows.md) — IMS converged charging flows: Nchf/CHF architecture, SCUR/ECUR/IEC/PEC methods, trigger tables (5.4.3.1–5.4.3.2), CHF CDR generation, Rf Charging Data Request/Response IEs (§6.1.1), Nchf offline message structure (§6.1.1a) (TS 32.260 §5.4 + §6.1.1–§6.1.2)
- [procedures/Cx-signalling-flows.md](procedures/Cx-signalling-flows.md) — Cx interface signalling flows: all 6 procedure descriptions (UAR/SAR/RTR/LIR/MAR/PPR) with full IE tables, HSS detailed behaviour, implicit registration, profile download rules, S-CSCF assignment, error handling (TS 29.228 §5–§8)
- [procedures/IMS-SC-registration.md](procedures/IMS-SC-registration.md) — IMS Service Continuity registration: 3rd-party registration to SCC AS, ATCF 11-step STN-SR flow, CS-to-PS Single Radio prerequisites, STI-rSR provisioning (TS 23.237 §6.1)
- [procedures/IMS-SC-origination-termination.md](procedures/IMS-SC-origination-termination.md) — IMS SC origination + termination: 6 origination variants (CS/PS/ATCF/fallback/CS-to-PS) and 6 termination variants with Mermaid flows; SCC AS as first/last AS in path; ATCF ATGW anchoring decision (TS 23.237 §6.2)
- [procedures/PS-CS-access-transfer.md](procedures/PS-CS-access-transfer.md) — IMS SC access transfer flows: PS→CS/CS→PS Dual Radio (with/without SSI), Single Radio SRVCC, CS→PS Single Radio with ATCF, PS-PS full/partial/early-dialog, PS-PS+PS-CS conjunction variants; Remote Leg Update and Source Access Leg Release sub-procedures (TS 23.237 §6.3.1–§6.3.2)
- [procedures/IMS-SC-media-and-services.md](procedures/IMS-SC-media-and-services.md) — IMS SC media adding/deleting (12 sub-cases: local/remote end, CS+PS and PS-only), ICS mid-call fallback (§6.3.5), operator policy/user preferences (§6.4), supplementary services with split Access Legs (§6.5: CDIV/HOLD/CONF/ECT/CW/MCID impacted; 14 services not impacted) (TS 23.237 §6.3.3–§6.5)
- [procedures/IMS-IUT-procedures.md](procedures/IMS-IUT-procedures.md) — Inter-UE Transfer: Collaborative Session establishment, media transfer/add/delete/modify, Control transfer, session release, IUT without Collaborative Session, target-initiated IUT (§6a.9), Media Flow Replication via MRF (§6a.10), Session Replication (§6a.11), User Authorization/Ut configuration (§6a.12) (TS 23.237 §6a)
- [procedures/SRVCC-emergency-procedures.md](procedures/SRVCC-emergency-procedures.md) — SRVCC/DRVCC emergency session procedures: EATF anchoring (§6c.1), active+early-dialog AT (§6c.2.1–2.2), multiple EATF instances forking/redirection (§6c.2.3–2.4), Normal/Limited Service Mode (§6c.3–4), WLAN DRVCC emergency (§6d) (TS 23.237 §6c–§6d)
- [procedures/SRVCC-enhancements.md](procedures/SRVCC-enhancements.md) — SRVCC enhancements: codec re-negotiation after SRVCC (B.2.1), codec inquiry prior to SRVCC (B.2.2), reject SRVCC on call state incompatibility (B.3) (TS 23.237 Annex B)
- [procedures/PCC-session-procedures.md](procedures/PCC-session-procedures.md) — PCC session lifecycle: IP-CAN establishment (19-step), termination, modification; PCRF discovery/DRA; Gateway Control session (BBERF); Sy spending limits; Np RUCI; Nt background data transfer; PFD management (TS 23.203 §7)
- [procedures/Sx-session-management.md](procedures/Sx-session-management.md) — Sx session establishment/modification/termination + TS 23.401/23.203/23.402/23.060 CUPS interaction flows + Sx session-level reporting procedures (TS 23.214 §6.2–§6.4)

## Protocols
- [protocols/Cx-Diameter.md](protocols/Cx-Diameter.md) — Cx/Dx Diameter protocol: 6 command pairs (UAR/SAR/LIR/MAR/RTR/PPR), result codes, session/routing rules (TS 29.229)
- [protocols/Rf-offline-charging.md](protocols/Rf-offline-charging.md) — Rf interface: Diameter offline charging — ACR/ACA message formats (Command-Code 271), event/session flows, error handling (failover, retransmit, duplicate detection, CDF timer), CDF stateless accounting model (TS 32.299 §6.1–§6.2)
- [protocols/Ro-online-charging.md](protocols/Ro-online-charging.md) — Ro interface: Diameter online charging — CCR/CCA message formats (Command-Code 272), IEC/ECUR/SCUR flows, MSCC quota management, Final-Unit-Indication, QHT/QCT/thresholds, CCFH, AVP bindings; IMS-specific Debit/Reserve Units message content (TS 32.299 §6.3–§6.5; TS 32.260 §6.2)
- [protocols/charging-AVPs.md](protocols/charging-AVPs.md) — Selective AVP reference: IETF Diameter reused AVPs, MSCC ABNF, Result-Code extensions, Service-Context-Id URNs, ICID, Node-Functionality, Role-Of-Node, PS-Information, IMS-Information, SDP-Media-Component, quota/tariff/AoC/SRVCC AVPs (TS 32.299 §7)
- [protocols/IMS-CDR-field-reference.md](protocols/IMS-CDR-field-reference.md) — IMS per-node CDR field content: 12 CDR types (S-CSCF, P-CSCF, I-CSCF, MRFC, MGCF, BGCF, SIP AS, IBCF, E-CSCF, TRF, ATCF, TF), CDR generation modes, distinctive fields per node, capability matrix (TS 32.260 §6.1.3)
- [protocols/Sh-Diameter.md](protocols/Sh-Diameter.md) — Sh interface Diameter protocol: 4 command pairs (UDR/UDA/PUR/PUA/SNR/SNA/PNR/PNA), Application-ID 16777217, ABNF message formats, result codes (5100–5108 + 4100–4101), operational flows; overload control (RFC 7683/Annex F), message priority/DRMP (RFC 7944/Annex I), load control (RFC 8583/Annex J) (TS 29.329; TS 29.328)
- [protocols/SIP-IMS-profile.md](protocols/SIP-IMS-profile.md) — SIP/SDP conformance profile for IMS: node SIP roles, URI/address assignments, security mechanism table (Table 4-1/4-2), trust domain (23 P-headers), ICID/IOI charging correlation, priority, overload control, II-NNI traversal, P-CSCF/S-CSCF restoration, RLOS (TS 24.229 §4)
- [protocols/SIP-IMS-extensions.md](protocols/SIP-IMS-extensions.md) — IMS SIP extensions: 8 new P-headers (§7.2.13–7.2.20), §7.2A parameter extensions (integrity-protected, ik/ck, mediasec, PANI/PCV access-network-info, orig, ICSI/IARI, sos, CPC/OLI), SIP timer tables (§7.7/7.8), media feature tags (§7.9), feature-capability indicators (§7.9A), reg-event/info-package extensions (TS 24.229 §7)
- [protocols/NAS-EMM-protocol.md](protocols/NAS-EMM-protocol.md) — NAS EMM stage-3: UE mode, NAS security, EMM state machines (UE + MME), elementary procedures (T3412/T3346/PSM/eDRX/SGC), common procedures (GUTI realloc/Auth/SMC/ID) (TS 24.301 §4–§5.4)
- [protocols/NAS-message-reference.md](protocols/NAS-message-reference.md) — NAS message reference: §7 error handling rules (PTI/EBI/IE errors) + all 34 EMM + 25 ESM message definitions with mandatory/optional IEs and message type code tables (TS 24.301 §7–§8)

## Interfaces
- [interfaces/reference-points.md](interfaces/reference-points.md) — All EPC reference points: S1, S5, S6a, S11, Gx, Rx, SGi and more
- [interfaces/IMS-reference-points.md](interfaces/IMS-reference-points.md) — All IMS reference points: Gm, Mw, ISC, Cx, Sh, Rx, Mi, Mj, Mr, Ici, Izi, Mx and more
- [interfaces/II-NNI.md](interfaces/II-NNI.md) — Inter-IMS Network to Network Interface: Ici/Izi reference points, IBCF/TrGW roles, traversal scenarios, SIP profile, addressing/URI rules, IOI charging, 26 MMTEL supplementary services, ICS/SRVCC/presence/messaging/OMR/IUT/overload/STIR/emergency (TS 29.165 §1–§33 + Annex A)
- [interfaces/Sx.md](interfaces/Sx.md) — Sx interface family (Sxa/Sxb/Sxc): PFCP-based CP↔UP control; session context types, message types (Establishment/Modification/Termination/Report), PDR/FAR/URR/QER parameters (TS 23.214)

## Sources
- [sources/ts23401-section4.md](sources/ts23401-section4.md) — 3GPP TS 23.401 §4: EPC architecture, network elements, EMM/ECM, bearer model
- [sources/ts23228-section4.md](sources/ts23228-section4.md) — 3GPP TS 23.228 §4: IMS architecture, CSCF roles, identity model, reference points
- [sources/ts23218-section5-6.md](sources/ts23218-section5-6.md) — 3GPP TS 23.218 §4–§6.5A: IM Call Model, SPTs, iFC evaluation, S-CSCF functional model, Transit Function
- [sources/ts23402-section4.md](sources/ts23402-section4.md) — 3GPP TS 23.402 v15.3.0: all normative sections ingested — §4–§13 non-3GPP architecture/procedures, §16 TWAN GTP/PMIP S2a, §17 E-UTRAN-HRPD SON (COMPLETE)
- [sources/ts23167.md](sources/ts23167.md) — 3GPP TS 23.167 v17.2.0: IMS emergency sessions — architecture, E-CSCF/LRF entities, §3–§7 procedures, NG-eCall, Annexes E/H/J/K/L access variants (COMPLETE)
- [sources/ts23216.md](sources/ts23216.md) — 3GPP TS 23.216 v16.4.0: SRVCC — all chunks complete (§1–§9: concepts, E-UTRAN SRVCC, UTRAN/HSPA SRVCC, all variants) (COMPLETE)
- [sources/ts33203.md](sources/ts33203.md) — 3GPP TS 33.203 v14.1.0: IMS access security — full main body §4–§9 + Annexes H/I/M/N/O/P ingested; remaining annexes (R/S/T/U/W/X) are access-specific extensions (COMPLETE for core 4G/IMS purposes)
- [sources/ts32299.md](sources/ts32299.md) — 3GPP TS 32.299 v16.2.0: Diameter charging applications — §4–§5 architecture, §6.1–§6.2 Rf offline, §6.3–§6.5 Ro online, §7 key AVPs (selective) ingested (COMPLETE)
- [sources/ts32260.md](sources/ts32260.md) — 3GPP TS 32.260 v17.3.0: IMS charging — complete (§4–§6.4 + Annex B ingested) (COMPLETE)
- [sources/ts29329.md](sources/ts29329.md) — 3GPP TS 29.329 v16.2.0: Sh Diameter protocol — all normative content ingested (§1–§6.4 + §7) (COMPLETE)
- [sources/ts29328.md](sources/ts29328.md) — 3GPP TS 29.328 v16.1.0: Sh signalling flows and message contents — complete (§3–§9 + Annexes A–J ingested) (COMPLETE)
- [sources/ts29229.md](sources/ts29229.md) — 3GPP TS 29.229 v16.2.0: Cx/Dx Diameter protocol — complete (§1–§7 ingested) (COMPLETE)
- [sources/ts29228.md](sources/ts29228.md) — 3GPP TS 29.228 v18.0.0: Cx/Dx signalling flows — all sections + Annexes A–K ingested (COMPLETE)
- [sources/ts29165.md](sources/ts29165.md) — 3GPP TS 29.165 v16.6.0: II-NNI — COMPLETE (all 3 chunks; §1–§33 + Annex A)
- [sources/ts24229.md](sources/ts24229.md) — 3GPP TS 24.229 v16.6.0: IMS SIP/SDP stage-3 — §4 General + registration procedures + §7 SIP extensions ingested (COMPLETE)
- [sources/ts23237.md](sources/ts23237.md) — 3GPP TS 23.237 v18.0.0: IMS Service Continuity — all 6 chunks complete (§3–§8, Annexes A–C ingested) (COMPLETE)
- [sources/ts23203.md](sources/ts23203.md) — 3GPP TS 23.203 v18.0.0: PCC architecture — all 5 chunks complete (§4–§7, Annex A.4/A.5, Annex H ingested) (COMPLETE)
- [sources/ts23214.md](sources/ts23214.md) — 3GPP TS 23.214 v16.1.0: CUPS architecture — ALL 5 chunks complete (§1–§7: architecture, functional splits, all Sx procedures, association management, full parameter tables) (COMPLETE)
- [sources/ts24301.md](sources/ts24301.md) — 3GPP TS 24.301 v17.6.0: NAS protocol for EPS Stage 3 — chunks 1–4 done (§4–§8: EMM state machines, all EMM+ESM procedures, error handling, all 59 message definitions)

## Analyses
- [analyses/iFC-worked-examples.md](analyses/iFC-worked-examples.md) — iFC triggering chain, call forwarding (redirect/proxy), announcement/conference/transcoding B2BUA flows, voicemail deposit/playback; AS mode patterns (TS 23.218 Annexes B/C)
