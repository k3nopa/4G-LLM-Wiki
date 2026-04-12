# Wiki Log
_Append-only event history. Parse with: `grep "^## \[" log.md | tail -10`_

---

## [2026-04-10] ingest | TS 23.402 §4.1–§4.5 — Non-3GPP Access Architecture [chunk 3b-1]

Ingested §4.1.1 (overview: trusted vs untrusted non-3GPP IP access); §4.1.2 (WiMAX interworking basics); §4.1.3 (IPMS: NBM via PMIPv6 on S2a/S2b vs HBM via DSMIPv6 on S2c or MIPv4 on S2a; final decision by HSS/AAA; 4 initial attach cases: DSMIPv6-only, MIPv4-only, neither, no capability; 6 handover cases a-f for inter-access transitions); §4.1.4 (trusted/untrusted detection: EAP auth or UE pre-configured policy; NOT a property of the access tech itself; all PDNs via same access share same trust); §4.1.5 (non-seamless WLAN offload: local IP, no EPC involvement, ePDG not required, ANDSF-governed); §4.2.1–4.2.3 (architecture reference models: non-roaming S5+S2a/S2b, non-roaming S2c/DSMIPv6, roaming home-routed S8-S2a/b chaining, roaming local breakout; all nodes: ePDG, AAA Server/Proxy, SGW as VPLMN anchor, PCRF h/v, HSS); §4.3.1.2 (trust classification is HPLMN operator/HSS-AAA policy decision); §4.3.2 (MME HRPD functions); §4.3.3.2 (SGW as local non-3GPP anchor + MAG in VPLMN); §4.3.3.3 (PGW: LMA for S2a/S2b PMIPv6, DSMIPv6 HA for S2c, MIPv4 HA, GTP); §4.3.4 (ePDG: IKEv2/IPsec termination on SWu, MAG for S2b PMIPv6, TFT uplink routing, SA per PDN or per bearer, QoS from AAA, MOBIKE, LI, charging); §4.3.5 (PCRF: hPCRF Gx/Gxa/Gxb/Gxc, vPCRF + S9); §4.4.1 (all reference points: S2a, S2b, S2c, SWu, SWa, STa, SWm, SWn, SWx, SWd, S6b, Gxa, Gxb, Gxc, S9, PMIP-S8, SGi); §4.4.2.1 (S5 requirements); §4.5.1 (PDN GW selection S2a/S2b: AAA Server returns PGW FQDN/IP+APN+PLMN; HSS gives existing PGW on handover); §4.5.2 (S2c HA discovery: PCO/IKEv2 Config Payload/DHCP/DNS); §4.5.3 (S-GW selection: only for S8-S2a/b chaining, by AAA Proxy); §4.5.4 (ePDG FQDN construction: Operator Identifier or TAI/LAI FQDN; DNS; H-ANDSF/USIM; single ePDG per UE).

Notable findings:
- Trust classification is a policy decision, not a technology property — this means the same WLAN hotspot can be trusted for one operator and untrusted for another. Architecturally significant: trusted = direct S2a to PGW, untrusted = ePDG + S2b mandatory.
- ePDG selection enforces a single-ePDG-per-UE rule — all PDN connections must go through the same ePDG instance. This simplifies routing but means ePDG failure affects all PDN connections simultaneously.
- Non-seamless WLAN offload completely bypasses EPC — distinct from trusted/untrusted non-3GPP which still connects to PGW. Three modes exist: 3GPP access, non-3GPP EPC-connected (trusted/untrusted), non-seamless WLAN (no EPC).

Next chunk: 3b-2 — TS 23.402 §7.1–§7.4 Untrusted Non-3GPP/ePDG attach procedures (PDF pp 153–172)
Pages touched: [wiki/concepts/non-3GPP-access-architecture.md (new), wiki/entities/ePDG.md (new), wiki/sources/ts23402-section4.md (new), wiki/index.md (updated)]

---

## [2026-04-10] ingest | TS 23.228 §5.7–5.11 — Termination Procedures + Session Release + Hold/Resume [chunk 2b-4]

