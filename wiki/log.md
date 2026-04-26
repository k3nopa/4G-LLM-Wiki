# Wiki Log
_Append-only event history. Parse with: `grep "^## \[" log.md | tail -10`_

---

## [2026-04-19] ingest | TS 24.301 §5.3–§5.5 — Elementary + Common + Specific EMM Procedures [chunk ts24301-2]

Ingested §5.3 (elementary EMM procedures), §5.4 (common EMM procedures), and §5.5 (specific EMM procedures: Attach, Detach, TAU) from 3GPP TS 24.301 v17.6.0.

§5.3 key content: NAS connection release (T3440, 11 trigger cases); CIoT suspend/resume; forbidden TA list management (two lists, ≥40 TAIs each, causes #12/#13/#15); T3412 periodic TAU timer (started on IDLE entry, stopped on CONNECTED, ISR dual-timer T3412+T3312); T3402 backoff (max attach attempts); T3346 MM congestion timer (started on cause #22, exemptions for emergency/detach/paging/AC11-15); T3448 CP-data congestion (NB-IoT); PSM (T3324 active timer — UE deactivates AS layer on expiry); eDRX negotiation; Service Gap Control (T3447).

§5.4 key content: GUTI reallocation (T3450, 5 retries, collision rules); EPS AKA authentication (RAND/AUTN/RES, 3 failure types: MAC-failure/#20 + T3418, synch-failure/#21 + AUTS + T3420, non-EPS-auth/#26 + T3418; AUTHENTICATION REJECT → USIM invalid + EU3; T3460 retry max 5); Security mode control (SMC sent unciphered+IP; MME resets DL NAS COUNT; UE resets UL NAS COUNT on new native context; SECURITY MODE COMPLETE IP+ciphered with new context; HASH_MME for replay check; reject causes #23/#24; T3460 retry max 5); Identification (T3470, always respond in EMM-CONNECTED); EMM Information (network→UE only, no response).

§5.5 key content: EPS Attach — 5 attach purposes; attach attempt counter; identity selection (GUTI/P-TMSI-mapped/IMSI/IMEI rules); ESM container; common procedures during attach; ATTACH ACCEPT (T3450 + GUTI + TAI list + default bearer activation); ATTACH COMPLETE → EU1 UPDATED; reject cause table (15 causes with specific EU/state transitions); abnormal cases (T3410 timeout, counter < 5 → T3411 retry, counter = 5 → T3402 + PLMN-SEARCH); combined attach (T3450 + TMSI + LAI). Detach — UE-initiated (T3421, switch-off vs not, EPS/combined/IMSI types, 5-retry rule, local bearer deactivation); network-initiated (T3422, re-attach-required/not-required/IMSI-detach, detailed EMM cause handling). TAU — 33-case trigger list (a-zg); T3430 supervision; GUTI realloc in ACCEPT; EPS bearer context status sync; ISR activation in EPS update result; reject cause table (adds #9/#10/#40 vs. attach); combined TAU (TMSI + LAI); T3450 for TMSI.

Next chunk: ts24301-3 — §5.6 + §6 (Service Request + ESM procedures)

Pages touched: [procedures/NAS-attach.md, procedures/NAS-detach.md, procedures/NAS-TAU.md, protocols/NAS-EMM-protocol.md, sources/ts24301.md, index.md]

---

## [2026-04-19] ingest | TS 24.301 §4 + §5.1–5.2 — General NAS / UE Modes / NAS Security / EMM State Machines [chunk ts24301-1]

New source: 3GPP TS 24.301 v17.6.0 "NAS Protocol for EPS; Stage 3". This is the stage-3 companion to TS 23.401 (EPC architecture). It specifies the exact NAS message procedures, state machines, security mechanisms, and IE encoding for EMM and ESM protocols between UE and MME.

**§4 General:** Two NAS sublayers: EMM (mobility management) and ESM (session management). During EMM procedures MME suspends ESM (except for Attach and Service Request). UE mode of operation: 4 modes — PS mode 1/2 (voice/data centric, EPS-only registration) and CS/PS mode 1/2 (combined EPS+non-EPS registration). Mode transitions driven by 4 trigger categories (usage setting, voice domain preference, IMS registration status, SMS config). NAS signalling low priority (Device properties IE). NAS security: native vs mapped EPS security contexts, eKSI (3-bit key set identifier), NAS COUNT (8-bit SN + 16-bit overflow = 24-bit), integrity (all messages post-SMC; exception table of 7 messages the UE accepts without IP; exception table for MME), ciphering (ATTACH REQUEST + TAU REQUEST always unciphered; CONTROL PLANE SERVICE REQUEST partially ciphered), replay protection, COUNT wrap-around (re-key or activate non-current context). E-UTRA / NB-IoT capability disable/re-enable rules. NB-S1 NAS timer adjustment (+240s on EMM timers, +180s on ESM timers).

**§5.1 EMM overview:** 3 procedure categories: common (GUTI realloc/auth/SMC/ID/EMM-info — network-initiated), specific (attach/detach/TAU — at most one UE-initiated at a time), connection management (service request/paging/NAS transport — S1 mode only). Full UE EMM state machine: 7 main states + 8 EMM-DEREGISTERED substates + 9 EMM-REGISTERED substates. MME state machine: 4 states. EPS update status: EU1 (UPDATED), EU2 (NOT UPDATED), EU3 (ROAMING NOT ALLOWED). EMM–GMM coordination for dual-mode UEs and ISR.

**§5.2:** Detailed UE behaviour descriptions per substate — entry conditions, allowed actions, constraints.

Key findings: ATTACH REQUEST is always unciphered (first message, no security context yet). NAS COUNT has only 8-bit SN transmitted in messages; receiver estimates full 24-bit COUNT. Mapped EPS security context (from UTRAN/GERAN) has KSI_SGSN type. Secure exchange of NAS messages is re-established after idle-mode service request differently from active-mode (lower-layer bearer setup triggers security resume). eCALL-INACTIVE substate suspends all NAS signalling until eCall trigger.

Next chunk: ts24301-2 — §5.3–§5.5 (Elementary EMM procedures + common procedures: Auth, SMC, GUTI realloc, ID + specific procedures: Attach, Detach, TAU)
Pages touched: [wiki/protocols/NAS-EMM-protocol.md (NEW), wiki/concepts/EMM-ECM-states.md (updated — stage-3 substates added), wiki/sources/ts24301.md (NEW), wiki/index.md (updated — 126 pages)]

---

## [2026-04-19] ingest | TS 23.214 §6.5 + §7 — Sx Association Management + Parameter Tables [chunk ts23214-5] ★ COMPLETE

Ingested §6.5 (Sx Association Setup/Update/Release: CP or UP can initiate setup; CP-only release; prerequisite for all Sx sessions) and §7 (parameter tables: PDR full attribute table including detection field combinations per node/direction; URR with all 12 trigger types and applicability per node; FAR including buffering/end-marker/DSCP attributes; QER including APN-AMBR correlation, gate status, MBR/GBR, packet rate; Usage Report structure; §7.8 functional description of how CP constructs rule sets at PDN/bearer/measurement-key aggregation levels; §7.9.1 PFD management parameters). TS 23.214 ingest is now **COMPLETE**.

Key findings: Bearer ID never sent to UP — CP maintains Bearer ID ↔ Sx rule mapping internally. APN-AMBR enforced via shared QER correlation ID across all PDN connections to same APN. PFD provisioning for an App ID is atomic (replaces all existing PFDs). Sx Association Release is CP-only.

Next chunk: None — TS 23.214 ingest complete.
Pages touched: [wiki/concepts/CUPS-parameter-reference.md (NEW), wiki/procedures/Sx-session-management.md, wiki/sources/ts23214.md, wiki/index.md]

---

## [2026-04-19] ingest | TS 23.214 §6.3.2–§6.4 — PCC/Non-3GPP/SGSN Updates + Sx Reporting [chunk ts23214-4]

Ingested §6.3.2 (TS 23.203 IP-CAN session / TDF interactions: establishment, termination, PCC rule modification, PFD management), §6.3.3 (TS 23.402 non-3GPP access: S2b/S2a establishment/release, bearer management, access handovers), §6.3.4.2.1–§6.3.4.3 (TS 23.060 SGSN procedures: S4 and Gn/Gp PDN modification groups 1–3, S4/Gn/Gp PDN connection establishment), and §6.4 (Sx Reporting: session-level reporting with 4 trigger cases: usage, start/stop of traffic detection, first DL for idle-mode UE).

Key findings: Gn/Gp SGSN bypasses SGW entirely (PGW-C↔PGW-U only). Reporting triggers are CP-configured but UP-initiated. Sx Report ACK can extend buffering parameters after first-DL-packet reports.

Next chunk: ts23214-5 — §6.5 + §7 (Sx Association management procedures + parameter tables: PDR/URR/FAR/QER)
Pages touched: [wiki/procedures/Sx-session-management.md, wiki/sources/ts23214.md, wiki/index.md]

---

## [2026-04-19] ingest | TS 23.214 §6.2–§6.3.1 — Sx Session Management + TS 23.401 Procedure Updates [chunk ts23214-3]

Ingested §6.2 (Sx Session Management: Establishment/Modification/Termination) and §6.3.1 (Updates to TS 23.401 procedures: §6.3.1.1–§6.3.1.7).

**§6.2 Sx session procedures:** Three procedures, all CP-initiated. Establishment: CP assigns Session ID, sends PDRs/FARs/URRs/QERs to UP, UP responds with allocated F-TEIDu(s). Modification: UP identifies session by Session ID and updates parameters. Termination: UP removes entire session context and may include final usage reports in response. For TDF in unsolicited reporting mode, establishment/modification triggered by TDF configuration (no peer MME/PCRF interaction).

**§6.3.1 TS 23.401 procedure mappings (7 sub-cases):**

1. **PDN connection establishment:** New SGW-C/PGW-C each establish Sx sessions at new SGW-U/PGW-U. Old CP functions terminate Sx at old UP functions. SGW-C performs two Sx modifications to update SGW-U with (a) PGW-U F-TEIDu and (b) eNB F-TEIDu. PGW-C does additional modification if Handover Indication present (start routing DL to new SGW-U).

2. **SGW change Type 1** (TAU/RAU/X2-HO/MME-relocation with SGW change — Create Session Request only): new SGW-C establishes Sx + gets all-bearer F-TEIDu; PGW-C modifies PGW-U with new SGW-U F-TEIDu; old SGW-C terminates.

3. **SGW change Type 2** (S1 HO + inter-RAT HOs — also Modify Bearer Request): additionally SGW-C modifies SGW-U with eNB F-TEIDu before PGW-C updates PGW-U. NOTE: indirect data forwarding tunnel support in CUPS is FFS (Editor's Note in spec).

4. **eNB F-TEIDu update** (Service Request, Connection Resume, E-RAB mod, X2/S1 HO no SGW change): SGW-C modifies SGW-U with new eNB F-TEIDu. For Service Request with buffering in SGW-C: SGW-C also pushes still-valid buffered packets to SGW-U.

5. **eNB F-TEIDu release** (Connection Suspend, S1 Release): SGW-C modifies SGW-U: release eNB F-TEIDu + configure buffering mode. If CP buffering: SGW-U FAR set to forward DL to SGW-C. If UP buffering: configure buffer behavior + optional DSCP indication + optional drop threshold.

6. **DL data buffered in SGW-U** (Network Triggered Service Request, PGW Pause of Charging): SGW-U sends Sx Report to SGW-C on first DL packet (with DSCP if Paging Policy Diff. active) or on drop threshold. SGW-C acks with optional extended buffering parameters.

7. **PDN connection release** (Detach variants, PDN disconnection): SGW-C first modifies SGW-U (stop counting/forwarding), then PGW-C terminates Sx at PGW-U (with final usage reports), then SGW-C terminates Sx at SGW-U (with final usage reports).

8. **Bearer modification** (dedicated bearer activation/modification/deactivation): PGW-C and SGW-C each modify their respective UP functions — allocate new F-TEID for activation, or stop counting/forwarding for deactivation.

Notable findings:
- **Indirect data forwarding tunnel in CUPS is FFS:** the spec has an Editor's Note flagging that S1 handovers requiring indirect forwarding tunnels are not yet fully specified for CUPS. This is a known gap.
- **Usage reports are piggybacked on Sx Session Termination Response:** the UP does not send a separate final report; final usage data is included in the termination response itself — no separate "flush" step needed.
- **SGW-C-buffered packets are pushed to SGW-U on Service Request:** when buffering was in SGW-C and UE transitions to ECM-CONNECTED, SGW-C proactively pushes still-valid packets to SGW-U in the same Sx Session Modification that provides the new eNB F-TEIDu — a single round-trip covers both path update and data handoff.

Next chunk: ts23214-4 — §6.3.2–§6.4 (TS 23.203 PCC updates, TS 23.402 non-3GPP updates, TS 23.060 SGSN updates, Sx reporting procedures)
Pages touched: [wiki/procedures/Sx-session-management.md (created), wiki/interfaces/Sx.md (created), wiki/sources/ts23214.md (updated), wiki/index.md (updated — 123 pages)]

---

## [2026-04-19] ingest | TS 23.214 §5 — CUPS Functional Description [chunk ts23214-2]

Ingested §5 (Functional description): §5.2 traffic detection/PDRs, §5.3 charging/URRs, §5.4 GTP-U TEID allocation, §5.5 UE IP address management, §5.6 forwarding/FARs, §5.7 UE permanent identifier, §5.8 end markers, §5.9 idle-state SGW buffering, §5.10 bearer/APN policing, §5.11 PCC/ADC functions, §5.12 UP function selection.

**§5.2 (PDRs):** 8 PDR usage scenarios defined for SGW-U/PGW-U/TDF-U (Table 5.2.2-1). Detection info: UE IP, F-TEIDu, SDF filters, App ID. Lowest-precedence PDRs catch remaining/unmatched traffic.

**§5.3 (Charging):** All charging interfaces (Gx/Sd/Gy/Gz/OFCS) stay at CP function. UP provides only raw usage counts via URRs (Usage Reporting Rules) keyed to Monitoring Keys or Charging Keys. Three reporting triggers: periodic, threshold, on-demand. PGW Pause of Charging: SGW-C adds buffered-packet measurement URR to SGW-U; on threshold, PGW-C stops/resumes all URRs via Sx session modification.

**§5.4 (TEID):** F-TEIDu allocation is mandatory in UP function (SGW-U/PGW-U manage their own TEID spaces). CP requests allocation over Sx and distributes results to peer nodes.

**§5.5 (UE IP):** All UE IP management is PGW-C. PGW-U relays DHCPv4/v6/RS/RA messages between UE and PGW-C. If external AAA/DHCP reachable only via PGW-U, PGW-C instructs PGW-U to forward.

**§5.6 (FARs):** 5 forwarding scenarios (Table 5.6.2-1): GTP-U encapsulation, traffic→CP (redirect/RADIUS/DHCP), buffered→CP, traffic steering (FMSS). UP→CP uses Sx-u GTP-U tunnel (per TS 29.244). CP can request multiple sequential forwarding actions.

**§5.8 (End markers):** Two constructors (UP or CP) — network-configured. UP constructor uses Sx indication; CP constructor waits for Sx confirmation before constructing and sending to UP for forwarding.

**§5.9 (SGW buffering):** SGW-U buffering mandatory; SGW-C optional (except NB-IoT). Configurable per UE session. Delay DDN, extended buffering, and throttling each have a "handled by SGW-U" vs "handled by SGW-C" mode with different Sx mechanisms.

**§5.10 (Policing):** ARP not sent to UP. QCI + optional ARP priority → transport marking. PGW-C sends APN-AMBR + QER correlation ID to PGW-U for cross-session APN-AMBR enforcement.

**§5.11 (PCC/ADC):** Detection filters split between CP (SDF) and UP (App ID). CP maintains Gx/Sd rule ↔ Sx PDR mapping. PFDs managed by CP; conflict risk if single UP controlled by multiple CPs for same App ID.

**§5.12 (UP selection):** CP selects UP based on load, capacity, UE location, capability match, and UE features. Same PGW-U required for same UE+APN when APN-AMBR enforcement active. Combined SGW/PGW-C selects co-located UP pair to avoid unnecessary S5/S8 hop.

Next chunk: ts23214-3 — §6.2–§6.3.1 (Sx session management + TS 23.401 EPS procedure updates)
Pages touched: [wiki/concepts/CUPS-architecture.md (updated — §5 sections added), wiki/sources/ts23214.md (updated — chunk 2 Done), wiki/index.md (updated)]

---

## [2026-04-19] ingest | TS 23.214 §1–§4 — CUPS Architecture Model and Functional Splits [chunk ts23214-1]

New source: 3GPP TS 23.214 v16.1.0 "Architecture enhancements for control and user plane separation of EPC nodes" (CUPS). This spec defines how SGW, PGW, and TDF are each split into a Control Plane (CP) function and a User Plane (UP) function, connected by a new Sx reference point family (Sxa/Sxb/Sxc).

**§3 Abbreviations:** SGW-C/SGW-U, PGW-C/PGW-U, TDF-C/TDF-U, CP function, UP function, F-TEID/F-TEIDu.

**§4.1 General concepts:** CUPS is transparent to UE and RAN. Interworking with non-split networks is supported. CP function selects its UP function. One CP can control multiple UPs (1:N). CUPS only supported with GTP-based S2a/S2b/S5/S8 (no PMIP, no S2c). TDF-C selected by config in PGW or PCRF.

**§4.2 Architecture reference model:** Three new Sx reference points: Sxa (SGW-C↔SGW-U), Sxb (PGW-C↔PGW-U), Sxc (TDF-C↔TDF-U). Existing reference points get -C/-U suffixes. S11-U is a new interface between MME and SGW-U for CP CIoT EPS Optimisation. Combined SGW/PGW uses a combined Sxa/Sxb interface.

**§4.3.2 Functional split tables (key findings):**
- **SGW-U** is a pure forwarding engine: packet forwarding, transport marking, DL buffering, end marker generation, per-UE/bearer accounting. No PCC state.
- **PGW-C** makes bearer binding _decisions_ (has QoS/TFT policy from PCRF via Gx). **PGW-U** performs bearing binding _verification_ and DL traffic mapping — DPI (service detection) is PGW-U only; CP never processes user data.
- **TDF-U** APN-AMBR scope is narrower than PGW's: covers flows in the current TDF session only, not all PDN connections of the UE to the same APN.
- Event reporting split: user-plane events (app detection) reported by UP function; control-plane events (RAT change) by CP function only.

**§4.3.3 UP Function selection:** CP selects UP based on UE location, UP capability, required features, and deployment mode (central vs. distributed/RAN-site-local). Details in §5.12.

**§4.3.4 SGW-C Partitioning:** SGW-C can be partitioned when SGW-U service area < SGW-C service area. Each partition treated as a legacy SGW by MME. One Sxa per SGW-C↔SGW-U pair. TAI List scoped to TAs with connectivity to the specific SGW-U.

Next chunk: ts23214-2 — §5 Functional description (traffic detection, charging, GTP-U TEID allocation, buffering, PCC/ADC, UP function selection)
Pages touched: [wiki/concepts/CUPS-architecture.md (created), wiki/sources/ts23214.md (created), wiki/index.md (updated — 121 pages)]

---

## [2026-04-19] ingest | TS 29.165 §8–§12 — Numbering/Addressing, Charging, 26 Supplementary Services [chunk ts29165-2]

Ingested §8 (Numbering/Naming/Addressing), §9 (IP Version), §10 (Security), §11 (Charging/IOI), and §12 (26 MMTEL supplementary services over II-NNI). All content added to the existing `interfaces/II-NNI.md` page.

**§8 Addressing:** SIP URI mandatory at all II-NNI scenarios; tel URI (global E.164) mandatory at roaming II-NNI and loopback. URI parameters: `sos` (roaming-mandatory), `oli`/`cpc`, `rn`/`npdi` (number portability), `isub`, `premium-rate` (all per agreement except `sos`). SDP: MSRP URI required at roaming II-NNI.

**§11 Charging:** P-Charging-Vector IOI rules: type 1 (roaming II-NNI — visited network identities), type 2 (home network II-NNI between two home networks or LBO — home network identities), transit-ioi for transit scenarios. Network identifiers must be pre-agreed. Tariff information via `application/vnd.etsi.sci+xml` in INVITE/18x/INFO per bilateral agreement.

**§12 Supplementary services:** 26 services documented with their II-NNI SIP requirements. All support conditional on bilateral operator agreement. Key findings:
- **OIP/OIR**: P-Asserted-Identity is stripped by IBCF on no-trust; trust relationship determines whether it passes through
- **CDIV**: History-Info header with `mp` parameter and `cause` URI parameter is the II-NNI mechanism for preserving diversion history
- **CONF**: REFER is mandatory at roaming II-NNI but only optional at non-roaming (key asymmetry); `isfocus` parameter + `conference` event package needed
- **ECT**: Assured transfer adds `method=CANCEL` in Refer-To header — distinct from blind/consultative transfer mechanism
- **AOC**: 504 Server Time-out with Reason SIP cause=504 or Q.850 cause=31 is the specified tariff failure code at II-NNI

Notable findings:
- **IOI type asymmetry between roaming and home II-NNI**: roaming II-NNI uses type 1 IOI (visited-network granularity), home-network II-NNI uses type 2 IOI (home-network granularity). The distinction matters for inter-operator billing reconciliation — type 2 gives finer-grained origin/destination identification needed for direct home-home peering.
- **26 supplementary services are all bilateral**: none is mandated unilaterally. This is consistent with the IMS inter-operator model where home operators can choose what services to expose at their II-NNI border. The IBCF can suppress any service not agreed upon.

Next chunk: ts29165-3 — §13–§33 + Annex A (ICS, SRVCC continuity, presence, messaging, OMR, IUT, P-CSCF restoration, emergency, RLOS, SIP header summary)
Pages touched: [wiki/interfaces/II-NNI.md (updated — §8-§12 added), wiki/sources/ts29165.md (updated — chunk 2 Done), wiki/index.md (updated)]

---

## [2026-04-19] ingest | TS 23.203 Annex A.4 + A.5 + Annex H — EPC access-specific PCC [chunk ts23203-5]

Ingested the final chunk of TS 23.203: Annex A.4 (3GPP GTP-based EPC), Annex A.5 (3GPP PMIP-based EPC), Annex H (EPC-based non-3GPP access: HRPD, Trusted WLAN S2a, Untrusted non-3GPP/ePDG S2b). Also noted Annex G (PCC rule precedence), Annex K/L (Limited PCC Deployment).

**Annex A.4 (GTP-EPC):** PCEF at PDN-GW; no BBERF; Case 1. PCEF performs bearer binding. Default EPS Bearer QoS (QCI+ARP) authorized by PCRF via Gx — Table A.4.3.4-1 supports up to 4 Subsequent QoS instances with future change times. EPS credit re-auth triggers (Table A.4.3-1): SGSN change, SGW change, RAT change, location changes (TA/RA/ECGI/CGI-SAI/eNodeB ID), PRA presence, User CSG info change. EPS event triggers (Table A.4.3-2) add: Subscribed APN-AMBR change, EPS Subscribed QoS change, 3GPP PS Data Off. Key rule: modifying Default EPS Bearer QoS simultaneously modifies all PCC rules that share the same QoS as the default bearer.

**Annex A.5 (PMIP-EPC):** PCEF at PDN-GW; BBERF at SGW (Gxc). BBERF performs bearer binding and enforces Default EPS Bearer QoS. Same EPS event triggers apply at BBERF. RAT type change: BBERF→PCRF→PCEF chain. Same Default EPS Bearer QoS cascade rule applies to QoS rules.

**Annex H (EPC-based non-3GPP):**
- **H.2 cdma2000 HRPD**: BBERF in HSGW (Gxa). HSGW can be told via HSS Charging Characteristics not to establish GCS. During EUTRAN→HRPD pre-registration, HSGW is non-primary BBERF; PCRF provides IPv6 prefix to HSGW to link GCS to existing Gx session. BCM must be same value across all simultaneous UE-PDN IP-CAN sessions.
- **H.3 Trusted WLAN S2a**: No BBERF; Case 1. PCEF provides TWAN ID, Time Zone, trusted-access indication. Single trigger: RAT type change. TWAN Release Cause at termination.
- **H.4 Untrusted non-3GPP (ePDG)**: No BBERF; Case 1. PCEF provides ePDG IP, Serving Network, untrusted indication. Location includes ePDG IKEv2 IP + UE local IP/port if NAT. PCRF forwards ePDG IP to P-CSCF. Warning: UE local IP not reliable (spoofable). UWAN Release Cause at termination.

Notable findings:
- **Default EPS Bearer QoS cascade** (A.4.4.4.2 / A.5.4.4): modifying the default bearer QoS implicitly modifies all rules sharing that QoS — this avoids a scenario where the default bearer QCI changes but associated PCC rules retain the old QCI, creating a mismatch between bearer parameters and rule parameters.
- **HSGW GCS suppression via HSS Charging Characteristics** (H.2): the only case in the spec where a non-3GPP gateway can be configured to skip PCRF interaction entirely via a subscriber-profile indicator. This reduces Gxa signalling load for HRPD operators who do not need per-session dynamic QoS.
- **ePDG IP address reliability caveat** (H.4): the spec explicitly notes that the UE local IP address as reported by ePDG is not reliable (UE can spoof). This affects IMS P-CSCF behavior when the AF receives access network info.

TS 23.203 is now **COMPLETE** — all 5 chunks ingested.

Next chunk: No next chunk for TS 23.203. Next planned work: TS 23.203 is complete; remaining Phase 3 sources TBD or Phase 4 deep dives.
Pages touched: [wiki/concepts/PCC-EPC-access-specifics.md (created), wiki/sources/ts23203.md (updated — ts23203-5 Done, COMPLETE), wiki/index.md (updated — 119 pages)]

---

## [2026-04-14] ingest | TS 29.329 §6.3–§6.4 + §7 — Sh AVPs, Namespaces, Version Control [chunk ts29329-2]

Ingested all 35 AVP definitions (§6.3.1–§6.3.35) plus re-used AVPs, namespace assignments (§6.4), and version control requirements (§7). TS 29.329 is now COMPLETE.

**§6.3 — AVP master table:** 35 Sh-native AVPs (codes 700–722 range, Vendor-ID 10415) plus 4 IETF AVPs and 1 re-used 3GPP AVP. Key structural AVPs: User-Identity (700, Grouped — contains Public-Identity | MSISDN | External-Identifier), Repository-Data-ID (715, Grouped — contains Service-Indication + Sequence-Number for concurrency control), Call-Reference-Info (720, Grouped — correlates with CAMEL/MAP call reference).

**§6.3.4 Data-Reference enumeration — 35 values (0–35, value 20 reserved):** RepositoryData(0), IMSPublicIdentity(10), IMSUserState(11), S-CSCFName(12), InitialFilterCriteria(13, AS reads its own iFC), LocationInformation(14), UserState(15), ChargingInformation(16), MSISDN(17), PSIActivation(18), DSAI(19), ServiceLevelTraceInfo(21), IPAddressSecureBindingInformation(22), ServicePriorityLevel(23), SMSRegistrationInfo(24), UEReachabilityForIP(25, supports One-Time-Notification), TADSinformation(26), STN-SR(27), UE-SRVCC-Capability(28), ExtendedPriority(29), CSRN(30), ReferenceLocationInformation(31), IMSI(32), IMSPrivateUserIdentity(33), IMEISV(34), UE-5G-SRVCC-Capability(35).

**§6.3.7A Requested-Nodes bitmask:** bit0=MME, bit1=SGSN, bit2=3GPP-AAA-SERVER-TWAN, bit3=AMF — allows a single UDR/SNR to request data for multiple serving node types simultaneously.

**§6.4 — Namespaces:** Experimental-Result-Codes 4100–4101 (transient) and 5100–5105 (permanent) assigned; Command codes 306–309 from IANA (RFC 3589); Application-ID 16777217 (IANA).

**§7 — Version control:** UDR/PUR/SNR retransmitted with same Session-Id on timeout (per TS 29.229 timer). Repository data uses Sequence-Number for optimistic locking.

Notable findings:
- **Data-Reference 13 (InitialFilterCriteria) is read-only for AS**: the value note says "This value is used to request initial filter criteria relevant to the requesting AS". The AS receives its own iFC slice but cannot modify it via Sh — iFC changes must go through operator provisioning (HSS management interface). This is a key security boundary.
- **Requested-Nodes bitmask (§6.3.7A) unifies multi-RAT queries**: a single UDR can request LocationInformation from MME (LTE), SGSN (UMTS/GPRS), 3GPP-AAA-SERVER-TWAN (WLAN), and AMF (5G) simultaneously. The HSS aggregates available data from all indicated nodes. This is more efficient than four separate UDRs and reduces Sh round-trips in multi-RAT deployments.
- **UDR-Flags bitmask (§6.3.28) connects Sh to EPS bearer layer**: bit0 (Location-Information-EPS-Supported) signals to the HSS that the AS understands EPS-format location data (TAI, ECGI). Without this flag the HSS would return pre-LTE format location. Bit1 (RAT-Type-Requested) asks the HSS to include the RAT type in the LocationInformation response. These flags exist because Sh predates LTE; the flags are backwards-compatibility shims for HSS implementations.
- **Call-Reference-Info (§6.3.29) bridges Sh into CAMEL/CS territory**: the Call-Reference-Number + AS-Number grouped AVP allows an IMS AS (e.g. TAS acting as MSRN anchor) to correlate an Sh UDR with an ongoing CS call reference at the GMSC. This is the mechanism by which IMS-based services can query subscriber data in the context of a CAMEL-controlled CS call, enabling seamless CS/IMS service coordination.
- **DSAI-Tag (§6.3.18) + Data-Reference 19 (DSAI) form a feedback loop with S-CSCF iFC evaluation**: AS uses PUR to write a new DSAI value (active/inactive) → HSS stores it → next S-CSCF iFC evaluation reads DSAI SPT → trigger or skip AS. This is the only mechanism in 3GPP IMS for an AS to dynamically control whether it is included in future call flows, without touching the iFC XML itself.

TS 29.329 is now **COMPLETE** — all 2 chunks ingested.

Next chunk: No next chunk for TS 29.329. Remaining undigested source: TS 29.328 (Sh signalling flows and message contents) if needed, or proceed with Phase 4 deep dives.
Pages touched: [wiki/protocols/Sh-Diameter.md (updated — §6.3 AVP table + Data-Reference enumeration + namespace section added), wiki/sources/ts29329.md (updated — ts29329-2 Done, COMPLETE), wiki/index.md (updated)]

---

## [2026-04-14] ingest | TS 29.329 §1–§6.2 — Sh Diameter Commands + Result Codes [chunk ts29329-1]

New source: 3GPP TS 29.329 v16.2.0 "Sh interface based on the Diameter protocol". This spec defines the Diameter wire protocol for the Sh reference point between AS/SCS and HSS. Companion to TS 29.328 (which defines signalling flows and message contents).

**§1 Scope:** Sh interface protocol for IMS CN. Applicable to AS↔HSS and SCS↔HSS. Based on IETF RFC 6733 with 3GPP extensions. Vendor-ID = 10415, Application-ID = 16777217.

**§6.1 — 8 Command Codes in 4 pairs:**
- UDR/UDA (306): AS→HSS pull of user profile data (one or more Data-References per request). UDR carries User-Identity + *Data-Reference as mandatory; optional fields include Wildcarded-Public-Identity, Wildcarded-IMPU, Server-Name, *Service-Indication, *DSAI-Tag, Requested-Nodes, Serving-Node-Indication, UDR-Flags, Call-Reference-Info. UDA returns User-Data AVP.
- PUR/PUA (307): AS→HSS update of transparent or non-transparent data. PUR carries User-Identity + *Data-Reference + User-Data (mandatory). Multiple Data-Reference AVPs permitted only when Update-Eff-Enhance feature is negotiated. PUA returns Repository-Data-ID on successful Repository Data update.
- SNR/SNA (308): AS→HSS subscription to data-change notifications. SNR carries Subs-Req-Type (0=Subscribe, 1=Unsubscribe). Expiry-Time in SNA sets subscription TTL. One-Time-Notification flag for single-fire subscriptions. Send-Data-Indication causes SNA to include current data snapshot.
- PNR/PNA (309): HSS→AS push notification (HSS is Diameter client). PNR has mandatory Destination-Host (unicast) and mandatory User-Data (changed data always included). PNA acknowledges; AS can return error codes (e.g. 5107 if not subscribed).

**§6.2 — Result codes:** 12 permanent failures (5xxx in Experimental-Result), 2 transient failures (4xxx). Key design decisions: TRANSPARENT_DATA_OUT_OF_SYNC (5105) enforces optimistic concurrency on Repository Data — AS must include current Sequence-Number in PUR; HSS rejects if stale. PRIOR_UPDATE_IN_PROGRESS (4101) is transient (retry-able), not permanent. DSAI_NOT_AVAILABLE (5108) ties to DSAI-Tag feature for dynamic service activation control.

Notable findings:
- **PNR role reversal is architecturally significant:** the HSS acts as Diameter client when pushing notifications. This means the HSS must maintain persistent Diameter connections to every subscribed AS, which has operational implications for HSS capacity and connection management. Unlike most IMS interfaces where the HSS is always the server, Sh requires the HSS to initiate TCP/SCTP connections.
- **Sequence-Number concurrency control on Repository Data** means the Sh interface has a built-in optimistic locking mechanism. If two ASes simultaneously update the same Repository Data (e.g. presence state), one will win and the other will receive 5105. The loser must re-read before retrying. This is the only data type on Sh with this constraint — non-transparent HSS data (iFC, charging info) does not use sequence numbers.
- **One-Time-Notification in SNR** is useful for "watch once" patterns: AS subscribes, gets a single PNR when data changes, subscription auto-cancels. This avoids the AS needing to explicitly unsubscribe and reduces HSS subscription table churn for short-lived data queries.
- **DSAI (Dynamic Service Activation Information)** ties Sh directly to the iFC evaluation in S-CSCF: the DSAI-Tag in UDR/SNR requests the activation state of a DSAI boolean flag in the HSS subscriber data. S-CSCF evaluates DSAI-Tag SPTs by querying this state. An AS managing its own activation state therefore uses Sh (PUR) to flip the DSAI bit, and the next S-CSCF iFC evaluation will reflect the change — no S-CSCF signalling needed.

Next chunk: ts29329-2 — §6.3–§6.4 + §7: AVP definitions (35 AVPs: User-Identity, MSISDN, User-Data, Data-Reference, Service-Indication, Subs-Req-Type, Repository-Data-ID, Expiry-Time, DSAI-Tag, Wildcarded-IMPU, UDR-Flags, etc.), namespace assignments, version control
Pages touched: [wiki/protocols/Sh-Diameter.md (new), wiki/sources/ts29329.md (new), wiki/index.md (updated, 82→84 pages)]

---

## [2026-04-12] ingest | TS 32.299 §6.3–§6.5 — Diameter Online Charging (Ro interface) [chunk ts32299-3]

Ingested §6.3 (Ro protocol flows): §6.3.2 IEC — single CCR[EVENT_REQUEST, RA=DIRECT_DEBITING, RSU] → CCA[EVENT, GSU, Cost-Info, Remaining-Balance]; IEC Refund variant: RA=REFUND_ACCOUNT; §6.3.3 ECUR — CCR[INITIAL, RSU] → CCA[INITIAL, GSU, VT] → service → CCR[TERMINATION, USU] → CCA[TERMINATION, Cost-Info]; §6.3.4 SCUR — CCR[INITIAL, RSU] → CCA[INITIAL, GSU, VT, QHT] → {CCR[UPDATE, USU+RSU] → CCA[UPDATE, GSU, FUI]}* → CCR[TERMINATION, USU] → CCA[TERMINATION]; debit+reserve combined in single UPDATE CCR (one round-trip per cycle).

Ingested §6.4 (Ro message formats): §6.4.2 CCR format (272 REQ PXY): M: Session-Id, Origin-Host/Realm, Destination-Realm, Auth-Application-Id=4, Service-Context-Id, CC-Request-Type, CC-Request-Number; Om: Subscription-Id (Type+Data: IMSI/MSISDN), Multiple-Services-Indicator, Service-Information; Oc: Multiple-Services-Credit-Control (MSCC grouped AVP containing RSU/USU, Service-Id, Rating-Group, G-S-U-Pool-Reference, VT, FUI, Result-Code, QHT, QCT, Reporting-Reason, Trigger, Time/Volume/Unit-Quota-Threshold), Requested-Action (IEC only: DIRECT_DEBITING/REFUND_ACCOUNT/CHECK_BALANCE/PRICE_ENQUIRY), Termination-Cause, User-Equipment-Info, CC-Correlation-Id, QoS-Information, OC-Supported-Features; §6.4.3 CCA format (272 PXY): M: Session-Id, Result-Code, Origin-Host/Realm, Auth-Application-Id=4, CC-Request-Type, CC-Request-Number; Oc: MSCC (GSU with CC-Time/CC-Total/Input/Output-Octets/CC-Service-Specific-Units, Tariff-Time-Change, VT, FUI, per-MSCC Result-Code, quota thresholds, QHT, QCT, Reporting-Reason, Trigger), Cost-Information, Low-Balance-Indication, Remaining-Balance, Credit-Control-Failure-Handling (TERMINATE/CONTINUE/RETRY_AND_TERMINATE), Direct-Debiting-Failure-Handling, CC-Session-Failover, OC-OLR, Redirect-Host, Service-Information.

Ingested §6.5 (quota management procedures): idle timeout via Quota-Holding-Time; Validity-Time expiry requiring CCR-UPDATE; re-auth trigger framework (Trigger/Trigger-Type AVP list: SGSN IP change, QoS change, location change, RAT change, etc.); Reporting-Reason enum (THRESHOLD, QHT, FINAL, QUOTA_EXHAUSTED, VALIDITY_TIME, OTHER_QUOTA_TYPE, RATING_CONDITION_CHANGE, FORCED_REAUTHORISATION, POOL_EXHAUSTED); threshold-based re-auth (Time/Volume/Unit-Quota-Threshold); Final-Unit-Indication (FUA: TERMINATE/REDIRECT/RESTRICT_ACCESS); Quota-Consumption-Time for CTP/DTP combinational quota; envelope reporting (online and offline envelopes, Offline-Charging AVP bridge); OCS-initiated RAR/ASR actions; Credit-Control-Failure-Handling and CC-Session-Failover for OCS unreachable scenarios. §6.6 AVP bindings tables (6.6.1.1 offline; 6.6.2.1 online) mapping logical fields to Diameter AVPs.

Notable findings:
- The MSCC (Multiple-Services-Credit-Control) grouped AVP is the design center of Ro — it makes the CCR/CCA exchange effectively a multi-service quota management protocol in a single message. One CCR UPDATE can simultaneously debit multiple old service quotas and request multiple new ones, in a single Diameter round-trip.
- Quota-Consumption-Time introduces time-as-volume semantics: a session consuming zero bytes still burns time quota at rate 1 QCT-period per unit. This is critical for voice (VoLTE) where the "volume" being charged is airtime, not bytes.
- Final-Unit-Indication is the OCS's policy enforcement hook: when it signals REDIRECT rather than TERMINATE, the network becomes a paywall — the UE stays connected but is redirected to a top-up portal. This is the architectural basis for zero-rating portals.
- CCFH=CONTINUE is a deliberate revenue-loss tradeoff: operators allow service to continue when OCS is unreachable (rather than angering subscribers with dropped calls), accepting that some usage won't be charged in exchange for reliability. RETRY_AND_TERMINATE is the compromise.
- The Offline-Charging AVP in CCA is architecturally interesting: it lets the OCS (online path) instruct the CTF on how to perform offline (CDR) charging for the same session — effectively making online charging the master configuration channel for offline CDR generation.

Next chunk: ts32299-4 — TS 32.299 §7: Key AVPs (selective — IMS/EPS-relevant AVPs, grouped AVP structures, enum values)
Pages touched: [wiki/protocols/Ro-online-charging.md (new), wiki/sources/ts32299.md (updated), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 32.299 §6.1–§6.2 — Diameter Offline Charging (Rf) [chunk ts32299-2]

Ingested §6.1.0 (introduction: CTF=Diameter accounting client; CDF=Diameter accounting server; RFC 6733 DBPA reused; CDF implements SERVER STATELESS ACCOUNTING so no ordering constraint on ACR arrival; CDF supervision timer started on first ACR[START], reset on INTERIM, stopped on STOP; expiry closes CDR with error indication; ACR record types: START/INTERIM/STOP for sessions, EVENT for one-shots); §6.1.1 (event-based: single ACR[EVENT_RECORD]→ACA, operation may occur before/during/after service delivery); §6.1.2 (session-based: ACR[START]→ACA[AII=N]→ACR[INTERIM]*→ACR[STOP]; CDF opens CDR on START, updates on INTERIM, closes on STOP; AII=Acct-Interim-Interval in ACA[START] sets periodic reporting interval); §6.1.3 error cases: §6.1.3.1 CDF connection failure (failover to secondary CDF; if no CDF reachable, buffer ACRs in non-volatile memory; replay in order when CDF reconnects); §6.1.3.2 no reply (CTF retransmits ACR with configurable timer+max; T-flag set on retransmissions; after max retransmissions → connection failure procedure); §6.1.3.3 duplicate detection (T-flag: discard if original already received; if original missing, use T-flagged copy and mark CDR); §6.1.3.4 CDF detected failure (timer expiry → CDR close; operator-configurable behavior); §6.2.1 offline message reference table (ACR/ACA=271, CER/CEA for peer capability; DPR/DPA, DWR/DWA per RFC 6733); §6.2.2 ACR format (CCo=271 REQ PXY; M: Session-Id, Origin-Host/Realm, Destination-Realm, Accounting-Record-Type, Accounting-Record-Number; Om: Acct-Application-Id=3, Service-Context-Id, Service-Information; Oc: User-Name, Destination-Host, Acct-Interim-Interval, Origin-State-Id, Event-Timestamp, Proxy-Info, Route-Record, AVP; NOT used: Vendor-Specific-Application-Id, Sub/RADIUS/Multi-Session-Id, Accounting-Realtime-Required); §6.2.3 ACA format (271 PXY; M: Session-Id, Result-Code, Origin-Host/Realm, Record-Type/Number; Om: Acct-Application-Id=3; Oc: Experimental-Result, User-Name, Error-Message, Error-Reporting-Host, Acct-Interim-Interval, Failed-AVP, Origin-State-Id, Event-Timestamp, Proxy-Info, AVP).

Notable findings:
- CDF SERVER STATELESS ACCOUNTING is architecturally significant: it decouples CTF retry/failover behavior from CDR integrity — CDRs are built from whatever ACRs arrive, with sequence-number gaps surfaced as CDR anomalies rather than session errors. This differs from online charging where ordering and session state matter strictly.
- The T-flag duplicate detection mechanism creates a subtle data quality question: when the original ACR is lost and only the T-flagged retransmission arrives, the CDR is marked but the data itself is used. An operator querying CDRs must understand that "duplicate-sourced" CDRs are still revenue-bearing records.
- Service-Context-Id + Service-Information is the extension point for all service domains. A single Rf protocol stack handles EPS, IMS, SMS, MMS charging identically — only the content of Service-Information differs. This means the CDF sees the same ACR wire format regardless of which service produced the charging event.
- Non-volatile ACR buffering is mandatory semantics for offline charging continuity: a CDF outage should not cause permanent revenue loss. The ordering requirement for buffer replay ensures CDR chronology is preserved.

Next chunk: ts32299-3 — TS 32.299 §6.3–§6.5: Diameter online charging (Ro interface) — CCR/CCA message formats, IEC/ECUR/SCUR protocol flows, re-authorization, quota management (PDF pp 62–93)
Pages touched: [wiki/protocols/Rf-offline-charging.md (new), wiki/sources/ts32299.md (updated), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 32.299 §4–§5 — Charging Architecture + Requirements [chunk ts32299-1]

New source: 3GPP TS 32.299 v16.2.0 "Diameter charging applications". Ingested §4 (logical offline and online charging architecture: CTF→CDF via Rf for offline; CTF→OCF via Ro for online; CGF as CDR aggregator; Ga→BD; CTF address priority lists; same-PLMN charging rule; Single IMSI architecture for EU roaming unbundling; PCEF uses Gy/TDF uses Gyn for PS-domain roaming online charging) and §5 (charging requirements: §5.1 offline — event-based [ACR[Event]] and session-based [ACR[Start+Interim+Stop]]; §5.1.2 Charging Data Request/Response fields including Session-Id, Operation-Type, Operation-Number, Service-Information; §5.2 online — Rating and Unit Determination each centralized/decentralized; constraint: Centralized UD + Decentralized Rating not supported; three cases: IEC immediate debit via Direct Debit One-Time Event [RFC 4006 §6.3.3]; ECUR reserve-then-debit for events; SCUR session-based reserve+debit [RFC 4006 Session-Based] with Debit+Reserve combined in single CCR; reserved≠consumed, CTF can modify/return units; three sub-variants per case per UD/Rating combination: IEC-a Decentral-UD+Central-Rating, IEC-b Central-UD+Central-Rating, IEC-c Decentral-UD+Decentral-Rating; §5.2.3 Debit/Reserve Units Request/Response message fields; §5.3 re-authorization: idle timeout, threshold, mid-session triggers; termination action; account expiration).

Notable findings:
- The CTF is a logical function embedded in every NE that generates charging events — it is not a standalone physical node. The CDF/OCF are separate charging infrastructure nodes. This means every PGW, S-CSCF, P-CSCF, TAS, etc. contains a CTF instance.
- IEC is fundamentally different from ECUR/SCUR in credit risk posture: IEC debits immediately (or concurrently/after), so the operator absorbs any service delivered before the debit if the OCF is unreachable. ECUR/SCUR reserve first, guaranteeing credit before service starts.
- For IMS (VoLTE calls), SCUR is the natural fit: the call is a session, units are time, and the operator needs real-time credit supervision. ECUR applies to IMS short events (e.g. messaging). IEC applies to non-session service lookups.
- The Single IMSI architecture for EU roaming is a cross-PLMN exception to the same-PLMN-only charging rule — a Proxy Function in the VPLMN forwards Ro requests to the HPLMN OCS.

Next chunk: ts32299-2 — TS 32.299 §6.1–§6.2: Diameter offline charging (Rf interface) — ACR/ACA message formats, CDF error handling (PDF pp 52–61)
Pages touched: [wiki/concepts/charging-architecture.md (new), wiki/sources/ts32299.md (new), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 33.203 Annexes H+I+M+N+O+P — Auth Alternatives + Protocol Details [chunk ts33203-2]

Ingested Annex H (RFC 3329 3GPP profile: BNF for Security-Client/Server/Verify SIP headers; mechanism-name="ipsec-3gpp" or "tls"; parameters: alg={hmac-sha-1-96,aes-gmac,null}, ealg={aes-cbc,aes-gcm,null}, mod={trans,tun,UDP-enc-tun}, prot=esp, spi-c/spi-s/port-c/port-s; 3GPP adds aes-gmac/aes-gcm, removes hmac-md5-96/des-ede3-cbc; "ipsec-3gpp" extends RFC 3329: server stores Security-Client from SM1; client includes both Security-Verify+Security-Client in SM7; server verifies consistency → bidding-down attack prevention); Annex I (key expansion: HMAC-SHA-1-96: IK_ESP = IK_IM||32-zeros=160-bit; AES-GMAC: IK_ESP = IK_IM + 32-bit salt via KDF(CK_IM||IK_IM, FC=0x58, P0="AES_GMAC_SALT"); AES-CBC: CK_ESP = CK_IM; AES-GCM: CK_ESP = CK_IM + 32-bit salt via KDF(CK_IM||IK_IM, FC=0x59, P0="AES_GCM_SALT"); KDF defined in TS 33.220 Annex B); Annex M (NAT traversal: UE behind NAT→UDP encapsulated tunnel mode RFC 3948; P-CSCF detects NAT by comparing packet source IP vs Via header IP; outer IP header used as SA selector; port 4500 for UDP encapsulation; port_Uenc=NAT-assigned port learned from first protected UDP packet; not IKE-based; media NAT out-of-scope); Annex N (SIP Digest: non-3GPP access only; cannot combine with IPsec; SD-AV=(realm,algo,qop="auth",H(A1)); flow same shape as IMS AKA but with HTTP Digest challenge/response; SM1 IMPI optional; CM2 delivers H(A1) not XRES/CK/IK; SM6=401 with nonce; SM7=REGISTER with response/cnonce/qop/nonce-count; S-CSCF stores H(A1)+nonce, verifies locally; SM12 has Authentication-Info for mutual auth; non-registration auth: P-CSCF IP address check table → IMPU assertion; proxy auth via Proxy-Authorization for INVITE; sync failure: stale=TRUE → UE retries; dynamic password: HSS pushes H(A1) via Cx; S-CSCF holds up to 2 H(A1) during transition); Annex O (TLS: non-3GPP access only; SIP Digest mandatory with TLS; P-CSCF authenticated by X.509 server cert; UE not cert-authenticated; two setup variants: during-registration (TLS handshake after SM6, before SM7) and prior-to-registration (TLS before SM1); P-CSCF marks SM8 with TLS integrity indicator "authentication pending"; S-CSCF state "tls-protected"; post-completion: both UE and P-CSCF reject SIP outside TLS except emergency/errors; session management: P-CSCF triggers renegotiation via HELLO; re-registration uses existing TLS session; cert profile: FQDN in CN+subjectAltName; chain to trusted root; CRL optional); Annex P (auth scheme coexistence: 6 schemes total; same S-CSCF must serve all; P-CSCF determines scheme from REGISTER content: Authorization header presence, Security-Client header presence, access type; S-CSCF stepwise analysis in §P.4 to determine scheme).

Notable findings:
- SIP Digest and IPsec are mutually exclusive — technically incompatible because SIP Digest generates no session keys (CK/IK), so IPsec SA cannot be established after SIP Digest auth. The combination "SIP Digest + TLS" is the non-3GPP equivalent of "IMS AKA + IPsec".
- AES-GCM/GMAC require additional salt material derived via the 3GPP generic KDF (TS 33.220 Annex B) because RFC 4106/4543 require a salt that cannot be embedded in the key. The simpler AES-CBC and HMAC-SHA-1-96 algorithms use CK_IM/IK_IM directly (or with zero-padding for SHA-1-96).
- Bidding-down protection in "ipsec-3gpp" is mandatory (Annex H): the Security-Client echo in SM7 proves that the UE's advertised algorithm list in SM1 has not been modified by a MITM before reaching the P-CSCF. An attacker stripping strong algorithms from SM1 would be detected at consistency check.
- SIP Digest password change (N.2.5): S-CSCF may hold two H(A1) values simultaneously. This "soft cutover" prevents service interruption when a user changes their IMS password — the old password remains valid until the S-CSCF confirms the new one works. Security implication: if password is compromised, admin de-registration before change is recommended to eliminate the grace period.
- TLS pre-registration variant (O.2.3): when TLS is selected from the start (before SM1), the RFC 3329 sip-sec-agree mechanism is not used — TLS is a fixed choice, not negotiated. This simplifies deployment for fixed-line operators who always use TLS.

Next chunk: ts33203 ingestion complete for core wiki purposes. Remaining annexes (R=NASS-bundled, S=3GPP2, T=GIBA, U=TNA, W=restrictive firewalls, X=WebRTC) are access-specific extensions not central to 4G EPC/IMS wiki goals.
Pages touched: [wiki/concepts/IMS-auth-alternatives.md (new), wiki/sources/ts33203.md (updated), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 33.203 §4–§9 — IMS Access Security Main Body [chunk ts33203-1]

Ingested §4 (5-SA security architecture overview; Figures 1-3: P-CSCF in VN vs HN topology with SEGs and Za/Zb NDS interfaces; IMS AKA independence from PS-domain AKA; ISIM on UICC as security anchor); §5.1.1 (mutual auth via IMS AKA; S-CSCF challenges via Cx-AV; UMTS/EPS AKA same computation reused with SIP transport per RFC 3310); §5.1.2 (re-authentication policy; unprotected REGISTER treated as initial registration); §5.1.3 (confidentiality via IPsec ESP, operator-optional, CK_ESP via key expansion); §5.1.4 (integrity mandatory, IK_ESP via key expansion; anti-replay; 4 mechanisms); §5.2 (network topology hiding: I-CSCF/IBCF encrypts Via/Record-Route/Route/Path; AES-CBC 128-bit; random IV per encryption; shared key Kv; P-CSCF receives but cannot decrypt); §5.3 (SIP Privacy: RFC 3323/3325; P-Asserted-Identity); §5.4 (non-IMS interworking: trust decision; untrusted = strip privacy headers); §6.1.0 (IMS AKA: IMPI as auth identity; two pairs of SAs per registered contact; emergency creates separate SAs); §6.1.1 (12-step auth flow: SM1-SM12 with Cx messages CM1-CM4; S-CSCF stores IK/CK in challenge; P-CSCF strips IK/CK before UE sees SM6); §6.1.2.1-6.1.2.3 (3 auth failure modes: user failure, network failure, sync failure); §6.1.3 (sync failure: UE sends AUTS; S-CSCF resync via Cx-AV-Req with RAND+AUTS); §6.1.4 (network-initiated re-auth: S-CSCF sends Auth-Required → UE re-registers); §6.1.5 (integrity protection indicator: P-CSCF annotates REGISTER for S-CSCF); §6.2 (IPsec ESP confidentiality: transport mode; CK_ESP from key expansion; dummy packets forbidden since Rel-11); §6.3 (IPsec ESP integrity: mandatory; transport mode or UDP tunnel for NAT; IK_ESP from key expansion); §6.4 (hiding: AES-CBC; random IV per encryption prevents ciphertext reuse); §6.5 (CSCF/non-IMS proxy TLS per RFC 3261); §7.0 (SA setup general: security mode setup during registration; subsequent signalling integrity-protected using derived keys); §7.1 (SA parameters: negotiated={encryption algo, integrity algo, SPIs}; fixed={transport mode, SA duration=2^32-1 sec, UDP+TCP}; 4-port model: port_us/port_uc at UE, port_ps/port_pc at P-CSCF; SA table structure; max 6 SAs per direction per IMPI; unprotected vs protected port rules); §7.2 (successful SA setup: 3-phase: unprotected REGISTER SM1 with Security-setup=SPI_U,Port_U,algo-list; P-CSCF selects algos, assigns SPI_P, Port_P; 4xx Auth_Challenge SM6 stripped of IK/CK; UE derives keys, creates SAs, SM7 protected with new outbound SA; P-CSCF consistency check; SM8 carries Integrity-Protection=Successful; SM12 signals success without Security-setup line); §7.3 (error cases: user auth fail = SM7 discarded at P-CSCF if IK wrong, or Auth_Failure at S-CSCF; network auth fail = REGISTER(Failure=AuthFail); sync fail = REGISTER(Failure=SyncFail,AUTS); security inconsistency = abort; proposal reject by P-CSCF or UE); §7.4 (re-registration SA transition: old SAs protect in-flight SIP transactions; new SAs for SM7 onwards; lifetime = max(old SA lifetime, registration timer); P-CSCF deletes old SAs when no pending transactions or lifetime expires); §7.5 (IP address change: delete all SAs, restart with unprotected REGISTER); §8.0-8.2 (ISIM: 3 implementation options; ISIM priority over USIM; contents = IMPI+IMPU+domain+SQN+algo+K; session keys deleted at power-off; sharing rules with USIM: CS/PS and IMS always run separate auth even when sharing keys); §9 (IMC: software credential for non-3GPP-only terminals; same contents as ISIM; not used if ISIM/USIM present).

Notable findings:
- IMS AKA is structurally independent from EPS AKA: even if ISIM/USIM share the same K, separate authentication runs produce different (CK,IK) pairs, eliminating cross-domain key compromise. Architectural decision: IMS security cannot be broken by compromising PS-domain security.
- P-CSCF is the sole IPsec termination anchor on the network side: it intercepts CK/IK from the 4xx Auth_Challenge (SM5), strips them before SM6 to UE, derives IK_ESP/CK_ESP, and creates/manages all 4 SAs. No other IMS node holds Gm session keys.
- The 4-port model (port_us, port_uc, port_ps, port_pc) is the mechanism that makes replay/reflection attacks infeasible: uplink and downlink SAs are distinct at the port level; SPIs further distinguish SAs per UE. UE randomly selects port numbers from a large space to resist DoS.
- Confidentiality is optional; integrity is mandatory — this is a deliberate 3GPP policy choice. An operator that trusts its access network can deploy IMS with NULL cipher while still enforcing message integrity on Gm.
- SA re-registration transition: the "grace period" for old SAs during re-authentication allows in-flight SIP dialogs (e.g. ongoing INVITE) to complete using the old SA without disruption — decouples SIP application state from IPsec SA lifecycle.
- AES-CBC with random IV for topology hiding: the same S-CSCF address encrypted twice in different Via headers produces different ciphertext — leaks no information about S-CSCF population even if attacker can observe traffic.

Next chunk: ts33203-2 — Annex H (RFC 3329 SIP Security Mechanism Agreement syntax) + Annex I (key expansion functions for IK_ESP/CK_ESP) + Annex N (SIP Digest alternative auth) + Annex O (TLS alternative transport security)
Pages touched: [wiki/concepts/IMS-access-security.md (new), wiki/procedures/IMS-AKA-registration.md (new), wiki/sources/ts33203.md (new), wiki/index.md (updated)]

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

---

## [2026-04-12] ingest | TS 33.203 §4–§9 [chunk ts33203-1] — IMS Access Security Main Body

Began ingest of 3GPP TS 33.203 v14.1.0. Chunk ts33203-1 covers the entire main body: §4 (security architecture overview — 5 SAs, Figures 1–3), §5 (security features: authentication, confidentiality, integrity, topology hiding, SIP privacy), §6 (security mechanisms: IMS AKA, authentication failures, ESP negotiation, topology hiding ciphering, non-IMS interworking), §7 (SA setup procedure: parameters, successful IMS registration case, error cases, re-registration SA transition), §8 (ISIM requirements: 3 implementation options, USIM sharing rules), §9 (IMC: IMS Credentials for non-3GPP/fixed-broadband terminals).

Notable findings:
- The IMS has its own 5 security associations independent of EPS/UMTS; only SA② (Gm: UE↔P-CSCF) is specified in this document — the others (NDS/IP on Zb and Za/Zc) are specified in TS 33.210.
- IMS AKA is architecturally independent of EPS AKA: even when both use the same UICC/USIM, the IMS authentication produces domain-specific session keys (CK_IM, IK_IM) that are separate from the EPS keys. This prevents cross-domain key reuse.
- P-CSCF is the IPsec termination anchor: it strips CK/IK from the 401 challenge before forwarding to the UE, derives IK_ESP/CK_ESP from them, and creates 4 inbound+outbound SAs using the 4-port model (port_us, port_uc, port_ps, port_pc). The UE never sees the raw AUTN nonce in cleartext at the IP layer.
- The 4-port model (unique SPI per direction per port pair) prevents reflection attacks where an attacker replays a protected message from one SA direction into another.
- Integrity protection is mandatory; confidentiality is operator-optional (NULL cipher permitted).
- Topology hiding uses AES-CBC with a random IV — same plaintext header always produces a different ciphertext, preventing correlation attacks based on ciphertext matching.
- ISIM on UICC takes strict priority over USIM for IMS credentials. Session keys (CK_IM, IK_IM) MUST be deleted at UE power-off.

Next chunk: ts33203-2 — Annexes H (RFC 3329 SIP Security header BNF), I (key expansion), M (NAT traversal), N (SIP Digest), O (TLS), P (auth scheme coexistence)
Pages touched: [wiki/concepts/IMS-access-security.md (new), wiki/procedures/IMS-AKA-registration.md (new), wiki/sources/ts33203.md (new), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 33.203 Annexes H/I/M/N/O/P [chunk ts33203-2] — Auth Alternatives and Security Details

Final chunk of TS 33.203 ingest. Covered: Annex H (normative — RFC 3329 Security Mechanism Agreement BNF for `ipsec-3gpp` and `tls` mechanisms, bidding-down protection via Security-Client echo in SM7), Annex I (normative — IK_ESP/CK_ESP key expansion functions: simple algorithms use CK_IM/IK_IM directly or with zero-padding; AES-GCM/AES-GMAC use KDF from TS 33.220 Annex B with 32-bit salt), Annex M (normative — NAT traversal: UDP port 4500 encapsulation per RFC 3948; P-CSCF detects NAT by comparing packet source IP to Via header), Annex N (normative — SIP Digest: SD-AV structure, HTTP Digest challenge/response flow, dynamic password change via re-registration), Annex O (normative — TLS: profile, session setup during/prior-to registration, certificate validation rules), Annex P (normative — auth scheme coexistence: 6 schemes, P-CSCF/S-CSCF selection rules; same S-CSCF must be capable of serving both fixed and mobile subscribers).

Notable findings:
- SIP Digest is fundamentally incompatible with IPsec SA establishment — no session keys are generated. Combined IMS AKA + SIP Digest is technically impossible; they target different security mechanism classes.
- SIP Digest + TLS is the non-3GPP access equivalent of IMS AKA + IPsec — both provide mutual authentication and SIP confidentiality/integrity via different mechanisms suited to different access types.
- The `ipsec-3gpp` mechanism in RFC 3329 extends the base spec with bidding-down protection: the P-CSCF stores the UE's Security-Client from SM1 and verifies that the Security-Verify in SM7 echoes it unchanged. Without this, an attacker could downgrade the negotiated algorithm set.
- Key expansion for AES-GCM (preferred for newer implementations) requires KDF(CK_IM || IK_IM, ...) per TS 33.220 Annex B — the resulting IK_ESP and CK_ESP are 256-bit keys suitable for GCM's integrity+confidentiality combined mode.

TS 33.203 ingest COMPLETE for core wiki purposes. Remaining annexes (R=NASS-bundled, S=3GPP2, T=GIBA, U=TNA, W=restrictive firewalls, X=WebRTC) are access-specific extensions not central to 4G EPC/IMS domain.
Pages touched: [wiki/concepts/IMS-auth-alternatives.md (new), wiki/sources/ts33203.md (updated — chunk ts33203-2 added, status COMPLETE), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 32.299 §4–§5 [chunk ts32299-1] — Charging Architecture and Requirements

Began ingest of 3GPP TS 32.299 v16.2.0. Chunk ts32299-1 covers §4 (high-level architecture: CTF→CDF (offline Rf) and CTF→OCF (online Ro) with CGF→Billing Domain; priority-ordered failover address list; PLMN-local charging entity rule; roaming online charging via PCEF Gy to HPLMN OCS; EU roaming unbundling proxy on Ro) and §5 (3GPP charging application requirements: §5.1 offline scenarios — event-based (single ACR[Event]) and session-based (ACR[Start+Interim*+Stop]); §5.2 online scenarios — IEC, ECUR, SCUR; §5.3 other requirements including re-authorization triggers, termination action, account expiration).

Notable findings:
- The 3GPP charging architecture is a two-plane model: offline (post-payment CDR collection) and online (real-time credit control) share the same CTF in each network element but use different protocols and servers. The same NE can report to both simultaneously for services that require both.
- The "Centralized UD + Decentralized Rating" combination (centralized Unit Determination at OCF, decentralized Rating at CTF) is explicitly NOT supported. This eliminates an architecturally ambiguous case that would require the OCF to communicate unit-to-price mapping to the CTF in real-time.
- SCUR's combined debit+reserve in a single CCR[UPDATE] is the key efficiency mechanism — one round-trip handles both reporting consumed units and requesting the next quota grant.

Next chunk: ts32299-2 — §6.1–§6.2 Diameter offline charging (Rf): ACR/ACA message formats
Pages touched: [wiki/concepts/charging-architecture.md (new), wiki/sources/ts32299.md (new), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 32.299 §6.1–§6.2 [chunk ts32299-2] — Offline Charging Protocol (Rf)

Ingested §6.1 (Diameter offline charging principles: CTF as Diameter Accounting client, CDF as Diameter Accounting server per RFC 6733; CDF implements SERVER STATELESS ACCOUNTING — no ordering constraint on ACR receipt; record types START/INTERIM/STOP/EVENT; ACA[START] carries Acct-Interim-Interval; error cases: CDF connection failure with failover and non-volatile buffering; no-reply retransmission with T-flag; duplicate detection; CDF timer-based CDR close) and §6.2 (ACR/ACA message formats: Command-Code 271; 3GPP-added Service-Context-Id and Service-Information; unused AVPs list; ACA fields including Error-Reporting-Host and Acct-Interim-Interval).

Notable findings:
- The CDF's STATELESS ACCOUNTING model (per RFC 6733 §9.7) means the CDF processes each ACR independently regardless of receipt order. This is critical for fault-tolerant deployments where ACR[Interim] records may arrive out of order or duplicated after CTF failover. The CDF must not reject an ACR[Stop] that arrives before a corresponding ACR[Start].
- The non-volatile buffer requirement (§6.1.3.1) means CTFs must persist charging data locally when the primary CDF is unreachable and flush it when connectivity is restored. Data loss = revenue loss in this domain; the protocol explicitly mandates storage before forwarding.
- The T-flag on retransmitted ACRs is the duplicate detection signal. CDFs track T-flagged records: if the original was already processed, the duplicate is silently discarded; if the original was lost (T-flag but no prior record), the retransmitted ACR is used to create the CDR. This asymmetry prevents both double-billing and lost billing.

Next chunk: ts32299-3 — §6.3–§6.5 Diameter online charging (Ro): CCR/CCA, IEC/ECUR/SCUR flows
Pages touched: [wiki/protocols/Rf-offline-charging.md (new), wiki/sources/ts32299.md (updated), wiki/index.md (updated)]

---

## [2026-04-12] ingest | TS 32.299 §6.3–§6.5 [chunk ts32299-3] — Online Charging Protocol (Ro)

Ingested §6.3 (Ro protocol flows for IEC, ECUR, and SCUR with Debit+Reserve combined in single CCR[UPDATE] for SCUR), §6.4 (CCR/CCA message formats: Command-Code 272, Auth-Application-Id=4; full MSCC AVP structure with RSU/USU/Service-Id/Rating-Group/FUI/VT/QHT/QCT/Thresholds/Reporting-Reason/Trigger; Re-Auth/RAR and Capabilities-Exchange/CER messages), §6.5 (procedural descriptions: idle timeout/QHT, change-of-charging-conditions, quota consumption time/QCT, service termination, envelope reporting, combinational quota, online control of offline charging, multi-service, Supported Features), §6.6 (AVP bindings tables 6.6.1.1 offline and 6.6.2.1 online), §6.7 (NDS/IP security for charging interfaces).

Notable findings:
- The combined debit+reserve pattern in SCUR CCR[UPDATE] (used+reserve in one message) means the OCF processes consumed units and grants new quota atomically. If the OCF is unreachable mid-session, the CTF behavior is governed by CCFH (Credit-Control-Failure-Handling): TERMINATE, CONTINUE (allow service without credit control), or RETRY_AND_TERMINATE.
- QCT (Quota-Consumption-Time) enables time quota to be consumed proportionally with volume usage — a sophisticated mechanism to prevent "buy time, use as little as possible" abuse. The CTF uses CTP (volume-triggered time period) or DTP (duration-triggered time period) modes.
- Envelope reporting (§6.5.6) allows the OCF to request that the CTF split the online charging session into usage envelopes at quota boundaries. This is used for detailed analytics (e.g. when the tariff changes mid-session) and generates parallel offline CDR records via the Offline-Charging AVP in the CCA.
- The Final-Unit-Indication (FUI) AVP with FUA=REDIRECT is the mechanism for "pay wall" scenarios: when the account is nearly exhausted, the OCF tells the CTF to redirect HTTP traffic to a top-up portal while allowing other traffic to continue. FUA=RESTRICT_ACCESS installs packet filters instead.

Next chunk: ts32299-4 — §7 Key AVPs (selective)
Pages touched: [wiki/protocols/Ro-online-charging.md (new), wiki/sources/ts32299.md (updated), wiki/index.md (updated, header note updated to "ts32299-3 complete")]

---

## [2026-04-12] ingest | TS 32.299 §7 [chunk ts32299-4] — Key AVP Reference (Selective) — TS 32.299 COMPLETE

Final chunk of TS 32.299 ingest. Covered §7.1 (IETF Diameter AVPs reused by 3GPP: full Table 7.1.0.1 cross-reference; detailed description of Accounting-Input/Output-Octets, Acct/Auth-Application-Id, Called-Station-Id/APN, Event-Timestamp, Experimental-Result 3GPP extensions, MSCC ABNF, Rating-Group, Result-Code 3GPP extensions, Service-Context-Id with 3GPP TS URN table, Service-Identifier, Used-Service-Unit ABNF, User-Name, Vendor-Id=10415, User-Equipment-Info IMEISV encoding) and §7.2 (3GPP-specific AVPs: full Table 7.2.0.1 with ~250 AVPs; detailed descriptions of key AVPs in categories: session/subscriber identity, Service-Information umbrella, PS-Information, IMS-Information, SDP-Media-Component, role/routing identity, online quota management, tariff/AoC, SRVCC/Access-Transfer, AF-Correlation, AVP code range allocation).

Notable findings:
- The **IMS-Charging-Identifier (ICID) — AVP 841** is the linchpin for IMS charging correlation across all nodes handling a single SIP session. It is generated by the first IMS entity (typically P-CSCF) and propagated in the SIP `P-Charging-Vector` header. Every CSCF and AS that generates a charging record for that session includes the same ICID in its ACR/CCR, allowing the billing domain to correlate records from 5+ nodes into a single billable event. Without ICID, IMS billing would be operationally infeasible.
- **Node-Functionality (AVP 862)** encodes which type of IMS node generated the charging record (S-CSCF=0, P-CSCF=1, I-CSCF=2, MRFC=3, MGCF=4, BGCF=5, AS=6, etc.). Combined with Role-Of-Node (Originating=0, Terminating=1), this gives the billing domain a complete picture of the call path topology from charging records alone — without needing to inspect SIP traces.
- The **Service-Context-Id URN structure** (`"extensions".MNC.MCC."Release"."32260@3gpp.org"`) is designed for per-operator extensibility. The MNC.MCC prefix means two operators deploying the same TS 32.260 can have operator-specific charging parameters without AVP code collision. The "Release" field provides forward/backward compatibility — a Release-12 CTF talking to a Release-10 CDF can signal the version mismatch via this AVP.
- **Access-Transfer-Information (AVP 2709)** + **Related-IMS-Charging-Identifier (AVP 2711)** together solve the SRVCC billing continuity problem: when a VoIP session transfers from PS (IMS) to CS, the IMS charging record for the CS leg must reference the ICID of the original PS leg so that the operator charges for one call rather than two disconnected segments. The Related-IMS-Charging-Identifier carries the pre-transfer ICID; Access-Transfer-Type identifies the direction (PS-to-CS=0, CS-to-PS=1).
- The **AVP code range allocation** (§7.2 table NOTE) is a significant architectural choice: TS 29.230 pre-reserves code ranges per 3GPP release for TS 32.299, ensuring that new AVPs added in future releases cannot collide with existing deployed AVPs. This allows charging implementations to safely ignore unknown AVPs from newer releases without risk of misinterpretation.

TS 32.299 ingest COMPLETE. All 4 chunks (ts32299-1 through ts32299-4) written.
Pages touched: [wiki/protocols/charging-AVPs.md (new), wiki/sources/ts32299.md (updated — ts32299-4 Done, status COMPLETE), wiki/index.md (updated, 71→72 pages)]

---

## [2026-04-12] ingest | TS 32.260 §4 + §5.1 [chunk ts32260-1] — IMS Charging Architecture and Principles

Began ingest of 3GPP TS 32.260 v17.3.0. Chunk ts32260-1 covers §4 (IMS charging architectures: §4.2 offline — all IMS NEs as CTFs send Rf to CDF; CDF→CGF via Ga; CGF→Billing Domain via Bi; 12 node types: S-CSCF, P-CSCF, I-CSCF, MRFC, MGCF, BGCF, SIP AS, IBCF, E-CSCF, TRF, ATCF, IMS Transit Functions; §4.3 online — MRFC/SIP AS send Ro directly to OCS; S-CSCF uses ISC→IMS-GWF→Ro→OCS; §4.4 converged — Nchf/N45 to CHF; three CDR forwarding options; roaming: P-CSCF in VPLMN, H-CHF in HPLMN via NRF) and §5.1 (IMS charging principles: §5.1.1 method selection via P-Charging-Function-Addresses CCF/ECF parameters; §5.1.2 charging correlation via P-Charging-Vector: ICID globally unique ≥1 month, generated at first IMS NE; Related ICID for SRVCC target access leg; access network charging identifier per access type; IOI Orig/Term/Transit list; visited network identifier; loopback-indication for TRF; FE Identifier List; §5.1.3 SDP offer/answer handling with SDP-type parameter; §5.1.4 trigger conditions — I-CSCF/BGCF event-only, others session-based; location capture delay at BYE; §5.1.5 RTTI with IBCF trust filtering; §5.1.6 served user via P-Header-Served-User; §5.1.7 OneChargingSession for B2BUA; §5.1.8/8A roaming LBO vs home-routed; §5.1.9 location retrieval and CDR recording per access type; §5.1.10–§5.1.13 TRF, service continuity/ATCF, announcements, UE location/TimeZone handling).

Notable findings:
- The key distinction between IMS online charging and EPC/PS domain online charging: in EPC the PGW is the dominant online CTF (Gy to OCS). In IMS, each service node (MRFC, SIP AS) contacts the OCS directly via Ro, while the S-CSCF acts as a relay via ISC→IMS-GWF→Ro. This means IMS online charging generates multiple simultaneous Ro sessions per call (one per involved online-capable node), all correlated by ICID.
- The ICID's 1-month global uniqueness requirement is operationally significant: it means an IMS node cannot simply use a random number — it must incorporate node-specific entropy (topology/location information, high-granularity time) to guarantee no collision across the entire 3GPP ecosystem during the correlation window.
- The P-Charging-Function-Addresses header is the runtime mechanism by which the IMS node discovers charging server addresses. This means charging server selection is dynamic per-session (carried in SIP signalling), not purely static configuration. An operator can redirect all charging for a roaming subscriber to a different CDF/OCS by injecting different addresses into the P-Charging-Function-Addresses header at the P-CSCF.
- The OneChargingSession mechanism for B2BUA/ATCF addresses a fundamental accounting problem: when a node acts as B2BUA it creates two dialog legs. Without OneChargingSession, each leg generates a separate charging record with a different ICID, making it impossible for the billing domain to determine they belong to the same call. By preserving ICID across legs, a single charging session record covers both dialog legs.

Next chunk: ts32260-2 — §5.2.1.1–§5.2.1.12 Offline charging: MO/MT session flows, mid-session, session release, PSTN/IMS-initiated, multi-party
Pages touched: [wiki/concepts/IMS-charging-architecture.md (new), wiki/sources/ts32260.md (new), wiki/index.md (updated, 72→74 pages)]

---

## [2026-04-13] ingest | TS 32.260 §5.2.1–§5.2.2.1.12 [chunk ts32260-2] — IMS Offline Charging Flows

Ingested §5.2.1 (Basic principles: Table 5.2.1.1-1 — CDR trigger table for all non-MRFC/non-AS IMS nodes via Rf: CDR[Start] at SIP 2xx/ACK/ISUP:ANM; CDR[Interim] at RE-INVITE/UPDATE/1xx/RTTI/failed RE-INVITE; CDR[Stop] at BYE/200-OK-BYE-for-legal/4xx-setup/REL/CANCEL; CDR[Event] for I-CSCF/BGCF session success, NOTIFY/MESSAGE/REGISTER/SUBSCRIBE/PUBLISH/REFER/3xx/failed-unrelated; Table 5.2.1.1-2 — MRFC trigger table: CDR[Start] at SIP 2xx conference-initiating INVITE; CDR[Interim] per UE joining (ACK) + RE-INVITE + participant-leaving BYE + interim interval expiry; CDR[Stop] at conference-terminating BYE/CANCEL/4xx; §5.2.1.2 Nchf converged triggers: all Immediate default, CHF changes Not Applicable for IMS nodes; Table 5.2.1.2-3/4 for IMS-GWF/AS and MRFC converged events) and §5.2.2.1.1–§5.2.2.1.12 (message flows: MO session establishment 3 SDP scenarios [200 OK SDP answer = CDR[Start] immediate; 200 OK SDP offer = CDR[Start]+CDR[Interim]-at-ACK; ACK-only = CDR[Start]-at-ACK]; MT session [P-CSCF/S-CSCF session-based; I-CSCF event-only at 200 OK]; mid-session RE-INVITE 2 scenarios; session release MO 2 scenarios [BYE trigger vs 200-OK trigger for location]; session-unrelated procedures; PSTN-initiated via MGCF [CDR[Start] at ISUP:ANM]; IMS-initiated PSTN breakout [MGCF CDR[Start] at ANM; BGCF CDR[Event] at 200 OK]; PSTN-initiated release [MGCF CDR[Stop] at REL]; IMS-initiated release [MGCF CDR[Stop] at BYE]; multi-party MRFC conference [28-step flow; conference-ID in Service-ID; Application-Provided-Called-Parties updated per UE joining; OneChargingSession in SCC AS]; AS redirect server [CDR[Event] after 302]; AS voicemail server [CDR[Start/Stop] session charging]).

Notable findings:
- The CDR trigger timing asymmetry between **session release Scenario 1 vs 2** is architecturally important: in Scenario 2 (triggered at 200 OK rather than BYE), the CDR timestamp must still reflect BYE reception time per TS 29.214 Annex A.10.5. This means the CDF can receive an ACR[Stop] where the record "Stop Time" predates the message delivery time — the CDF must not reject this. This is the same stateless accounting property exploited in the Rf protocol generally.
- The **BGCF always produces CDR[Event], never CDR[Start/Stop]**. This is architecturally consistent: BGCF is only involved in the initial routing decision (routing from IMS to PSTN), not in the ongoing session. Once the INVITE has been forwarded to the MGCF, the BGCF exits the signalling path. Its single CDR records the routing event only.
- The **MRFC CDR[Interim] per-participant-join** (step 9, 17, 25) means the MRFC CDR evolves incrementally as each new participant joins the conference. The field `Application-Provided-Called-Parties` is updated in each Interim, giving the billing domain a full participant list. This is more sophisticated than the CSCF model where one CDR records one session pair.
- The **AS charging mode selection** (event vs session) is service-determined, not protocol-determined. The same AS node uses CDR[Event] for stateless services (redirect/call forwarding — executes in milliseconds and exits the signalling path) and CDR[Start/Stop] for stateful services (voicemail — maintains a session for potentially minutes). This flexibility means AS billing can accurately reflect the nature of the service consumed.

Next chunk: ts32260-3 — §5.2.2.1.13–§5.2.2.2 (AS as SCC AS, remaining AS flows, error cases) + §5.3 (IMS online charging: IEC/ECUR/SCUR via Ro)
Pages touched: [wiki/procedures/IMS-offline-charging-flows.md (new), wiki/sources/ts32260.md (updated — ts32260-2 Done), wiki/index.md (updated, 74→75 pages)]

---

## [2026-04-13] ingest | TS 32.260 §5.2.1.13–§5.2.2.2 + §5.3 [chunk ts32260-3] — AS-Related Offline Flows + IMS Online Charging

Ingested §5.2.2.1.13–§5.2.2.2 (AS-related offline flows) and §5.3 (IMS online charging).

**§5.2.2.1.13 — SCC AS (B2BUA):** All 10 sub-scenarios documented — MO/MT PS-only (#13.1–2), PS→CS transfer (#13.3–4, STN-SR/SRVCC), CS→PS (#13.5–6), inter-UE transfer (#13.7–8), emergency (#13.9–10). SCC AS generates one CDR per dialog leg (Call-ID#inc / Call-ID#out), with OneChargingSession ICID propagation to correlate both legs. ATCF/EATF variants parallel SCC AS pattern in visited network.

**§5.2.2.1.14–22 — Remaining offline scenarios:** §14 Alternate Charged Party (AS substitutes Subscription-Id in CDR to bill third party); §15 IBCF (opens CDR per home IMS in inter-network sessions); §16 MMTel reference to TS 32.275; §17–18 RTTI (at setup: ACR[Start] delayed until RTTI received in SIP INFO; mid-session: ACR[Interim] triggered by RTTI in SIP INFO); §19 OMR (4 sub-scenarios, 4 CDR indicator fields: Local GW Not/Inserted, IP realm Not Default, Transcoder Not/Inserted); §20 B2BUA single charging session (ICID preserved across Call-ID legs); §21 LBO roaming (TRF CDR with NNI indicators: roaming-loopback vs non-roaming vs non-loopback); §22 ATCF (6 sub-scenarios for PS↔CS media anchoring in visited network).

**§5.2.2.2 — Error cases:** Full table of 6 error scenarios with handling rules (SIP 4xx/5xx/6xx, no response from CDF, duplicate detection, etc.).

**§5.3 — IMS online charging (Ro):** §5.3.1 basic principles — IEC/ECUR/SCUR comparison, IMS-GWF/AS/MRFC as online CTFs, Table 5.3.1.1 trigger SIP methods for IMS-GWF/AS, Table 5.3.1.2 MRFC triggers. §5.3.2 message flows — IEC Scenario 1 with Mermaid flow, SCUR Scenarios 1–5 (session establishment SDP-answer-in-200-OK, early media, mid-session RE-INVITE [two Update CCRs per RE-INVITE], session release BYE vs 200-OK variants, RTTI tariff update), ECUR session-unrelated procedure, error cases table. §5.3.2.3 OCS-initiated termination — 5 trigger conditions (FUI, ASR, failed Diameter result), Scenario A (SIP error + optional top-up Contact URI redirect), Scenario B (SIP BYE/CANCEL + Reason header).

Notable findings:
- **Two Reserve Units[Update] per RE-INVITE** is a critical SCUR detail: one sent at the RE-INVITE (reporting pre-change units), a second at the 200 OK (confirming media change and reporting units). A naive implementation that sends only one would miss either the change event or the confirmation.
- **RTTI flow asymmetry offline vs online**: In offline charging, RTTI in SIP INFO delays ACR[Start] (the CDR does not open until tariff is known). In online charging, RTTI triggers an additional Reserve Units[Update] mid-session with Tariff-Information mapped from RTTI XML — i.e., the CC session is already open when tariff arrives, and the OCS adjusts the grant.
- **IMS-GWF is not a stand-alone node** — it is a logical function embedded within S-CSCF scope. The ISC interface between S-CSCF and IMS-GWF is internal, making the S-CSCF transparent to the OCS. This architectural choice means S-CSCF upgrades to support online charging only require adding the IMS-GWF function — no new reference point toward the OCS.
- **OCS top-up redirect** via Contact URI in SIP error response enables "in-call balance top-up" UX: when credit exhausts, the user receives a SIP error that auto-redirects their client to a web portal rather than a silent drop. This is explicitly operator-configurable per SIP method and per Originating/Terminating direction.

Next chunk: ts32260-4 — §5.4 converged charging scenarios + §6.1.1–§6.1.2 (Rf/Bi message contents)
Pages touched: [wiki/procedures/IMS-offline-charging-flows.md (extended — §5.2.2.1.13–§5.2.3), wiki/procedures/IMS-online-charging-flows.md (new — §5.3), wiki/sources/ts32260.md (updated — ts32260-3 Done), wiki/index.md (updated, 75→76 pages)]

---

## [2026-04-13] ingest | TS 32.260 §5.4 + §6.1.1–§6.1.2 [chunk ts32260-4] — IMS Converged Charging + Charging Data Message Structures

Ingested §5.4 (converged charging) and §6.1.1–§6.1.2 (Charging Data message structure definitions).

**§5.4 Converged charging:** §5.4.1 establishes that MRFC, SIP AS, and IMS-GWF use Nchf (HTTP/JSON SBI) to the CHF, which acts as combined OCF+CDF. §5.4.3 Table 5.4.3.1 — Default Trigger conditions: same SIP events as offline/online, but with two new control columns: "CHF allowed to change category" (most Immediate → fixed) and "CHF allowed to enable/disable" (quota and CHF-limit triggers can be dynamically toggled by CHF via the response). Table 5.4.3.2 maps each event to the exact Charging Data Request type (SCUR/ECUR/IEC/PEC CDR[Initial/Update/Termination/Event]). §5.4.4 CHF CDR triggers: open at CDR[Initial], update at CDR[Update], close at CDR[Termination], one-off at CDR[Event]. §5.4.5–6 reference TS 32.295 (Ga) and TS 32.297 (Bi).

**§6.1.1 Rf message contents:** Charging Data Request (Table 6.1.1.1.1) adds two IMS-specific IEs on top of TS 32.299 base: Operation Token (OM, identifies domain/subsystem/service/release) and Service Information (OM, holds IMS charging parameters from §6.3). Source nodes: all 12 IMS CTF types → CDF. Charging Data Response (Table 6.1.1.2.1) carries Operation Result (M) and standard session/originator IEs.

**§6.1.1a Nchf offline message contents:** MRFC + SIP AS → CHF. Charging Data Request uses 5G SBI fields (Subscriber Identifier M, NF Consumer Identification M, Invocation Timestamp M, Invocation Sequence Number M) plus IMS-specific Triggers (OC, per §5.4.3) and IMS Charging Information (OM, per §6.4). Response carries Invocation Result (M) + Multiple Unit information (Granted Unit, Validity Time). Full structure marked FFS in Release 17.

**§6.1.2 GTP':** One line — "Not applicable." IMS charging does not use GTP' at the node level; GTP' is the Ga transport between CDF and CGF.

Notable findings:
- **PEC (Partial Event Charging)** is a new charging method that exists only in converged charging (not in Rf or Ro). It applies specifically to SIP 4xx/5xx/6xx final error responses on unsuccessful procedures. The billing model for PEC differs from IEC: a partial event (a SIP transaction that failed) can have a different rate than a successful event. This suggests the CHF can discriminate pricing based on outcome — e.g., charging less (or zero) for a failed call attempt.
- **Dynamic trigger control** is architecturally new in converged charging: the CHF can return an updated Triggers AVP in the Charging Data Response, enabling or disabling future reporting triggers mid-session. In Rf and Ro, trigger configuration is static at node provisioning. This allows the CHF to adapt monitoring granularity based on account status — e.g., increase update frequency as quota runs low.
- **The Nchf offline-only structure is FFS** (For Further Study) as of Release 17 — the spec acknowledges via Editor's Note that the full CDR body was not yet normatively defined. This reflects the state of 5G SBI standardization at the time: the IMS/Nchf integration was in progress and implementations were expected to follow TS 32.290 for the base structure.
- **Operation Token** in the Rf Charging Data Request (§6.1.1) is the IMS equivalent of Service-Context-Id in TS 32.299 — it encodes the domain (IMS), subsystem (S-CSCF, P-CSCF, etc.), service, and 3GPP release. This allows the CDF to process IMS charging data correctly even if the CDF is a generic Diameter accounting server that handles multiple domains.

Next chunk: ts32260-5 — §6.1.3 Per-node CDR field content (S-CSCF, P-CSCF, I-CSCF, MRFC, MGCF, BGCF, SIP AS, IBCF, E-CSCF, TRF, ATCF, TF CDR tables)
Pages touched: [wiki/procedures/IMS-converged-charging-flows.md (new — §5.4 + §6.1.1–§6.1.2), wiki/sources/ts32260.md (updated — ts32260-4 Done), wiki/index.md (updated, 76→77 pages)]

---

## [2026-04-13] ingest | TS 32.260 §6.1.3 [chunk ts32260-5] — Per-Node CDR Field Content (All 12 IMS CDR Types)

Ingested §6.1.3 — the full per-node CDR field content tables for all 12 IMS CDR types, covering §6.1.3.1 (S-CSCF), §6.1.3.2 (CDR triggers at CDF), §6.1.3.3 (S-CSCF), §6.1.3.4 (P-CSCF), §6.1.3.5 (I-CSCF), §6.1.3.6 (MRFC), §6.1.3.7 (MGCF), §6.1.3.8 (BGCF), §6.1.3.9 (SIP AS), §6.1.3.10 (IBCF), §6.1.3.11 (E-CSCF), §6.1.3.12 (TRF), §6.1.3.13 (ATCF), §6.1.3.14 (TF).

**CDR type modes:** S-CSCF (session+event), P-CSCF (session+event), I-CSCF (event only), MRFC (session only), MGCF (session+event), BGCF (event only), SIP AS (session+event), IBCF (session+event), E-CSCF (session+event), TRF (session+event), ATCF (session+event), TF (session+event).

**Per-node distinctive fields:**
- S-CSCF: Application Servers Information, Transit IOI List, NNI Information, Route Header (Received/Transmitted)
- P-CSCF: Served Party IP Address (originating/terminating), Local GW Inserted Indication, IP realm Default Indication, Related ICID, IMS Application Reference ID
- I-CSCF: S-CSCF information field; no SDP, no service duration — pure event record
- MRFC: Session-Id = conference ID; Online Charging Flag with "no proof" note; no Access Network Info fields
- MGCF: ISUP Cause, Trunk Group ID (incoming/outgoing), Bearer Service; no Access Correlation ID
- BGCF: event-only, simplified; NNI Information (loopback routing)
- SIP AS: Outgoing Session ID, Alternate Charged Party Address, Service Specific Info (TAD Identifier for ICS), VLR/MSC Number/Address (SRVCC), Access Transfer Information list, Initial IMS Charging Identifier, 3GPP PS Data Off Status, IMS Application Reference ID, Requested Party Address
- IBCF: Transcoder Inserted Indication; NNI Information (4 sub-fields: Session Direction, NNI Type, Relationship Mode, Neighbour Node Address) may appear multiple times; IMS Communication Service ID with "+g.3gpp.ics-ref" ICSI rule
- E-CSCF: Called Party Address can be a URN (tel URI or SOS URN); structure similar to S-CSCF
- TRF: NNI Information may appear multiple times (per interconnect peer); Transit IOI List includes inbound + outbound TRF IOIs
- ATCF: Outgoing Session ID (ATCF outgoing dialog); Initial ICID; Correlation MSISDN in calling address; collocated with P-CSCF or IBCF
- TF: Transit IOI List includes TF's own IOI; Application Servers Information for transit network ASes; omits Access Network Info fields; Editor's Note flagging multiple items as FFS

Notable findings:
- **I-CSCF and BGCF are purely event-CDR nodes.** Their CDRs record routing decisions only — no session duration, no SDP media info. This reflects their role: I-CSCF selects an S-CSCF and exits the path; BGCF selects a MGCF for PSTN breakout. The billing domain gets a routing trace, not a usage record.
- **MRFC Session-Id = conference-ID**, not the SIP dialog Call-ID. This is a critical implementation detail: conference charging must use the conference application's ID so all participants map to the same CDR sequence. An MRFC using Call-ID would generate a separate CDR per participant INVITE with no common key.
- **IBCF NNI Information can appear multiple times** per CDR — once per interconnect peer. In a multi-hop inter-PLMN transit scenario, the IBCF traverses multiple NNI segments and records each one. The Relationship Mode (bilateral agreement, roaming, other) per segment is critical for settlement between operators.
- **Transcoder Inserted Indication** appears only in IBCF (not S-CSCF, P-CSCF, or TRF), confirming that media transcoding at NNI boundaries is specifically IBCF responsibility. The OMR (Optimal Media Routing) field set includes: Local GW Inserted (P-CSCF + IBCF), IP realm Default (P-CSCF + IBCF), Transcoder Inserted (IBCF only).
- **TF (Transit Function) CDR** includes the TF's own Transit IOI in the Transit IOI List, unlike other nodes that record external IOIs. This enables transit operators to identify their own network's IOI in the CDR chain — critical for inter-carrier settlement in transit routing scenarios.

Next chunk: ts32260-6 — §6.2–§6.4 + Annex B (Ro message formats, IMS Information AVP, converged charging data, OCS termination Annex)
Pages touched: [wiki/protocols/IMS-CDR-field-reference.md (new), wiki/sources/ts32260.md (updated — ts32260-5 Done), wiki/index.md (updated, 77→78 pages)]

---

## [2026-04-13] ingest | TS 32.260 §6.2–§6.4 + Annex B [chunk ts32260-6] — Ro Message Content, IMS Information Parameters, Converged Charging Data, OCS Termination Flows

Ingested §6.2 (Ro message contents for IMS), §6.3 (IMS charging specific parameters), §6.4 (converged charging data), and Annex B (OCS service termination flows). This completes all normative content of TS 32.260.

**§6.2 — Ro message contents:** Two messages. Debit/Reserve Units Request (MRFC/AS/IMS-GWF → OCS) adds IMS-specific IEs on top of TS 32.299: User Name = IMPI (Private User Identity per TS 23.003), Subscriber Identifier = MSISDN or SIP-URI, Service Information = IMS Information + PS Information. Debit/Reserve Units Response (OCS → CTF) adds: Operation Failure Action (for SCUR/ECUR behavior on OCS failure), Operation Event Failure Action (for IEC failure), Announcement Information. Notably, Redirection Host/Usage/Cache-Time, Proxy/Route Info, and Failed Parameter — all present in the base TS 32.299 CCA — are not used by IMS nodes (all "-" in the per-node table).

**§6.3 — IMS charging specific parameters:** §6.3.1.1 Service Information components: outer container wrapping IMS Information (Om, all nodes), PS Information (Oc, all), Subscriber Identifier (Om, not IBCF/I-CSCF/MGCF/BGCF/TF), VCS/ISUP/VLR/MSC for MGCF/AS. §6.3.1.2 IMS Information structure: the comprehensive ~50-field IMS-specific block embedded in every Rf/Ro charging message. Key fields: Event Type (SIP method + Event + expires headers), Node Functionality (M, all), Role of Node (Om, not MRFC), ICID (Om, all), Inter Operator Identifier (Oc, multi-valued, all), Calling/Called Party Address (Om, all), SDP Session Description + Media Component (both not I-CSCF/BGCF), Access Network Information (not TF/TR), FE Identifier List (Oc, all, enables post-hoc CDR correlation). §6.3.2 offline per-node matrix: I-CSCF = Event-only, BGCF = Event-only, MRFC = S/I/S (no Event). §6.3.3 online per-node matrix: IMS-GWF = I/U/T/E, MRFC = I/U/T, AS = I/U/T/E. §6.3.4 formal parameter encoding → TS 32.298 (CDRs) + TS 32.299 (events).

**§6.4 — Converged charging data:** §6.4.1 message contents: Charging Data Request (IMS Node → CHF) and Response (CHF → IMS Node), both per TS 32.290. Request: NF Consumer Identification (M), Triggers (Oc, IMS-specific §5.4 triggers), IMS Charging Information (Om). Response: Multiple Unit information with Granted Unit, Validity Time, Final Unit Indication, Announcement Information. Both full structures marked FFS (Editor's Note). §6.4.2 IMS Charging Information structure: same fields as §6.3.1.2 but adapted for SBI — IMS Node Functionality (Om), User Information (User Identifier + Equipment Info), UE Time Zone (vs MS Time Zone), Serving Node Address (vs GGSN Address). Per-operation matrix (Table 6.4.2.3.1): Session Identifier present only from Update onward (-UTE); Triggers and Multiple Unit Usage present only at Update/Terminate (-UT-); Cause Code/Reason Header at Terminate/Event only (--TE); IMS Communication Service ID only at Update (-U--). Response: Session Failover only at Initial (I---); Triggers not present; Multiple Unit information at Initial/Event (I--E). Formal encoding → TS 32.298 + TS 32.291.

**Annex B — OCS service termination flows (informative):** 5 main scenarios (B.1–B.5), 9 sub-flows total. B.1 SCUR/Initial INVITE: OCS rejects at Initial CCR → 4xx/5xx/6xx to UE, no dialog. B.2 SCUR/Early dialog: 3 sub-scenarios (200 OK to INVITE, SIP UPDATE, 200 OK to UPDATE) — each results in no established session; OCS returns Update Termination → BYE or CANCEL toward UE. B.3 SCUR/Confirmed dialog: 3 sub-scenarios (re-INVITE, 200 OK to re-INVITE, autonomous quota/reauth trigger) — session is released cleanly; B.3.3 is notably different in that no SIP request triggered the Ro Update — the IMS-GWF initiates the CCR autonomously. B.4 ECUR/non-INVITE: Reserve[Initial] blocked by OCS → 4xx to UE. B.5 IEC/non-INVITE: Debit[Event] blocked by OCS → 4xx to UE; simplest 4-step flow.

Notable findings:
- **Online Charging Flag is not a proof of action** (§6.3.1.2 note): its presence in the CDR indicates that an ECF address was present in P-Charging-Function-Addresses, not that Ro credit control was actually performed. An auditor cannot rely on this flag alone to determine if online charging occurred.
- **Annex B.3.3 autonomous termination is the "quota expiry" pattern**: when quota is exhausted or a re-auth trigger fires, the IMS-GWF can initiate an Update CCR without any UE action. The OCS response contains Service Termination and the IMS-GWF generates the BYE on both legs. This is distinct from FUI-driven termination (which is a CCA-driven action on an existing CCR) — here the CTF initiates the CCR specifically because of a charging event.
- **I-CSCF and BGCF generate Event-only CDRs in offline charging** — their operation type is E only in Table 6.3.2.1. This confirms that they never generate session CDRs (S/I/S); their CDR role is purely to record routing decisions at specific SIP transaction points.
- **§6.4 converged charging Request is FFS** — both the Charging Data Request and Response full structures are marked "Editor's Note: FFS" in Release 17. The tables provided (6.4.1.2.1, 6.4.1.2.2, 6.4.2.3.1, 6.4.2.3.2) define the IMS-specific view, but TS 32.290 is the normative base. This means Release 17 implementations must consult TS 32.290 for the complete SBI message encoding.

TS 32.260 is now **COMPLETE** — all 6 chunks ingested.

Next chunk: Review remaining sources. TS 23.216 (SRVCC) §6.4–§9 and TS 23.402 §9+ are both IN PROGRESS per the source pages. Suggest: ts23216-3 — §6.4 CS-to-PS SRVCC + §6.5 SRVCC failure + §7–§9 (vSRVCC variants, 5G SRVCC, call flows).
Pages touched: [wiki/concepts/IMS-charging-information.md (new), wiki/protocols/Ro-online-charging.md (updated — §6.2 IMS Ro content + updated sources/tags), wiki/procedures/IMS-online-charging-flows.md (extended — Annex B OCS termination flows), wiki/sources/ts32260.md (updated — ts32260-6 Done, COMPLETE), wiki/index.md (updated, 78→80 pages)]

## [2026-04-13] ingest | TS 23.402 §9–§13 | chunk ts23402-6

Ingested §9 (HRPD optimized handover), §10 (WiMAX HO principles), §11 (empty), §12 (HSS/AAA server interactions), and §13 (non-3GPP information storage) from TS 23.402.

**§9 — HRPD optimized handover:** Defines S101 (MME↔HRPD AN, S101-AP/UDP) and S103 (S-GW↔HS-GW, GRE RFC 2784/2890) reference points. Three procedure flows: (1) Pre-registration 9-step: UE registers to HRPD from E-UTRAN via S101 tunnel; EAP-AKA at HS-GW+AAA; Gateway Control Session established; S101 Session ID stored at MME. (2) Active HO 19-step: GRE key allocation → S103 DL forwarding tunnel pre-established → HS-GW sends Proxy Binding Update with all-zero CoA (0.0.0.0) to PGW (session continuation signal; PGW redirects without changing UE IP) → A11 signalling → HO Complete → MME stops S103 forwarding. (3) Idle-mode 8-step: ECM-IDLE UE reselects HRPD → A11 Reg Req → HS-GW fetches PGW identity from AAA (which gets it from HSS via SWx) → PMIP BU/BA → PCEF IP-CAN Modification → E-UTRAN resources deactivated. S101 Tunnel Redirection (§9.7): on TAU with MME change, new MME sends Notification Request (Redirection) to HRPD AN with new MME address; S101 Session ID preserved.

**§10 — WiMAX HO:** ANDSF-driven dual-radio HO; reuses S2a/S2c procedures; no additional procedure flows.

**§12 — HSS/AAA interactions:** UE Registration Notification (AAA→HSS after EAP-AKA); AAA-initiated De-registration; HSS-initiated De-registration (Push-Notification); PDN GW Identity Notification from AAA (§12.4) and MME/SGSN cascade (§12.5, with mandatory ePDG/TWAN notification if simultaneously connected); User Profile Update; Provide User Profile; Authentication → TS 33.402.

**§13 — Information Storage:** HSS: 3GPP AAA Server name (FQDN), QoS Profile per access type, ODB, Access Restriction. MME: S101 Source IP per UE, S103 Forwarding Address per PDN, S103 GRE Keys per PDN. S-GW: S103 Forwarding Address + GRE keys per PDN. Wild Card APN.

Notable findings:
- **All-zero CoA is the PGW session-continuation signal**: HS-GW uses CoA=0.0.0.0 in PMIPv6 Proxy Binding Update to PGW during HRPD active HO. This tells the PGW to switch its downlink path to HS-GW without creating a new PDN context or reallocating UE IP. A non-zero CoA would trigger new context creation.
- **S103 GRE keys are stored at MME, not S-GW**: the keys are negotiated via the HS-GW during HO preparation and stored in the MME's UE context. The S-GW receives them from the MME and uses them for the forwarding tunnel. This means S-GW has no independent forwarding state — it follows MME instructions.
- **§12.5 cascade is mandatory if UE is simultaneously connected**: when a 3GPP MME assigns a PGW, it MUST cascade the PGW identity to ePDG/TWAN if the UE has simultaneous non-3GPP connections. Failure to do so would result in the MAG sending PMIPv6 PBUs to a different PGW than the one holding the UE's IP address — silent session split.
- **Wild Card APN prevents attach failures**: allows the HSS to grant non-3GPP connectivity even when the UE requests an APN that is not explicitly in its subscription, as long as the operator has provisioned wildcard access.

Next chunk: ts23402-7 — TS 23.402 §16 TWAN multi-connection procedures (S2a with multi-PDN TWAN context, NSWO, SaMOG architecture).
Pages touched: [wiki/procedures/HRPD-optimized-handover.md (new), wiki/concepts/non-3GPP-access-architecture.md (updated — §12 and §13 appended), wiki/sources/ts23402-section4.md (updated — ts23402-6 Done), wiki/index.md (updated, 80→81 pages, ts23216 stale entry fixed)]

## [2026-04-13] ingest | TS 23.402 §13.5–§13.6 + §15 + §16 + §17 | chunk ts23402-7

Ingested §13.5 (ePDG emergency config data), §13.6 (TWAN emergency config data), §14 (Void), §15.1 (S2c DSMIPv6 bootstrapping), §16 (TWAN GTP/PMIPv6 S2a full procedures), §17 (E-UTRAN-HRPD Inter-RAT SON), and Annexes A/B/C (informative). This completes TS 23.402.

**§13.5–§13.6:** Emergency Configuration Data for ePDG and TWAN: 5 fields each (Emergency APN, QoS profile, APN-AMBR, PDN GW identity, fallback PDN GW identity). Used instead of HSS subscription data for emergency PDN connections.

**§15.1 S2c Bootstrapping via DSMIPv6 Home Link:** UE already on 3GPP PDN establishes S2c IKEv2 SA to PGW to prepare future non-3GPP HO. 3 steps: PGW discovery → IKEv2 SA + IPv6 HoA allocation (EAP auth with AAA/HSS) → Home Link Detection. One SA per PDN connection required.

**§16 TWAN Architecture:** TWAN = WLAN AN + TWAG (S2a termination) + TWAP (STa relay). Three connection modes negotiated via EAP-AKA': TSCM (default, no APN/HO-indicator from UE), SCM (single PDN), MCM (multiple PDN + NSWO via WLCP). WLCP is a 3GPP-defined control protocol between UE and TWAG over DTLS/UDP/IP; carries PDN Connection ID (TWAG-allocated MAC address), APN, PDN type, TFT, Bearer QoS. Bearer model: TSCM/SCM = single point-to-point UE-TWAN (TFT routing like ePDG); MCM = one WLCP bearer per S2a bearer 1:1. TWAN Identifier (SSID + BSSID/civic/logical) reported over S2a/Gx/Gy for NPLI. STa extended for TWAN: connection mode indication, PDN addresses, TWAG WLCP IP, SM back-off timer, emergency support.

**§16.2 Initial Attach:** GTP S2a 15-step with Scenario A (L2 trigger, recommended) and B (DHCPv4 L3 trigger, TSCM IPv4 only). EAP-AKA' extended: UE→network = mode, IMEI(SV), APN, PDN type, HO indicator; network→UE = supported modes, TWAG WLCP IP (MCM), PDN addr (SCM), SM back-off. Handover Indication in Create Session Request → PCEF IP-CAN Modification (not Establishment) → UE keeps same IP. Create Session Response carries same Charging ID as 3GPP default bearer. PMIP variant: PBU includes TWAN Identifier + IMEI(SV).

**§16.3–§16.6 Bearer management:** Detach GTP 6-step (Delete Session includes TWAN ID + UE Time Zone); HSS/AAA-initiated Detach via Session Termination Request → §16.3.1.1 steps. PGW-initiated bearer deactivation (§16.4): PCRF → Delete Bearer → MCM WLCP PDN/Bearer Disconnect ↔ UE → Delete Bearer Response; PMIP = Binding Revocation. Dedicated bearer activation (§16.5): Create Bearer → MCM WLCP Bearer Creation ↔ UE; Charging ID reuse from 3GPP HO bearers. Bearer modification (§16.6): PGW-initiated Update Bearer + WLCP Bearer Update; HSS-initiated via Modify Bearer Command.

**§16.7–§16.9 MCM-specific procedures:** MCM detach = Delete Session per PDN. UE-initiated PDN connectivity (§16.8) 7-step: WLCP PDN Connection Request → Create Session → IP-CAN → WLCP PDN Connection Response (PDN Connection ID = TWAG MAC, PDN addr). PDN disconnection (§16.9) 9-step: WLCP PDN Disconnection Request(PDN Connection ID) → Delete Session → WLCP PDN Disconnection Response.

**§16.10–16.11 Handover:** 3GPP→TWAN SCM GTP 11-step: HO Indicator in Create Session → PCEF IP-CAN Modification (IP preserved, same Charging ID) → GTP tunnel → EAP Completion → PGW deactivates 3GPP bearers. MCM 6-step: EAP auth MCM + WLCP PDN per PDN (Request Type=Handover). TWAN→3GPP: §8.2.1.x with TWAN resource deactivation via §16.4; Charging ID transferred to 3GPP bearers.

**§17:** S121 reference point (MME↔HRPD AN, S121-AP/UDP); MME relays RIM messages transparently between S1 (eNB) and S121 (HRPD AN) for SON neighbor optimization.

Notable findings:
- **PDN Connection ID is a TWAG MAC address, not a numeric ID**: the TWAG allocates a MAC address per PDN connection for MCM. The UE and TWAN use this MAC address to multiplex user plane packets across multiple PDN connections. This is fundamentally different from 3GPP where EPS Bearer IDs are integers — the MAC-based multiplexing is specific to 802.11 point-to-point link management.
- **TSCM is the fallback mode**: if neither UE nor network indicates otherwise, TSCM is used. In TSCM the UE cannot request a specific APN, indicate a handover, or get IP address preservation — it simply attaches to the default APN. This means TSCM is not suitable for MAPCON or for VoLTE handover from 3GPP.
- **PMIP-based S5/S8 cannot coexist with dynamic PCC in TWAN SCM**: a note in §16.2.1 states that PMIP-based S5/S8 with dynamic PCC cannot be deployed because it will result in wrong session linking between Gateway Control Session and Gx session in PCRF.
- **S121 adds a third HRPD-related reference point**: §9 defined S101 (pre-registration signalling) and S103 (data forwarding). §17 adds S121 (SON optimization) — all three use the MME as the E-UTRAN-side relay, confirming the MME's role as the single integration point for HRPD inter-RAT functions.

TS 23.402 is now **COMPLETE** — all normative sections ingested across chunks 3b-1 through ts23402-7.
Pages touched: [wiki/procedures/TWAN-S2a-procedures.md (new), wiki/sources/ts23402-section4.md (updated — ts23402-7 Done, COMPLETE), wiki/index.md (updated, 81→82 pages)]

## [2026-04-14] ingest | TS 29.328 §3–§6 — Sh signalling flows and procedure descriptions | chunk ts29328-1

Ingested §3 (definitions: transparent/non-transparent data), §4 (main concept), §5 (general architecture: AS/HSS/Presence Network Agent functional requirements; Sh procedure functional classification), and §6 (all 4 procedure descriptions: Sh-Pull, Sh-Update, Sh-Subs-Notif, Sh-Notif, AS permissions list, SLF/Dh resolution) from TS 29.328 v16.1.0.

**§5 Architecture:** Ph interface (HSS ↔ Presence Network Agent) is functionally identical to Sh — all AS rules apply to Presence Network Agent. Sh procedures are classified into: (1) Data handling — Sh-Pull (download) and Sh-Update (write); (2) Subscription/notification — Sh-Subs-Notif (subscribe) and Sh-Notif (HSS push).

**§6.1.1 Sh-Pull (UDR/UDA):** AS reads transparent and/or non-transparent data for a user. Full 19-IE request table and 4-IE response table. HSS processing: (1) AS permissions check; (2) User Identity exists; (2a) Private Identity correspondence; (3) User Identity type per Table 7.6.1; (3a) IPAddressSecureBindingInfo shared-IMPU restriction; (4) concurrent update conflict; (4a) T-ADS: contact MME/SGSN/AMF for IMS VoPS indication (Gn/Gp-SGSN exception applies); (4b) CSRN/MTRR empty element handling; (5) return User-Data AVP with unavailable data represented by empty XML elements.

**§6.1.2 Sh-Update (PUR/PUA):** AS writes data. Sequence-Number 0 = new creation (reserved); n+1 for modification/deletion (wraps 65535→1). Special handling for: PSIActivation (ACTIVE→INACTIVE triggers network-initiated deregistration); DSAI (value change triggers iFC mask/unmask and Sh-Notif/Cx-Update cascade); SMSRegistrationInfo (IP-SM-GW number/Diameter Identity; preconfigured values not overwritten; Service Centre Address not writable via Sh); STN-SR (HSS + UDM interworking via TS 23.632). Update-Eff and Update-Eff-Enhance atomicity for multi-instance repository updates. Repository sequence-number optimistic concurrency: TRANSPARENT_DATA_OUT_OF_SYNC (5105) if version mismatch.

**§6.1.3 Sh-Subs-Notif (SNR/SNA):** AS subscribes to data-change notifications. Subscription-request-type distinguishes Subscribe(0) from Unsubscribe(1). One-Time-Notification shall be present for UE reachability for IP subscriptions (ends subscription after first notification). Expiry-Time handling: HSS may set earlier than requested; unlimited if absent. Notif-Eff feature enables multi-Data-Reference and multi-Service-Indication atomic subscription requests.

**§6.1.4 Sh-Notif (PNR/PNA):** HSS pushes change notifications — HSS is Diameter client; AS is server. Authentication pending state transitions are NOT notified. Removed data encoded via empty XML elements (no ServiceData child, empty SCSCFName, empty IPv4Address/IPv6Prefix, empty IFCs). DeletedIdentities element signals removal of Public Identity with active subscriptions. If AS returns DIAMETER_ERROR_USER_UNKNOWN, HSS removes ALL subscriptions for that AS + User Identity.

**§6.2 AS Permissions List:** System-wide (not per-user) table: AS Identity (Origin-Host) × Data-Reference → allowed operations (Sh-Pull / Sh-Update / Sh-Subs-Notif / combination). Permission revocation triggers removal of all active subscriptions for that AS.

**§6.5 SLF/Dh resolution:** Dh interface uses Diameter redirect agent (SLF) to resolve User Identity to HSS identity in multi-HSS deployments. AS then routes Sh requests directly to identified HSS.

Notable findings:
- **PSIActivation write triggers deregistration:** Changing PSI state ACTIVE→INACTIVE via Sh-Update causes the HSS to initiate network-initiated deregistration of that PSI — a direct consequence of an AS Diameter write, not a SIP signalling event.
- **DSAI write cascades to Cx:** Sh-Update of DSAI triggers iFC mask/unmask evaluation and may cause the HSS to send Cx-Update_Subscr_Data to the S-CSCF — bridging the Sh (AS→HSS) and Cx (HSS→S-CSCF) interfaces.
- **SMSRegistrationInfo write is selective:** The IP-SM-GW Diameter Identity (from Origin-Host + Origin-Realm) can be written via Sh, but the preconfigured number is never overwritten, and the Service Centre Address field is read-only on Sh (only writable via MAP/SS7 procedures).
- **T-ADS Gn/Gp exception:** If a Gn/Gp-SGSN is registered, the HSS does NOT additionally contact the MME for T-ADS — the SGSN's indication takes precedence, preventing duplicate/conflicting IMS VoPS capability reports that could cause the AS to make wrong codec/bearer decisions.

Next chunk: ts29328-2 — TS 29.328 §7–§9 + Annex A (all 23 IE contents, protocol version, Diameter command mapping)
Pages touched: [wiki/procedures/Sh-signalling-flows.md (new), wiki/protocols/Sh-Diameter.md (updated — TS 29.328 source + reference to signalling-flows page), wiki/index.md (updated, 84→85 pages), wiki/log.md (this entry)]

## [2026-04-14] ingest | TS 29.328 §7–§9 + Annex A — IE contents and Diameter mapping | chunk ts29328-2

Ingested §6.5 continued (SLF redirect agent details), §7 (all information element contents: Table 7.6.1 + §7.1–§7.23), §8 (protocol version → TS 29.329), §9 (operational aspects → TS 29.329), and Annex A normative (Sh-to-Diameter command mapping).

**§6.5 SLF detail:** SLF as Diameter redirect agent may return multiple HSS identities in ordered list; AS tries first then tries next in order until successful. Diameter Proxy Agent may also consider load values (RFC 8583 Load AVPs, HOST type) for HSS selection. AS should cache HSS identity/Realm for future Sh requests on same IMPU.

**§7.1 User Identity:** Grouped AVP containing one of: IMS Public User Identity/PSI (Public-Identity), MSISDN, or External Identifier. Each conditional per Table 7.6.1.

**Table 7.6.1 (complete — 35 Data-References, 0–35, value 20 reserved):** Full per-Data-Reference access key and operations matrix. Key structural rules: (1) RepositoryData (0) access key requires Service-Indication in addition to User-Identity — all IMPUs in an Alias PUIS set share the same repository data; (2) PSI operations apply at Wildcarded PSI granularity, not per-individual PSI; (3) IMSI (32) and IMSPrivateUserIdentity (33) are flagged sensitive — operator may restrict via AS Permissions List; (4) several data types (T-ADS, UE reachability, STN-SR, SRVCC capabilities, CSRN, location) require Private Identity when referencing a specific Private Identity within a multi-IMPI subscriber.

**§7.6.2 IMSPublicIdentity:** Four Requested-Identity-Set variants: IMPLICIT_IDENTITIES (same implicit registration set), ALIAS_IDENTITIES (same Alias PUIS), REGISTERED_IDENTITIES (registered state across all Private Identities), ALL_IDENTITIES (all non-barred IMPUs for all Private Identities). Default (not included) = ALL_IDENTITIES.

**§7.6.3 IMS User State:** Four values: REGISTERED, NOT_REGISTERED, AUTHENTICATION_PENDING, REGISTERED_UNREG_SERVICES. For shared IMPUs (multiple IMPI associations): HSS reports most-registered state using precedence REGISTERED > REGISTERED_UNREG_SERVICES > AUTHENTICATION_PENDING > NOT_REGISTERED.

**§7.6.6 Location Information:** 5 sub-domains (CS/GPRS/EPS/TWAN/5GS), each with distinct sub-elements. Fallback to locally stored location when serving node unreachable. Serving Node Indication suppresses detailed location retrieval from serving nodes (serving node address only). CS location has SGs-mode exception: when MSC uses SGs interface, E-UTRAN Cell Global ID + TAI replace Location number/Service area ID/CGI/LAI.

**§7.6.8 Charging Information:** Four charging function addresses. Clash rule: ISC addresses take precedence over Sh-received addresses. Sh is a special-case path (pre-REGISTER, not general-purpose Rf/Ro alternative).

**§7.6.11 DSAI:** Each DSAI implicitly bound to ≥1 iFC; all iFCs bound to same DSAI should share the same AS ServerName. iFC included in Cx Service-Profile iff: no DSAI bound OR at least one bound DSAI is ACTIVE. Wildcarded PSI: masking from any identity in wildcarded set applies to all identities in that set.

**§7.6.17 UE Reachability for IP:** Four sub-elements (MME, SGSN, AMF-3GPP, AMF-non-3GPP), each with value REACHABLE(0). Reflects URRP-MME/URRP-SGSN parameter changes. Used with One-Time-Notification for IP reachability subscriptions.

**§7.6.18 T-ADS:** Three values for IMS VoPS support: NOT_SUPPORTED(0), SUPPORTED(1), UNKNOWN(2). HSS uses S6a/S6d/MAP/Namf to retrieve from serving nodes.

**§7.6.20 STN-SR write cascade:** When STN-SR updated via Sh-Update, HSS uses S6a/S6d-IDR to push to MME/SGSN AND (if UDM interworking) UDM uses Nudm_MT_Update to push to AMF — one Sh write cascades to three serving nodes.

**§7.20 UDR Flags:** Two flags defined: Location-Information-EPS-Supported (allows EPS location info alongside CS location request, HSS informs MSC/VLR of EPS capability) and RAT-Type-Requested (requests RAT type alongside PS/EPS location info).

**Annex A:** Normative Sh-to-Diameter command mapping (Table A.2.1): 4 procedures × 2 directions = 8 message mappings (Sh-Pull↔UDR/UDA, Sh-Update↔PUR/PUA, Sh-Subs-Notif↔SNR/SNA, Sh-Notif↔PNR/PNA).

Notable findings:
- **IMS User State precedence for shared IMPUs is asymmetric:** REGISTERED_UNREG_SERVICES outranks AUTHENTICATION_PENDING. This means an IMPU that is in unregistered-services state for one IMPI but being authenticated for another reports REGISTERED_UNREG_SERVICES — the AS sees unregistered-services state even though auth is in progress on another IMPI.
- **DSAI masking scope with Wildcarded PSI:** A DSAI mask set for any individual PSI in a wildcarded set silently applies to all PSIs in the wildcarded set. An operator managing DSAI for a hosted AS service must be aware that the first masked PSI masks the entire wildcard range.
- **UDR Flags allow mixed CS+EPS location in one Sh-Pull:** Setting Location-Information-EPS-Supported allows the MSC/VLR to return both CS location info AND EPS location info in the same response — useful for IMS emergency services needing UE's current RAT.
- **STN-SR write is a 3-node cascade:** One Sh-Update sets off a chain: HSS → S6a/S6d-IDR → MME/SGSN, and in 5G deployments additionally UDM → Nudm_MT_Update → AMF. The Sh-Update response is returned to the AS before all cascades complete.

Next chunk: ts29328-3 — Annexes B–J (message flow examples, UML model, XML schema, T-ADS HSS handling, overload/priority/load control)
Pages touched: [wiki/concepts/Sh-user-profile-data.md (new), wiki/sources/ts29328.md (new), wiki/index.md (updated, 85→87 pages), wiki/log.md (this entry)]

## [2026-04-14] ingest | TS 29.328 Annexes B–J — UML model, XML schema, T-ADS algorithm, resilience mechanisms | chunk ts29328-3

Ingested all remaining annexes of TS 29.328 v16.1.0 (spec pages 48–82): Annex B (informative message flow), Annex C (informative UML class model), Annex D (normative XML schema), Annex E (informative T-ADS HSS algorithm), Annex F (normative Diameter overload control), Annex G (informative overload node behaviour), Annex H (informative repository data sharing), Annex I (normative Diameter message priority), Annex J (normative Diameter load control). Annex K (change history) skipped.

**Annex B (message flow):** 13-step composite flow illustrating IMS registration → third-party registration → AS Sh-Subs-Notif subscription with data download → Sh-Update → Sh-Notif cycle. Confirms the flow already captured in `procedures/Sh-signalling-flows.md`; no new normative content.

**Annex C (UML model):** Full class hierarchy of `Sh-Data` root element added as Mermaid graph to `concepts/Sh-user-profile-data.md`. Key rule: when Notif-Eff or multiple Identity Sets are involved, `PublicIdentifiers` carries only MSISDNs; IMS identities travel via RegisteredIdentities/ImplicitIdentities/AllIdentities/AliasIdentities. `EnhancedIMSPublicIdentifiers` class used when mixed identity types (distinct PUI + Wildcarded PUI) must be returned together. DeletedIdentities used when HSS deletes a public identity while AS has active subscription.

**Annex D (XML schema):** Tables D.1/D.2 added as concise reference tables. Key normative enumerations: tDirectionOfRequest (0–4: ORIGINATING/TERMINATING/TERMINATING_UNREGISTERED/ORIGINATING_UNREGISTERED/ORIGINATING_CDIV), tPSUserState (0–6 including value 5=NotProvided-from-SGSN-or-MME-or-AMF), tServicePriorityLevel (0–4, 0=highest). Extension chain (tSh-Data-Extension1–7 and tShIMSDataExtension1–7) documented — each layer adds fields introduced in later releases without breaking backward compatibility.

**Annex E (T-ADS algorithm):** Decision flowchart for HSS T-ADS request handling added to `concepts/Sh-user-profile-data.md`. Algorithm: (1) if no S4/MME registered → NOT_SUPPORTED; (2) Gn/Gp-SGSN suppresses MME query; (3) if all registered nodes sent consistent Homogeneous-Support-IMS-VoPS in ULR → return that value directly; (4) otherwise query nodes that support active T-ADS retrieval; (5) if two responses → use response from node with most recent UE Activity Time; (6) no response from queried node → UNKNOWN. UE Activity Time and Last RAT Type are co-reported with T-ADS value.

**Annex F/G (overload, RFC 7683):** HSS=reporting node, AS=reacting node. HSS includes OC-OLR AVP in answer messages to request traffic reduction. AS throttles via prioritized deferral. MPS/emergency traffic (identified by SIP Resource-Priority header) is last to be throttled. Annex G guidance: defer Subs-Notif before Sh-Pull; deprioritize bulk provisioning updates vs. live-call retrieval.

**Annex I (DRMP priority, RFC 7944):** DRMP AVP (code 301) signals per-message priority for routing, resource allocation, and DSCP marking. If request contains both Session-Priority and DRMP, DRMP takes precedence. MPS/emergency Sh requests shall include DRMP with high priority.

**Annex J (load control, RFC 8583):** Passive load advertisement via Load AVP of type HOST in HSS answers. Distinct from overload control (Annex F): OC-OLR actively commands rate reduction; Load AVP passively informs agent-selection decisions. AS uses PEER-type Load AVPs from candidate next-hop Diameter Agents for routing.

Notable findings:
- **XML Extension chain is the backward-compatibility mechanism:** All new fields added post-Rel-8 go into progressively numbered Extension elements rather than directly into tSh-Data. A parser compiled against any earlier release schema silently ignores newer Extension nodes. This is the normative extensibility pattern for the Sh XML profile.
- **T-ADS Gn/Gp exception is asymmetric:** If both MME and Gn/Gp-SGSN are registered, the HSS treats MME as not registered for T-ADS purposes and uses only the SGSN's homogeneous indication. The reverse does not apply — a Gn/Gp-SGSN can independently indicate NOT_SUPPORTED without suppressing other nodes.
- **DRMP overrides Session-Priority:** If both are present in the same request, DRMP wins. This is architecturally significant because Session-Priority (TS 29.229) predates DRMP (RFC 7944) — the newer mechanism takes precedence.
- **Load AVP (Annex J) vs OC-OLR (Annex F) coexist:** Both may appear in the same answer message. They address different concerns: OC-OLR is an active request to reduce request rate; Load is passive telemetry for path selection.

Next chunk: — (TS 29.328 is now COMPLETE; next source to ingest TBD by user)
Pages touched: [wiki/concepts/Sh-user-profile-data.md (updated — Annexes C/D/E/H added), wiki/protocols/Sh-Diameter.md (updated — Annexes F/G/I/J added), wiki/sources/ts29328.md (updated — COMPLETE), wiki/index.md (updated), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.229 §1–§6.2 — Cx/Dx Diameter commands and result codes | chunk ts29229-1

Ingested TS 29.229 v16.2.0 §1–§6.2 (PDF pages 8–21): scope, Diameter base protocol usage (§5), all 12 Cx command definitions with full ABNF (§6.1), and all 15 Cx-specific result codes (§6.2). Created new protocol page `protocols/Cx-Diameter.md` and source summary `sources/ts29229.md`.

**§5 Base protocol usage:** Sessions are implicitly terminated (NO_STATE_MAINTAINED). Accounting not used. Transport: SCTP. Routing: I-CSCF/S-CSCF include Destination-Host only when HSS address already known; HSS-initiated commands (RTR/PPR) always include Destination-Host. Application advertised via Auth-Application-Id = 16777216 in Vendor-Specific-Application-Id.

**§6.1 Commands (Application-ID 16777216):** 6 request/answer pairs — UAR/UAA (code 300, I-CSCF→HSS, authorize registration), SAR/SAA (301, S-CSCF→HSS, store server name + download service profile), LIR/LIA (302, I-CSCF→HSS, locate serving S-CSCF for terminating routing), MAR/MAA (303, S-CSCF→HSS, fetch IMS AKA authentication vectors), RTR/RTA (304, HSS→S-CSCF, network de-registration), PPR/PPA (305, HSS→S-CSCF, push updated profile or SIP Digest credentials). Full ABNF for all 12 messages captured with key AVPs annotated.

**§6.2 Result codes:** 4 success codes (2001 FIRST_REGISTRATION, 2002 SUBSEQUENT_REGISTRATION, 2003 UNREGISTERED_SERVICE, 2004 SUCCESS_SERVER_NAME_NOT_STORED) and 11 permanent failure codes (5001–5009, 5011–5012) defined. All carried in Experimental-Result AVP.

Notable findings:
- **RTR and PPR are the only HSS-initiated Cx commands:** All other Cx commands are CSCF-initiated. This asymmetry means the HSS must maintain the S-CSCF address per registration to be able to target PPR/RTR.
- **2003 UNREGISTERED_SERVICE vs 2001 FIRST_REGISTRATION distinction is critical for iFC:** When HSS returns 2003, the I-CSCF assigns an S-CSCF to handle unregistered-state services (e.g., voicemail termination to an unregistered user) — the UE never sent a REGISTER.
- **5012 SERVING_NODE_FEATURE_UNSUPPORTED is specific to P-CSCF Restoration (TS 23.380):** This error tells the S-CSCF that HSS has the feature enabled but no serving node can honour it — not a general-purpose feature-mismatch code.
- **5007 ERROR_IN_ASSIGNMENT_TYPE handles re-registration edge cases:** Returned when the S-CSCF trying to register is the same server already stored, but the registration state/Public-Identity type doesn't permit the requested Server-Assignment-Type. Distinguishes from 5005 (different server trying to overwrite).

Next chunk: ts29229-2 — §6.3 AVP definitions (Table 6.3.1 + 71 AVPs), §6.4 namespaces, §7 version/feature control
Pages touched: [wiki/protocols/Cx-Diameter.md (new), wiki/sources/ts29229.md (new), wiki/index.md (updated, 87→89 pages), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.229 §6.3–§7 — AVP reference, namespaces, feature list, versioning | chunk ts29229-2

Ingested TS 29.229 v16.2.0 §6.3–§7 (PDF pages 21–43): full AVP table (Table 6.3.1), all 71 AVP subsections, §6.4 namespace codes, §7 version control and supported-features mechanism. Updated `protocols/Cx-Diameter.md` with all content.

**§6.3 AVP Table:** 61 named entries in Table 6.3.1 spanning codes 104–661 (plus ETSI code 500 and IETF RFC codes 301/104/110/111/121). All Cx-defined AVPs use Vendor-ID 10415; Line-Identifier (500) uses Vendor-ID 13019 (ETSI). Nine IETF-referenced AVPs (Framed-IP-Address/IPv6, DRMP, Load, Digest-Realm/Algorithm/QoP/HA1) reference their respective RFCs. Full grouped AVP ABNF captured for key compound AVPs: SIP-Auth-Data-Item (612), Deregistration-Reason (615), Charging-Information (618), Supported-Features (628), SCSCF-Restoration-Info (639), Restoration-Info (649).

**§6.3.15 Server-Assignment-Type:** 15 values (0–14) covering normal registration lifecycle (0–5), store-server-name variants for P-CSCF Restoration (6–7), admin deregistration (8), auth failures (9–10), data overflow (11), and SWx-only values (12–14) not used in Cx.

**§6.3.17 Reason-Code:** 4 values — PERMANENT_TERMINATION(0), NEW_SERVER_ASSIGNED(1), SERVER_CHANGE(2), REMOVE_S-CSCF(3) — carried in Deregistration-Reason inside RTR.

**§7.1.1 Feature-List-ID 1:** 4 defined features — SiFC(bit 0, Optional): shared iFC sets downloaded as identifiers rather than full XML; AliasInd(bit 1, Mandatory): alias group ID in service profiles; IMSRestorationInd(bit 2, Optional): S-CSCF sends restoration info on P-CSCF failure, I-CSCF triggers re-assignment; P-CSCF-Restoration-mechanism(bit 3, Optional): HSS-based restoration via SAR-Flags bit and SCSCF-Restoration-Info in SAA.

**§7.2 Supported features protocol:** M-bit in Supported-Features means feature is required to process the request. Answers always list the full supported feature set. If M-bit feature unsupported: DIAMETER_ERROR_FEATURE_UNSUPPORTED + list. If Supported-Features AVP unknown to receiver: DIAMETER_AVP_UNSUPPORTED + Failed-AVP. Sender may retry with only common features.

Notable findings:
- **SIP-Authentication-Scheme "Digest-AKAv1-MD5" covers both AKAv1 and AKAv2:** The S-CSCF uses this single scheme string for both IMS-AKA versions because the HSS handles both the same way. The version differentiation is internal to the UE-side security algorithm.
- **User-Data-Already-Available controls profile download scope:** When S-CSCF sends SAR with USER_DATA_ALREADY_AVAILABLE=1, the HSS omits the User-Data AVP from SAA. This is the mechanism by which re-registration avoids redundant profile re-download.
- **Session-Priority placed near Diameter header for fast processing:** The spec explicitly recommends placing Session-Priority as close to the header as possible so receivers can prioritize resource allocation before full message parsing.
- **SAR-Flags bit 0 links P-CSCF Restoration to HSS:** This flag is the trigger for the HSS to include SCSCF-Restoration-Info in the SAA so the new S-CSCF (after P-CSCF failure) gets the needed Path/Contact/CSeq restoration context. It only applies when SAT is ADMINISTRATIVE_DEREGISTRATION or UNREGISTERED_USER.
- **AliasInd is the only mandatory (M) feature:** All other Feature-List-ID 1 features are optional. This means HSS and S-CSCF must agree on alias handling, but shared iFCs and P-CSCF Restoration are negotiated capabilities.

Next chunk: — (TS 29.229 is now COMPLETE; next source to ingest TBD by user)
Pages touched: [wiki/protocols/Cx-Diameter.md (updated — AVP table, SAT values, Reason-Code, features, namespaces added), wiki/sources/ts29229.md (updated — COMPLETE), wiki/index.md (updated), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.228 §5–§8 — Cx/Dx signalling flows, all procedure IE tables, HSS detailed behaviour | chunk ts29228-1

Ingested TS 29.228 v18.0.0 §1–§8 (PDF pages 9–52): §5 general architecture and Cx procedure classification, §6 all six Cx procedure descriptions with complete IE tables and HSS processing algorithms, §7 information element definitions (§7.1–§7.31), §8 error handling. Created new procedures page and source summary.

**§6.1.1 UAR/UAA**: 6-step HSS algorithm covering identity/association checks, barring check (bypassed for emergency registration), roaming check (bypassed for emergency), S-CSCF state inspection. Key: SUBSEQUENT_REGISTRATION returned whenever any identity in the IMS subscription has an S-CSCF assigned.

**§6.1.2 SAR/SAA**: Most complex Cx procedure — 10-column SAR request table, 12-column SAA response table. HSS processing covers all 14 Server-Assignment-Type values. P-CSCF Restoration via SAR-Flags triggers direct HSS notification to SGSN/MME (S6a/S6d IDR/IDA) and SMF/AMF (Nudm). Wildcarded PUI matching rules: S-CSCF must use Wildcarded-Public-Identity if received from I-CSCF; must NOT perform wildcarded matching if not received.

**§6.1.3 RTR/RTA**: 4 reason codes (PERMANENT_TERMINATION/NEW_SERVER_ASSIGNED/SERVER_CHANGE/REMOVE_S-CSCF). Emergency registration protection: identities emergency-registered in S-CSCF are immune to PERMANENT_TERMINATION and REMOVE_S-CSCF; HSS receives DIAMETER_UNABLE_TO_COMPLY or DIAMETER_LIMITED_SUCCESS with protected pairs.

**§6.1.4 LIR/LIA**: Location query including PSI direct routing (AS Name returned) and IMS Restoration trigger (UAT=REGISTRATION_AND_CAPABILITIES). LIA-Flags carries PSI Direct Routing Indication.

**§6.2 PPR/PPA**: HSS-initiated update; cannot modify IMSI. Valid PPA result codes table. USER_DATA_ALREADY_AVAILABLE suppresses profile re-download. DIAMETER_ERROR_TOO_MUCH_DATA → HSS initiates RTR SERVER_CHANGE → new S-CSCF selection.

**§6.3 MAR/MAA**: All authentication scheme IE tables (IMS-AKA, SIP Digest, NASS Bundled, GIBA). SIP Digest backward compat: Digest-Algorithm "MD5" only in Digest-Algorithm sub-AVP; Alternate Digest Algorithm coexists with primary in same grouped AVP.

**§6.4 SLF/Dx**: SLF as Diameter redirect agent vs Diameter Proxy Agent. S-CSCF stores HSS identity after first exchange; I-CSCF is stateless.

**§6.5 Implicit registration**: S-CSCF takes first non-barred distinct PUI as Default Public User Identity. Moving a PUI between implicit sets requires two-step: remove from old set (PPR without the identity), add to new set (PPR with the identity).

**§6.7 S-CSCF Capabilities**: Table 6.7 with 20 named capabilities (18 mandatory, 2 optional). Includes WebRTC (WAF/WWSF), GIBA, IMSI handling, Reference Location, simultaneous registrations limit, Extended Priority.

**§8.1 Error handling**: Old S-CSCF cancellation via RTR when new S-CSCF registered — with/without IMS Restoration Procedures the flow differs.

Notable findings:
- **S-CSCF reassignment pending flag is the HSS-side mechanism for IMS Restoration**: Flag set when UAT=REGISTRATION_AND_CAPABILITIES in UAR or when IMS Restoration supported + new S-CSCF encountered. The flag allows the HSS to permit an otherwise-rejected S-CSCF overwrite.
- **Emergency registration blocks de-registration at the Cx level**: Both PERMANENT_TERMINATION and REMOVE_S-CSCF RTR reason codes are blocked for emergency-registered identities. DIAMETER_LIMITED_SUCCESS (partial success) is a valid RTR answer code.
- **User-Data-Already-Available is per-request**: HSS MAY still override USER_DATA_ALREADY_AVAILABLE=1 and push data — the field is a hint, not a guarantee of data suppression.

Next chunk: ts29228-2 — Annex A (Cx→Diameter mapping, AVP mapping, 7 message flow scenarios) + Annex B (User Profile UML model) (PDF pages 53–67)
Pages touched: [wiki/procedures/Cx-signalling-flows.md (new), wiki/sources/ts29228.md (new), wiki/index.md (updated, 89→91 pages), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.228 Annex A + Annex B — Cx/Diameter mappings, 7 message flows, User Profile UML model | chunk ts29228-2

Ingested TS 29.228 v18.0.0 §8.1.2–§8.1.3 + §9–§10 (PDF page 52) and Annex A (normative, pages 53–60) + Annex B (informative, pages 61–67). Updated two existing wiki pages with new content sections.

**§8.1.2 Error in S-CSCF name**: SAR with differing S-CSCF name and no reassignment pending flag → for NO_ASSIGNMENT SAT: DIAMETER_UNABLE_TO_COMPLY; for all others: DIAMETER_ERROR_IDENTITY_ALREADY_REGISTERED. With IMS Restoration and reassignment pending flag set → overwrite allowed.

**§8.1.3 Error in S-CSCF assignment type**: SAT not applicable to current user state or identity type → DIAMETER_ERROR_IN_ASSIGNMENT_TYPE.

**§9–§10**: Both sections defer entirely to TS 29.229.

**Annex A.2 (normative)**: Complete Cx operation → Diameter command mapping table (12 entries, all 6 request/answer pairs). Stage-2 operation names mapped to stage-3 command names and abbreviations.

**Annex A.3 (normative)**: Complete Cx parameter → Diameter AVP mapping table (~35 entries). All information elements in TS 29.228 procedures mapped to their AVP names. Notable: "Routing Information" maps to Destination-Host; "Result" maps to Result-Code/Experimental-Result-Code (dual mapping); S-CSCF Name and AS Name both map to Server-Name.

**Annex A.4 — 7 message flow scenarios**: Key contrast between A.4.1 (first registration: UAR→MAR→challenge→UAR→SAR) and A.4.2 (re-registration: UAR→SAR only, no MAR). De-registration variants documented: UE-initiated (UAR→SAR USER_DEREGISTRATION), timeout (SAR alone), administrative (RTR→RTA), service-platform-initiated (SAR ADMINISTRATIVE_DEREGISTRATION). Two session setup flows: A.4.5 (registered user: LIR→LIA→INVITE) and A.4.6 (non-registered: LIR→LIA UNREGISTERED_SERVICE→SAR UNREGISTERED_USER→SAA). A.4.6a covers AS originating for unregistered user (two paths: via I-CSCF LIR/LIA, or directly to S-CSCF). A.4.7: PPR→PPA.

**Annex B — User Profile UML Model**: IMS Subscription → 1..n ServiceProfile + 0..1 ReferenceLocationInformation. ServiceProfile → 1..n PublicIdentity + 0..1 CoreNetworkServicesAuthorization + 0..n InitialFilterCriteria. PublicIdentity has 10 attributes including BarringIndication, IdentityType (5 values), AliasIdentityGroupId (alias feature), ServiceLevelTraceInfo, MaxNumOfAllowedSimultRegs, ExtendedPriority (0..n). CoreNetworkServicesAuthorization → SubscribedMediaProfileId + ListOfServiceIds. InitialFilterCriteria → TriggerPoint (0..1, ConditionTypeCNF boolean) + ApplicationServer (1, includes IncludeRegisterRequest/Response). SPT subtypes: Request-URI, SIP-Method, SIP-Header, Session-Case, Session-Description.

Notable findings:
- **Re-registration skips MAR**: A.4.2 confirms that re-registration does NOT require a new MAR — the S-CSCF already has auth context. Only when the S-CSCF has no auth state (first registration or after timeout) is MAR required.
- **IncludeRegisterRequest/Response in ApplicationServer**: These optional classes in the iFC ApplicationServer element tell the S-CSCF to forward the SIP REGISTER request (or response) to the AS — enabling AS-based registration handling (e.g., AS needs to log registrations or install CDRs at registration time). This is the stage-3 representation of what TS 23.218 §6 describes.
- **CoreNetworkServicesAuthorization absent = no restriction**: If absent, no ICSI filtering and no media profile authorization applies. S-CSCF treats all IMS Communication Service Identifiers as permitted.
- **AliasIdentityGroupId distinguishes two alias models**: With AliasInd feature → alias groups tracked by ID, different groups in separate service profiles. Without AliasInd → all PUIs in same implicit set are aliases and share service profile.

Next chunk: ts29228-3 — Annexes C–K: CNF/DNF (C), High-level User Profile format (D), XML schema (E, normative), SPT parameters (F, normative), Emergency registrations (G, normative), Diameter overload/priority/load control (H/J/K, normative) (PDF pages 68–85)
Pages touched: [wiki/procedures/Cx-signalling-flows.md (updated — 7 Annex A.4 flow diagrams added), wiki/protocols/Cx-Diameter.md (updated — Annex A.2/A.3 tables + Annex B UML model added), wiki/sources/ts29228.md (updated), wiki/index.md (updated), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.228 Annexes C–K — CNF/DNF, XML schema, SPT matching, emergency registration, Diameter overload/priority/load | chunk ts29228-3

Ingested TS 29.228 v18.0.0 Annexes C–K (PDF pages 68–85). Updated three existing wiki pages.

**Annex C (informative) — CNF/DNF for TriggerPoint**: `ConditionTypeCNF=TRUE` → Conjunctive Normal Form: same Group value = OR'd SPTs; groups AND'd together. `ConditionTypeCNF=FALSE` → Disjunctive Normal Form: same Group value = AND'd SPTs; groups OR'd. Both forms documented with complete XML examples showing the same logical trigger expressed in each form. Added to IM-call-model.md §18.

**Annex D (informative) — High-level User Profile format**: Informative diagram; no separate wiki page created (structural information already covered by Annex B UML model, Annex E schema).

**Annex E (normative) — XML schema**: Key simple type enumerations documented: tIdentityType (0–4: DISTINCT_PUBLIC_USER_IDENTITY, DISTINCT_PSI, WILDCARDED_PSI, NON_DISTINCT_IMPU, WILDCARDED_IMPU), tDirectionOfRequest (0–4: ORIGINATING_REGISTERED through ORIGINATING_CDIV), tDefaultHandling (0–1), tRegistrationType (0–2), tServicePriorityLevel (0=highest, 4=lowest). PublicIdentity backward-compatibility Extension chain documented: tPublicIdentityExtension → Extension2 → Extension3 → Extension4 → Extension5 (MaxNumOfAllowedSimultRegistrations). Added to IM-call-model.md §20.

**Annex F (normative) — SPT matching parameters**: ERE (POSIX Extended Regular Expression) matching for RequestURI, SIPHeader, SessionDescription. SIPHeader evaluated against each header instance individually — fires if any one matches. SWS/LWS normalised to single SP before comparison. RegistrationType: only meaningful for SIP-Method=REGISTER SPTs; multiple values = OR; unsupported S-CSCF matches all REGISTERs. SessionCase values 0–4 documented (ORIGINATING_REGISTERED through ORIGINATING_CDIV). Added to IM-call-model.md §19.

**Annex G (normative) — Emergency registrations**: Four rules: (G.1) emergency-registered IMPU stored as REGISTERED in HSS (not NOT_REGISTERED); (G.2) S-CSCF ignores PERMANENT_TERMINATION and REMOVE_S-CSCF RTR for emergency-registered pairs, returns DIAMETER_LIMITED_SUCCESS with protected pairs listed; (G.3) IMS Restoration procedures do not apply to emergency sessions; (G.4) normal SAR cleanup after emergency session ends. Mermaid sequence diagram. Added to Cx-signalling-flows.md.

**Annex H (normative) — Diameter overload control (RFC 7683)**: HSS = reporting node (sends OC-OLR in answers); I/S-CSCF = reacting nodes (throttle requests). Loss and Rate reduction algorithms. MPS and emergency requests throttled last. Added to Cx-Diameter.md.

**Annex I (informative) — Overload node behaviour**: Guidance on I/S-CSCF throttling prioritisation; informative only — no separate wiki page.

**Annex J (normative) — Diameter message priority (RFC 7944)**: DRMP AVP (code 301) carries priority 0–15. DRMP takes precedence over Session-Priority when both present (§J.2.4 key rule). Added to Cx-Diameter.md.

**Annex K (normative) — Diameter load control (RFC 8583)**: HSS passively advertises Load AVP in any Cx answer. Load-Type (WEIGHT/HOST) and Load-Value (0–65535). Distinction from overload control: load = proactive routing signal; overload = reactive throttle demand. Added to Cx-Diameter.md.

Notable findings:
- **CNF vs DNF Group semantics are inverted**: In CNF the Group attribute forms OR-clauses (groups AND'd); in DNF the Group attribute forms AND-clauses (groups OR'd). Both models express equivalent boolean expressions but with structurally opposite semantics for the Group field — easy to confuse.
- **Annex G keeps HSS state REGISTERED during emergency**: This is a deliberate deviation from the normal lifecycle — emergency registration must keep S-CSCF assigned. Applying IMS Restoration on top of an emergency session would break the emergency call.
- **DRMP precedence over Session-Priority**: The Diameter-layer priority (DRMP) takes precedence over the application-layer IMS priority (Session-Priority) for routing/scheduling decisions. Both coexist and serve different layers.

Next chunk: — (TS 29.228 is now COMPLETE; source fully ingested)
Pages touched: [wiki/concepts/IM-call-model.md (updated — §18 CNF/DNF, §19 SPT matching, §20 XML enum types), wiki/protocols/Cx-Diameter.md (updated — Annexes H/J/K overload/priority/load), wiki/procedures/Cx-signalling-flows.md (updated — Annex G emergency registration), wiki/sources/ts29228.md (updated — COMPLETE), wiki/index.md (updated), wiki/log.md (this entry)]

## [2026-04-17] ingest | TS 29.165 §1–§7 — II-NNI reference model, IBCF/TrGW, control-plane SIP spec, user plane [chunk ts29165-1]

New source: 3GPP TS 29.165 v16.6.0 "Inter-IMS Network to Network Interface (NNI)". This spec defines the II-NNI (Inter-IMS NNI) consisting of two reference points: Ici (SIP control, IBCF↔IBCF) and Izi (RTP media, TrGW↔TrGW).

**§4 Overview:** II-NNI interconnects two IM CN subsystems. Control plane over Ici, user plane over Izi. Three traversal scenarios (non-roaming home-to-home, loopback LBO, roaming) identified by `iotl` SIP URI parameter (RFC 7549). Out of scope: Mr/Mr'/Rc/ISC for internal MRFC/AS/enterprise interworking.

**§5 Reference model:** Two new entities defined: IBCF (Interconnection Border Control Function) — SIP/SDP border node with topology hiding, IMS-ALG (acts as B2BUA), CDR generation, IOI insertion, privacy protection; operates via Mx internally and Ici externally. TrGW (Transition Gateway) — media-plane relay with NA(P)T-PT for IPv4/IPv6 translation, controlled by IBCF, uses Izi externally. Traversal scenario identified via `iotl` parameter in Table 5.3.2.1.

**§6 Control-plane SIP:** 23 supported SIP methods (Table 6.1) — ACK/BYE/CANCEL/INVITE/OPTIONS/PRACK/UPDATE mandatory; REGISTER/NOTIFY/SUBSCRIBE mandatory only on roaming II-NNI. 25 SIP header fields with trust/no-trust behaviour (Table 6.2): P-Asserted-Identity stripped when no trust; Attestation-Info/Origination-Id only on non-roaming; P-Served-User/Auth headers only on roaming. 127-item capability profile (Table 6.1.3.1) with 8 conditional codes. SDP not encrypted at II-NNI. Control transport: TCP default, SCTP optional (recommended). SIP timers: T1=500ms default, Timer C >3min.

**§7 User plane:** RTP/UDP mandatory; TCP/SCTP optional. IBCF can insert/remove codecs in SDP; TrGW transcodes. Transcoding should be minimized per operator agreements.

Notable findings:
- **REGISTER mandatory only on roaming II-NNI**: On non-roaming II-NNI, both UEs are in their home networks and REGISTER does not traverse the inter-home II-NNI. On roaming, P-CSCF is in visited network so REGISTER must traverse the roaming II-NNI to reach home I-CSCF.
- **SDP must not be encrypted at II-NNI**: This is a hard constraint — IBCF must be able to read and modify SDP (c= and m= lines) to insert TrGW media addresses. This contrasts with Gm (UE-P-CSCF) where end-to-end SDP encryption would be architecturally possible.
- **IBCF B2BUA mode for IMS-ALG**: When performing IPv4/IPv6 translation, the IBCF cannot operate as a SIP proxy — it must terminate and originate new dialogues to rewrite addresses. This is why IBCF is architecturally heavier than BGCF (which is a pure proxy).
- **IOI insertion at II-NNI**: IBCF inserts transit Inter-Operator Identifier (IOI) in P-Charging-Vector. This is how inter-operator charging correlation is achieved across the II-NNI — each IBCF on the path adds its network's IOI, creating an end-to-end IOI chain.
- **`iotl` parameter absence = non-roaming**: If no `iotl` parameter is in Request-URI or Route headers, the non-roaming II-NNI scenario is assumed. This is the default for simple home-to-home calling scenarios.

Next chunk: ts29165-2 — §8–§12: Numbering/addressing, security, charging + all 26 supplementary services over II-NNI (PDF pp 44–57)
Pages touched: [wiki/entities/IBCF.md (new), wiki/entities/TrGW.md (new), wiki/interfaces/II-NNI.md (new), wiki/interfaces/IMS-reference-points.md (updated — Ici/Izi/Mx added), wiki/sources/ts29165.md (new), wiki/index.md (updated), wiki/log.md (this entry)]

## [2026-04-18] ingest | TS 24.229 §4 General — SIP/SDP conformance profile for IMS | chunk ts24229-1

New source: 3GPP TS 24.229 v16.6.0 "IP multimedia call control protocol based on SIP and SDP; Stage 3". This is the stage-3 SIP/SDP specification for IMS — the normative protocol complement to TS 23.228 (stage-2 architecture). It defines exact SIP message procedures per-node and all SIP/SDP extensions for IMS. Ingested §4 General (PDF pp 70–101, spec pp 67–98).

**§4.1 Node SIP Roles:** Complete per-entity role table captured. UE = UA. P-CSCF = proxy (B2BUA for security). I-CSCF = proxy. S-CSCF = proxy + UA (for S-CSCF-initiated dialog release, MESSAGE, Transit Function). AS roles vary: terminating/originating UA, SIP proxy, B2BUA, 3PCC per TS 23.218 §9.1.1.x. IBCF = proxy; IMS-ALG mode → B2BUA. E-CSCF = proxy (UA for certain emergency cases: location by value per RFC 6442, privacy suppression, notifier role). Key rule: proxy-role entities shall not modify message bodies.

**§4.2 URI/Address Assignments:** SIP URIs for I-CSCFs (DNS SRV load-balanced); dual-stack IP. IMPI from ISIM (or derived from USIM; or generated if no ISIM/USIM). One or more IMPUs (SIP URI + tel URI); at least one SIP URI required. Tel URI rule: each tel URI has a companion SIP URI with `user=phone` parameter. Shared PUI across multiple UEs possible. Instance ID required for GRUU/multiple registration. ICSI values (urn:urn-7:3gpp-service.*) signalled in REGISTER per §7.2A.8.

**§4.2B Security Mechanisms (Table 4-1 + 4-2):** Full 9-row security mechanism table captured: IMS AKA + IPsec ESP (mandatory for all UEs with UICC, mandatory for P/I/S-CSCF); IMS AKA + HTTP Digest AKAv2 without IPsec (mandatory for WIC/UICC, mandatory for eP-CSCF); SIP Digest (without TLS, with TLS) — optional; NASS-IMS bundled / GPRS-IMS-Bundled / Trusted node — optional network-only. SIP over TLS + client cert — mandatory for external attached network UEs. Media security table (Table 4-2): end-to-access-edge (SDES/SRTP, MSRP TLS, BFCP TLS, UDPTL DTLS); P-CSCF (IMS-ALG) required for all end-to-access-edge mechanisms.

**§4.4 Trust Domain (23 headers):** The trust domain = same operator network entities. Complete per-header trust boundary action table captured for all 23 headers/parameters: P-Asserted-Identity (remove at boundary per RFC 3325), P-Access-Network-Info (remove per RFC 7315), History-Info (remove per RFC 7044), P-Asserted-Service (remove per RFC 6050), Resource-Priority (remove when forwarding outside; strip from untrusted source), Reason (remove per RFC 6432), P-Profile-Key, P-Served-User, P-Private-Network-Indication, P-Early-Media, CPC/OLI URI params, Feature-Caps, Priority (psap-callback), iotl, Restoration-Info, Relayed-Charge, Service-Interact-Info, Cellular-Network-Info, Response-Source, Attestation-Info, Origination-Id, Additional-Identity.

**§4.5 Charging Correlation (5 elements in P-Charging-Vector):** ICID (icid-value), ICID origin (icid-generated-at), Related ICID (related-icid), access-network-charging-info, orig-ioi, term-ioi, transit-ioi, fe-identifier. ICID generation table (Table 4-2A): first IM CN subsystem entity receiving a message inserts the icid-value; for dialogs generated on initial request only. Per-entity ICID responsibilities: P-CSCF for UE-originated; I-CSCF for UE-terminated (if none in initial request); MGCF for PSTN-originated; AS when originating UA; MSC server for ICS/SRVCC. Three IOI types defined (Table 4-2B): Type 1 = visited↔home; Type 2 = originating↔terminating; Type 3 = any AS interaction. IOI insertion rule: sending entity inserts orig-ioi, does NOT insert term-ioi; receiving entity in response inserts term-ioi from received orig-ioi. Transit IOI (transit-ioi) is an indexed list of transit networks; deleted by S-CSCF of home terminating network. CDF/OCF addresses via Cx → S-CSCF → P-Charging-Function-Addresses; not passed to UE.

**§4.11–§4.12 Priority and Overload:** RFC 4412 Resource-Priority; three authorization modes (subscription, database/dial-string, database/REGISTER-based). Overload: RFC 7339 feedback (Via header, loss-based algorithm default) + RFC 7200 load filter event package. Priority ordering: emergency calls exempt (threshold-based), priority calls exempt last before network instability, mid-dialog messages last to drop before initial requests.

**§4.13 II-NNI Traversal + §4.14 Restoration:** iotl URI parameter identifies II-NNI traversal scenario; added at start of leg, removed at end; absence = non-roaming default. P-CSCF restoration: two variants (UDM/HSS-based for terminating; PCF/PCRF-based for originating-INVITE via IBCF). S-CSCF restoration: P-CSCF informs UE via 504 + 3GPP IM CN subsystem XML body (§7.6) → UE performs initial registration.

Notable findings:
- **S-CSCF plays UA role only in 4 specific cases**: S-CSCF-initiated dialog release, providing messaging (MESSAGE), Transit Function, and when providing server functionality for a final response (per RFC 3261). All other cases are proxy. This is the authoritative stage-3 statement of what "proxy" means for S-CSCF.
- **ICID in REGISTER is unique and separate**: The P-CSCF generates a unique ICID per REGISTER in its own unique P-Charging-Vector header instance. This ICID is NOT passed to the UE and is separate from session ICIDs. It allows correlation of registration CDRs.
- **Table 4-1 is definitive for security mechanism support level**: The distinction between "mandatory", "optional", and "not supported" in Table 4-1 makes clear that IMS AKA + IPsec ESP is the only fully mandatory mechanism across all UEs with UICC cards and all P/I/S-CSCF nodes. SIP Digest is always optional on both UE and network sides.
- **iotl removal is the responsibility of the ending entity**: The entity at the END of the II-NNI traversal leg (e.g. S-CSCF, TRF, P-CSCF) removes the iotl parameter — not the IBCF that terminates the Ici interface. This means the iotl may traverse multiple intermediate nodes within the IMS network of the destination operator.

Next chunk: ts24229-2 — §5.1.1 UE Initial Registration (IMS AKA SIP messages §5.1.1.2.2) + §5.2.2 P-CSCF Registration procedures + §5.4.1 S-CSCF registration SIP message procedures (PDF pp 106–261, selective)
Pages touched: [wiki/protocols/SIP-IMS-profile.md (new), wiki/sources/ts24229.md (new), wiki/index.md (updated, 95→97 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 24.229 §5.1.1 + §5.2.2 + §5.4.1 — IMS SIP Registration stage-3 procedures | chunk ts24229-2

Ingested the normative stage-3 registration procedures for UE (§5.1.1), P-CSCF (§5.2.2), and S-CSCF (§5.4.1) from 3GPP TS 24.229 v16.6.0. Covers IMS AKA 2-phase REGISTER, SIP Digest variants, user-initiated and network-initiated deregistration. PDF pages ~106–253 (selective, focused on §5.1.1.2, §5.2.2.1–§5.2.3, §5.4.1.2–§5.4.1.5).

**UE (§5.1.1):** Complete header set for Phase-1 unprotected REGISTER (Contact with +sip.instance, Via with rport, Expires=600000, Supported=path/gruu/outbound, P-Access-Network-Info, Security-Client, Authorization with nonce=""/response=""). On 401: AUTN verification, CK/IK derivation, temporary SA setup, protected REGISTER (same Call-ID, response=RES, Security-Verify=Security-Server content). On 200 OK: SA promotion (lifetime=max(prev, expiry+30s)), store default PUI (first P-Associated-URI URI), store Service-Route (preloaded Route for non-REGISTER), store pub-gruu/temp-gruu, store ICSI Feature-Caps.

**P-CSCF (§5.2.2):** Phase-1: insert Path (IMS flow token + ob), P-Charging-Vector (icid + orig-ioi type1), P-Visited-Network-ID; set rport/received; store temp SA (reg-await-auth lifetime). On 401: delete temp SA, strip ik/ck from WWW-Authenticate, insert Security-Server, set new temp SA. On 200 OK: temp SA → newly established SA (taken into use); delete previous SA immediately (initial) or all except in-use SA (re-auth). Store Service-Route (bound to contact+SA), P-Associated-URI, CFA, term-ioi. After first REGISTER per IMPI: send SUBSCRIBE to S-CSCF for reg event package.

**S-CSCF (§5.4.1.2.1 + §5.4.1.2.2 + §5.4.1.2.2F):** Phase-1: identify user from To/Authorization, MAR→HSS for auth vectors, generate 401 (WWW-Authenticate: realm=S-CSCF-name, nonce=RAND+AUTN, algorithm=AKAv1-MD5, ik=IK, ck=CK). Start reg-await-auth timer. Phase-2: match Call-ID to 401, stop timer, verify RES vs XRES (RFC 3310/4169), SAR→HSS to download service profiles. Store PUIs (barred/non-barred), all service profiles+iFCs, restoration info, PCRF-based P-CSCF data. Build preloaded Route from Path header list; bind expiry to contact. Store icid-value + orig-ioi type1. 200 OK per §5.4.1.2.2F: P-Associated-URI (first=default PUI; no barred); Service-Route with unique S-CSCF URI per registration (IBCF topmost if topology hiding; iotl=visitedA-homeA if roaming+iotl supported); Contact with all saved params (not pub-gruu/temp-gruu); GRUUs per contact; Feature-Caps ICSIs; Require: outbound if applicable. Send third-party REGISTER to matching ASes.

**Deregistration (§5.4.1.4):** Expires=0 → verify contact/flow exists (481 if not), release INVITE dialogs, remove binding, third-party REGISTER to ASes, 200 OK with Contact list (deregistered entries expiry=0). **Network-initiated (§5.4.1.5):** NOTIFY to UE/P-CSCF, release multimedia sessions per §5.4.5.1.2.

Notable findings:
- **P-CSCF strips ik/ck from 401 before forwarding to UE**: IK and CK in WWW-Authenticate are solely for P-CSCF SA setup — the UE must derive its own CK/IK directly from RAND via TS 33.203. This separation means a compromised transmission between S-CSCF and P-CSCF does not expose keying material to the air interface.
- **Service-Route URI must be unique per registration/flow**: S-CSCF uses a different SIP URI for each registration (and per registration flow if multiple registrations in use). This allows the S-CSCF to identify which contact address or flow generated an incoming request and apply correct routing.
- **401 Call-ID must match protected REGISTER Call-ID**: S-CSCF only proceeds with authentication if the Call-ID in the protected REGISTER matches the Call-ID in the 401. This is the anti-replay mechanism at the SIP layer (complementing the IPsec SA at the transport layer).
- **S-CSCF stores the unregistered iFC part**: When downloading iFCs, the unregistered part of the Filter Criteria is retained even after successful registration — needed immediately if the user deregisters, without requiring a new Cx download.

Next chunk: ts24229-3 — §7 SIP extensions: P-header definitions (§7.2.11–§7.2.20 + §7.2A series), timer tables (§7.7, §7.8), media feature tags (§7.9, §7.9A)
Pages touched: [wiki/procedures/IMS-SIP-registration.md (new), wiki/sources/ts24229.md (updated — ts24229-2 Done), wiki/index.md (updated, 97→98 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 24.229 §7.2.13–§7.2.20 + §7.2A + §7.7–§7.9A — SIP extensions, timers, feature tags | chunk ts24229-3

Ingested all SIP extension definitions from TS 24.229 §7: new P-header fields (§7.2.13–§7.2.20), extensions to existing headers (§7.2A.1–§7.2A.18), SIP and IM CN subsystem timer tables (§7.7–§7.8), media feature tags (§7.9), feature-capability indicators (§7.9A), reg-event package extensions (§7.10), URNs (§7.11), and info packages (§7.12). PDF pages ~403–481 (selective).

**§7.2.13–§7.2.20 — New P-headers:** Resource-Share (media stream sharing between sessions via P-CSCF; values: supported/media-sharing/no-media-sharing; parameters: origin, rules with new-sharing-key/directionality, timestamp counter); Service-Interact-Info (AS-to-AS service conflict notification, trust domain only); Cellular-Network-Info (cellular cell ID carried when UE on non-cellular IP-CAN, trust domain only, UE does not receive); Priority-Share (P-CSCF instructs access GW on bearer sharing: allowed/not-allowed); Response-Source (URN identifying entity generating error response; urn:3gpp:fe namespace; fe-id: ue/p-cscf/i-cscf/s-cscf/e-cscf/mgcf/bgcf/ibcf/trf/atcf/mrfc/lrf/msc-server/as; fe-param: role/side); Attestation-Info (STIR attestation level A/B/C per RFC 8588; inserted by S-CSCF/AS/entry IBCF); Origination-Id (UUID of attesting node, companion to Attestation-Info); Additional-Identity (alternative originating/target identity for multi-identity services; removed at trust boundary).

**§7.2A — Header extensions:** WWW-Authenticate extension (ik/ck hex params for P-CSCF — P-CSCF stores and removes before forwarding to UE); Authorization integrity-protected parameter (8 values: yes/no/tls-pending/tls-yes/ip-assoc-pending/ip-assoc-yes/auth-done/tls-connected — P-CSCF inserts, S-CSCF reads); tokenized-by parameter (IBCF THIG encryption marker); PANI extensions (daylight-saving-time, UE-local-IP-address, ePDG-IP-address, network-provided, extended access-class and access-type for NR/ProSe/DVB); P-Charging-Vector access-network-charging-info per IP-CAN (GPRS: ggsn/gcid/flow-id; EPS: pdngw/ecid/eps-sig; 5GS: smf/5gscid; xDSL: bras/dslcid; DOCSIS: bcid; cdma2000: icn-bcp; Ethernet/Fiber: ip-edge; IOI type1/3 string prefixes; loopback param; fe-identifier); orig URI param (appended to S-CSCF/I-CSCF/IBCF address by AS to force originating service); mediasec parameter (labels Security-Client/Server/Verify entries as media plane: sdes-srtp/msrp-tls/bfcp-tls/udptl-dtls); ICSI/IARI coding (urn:urn-7:3gpp-service/application.ims.icsi/iari.*; subclass "is a" relationship; percent-encoded in feature tag); phone-context per IP-CAN; CPC/OLI params (ordinary/test/operator/payphone/mobile-hplmn/mobile-vplmn/emergency); sos URI param (emergency registration marker); P-Associated-URI proxy modification extension; premium-rate URI param (information/entertainment).

**§7.7 SIP Timers (Table 7.7.1):** Three-column timer values for IM CN / UE / P-CSCF-toward-UE. Key values: T1=500ms/2s/2s; T2=4s/16s/16s; T4=5s/17s/17s; Timer F = 64×T1 = 128s at UE (IM CN subsystem). T1 extendable for MRFC+AS under same operator.

**§7.8 IM CN Timers (Table 7.8.1):** reg-await-auth = 4 min at S-CSCF (worst case 2×Timer-F = 256s); request-await-auth = 4 min at S-CSCF (non-REGISTER auth); emerg-reg = 8–20s UE; emerg-request = 5–15s UE; NoVoPS-dereg = 0–65535s UE; emerg-non3gpp = 5–20s UE. NOTE: UE and P-CSCF use reg-await-auth value for temporary SA SIP-level lifetime.

**§7.9 Media Feature Tags:** g.3gpp.icsi-ref (OID 1.3.6.1.8.2.4, ICSI URN, multiple values); g.3gpp.iari-ref (OID 1.3.6.1.8.2.5, IARI URN); g.3gpp.registration-token (OID 1.3.6.1.8.2.27, unique per registration, S-CSCF→AS correlation); g.3gpp.ps-data-off (OID 1.3.6.1.8.2.35, active/inactive); g.3gpp.rlos (boolean, RLOS support).

**§7.9A Feature-Capability Indicators:** +g.3gpp.icsi-ref (ICSI in Feature-Caps for standalone/dialog); +g.3gpp.trf (TRF SIP URI for local breakout); +g.3gpp.loopback (home→visited loopback); +g.3gpp.home-visited (home supports loopback to visited); +g.3gpp.mrb (visited MRB URI); +g.3gpp.registration-token (AS registration correlation); +g.3gpp.thig-path (IBCF THIG SIP URI); +g.3gpp.priority-share; +g.3gpp.verstat (STIR support); +g.3gpp.anbr (ANBR support). IMS-ALG media capability indicators: sip.audio/application/data/video/control/text/message/ice/app-subtype/fax — absent = all assumed supported.

**§7.10–§7.12:** Reg-event wildcarded IMPU transport (XML <wildcardedIdentity>); reg-event policy transport (RPH, privSender, pni, privSenderPNI); DTMF info package (infoDtmf, MIME application/session-info, SubsequentDigit body); current-location-discovery info package (g.3gpp.current-location-discovery, MIME application/vnd.3gpp.current-location-discovery+xml).

Notable findings:
- **reg-await-auth = 4 minutes is tied directly to 2×Timer-F at S-CSCF**: Timer F = 64×T1; at IM CN level T1 = 2s (UE value), so Timer F = 128s. Worst-case authentication takes 2 round trips (challenge + response), hence 2×128s = 256s < 4min. The 4-minute value provides a safety margin for network latency. This means operators that increase T1 to reduce retransmissions should also consider increasing reg-await-auth.
- **ik/ck in WWW-Authenticate are directives to P-CSCF, not for the UE**: The IK/CK are transferred from S-CSCF to P-CSCF via the 401 response. P-CSCF stores them for SA setup and strips them. The UE derives its own IK/CK from RAND independently. This separation is deliberate: if the Mw interface (S-CSCF↔P-CSCF) is compromised, only the P-CSCF's SA keys are exposed — not the air interface.
- **Response-Source enables P-CSCF restoration decisions**: The Response-Source `fe=<urn:3gpp:fe:s-cscf.term>` in a 504 from the S-CSCF allows the P-CSCF to distinguish S-CSCF failure from other network failures and initiate S-CSCF restoration (see §4.14 + SIP-IMS-profile.md).
- **+g.3gpp.thig-path gives P-CSCF the THIG IBCF address**: In roaming scenarios where the visited IBCF applied topology hiding on the Path header in REGISTER, the IBCF includes `+g.3gpp.thig-path=<ibcf-sip-uri>` in the 200 OK Feature-Caps. P-CSCF stores this URI and uses it as the destination for subsequent UE-originated requests (instead of the P-CSCF's own preloaded Route), enabling IBCF to re-encrypt the Route URI for the home network.

TS 24.229 is now **COMPLETE** — all 3 chunks ingested.

Next chunk: No next chunk for TS 24.229. Remaining undigested source: ts29165-2 (TS 29.165 §8–§12 — II-NNI topology, interworking, IBCF detailed procedures).
Pages touched: [wiki/protocols/SIP-IMS-extensions.md (new), wiki/sources/ts24229.md (updated — ts24229-3 Done, COMPLETE), wiki/index.md (updated, 98→99 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 23.237 §3–§5 — IMS Service Continuity definitions, architecture, entities | chunk ts23237-1

New source: 3GPP TS 23.237 v18.0.0 "IMS Service Continuity; Stage 2" (~191 pages). Ingested §3 (definitions/abbreviations), §4 (high-level principles and architectural requirements), and §5 (architecture model, functional entities, signalling/bearer paths).

**§3 Key definitions:** Access Leg (UE↔SCC AS control leg), Remote Leg (SCC AS↔remote party), Access Transfer (IMS-level media path transfer between PS and CS, or between IP-CANs), Collaborative Session (≥2 Access Legs on ≥2 UEs presented as one Remote Leg), Controller UE (Collaborative Session owner), Controllee UE (subordinate, unaware of role), STN/STN-SR (Session Transfer Numbers routing SRVCC request to ATCF/SCC AS), STI (dynamic session transfer identifier from UE), ATU-STI (ATCF uses to notify SCC AS post-AT), C-MSISDN (correlation MSISDN bound to IMPI), IMRN (IP Multimedia Routing Number), E-STN-DR/E-STN-SR (emergency DRVCC/SRVCC numbers), Session State Information (sent by SCC AS to MSC Server for PS-CS continuity without ICS).

**§4 Architectural requirements:** Session transfer independent of network-layer mobility; no impact on radio/transport/PS core; UE must be IMS-registered before invoking Session Transfer; SCC AS inserts 3rd-party registration (iFC); charging correlation required; SCC AS maintains Session State Information for all active+inactive sessions; MSC Server mid-call feature available when UE+network+SCC AS all support it.

**§5 Functional entities:**
- **SCC AS** (home): 3pcc anchor (Access Leg + Remote Leg); retrieves C-MSISDN from HSS on 3rd-party registration; clears/updates STN-SR in HSS; when ATCF deployed: provides C-MSISDN + ATU-STI to ATCF; provides Session State Information to MSC Server; executes IUT procedures; T-ADS; generates operator policy via OMA DM.
- **IMS SC UE**: stores/applies operator policy; initiates Access Transfer; stores STI-rSR (CS-to-PS); stores E-STN-DR (DRVCC); Controller UE / Controllee UE roles for IUT.
- **EATF** (serving network, emergency): B2BUA using 3pcc; anchors IMS emergency sessions; PS-to-CS AT for emergency; generates/sends E-STN-DR to UE for DRVCC.
- **ATCF** (serving network, optional): included for (v)SRVCC/5G-SRVCC; co-located with P-CSCF (Iq to ATGW) or IBCF (Ix to ATGW); allocates STN-SR; inserts into SIP path during registration; instructs ATGW to anchor media; performs AT locally; sends ATU-STI to SCC AS; provides STI-rSR to UE for CS-to-PS SRVCC; interfaces: Mw/Mx (I/S-CSCF), Iq/Ix (ATGW), Mw/I2 (MSC Server).
- **ATGW** (serving network): controlled by ATCF via Iq or Ix; media anchor for full call duration; supports post-SRVCC transcoding; may be IMS-AGW (P-CSCF co-location) or TrGW (IBCF co-location).
- **HSS**: allows SCC AS to update STN-SR; STN-SR addresses ATCF (if deployed) else SCC AS.
- **MSC Server enhanced for SRVCC**: no announcements during AT; I2 interface to ATCF.

**§5.4 Signalling/bearer paths:** (1) PS media no ATCF: SCC AS 3pcc home; direct IP-CAN media. (2) PS media with ATCF: ATCF in serving signalling path; ATGW anchors media in serving network; home leg via SCC AS. (3) CS media: TS 23.292 applies. (4) CS media with ATCF: ATCF + ATGW in serving path; MSC Server on CS access leg.

**§5.5 IUT:** Multi-media session split across ≥2 UEs; UEs may belong to different IMS subscriptions under same operator.

Notable findings:
- **ATCF is in serving network; SCC AS in home network** — this separation is the key architectural decision enabling SRVCC without home-network round-trips on the media path. The ATGW anchors media locally so Access Transfer only touches the serving leg.
- **STN-SR in HSS routes to ATCF (not SCC AS) when ATCF deployed** — this is the critical routing hook. Without ATCF, STN-SR points to SCC AS directly. ATCF changes the routing so the SRVCC request arrives at ATCF first.
- **ATU-STI is the registration-time token** — SCC AS generates ATU-STI and provides it to ATCF at registration. ATCF includes it in the Access Transfer Update notification so SCC AS can correlate the post-AT Access Leg to the pre-AT Remote Leg.
- **Controllee UE is deliberately unaware** — any IMS UE can be a Controllee UE; it only needs to respond to IMS session setup procedures. This is why "Collaborative Session procedures for a Controllee UE without IUT capabilities shall have no impact to the UE."
- **ATGW co-location determines the interface** — Iq (TS 23.334) when co-located with P-CSCF; Ix (TS 29.162) when co-located with IBCF. The ATGW is not a new node — it reuses IMS-AGW or TrGW depending on deployment.

Next chunk: ts23237-2 — TS 23.237 §6.0–§6.2 — Registration, Origination, Termination for IMS Service Continuity
Pages touched: [wiki/concepts/IMS-service-continuity.md (new), wiki/entities/SCC-AS.md (new), wiki/entities/ATCF.md (new), wiki/entities/ATGW.md (new), wiki/interfaces/IMS-reference-points.md (updated — Iq/Ix/I2 added), wiki/sources/ts23237.md (new), wiki/index.md (updated, 99→104 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 23.237 §6.0–§6.2 — IMS SC Registration, Origination, Termination | chunk ts23237-2

Ingested §6.0 (CS/IMS Intermediate Nodes abstraction), §6.1 (Registration), and §6.2 (Origination and Termination).

**§6.1.1 General:** On every IMS registration, S-CSCF performs 3rd-party registration to SCC AS per TS 23.218. SCC AS retrieves C-MSISDN from HSS, binds to ongoing session identifiers (GRUUs, contact address). If no ATCF STN-SR received: SCC AS clears existing STN-SR from HSS and provides home-configured STN-SR (pointing to SCC AS).

**§6.1.2 ATCF registration flow (11 steps):** UE REGISTER → AMF → ATCF (step 2: allocate STN-SR + ATCF management URI) → S-CSCF → 3rd-party REGISTER to SCC AS → SCC AS Sh-Pull (UE SRVCC capability + existing STN-SR) → if STN-SR differs: Sh-Update → HSS Insert Subscription Data to MME/SGSN → 200 OK. SCC AS then informs ATCF of UE SRVCC capability; ATCF stores this for ATGW anchoring decisions. If UE registers from multiple accesses, SCC AS only accepts and uses one STN-SR.

**§6.1.3 CS-to-PS Single Radio registration:** UE must include pre-defined port+codec for voice downlink during CS-to-PS AT, and include CS-to-PS SRVCC capability. ATCF provides dynamic STI-rSR to UE at registration. MSC Server enhanced for ICS can subscribe to SCC AS for UE IMS registration status updates and ATCF management URI changes (3-step notification flow).

**§6.2.1 Origination:** SCC AS is the FIRST AS remaining in call path after origination. Six variants: (1) CS media → TS 23.292 §7.3.2, SCC AS anchors + assigns STI; (2) PS-only → standard TS 23.228 MO, SCC AS anchors + assigns STI; (3) PS-only with ATCF in path → INVITE traverses ATCF → I/S-CSCF → SCC AS, ATCF may anchor ATGW (decision at INVITE or at completion), ATCF MUST NOT modify dynamic STI; (4) PS-only fallback to CS in pre-alerting → PS bearer failure → Cancel → UE initiates CS call; (5) CS-to-PS Single Radio via ATCF → MSC Server INVITE includes C-MSISDN, uses ATCF management URI.

**§6.2.2 Termination:** SCC AS is the LAST AS remaining in call path after termination. Six variants: (1) CS media → TS 23.292 §7.4.2, SCC AS T-ADS may split speech/non-speech media, C-MSISDN for correlation; (2) PS-only → TS 23.228 MT, SCC AS anchors + STI; (3) PS-only + ATCF → T-ADS routes to ATCF; ATCF may anchor ATGW; if UE rejects speech, ATCF releases ATGW and removes itself; (4) PS-only fallback to CS in pre-alerting → P-CSCF signals failure → SCC AS re-attempts CS delivery; (5) Gm speech rejection → SCC AS splits non-speech from speech; re-attempts on CS domain; (6) CS-to-PS Single Radio via ATCF → MSC Server routes via ATCF management URI.

Notable findings:
- **SCC AS first/last AS rule is structurally load-bearing for AT**: All other ASes in the iFC chain (TAS, voicemail AS, etc.) must complete before SCC AS on origination, and after SCC AS on termination. This is the only way the SCC AS can wrap the 3pcc legs around the entire session. If this order is violated, AT cannot be executed because the Access Leg won't be anchored at SCC AS.
- **ATCF decision to anchor ATGW may be deferred to session completion**: The ATCF need not decide to anchor at INVITE time — it can anchor after the SDP offer/answer exchange at session completion (step 6 in §6.2.1.4/6.2.2.5). This means the ATGW anchor decision can be based on the final negotiated codec/media parameters, reducing premature resource allocation.
- **ATCF management URI is the MSC Server's routing handle**: All CS-initiated (INVITE from MSC Server) and CS-terminating calls in the ATCF model use this URI (provisioned by SCC AS to MSC Server via §6.1.3.2 subscription). Without it, the MSC Server cannot include ATCF in the signalling path for CS-originated/terminated sessions.
- **C-MSISDN travels in CS-path INVITEs for correlation**: In CS-to-PS origination and CS-to-PS termination, the MSC Server includes C-MSISDN in the INVITE to the ATCF. ATCF uses this to correlate the CS Access Leg with the IMS session at the SCC AS. This is the same C-MSISDN that SCC AS retrieved from HSS at registration.

Next chunk: ts23237-3 — TS 23.237 §6.3.1–§6.3.2 — PS–CS and PS–PS Access Transfer procedures
Pages touched: [wiki/procedures/IMS-SC-registration.md (new), wiki/procedures/IMS-SC-origination-termination.md (new), wiki/sources/ts23237.md (updated — ts23237-2 Done), wiki/index.md (updated, 104→106 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 23.237 §6.3.1–§6.3.2 — PS–CS and PS–PS Access Transfer | chunk ts23237-3

Ingested §6.3.1 (core AT sub-procedures: Remote Leg Update, Source Access Leg Release, AT Info for ATCF) and §6.3.2 (all AT scenario variants: PS→CS Dual Radio, CS→PS Dual Radio, SRVCC Single Radio, CS→PS Single Radio with ATCF, PS-PS full/partial/early-dialog, PS-PS+PS-CS conjunction, active/held session handling).

**§6.3.1 Core sub-procedures:** (1) Remote Leg Update — SCC AS sends UPDATE or Re-INVITE to remote party via S-CSCF after Target Access Leg is established; two variants: IMS UE remote party and MGCF/PSTN remote party. (2) Source Access Leg Release — UE + SCC AS release old access leg per TS 23.228; conditioned on Gm retention for SRVCC. (3) AT Info for ATCF — SCC AS provides ATU-STI + C-MSISDN to ATCF after registration; these are the prerequisites for ATCF-autonomous AT execution.

**§6.3.2.1 PS→CS Dual Radio (§6.3.2.1.1):** UE dials STN-SR → CS SETUP → INVITE(STN-SR, IMRN, SDP-MGW) via MSC Server → SCC AS → Remote Leg Update → Source Access Leg Release.

**§6.3.2.1.1a PS→CS Dual Radio + SSI:** Extends the base flow — SCC AS sends Session State Information (SSI) to MSC Server with STN-SR per additional session; MSC Server may initiate additional CS calls for held sessions.  Session selection algorithm: most-recently-active→transferred; second-most-recently-active→held (SSI); others→released.

**§6.3.2.1.2 CS→PS Dual Radio:** UE on new PS sends INVITE(STI) → I/S-CSCF → SCC AS → Remote Leg Update (PS SDP) → Source Access Leg (CS) Release.

**§6.3.2.1.4 PS→CS Single Radio (SRVCC):** TS 23.216 radio HO triggers INVITE(STN-SR, SDP-MGW) via ATCF (if deployed) → SCC AS → Remote Leg Update (voice only) → optional Re-INVITE to retain non-voice media on PS.

**§6.3.2.1.10 CS→PS Single Radio with ATCF:** MSC Server Session Transfer Notification → ATCF → ATGW switch (media DL path to UE via PS) → UE sends INVITE(STI-rSR) → ATCF correlates → Access Transfer Update (ATU-STI + C-MSISDN) to SCC AS → SCC AS responds with SSI for additional sessions. ATCF executes locally without home round-trip until step 7.

**§6.3.2.2 PS-PS:** Full transfer: UE registers on new IP-CAN + INVITE(STI) → SCC AS Remote Leg Update → Source Access Leg Release. Partial transfer: same but INVITE specifies subset; old IP-CAN retained for remaining media. Early-dialog: UE connects to new IP-CAN mid-alerting; INVITE(STI) over new access; SCC AS updates remote during early dialog.

**§6.3.2.3 PS-PS + PS-CS conjunction:** Non-ICS UE: two separate INVITEs (STI-1 for speech→CS, STI-2 for non-RT→PS); SCC AS does two Remote Leg Updates. Non-ICS CS→PS: single INVITE with STI-1 + STI-2. ICS UE with Gm: INVITE(STI, SDP cs+ps) → SCC AS provides PSI DN → UE CS SETUP → CS-path INVITE(PSI DN) → SCC AS combines legs. Active/held sessions: active transferred first, then held sessions in sequence; CS media not established for held session until resumed.

Notable findings:
- **CS→PS Single Radio with ATCF is the fastest AT path**: ATCF executes media handover (steps 1–6) entirely in the serving network before contacting SCC AS, eliminating the home round-trip from the critical media-switch path. This is the architectural reason for deploying ATCF.
- **SSI is the mechanism for held-session transfer**: without SSI, SRVCC can only transfer the active session. SSI enables the MSC Server to initiate additional CS calls for held sessions, enabling full session-state preservation across SRVCC.
- **ATCF MUST NOT modify the dynamic STI** — verified in AT flows as well as origination/termination. The STI is an end-to-end token between UE and SCC AS.
- **Gm retention condition on Source Access Leg Release**: SCC AS releases old PS leg only if Gm is retained; otherwise radio procedure already cleared it. Confusing but correct — the source leg may not exist by the time SCC AS tries to release it.
- **PS-PS partial transfer leaves two coexisting IP-CAN legs**: this is not a transitional state — both are maintained intentionally for the duration of the split-media session.

Next chunk: ts23237-4 — TS 23.237 §6.3.3–§6.3.5 — Media adding/deleting and supplementary services
Pages touched: [wiki/procedures/PS-CS-access-transfer.md (new), wiki/sources/ts23237.md (updated — ts23237-3 Done), wiki/index.md (updated, 106→107 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 23.237 §6.3.3–§6.5 — Media Adding/Deleting, Operator Policy, Supplementary Services | chunk ts23237-4

Ingested §6.3.3 (12 media adding/deleting sub-cases), §6.3.4 (void), §6.3.5 (ICS UE mid-call fallback), §6.4 (Operator Policy and User Preferences), and §6.5 (Supplementary Services with multiple Access Legs, 22 sub-sections).

**§6.3.3 Media Adding/Deleting — 12 sub-cases across two dimensions:** (Local vs Remote end initiation) × (CS+PS session vs PS-only session). Core pattern: UE-1 adds PS media via INVITE/Re-INVITE → S-CSCF → SCC AS correlates with existing session → Remote Leg Update. Remote-end variant: UE-2 Re-INVITE → SCC AS T-ADS decides PS delivery → C-MSISDN correlation → new INVITE toward UE-1 via PS access. Removing media: local BYE → SCC AS Re-INVITE to remote; remote Re-INVITE → SCC AS BYE to UE-1. Gm extension variants: INVITE includes SDP CS to establish/extend ICS control.

**§6.3.5 ICS mid-call fallback:** If Gm is lost and MSC Server assisted mid-call feature is available, SCC AS performs §6.3.2.1.4a steps 4-6 via MSC Server; thereafter UE uses TS 24.008 (CS NAS) for service control.

**§6.4 Operator Policy:** OMA DM provisioned; covers restricted/preferred access networks, transfer mandate (`shall`/`should`/`may`), and partial AT media retention. UE combines operator policy + user preferences + LOEI (radio quality, QoS, power) when deciding AT. Operator policy must be consistent with T-ADS policy.

**§6.5 Supplementary Services (22 sub-sections):** 14 services not impacted (OIP/OIR/TIP/TIR/CB/MWI/CUG/FA/CCBS/CCNR/CAT/RC/PNM/CRS). 6 impacted: CDIV (UE can use any leg; SCC AS terminates others and diverts), HOLD (propagated to all legs with affected media; SCC AS fans out), CONF/3PTY (UE may use any leg; remote-requested replacement may be delivered on any leg; outside-dialog → UE creates new session), ECT (UE may send from any leg; SCC AS delivers to any leg), CW (UE may use any leg), MCID (Re-INVITE on any leg in temporary subscription mode). AOC: SCC AS delivers on any leg.

Notable findings:
- **T-ADS applies to media modification, not just session setup**: UE-2 adding media triggers T-ADS at SCC AS to select PS access for UE-1 delivery — identical logic to terminating session setup.
- **Session splitting on media addition is subscriber-scoped**: split session is delivered only to UE-1, not to other UEs of the same subscriber — T-ADS ensures per-UE targeting even for mid-session media additions.
- **`shall/should/may` operator policy controls AT autonomy spectrum**: `shall` = automatic AT; `should` = UE applies LOEI before deciding; `may` = user preferences govern. The LOEI concept (radio quality, QoS, power, memory) is what makes `should`/`may` non-trivial — UE may decline a preferred access that has poor signal.
- **HOLD is the only service with mandatory symmetric fanout**: both local and remote HOLD must propagate to ALL Access Legs with affected media. Other impacted services are UE-choice (any leg).
- **Gm fallback (§6.3.5) bridges IMS and CS service control**: when ICS loses SIP reachability, CS NAS (TS 24.008) takes over. This is an important resilience path — ICS isn't all-or-nothing.

Next chunk: ts23237-5 — TS 23.237 §6a — Inter-UE Transfer procedures (Collaborative Session establishment, media transfer, session control transfer, replication, loss of Controller UE)
Pages touched: [wiki/procedures/IMS-SC-media-and-services.md (new), wiki/sources/ts23237.md (updated — ts23237-4 Done), wiki/index.md (updated, 107→108 pages), wiki/log.md (this entry)]

---

## [2026-04-18] ingest | TS 23.237 §6a — Inter-UE Transfer procedures | chunk ts23237-5

Ingested §6a.0–§6a.8.3: complete IUT procedure set including Collaborative Session establishment (§6a.2), media transfer/add/delete/modify within Collaborative Session (§6a.3–§6a.4), Collaborative Session Control transfer (§6a.4a), session release (§6a.5), IUT without Collaborative Session (§6a.6), supplementary services for IUT (§6a.7), and target/session discovery (§6a.8).

**§6a.2 Collaborative Session establishment — 4 variants:** (1) transferring media from UE-1 to UE-2 (3 steps), (2) adding new media on UE-2 (3 steps), (3) at originating session setup — Access Leg at UE-2 established BEFORE remote party engagement (6 steps), (4) at terminating session setup — UE-1 routes Media-B to UE-2 in response to incoming call (5 steps).

**§6a.3 Media transfer — patterns:** Controller→Controllee same subscription (3 steps); Controller→Controllee different subscriptions (11 steps, two SCC ASes); Controller pulls media back from Controllee (3 steps); Controller moves media between Controllees (3/8 steps); Controllee-initiated transfer routes via SCC AS-2→SCC AS-1→Controller for authorization (8 steps).

**§6a.4 Media add/delete/modify:** All adding operations follow authorization→Access Leg setup→Remote Leg Update pattern. Release by Controllee (§6a.4.5) notifies Controller which then decides: pull back, redirect, or release session. Remote party add media: Controller may accept on itself or use IUT-Redirect-Media to send to Controllee. Remote party remove/modify: SCC AS handles directly for Controller-leg media; notifies Controller for Controllee-leg media.

**§6a.4a Session Control Transfer:** Push (Controller→Controllee without/with media, 7-8 steps), Pull (Controllee→Controller, 7 steps), Loss detection (SCC AS uses control loss preference to offer control to next UE in priority order; if none accepts → release session), Loss with media (UE-2 may accept control+media or just control then manage media separately).

**§6a.5 Session Release:** Controller-initiated: Release request → SCC AS releases Controllee Access Leg → releases Remote Leg (5 steps). Remote-initiated: SCC AS releases Controller and Controllee Access Legs in parallel (4 steps).

**§6a.6 IUT without Collaborative Session:** Source-initiated (4 steps: transfer request → SCC AS authorization + session setup at UE-2 + Remote Leg Update → UE-1 released). Target-initiated (5 steps: UE-2 discovers session, requests transfer, SCC AS authorizes with UE-1). UE-2 does NOT need IUT capabilities for §6a.6.1.

**§6a.7 Supplementary services:** HOLD, CONF, ECT are Controller-only — SCC AS rejects Controllee requests. Key: identification to remote = Controller UE identity always. HOLD by Controllee requires Controller authorization. CAT/CRS are Controller UE-associated.

**§6a.8 Discovery:** Target discovery via reg-event/presence subscription (13 steps) + optional capability query. Session discovery: UE-1 → SCC AS requests session info for peer UEs → SCC AS filters and responds (3 steps). Session info includes: session ID, GRUU/IMPU of UE, remote end identity, Controller UE identity, media type/status, Service Identifier.

Notable findings:
- **Controllee Access Leg established before remote party (§6a.2.3)** — SCC AS must have UE-2's SDP before constructing the offer to the remote party. This makes originating Collaborative Session setup a 3-party parallel operation (UE-1 channel, UE-2 channel, then remote party).
- **Cross-subscription SCC AS relay**: SCC AS-2 acts as a transparent relay for future Controllee requests in a cross-subscription Collaborative Session, forwarding them to SCC AS-1. This means the Controller's SCC AS is the true session anchor even for operations initiated by Controllees on different subscriptions.
- **Control loss preference is a session-time setting**: UE-1 can update it at any point during the session. The SCC AS stores it and activates on loss detection. Without it: session released. This is the IUT resilience mechanism.
- **Session Discovery requires SCC AS filtering**: the SCC AS applies operator policy and user service configuration before revealing session info. A UE cannot arbitrarily discover all sessions on a peer — the SCC AS gatekeeps this.

Next chunk: ts23237-6 — TS 23.237 §6c–§6d, §7–§8, Annexes B–C — SRVCC/DRVCC emergency sessions, security, charging, ATCF annex
Pages touched: [wiki/procedures/IMS-IUT-procedures.md (new), wiki/sources/ts23237.md (updated — ts23237-5 Done), wiki/index.md (updated, 108→109 pages), wiki/log.md (this entry)]

## [2026-04-18] ingest | TS 23.237 §6c–§6d, §7–§8, Annexes A–C | chunk ts23237-6

Ingested the final chunk of TS 23.237: §6c (SRVCC emergency session procedures), §6d (DRVCC emergency for WLAN), §7 (Security), §8 (Charging), Annex A (informative: Controller/Controllee operations table), Annex B (Normative: SRVCC enhancements — codec re-negotiation, codec inquiry, reject SRVCC), Annex C (Normative: ATCF in architectures without IMS-level roaming interfaces). Also completed §6a.8.3.3–§6a.12 that were missed in chunk 5.

**§6c SRVCC emergency:** EATF anchors IMS emergency session as routing B2BUA (same mechanism as ATCF for normal sessions). E-STN-SR allocated at origination, returned to UE in response. For active session AT: EATF uses E-STN-SR to identify session, performs Remote Leg Update toward E-CSCF/PSAP, then Source Access Leg Release. For early-dialog AT: UPDATE (not Re-INVITE) used; Session State Information sent to MSC to align CS call state. Multiple EATF instances handled by forking (I-CSCF sends one INVITE per EATF; matching EATF responds 200) or redirection (EATF-1 returns 3xx with backup EATF-2 contact).

**§6d DRVCC (WLAN emergency):** Identical to §6c but uses E-STN-DR. DRVCC is "make-before-break": WLAN emergency call stays active while CS leg is set up using NORMAL CS call (not CS emergency setup → no higher RRC priority). EATF correlates via E-STN-DR.

**§7 Security:** SC adds no requirements beyond TS 33.102 (CS) and TS 33.203 (IMS).

**§8 Charging:** SCC AS is the cohesive CDR anchor for the entire SC session lifecycle. Online charging is IMS-only — CS prepaid logic MUST NOT be invoked for anchored CS legs. SCC AS returns ICID to MSC Server in roaming for charging correlation. Start/stop alignment across domain transitions prevents double billing.

**Annex B codec enhancements:** B.2.1 re-negotiation (MSC Server initiates Re-INVITE post-SRVCC to negotiate codec-B end-to-end; eliminates ATGW transcoding). B.2.2 codec inquiry prior to SRVCC (PS-to-CS-Preparation-Request/Response exchange before radio reservation; ATCF returns IMS Preferred Codec List from MSC Supported Codec List). B.3 reject SRVCC if call state incompatible (IMS signals call state to MSC Server; MSC may abort).

**Annex C ATCF in HPLMN (no IMS roaming):** ATCF in HPLMN; MSC Server in VPLMN signals via Mx through IBCF chain. No ATGW media anchor — ATCF relays AT message to SCC AS with ATU-STI. Mid-call feature transport depends on SIP header fidelity across IBCF chain.

**§6a.9–§6a.12 (missed in chunk 5):** Target-initiated IUT — same and different subscription variants including "Solicit IUT" (UE-2 advertises intent; UE-1 decides). Media Flow Replication via MRF — source UE keeps original stream; MRF copies to Controllee (push/pull × same/different subscription). Session Replication — creates entirely new independent IMS sessions, no Collaborative Session, source session unchanged. Authorization — 3-layer SCC AS check (subscription + user preferences + remote party network restrictions); Ut interface for authorized UE list and incoming session routing criteria.

Notable findings:
- EATF is structurally the ATCF equivalent for emergency calls; E-STN-SR vs STN-SR is the only identifier distinction
- DRVCC "make before break" preserves emergency call continuity at the cost of no CS emergency priority escalation
- Online charging for SC is IMS-centralized: SCC AS is the only node that reports to OCS; CS prepaid CAMEL/IN is explicitly excluded from anchored CS legs
- Codec inquiry (B.2.2) before radio reservation is the proactive complement to codec re-negotiation (B.2.1) after SRVCC — together they eliminate the need for post-transfer transcoding in most cases
- Annex C ATCF (HPLMN without IMS roaming) trades local media anchoring for broader roaming compatibility; mid-call features are a best-effort based on IBCF chain SIP header transparency

**TS 23.237 COMPLETE — all 6 chunks ingested.**

Next chunk: N/A — TS 23.237 is complete. Next source TBD.
Pages touched: [wiki/procedures/IMS-IUT-procedures.md (updated — §6a.9–§6a.12 appended), wiki/procedures/SRVCC-emergency-procedures.md (new), wiki/concepts/IMS-SC-charging.md (new), wiki/procedures/SRVCC-enhancements.md (new), wiki/entities/ATCF.md (updated — Annex C), wiki/sources/ts23237.md (updated — ts23237-6 Done), wiki/index.md (updated, 109→112 pages), wiki/log.md (this entry)]

---

## [2026-04-19] ingest | TS 23.203 §4–§5 — PCC Architecture and Reference Points | chunk ts23203-1

Ingested §4 (high-level requirements) and §5 (architecture + 17 reference points) of 3GPP TS 23.203 v18.0.0 (Policy and Charging Control architecture). Also captured the beginning of §6.1 (binding mechanism, reporting, credit management, event triggers).

**§4 High-level requirements:** Charging: five models (volume/time/volume+time/event/no-charging); different rates per roaming, CSG, time-of-day, QoS; per-service usage limits; online charging thresholds; charging correlation between SDF and application levels (IMS/ICID). Policy: gating (per SDF), QoS at SDF/bearer/APN levels, QoS conflict handling (pre-emption priority), subscriber spending limits (Sy→OCS), usage monitoring (PCEF+TDF, volume or time), ADC (solicited=PCRF-instructed vs unsolicited=TDF-pre-configured), RAN congestion mitigation (RCAF→Np→PCRF→PCEF/TDF/AF), traffic steering ((S)Gi-LAN steering via TSSF/St), PFDF management (3rd-party PFDs via Nu/Gw/Gwn).

**§5 Architecture:** 11+ functional entities. Non-roaming: single PCRF; PCEF at PGW, BBERF absent (GTP) or at SGW (PMIP). Roaming home-routed: H-PCRF in HPLMN, V-PCRF in VPLMN, linked via S9; RCAF reports to V-PCRF. Roaming local-breakout: V-PCRF acts as primary PCRF for local PCEF; H-PCRF provides subscription-based policy via S9. 17 reference points catalogued in full (Rx, Gx, Sp, Ud, Gy, Gz, S9, Gxx, Sd, Sy, Gyn, Gzn, Np, Nt, St, Nu, Gw, Gwn).

**§6.1 (partial):** 3-step binding mechanism (session binding → PCC rule auth/QoS rule generation → bearer binding). BBF location depends on GTP (PCEF) vs PMIP (BBERF). Reporting: usage info reported to online+offline charging per charging key. Credit management: per charging key; credit pooling; re-auth triggers from OCS. Event Triggers Table 6.2: ~25 triggers including location change (cell/area/CN), SRVCC CS to PS, usage report, start/stop ADC, UE IP change, credit management session failure.

Notable findings:
- **Gx is the primary PCRF control channel to the enforcement point** — all PCC decisions flow from PCRF to PCEF over Gx; Rx is the AF input that drives PCRF decisions for IMS sessions (P-CSCF as AF).
- **The BBF location determines the PMIP vs GTP boundary**: in PMIP-based S5/S8, bearer binding moves to the SGW (BBERF), and Gxx (Gxc) is the PCRF-BBERF control plane. This is the architectural reason TS 23.402 defines BBERF.
- **Sy reference point (PCRF↔OCS) always in HPLMN**: policy counter state from OCS feeds dynamic policy decisions in PCRF — this is the mechanism for operator spending limit enforcement independently of Gy credit management.
- **ADC rules (Sd) operate orthogonally to PCC rules (Gx)**: TDF detects applications via ADC rules; PCEF enforces SDFs via PCC rules. The same IP-CAN session must not have both TDF and PCEF charging for the same traffic.
- **Event trigger "SRVCC CS to PS" at PCEF** allows PCRF to provision PCC rules to allow voice media over the default bearer during CS to PS SRVCC — cross-spec link to TS 23.216.

Next chunk: ts23203-2 — TS 23.203 §6.1 (complete) + §6.2 — Full functional description: PCRF, PCEF, AF, BBERF, TDF, RCAF entities
Pages touched: [wiki/concepts/PCC-architecture.md (new), wiki/sources/ts23203.md (new), wiki/index.md (updated, 112→114 pages), wiki/log.md (this entry)]

## [2026-04-19] ingest | TS 23.203 §6.1 (complete) + §6.2 — Functional Description | chunk ts23203-2

Ingested §6.1 (complete: QCI/ARP tables, IP-CAN bearer mode, gating, IMS emergency/RLOS/MPS, ADC, resource sharing, RUCI/RCAF, background data transfer, traffic steering, NBIFOM, PS Data Off) and §6.2 (complete: all 13 functional entity descriptions — PCRF, PCEF, AF, SPR, OCS, OFCS, BBERF, UDR, TDF, RCAF, SCEF, TSSF, PFDF).

**§6.1 notable content:** QCI Table A/B — 28 standardized QCI values across GBR/Non-GBR/delay-critical types; QCI 5 is highest-priority non-GBR (IMS signalling); QCI 1 is VoLTE voice (GBR, 100ms PDB); QCI 9 is default bearer. ARP: priority 1–15, pre-emption capability/vulnerability; 1–8 = serving-network authorized; 9–15 = home-only. NBIFOM enables simultaneous 3GPP+WLAN IP flow mobility. PS Data Off: UE feature gating mobile data (PCRF closes/deactivates PCC rules for non-exempt SDFs). MPS: PCRF elevates ARP for default bearer + IMS signalling bearer on priority bearer activation.

**§6.2.1 PCRF:** Full functional description including: input sources table (PCEF, BBERF, AF, SPR, OCS, RCAF), session binding decision logic (IPv4/IPv6/PDN correlation), V-PCRF/H-PCRF role split, multiple BBF classification (primary/non-primary BBERF during handover), usage monitoring control, subscriber spending limits (Sy), background data transfer (Nt/SCEF reference ID mechanism), MPS, IMS emergency/RLOS handling, 3GPP PS Data Off.

**§6.2.2 PCEF:** SDF detection (SDF template = set of SDF filters OR application ID; downlink applies all templates in precedence order; uplink applies templates on that bearer only); measurement granularity (volume/time/combined/event per bearer+charging-key pair, optional per service-ID); QoS enforcement (bearer GBR = sum of active GBR rules; bearer MBR = max; resource sharing = max across sharing group); gate control; BBF location (GTP→PCEF, PMIP→BBERF); ADC at PCEF when TDF absent; traffic steering policy identifiers; PS Data Off gate/rule enforcement.

**§6.2.3–§6.2.13:** AF (P-CSCF as IMS AF; sponsored data; background data transfer; RAN congestion re-try). SPR (subscription data model: services, QoS limits, charging info per APN, MPS, IMS signalling priority, spending limits, ADC profile, PS Data Off exemptions). OCS/OFCS (brief; separate specs cover protocol details). BBERF (PMIP bearer binding; uplink bearer binding verification — compares UE-signalled QoS against PCRF-authorized QoS). UDR (UDC architecture SPR replacement). TDF (solicited ADC via Sd vs unsolicited pre-config; start/stop reporting; enforcement gating/redirect/BW/charging; TDF session measurement). RCAF (RUCI context per UE+APN; routing to PCRF via SLF; congestion level reports; PCRF configures reporting restrictions). SCEF (AF role via Rx; Nt background data; Nu/PFDF management). TSSF (steering policy execution). PFDF (3rd-party PFD management; push/pull to PCEF via Gw and TDF via Gwn).

Notable findings:
- **PCEF SDF detection is asymmetric**: downlink checks ALL activated SDF templates on the IP-CAN session; uplink checks only templates bound to the specific bearer. This is architecturally significant for correct PCC rule design.
- **GBR per bearer = SUM of GBR rules**: not the max. This means the PCEF actively sums all active GBR PCC rules when establishing or modifying a GBR bearer — contrasts with MBR which uses max.
- **Inactivity Detection Time (IDT)**: PCRF-set parameter stops time-based charging measurement during idle periods — prevents billing for an open session with no traffic.
- **Background data transfer (Nt)**: the agreed policy is stored in SPR with a Reference ID; the AF uses this ID in the Rx AAR to reclaim the negotiated window at session time. This is an operator-level QoS pre-negotiation mechanism.
- **RCAF routes RUCI to V-PCRF in roaming**: RCAF is deployed in visited network; in home-routed roaming, V-PCRF receives RUCI and forwards to H-PCRF if H-PCRF holds subscription data.

Next chunk: ts23203-3 — TS 23.203 §6.3–§6.12 (PCC rules, QoS rules, ADC rules, usage monitoring, application detection, traffic steering, NBIFOM details, and other per-feature §6 subsections)
Pages touched: [wiki/entities/PCEF.md (new), wiki/entities/PCRF.md (updated with §6.2.1 detail), wiki/concepts/QCI-characteristics.md (new), wiki/sources/ts23203.md (ts23203-2 marked done), wiki/index.md (updated, 114→116 pages), wiki/log.md (this entry)]

## [2026-04-19] ingest | TS 23.203 §6.3–§6.12 — Rule Schemas and Feature Subsystems | chunk ts23203-3

Ingested §6.3 (PCC rule operations), §6.4 (IP-CAN session policy info), §6.4a (TDF session policy info), §6.4b (APN-AMBR), §6.5 (QoS rule), §6.6 (Usage Monitoring Control), §6.7 (S2c IP flow mobility routing), §6.8 (ADC rule), §6.9 (spending limits), §6.11 (Traffic Steering Control), §6.12 (NBIFOM routing rule). Also read §7.1–§7.6 (procedure introductions and PCRF discovery) which will be covered in chunk ts23203-4.

**§6.3 PCC rule operations:** Activation (dynamic = full info via Gx; predefined = identifier only), deactivation (all rules on bearer deactivated at bearer termination), deferred activation/deactivation mechanism. Predefined rules are bound one-to-one per IP-CAN session.

**§6.4–§6.4b Session policy information:** IP-CAN session/bearer info (Table 6.4): charging info, default charging method, event triggers, authorized QoS per bearer, authorized MBR per QCI, revalidation time limit, PRA identifiers, default NBIFOM access. TDF session info (Table 6.4a): adds ADC revalidation time limit, max DL/UL bit rate (note: max DL = DL APN-AMBR recommended to avoid discard). APN-AMBR control (Table 6.4b-1): Authorized APN-AMBR (conditional per RAT/IP-CAN type), Subsequent APN-AMBR + APN-AMBR change time (up to 4 instances with different timings).

**§6.5 QoS rule (BBERF):** Derived from PCC rules; same SDF template + precedence + QoS info. Key difference from PCC rule: BBERF only supports SDF filter templates (no Application ID). PCRF must maintain 1:1 correspondence PCC rule ↔ QoS rule at all times.

**§6.6 Usage Monitoring Control:** Monitoring key groups PCC/ADC rules with a common budget. Table 6.6: volume threshold, time threshold, monitoring time (re-applies thresholds at that point), subsequent thresholds (for post-monitoring-time period), inactivity detection time (stops time clock during idle). PCRF should avoid double-counting by not monitoring same traffic via both PCC and ADC rules.

**§6.8 ADC rule:** Full Table 6.8 with 20+ fields. Application identifier (DPI) or SDF filter list (mutually exclusive). DSCP marking for (S)Gi-LAN. Sponsored data fields (Sponsor ID, ASP ID). Traffic steering policy identifier(s) — separate for UL and DL direction. Sd reference point for TDF; St reference point for TSSF.

**§6.9 Spending limits:** Initial/Intermediate/Final Spending Limit Report Requests via Sy. OCS returns current + pending counter statuses with future activation times. PCRF uses as input to QoS/gating/charging decisions.

**§6.11 Traffic Steering:** Three paths (Gx→PCEF, Sd→TDF, St→TSSF). Identifiers are local references — policy content is not transported over PCC interfaces. St operations: provision/modify/remove. TSSF uses UE IP + APN to identify the PDN.

**§6.12 NBIFOM routing rule:** PCEF generates routing rules from UE Binding Updates; reports to PCRF via Gx. PCRF creates/modifies PCC rules with Allowed Access Type to match. Routing Access Information change = PCRF updates Allowed Access Type in PCC rule. Rule removal = PCRF removes the PCC rule or just the Allowed Access Type depending on whether the PCC rule was NBIFOM-created.

Notable findings:
- **QoS rules are a strict derivative of PCC rules**: every active PCC rule must have a corresponding QoS rule in the BBERF. The PCRF is responsible for maintaining this invariant across handover procedures where the BBERF may temporarily be unavailable.
- **Monitoring time enables budget reset without PCRF intervention**: the Subsequent Volume/Time threshold mechanism allows the PCRF to pre-schedule a threshold renewal at a specific future time, avoiding a Gx/Sd round-trip just to reset the counter.
- **ADC rule DSCP field enables integrated traffic steering from TDF**: TDF marks packets → downstream (S)Gi-LAN nodes route based on DSCP without needing separate steering instructions, allowing TDF-based traffic identification to feed into steering decisions.
- **APN-AMBR scheduling**: up to 4 Subsequent APN-AMBR instances allow time-of-day pricing without requiring the PCRF to make a decision at each transition time.

Next chunk: ts23203-4 — TS 23.203 §7 (PCC Procedures: IP-CAN session establishment, modification, termination, PCRF discovery, Gateway Control session procedures, Spending Limit procedures)
Pages touched: [wiki/concepts/PCC-rules-reference.md (new), wiki/sources/ts23203.md (ts23203-3 marked done), wiki/index.md (updated, 116→117 pages), wiki/log.md (this entry)]

## [2026-04-19] ingest | TS 23.203 §7 — PCC Procedures and Flows | chunk ts23203-4

Ingested §7.1–§7.12: IP-CAN session establishment (§7.2), termination UE and GW-initiated (§7.3), modification GW and PCRF initiated (§7.4), subscription update (§7.5), PCRF discovery/DRA (§7.6), Gateway Control session procedures (§7.7: establishment during attach and BBERF relocation, termination BBERF and PCRF initiated, QoS rules request and provision), MPS priority change (§7.8), Sy spending limit procedures (§7.9.1–7.9.5), Np RUCI procedures (§7.10.1–7.10.3), Nt background data transfer (§7.11.1), PFD management pull and push (§7.12). Also read beginning of Annex A (GPRS access specifics) which is scoped to ts23203-5.

**§7.2 IP-CAN session establishment:** 19-step flow involving BBERF, PCEF, TDF, TSSF, PCRF, SPR, OCS. Case 1 (GTP, no BBERF) vs case 2a/2b (PMIP, with BBERF Gxx). PCRF queries SPR, makes policy decision, provisions PCC rules to PCEF, ADC rules to TDF, traffic steering info to TSSF. OCS credit established at both PCEF (Gy) and TDF (Gyn). BBERF performs bearer binding on received QoS rules.

**§7.3 Termination:** Full cleanup — AF notified, TDF session terminated (ADC rules removed, TDF returns credit), PCEF returns credit (Gy final report), PCRF cancels SPR subscription/stores remaining usage allowance, RCAF context released, TSSF traffic steering info removed. Final Spending Limit Report to OCS if this is the last IP-CAN session for the subscriber.

**§7.4 Modification:** GW-initiated covers both bearer-level changes and ADC-driven application detection events. PCRF-initiated covers AF-driven (Rx) and OCS spending limit and RCAF congestion and subscription-change triggers. PCRF notifies AF of bearer level events on both trigger paths.

**§7.6 PCRF Discovery/DRA:** DRA selects PCRF at first interaction (PDN GW establishes Gx) and caches the assignment. All subsequent reference points (Rx, Gxx, Sd, Np) for the same IP-CAN session are routed to the same PCRF via DRA. In roaming, separate DRAs in VPLMN and HPLMN select V-PCRF and H-PCRF respectively.

**§7.7 Gateway Control:** During attach: BBERF sends IP-CAN type + UE identity + PDN identifier + bearer establishment modes → PCRF returns QoS rules + event triggers → BBERF performs bearer binding. During BBERF relocation: target BBERF establishes new session, receives QoS rules; source BBERF terminated; if primary BBERF QoS rule install fails → PCRF removes the QoS rule from all BBERFs and removes the corresponding PCC rule from PCEF.

**§7.9 Sy procedures:** Initial/Intermediate/Final Spending Limit Report Request protocol: PCRF subscribes to counter status, OCS returns current + pending statuses with future activation times. Spending Limit Report is OCS-push to PCRF. Sy Session Termination is OCS-initiated (optional).

**§7.10 Np/RCAF:** Non-aggregated RUCI per UE per APN (DRA-routed by IMSI+APN). Aggregated RUCI for multiple UEs to a logical PCRF ID. PCRF stores RCAF identity when RUCI indicates congestion. UE mobility between RCAFs: new RCAF reports → PCRF releases old RCAF context.

**§7.11 Nt:** Background data transfer negotiation 9-step: SCEF sends ASP ID + volume + UE count + desired time window → PCRF queries SPR for existing transfer policies → PCRF derives transfer policy → H-PCRF responds with transfer policies + Reference ID → SCEF/AF confirms selected policy → H-PCRF stores in SPR. Max aggregated bitrate not enforced.

**§7.12 PFD:** Pull (PCEF/TDF-initiated, when PFDs absent or caching timer expired) and Push (PFDF-initiated with optional Allowed Delay). PFD IDs allow granular partial updates.

Notable findings:
- **IP-CAN session establishment step 14 is the master PCC provisioning step**: PCRF sends PCC rules, event triggers, charging info, and IP-CAN bearer establishment mode all in a single Gx interaction. This is the "commit" point for the PCEF — everything before is PCRF-internal decision-making.
- **DRA is a session affinity router, not a policy agent**: DRA has no PCC logic; it simply ensures session stickiness for a given IP-CAN session across all Diameter reference points. The DRA must remain transparent at the Diameter application layer.
- **Non-primary BBERF cannot authorize new QoS rules**: PCRF rejects requests from non-primary BBERFs for new QoS rule authorizations. Only the primary BBERF (same IP-CAN type as PCEF) can drive QoS rule activation. This prevents split-brain QoS scenarios during handover.
- **Sy subscription is session-scoped per PDN connection**: Initial SLR at session start, Final SLR at last session termination for that subscriber. Intermediate SLR for subscription changes mid-session. Counter status tracking ties spending limits to PDN-connection granularity.
- **Background data transfer is a session-independent pre-negotiation**: the Nt interaction between SCEF and H-PCRF happens outside of any active IP-CAN session. The Reference ID is the linkage mechanism — AF presents it at session time to claim the pre-negotiated terms.

Next chunk: ts23203-5 — Annex A.4 (3GPP GTP-EPC access-specific) + Annex H (EPC-based non-3GPP), which complete the normative PCC content for EPC/IMS use
Pages touched: [wiki/procedures/PCC-session-procedures.md (new), wiki/sources/ts23203.md (ts23203-4 marked done), wiki/index.md (updated, 117→118 pages), wiki/log.md (this entry)]

---

## [2026-04-19] ingest | TS 29.165 §13–§33 + Annex A — ICS, SRVCC, Presence, Messaging, OMR, IUT, LBO, MRB, Overload, STIR, Emergency, and SIP header summary [chunk ts29165-3]

Ingested §13–§33 and Annex A (Table A.1) from TS 29.165 v16.6.0. All content added to `wiki/interfaces/II-NNI.md`. TS 29.165 is now COMPLETE — all 3 chunks ingested.

**§13 ICS:** `g.3gpp.ics`/`g.3gpp.accesstype` feature tags; Target-Dialog header; REFER with `method=BYE`/`method=INVITE`.

**§14 Service Continuity:** 5 PS-to-CS SRVCC variants (basic/alerting/pre-alerting/ATCF/mid-call), CS-to-PS SRVCC, PS-to-CS DRVCC, CS-to-PS DRVCC, PS-PS access transfer. Feature tags: `g.3gpp.srvcc`, `g.3gpp.cs2ps-srvcc`, `g.3gpp.dynamic-stn`; ATCF tags: `g.3gpp.atcf`, `g.3gpp.atcf-mgmt-uri`, `g.3gpp.atcf-path`.

**§15 Presence:** PUBLISH/SUBSCRIBE/NOTIFY with pidf+xml, watcherinfo+xml, pidf-diff+xml. OMA R1.1 Suppress-If-Match; OMA R2.0 `application/vnd.oma.suppnot+xml`.

**§16 Messaging:** Page-mode (MESSAGE + recipient-list + im-iscomposing+xml), session-mode (MSRP), conference messaging.

**§17 OMR:** 9 SDP attributes for media routeing optimisation; globally unique IP realm names required.

**§18 IUT:** REFER+Replaces/Target-Dialog (no collaborative session); `g.3gpp.iut-controller` (collaborative); pull/push replication via `application/vnd.3gpp.replication+xml`.

**§19–§21 LBO/MRB/Overload:** `g.3gpp.trf`/`g.3gpp.loopback` tags for LBO; `g.3gpp.mrb` feature-capability for MRB; RFC 7339 feedback + SUBSCRIBE/NOTIFY `application/load-control+xml` for overload (MPS exempt).

**§22–§33 Advanced features:** History-Info `mp` for original destination identity; `+sip.clue` for telepresence; `premium-rate` URI for barring; PCRF/PCF and HSS/UDM P-CSCF restoration mechanisms; Resource-Share header; `cause=380` for service access translation; MCPTT/MCVideo/MCData ICSIs + affiliation + MBMS + functional alias; Attestation-Info/Origination-Id/verstat/607 for STIR; sos URI + emergency URN + NG-eCall MSD; `+g.3gpp.rlos` for RLOS; `+g.3gpp.ps-data-off` for PS Data Off; `+sip.app-subtype=webrtc-datachannel` + `a=dcmap` + `a=3GPP-qos-hint` for MTSI data channel.

**Annex A (Table A.1):** 76 SIP headers catalogued — 34 mandatory (m), 7 not applicable (n/a), ~11 conditional on roaming II-NNI, 4 conditional on trust, 4 conditional on non-roaming II-NNI.

Notable findings:
- **§29 STIR/SHAKEN only at non-roaming II-NNI**: Attestation-Info and Origination-Id headers are explicitly listed as non-roaming-only in Annex A. This makes semantic sense — attestation of calling number identity is end-to-end between home networks, not applicable in visited/roaming paths where the home network cannot attest for a UE on a foreign visited network.
- **§17 OMR realm name uniqueness is an inter-operator agreement concern**: The spec states that IP realm names at II-NNI must be globally unique — but there is no centralised registry. Operators must bilaterally coordinate realm name assignments to avoid collision, which is an operational rather than a protocol constraint.
- **§21 MPS exemption from load control is normative**: "MPS sessions shall not be subject to load filtering" is normative language. This prevents emergency and priority services from being throttled even during overload.
- **§28 mission critical (MCPTT) is the largest subsection**: §28 covers 9 distinct sub-features including affiliation (mandatory + negotiated mode), MBMS bearer negotiation, group regrouping, functional alias management. This reflects the scale of MCPTT standardisation at inter-operator boundaries.

Next chunk: ts29228-1 — TS 29.228 §1–§5 (Cx interface: HSS-I/S-CSCF, UAR/UAA, SAR/SAA, LIR/LIA, MAR/MAA message flows)
Pages touched: [wiki/interfaces/II-NNI.md (major update §13–§33 + Annex A), wiki/sources/ts29165.md (ts29165-3 done; COMPLETE), wiki/index.md (updated), wiki/log.md (this entry)]

---

## [2026-04-19] update | Correction: ts29228 was already complete; ts29165-3 "Next chunk" was wrong

TS 29.228 was ingested in full on 2026-04-17 (chunks ts29228-1 through ts29228-3). The "Next chunk: ts29228-1" pointer at the end of the ts29165-3 log entry was incorrect — no further ts29228 ingest is needed.

**Current wiki state after ts29165-3:**
All planned sources from Phases 1–3 + supplementary specs are COMPLETE:
- Phase 1: TS 23.401 §4, TS 23.228 §4 ✓
- Phase 2a: TS 23.401 §5 (chunks 2a-1 through 2a-5) ✓
- Phase 2b: TS 23.228 §5 (chunks 2b-1 through 2b-4) ✓
- Phase 3: TS 23.218 (chunks 3a-1 through 3a-3), TS 23.402 (chunks 3b-1 through 3b-7) ✓
- Supplementary: TS 23.167, TS 23.216, TS 23.237, TS 24.229, TS 29.165, TS 29.228, TS 29.229, TS 29.328, TS 29.329, TS 32.260, TS 32.299, TS 33.203 (chunk 1) ✓

**Remaining pending:**
- ts33203-2: TS 33.203 security annexes (H/I/L/M/N/O/P/R/T) — marked "COMPLETE for core wiki purposes" in source summary; these are access-specific security extensions (SIP Digest, TLS, NASS-bundled, GIBA, NAT traversal, fixed broadband)
- Phase 4 entity deep-dives (MME, PGW, SGW, HSS, PCRF, P-CSCF, S-CSCF, TAS) — not yet started

Next action: ts33203-2 (IMS access security annexes) or Phase 4-1 (MME deep-dive)
Pages touched: [wiki/log.md (correction entry)]

---

## [2026-04-20] ingest | TS 24.301 §5.6 + §6 — Service Request + Full ESM Procedures [chunk ts24301-3]

Ingested §5.6 (EMM connection management: Service Request, Paging, NAS transport, Generic NAS transport, EMM STATUS) and §6 (EPS Session Management: overview/states, IP allocation, network-initiated/UE-requested bearer procedures, miscellaneous) from 3GPP TS 24.301 v17.6.0.

**§5.6 Service Request:** 17 trigger conditions; WB-S1 uses SERVICE REQUEST (T3417 5 s); NB-S1 uses CONTROL PLANE SERVICE REQUEST (T3417ext 1.5 s, may piggyback ESM DATA TRANSPORT or PDN CONNECTIVITY REQUEST); CSFB/CSNA uses EXTENDED SERVICE REQUEST. Reject cause table covers 17 EMM causes (#3/#6/#7/#8/#9/#10/#11/#12/#13/#15/#18/#22/#25/#31/#39/#40) with per-cause UE actions. Emergency (#7/#11) and RLOS (#39/#78) special handling. T3417: 4 retransmissions. Paging → UE responds with SERVICE REQUEST or EXTENDED SERVICE REQUEST.

**§6.1 ESM Overview:** Two procedure categories (bearer context related / transaction related). ESM sublayer states: UE has BEARER CONTEXT INACTIVE/ACTIVE and PROCEDURE TRANSACTION INACTIVE/PENDING; MME has 5 states including ACTIVE PENDING, INACTIVE PENDING, MODIFY PENDING. ISR: TIN must be "GUTI" to locally deactivate ISR before ESM changes. ESM/SM inter-system parameter mappings.

**§6.2 IP Allocation:** PDN type selection rules (UE sets IPv4/IPv6/IPv4v6 based on capability and prior rejections); ESM causes #50/#51 override dual-stack requests. §6.2A: ROHC header compression for CIoT.

**§6.3 General:** PTI/EBI addressing (6 figure-illustrated combinations); APN congestion control (T3396 per APN); back-off timer + re-attempt indicator for non-congestion rejection; WLAN offload, Serving PLMN rate control, APN rate control, 3GPP PS data off, Reliable Data Service, Ethernet PDN type; UAS/drone authorization (UUAA-SM for UAV ID, C2 pairing authorization, UAV flight authorization §6.3.13).

**§6.4 Network-initiated:** Default bearer activation (ACTIVATE DEFAULT EPS BEARER CONTEXT REQUEST, T3485, 4 retransmits; EBI/APN/PDN address/session-AMBR/QoS/WLAN offload/rate control in REQUEST); Dedicated bearer activation (TFT = "Create new TFT", linked EBI, REJECT causes #41–#45 for TFT errors); Bearer modification (MODIFY, T3486; operations differ from activation; cause #43 → local deactivation); Deactivation (DEACTIVATE, T3495; default bearer → deactivate all; cause #39 → re-initiate PDN connectivity; §6.4.4.6 5 local-deactivation cases without NAS signalling).

**§6.5 UE-requested:** Max 15 EPS bearer contexts (15-bearers bit); PDN connectivity (6 request types, 18 reject causes, ESM info request sub-procedure, T3482); PDN disconnect (linked EBI, cause #49 = last PDN, T3492); Bearer resource allocation (triggers dedicated activation or modification, T3480); Bearer resource modification (6 purposes including PS data off, ROHC, C2 UAS auth, T3481); Dual priority handling (T3396 bypass with non-low-priority retry).

**§6.6 Miscellaneous:** PCO/ePCO exchange rules (ePCO used for NB-S1/UAS/non-IP/Ethernet); ESM information request (T3489, 2 retransmits); Notification (#1 = SRVCC cancelled/IMS re-establishment required); Remote UE Report (ProSe relay, T3493, 2 retransmits); Control plane data transport (ESM DATA TRANSPORT, max 1358 octets MTU, Release assistance indication for NAS release optimization).

Notable findings:
- **§5.6.1.2 NB-S1 piggybacking is normative**: CONTROL PLANE SERVICE REQUEST may carry a complete PDN CONNECTIVITY REQUEST or ESM DATA TRANSPORT message in-band. This is the CIoT equivalent of the WB-S1 service request + data separation.
- **§6.3.13 UAS/drone procedures are fully specified in TS 24.301**: UUAA-SM (USS-level UAV authentication) and C2 authorization are defined as ESM sub-procedures with service-level-AA containers in PCO/ePCO IEs. This is R17 material.
- **§6.4.4.6 local deactivation without NAS signalling has 5 enumerated cases**: Each case is tied to a specific mobility event (service request partial bearer, TAU pending data, handover, E-UTRAN bearer release, NBIFOM). The EPS bearer context status IE in the next TAU REQUEST is the resynchronization mechanism.
- **§6.5.1 PDN CONNECTIVITY REJECT has 18 distinct cause values** with different back-off/re-attempt behaviors — notably cause #54 "PDN connection does not exist" resets the request type to "initial request" on retry.

Next chunk: ts24301-4 — §7 + §8 (Error handling + message definitions)
Pages touched: [wiki/procedures/NAS-service-request.md (NEW), wiki/procedures/NAS-ESM-procedures.md (NEW), wiki/sources/ts24301.md (chunk 3 done), wiki/index.md (131 pages), wiki/log.md (this entry)]

---

## [2026-04-20] ingest | TS 24.301 §7 + §8 — Error Handling + NAS Message Definitions [chunk ts24301-4]

Ingested §7 (error handling precedence rules) and §8 (all EMM and ESM message functional definitions) from 3GPP TS 24.301 v17.6.0.

**§7 Error handling:** 8-clause ordered precedence: §7.2 message too short → silently ignore; §7.3.1 unknown PTI → ESM STATUS #81 or for ACTIVATE/MODIFY messages UE may ACCEPT (retransmission) or REJECT #47; §7.3.2 unknown EBI → network replies #43, UE accepts unknown-EBI DEACTIVATE; §7.4 unknown message type → STATUS #97/#98; §7.5 mandatory IE errors → REJECT #96 for most REQUESTs, ACCEPT for DEACTIVATE; §7.6 unknown IEIs → ignore; §7.7 non-imperative errors → treat as absent (#100 for conditional IEs); §7.8 semantic errors → STATUS #95.

**§8.2 EMM messages (34 total):** All dual-significance except EMM INFORMATION and EMM STATUS (local). Key highlights: (1) SERVICE REQUEST uses non-standard structure — KSI+5-bit SN+Short MAC rather than standard security header; (2) SECURITY MODE COMMAND includes optional Hash_MME to detect replayed ATTACH/TAU REQUEST without integrity; (3) CONTROL PLANE SERVICE REQUEST (NB-S1) may piggyback ESM message container or SMS NAS message container; (4) TAU ACCEPT carries 30+ optional IEs covering GUTI reallocation, bearer sync, PSM/eDRX, Service Gap Control, and RACS (UE radio capability ID); (5) EXTENDED SERVICE REQUEST carries UE request type and Paging restriction for MUSIM.

**§8.3 ESM messages (25 total):** All dual-significance except NOTIFICATION (local). Key highlights: (1) ACTIVATE DEFAULT EPS BEARER CONTEXT REQUEST is the richest ESM message — carries APN-AMBR, Connectivity type (LIPA indicator), Header compression configuration (CIoT), Control plane only indication, and Serving PLMN rate control; (2) BEARER RESOURCE MODIFICATION REQUEST can carry Header compression configuration for ROHC re-negotiation triggered by N1→S1 inter-system change; (3) ESM DUMMY MESSAGE used during attach when UE does not request PDN connection; (4) ESM DATA TRANSPORT (§8.3.25) carries Release assistance indication for NAS connection release optimization after single CP data exchange; (5) REMOTE UE REPORT/RESPONSE covers ProSe UE-to-network relay connected/disconnected UE reporting.

Notable findings:
- **§7.3.2 DEACTIVATE with unknown EBI is accepted**: This allows the network to deactivate a bearer the UE has already locally deactivated without causing a reject loop — important for state re-synchronization.
- **SERVICE REQUEST non-standard structure**: The KSI+SN+Short MAC format (§8.2.25) means this is the only EMM message that cannot be wrapped in a SECURITY PROTECTED NAS MESSAGE — integrity is inherent in the short MAC computed over NAS COUNT.
- **EMM INFORMATION is local significance**: The network sends network name, time zone, and daylight saving time to the UE, but the UE may ignore it entirely. No acknowledgment is specified.
- **ESM DUMMY MESSAGE enables attachments without PDN connection**: Allows the UE to attach without triggering a PDN connectivity request — required for NB-IoT CP optimization devices that manage PDN connectivity separately.

Next chunk: ts24301-5 — §9 + §10 + Annexes (IE coding + timers + cause tables)
Pages touched: [wiki/protocols/NAS-message-reference.md (NEW), wiki/sources/ts24301.md (chunk 4 done), wiki/index.md (132 pages), wiki/log.md (this entry)]
