# Wiki Log
_Append-only event history. Parse with: `grep "^## \[" log.md | tail -10`_

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