Ingested §5.7.0 (Termination general: path fixed at registration; P-CSCF performs QoS auth; PSTN sessions use MGCF); §5.7.1 (MT#1 mobile termination roaming: 33-step Figure 5.17; S-CSCF knows P-CSCF in visited from registration; P-CSCF MPS check + AAR; gate open at step 27; UE alerts user at step 19); §5.7.2 (MT#2 mobile termination home: identical to MT#1 except P-CSCF also in home network; Figure 5.18); §5.7.2a (MT#3 CS domain roaming: user unregistered for IMS; S-CSCF service control may redirect or route to CS domain E.164; BGCF → MGCF; Figure 5.18a); §5.7.3 (PSTN-T: MGCF receives INVITE; H.248 for MGW; IAM→ACM→ANM; 16-step Figure 5.19); §5.7.4 (NI-T: external SIP client; two-phase setup—inactive media phase then re-INVITE with active media; P-CSCF gate open at phase 2 step 17; Figures 5.19b+5.19c); §5.7.5 (AS-T#1: I-CSCF direct to AS hosting PSI via HSS Cx; Figure 5.19d); §5.7.6 (AS-T#2: I-CSCF → S-CSCF → iFC → AS; Figure 5.19e); §5.7.7 (AS-T#3: I-CSCF DNS-based routing to AS for PSI subdomain; Figure 5.19f); §5.7.8 (AS-T#4: S-CSCF iFC diverts to AS that terminates session instead of forwarding to UE; Figure 5.19g); §5.7a.1 (sessions without preconditions: general — no dedicated bearer required before session active; for non-real-time QoS services); §5.7a.2 (cross-operator flow without preconditions: 32-step Figure 5.19h; P-CSCF media policy check at step 2 and 10; UE#2 may ring immediately; bearer reservation can happen during or after answering; QoS auth after 200 OK not before); §5.8.0 (routing interrogation general: I-CSCF always queries HSS for MT INVITE; SLF resolves which HSS); §5.8.1 (SLF: Dx interface I-CSCF↔SLF; Dh interface AS↔SLF; DX_SLF_QUERY/RESP; DH_SLF_QUERY/RESP; E.164 → Tel URI per RFC 3966 before DX_SLF_QUERY); §5.8.2 (SLF on register: I-CSCF or S-CSCF queries SLF; two cases); §5.8.3 (SLF on UE INVITE); §5.8.4 (SLF on AS access to HSS); §5.9 (mid-session signalling: 4 nodes must stay in path: P-CSCF-orig, S-CSCF-orig, S-CSCF-term, P-CSCF-term; I-CSCF optional); §5.10.0 (session release general: billing integrity; 4 trigger scenarios); §5.10.1 (terminal-initiated: 18-step Figure 5.22; steps 4+10 remove PCRF auth/delete bearers; steps 6+8 invoke S-CSCF service logic; bearer deletion and SIP OK parallel); §5.10.2 (PSTN-initiated: 14-step Figure 5.23; ISUP REL→MGCF BYE→S-CSCF; H.248 tear-down step 4; RLC timing depends on PSTN type); §5.10.3.0 (signalling bearer loss: try re-establish first; if fails deactivate all IMS bearers; P-CSCF notified via PCRF; reject subsequent INVITEs until re-registration or timer); §5.10.3.1.1 (P-CSCF-initiated: bearer release indication → P-CSCF decides → generates BYE; Figure 5.26, 16-step); §5.10.3.2 (S-CSCF-initiated: admin/expiry → S-CSCF generates BYE to both parties simultaneously; Figure 5.27, 16-step); §5.11.1.0 (session hold general: resources kept; only holding party can resume); §5.11.1.1 (mobile-to-mobile hold/resume: 24-step Figure 5.28; Hold re-INVITE a=inactive, gate closed; Resume re-INVITE a=sendrecv, gate re-opened).

Notable findings:
- MT#1 and MT#2 are structurally identical (33 steps) — same as MO#1/MO#2 relationship in origination; topology is the only difference (visited vs home P-CSCF)
- NI-T two-phase approach: phase 1 establishes session with inactive media (precondition-free); phase 2 activates media via re-INVITE after UE reserves bearer. This is the inverse of NI-O: in NI-O the external client originates, in NI-T the external client terminates.
- AS-T#1 direct vs AS-T#2 indirect: the difference is purely in how HSS stores the PSI routing — direct means HSS returns AS address to I-CSCF; indirect means HSS returns S-CSCF address and S-CSCF evaluates iFC
- Session Hold: bearer is NOT deleted on hold (unlike release). Gate is closed by PCRF. This allows instantaneous resume without re-establishment delay — key for UX.
- S-CSCF-initiated release (§5.10.3.2) sends BYE to both parties simultaneously (steps 2 and 8 in parallel) — this is different from terminal-initiated where only one side generates the BYE
- SLF Dx interface is mandatory by default ("the resolution mechanism shall be supported") but can be disabled in single-HSS deployments (e.g. server farm architecture)
- Sessions without preconditions (§5.7a.2): P-CSCF at step 2 still does media policy check before forwarding INVITE — this is the same check as in NI-O §5.6.4; it acts as a policy gate even without QoS preconditions

Next chunk: Phase 2 complete. Ready for Phase 3 (TS 23.218 or TS 23.402) or Phase 4 deep-dive synthesis.
Pages touched: [wiki/procedures/VoLTE-MT-call.md (new), wiki/procedures/session-release.md (new), wiki/index.md (updated)]

---

## [2026-04-10] ingest | TS 23.228 §5.5–5.6 — S-S Routing Variants + All Origination Procedures [chunk 2b-3]

Ingested §5.5.1 step-by-step text (40-step S-S#1 narrative completion, p108); §5.5.2 (S-S#2 same operator: S-CSCF#1 → local I-CSCF → HSS LIR → S-CSCF#2, Figure 5.11); §5.5.3 (S-S#3 PSTN termination same network: S-CSCF → BGCF → MGCF in same network, Figure 5.12, 28-step); §5.5.4 (S-S#4 PSTN termination different network: S-CSCF → BGCF#1 → BGCF#2 → MGCF, Figure 5.13, 37-step); §5.6.0 (Origination general rules: signalling path fixed at registration; P-CSCF always present and does QoS; PSTN-O uses MGCF as S-CSCF equivalent); §5.6.1 (MO#1 mobile origination roaming: UE in visited, 33-step Figure 5.14; P-CSCF priority insertion; S-CSCF GRUU validation + iFC; AAR at step 7; gate open at step 28); §5.6.2 (MO#2 mobile origination home: identical to MO#1 except P-CSCF/S-CSCF both in home network, Figure 5.15); §5.6.3 (PSTN-O PSTN origination: MGCF with H.248/MGW, IAM→INVITE→ACM→ANM, 16-step Figure 5.16); §5.6.4 (NI-O non-IMS external SIP client: no preconditions; P-CSCF media policy check at step 3; resource reservation post-accept, Figure 5.16a, 15-step); §5.6.5.1 (AS-O: AS originates on behalf of user/PSI; four routing options 2a/2b/2c/2d; trusted entity — no auth check; Figure 5.16b, 18-step); §5.6.5.3 (S-CSCF selection by I-CSCF for AS originating: I-CSCF Cx-LocQuery → HSS returns capabilities/S-CSCF name → I-CSCF selects → S-CSCF Cx-Put/Pull; Figure 5.16c). Observed start of §5.7.0 (Termination procedures general) and §5.7.1 (MT#1 heading) — outside scope of this chunk.

Notable findings:
- MO#1 and MO#2 are structurally identical (33 steps); the only difference is whether the P-CSCF→S-CSCF hop crosses a visited-to-home network boundary. The procedure steps are the same.
- P-CSCF gate open (step 28) is triggered by the 200 OK arrival — not by the resource reservation confirmation. Media cannot flow before the 200 OK even if the bearer exists.
- S-CSCF GRUU check (step 3 in both MO#1 and MO#2): validates that the GRUU in the Contact header belongs to the same service profile as the calling IMPU. This is the originating-side identity validation.
- NI-O: P-CSCF media policy check at step 3 is before the UE even sees the INVITE — operator can reject non-IMS sessions based on bandwidth policy without bothering the UE
- AS-O routing option 2d (AS→I-CSCF): HSS responds to Cx-LocQuery even for unregistered identities — critical for services-for-unregistered-state (e.g., voicemail deposit to powered-off UE)
- S-S#3 vs S-S#4 distinction: BGCF local policy determines whether MGCF is in same or different network. The S-CSCF always sends to its local BGCF regardless; the BGCF makes the routing decision.

Next chunk: 2b-4 — Termination (MT Call) + Session Release (TS 23.228 §5.7 + §5.10)
Pages touched: [wiki/procedures/VoLTE-MO-call.md (new), wiki/index.md (updated)]

---

## [2026-04-10] ingest | TS 23.228 §5.4.5–5.5.1 — Session Path + QoS/PCC Interaction + S-S#1 Session Flow [chunk 2b-2]

Ingested §5.4.6.3 (bearer establishment with pre-alerting: 3-phase flow before ringing); §5.4.7.0 (8 PCC interactions taxonomy: Authorize QoS Resources, Resource Reservation, Enable/Disable media flows, Revoke Authorization, IP-CAN bearer release indication, Authorization of bearer modification, Indication of bearer modification); §5.4.7.1 (Authorize QoS Resources: P-CSCF derives SDP→AAR; per-session independent authorization; IP resource limits and 5-tuple restrictions); §5.4.7.1a (Resource Reservation with PCC: UE-initiated vs network-initiated; validation: request ≤ sum authorized IP resources); §5.4.7.2 (Enable media flows: gate open; logical OR for forked sessions); §5.4.7.3 (Disable media flows: gate close on hold/call-waiting); §5.4.7.4 (Revoke authorization: STR at BYE, PCRF removes PCC rules); §5.4.7.5 (IP-CAN bearer release indication: PCEF→PCRF→P-CSCF chain; may trigger BYE or re-INVITE); §5.4.7.6 (Bearer modification authorization: PCEF validates against authorized limits); §5.4.7.7 (Indication of bearer modification: MBR→0 kbit/s triggers possible session release); §5.4.7.8 (Resource sharing for concurrent sessions: uplink/downlink tagging; emergency sessions never share; gate management); §5.4.7.9 (Priority sharing: MCPTT use case; same ARP across sessions); §5.4.8 (QoS-Assured Preconditions: 3 cases, segmented resource reservation per endpoint); §5.4.9 (Event/information distribution: SIP SUBSCRIBE/NOTIFY; Figure 5.8a UE→P-CSCF→S-CSCF→AS); §5.4.12 (PSI routing: distinct, wildcarded, subdomain-based; originating via iFC; terminating via HSS direct-AS or S-CSCF); §5.4a/Table 5.2 (Session flow taxonomy: MO#1-2, PSTN-O, AS-O, NI-O; S-S#1-4; MT#1-3, AS-T, PSTN-T, NI-T); §5.5.1/Figure 5.10 (S-S#1 different operators: 40-step flow, I-CSCF#2 as boundary, LIR to HSS#2, dual iFC evaluation).

Notable findings:
- P-CSCF is the sole Rx/N5 anchor: all IMS QoS authorization flows through P-CSCF → PCRF; no other IMS node talks to PCRF directly
- Forked session gate logic is "logical OR" — if any forked leg is active the media gate is open; this prevents blocking RTP before the first answering UE picks up
- Resource sharing (§5.4.7.8) is the mechanism that prevents double-billing/double-reservation when a UE has call-on-hold + active call sharing the same dedicated bearer; gates enforce which session's media actually flows
- QoS preconditions (§5.4.8) decouple IP-CAN bearer establishment from SIP session completion — the phone does not ring until the bearer is up; this is mandatory for VoLTE QoS guarantees
- I-CSCF#2 in S-S#1 always does an LIR: even if the UE just registered, I-CSCF has no cached state (confirmed from chunk 2b-1 finding)
- PSI wildcarded matching in HSS enables a single AS to serve an arbitrarily large number of conference/service addresses without individual HSS provisioning

Next chunk: 2b-3 — S-CSCF Routing + Origination (MO Call) (TS 23.228 §5.5–5.6)
Pages touched: [wiki/procedures/IMS-QoS-bearer.md (new), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.228 §5.1–5.3 — P-CSCF Discovery + IMS Registration + De-registration [chunk 2b-1]

Ingested §5.1.1 (P-CSCF Discovery via DHCP+DNS and PCO), §5.1.2.1–5.1.2.2 (S-CSCF assignment by I-CSCF; six selection criteria; four Cx information transfer types; cancellation triggers), §5.1.3 (I-CSCF DNS-based selection), §5.1.4 (P-CSCF routing rules: fresh DNS per REGISTER, session routing uses registration-time state), §5.1.5 (HSS subscription update push model), §5.2.1 (18 registration requirements including GRUU, implicit registration, P-CSCF access network type inclusion), §5.2.1a (Implicit Registration Set rules: atomic registration/de-registration, no individual control, S-CSCF stores all service profiles, ISIM-less UE Temporary IMPU), §5.2.2.3 (initial registration 11-step flow with Cx-Select-Pull for new S-CSCF assignment), §5.2.2.4 (re-registration 11-step flow with Cx-Query for existing S-CSCF), §5.2.2.5 (stored information table: I-CSCF holds NO state post-registration; HSS holds only S-CSCF name; S-CSCF holds full context including GRUUs), §5.3.1 (mobile-initiated de-registration via REGISTER Expires=0; Cx-Put clear vs keep S-CSCF name for unregistered state services), §5.3.2.1 (timeout-based de-registration: timer expiry at P-CSCF + S-CSCF independently; UE not notified), §5.3.2.2.1 (HSS-administrative de-registration via Cx-Deregister; 7-step through S-CSCF→P-CSCF→UE), §5.3.2.2.2 (service platform de-registration: S-CSCF-initiated, bypasses I-CSCF). Observed §5.4 (session procedures) starting at p85 — outside this chunk scope.

Notable findings:
- I-CSCF holds **no state** after registration is complete (Table 5.1) — it is a pure routing element at registration time; this is why it is always traversed again on subsequent REGISTERs (with fresh DNS, not cached state)
- P-CSCF routing rule asymmetry: REGISTER uses fresh DNS (no prior-registration state), but INVITE uses home network contact point stored during registration — this enables P-CSCF migration between registrations without breaking sessions
- Cx-Put "keep S-CSCF name" option at de-registration is critical for services-for-unregistered-state (e.g., VoLTE MT call routing to a de-registered UE that still has an S-CSCF assigned for termination handling)
- S-CSCF may optionally omit Cx-Put/Cx-Pull on re-registration (optimisation); it can detect from context whether it is a re-registration
- Implicit Registration Set: the S-CSCF stores ALL service profiles for ALL IMPUs in the set — this means a single REGISTER triggers full AS interaction (iFC evaluation) for the entire set
- P-CSCF may force UE to use a different P-CSCF on re-registration (step 2 note in §5.2.2.4) — enables P-CSCF load redistribution without service interruption
- HSS-initiated de-registration (§5.3.2.2.1) can be used to force an S-CSCF change: HSS Cx-Deregisters UE, then on re-registration I-CSCF can select a new S-CSCF with required capabilities

Next chunk: 2b-2 — Session Path + QoS/PCC Interaction (TS 23.228 §5.4.5 + §5.4.7)
Pages touched: [wiki/procedures/IMS-registration.md (new), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.401 §5.7 + §5.10 — Information Storage + PDN Connectivity [chunk 2a-5]

Ingested §5.7.1–5.7A.4 (HSS data model Table 5.7.1-1; MME MM/EPS bearer context Table 5.7.2-1 and Emergency Configuration Table 5.7.2-2; Serving GW context Table 5.7.3-1; PDN GW context Table 5.7.4-1; UE context Table 5.7.5-1; Wild Card APN rules §5.7.6; Charging §5.7A.1–5.7A.4 covering per-bearer SGW/PGW accounting, Secondary RAT Usage Data Reporting procedures) and §5.9.1–5.9.3 (Location Reporting Procedure, Location Change Reporting Procedure including Presence Reporting Area mechanics, IMSI/APN information retrieval via RCAF) and §5.10.1–5.10.4 (Multiple PDN general rules, UE Requested PDN Connectivity 16-step procedure, UE/MME Requested PDN Disconnection 10-step procedure, MME Triggered Serving GW Relocation) from TS 23.401 v15.4.0 (PDF pages 287–324). Also observed §5.11 (UE Capability Handling) beginning on p323 — not the primary target of this chunk, noted in passing.

Notable findings:
- HSS subscription data is three-tiered: UE-level → per-PDN-context (per APN) → implicit within subscription; wildcard APN allows connectivity to unlisted APNs but the default PDN context must not be wildcard
- MME stores both Subscribed UE-AMBR (from HSS) and UE-AMBR (currently enforced) as distinct fields; UE-AMBR is recalculated at each PDN connectivity change (§4.7.3)
- SGW stores PDN GW TEIDs for S5/S8 user plane in the MME context (not just SGW context) so that a new SGW can be brought up during TAU without contacting the source SGW directly
- UE context "TIN" (Temporary Identity used in Next update) controls whether the UE presents a GUTI or P-TMSI at the next TAU/RAU — key for inter-RAT reselection behaviour
- §5.10.2 PDN Connectivity procedure: SGW immediately begins buffering DL from PGW at step 3 (same pattern as Attach); releases buffered packets only after Modify Bearer Response in step 14
- Maximum APN Restriction is checked at PGW level; conflict → PGW rejects the Create Session Request; MME maintains aggregate maximum across all active PDN connections
- Notify Request (step 15) to HSS is conditional: only sent when a NEW PGW is selected for a non-handover first PDN connection; ensures HSS has accurate PGW identity for non-3GPP access handover
- MME-triggered SGW relocation (§5.10.4) uses Operation Indication NOT set in the Delete Session Request to old SGW — same pattern as S1 handover with SGW relocation — preventing double PGW teardown
- Location Change Reporting (§5.9.2): PGW controls reporting via three independent action IEs (MS Info Change, CSG Information, Presence Reporting Area); MME propagates changes via Change Notification chain MME→SGW→PGW
- Secondary RAT Usage Data (§5.7A): reported per EPS bearer at X2/S1 handover, S1 Release, Connection Suspend, Bearer Deactivation; two paths depending on whether PGW secondary RAT reporting is active

Next chunk: 2b-1 — P-CSCF Discovery + IMS Registration + De-registration (TS 23.228 §5.1–5.3)
Pages touched: [wiki/concepts/information-storage.md (new), wiki/procedures/PDN-connectivity.md (new), wiki/index.md (updated)]

---

## [2026-04-08] create | Wiki initialized

Bootstrapped the 4G EPC & IMS knowledge wiki. Created directory structure, CLAUDE.md schema, index.md, log.md, and overview.md. No sources ingested yet.

Pages touched: [CLAUDE.md, wiki/index.md, wiki/log.md, wiki/overview.md]

---

## [2026-04-08] ingest | 3GPP TS 23.228 §4 — IMS Architecture & Functional Entities

Ingested §4 of TS 23.228 v15.6.0 (PDF pages 27–66, spec pages 26–65). Covered: full IMS architecture diagram, all CSCF functional roles (P-CSCF, I-CSCF, S-CSCF), TAS/MMTEL, BGCF, MRFC/MRFP, IP-SM-GW, IMS identity model (IMPI, IMPU, GRUU P-GRUU/T-GRUU, wildcarded IMPU, implicit registration set), service profile and iFC structure (SPTs, AS modes, ODI), all IMS reference points (Gm, Mw, ISC, Cx, Sh, Rx, Mi, Mj, Mk, Mx, Mr, Mp, Ma, Mm, Ut), roaming architectures (home-routed, local breakout), Cx Diameter commands (UAR, SAR, MAR, LIR, RTR, PPR), Sh interface for AS subscriber data access.

Notable findings:
- P-CSCF is the sole AF in IMS — all QoS authorization flows through P-CSCF Rx→PCRF
- S-CSCF is the central routing engine; all sessions (orig and term) pass through it
- iFC chain with ODI allows AS invocation and return without losing dialog state
- Wildcarded IMPU allows bulk-registration of entire number ranges without per-UE signaling
- BGCF decision is binary: local breakout (Mj→MGCF) or remote PLMN (Mk→peer BGCF)
- MRFC/MRFP decomposition: control (SIP Mr) vs media (H.248 Mp) mirrors BICC/MGW pattern
- T-GRUU provides unlinkability across sessions; P-GRUU is durable for device targeting

Pages touched: [wiki/sources/ts23228-section4.md, wiki/entities/P-CSCF.md, wiki/entities/I-CSCF.md, wiki/entities/S-CSCF.md, wiki/entities/TAS.md, wiki/entities/BGCF.md, wiki/entities/MRF.md, wiki/entities/HSS.md (updated), wiki/concepts/IMS-identity-model.md, wiki/interfaces/IMS-reference-points.md, wiki/overview.md (updated), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.401 §5.5.1.1–5.5.1.2 — Intra-LTE Handover (X2 + S1) [chunk 2a-4]

Ingested §5.5.1.1 (X2-based HO general, §5.5.1.1.2 without SGW relocation, §5.5.1.1.3 with SGW relocation) and §5.5.1.2 (S1-based HO general, §5.5.1.2.2 normal 21-step procedure, §5.5.1.2.3 reject, §5.5.1.2.4 cancel) from TS 23.401 v15.4.0 (PDF pages 237–256). Also noted §5.5.2 (Inter-RAT HO) starts at p251 — not ingested in this chunk.

Notable findings:
- X2 HO without SGW relocation uses Path Switch Request/Ack + Modify Bearer; SGW sends end-marker packets on old path to source eNB immediately after switching TEIDs to target eNB
- X2 HO with SGW relocation uses Create Session Request at target SGW; the Delete Session Request to source SGW carries Operation Indication = NOT set, telling source SGW not to trigger PGW teardown (path already switched)
- S1 HO can relocate both MME and SGW; MME should NOT be relocated within MME pool area — only when UE leaves pool
- Forward Relocation Request carries full MM context + EPS Bearer Contexts from source MME to target MME; CIoT/Non-IP/SCEF bearers cannot be transferred and are released on HO success
- Indirect data forwarding (source eNB → source SGW → target SGW → target eNB) uses temporary GTP-U tunnels cleaned up by source MME timer (after Forward Relocation Complete Acknowledge)
- Source MME timer drives source resource cleanup (UE Context Release Command, Delete Session Request with Operation Indication not set); timer expiry also triggers Delete Indirect Data Forwarding Tunnel
- End-marker mechanism: after path switch, PGW/SGW sends end-marker packet(s) on old path so target eNB's PDCP reordering has a sequence terminator
- MME-initiated bearer freeze: all PDN GW initiated bearer requests received during HO are rejected with "temporarily rejected due to HO in progress"; PGW retries using guard timer after HO completion/failure
- Post-HO TAU is a subset procedure: context transfer steps skipped because target MME already has UE context from HO messages; ISR maintained if previously active

Next chunk: 2a-5 — Information Storage + PDN Connectivity (TS 23.401 §5.7 + §5.10)
Pages touched: [wiki/procedures/X2-handover.md (new), wiki/procedures/S1-handover.md (new), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.401 §5.3.8 + §5.4.1–5.4.5 — Detach + Bearer Management [chunk 2a-3]

Ingested §5.3.8.2.1–5.3.8.4 (UE-initiated Detach on E-UTRAN, UE-initiated on GERAN/UTRAN with ISR, MME-initiated, SGSN-initiated with ISR, HSS-initiated) and §5.4.1–5.4.5 (Dedicated Bearer Activation, Bearer Modification with QoS update, HSS Initiated Subscribed QoS Modification, Bearer Modification without QoS update, PDN GW Initiated Bearer Deactivation, MME Initiated Dedicated Bearer Deactivation, UE Requested Bearer Resource Modification) from TS 23.401 v15.4.0 (PDF pages 203–234).

Notable findings:
- ISR deactivation on first Delete Session Request is at SGW: it deactivates ISR context but does NOT release the CP-TEID until both MME and SGSN have sent Delete Session Requests — prevents orphaned GTP tunnels
- HSS-initiated detach uses Cancel Location (Subscription Withdrawn) path; for subscription RAT restriction changes, Insert Subscriber Data is used instead (no detach)
- Dedicated bearer activation is always PGW-initiated (PCRF push); the UE cannot directly create bearers — it can only send a bearer resource request (§5.4.5) which the network evaluates via PCRF
- LBI (Linked Bearer Identity) is mandatory on all dedicated bearer Create Bearer Requests, binding each dedicated bearer to its default bearer
- GBR↔non-GBR QCI type switching is NOT supported by bearer modification; requires deactivate + activate cycle
- ECM-IDLE UEs during PGW-initiated bearer creation: MME triggers Network Triggered SR before proceeding; if extended idle DRX UE doesn't respond, MME rejects with "temporarily not reachable" and PGW retries after next Modify Bearer Request
- §5.4.3 (bearer modification without QoS update) requires no RRC Connection Reconfiguration — TFT/PCO/APN-AMBR changes go purely via NAS Downlink/Uplink Transport
- MME-initiated dedicated bearer deactivation (§5.4.4.2) uses Delete Bearer Command (MME→SGW→PGW) rather than PGW-push path; triggered by eNB radio bearer release (resource limitation)
- If all bearers of a UE are released and UE doesn't support "Attach without PDN connectivity", MME transitions to EMM-DEREGISTERED and sends S1 Release Command

Next chunk: 2a-4 — Intra-LTE Handover X2 + S1 (TS 23.401 §5.5.1.1–5.5.1.2)
Pages touched: [wiki/procedures/detach.md (new), wiki/procedures/dedicated-bearer.md (new), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.401 §5.3.3–5.3.5 — TAU + Service Request + S1 Release [chunk 2a-2]

Ingested §5.3.3.0–§5.3.3.2/3.3/3.6 (TAU triggers, TAU with S-GW change, TAU with data forwarding, TAU without S-GW change, RAU with MME interaction) and §5.3.4.1–5.3.4.3/4A/4B (UE triggered Service Request, Delay DDN mechanism, Network Triggered Service Request with paging/extended buffering, Connection Suspend, CP CIoT EPS Optimisation data transport) and §5.3.5 (S1 Release — eNodeB-initiated and MME-initiated, bearer preservation rules) from TS 23.401 v15.4.0 (PDF pages 136–200).

Notable findings:
- TAU triggers include 14 distinct conditions; S-GW relocation decision is entirely at the MME's discretion, independent of the trigger
- ISR is never activated in same-TAU as S-GW change or MME change; requires separate RAU with same S-GW to first activate
- "Delay Downlink Packet Notification Request" (parameter D) is a feedback mechanism from MME to SGW to suppress spurious DDN when UE uplink response races ahead of Modify Bearer Request; MME adaptively sets D based on observed DDN rate
- In Network Triggered SR, SGW includes both ARP and EPS Bearer ID in DDN; Paging Policy Indication enables differentiated paging priority per APN/bearer QoS, mapped to eNodeB paging priority level
- Extended buffering (PSM/extended DRX UEs): MME indicates DL Buffering Duration to SGW so SGW doesn't send duplicate DDNs; DL Data Buffer Expiration Time stored in MM context drives "Buffered DL Data Waiting" flag in TAU Context Response
- After S1 Release, SGW retains S1-U config (TEIDs) and begins buffering DL packets immediately — same SGW buffering state as ECM-IDLE attach; enables transparent Network Triggered SR without bearer re-creation
- GBR bearer deactivation after S1 Release is cause-dependent: deactivated for radio link loss/eNodeB failure but preserved for user inactivity and Inter-RAT Redirection
- Service Gap timer starts on S1 Release to ECM-IDLE unless the release follows an MT paging or TAU without active flag

Next chunk: 2a-3 — Detach + Bearer Management (TS 23.401 §5.3.8 + §5.4.1–5.4.5)
Pages touched: [wiki/procedures/TAU.md (new), wiki/procedures/service-request.md (new), wiki/index.md (updated)]

---

## [2026-04-09] ingest | TS 23.401 §5.3.2.1 — E-UTRAN Initial Attach [chunk 2a-1]

Ingested §5.3.2.1 of TS 23.401 v15.4.0 (PDF pages 122–136). Covered the full 26-step E-UTRAN Initial Attach procedure: UE Attach Request composition (GUTI/IMSI, UE capabilities, ESM container, PDN type, PCO, security params), eNodeB→MME forwarding (Initial UE Message with TAI+ECGI), old MME/SGSN identity resolution (Identification Request/Response), IMSI request fallback (step 4), NAS authentication + IMEISV retrieval + EIR ME Identity Check (5a/5b), ciphered options retrieval (step 6), stale bearer cleanup in new MME (step 7), Update Location to HSS with ULR-Flags=Initial-Attach-Indicator (step 8), Cancel Location at old MME/SGSN (steps 9–10), subscription data download + TA validation (step 11), MME SGW/PGW selection + Create Session Request chain MME→SGW→PGW (steps 12–13), IP-CAN Session Establishment with PCRF for PCC rules (step 14), Create Session Response chain PGW→SGW→MME with PDN address + TEIDs (steps 15–16), Attach Accept delivery via S1-AP Initial Context Setup Request (step 17), RRC Connection Reconfiguration + radio bearer activation (steps 18–20), Attach Complete (steps 21–22), Modify Bearer Request path-switch to eNodeB (steps 23–24, box D for handover), and PDN GW registration at HSS (steps 25–26).

Notable findings:
- SGW **buffers all downlink packets** from step 13 until Modify Bearer Request (step 23) — this is the SGW's core ECM-IDLE buffering function applied even during initial attach
- MME sends Notify Request for "Homogeneous Support of IMS Voice over PS Sessions" **in parallel** to steps 9–24, after step 8 — asynchronous with bearer setup
- UE-AMBR = min(subscribed UE-AMBR, APN-AMBR of default APN); this enforces aggregate rate at eNodeB
- PDN address = 0.0.0.0 signals DHCPv4 negotiation post-attach; does not block bearer establishment
- For PMIP-based S5/S8, steps 7/10/13/14/15/23a/23b (GTP-specific) are replaced by procedures in TS 23.402
- Emergency Attach path skips HSS Update Location, PCRF sets emergency ARP, no Notify sent to HSS
- Maximum APN Restriction enforcement is at PGW; MME tracks the aggregate restriction value across all active bearers

Next chunk: 2a-2 — TAU + Service Request + S1 Release (TS 23.401 §5.3.3–5.3.5)
Pages touched: [wiki/procedures/EPS-attach.md (new), wiki/index.md (updated)]

---

## [2026-04-08] ingest | 3GPP TS 23.401 §4 — EPC Architecture & Network Elements

Ingested §4 of TS 23.401 v15.4.0 (spec pages 19–94, PDF pages 21–95). Covered: architecture reference models (non-roaming, roaming home-routed, roaming local breakout), all reference point definitions (S1-MME, S1-U, S3–S13, SGi, Gx, Rx, S9, etc.), high-level functions (mobility management including ISR, TAI list, MME overload/load balancing, GTP-C load/overload control, RFSP Index), all network element definitions (MME, SGW, PGW, PCRF H/V, HSS stub, SGSN, RCAF, CSS), EMM/ECM state machines with diagrams, and the EPS bearer model (default/dedicated, TFT, QCI/GBR/AMBR).

Notable findings:
- EMM and ECM states are independent — EMM-DEREGISTERED can occur in ECM-CONNECTED
- RCAF→PCRF (Np) path brings RAN congestion into policy decisions — relevant for VoLTE QoS
- PGW hosts the PCEF; Gx is the live policy channel between PGW and PCRF
- MME overload control uses S1-AP OVERLOAD START to eNodeBs (not just NAS back-off)

Pages touched: [wiki/sources/ts23401-section4.md, wiki/entities/MME.md, wiki/entities/SGW.md, wiki/entities/PGW.md, wiki/entities/PCRF.md, wiki/entities/HSS.md, wiki/concepts/EPS-bearer.md, wiki/concepts/EMM-ECM-states.md, wiki/interfaces/reference-points.md, wiki/index.md]

## [2026-04-10] ingest | TS 23.218 §4–§6.5A — IM Call Model + S-CSCF Architecture [chunk 3a-1]

Ingested §4 (abbreviations: ODI, TIC, SCIM added to glossary); §5.1.1 (service provision architecture: ISC interface to SIP AS/IM-SSF/OSA SCS/SCIM; S-CSCF as AS invocation hub); §5.1.2 (media resource provision: direct Mr vs In-Line MRB mode via Rc/Mr'/Cr); §5.1.2A (ISC gateway function: THIG + Screening + IMS-ALG + TrGw via Ix; for third-party AS or cross-network ISC); §5.1.3 (border control: IBCF on Mr/Mm/Ici; Mx interface); §5.1A (ICSI: unauthenticated=UE-provided stripped by S-CSCF; authenticated=S-CSCF-inserted after validation; usable as SPT); §5.1B (IARI: identifies terminating application; absent→default app assumed); §5.2.1 (6 SPT types: SIP Method, Registration Type, Header Presence, Header Content/Request-URI, Direction, Session Description; bar check precedes; REGISTER=always UE-originating); §5.2.2 (Filter Criteria structure: priority+AS addr+trigger point with AND/OR/NOT SPTs+default handling+service info+include-REGISTER flags); §5.2.3 (iFC evaluation algorithm: priority-ordered; lock priority list; ODI added before ISC forward; AS returns same ODI; AS 4xx→abandon lower FCs; AS denial via Record-Route not iFC; list unlocks when request exits via Mw); §5.2.4–5.2.5 (Transit Invocation Criteria: same algorithm as §5.2.3; network-configured not user-specific; Transit Function has no Registrar); §6.1.1 (S-CSCF functional model: ILCM+OLCM+Combined I/OLSM+ILSM+OLSM+Registrar+Notifier; each leg independent as proxy/redirect/UA); §6.1.2 (Transit Function: same minus Registrar); §6.2 (S-CSCF interfaces: Mw+ISC+Cx+MRB+Mr); §6.3 (registration handling: authenticate via Cx; download iFC via SAR/SAA; validate ICSI/IARI; evaluate iFC on REGISTER; third-party REGISTER to matched ASes or reg event package; apply default handling on AS failure); §6.4.1 (UE-originating registered: served user determination→bar check→ICSI validation→originating iFC chain→normal routing); §6.4.2 (UE-originating unregistered: fetch profile from HSS first; same then normal routing — does not reject); §6.5.1 (UE-terminating registered: GRUU→IMPU mapping→bar check→terminating iFC→if Request-URI changed: option a re-route/b switch to originating iFC/c continue terminating); §6.5.2 (UE-terminating unregistered: fetch from HSS; if no terminating iFC matches→REJECT — unlike originating unregistered which continues routing); §6.5A.1 (Transit Function request handling: same SPT/ODI algorithm on TIC; no subscriber context).

Notable findings:
- ODI mechanism is the key to ordered AS chaining: S-CSCF inserts a token before ISC forward; if AS strips or changes it the chain breaks. This is the handshake that ensures the request returns to the correct dialog.
- Terminating vs originating unregistered user diverge critically: originating continues to normal routing after last FC (the call goes through); terminating REJECTS if no iFC matches. This prevents unknown traffic to unregistered terminating users.
- Request-URI change at AS (§6.5.1 options a/b/c) is the most complex case: option b (switch to originating iFC) effectively repurposes the S-CSCF into an originating proxy mid-session — powerful for AS-generated calls like voicemail, call forwarding.
- iFC priority list locking is a crucial concurrency guard: once a dialog starts evaluation the list cannot change until routing exits S-CSCF. Prevents race conditions with re-registration mid-call.
- Transit Function TIC is entirely operator-provisioned (not HSS-downloaded) — operator has full control over transit service layer without HSS involvement.

Next chunk: 3a-2 — TS 23.218 §6.6–§9: AS interaction modes, subscriber data (Sh), session release interaction, charging triggers (PDF pp 28–46)
Pages touched: [wiki/concepts/IM-call-model.md (new), wiki/sources/ts23218-section5-6.md (new), wiki/index.md (updated, 36→38 pages)]

---

## [2026-04-10] ingest | TS 23.218 §6.6–§9 — Session Release, Charging, Subscriber Data, MRFC, AS Interaction Modes [chunk 3a-2]

Ingested §6.6.0 (session release intro: S-CSCF either proxies or initiates); §6.6.1 (proxying BYE: pass-through with From/To/Call-ID unchanged, route-based); §6.6.2 (initiating BYE: S-CSCF generates BYE to both UE and AS simultaneously, independent CSeq per destination; Figure 6.6.2.1); §6.7 (subscription and notification: reg event package; NOTIFY carries implicitly registered IMPUs, GRUUs, ICSIs, IARIs per registered contact; AS subscribes via SUBSCRIBE/NOTIFY dialog with S-CSCF); §6.8 (S-CSCF IMS charging: ICID from upstream P-CSCF stored by S-CSCF; IOI identifies home network; charging function addresses from HSS; originating case: S-CSCF adds ICID+charging addrs to outgoing message; IOI response from peer retained for AS contact; IP-CAN charging info removed when crossing network boundaries; terminating case same structure; CDR generated per session); §6.8A (Transit Function charging: CDR generated; charging addrs locally configured; when sending to AS: remove received IOI + insert own; when forwarding non-AS downstream: keep received ICID+IOI; response handling symmetrical); §6.9.1 (Application Server Subscription Information = complete iFC set for service profile; downloaded via Cx at registration; valid through registration lifetime or until profile changes); §6.9.2 (Filter Criteria components: AS addr; default handling SESSION_CONTINUED/SESSION_TERMINATED; trigger point/SPTs AND/OR/NOT; iFC priority—start with highest; service info opaque passed in ISC body; include-Register-req flag; include-Register-resp flag; security note: do not set include flags for AS outside trust domain); §6.9.3 (authentication data via Cx; definition TS 23.008; handling TS 33.203); §7.1 (HSS stores data for S-CSCFs via Cx; IM-SSF via Si/Sh; AS via Sh); §7.2 (HSS interfaces: Cx/SIP-CSCF protocol TS 29.228; Sh/AS-SIP protocol TS 29.328+29.329 — also supports AS iFC activate/deactivate per subscriber; CSE via MAP TS 23.278; IM-SSF via Si MAP or Sh Diameter); §8.1.1 (MRFC overview: AS in control of tone/announcement selection; AS↔MRFC via Mr direct or via S-CSCF ISC; MRFC supports offer/answer + offer/answer-with-preconditions); §8.1.2 (tones/announcements: INVITE carries info to play tone or link to Cr media control command; announcement files fetched via Cr; tone ends on BYE or expiry; MRFC auto-generates BYE on expiry); §8.1.3 (ad-hoc conference: INVITE initiates/adds/removes parties; re-INVITE for floor control; media control via Cr; offer/answer-with-preconditions supported); §8.1.4 (transcoding: INVITE with SDP; offer/answer→200 OK with resources; offer/answer-with-preconditions→183 then 200 OK with resources after PRACK); §8.2.1 (Mr: MRFC↔S-CSCF/AS SIP; used for media control channel with MRF or between MRF and MRB); §8.2.2 (Cr: AS↔MRFC; media control protocol + resource fetch; established via SIP on Mr'/ISC; protocol specs TS 24.229/24.147/24.247); §8.2.3 (Mr': direct AS↔MRFC without S-CSCF); §8.2.4 (MRB↔MRFC Mr': In-Line mode; also media control channel MRFC→MRB); §8.2.5 (MRB↔MRFC Cr: MRFC publishes resource info to MRB); §9.1.0 (AS functional model: AS-ILCM stores transaction state + optional session state; AS-OLCM same; Application Logic provides services + coordinates ILCM/OLCM; AS accesses HSS via Sh/Si); §9.1.1 (5 modes: (1) terminating UA/redirect server—S-CSCF proxies to AS, AS terminates or 3xx; (2) originating UA—AS generates new SIP request to S-CSCF, S-CSCF proxies; (3) SIP proxy—S-CSCF proxies to AS, AS modifies headers, AS returns to S-CSCF with ODI intact; (4) routing B2BUA—AS terminates incoming dialog, generates new dialog to S-CSCF; initiating B2BUA—AS generates both dialogs; (5) not involved—S-CSCF proxies direct; AS removes itself from Record-Route); §9.1.2 (same modes apply between AS and Transit Function); §9.2 (AS interfaces: ISC/S-CSCF SIP; Sh/HSS Diameter user profile + iFC activate/deactivate; Dh/SLF HSS address resolution; Cr/MRFC media control; Rc/MRB Query mode media resource request; ISC-via-Transit = Mf reference point); §9.3 (AS subscriber data: service key, STP, service scripts CGI/CPL/Java); §9.4.1 (UE-originating: AS-ILCM reports to Logic; Logic may instruct AS-OLCM to modify request; B2BUA correlates dialog IDs; ICSI insertion rules per mode); §9.4.2 (UE-terminating: same as originating); §9.4.3 (SIP registration: third-party REGISTER carries IMPU/S-CSCF addr/expiry/IMSI/body; AS can subscribe to reg event package for NOTIFY of implicit set/GRUUs/ICSIs/IARIs; subscription info from HSS via Sh/Si in two ways: manual provisioning or automatic via Sh); §9.4.4 (session release: sub-mode A—AS receives BYE as UA/B2BUA → 200 OK; sub-mode B—AS proxies BYE downstream; sub-mode C—AS initiates BYE on all managed dialogs simultaneously); §9.4.5 (AS charging: receives ICID/IOI/charging addrs via ISC; must pass in outgoing; originating UA mode: AS generates own ICID if none received; charging addrs from Sh as fallback; ISC precedence over Sh on conflict); §10-12 (IM-SSF→TS23.278; OSA-SCS→TR29.998; Charging Server ECF/SCF→TS32.240/32.260; all implement generic §9 SIP AS behaviour).

Notable findings:
- The 5 AS modes can be mixed within a single session lifetime — an AS may start as proxy, then switch to B2BUA (e.g. after detecting a trigger condition). This is explicitly permitted by the spec.
- AS-ILCM/OLCM correlation: in routing B2BUA mode, the AS is entirely responsible for correlating dialog #1 (from S-CSCF) with dialog #2 (new to S-CSCF). The S-CSCF has no knowledge of this correlation — it sees two independent dialogs.
- Sh interface iFC control: an AS can use Sh to activate/deactivate its own iFC entries per subscriber. This is dynamic service management without requiring HSS re-provisioning — powerful for subscription-based service enable/disable.
- Include-Register-Request/Response flags: security implication is non-obvious — if set for an untrusted AS, the S-CSCF would forward raw REGISTER content (including sensitive headers) to that AS. The spec explicitly warns this should not be done.
- IOI replacement rule (§6.8, §6.8A): when S-CSCF or Transit Function sends to an AS, it removes the received IOI and inserts its own. This is the mechanism for per-hop operator accounting. The original IOI is not preserved end-to-end through AS chains.
- MRFC Mr vs Mr': Mr goes via S-CSCF (S-CSCF is in the signalling path, sees the session); Mr' bypasses S-CSCF entirely (S-CSCF has no knowledge of the media resource leg). For transcoding and conferencing where S-CSCF involvement in media signalling is not needed, Mr' is preferred.

Next chunk: 3a-3 — TS 23.218 §13 + Annex B + C: MRB procedures (In-Line/Query modes), iFC worked examples, ICSI/IARI registration (PDF pp 47–75)
Pages touched: [wiki/concepts/AS-interaction-modes.md (new), wiki/concepts/IM-call-model.md (updated §14–17), wiki/entities/MRF.md (updated §8 detail), wiki/sources/ts23218-section5-6.md (updated), wiki/index.md (updated, 38→39 pages)]

---

## [2026-04-10] ingest | TS 23.218 §13 + Annexes A/B/C — MRB, Worked Examples, iFC Triggering [chunk 3a-3]

Ingested §13.0 (MRB general: pools heterogeneous MRF resources; shares across multiple apps; selection criteria: resource characteristics, app identity, SLA/QoS, capacity, future reservations, visited-network MRB; two modes: Query and In-Line; MRB can operate in both simultaneously; AS can use Query for some calls, In-Line for others); §13.1 (Query mode: AS queries MRB via Rc with required MRF attributes; MRB responds with MRFC addresses; AS establishes dialog with MRFC via S-CSCF or Mr' direct; AS notifies MRB when done; MRB returns resource to pool; control packages over Cr); §13.2 (In-Line mode: MRB in SIP path; AS sends to MRB via Mr' or S-CSCF ISC+Mr; MRB selects MRFC and forwards; subsequent messages traverse MRB + S-CSCF if applicable; Cr direct AS↔MRFC bypassing MRB; MRB infers release from BYE); §13.3 (MRB knowledge: available resources/attributes, fair-share rules, capacity models, future reservations; acquired via O&M interfaces or direct MRB-MRFC interface); Annex A (scalability: any number of S-CSCFs/ASes; signaling delay concern; AS-as-gateway to external ASes; feature interaction priority/order concern); Annex B.1.0 (CFonCLI service description: forward based on calling CLI; AS types: SIP AS, OSA AS, CSE); Annex B.1.3 (UE-redirect 17-step: AS as redirect server, 302 Moved Temporarily, UE re-issues INVITE to UE3); Annex B.1.4 (S-CSCF redirect via proxy mode: AS sends 181 "Call Is Being Forwarded" to all parties in path, modifies Request-URI, S-CSCF routes INVITE to UE3 home net); Annex B.2.1 (announcement 30-step: AS B2BUA, initial outgoing call fails at S-CSCF, AS falls back to MRFC announcement, Call-ID 3 established with MRFC, ACK at step 26 triggers play; SDP-M from MRFC used in 183 to UE); Annex B.2.2 (ad-hoc conference 41-step: AS B2BUA, three parties UE1/UE2/UE3, same conference identifier reused in all MRFC INVITEs, Call-IDs 2/4/6 to MRFC, Call-IDs 3/5/1 to UEs, paths established to MRFP); Annex B.2.3 (transcoding variant 1 53-step: called UA 606 with codec, AS invokes MRFC three times—Call-IDs 3/4/5—one for called UA codec side, one for calling UE codec side; variant 2 31-step: called UA 606 no SDP, AS queries MRFC for codec list via INVITE-no-SDP, MRFC returns 183 with MRF SDP, AS sends codec list to called UA); Annex B.3.1 (voicemail out-of-coverage 23-step: unregistered terminating iFC, S-CSCF proxies to voicemail AS as terminating UA, PRACK+preconditions+UPDATE+200 OK, caller leaves message, BYE proxied); Annex B.3.2 (voicemail on-registration 34-step: third-party REGISTER trigger, AS as originating UA, downloads subscriber profile via Sh, detects messages, generates INVITE to UE, 34-step session establishment with preconditions); Annex C (iFC triggering example: two ASes, FC-X→AS1 FC-Y→AS2, S-CSCF evaluates FC-X, forwards to AS1, AS1 returns, S-CSCF evaluates FC-Y, forwards to AS2, AS2 returns, S-CSCF routes to destination; confirms re-evaluation after each AS return); Annex D (change history, informative, not ingested).

Notable findings:
- MRB Query mode vs In-Line mode is a fundamental design choice: Query mode gives AS full resource lifecycle control (important for services that need to pre-allocate resources); In-Line mode is simpler for AS (transparent resource selection) but puts MRB in the critical path for all in-dialog SIP.
- Ad-hoc conference uses same conference identifier across all MRFC INVITEs — this is the key mechanism by which MRFC knows all parties belong to the same conference bridge. The first INVITE creates the bridge; subsequent ones add to it.
- B2BUA announcement: the AS must handle a failure on Call-ID 2 (step 5-7) and pivot to Call-ID 3 (MRFC). This shows real-world B2BUA service logic: try outgoing call, detect failure, apply fallback service. The pivot is transparent to UE1 which only ever sees Call-ID 1.
- Voicemail on-registration (B.3.2) is the cleanest example of originating UA mode: no incoming SIP request; pure event-driven origination triggered by third-party REGISTER. AS downloads subscriber profile via Sh to discover waiting messages — this is the Sh interface's most important use case.
- Transcoding: AS must establish separate MRFC dialogs for EACH side of the transcoding session (one for calling UE's codec, one for called UA's codec). MRFP connects both RTP streams. Total: 5 Call-IDs in the 53-step flow.
- Annex C confirms ODI re-evaluation behavior: after AS1 returns the request to S-CSCF, S-CSCF re-checks all remaining FCs (not just FC-Y). This means an AS returning a modified request can potentially trigger a different FC that wouldn't have fired on the original request.

Next chunk: Phase 3b — TS 23.402 Non-3GPP access (~3 chunks, chunk tables to be defined from TOC)
Pages touched: [wiki/entities/MRB.md (new), wiki/analyses/iFC-worked-examples.md (new), wiki/sources/ts23218-section5-6.md (updated — fully ingested), wiki/index.md (updated, 39→41 pages)]

---

## [2026-04-10] ingest | TS 23.402 §4.1–§4.5 — Non-3GPP Architecture, Reference Models, ePDG Selection [chunk 3b-1]

_Note: This log entry was reconstructed after session interruption. Pages were written but log was not updated._

Ingested §4.1.2 (WiMAX interworking); §4.1.3 IPMS (NBM=PMIPv6/S2a/S2b vs HBM=DSMIPv6/MIPv4/S2c; final decision by HSS/AAA; 4 initial attach cases; 6 handover cases); §4.1.4 (trust not a property of access technology; EAP result or pre-configured policy determines trust); §4.1.5 (non-seamless WLAN offload: local IP, no EPC involvement, no ePDG); §4.2 (4 reference model variants: non-roaming S5+S2a/S2b, non-roaming S2c, roaming home-routed with PMIP-S8 chaining, roaming local breakout); §4.3.1–4.3.5 (network element roles: ePDG as IKEv2/IPsec termination + MAG; PGW as LMA+HA+MIPv4-FA; SGW as VPLMN non-3GPP anchor; PCRF h/v with Gxa/Gxb/Gxc/S9); §4.4 (all non-3GPP reference points: S2a/S2b/S2c/SWu/SWa/STa/SWm/SWn/SWx/SWd/S6b/Gxa/Gxb/Gxc/S9/PMIP-S8); §4.5.1 PDN GW selection S2a/S2b (AAA returns PGW FQDN; HSS provides existing PGW on HO); §4.5.1a eHRPD SIPTO; §4.5.2 S2c (PCO/IKEv2/DHCP/DNS); §4.5.3 S-GW selection (AAA Proxy only); §4.5.4 ePDG selection (Operator ID FQDN, TAI/LAI FQDN, DNS; H-ANDSF/USIM config; single ePDG per UE).

Notable findings:
- Trust classification applies to all PDN connections via a given access — no per-PDN override
- IPMS is entirely separate from GTP vs PMIP selection on S5/S8 — two independent protocol choices
- SGW as local non-3GPP anchor in roaming: performs MAG role for PMIP-S8, same SGW serves both S2a/S2b and S8

Next chunk: 3b-2 — TS 23.402 §4.5.5–§4.13 + §7.1–§7.5: remaining §4 concepts (ANDSF, QoS, identities, IP allocation) + untrusted non-3GPP attach/detach procedures
Pages touched: [wiki/concepts/non-3GPP-access-architecture.md (new), wiki/entities/ePDG.md (new), wiki/sources/ts23402-section4.md (new), wiki/index.md (updated, 41→44 pages)]

---

## [2026-04-10] ingest | TS 23.402 §4.5.5–§4.13 + §7.1–§7.5 — ANDSF, QoS, Identities, S2b/S2c Procedures [chunk 3b-2]

Ingested §4.5.5 (PCRF selection for non-3GPP: Gx=PGW, Gxa=TWAN, Gxc=SGW as anchor, Gxb=ePDG not fully specified Rel-15); §4.5.6 (DSMIPv6 Home Link Detection); §4.5.7 (IMS Emergency Session over WLAN: untrusted uses §7.2.5, trusted uses §6.2.1a; Emergency Config Data at ePDG/TWAG; PDN GW "currently in use for emergency" reported to HSS); §4.5.8 (APN congestion for eHRPD); §4.5.9 (GTP-C load/overload control on S2a/S2b: PGW sends to TWAN/ePDG; ePDG may apply back-off timer); §4.6.1 (NAI RFC 4282; IMSI username; IMEI for emergency or unauthenticated); §4.6.2 (EPS Bearer ID on GTP S2b/S2a: ePDG/TWAN allocated; independent namespace from S5/S8; may overlap; MAPCON: same bearer ID designates distinct TFAs simultaneously); §4.7.1 (PMIP S5/S8 IPv4 DHCPv4 via SGW relay, PGW server, 14-step DHCP flow; IPv6 prefix via Router Advertisement after PBA; deferred IPv4 via DHCPv4 procedure indication in PBA); §4.7.2 (Trusted S2a: MAG acts as DHCPv4/v6 relay; 8-step DHCP flow; static IP from HSS/AAA via access authentication); §4.7.3 (Untrusted S2b dual IP: local IP from WLAN for IPsec SWu outer; PDN IP(s) from PGW via PBA/Create Session Response delivered by ePDG via IKEv2 Config Payload; static IP possible via HSS/AAA); §4.7.4 (S2c: CoA from ePDG or access; HNP from PGW during IKEv2 bootstrapping; HoA autoconfigured from HNP; optional IPv4 HoA via RFC 5555); §4.7.5–4.7.6 (DHCPv6 PD on S2c via RFC 6276; PMIP S5/S8 via RFC 7148 with DMNP option, SGW as relay); §4.8.0 (ANDSF general principles: HPLMN/VPLMN scope; ANDSF vs RAN rules co-existence); §4.8.1 (H-ANDSF/V-ANDSF architecture; S14 interface UE↔ANDSF; pull and push; optional); §4.8.2.1.2 (ISMP: per-rule: validity conditions, prioritized access list, rule priority; single-radio only); §4.8.2.1.3 (access network discovery information: access type, SSID, carrier frequencies, validity); §4.8.2.1.4 (ISRP: per IFOM/MAPCON; rules for IFOM/MAPCON/NSWO; IP traffic filter matching; priority evaluation); §4.8.2.1.5 (IARP: per-APN routing including NSWO; IARP evaluated before ISRP in priority order); §4.8.2.1.6 (WLANSP: HS2.0 criteria, PreferredSSIDList, HomeNetwork flag); §4.8.2.1.7 (VPLMNs with preferred WLAN Selection Rules); §4.8.2.1.9–10 (Home/Visited Network Preferences: EHSP, PSPL, S2a connectivity preference, prefer 3GPP RPLMN); §4.8.2a (active rule selection: IARP from HPLMN always; ISMP/ISRP/WLANSP: prefer VPLMN if in preferred-VPLMN list, else prefer HPLMN); §4.8.2b (3-step WLAN selection procedure: build prioritized list by WLANSP criteria → S2a preference → NAI construction for auth); §4.8.3 (S14 reference point); §4.8.6 (RAN Assistance Information: 3GPP/WLAN thresholds, OPI bitmap; OPI condition via bitwise AND); §4.8.7 (LWA/LWIP/RCLWI co-existence rules); §4.9 (EAP access auth on SWa/STa; tunnel auth IKEv2/SWu; EAP re-auth RFC 6696); §4.10.1 (QoS: packet filters + QCI/ARP/MBR/GBR; Gxa/Gxb/Gxc same granularity as Gx); §4.10.3 (PMIP S5/S8 EPS bearer: Radio Bearer + S1 bearer + GRE/PMIPv6 S5 per-PDN tunnel); §4.10.4 (PCC: dynamic via PCRF or static from AAA; BCM indication to UE); §4.10.5 (GTP S2b: single IPsec SA per PDN — TFT routing; single IPsec SA per bearer — 1:1 SA↔bearer, QCI in IKEv2, TSi/TSr NOT used for routing); §4.11 (charging notes); §4.12 (multiple PDN: at most 1 3GPP + 1 non-3GPP; same APN→same PGW); §4.13 (detach: all accesses independently; preserve via HO); §7.1 (protocol stacks: PMIPv6 and GTP variants for S2b; DSMIPv6 for S2c); §7.2.1 (Initial Attach PMIPv6 9-step detailed); §7.2.4 (Initial Attach GTP variant); §7.2.5 (emergency attach GTP: Emergency Config Data, IMEI identity, no subscription); §7.3 (S2c initial attach: access auth → IKEv2 → IPsec → MIPv6 SA → Binding Update → IP-CAN session → Binding Ack); §7.4.1.1 (UE/ePDG detach PMIPv6: IKEv2 release → PBU lifetime=0 → PBA lifetime=0); §7.4.2.1 (HSS/AAA detach PMIPv6); §7.4.3.1 (UE/ePDG detach GTP: Delete Session Request with Linked EPS Bearer ID + WLAN location); §7.4.4.1 (HSS/AAA detach GTP); §7.5 (S2c detach: UE-initiated BU lifetime=0; HSS/AAA-initiated; PGW-initiated; all end with IKEv2 SA + IPsec tunnel termination).

Notable findings:
- The ANDSF ISMP/ISRP rule selection when roaming: UE applies VPLMN rules (not HPLMN) only if the VPLMN is in the "VPLMNs with preferred WLAN Selection Rules" list. Otherwise HPLMN rules always win. This is a nuanced precedence that affects roaming WLAN behavior.
- The EPS Bearer ID namespace independence on S2b means that in MAPCON scenarios, the ePDG bearer ID space and MME bearer ID space are entirely separate. This allows the same numerical ID to be used in both S2b and S5/S8 simultaneously without conflict.
- GTP S2b carries WLAN Location Information (UE local IP + WLAN access network info) from ePDG to PGW in Delete Session Request. This is the only way the EPC learns the physical WLAN location for charging/policy — the ePDG is the sole location anchor for untrusted access.
- IPsec SA per bearer mode: QCI/GBR/MBR are conveyed in IKEv2 signaling — IKEv2 carries EPS bearer QoS info. This means the UE knows the QoS class of each bearer without S1AP signaling. Elegant but non-obvious.
- Emergency attach bypasses subscription entirely — ePDG uses static Emergency Configuration Data. This is the only procedure in the EPC where the PGW does not interact with subscriber data from AAA (APN comes from Emergency Config, not subscription).

Next chunk: 3b-3 — TS 23.402 §8.1–§8.2 + §6.1–§6.2 (partial): handovers between 3GPP and non-3GPP, trusted non-3GPP attach overview
Pages touched: [wiki/concepts/non-3GPP-access-architecture.md (updated §4.6–§4.13), wiki/procedures/S2b-attach.md (new), wiki/sources/ts23402-section4.md (updated), wiki/index.md (updated, 44→45 pages)]

---

## [2026-04-10] ingest | TS 23.402 §7.6–§7.11 + §8.1–§8.2 — Additional PDN, Bearer Management, Non-3GPP↔3GPP Handovers [chunk 3b-3]

Ingested §7.6 (additional PDN connectivity: same ePDG; PMIPv6/GTP/S2c variants each repeat initial attach for new APN; IPMS applies independently per PDN; same-APN → same PGW constraint); §7.8 (S2c bootstrapping via DSMIPv6 Home Link Detection); §7.9.1 (PGW-initiated resource deactivation PMIPv6: PCRF → Binding Revocation Indication → ePDG → IKEv2 SA delete → Binding Revocation Ack → AAA update); §7.9.2 (PGW-initiated resource deactivation GTP: PCRF → PGW Delete Bearer Request(EPS Bearer ID + Linked EPS Bearer ID) → ePDG IKEv2 INFORMATIONAL DEL_SPI → non-3GPP resource release → Delete Bearer Response with WLAN Location Info); §7.10 (Dedicated S2b Bearer Activation GTP — 6-step: PCRF PCC → PGW Create Bearer Request(EPS Bearer QoS, TFT, PGW U-plane TEID, Linked EPS Bearer ID) → ePDG CREATE_CHILD_SA with Notify EPS_BEARER_INFO(TFT + QoS) → UE SA response → ePDG Create Bearer Response(EPS Bearer ID, ePDG TEID, WLAN Location); single-SA-per-PDN mode: no new Child SA, TFT mapping only); §7.11.1 (PGW-initiated bearer modification: PCRF → PGW Update Bearer Request(EPS Bearer QoS, TFT) → ePDG IKEv2 INFORMATIONAL(EPS_BEARER_INFO) → UE response → ePDG Update Bearer Response + WLAN Location); §7.11.2 (HSS-initiated subscribed QoS modification: HSS User Profile Update → AAA Notify → ePDG Modify Bearer Command → PGW PCEF IP-CAN Modification → PGW Update Bearer Request → ePDG Update Bearer Response); §8.1 (handover general: multi-PDN — one PDN per APN on initial Attach(HO); remaining via UE-requested PDN Connectivity; PGW reuse via HSS identity return; no simultaneous multi-access); §8.2.1.1 (non-3GPP→E-UTRAN GTP 18-step: Attach(HO) → Auth → Location Update(HSS returns PGW identity) → Create Session Request(HO Indication) → optional PCEF IP-CAN Modification(access change) → Create Session Response(same IP + Charging ID) → Radio/Bearer setup → Modify Bearer Request(eNB, HO Indication) → PGW switches downlink from ePDG to SGW; deferred PCC applied at Modify Bearer → UE active on E-UTRAN → remaining PDNs via §5.10 → PGW deactivates non-3GPP session per §7.9); §8.2.1.2 (PMIP S5/S8 variant: Alt A GW Control Session + PBU + PCEF Modification; Alt B lower jitter — PBU then PCEF Modification after PBA); §8.2.1.3 (UTRAN/GERAN variant: SGSN instead of MME; Activate PDP Context; S4-based GTP).

Notable findings:
- PGW-initiated resource deactivation (§7.9) is the mechanism by which the non-3GPP side is torn down after handover to E-UTRAN (§8.2.1.1 step 18). The handover and deactivation are separate procedure invocations — PGW first establishes the new 3GPP path, then initiates deactivation of the old non-3GPP path. This ensures no packet loss window.
- Dedicated bearer activation on GTP S2b (§7.10) is a direct analog of S5/S8 dedicated bearer activation (TS 23.401 §5.4.1), but the QoS delivery mechanism differs: on S5/S8, QoS reaches UE via S1-AP/NAS; on S2b, QoS is conveyed via IKEv2 Notify payloads. The UE sees identical bearer semantics but through a different signaling channel.
- HSS-initiated QoS modification (§7.11.2) exposes a clean AAA→ePDG→PGW path that differs from the 3GPP HSS→MME→PGW path. The ePDG acts as a GTPv2-C proxy for HSS subscription changes — this is an architectural consequence of keeping the non-3GPP access node (ePDG) as the anchor point for S2b bearer management.
- Alt B in PMIP S5/S8 handover variant (§8.2.1.2) demonstrates a deliberate latency optimization: by separating the PMIPv6 PBU/PBA exchange from the PCRF round-trip, the packet forwarding path can be established ~1 PCRF RTT earlier. In VoLTE handover scenarios where PCRF adds tens of ms latency, this is operationally significant.
- The 18-step non-3GPP→E-UTRAN flow requires HSS to store the PGW identity — this state must be written by PGW during initial non-3GPP attach and read by MME during handover. The HSS is the only node with global visibility across both access types. Without this, IP address continuity across access type change would be impossible.

Next chunk: 3b-4 — TS 23.402 §6.1–§6.2: trusted non-3GPP access (TWAN) architecture and initial attach procedures (PDF pp ~80–120)
Pages touched: [wiki/procedures/non3GPP-handover.md (new), wiki/procedures/S2b-attach.md (updated §7.6/§7.9/§7.10/§7.11), wiki/sources/ts23402-section4.md (updated), wiki/index.md (updated, 42→43 pages)]

## [2026-04-10] ingest | TS 23.402 §6.1–§6.8 — Trusted Non-3GPP IP Access Procedures [chunk 3b-4]

Ingested §6.1 (protocol stacks: PMIPv6 on S2a — MAG in trusted access + LMA in PGW + GRE user plane; MIPv4 FACoA on S2a — FA in trusted access + HA in PGW + MIPv4/UDP; DSMIPv6 on S2c — UE=MN + PGW=HA + IKEv2-protected); §6.2.1 (Initial Attach PMIPv6 S2a 11-step: non-3GPP L2 procs → EAP auth via STa → L3 Attach Trigger → GW Control Session Establishment(BBERF Gxa) → PBU(MN-NAI,Lifetime,AT-Type,HO-Indicator,APN,GRE key) → IP-CAN Session Establishment → Update PDN GW Address(HSS) → PBA(UE Addr,GRE uplink,Charging ID) → PMIP tunnel → QoS Rules Provision → L3 Attach Completion; roaming: vPCRF in path; non-roaming: vPCRF absent); §6.2.2 (Void); §6.2.3 (Initial Attach MIPv4 FACoA S2a 14-step: non-3GPP procs → EAP auth → Agent Solicitation/Advertisement(CoA) → RRQ to FA → GW Control Session → FA relays RRQ to PGW → AAA → Update PDN GW Address → RRP(home addr) → MIPv4 tunnel); §6.2.4 (Chained PMIP S8-S2a/b roaming 8-step: AAA provides SGW+PGW selection; MAG PBU→SGW→PGW; PGW IP-CAN session; two chained PMIP tunnels MAG↔SGW↔PGW); §6.3 (Initial Attach DSMIPv6 S2c over trusted: Module A local IP + GW Control Session; Module B IKEv2 SA with PGW(HA) + HNP allocation; Module C BU(HoA,CoA) → IP-CAN Session → BA + QoS provision); §6.4.1.1 (UE/TNAN-initiated PMIPv6 detach 7-step: access trigger → GW Control Session Term → PBU(lifetime=0) → Update PDN GW Address → PCEF IP-CAN Term → PBA(lifetime=0) → resource release); §6.4.1.2 (Chained S8-S2a detach: two PBUs/PBAs via SGW); §6.4.2.1–6.4.2.2 (HSS/AAA-initiated PMIPv6: UE De-registration Request → detach procedures → Detach Ack; PDN GW notified but MAG owns tunnel teardown); §6.4.3 (UE-initiated MIPv4 FACoA: RRQ(lifetime=0) → relay → AAA → Update PDN GW Address → PCEF term → RRP(lifetime=0)); §6.4.4 (Network-initiated MIPv4 FACoA: Registration Revocation → PCEF term → Registration Revocation Ack); §6.4.5 (HSS/AAA-initiated MIPv4 FACoA); §6.5.2 (UE-initiated S2c: BU(lifetime=0) → PCEF term → BA → PCRF GW Control Term → IKEv2 SA Term); §6.5.3 (HSS/AAA-initiated S2c: Session Term → Detach Request → Ack → PCEF term → IKEv2 SA Term; implicit omits signalling to UE); §6.5.4 (PDN GW-initiated S2c: Detach Request → PCEF term → IKEv2 SA Term); §6.6.1 (Dynamic PCC S2a: PCRF GW Control+QoS Rules Provision → TNAN enforces → PCC Rules Provision to PCEF); §6.6.2 (Dynamic PCC S2c: same pattern); §6.7 (UE-initiated resource request/release: IP-CAN specific → TNAN reports to PCRF → PCRF decision → TNAN enforces → PCEF update); §6.8.1 (Additional PDN S2a PMIPv6: same as §6.2.1 per new APN; PDN Connection Identity for multiple PDNs per APN; also re-establishment after 3GPP→non-3GPP HO with Handover Indicator).

Notable findings:
- Trusted non-3GPP access has NO IKEv2/IPsec on S2a — the trusted access network itself acts as MAG/FA. This means the operator-controlled access gateway directly manages PMIP bindings with PGW. The security model is entirely different from untrusted: trust classification shifts the encryption boundary to S2a protocol (no additional UE-side tunnel required).
- The chained PMIP S8-S2a roaming case (§6.2.4) uses the SAME procedure figure for both S2a and S2b chaining (Figure 6.2.4-1). The SGW acts as PMIP intermediate anchor for both trusted and untrusted non-3GPP in roaming. This is an elegant symmetry — the SGW-based chaining model is access-type agnostic.
- MIPv4 FACoA initial attach (§6.2.3) exposes an important difference from PMIPv6: the PGW (HA) selects the FA address from the PGW's perspective — the HA Identity in RRP tells UE where the Home Agent is. In PMIPv6, UE has no involvement in MAG-LMA binding. In MIPv4, UE explicitly registers via FA. This creates a much larger UE-visible state in MIPv4 mode.
- HSS/AAA-initiated detach for PMIPv6 (§6.4.2.1) contains a non-obvious constraint: the PDN GW acknowledges the detach indication from HSS/AAA but does NOT unilaterally tear down the PMIP tunnel. Only the MAG (trusted access) is authorized to send PBU(lifetime=0). This prevents race conditions where both MAG and PGW try to tear down the tunnel simultaneously.
- S2c over trusted access (§6.3, §6.5) shows a curious hybrid: the trusted access provides the CoA (local IP), but the UE independently manages the binding with the PGW. This means the access network has no knowledge of the HoA or binding lifetime. PCC rules still flow to the access via PCRF (§6.6.2), but the trusted access only sees traffic with the CoA as endpoint — it cannot directly derive QoS enforcement from the HoA binding.

Next chunk: 3b-5 — TS 23.402 §5 (PMIP-based S5/S8 procedures for 3GPP accesses): initial E-UTRAN attach with PMIP S5/S8, dedicated bearers, TAU/handover PMIP variants (PDF pp 80–109)
Pages touched: [wiki/procedures/trusted-non3GPP-attach.md (new), wiki/sources/ts23402-section4.md (updated), wiki/index.md (updated, 43→44 pages)]

## [2026-04-11] ingest | TS 23.402 §5.1–§5.13 — PMIP-based S5/S8 Procedures for 3GPP Accesses [chunk 3b-5]

Ingested §5.1.2 (§5 specifies PMIP S5/S8 deltas from TS 23.401 GTP baseline); §5.1.3 (Control Plane: PMIPv6 RFC 5213 over IPv4/IPv6 SGW↔PGW); §5.1.4.1–4 (User Plane: E-UTRAN S1-U GTP-U + S5 GRE; 2G S4 GTP-U + S5 GRE; 3G S4 GTP-U relay + S5 GRE; 3G S12 direct RNC→SGW + S5 GRE); §5.2 (Initial Attach PMIP S5/S8: old SGW de-registration blocks A.1-A.4 and B.1-B.4: BBERF GW Control Session Term + PBU(lifetime=0) + PCEF IP-CAN Term + PBA; new S5 session C.1-C.5: GW Control Session Establishment(IMSI,APN-AMBR,Default Bearer QoS,UE Location) + PBU(MN NAI,Lifetime,AT-Type,HO-Indicator,APN,GRE key) + PCEF IP-CAN Session Establishment + PBA(UE Addr,GRE uplink,Charging ID,APN-AMBR) + GW Control+QoS Rules Provision; Emergency: IMSI marked unauthenticated if so flagged); §5.3 (Detach: A.1-A.4 BBERF GW Control Term + PBU(lifetime=0) + PCEF IP-CAN Term + PBA; repeats per PDN); §5.4.1 (Dedicated bearer general: critical PMIP departure — PCRF sends PCC to SGW(BBERF), SGW generates TFT, SGW drives Create Bearer to MME; PGW only receives B.2 PCC Rules Provision after SGW completes); §5.4.2 (Activation: PCRF→SGW A.1 → SGW→MME Create Bearer Request between A.1/B.1 → SGW→PCRF B.1 → PCRF→PGW B.2); §5.4.3.1 (PCC-initiated QoS modification: same Figure 5.4.1-1, SGW sends Update Bearer with TFT); §5.4.3.2 (HSS-initiated subscribed QoS: SGW→PCRF A.1 GW Control+QoS Request + PCRF→PGW A.2 PCC provision → TS 23.401 §5.4.2.1 Update Bearer → B.1/B.2 completion); §5.4.4 (Bearer modification without QoS update: SGW sends Update Bearer(TFT only)); §5.4.5.1 (PCC-initiated deactivation: SGW drives Delete Bearer to MME); §5.4.5.3 (MME-initiated deactivation: SGW→PCRF A.1 + PCRF→PGW A.2; SGW follows §5.4.4.2 independently of A.1 completion); §5.5 (UE-initiated resource request: UE TAD → SGW GW Control+QoS Rules Request → PCRF decision → PCEF update); §5.6.1 (UE-requested PDN connectivity: Alt A GW Control+PBU+IP-CAN Establishment+Modification+PBA; Alt B PBU first lower-jitter; HO re-establishment: Handover Indicator, Charging ID reuse from PMIP source or GTP Default Bearer); §5.6.2.1 (PDN disconnection: A.1-A.4 GW Control Term + PBU(lifetime=0) + PCEF term + PBA); §5.6.2.2 (PGW-initiated PDN disconnect: Binding Revocation Indication(PDN address) → bearer deactivation → BBERF GW Control Term → Binding Revocation Ack); §5.7.0 (without SGW relocation: GW Control+QoS Rules Request(RAT+Location) → PCC Rules to PGW; no PBU); §5.7.1 (with SGW relocation: new SGW GW Control Session + PCC Rules + PBU + PBA + End Marker Indication; old SGW GW Control Session Term; EPS Bearer ID transferred S10/S11); §5.7.2 (inter-RAT TAU without relocation: GW Control+QoS Rules Request; with relocation: same as §5.7.1); §5.10.1-5.10.10 (S4 GERAN/UTRAN: PMIP equivalents for TS 23.060 procedures; GW Control+QoS Rules Request pattern replaces GTP boxes); §5.11 (IPv4 address delete: PCEF Modification + GW Control Rules + TS 23.401 bearer mod + Binding Revocation Indication(IPv4 only, NOT full PDN)); §5.12 (Location Change Reporting: SGW→PCRF GW Control+QoS Rules Request + PCRF→PGW PCC provision); §5.13 (MTC/overload: PGW rejects PBU with APN-congested indication + back-off timer).

Notable findings:
- The BBERF model in PMIP S5/S8 fundamentally inverts the bearer decision chain compared to GTP S5/S8. In GTP, the PGW drives bearer lifecycle (it generates TFTs, sends Create/Update/Delete Bearer Request to SGW). In PMIP, the SGW (BBERF) makes bearer decisions based on PCRF QoS rules, and the PGW is only updated after the fact via B.2 PCC Rules Provision. This means the SGW in PMIP mode has significantly more intelligence — it must implement TFT generation logic that in GTP mode lives only at the PGW.
- The GW Control Session (Gxc) is the foundational difference: it pre-exists the PMIP binding and allows PCRF to correlate bearer-level QoS at the BBERF with the PCEF-level policy at the PGW. Without Gxc, the two policy planes would be decoupled.
- Intra-LTE HO with SGW relocation (§5.7.1) requires End Marker Indication — the new SGW cannot simply start forwarding downlink GRE packets until the old SGW signals it has flushed its buffer. This prevents out-of-order delivery at the target eNodeB during the HO window. The End Marker mechanism is functionally identical to the GTP-based variant in TS 23.401 §5.5.1.
- §5.11 (IPv4 address delete) uses a partial Binding Revocation — only the IPv4 address is revoked, not the full PMIP binding. This is unique to PMIP because PMIPv6 can maintain a dual-stack binding where only one address type expires. GTP S5/S8 has no equivalent — IPv4 address management is handled entirely within the GTP session context without partial binding revocation.
- The S4-based GERAN/UTRAN procedures (§5.10) demonstrate that PMIP S5/S8 can be combined with S4 (SGSN-based) access — the S4 interface uses GTP, but S5/S8 is still PMIP. This heterogeneous stack is fully supported and shows the clean decoupling between S4/S11 (GTP) and S5/S8 (PMIP/GTP) protocol choices.

Next chunk: Phase 3b complete. Phase 4 or lint pass?
Pages touched: [wiki/procedures/PMIP-S5S8-procedures.md (new), wiki/sources/ts23402-section4.md (updated), wiki/index.md (updated, 44→45 pages)]

---

## [2026-04-11] create | MME Deep-Dive Synthesis [chunk 4-1]

Synthesized all knowledge about the MME accumulated across Phase 1–3 ingest into a single comprehensive deep-dive page. Drew on: TS 23.401 §4 (architecture), TS 23.401 §5 procedures (attach, TAU, service request, detach, bearer lifecycle, handover), TS 23.402 §4/§7/§8 (non-3GPP handover MME role). No new PDF reads required — all source material was already ingested.

Sections produced: architectural position diagram (graph LR), complete interface table (9 interfaces: S1-MME, S6a, S11, S10, S3, S13, S7a, SGs, Nq), S6a Diameter message table (AIR/AIA, ULR/ULA, CLR/CLA, IDR/IDA, DSR/DSA, NOR/NOA, PUR/PUA, RSR/RSA — 16 message types), S11 GTPv2-C message table (25+ message types covering full bearer lifecycle + DDN + forwarding tunnels), S10 GTPv2 message table (Forward Relocation, Context Request/Response, Identification, Relocation Cancel), S1-AP message table (20+ message types), EMM state machine (stateDiagram-v2), ECM state machine (stateDiagram-v2), timers table, procedure participation for 9 procedures (each with flowchart/sequence diagram), UE context ER diagram (erDiagram), overload behavior state machine, load balancing diagram, configuration parameters table, full cross-references section.

Notable findings:
- MME owns both EMM and ECM state machines and is the sole node with complete visibility of UE mobility+session state simultaneously. Every other node only sees a partial view (SGW sees user-plane bearers; HSS sees subscription; eNB sees radio state).
- S11 carries ~25 distinct GTPv2-C message types — more than any other single interface in the EPC. This reflects the MME's role as the orchestration hub: it must express every bearer lifecycle operation across the user-plane path.
- The MME does not forward user-plane traffic at all — every S11 message is purely control-plane. The distinction between user-plane anchor (SGW/PGW) and control-plane orchestrator (MME) is architecturally clean and is the defining EPC design principle.
- The Mobile Reachability + Implicit Detach timer pair implements a two-stage presence model: first, the network stops trying to page (MRT expiry); second, the network discards all state (IDT expiry). This prevents unbounded UE context accumulation during long idle periods.

Next chunk: 4-2 — PGW deep-dive synthesis
Pages touched: [wiki/entities/MME-deepdive.md (new), wiki/index.md (updated, 45→46 pages)]

---

## [2026-04-11] create | PGW Deep-Dive Synthesis [chunk 4-2]

Synthesized all knowledge about the PGW from Phase 1–3 ingests (TS 23.401 §4/§5, TS 23.402 §4–§8, TS 23.228 §5) into a comprehensive deep-dive page. No new PDF reads required.

Sections produced: architectural position diagram, complete interface table (12 interfaces including S5/S8 GTP and PMIP variants, S2a/S2b/S2c, SGi, Gx, Gy, Gz/Rf, S6b, Gxb), full GTPv2-C message table for S5/S8 (session lifecycle + bearer lifecycle + commands), PMIPv6 message table for S5/S8 PMIP variant (PBU/PBA/BRI/BRA including partial BRI), PMIPv6 messages for S2a/S2b, Diameter message tables for Gx (CCR/CCA/RAR/RAA/ASR/ASA) and Gy (credit control) and S6b (non-3GPP auth + HSS-initiated profile), IP allocation flowchart, PCEF bearer binding + PCC rule lifecycle sequence diagram, procedure participation for 8 procedures (attach, dedicated bearer GTP, dedicated bearer PMIP, PDN disconnection, SGW relocation HO, non-3GPP→E-UTRAN HO, PDN connectivity, VoLTE bearer trigger), GTP vs PMIP dedicated bearer comparison table, Gx IP-CAN session state machine, PGW data model ER diagram (PDN connection / EPS bearer / TFT filter / PCC rule / PMIP binding), charging section (offline CDRs, online Gy quota, Charging ID persistence), MIPv4/DSMIPv6 roles table, End Marker mechanism sequence diagram, failure/overload state machine, Single Gateway (collocated) note, configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- The fundamental difference between GTP and PMIP S5/S8 appears most clearly from the PGW's perspective: in GTP mode the PGW drives all bearer lifecycle (Create/Update/Delete Bearer Request), while in PMIP mode the PGW is a passive recipient of B.2 PCC Rules Provision after the SGW (BBERF) has already acted. The PGW's procedural authority is significantly curtailed in PMIP mode.
- The Charging ID is the most durable PGW-issued identifier — it survives SGW relocation, RAT type change, and even access type handover (3GPP↔non-3GPP). It is the single thread connecting charging records across a UE's entire PDN connection lifetime.
- The PGW is the only EPC node that terminates ALL three access paths (3GPP via S5/S8, trusted non-3GPP via S2a, untrusted non-3GPP via S2b) and also directly hosts the UE mobility endpoint for S2c. No other EPC node has this multi-access convergence role.
- The Handover Indication in Create Session Request is the PGW's signal to suppress IP reallocation — without it, the PGW would allocate a new IP and IP continuity across access type change would be broken. The flag converts what would otherwise be a fresh attach into a path migration at the PGW level.

Next chunk: 4-3 — SGW deep-dive synthesis
Pages touched: [wiki/entities/PGW-deepdive.md (new), wiki/index.md (updated, 46→47 pages)]

---

## [2026-04-11] create | SGW Deep-Dive Synthesis [chunk 4-3]

Synthesized all knowledge about the SGW from Phase 1–3 ingests (TS 23.401 §4/§5, TS 23.402 §5 PMIP) into a comprehensive deep-dive page. No new PDF reads required.

Sections produced: architectural position diagram (graph LR showing S1-U, S11, S5/S8, S4, S12, Gxc), complete interface table (8 interfaces including PMIP S5/S8 and Gxc PMIP-only), full GTPv2-C S11 message table (session lifecycle + bearer lifecycle + commands + DDN/buffering + indirect forwarding tunnels), GTPv2-C S5/S8 relay behavior notes, PMIPv6 S5/S8 message table (PBU/PBA/BRI/BRA), Gxc Diameter message table (GW Control Session lifecycle + QoS Rules Provision), ECM-IDLE buffering/DDN sequence diagram (with extended buffering note), X2 HO path-switch sequence diagram, SGW relocation sequence diagram, S1 HO indirect data forwarding sequence diagram, PMIP BBERF flowchart comparing GTP vs PMIP bearer initiation, GTP vs PMIP capability comparison table, bearer context ER diagram (UE context / PDN connection / EPS bearer / TFT filter / DL buffer), procedure participation summary table (15 procedures), failure/overload state machine, configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- In GTP mode the SGW is architecturally thin — it stores TEID mappings and relays GTPv2-C messages, but does not know the UE's IP address or TFTs. All intelligence lives at PGW. This changes dramatically in PMIP mode where SGW becomes the BBERF and must implement TFT generation — a major functional expansion.
- The DDN → Paging coupling is the single most important reason the SGW must buffer (not discard) DL packets in ECM-IDLE. If the SGW discarded packets, the paging trigger would be lost and the UE would miss the notification. The buffer is a reliability mechanism, not a performance optimization.
- The SGW is the only EPC node that participates in every single procedure type (attach, TAU, service request, detach, bearer lifecycle, all handover variants, PDN connectivity). It is present in every path — making it the highest-availability requirement node in the EPC after the MME.
- Indirect Data Forwarding Tunnel (IDFT) state is ephemeral — created at HO preparation and deleted at HO completion. The SGW must handle a scenario where IDFT cleanup fails (e.g. MME crash post-HO): it should have a local timer to self-clean the forwarding state.

Next chunk: 4-4 — HSS deep-dive synthesis
Pages touched: [wiki/entities/SGW-deepdive.md (new), wiki/index.md (updated, 47→48 pages)]

---

## [2026-04-11] create | HSS Deep-Dive Synthesis [chunk 4-4]

Synthesized all knowledge about the HSS from Phase 1–3 ingests (TS 23.401 §4/§5, TS 23.228 §4–§5, TS 23.402 §4) into a comprehensive deep-dive page. No new PDF reads required.

Sections produced: architectural position diagram (graph LR showing S6a/S6d/S6b/Cx/Sh/Dh/SWx/MAP connections), complete interface table (9 interfaces), full S6a Diameter message table (AIR/AIA, ULR/ULA, CLR/CLA, IDR/IDA, PUR/PUA, NOR/NOA) with ULA subscription data AVP breakdown, full Cx Diameter message table (UAR/UAA, MAR/MAA, SAR/SAA, LIR/LIA, RTR/RTA, PPR/PPA) with SAR assignment type table, full Sh Diameter message table (UDR/UDA, PUR/PUA, SNR/SNA, PNR/PNA) with use case descriptions, SWx Diameter message table (MAR/MAA, SAR/SAA, RTR/RTA, PPR/PPA) with HSS-initiated QoS modification note, EPC subscriber data ER diagram (subscriber/EPS subscription/APN config/MME registration/SGSN registration), IMS subscriber data ER diagram (IMPI/implicit reg set/IMPU/service profile/iFC/SPT/S-CSCF registration/capabilities/charging info), IMS registration state machine (stateDiagram-v2: NotRegistered/Registered/UnregisteredServed), EPC location management state machine, HSS role in 4 key procedures (EPS attach sequence, IMS registration sequence, non-3GPP HO PGW identity reuse sequence, HSS-initiated push), multi-HSS/SLF routing flowchart, EPS AV generation flowchart (f1–f5 functions, Ki/SQN/RAND → RAND/XRES/AUTN/KASME), failure/overload state machine, key architectural properties table, configuration parameters table, full cross-references.

Notable findings:
- The HSS's most architecturally critical function for non-3GPP is PGW identity storage: it is the only node that can bridge the identity gap between a non-3GPP attach (where AAA registers the PGW identity to HSS via SWx) and a subsequent 3GPP attach (where MME retrieves it via S6a ULR). Without this cross-access state, IP continuity would be impossible.
- The SWx PPR message is the precise trigger for HSS-initiated QoS modification on S2b (§7.11.2): HSS→AAA via SWx, AAA→ePDG, ePDG→PGW. The PPR on SWx and the PPR on Cx are structurally identical (same Diameter application message) but serve completely different access domains.
- The HSS IMS registration state has three distinct states (NotRegistered, Registered, UnregisteredServed) — the UnregisteredServed state is important for voicemail and similar services that need to handle terminating calls for unregistered UEs. Without this state the S-CSCF would have no service profile to apply.
- The HSS is the only EPC/IMS node with a proactive push capability toward multiple peers simultaneously: it can send CLR to old MME while sending ULA to new MME, and PPR to S-CSCF while being queried by a new I-CSCF. This requires careful sequencing to avoid race conditions in the HSS.

Next chunk: 4-5 — PCRF deep-dive synthesis
Pages touched: [wiki/entities/HSS-deepdive.md (new), wiki/index.md (updated, 48→49 pages)]

---

## [2026-04-11] create | PCRF Deep-Dive Synthesis [chunk 4-5]

Synthesized all knowledge about the PCRF from Phase 1–3 ingests (TS 23.401 §4.4.7, TS 23.228 §5.4.5, TS 23.402 §4.3/§4.10/§5/§6/§7) into a comprehensive deep-dive page. No new PDF reads required.

Sections produced: architectural position diagram (graph LR with all interfaces including roaming S9), complete interface table (8 interfaces: Gx, Gxc, Gxa, Gxb, Rx, S9, Sp, Np), full Gx Diameter message table (CCR/CCA/RAR/RAA/ASR/ASA + CCR-Update triggers table), full Gxc Diameter message table with Gx vs Gxc comparison table, Gxa message table, full Rx Diameter message table with media component descriptor field breakdown, PCC rule generation flowchart (Rx AAR → SPR lookup → QCI mapping → TFT gen → charging → Gx RAR), QCI mapping table, roaming architecture diagrams (non-roaming/home-routed and local breakout), S9 Diameter message table, VoLTE bearer trigger end-to-end sequence diagram (with preconditions/gate control), gate control (2-phase) explanation, session correlation flowchart, PCC rule ER diagram (SDF filter / QoS params / charging params / gate status), PCRF session state machine, non-3GPP access comparison table (GTP vs PMIP vs trusted vs untrusted), failure/overload state machine, configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- The Gx vs Gxc distinction (PCC rules vs QoS rules) is the key to understanding why the PCRF must send to both PGW and SGW in PMIP mode: PCC rules carry charging parameters that the BBERF (SGW) cannot and does not enforce. The B.2 PCC Rules Provision to PGW is therefore not redundant — it delivers the charging enforcement half that the Gxc/QoS rules half cannot carry.
- The 2-phase gate control (gate DISABLED at Rx AAR, then gate ENABLED at answer) is the PCRF's implementation of IMS preconditions. Without this, RTP would flow before both endpoints confirm their codec/port parameters. The PCRF is the gating arbiter — not the endpoints.
- Session correlation by UE IP address is fragile in dual-stack scenarios: if UE has both IPv4 and IPv6, the P-CSCF may send AAR with either address. The PCRF must match against both. In practice this is handled by including both addresses in the Gx CCR-Initial and checking either against incoming AAR.
- The H-PCRF has final authority in roaming local breakout, but V-PCRF is the enforcement coordinator. If S9 goes down, V-PCRF must decide whether to continue with cached policy or fall back to best-effort — a split-brain scenario with no protocol-defined resolution, left to operator configuration.

Next chunk: 4-6 — P-CSCF deep-dive synthesis
Pages touched: [wiki/entities/PCRF-deepdive.md (new), wiki/index.md (updated, 49→50 pages)]

---

## [2026-04-11] create | P-CSCF Deep-Dive Synthesis [chunk 4-6]

Synthesized all knowledge about the P-CSCF from Phase 1–3 ingests (TS 23.228 §4.6, §5.1–§5.4, §5.10–§5.11) into a comprehensive deep-dive page.

Sections produced: architectural position diagram (VPLMN placement highlighted), complete interface table (Gm/Mw/Rx), P-CSCF discovery flowchart (PCO/DHCP/DNS), SIP message handling tables (registration + session), IPsec SA negotiation sequence diagram (4-SA model, key derivation from CK+IK), AF/Rx session lifecycle sequence diagram (gate=DISABLED preconditions → gate=ENABLED → Rx STR), Rx AAR content table (SDP field → Rx AVP mapping), P-CSCF response to PCRF notification table, P-header insertion table (P-Visited-Network-ID, P-Access-Network-Info, P-Associated-URI, P-Charging-Vector, etc.), procedure participation for 6 procedures (initial registration, VoLTE MO, VoLTE MT, session release, re-registration, network-initiated de-registration), SigComp section, emergency call handling flowchart, UE registration state ER diagram (registration / IPsec SA / Rx session), failure/overload state machine, configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- The P-CSCF's dual role (SIP proxy + Diameter AF) is not merely an implementation convenience — it is the architectural mechanism that bridges the IMS session model and the EPC bearer model. Without P-CSCF on both Mw and Rx, VoLTE dedicated bearer establishment would require out-of-band coordination between IMS and EPC.
- The 4-SA IPsec model (two directions × protect/unprotect) is subtle: the "unprotected" SAs exist only to receive the initial unauthenticated REGISTER. After authentication, all traffic uses the protected SAs. The port-based multiplexing (SPI+port pair) allows multiple UEs behind NAT to share the same P-CSCF IP address.
- P-CSCF's RTCP sub-component in the Rx AAR is easy to miss: without it, RTCP packets (RTP port+1) get no GBR treatment and land on the default bearer. RTCP carries QoS feedback — degraded RTCP is a common VoLTE quality bug traceable to this omission.
- P-CSCF has no HSS access — it is entirely stateless with respect to the subscriber's service profile. This is a deliberate design: the visited network should not need to access home subscriber data. All subscriber-specific routing is handled by S-CSCF in the HPLMN.

Next chunk: 4-7 — S-CSCF deep-dive synthesis
Pages touched: [wiki/entities/P-CSCF-deepdive.md (new), wiki/index.md (updated, 50→51 pages)]

---

## [2026-04-11] create | S-CSCF Deep-Dive Synthesis [chunk 4-7]

Synthesized all knowledge about the S-CSCF from Phase 1–3 ingests (TS 23.228 §4.8/§5, TS 23.218 §4–§9) into a comprehensive deep-dive page.

Sections produced: architectural position diagram (graph LR showing all peers), complete interface table (7 interfaces: Mw, ISC, Cx, Mi, Mr, Mm, Mj), full Cx Diameter message table (MAR/MAA, SAR/SAA, RTR/RTA, PPR/PPA + SAR assignment type table), IM Call Model iFC evaluation algorithm flowchart, SPT type table, ODI sequence diagram (iFC chain fork + resume), ICID/IOI charging notes, Transit Function section, originating session routing flowchart, terminating session routing flowchart, procedure participation for 7 procedures (initial registration 11-step, VoLTE MO, VoLTE MT, third-party REGISTER, network-initiated de-registration, session release, PPR mid-session), registration state ER diagram (subscriber context / registered contact / service profile / iFC / SPT / active dialog), S-CSCF selection and capabilities table, unregistered termination flowchart, AS interaction modes table (5 modes), failure/overload state machine (with Default Handling discussion), configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- The ODI (Original Dialog Identifier) is the key mechanism that makes multi-AS iFC chains work. Without ODI, the S-CSCF would have no way to know which iFC triggered the current AS return, and could not resume the chain at the right position. It is a private token — ASes must preserve it but never interpret it.
- The Default Handling=CONTINUE safety mechanism is architecturally important: it means AS failure does not automatically break calls. This is the correct default for most services (MMTEL features are nice-to-have, not call-breaking). Only compliance/legal services (lawful intercept) should use SESSION_TERMINATED to guarantee no calls proceed if the intercept AS is down.
- The unregistered termination flow (SAR UNREGISTERED_USER) is a special case where the S-CSCF is assigned by the HSS to handle a single terminating session, not a permanent registration. The S-CSCF gets the service profile for unregistered case — which typically has only the voicemail iFC. After the session, the S-CSCF sends SAR(UNREGISTERED_USER cleanup) and releases state.
- Two S-CSCFs participate in every VoLTE call: the originating S-CSCF (which evaluated originating iFCs and routed toward the destination) and the terminating S-CSCF (which evaluated terminating iFCs and delivered to the UE). They run completely independent iFC chains. This is why per-direction iFC filtering (Direction=Originating vs Terminating) is essential — without it, every iFC would fire on both sides.

Next chunk: 4-8 — TAS deep-dive synthesis
Pages touched: [wiki/entities/S-CSCF-deepdive.md (new), wiki/index.md (updated, 51→52 pages)]

---

## [2026-04-11] create | TAS Deep-Dive Synthesis [chunk 4-8] — Phase 4 Complete

Synthesized all knowledge about the TAS from Phase 1–3 ingests (TS 23.228 §4.13/§5, TS 23.218 §9 + Annexes B/C) into a comprehensive deep-dive page. Final chunk of Phase 4.

Sections produced: architectural position diagram, complete interface table (7 interfaces: ISC, Sh, Mr, Mr', Ut, Dh, Cr), Sh interface subscriber data table (CFU/CFB/CFNRy/CFNRc/barring/CLIR/CW/MWI), Sh fetch sequence diagram, Sh notification subscription sequence diagram, all 5 AS mode detailed flows (SIP proxy for CLIR, B2BUA for hold, 302 redirect for CFB, B2BUA conference with MRFC, originating UA for MWI), MMTEL call forwarding decision tree flowchart, call barring checks, CLIR sequence, call waiting description, voicemail integration (deposit + MWI on-registration), transcoding via MRFC B2BUA sequence, TAS subscriber data model ER diagram, procedure participation summary table (10 procedures), Ut/XCAP self-service flowchart with XCAP URI examples, failure/overload state machine, configuration parameters table, key architectural properties table, full cross-references.

Notable findings:
- TAS is the only AS that naturally exercises all 5 AS modes in a single VoLTE call: proxy (CLIR), B2BUA (hold), redirect (call forwarding), and potentially originating UA (MWI) — all within the same subscription. This makes TAS the most architecturally complex AS in IMS, despite appearing simple from the outside.
- The Sh cache + SNR subscription pattern is critical for TAS performance: fetching Sh on every ISC INVITE would add a Diameter RTT to every call. Operators must configure Sh SNR subscriptions to ensure TAS caches stay current when subscribers change settings via Ut/XCAP.
- The Mr vs Mr' distinction matters for session visibility: when TAS uses Mr (via S-CSCF), the S-CSCF sees the media resource as part of the session and can apply iFC evaluation to it. When TAS uses Mr' (direct), the S-CSCF is bypassed — correct for most conference/transcoding cases where S-CSCF involvement would add unnecessary signaling hops.
- The conference ID reuse pattern (TS 23.218 Annex B.2.2) is subtle but architecturally important: TAS sends multiple independent INVITEs to MRFC, each referencing the same conference ID. The MRFC interprets same-ID INVITEs as "add this party to the existing bridge" rather than "create a new bridge." TAS doesn't need a separate "add participant" API — it's the conference ID that carries the binding semantics.

Phase 4 complete. All 8 entity deep-dives written (MME, PGW, SGW, HSS, PCRF, P-CSCF, S-CSCF, TAS).
Next: Wiki is complete for Phases 1–4. Options: Phase 5 (additional entities: BGCF, MRF, I-CSCF, ePDG), lint pass, or protocol pages.
Pages touched: [wiki/entities/TAS-deepdive.md (new), wiki/index.md (updated, 52→53 pages)]

---

## [2026-04-12] ingest | TS 23.167 §3–§6 [chunk 5a-1] — IMS Emergency Architecture and Entities

Began ingest of 3GPP TS 23.167 v17.2.0 (IMS Emergency Sessions). This is a new source not in the original Phase 1–4 plan; added as Phase 5a. Chunk 5a-1 covers §3 (definitions/abbreviations), §4 (high-level principles), §5 (architecture and reference points), and §6 (functional description of all entities in the emergency system).

Sections produced: full architectural principles summary (30 principles distilled), reference architecture diagram (E-CSCF + LRF placement in serving network, all interfaces), new reference points table (Ml/I4/I5/I6/Le), UE-detectable vs non-detectable emergency flowchart, location information principles and source priority table, IP-CAN expectations table, E-CSCF entity page with full interface table and PSAP routing decision flowchart, LRF entity page with RDF/LS decomposition and ESQK binding pattern, source summary page.

Notable findings:
- The E-CSCF is architecturally mandated to reside in the **serving (visited) network** — the only CSCF role with this hard constraint. Unlike the S-CSCF (home-network anchor) or even the P-CSCF (typically visited but not mandatory), the E-CSCF must be local because PSAP routing is determined by the serving country's geographic regulations. A home-network E-CSCF would route to the home country's PSAP, which is wrong for a roaming user calling 112.
- The **ESQK indirection pattern** (LRF stores session record keyed by ESQK; PSAP queries LRF via Le/E2 using ESQK) is architecturally elegant: it decouples the IMS network from the PSAP's location intelligence system. The IMS network does not need to understand dispatchable location formats — it just passes an opaque ESQK token. The PSAP then queries the LRF independently to get the location in whatever format it needs. This also allows **location updates** after the call is established (PSAP re-queries LRF), without modifying the SIP session.
- **Anonymous emergency sessions** are explicitly a first-class architectural case, not an afterthought. A UE with no IMS credentials can reach PSAP via: P-CSCF → E-CSCF → PSAP using only an equipment identifier. This is a regulatory requirement (emergency always possible) and required the S-CSCF and HSS to be bypassed entirely on the path.
- NG-eCall is architecturally identical to regular IMS emergency with one addition (MSD transfer via in-band modem or IMS data channel per TS 26.267). The same E-CSCF/LRF/PSAP path is reused — no new entities needed.

Next chunk: 5a-2 — TS 23.167 §7.1–§7.4: Emergency session procedures (high-level flows, emergency registration, serving-IMS establishment, session without registration)
Pages touched: [wiki/entities/E-CSCF.md (new), wiki/entities/LRF.md (new), wiki/concepts/IMS-emergency-architecture.md (new), wiki/sources/ts23167.md (new), wiki/index.md (updated, 53→57 pages)]

---

## [2026-04-12] ingest | TS 23.167 §7.2–§7.7 + Annex C [chunk 5a-2] — Emergency Session Procedures

Ingested all core emergency session procedures from TS 23.167: §7.2 (IMS emergency registration), §7.3 (serving-IMS session establishment), §7.4 (anonymous/unregistered session), §7.5 (PSAP interworking variants), §7.6 (location information retrieval 13-step procedure), §7.7 (NG-eCall MSD transfer — two scenarios + updated MSD), and Annex C (fixed broadband access emergency location).

Sections produced: three-path emergency session decision flowchart (credentialed/anonymous/non-UE-detectable), emergency registration trigger conditions and anti-looping rule, P-CSCF action flowchart per registration state, E-CSCF→LRF→PSAP sequence diagram, anonymous session path (§7.4), PSAP interworking variant table (4 types), full 13-step location retrieval sequence diagram with per-step annotation table, NG-eCall §7.7.1 sequence (NG-capable PSAP), §7.7.2 inband fallback sequence (legacy PSAP via MGCF), §7.7.3 updated MSD sequence, domain priority and selection rules, Annex C fixed broadband location behaviour table.

Notable findings:
- The **anti-looping rule for emergency registration** (§7.2) is subtle: if the UE receives "IMS emergency registration required" after already having performed emergency IP-CAN access, it must try a *different* VPLMN or SNPN. Without this, a UE could loop between IP-CAN access → emergency registration required → IP-CAN access again in the same network. The rule forces escalation to a different serving network.
- **Location retrieval is inherently asynchronous with PSAP delivery** (steps 9–11 in §7.6). The LRF may send an initial location estimate to the PSAP proactively (before the PSAP even sends a location request at step 9). This "push-before-pull" pattern exists because PSAP dispatch starts immediately upon call answer — waiting for the PSAP to request location adds latency. The proactive push reduces dispatch time even if the location is only approximate.
- **NG-eCall routing is based on UE location + eCall type, NOT on MSD content** (§7.7.1 NOTE). This is architecturally correct: the MSD contains vehicle data (VIN, crash direction, propulsion type) — none of which is relevant for choosing a PSAP. Only location determines which PSAP serves that area. The MSD is transparent to the IMS routing layer — it rides as a body part in the INVITE.
- **Inband MSD fallback** (§7.7.2) is the graceful degradation mechanism when the PSAP is CS-domain or does not support NG-eCall. The same UE/IVS can attempt NG-eCall via IMS and if the PSAP cannot receive MSD in the SIP INVITE, the eCall Inband Modem (DTMF-derived modem per TS 26.267) transmits the MSD over the voice channel. No extra bearer is required — the voice channel itself is the transport.

Next chunk: 5a-3 — TS 23.167 Annex H (E-UTRAN/LTE specifics: domain priority, eCall over IMS/LTE) + Annexes J/K (WLAN-to-EPC emergency, roaming without IMS interfaces)
Pages touched: [wiki/procedures/IMS-emergency-session.md (new), wiki/sources/ts23167.md (updated), wiki/index.md (updated, 57→58 pages)]

---

## [2026-04-12] ingest | TS 23.167 Annexes E/H/J/K/L [chunk 5a-3] — Access-Specific Emergency Procedures — TS 23.167 COMPLETE

Final chunk of TS 23.167 ingest. Covered: Annex E (IP-CAN support matrix), Annex H (E-UTRAN/UTRAN/NG-RAN normative — the primary 4G annex), Annex I (HRPD/EPC, brief), Annex J (WLAN to EPC normative), Annex K (roaming without IMS-level interfaces / GIBA), Annex L (non-3GPP to 5GC normative). Annexes D and G noted (informative, NENA I2 examples and TEL-URI provisioning).

Sections produced: IP-CAN emergency capability table (8 access types × 3 capability dimensions), E-UTRAN UE behaviour flowchart (detectable vs non-detectable, emergency EPS bearer requirement, CGI inclusion), E-UTRAN location handling (PCC-based vs LRF-based operator policy comparison), Domain Priority Table H.1 (7-row matrix for voice emergency — CS/PS attach state × VoIMS/EMS indicators), eCall Domain Selection Table H.2 (6-row matrix for NG-eCall — ECL indicator driven), WLAN-to-EPC 7-step emergency session flow, WLAN emergency number detection sources table (4 sources: 3GPP MM / ANQP / DNS / IKEv2), GIBA authentication architecture diagram, Annex K 16-step emergency registration flow without IMS NNI (three branches: GIBA → normal, anonymous fallback, CS fallback), non-3GPP-to-5GC summary with location constraint, access-type summary comparison diagram.

Notable findings:
- **Domain Selection Table H.1 captures the fundamental 4G/VoLTE PS-CS interplay for emergencies.** Row E (CS=Y, PS=Y, VoIMS=Y, EMS=Y) is the most common VoLTE-capable scenario: the UE uses the same domain as for normal calls (per TS 22.101), which for a VoLTE UE means PS. This is elegant — it means operators don't need a special emergency domain selection policy if VoLTE is fully deployed. The complexity appears only in partial-deployment scenarios (rows B, C, F) where VoIMS or EMS is missing.
- **The eCall ECL indicator (Table H.2) is the gating mechanism for NG-eCall deployment.** Row B (ECL=N, VoIMS=Y, EMS=Y) is the transitional state: the cell/network supports VoLTE and IMS emergency, but not NG-eCall. In this state the UE first tries CS eCall (traditional), then falls back to a regular IMS emergency session (no MSD). This means the PSAP loses the MSD in the transitional deployment state — a deliberate trade-off between PSAP capability constraints and emergency availability.
- **GIBA (Annex K) solves a real deployment gap**: many roaming agreements have no IMS NNI, meaning the P-CSCF in the visited PLMN cannot reach the home S-CSCF. Without GIBA, roaming subscribers would always fall to anonymous emergency sessions (no callback number). GIBA enables the P-CSCF to derive a TEL-URI from the MSISDN retrieved from the PCRF, giving the PSAP a callback number without requiring home-network IMS involvement.
- **The N3IWF location limitation (Annex L §L.4)** is architecturally significant: for untrusted non-3GPP access to 5GC, the only location available is the UE's IP address at the N3IWF. This IP address has no geographic meaning for PSAP routing — the LRF cannot determine which PSAP serves that "location." In practice, such emergency calls will always default to a national or default PSAP, regardless of the UE's actual physical location. This is an unavoidable consequence of IPsec tunnelling hiding the UE's physical network attachment point.

TS 23.167 ingest COMPLETE. All 3 chunks written (5a-1 through 5a-3). Wiki now covers full IMS emergency session specification.
Pages touched: [wiki/procedures/IMS-emergency-access-variants.md (new), wiki/sources/ts23167.md (updated, marked COMPLETE), wiki/index.md (updated, 58→59 pages)]

---

## [2026-04-12] ingest | TS 23.216 §1–§5 [chunk 5b-1] — SRVCC Concepts and Architecture

Began ingest of 3GPP TS 23.216 v16.4.0 (Single Radio Voice Call Continuity). This is a new Phase 5b source. Chunk 5b-1 covers §1 (Scope), §2 (References), §3 (Definitions and abbreviations), §4 (High-level Principles and Concepts — all 7 sub-sections for all variants), and §5 (Architecture model and reference points — all 6 architecture variants, all functional entities, all reference points).

Sections covered: scope overview and 6 SRVCC variant list; key definitions (STN-SR, E-STN-SR, C-MSISDN, 1xCS IWS, vSRVCC, 5G-SRVCC, MME_SRVCC, PS bearer splitting); all 4 sets of architectural principles; high-level concept descriptions for all 6 variants with sequence/architecture figures extracted; all 6 reference architecture models (5.2.1–5.2.6); full functional description of all 11 enhanced entities (5.3.1–5.3.11); all 8 reference points (5.4.1–5.4.8).

Notable findings:
- The **QCI=1 exclusivity principle** is the lynchpin of PS bearer splitting. By mandating that QCI=1 is only used for SCC AS-anchored IMS voice bearers (enforced by PCRF), the MME/SGSN can unambiguously identify which PS bearer to split toward SRVCC. If PCC is not deployed, the PDN GW cannot enforce this, and the splitting function loses its reliability guarantee. This is explicitly noted as a risk in §5.3.5.
- **MME_SRVCC is architecturally minimal by design** (§5.3.3.4): it only converts between N26 and Sv messages and stores the N26/Sv binding per UE. It does not hold UE MM context. This is deliberate — 5G-SRVCC reuses the entire 3G SRVCC path (Sv → MSC Server → IMS Service Continuity per TS 23.237) unchanged, with MME_SRVCC as the only new network element needed. This minimises 5G-SRVCC deployment cost.
- **CS to PS SRVCC eligibility is triple-gated** (§5.3.2b): UE capability (from IMS/SCC AS per TS 23.237), subscription allowance (HSS MAP D, VPLMN-specific), AND active IMS registration. All three must be satisfied. This prevents a CS call transferring to PS when the PS-side IMS infrastructure cannot support it.
- **Emergency SRVCC's E-STN-SR bypass** (§4.2.4.1) is architecturally necessary for UICC-less UEs that have no HPLMN subscription to provision an STN-SR. The MSC Server's locally configured E-STN-SR routes to the EATF regardless of subscriber identity. Without this, any UICC-less emergency call (e.g., child without SIM using parent's phone) would fail SRVCC on coverage boundary crossing.
- **5G-SRVCC terminates all PDU sessions** (§4.2.7.1): after the UE moves to 3G CS, the AMF releases all 5G PDU sessions because the UE cannot maintain PS connectivity on the UTRAN CS access. This is a fundamental architectural difference from E-UTRAN→UTRAN SRVCC where non-voice PS bearers survive via a parallel PS-PS handover.

Next chunk: 5b-2 — TS 23.216 §6.1–§6.3: Procedures — E-UTRAN→3GPP2 1xCS (19-step call flow), E-UTRAN→UTRAN/GERAN (v)SRVCC call flows, UTRAN(HSPA)→UTRAN/GERAN call flows
Pages touched: [wiki/concepts/SRVCC.md (new), wiki/sources/ts23216.md (new), wiki/index.md (updated, 59→61 pages)]

---

## [2026-04-12] ingest | TS 23.216 §6.1–§6.3 [chunk 5b-2] — SRVCC Procedures from E-UTRAN and UTRAN(HSPA)

Ingested all call flow procedures for SRVCC originating from E-UTRAN (§6.1, §6.2) and UTRAN(HSPA) (§6.3). Coverage: §6.1 E-UTRAN Attach/Service Request/PS HO enablement for 1xCS + 19-step E-UTRAN→1xCS call flow; §6.2 all 5 (v)SRVCC scenarios with full step-by-step flows (§6.2.2.1, §6.2.2.1A, §6.2.2.2, §6.2.2.3, §6.2.2.4) plus enabling procedures (Attach, Service Request, PS HO, Dedicated Bearer for vSRVCC); §6.3 all 3 SGSN-based SRVCC flows (§6.3.2.1, §6.3.2.1A, §6.3.2.2) plus GPRS Attach and PS HO enablement for HSPA.

Notable findings:
- **Session Transfer independence** (§6.2.2.2 NOTE 3): Steps 10 (Session Transfer initiation) and 11/12 (remote end update + IMS access leg release) are explicitly independent of step 13 (PS to CS Response from MSC Server). Session Transfer can begin as soon as the Prepare HO Response is received (step 8b). This design minimises voice gap: the IMS path switches to the CS SDP as early as possible, even before the UE has received the HO Command.
- **SRVCC CS KEYS exchange** (§6.3.2.2): UTRAN(HSPA)→UTRAN SRVCC requires an additional message pair (SRVCC CS KEYS REQUEST/RESPONSE) before the Relocation Required message. This exchanges Integrity Protection Key, Encryption Key, and SRVCC Information from SGSN to RNC. The RNC cannot trigger CS+PS HO to UTRAN without these keys. There is no equivalent step for E-UTRAN→UTRAN SRVCC (the MME provides the keys in the Sv message directly to the MSC Server).
- **vSRVCC detection at SCC AS** (§6.2.2.3 step 10): The SCC AS determines whether vSRVCC vs SRVCC applies by checking whether the Session Transfer response (CS access leg SDP) contains a video component. This is architecturally elegant — no separate vSRVCC flag needed in SIP; the media description itself is the signal. If video SDP is absent (e.g., BS30 reservation failed and MSC Server falls back to voice-only), SCC AS silently handles it as voice-only SRVCC and releases the video bearer from the remote leg.
- **GERAN NONCE IE** (§6.3.2.2 step 13): for SRVCC to GERAN, the Relocation Command must include the SRVCC Information IE containing a NONCE IE for the source RNC. This NONCE is used by the GERAN-side to derive the CS security context. No equivalent NONCE step exists for UTRAN targets in §6.3.2.2.
- **Gn/Gp SGSN bearer suspension** (§6.3.2.1 step 22a): non-voice non-CS-suspended PDP Contexts are preserved by setting max bitrate to 0 kbit/s rather than deactivating them. This is the Gn/Gp-era mechanism for "bearer suspension" — it avoids re-establishment overhead while the UE is on CS, but keeps the PDP Context alive for resumption. The equivalent S4-SGSN mechanism uses Suspend Notification to S-GW.

Next chunk: 5b-3 — TS 23.216 §6.4 (CS to PS SRVCC flows), §6.5 (5G-SRVCC from NG-RAN to UTRAN), §7 (Charging), §8 (Handover Failure), §9 (Security)
Pages touched: [wiki/procedures/SRVCC-from-E-UTRAN.md (new), wiki/procedures/SRVCC-from-UTRAN-HSPA.md (new), wiki/sources/ts23216.md (updated), wiki/index.md (updated, 61→63 pages)]

---

## [2026-04-12] ingest | TS 23.216 §6.4–§9 + Annex A [chunk 5b-3] — CS-to-PS SRVCC, 5G-SRVCC, HO Failure, Security

Completed the final chunk of TS 23.216. Ingested CS-to-PS SRVCC reverse procedures (§6.4): 16-step call flow from GERAN → E-UTRAN/UTRAN(HSPA) with triple eligibility gate (UE capability + subscription + IMS registration), plus §6.4.3.2/3.3 variants. Ingested 5G-SRVCC (§6.5): 18-step NG-RAN → UTRAN(HSPA) flow via AMF → MME_SRVCC (N26) → MSC Server (Sv), reusing §6.2.2.1A procedure, with all 5G PDU sessions released by AMF after CS HO; plus §6.5.5 emergency PDU session enabling procedure. Ingested handover failure handling (§8): four PS-to-CS failure modes (before/after Prepare HO Response, after HO Command delivered, HO cancellation, pre-alerting) and three CS-to-PS failure modes (before PS HO Request, after PS HO Request, BSC/RNC-initiated cancellation). Ingested security (§9): NDS/IP per TS 33.210 on S102 and Sv interfaces. Ingested NCL determination (Annex A): dual-gate algorithm — "SRVCC operation possible" flag + QCI=1 bearer existence required for including VoIP-incapable cells in NCL.

Notable findings:
- **CS-to-PS step 11a as dynamic PCC trigger** (§6.4.3.1): CS to PS Complete Notification triggers the PCRF to authorise IMS voice traffic on the default bearer via ATGW, before the dedicated IMS bearer is ready (step 16). This temporary window requires ATGW to carry the IMS stream on the default bearer.
- **HO cancellation recovery** (§8.1.3): The "Session Transfer started" indication in the PS to CS Cancel Notification ACK tells the MME/SGSN that IMS is mid-transfer. This triggers Session Reestablishment to the UE — the UE must re-INVITE on the existing IMS bearer to recover call continuity after an aborted SRVCC.
- **Permanent vs temporary failure distinction** (§8.1.1a.1): Error cause in PS to CS Response/Complete Notification directly controls whether MME/SGSN will retry SRVCC for this subscriber. A misconfigured STN-SR generates permanent failure and disables SRVCC for that subscriber.
- **5G-SRVCC terminates all PDU sessions** (§6.5.4 step 17): Unlike E-UTRAN SRVCC which suspends non-voice bearers, 5G-SRVCC causes AMF to immediately release all PDU sessions — the UE has no PS path after NG-RAN → UTRAN CS HO.
- **NCL dual gate** (Annex A): SRVCC handover is only attempted when BOTH (1) "SRVCC operation possible" = TRUE (signalled by MME/eNB) AND (2) a QCI=1 bearer exists (confirming an active IMS voice session). Either gate alone is insufficient.

Next chunk: TS 23.216 COMPLETE. All chunks 5b-1, 5b-2, 5b-3 written. Wiki now covers the full SRVCC specification.
Pages touched: [wiki/procedures/SRVCC-from-E-UTRAN.md (updated — Part 3 §6.5 added), wiki/concepts/SRVCC.md (updated — §6.4 CS-to-PS flows, §8 HO failure, §9 security, Annex A NCL added), wiki/sources/ts23216.md (updated — chunk 5b-3 added, all sections marked Done, COMPLETE declared), wiki/index.md (updated)]
