---
title: "Source: TS 23.402 — Non-3GPP Access Architecture"
type: source
tags: [TS-23.402, non-3GPP, WLAN, ePDG, trusted, untrusted, S2a, S2b, S2c, PMIPv6, DSMIPv6, GTP, IPMS, AAA, roaming]
sources: [ts_123402v150300p.pdf]
updated: 2026-04-10
---

# Source: 3GPP TS 23.402 v15.3.0 — IN PROGRESS (§5–§6, §8+ remaining)

**Full title:** Architecture enhancements for non-3GPP accesses
**Version:** 15.3.0 (Release 15)
**PDF:** `raw/papers/ts_123402v150300p.pdf`
**Sections ingested:** §4.1–§4.13 (PDF pp 19–80), §5.1–§5.13 (PDF pp 80–109), §6.1–§6.8 (PDF pp 110–138), §7.1–§7.11 (PDF pp 151–189), §8.1–§8.2 (PDF pp 190–208)

---

## Scope of Ingested Content

TS 23.402 defines how UEs access the EPC via non-3GPP accesses (WLAN, WiMAX, CDMA2000,
fixed broadband). It specifies the architecture (trusted vs untrusted non-3GPP), the
mobility protocols (PMIPv6/GTP on S2a/S2b, DSMIPv6 on S2c), procedures for attachment
and handover, and the selection functions for PDN GW and ePDG.

---

## §4.1 — General Concepts

### §4.1.2 — WiMAX Interworking
Basic WiMAX integration into EPC; uses same reference model as other non-3GPP accesses.

### §4.1.3 — IP Mobility Management Selection (IPMS)

Determines NBM (network-based, PMIPv6) vs HBM (host-based, DSMIPv6/MIPv4) per PDN
connection at initial attach. Final decision by HSS/AAA — UE indicates capability only.
IPMS is separate from PMIP vs GTP selection on S5/S8.

- **4 initial attach cases**: DSMIPv6-only UE → S2c; MIPv4-only → S2a; neither
  indicated → NBM; no capability → assume no HBM, use NBM
- **Handover cases (§4.1.3.2.2–4.1.3.2.3)**: 6 inter-access handover scenarios ensuring
  IP address continuity by reusing same PGW; IPMS rules govern mobility protocol
  continuity on 3GPP ↔ non-3GPP transitions

### §4.1.4 — Trusted/Untrusted Detection

Trust is NOT a property of the access technology. Two determination mechanisms:
1. 3GPP-based EAP authentication result conveys trust
2. Pre-configured policy in UE (H-ANDSF or USIM)

All PDN connections via one access share the same trust classification.

### §4.1.5 — Non-Seamless WLAN Offload

UE routes traffic via WLAN without EPC: local IP address, no IP preservation, ePDG not
required. Controlled by ANDSF or local UE policy.

---

## §4.2 — Architecture Reference Models

Four reference model variants:

| Model | PGW location | Key path |
|---|---|---|
| Non-roaming S5+S2a/S2b | HPLMN | ePDG/TWAN → PGW direct (S2b/S2a) |
| Non-roaming S2c | HPLMN | UE → PGW directly via DSMIPv6 (no ePDG) |
| Roaming home-routed | HPLMN | ePDG(VPLMN) → SGW(VPLMN) → PGW(HPLMN) via PMIP-S8 |
| Roaming local breakout | VPLMN | PGW(VPLMN); Gx→vPCRF→S9→hPCRF |

Key nodes: ePDG, 3GPP AAA Server/Proxy, SGW (roaming anchor), PGW, PCRF h/v, HSS.

---

## §4.3 — Network Element Functions

### §4.3.1.2 — Trusted/Untrusted Non-3GPP Access Node
Trust determined by operator/HSS-AAA policy, not by access technology itself.

### §4.3.2 — MME Additional Functions
HRPD Pre-registration; handover coordination with non-3GPP.

### §4.3.3.2 — SGW as Non-3GPP Anchor (Roaming)
In VPLMN S8-S2a/b chaining: SGW acts as local non-3GPP anchor + MAG for PMIPv6 toward HPLMN PGW.

### §4.3.3.3 — PGW in Non-3GPP Context
- LMA for PMIPv6 S2a and S2b
- DSMIPv6 Home Agent for S2c
- MIPv4 Home Agent for S2a MIPv4 FACoA
- GTP endpoint for GTP-based S2a/S2b

### §4.3.4 — ePDG Functions
See [ePDG entity page](../entities/ePDG.md) for full detail.
Core: IKEv2/IPsec termination (SWu), MAG (S2b), TFT-based routing, IPsec SA per PDN or
per bearer, QoS from AAA, MOBIKE, LI, charging.

### §4.3.5 — PCRF (Home/Visited)
hPCRF terminates: Gx (PGW), Gxa (trusted non-3GPP), Gxb (ePDG, not specified this
Release), Gxc (SGW as non-3GPP anchor). vPCRF terminates same + S9 to hPCRF.

---

## §4.4 — Reference Points

### §4.4.1 — All Non-3GPP Reference Points
S2a, S2b, S2c, SWu, SWa, STa, SWm, SWn, SWx, SWd, S6b, Gxa, Gxb, Gxc, S9, PMIP-S8, SGi.
Full table in [Non-3GPP Access Architecture](../concepts/non-3GPP-access-architecture.md).

### §4.4.2.1 — S5 Requirements
S5 unchanged; PGW serves as common anchor for both 3GPP and non-3GPP.

---

## §4.5 — Selection Functions

### §4.5.1 — PDN GW Selection (S2a/S2b)
ePDG/TWAN queries 3GPP AAA Server/Proxy; AAA returns PGW FQDN/IP + APN + PLMN.
On handover: HSS provides existing PGW to ensure IP continuity.

### §4.5.2 — PDN GW Selection (S2c)
UE discovers HA address via PCO, IKEv2 Config Payload, DHCP, or DNS.

### §4.5.3 — S-GW Selection (Non-3GPP)
Only for VPLMN roaming S8-S2a/b chaining. Selected by 3GPP AAA Proxy (not MME).

### §4.5.4 — ePDG Selection
UE constructs Operator Identifier FQDN or TAI/LAI FQDN. DNS resolution.
Configurable via H-ANDSF/USIM. Single ePDG per UE for all PDN connections.

---

## §4.6–§4.13 (chunk 3b-2)

§4.6.1 (NAI user identity; IMEI fallback for emergency); §4.6.2 (EPS Bearer ID on GTP S2b/S2a: allocated by ePDG/TWAN, independent namespace from S5/S8, may overlap in value, MAPCON designates distinct TFAs); §4.7.1 (PMIP S5/S8 IPv4 via DHCPv4 relay at SGW; IPv6 prefix via Router Advertisement; deferred IPv4 allocation; DHCPv4 Address Allocation Procedure Indication in PBA); §4.7.2 (Trusted S2a: MAG acts as DHCPv4 relay/server; static IP support from HSS/AAA); §4.7.3 (Untrusted S2b: two IPs — UE local IP for IPsec outer header + PDN IP(s) from PGW via ePDG/IKEv2 Config Payload; static IP from HSS/AAA during auth); §4.7.4 (S2c: CoA from ePDG; HNP from PGW; HoA from prefix via autoconfiguration; IPv4 HoA optional via DSMIPv6); §4.7.5–4.7.6 (IPv6 Prefix Delegation via DHCPv6 PD on S2c/PMIP S5/S8); §4.8 ANDSF (H-ANDSF/V-ANDSF architecture; S14 interface; pull/push; ISMP for single-radio access selection; ISRP for multi-radio routing IFOM/MAPCON; IARP for per-APN and NSWO routing; WLANSP for WLAN network selection including SSID/BSSID/HS2.0 criteria; 3-step WLAN selection procedure; Home/Visited Network Preferences; co-existence with RAN rules and LWA/LWIP/RCLWI); §4.9.1 (EAP-based access auth over SWa/STa per TS 33.402); §4.9.2 (Tunnel auth UE↔ePDG on SWu via IKEv2/IPsec); §4.9.3 (EAP re-auth RFC 6696 for fast WLAN link setup; reduces VoIP handoff delay); §4.10.1 (QoS: packet filters + QCI/ARP/MBR/GBR; PCRF signals same over Gxa/Gxb/Gxc as over Gx); §4.10.3 (EPS bearer PMIP S5/S8: Radio Bearer + S1 bearer + GRE/PMIPv6 S5; SGW maps TFT→S1; PGW enforces APN-AMBR); §4.10.4 (PCC: dynamic PCRF via Gx/Gxa/Gxc; static if no Gxa/b; ePDG static profile from AAA via SWm); §4.10.5 (GTP S2b PDN connectivity: single IPsec SA per PDN connection — ePDG routes by TFT; single IPsec SA per bearer — 1:1 SA↔S2b bearer, QCI/GBR/MBR in IKEv2, TSi/TSr not for routing); §4.11 (charging: per-UE per-QCI accounting for inter-operator; TS 32.240/32.260); §4.12 (multiple PDN: same APN → same PGW; at most 1 3GPP + 1 non-3GPP access simultaneously; MAPCON/IFOM for simultaneous); §4.13 (detach principles: multi-access UE must detach each; preserve via PDN disconnect + HO; HSS-initiated covers all nodes).

## §7.1–§7.5 (chunk 3b-2)



§7.1.1 (PMIPv6 S2b stack: IKEv2+IPv4/6 on SWu; PMIPv6+IPv4/6+GRE on S2b; user plane: IPsec+GRE from UE to ePDG to PGW); §7.1.2 (GTP S2b stack: IKEv2+IPv4/6 on SWu; GTPv2-C on S2b control; IPsec relaying + GTP-U on S2b user plane); §7.1.2 (S2c DSMIPv6+IPsec+IKEv2); §7.2.1 (Initial Attach PMIPv6 S2b 9-step: access auth → IKEv2 + EAP-AKA → ePDG gets PGW address from AAA → PBU(NAI,APN,AccessType,APN-AMBR,location) → IP-CAN session at PGW → PGW informs AAA → PBA(IP addrs,GRE key,Charging-ID) → IKEv2 completion → IP addr config); §7.2.2 (Void); §7.2.3 (Chained PMIP S8-S2b — see §6.2.4); §7.2.4 (Initial Attach GTP S2b: same as PMIPv6 but B.1=Create Session Request with IMSI/APN/RAT/ePDG TEID/EPS Bearer QoS/location; D.1=Create Session Response with PGW TEID/PDN addr/APN-AMBR); §7.2.5 (Emergency attach GTP S2b: releases existing connections, emergency ePDG selection per §4.5.4a, Emergency Config Data replaces subscription, IMEI as identity if no IMSI, no external AAA); §7.3 (S2c initial attach: access auth → IKEv2 to ePDG → IPsec → MIPv6 SA to PGW → BU(HoA,CoA) → IP-CAN session → BA(IPv4 HoA)); §7.4.1.1 (UE/ePDG detach PMIPv6: IKEv2 release → PBU(lifetime=0) → AAA de-reg → PCEF session termination → PBA(lifetime=0) → resource release); §7.4.2.1 (HSS/AAA detach PMIPv6: Detach Indication → ePDG follows §7.4.1.1 → Detach Ack); §7.4.3.1 (UE/ePDG detach GTP: IKEv2 release → Delete Session Request with Linked EPS Bearer ID/UWAN Release Cause/location → IP-CAN termination → Delete Session Response); §7.4.4.1 (HSS/AAA detach GTP: follows §7.4.2.1 then §7.4.3.1); §7.5.2 (S2c UE detach: BU(lifetime=0) → AAA notify → PCEF termination → BA → IKEv2 SA termination → IPsec tunnel termination); §7.5.3 (S2c HSS/AAA detach: Session Termination → PGW Detach Request → UE Ack → PCEF termination → IKEv2 SA term); §7.5.4 (S2c PGW-initiated: explicit Detach Request or implicit lifetime expiry → cleanup).

## §5.1–§5.13 (chunk 3b-5)

§5.1.1 (Void); §5.1.2 (General: TS 23.401 defines GTP S5/S8; §5 defines PMIP S5/S8 deltas only); §5.1.3 (Control Plane: PMIPv6/IPv4-IPv6 SGW↔PGW); §5.1.4.1–5.1.4.4 (User Plane: E-UTRAN PDCP/GTP-U/GRE; 2G S4/A-Gb SNDCP/GTP-U/GRE; 3G S4/Iu GTP-U relay/GRE; 3G S12 direct UTRAN→SGW/GRE); §5.2 (Initial E-UTRAN Attach PMIP S5/S8: steps A.1-A.4 = old SGW de-registration between TS 23.401 steps A→B: BBERF GW Control Session Term + PBU(lifetime=0) + PCEF IP-CAN Term + PBA; steps B.1-B.4 = second de-registration A→C; steps C.1-C.5 = new S5 session between B→C: C.1 GW Control Session Establishment(IMSI,APN-AMBR,Default Bearer QoS,UE Location) → C.2 PBU(MN NAI,Lifetime,AT-Type=E-UTRAN,HO-Indicator,APN,GRE key,Charging Chars) → C.3 PCEF IP-CAN Session Establishment → C.4 PBA(UE Addr,GRE uplink,Charging ID,APN-AMBR,DHCPv4 Alloc Indicator) → C.5 GW Control+QoS Rules Provision; Emergency Attach: IMSI marked unauthenticated if so provided); §5.3 (Detach PMIPv6 S5/S8: A.1 BBERF GW Control Session Term + A.2 PBU(lifetime=0) + A.3 PCEF IP-CAN Term + A.4 PBA; repeats per PDN); §5.4.1 (Dedicated bearer general: PCRF sends PCC to SGW(BBERF) not PGW first; SGW generates TFT from QoS policy; PMIP difference — SGW drives Create Bearer, not PGW); §5.4.2 (Dedicated bearer activation: PCRF→SGW GW Control Rules A.1 → SGW determines bearer → TS 23.401 §5.4.1 Create Bearer Request to MME between A.1 and B.1 → SGW→PCRF GW Control B.1 → PCRF→PGW PCC Rules B.2); §5.4.3.1 (PCC-initiated QoS modification: same Figure 5.4.1-1 pattern, SGW uses QoS to modify TFT, sends Update Bearer to MME); §5.4.3.2 (HSS-initiated subscribed QoS: A.1 SGW GW Control+QoS Policy Request → A.2 PCRF→PGW PCC Rules Provision → TS 23.401 §5.4.2.1 Update Bearer Request → B.1 GW Control end → B.2 PCC Rules final); §5.4.4 (Bearer modification without QoS update: SGW uses QoS to determine TFT update, sends Update Bearer(TFT only) per TS 23.401 §5.4.3); §5.4.5.1 (PCC-initiated dedicated bearer deactivation: same Figure 5.4.1-1, SGW determines deactivation, Delete Bearer Request to MME); §5.4.5.3 (MME-initiated deactivation: SGW reports deleted QoS rules to PCRF A.1 + PCRF→PGW A.2; SGW follows TS 23.401 §5.4.4.2 independently); §5.5 (UE-initiated resource request/release: UE sends TAD via NAS → SGW→PCRF GW Control+QoS Rules Request → PCRF decision → SGW enforces → PCEF update); §5.6.1 (UE-requested PDN connectivity: Alt A = GW Control Session Establishment + PBU + IP-CAN Establishment + IP-CAN Modification + PBA; Alt B = PBU first(lower jitter) + IP-CAN Modification + PBA; re-establishment after non-3GPP HO: Handover Indicator in PBU; Charging ID reuse from PMIP source or Default Bearer GTP source); §5.6.2.1 (PDN disconnection UE/MME/SGW: A.1 GW Control Session Term + A.2 PBU(lifetime=0) + A.3 PCEF IP-CAN Term + A.4 PBA); §5.6.2.2 (PDN GW-initiated PDN disconnection: A.1 Binding Revocation Indication(PDN address) → TS 23.401 §5.4.4.1 bearer deactivation → B.1 BBERF GW Control Session Term → B.2 Binding Revocation Ack); §5.7.0 (Intra-LTE TAU/HO without SGW relocation: SGW→PCRF GW Control+QoS Rules Request(RAT change+UE Location) → PCRF→PGW PCC Rules Provision; no PBU/PBA); §5.7.1 (Intra-LTE TAU/HO with SGW relocation: new SGW GW Control Session + PCC Rules Provision + PBU + PBA + End Marker Indication; old SGW GW Control Session Term; EPS Bearer ID transferred S10/S11 before step A); §5.7.2 (Inter-RAT TAU/RAU UTRAN↔E-UTRAN: without SGW relocation = GW Control+QoS Rules Request; with SGW relocation = same as §5.7.1 pattern); §5.8 (ME Identity Check: same as GTP-based per TS 23.401 initial attach); §5.9 (UE-triggered Service Request: SGW→PCRF GW Control+QoS Rules Request if RAT Type changed); §5.10.1–5.10.10 (S4-based GERAN/UTRAN: PMIP equivalents for PDP Context Activation/Deactivation, RAU/SRNS, intra/inter-SGSN change, PDN GW initiated deactivation; wherever TS 23.060 has GTP box, replaced by GW Control+QoS Rules Request pattern); §5.11 (PDN GW-initiated IPv4 address delete: PCEF IP-CAN Modification → GW Control+QoS Rules Provision → TS 23.401 §5.4.3 bearer mod without QoS → Binding Revocation Indication(IPv4 address only, NOT full PDN) → Binding Revocation Ack); §5.12 (Location Change Reporting: SGW→PCRF GW Control+QoS Rules Request(UE Location+CSG) → PCRF→PGW PCC Rules Provision); §5.13 (MTC: common procedures in TS 23.401; PDN GW overload rejects PBU with APN congested indication + back-off timer; low access priority indicator forwarded in PBU).

## §6.1–§6.8 (chunk 3b-4)

§6.1 (Protocol stacks: PMIPv6 on S2a — MAG in trusted access, LMA in PGW, GRE user plane; MIPv4 FACoA on S2a — FA in trusted access, HA in PGW, MIPv4 UDP control; DSMIPv6 on S2c — UE=MN, PGW=HA, IKEv2-protected DSMIPv6); §6.2.1 (Initial Attach PMIPv6 S2a 11-step: Non-3GPP L2 procs → EAP auth via STa → L3 Attach Trigger → GW Control Session Establishment(BBERF Gxa) → PBU(MN-NAI,Lifetime,AT-Type,HO-Indicator,APN,GRE key) → IP-CAN Session Establishment → Update PDN GW Address → PBA(UE Addr,GRE uplink,Charging ID) → PMIP tunnel → QoS Rules Provision → L3 Attach Completion; roaming/LBO: vPCRF in path; non-roaming: vPCRF absent); §6.2.2 (Void); §6.2.3 (Initial Attach MIPv4 FACoA S2a 14-step: non-3GPP procs → EAP auth → Agent Solicitation/Advertisement → RRQ(MN-NAI,APN,reverse tunnel request) to FA → GW Control Session → FA relays RRQ to PGW → AAA interactions → PGW allocates home addr → Update PDN GW Address → RRP(Home Addr,Lifetime) FA→UE → MIPv4 tunnel established); §6.2.4 (Initial Attach Chained PMIP S8-S2a/b roaming 8-step: auth includes SGW selection from AAA; MAG sends PBU to SGW→SGW forwards to PGW; PGW IP-CAN session; Update PDN GW Address; PBA PGW→SGW→MAG; two chained PMIP tunnels MAG↔SGW↔PGW); §6.3 (Initial Attach DSMIPv6 S2c over trusted non-3GPP: Module A local IP connectivity — EAP auth + L3 attach + GW Control Session; Module B HA discovery — IKEv2 security association with PGW; HNP allocated by PGW; Module C Binding Update — BU(HoA,CoA) → IP-CAN Session Establishment → Binding Ack); §6.4.1.1 (UE/TNAN-initiated detach PMIPv6 7-step: access-specific detach trigger → GW Control Session Termination → PBU(lifetime=0) → Update PDN GW Address → PCEF IP-CAN Session Termination → PBA(lifetime=0) → non-3GPP resource release); §6.4.1.2 (Chained PMIP S8-S2a detach: two PBUs, two PBAs via SGW); §6.4.2.1 (HSS/AAA-initiated PMIPv6: UE De-registration Request → §6.4.1.1 → Detach Ack; PDN GW receives notification but MAG responsible for tunnel teardown); §6.4.2.2 (HSS/AAA-initiated chained: same pattern via §6.4.1.2); §6.4.3 (UE-initiated MIPv4 FACoA 9-step: RRQ(lifetime=0) → GW Control Session Term → relay to PGW → AAA → Update PDN GW Address → PCEF IP-CAN Term → RRP(lifetime=0) → non-3GPP release); §6.4.4 (Network-initiated MIPv4 FACoA: Registration Revocation(RFC 3543) → Update PDN GW Address → PCEF term → Registration Revocation Ack); §6.4.5 (HSS/AAA-initiated MIPv4 FACoA: Detach Indication → §6.4.4 → Detach Ack); §6.5.2 (UE-initiated S2c detach: BU(lifetime=0) → Update PDN GW Address → PCEF IP-CAN Term → BA → PCRF GW Control Term → IKEv2 SA Term → non-3GPP release); §6.5.3 (HSS/AAA-initiated S2c: Session Termination → Detach Request → Detach Ack → PCEF term → Session Term Ack → PCRF GW Control Term → IKEv2 SA Term; implicit detach omits steps 2-3); §6.5.4 (PDN GW-initiated S2c: same pattern, PGW sends Detach Request; implicit: steps 1-2 omitted); §6.6.1 (Dynamic PCC S2a: PCRF GW Control+QoS Rules Provision → TNAN enforces → PCC Rules Provision to PCEF); §6.6.2 (Dynamic PCC S2c: identical pattern); §6.7 (UE-initiated resource request/release: IP-CAN specific → TNAN reports to PCRF Gxa → PCRF decision → TNAN enforces → PCEF update); §6.8.1 (Additional PDN S2a PMIPv6: same as §6.2.1 per new APN; PDN Connection Identity for multiple PDNs per APN; also used for re-establishment after HO from 3GPP with Handover Indicator).

## §7.6–§7.11 + §8.1–§8.2 (chunk 3b-3)

§7.6.1 (UE-initiated additional PDN connectivity PMIPv6: same ePDG, new IKEv2 Child SA, new S2b PMIPv6 binding per PDN, IPMS applies independently per PDN); §7.6.2 (GTP S2b additional PDN: same flow, new Create Session per APN); §7.6.3 (S2c additional PDN: new IKEv2 SA per PDN, new MIPv6 BU to PGW-HA); §7.8 (S2c bootstrapping via DSMIPv6 Home Link Detection — HA discovery and BootStrap); §7.9.1 (PGW-initiated resource deactivation PMIPv6: PCRF IP-CAN Session Modification → Binding Revocation Indication to ePDG → IKEv2 IKE_SA or Child SA delete → non-3GPP resource release → Binding Revocation Ack → AAA update); §7.9.2 (PGW-initiated resource deactivation GTP: PCRF → PGW Delete Bearer Request with EPS Bearer ID + Linked EPS Bearer ID → ePDG IKEv2 INFORMATIONAL DEL_SPI → non-3GPP resource release → Delete Bearer Response with WLAN Location Info → AAA update); §7.10 (Dedicated S2b Bearer Activation GTP only: PCRF PCC decision → PGW Create Bearer Request(IMSI, EPS Bearer QoS, TFT, PGW U-plane TEID, Linked EPS Bearer ID) → ePDG allocates EPS Bearer ID → IKEv2 CREATE_CHILD_SA with Notify EPS_BEARER_INFO(TFT, QoS) → UE CREATE_CHILD_SA Response → ePDG Create Bearer Response(EPS Bearer ID, ePDG U-plane TEID, WLAN Location) → PCRF completion; single-SA-per-PDN mode: no new Child SA, TFT mapping only); §7.11.1 (PGW-initiated bearer modification: PCRF → PGW Update Bearer Request(EPS Bearer QoS, TFT) → ePDG IKEv2 INFORMATIONAL(EPS_BEARER_INFO) → UE Response → ePDG Update Bearer Response + WLAN Location); §7.11.2 (HSS-initiated subscribed QoS modification: HSS User Profile Update → AAA Notify → ePDG Modify Bearer Command → PGW PCEF IP-CAN Modification → PGW Update Bearer Request → ePDG Update Bearer Response); §8.1 (handover general: multi-PDN rules — one PDN per APN on Attach(HO); remaining PDNs via UE-requested PDN Connectivity; 3GPP→non-3GPP uses HSS PGW identity return; non-3GPP→3GPP uses Attach(HO) + MME/HSS coordinate PGW reuse; no simultaneous multi-access); §8.2.1.1 (non-3GPP → E-UTRAN GTP S5/S8 18-step: UE discovers E-UTRAN → Attach(HO) → Auth → Location Update(HSS returns PGW identity) → SGW Create Session Request(HO Indication) → SGW→PGW Create Session Request(HO) → opt PCEF IP-CAN Modification(access type change) → PGW Create Session Response(same IP + Charging ID) → SGW Create Session Response → Radio+Access Bearer setup → MME Initial Context Setup → MME Modify Bearer Request(eNB, HO Indication) → SGW Modify Bearer Request(prompts PGW tunnel switch; deferred PCC applied) → Modify Bearer Responses → UE sends/receives over E-UTRAN → UE-requested PDN Connectivity for remaining PDNs → PGW resource deactivation for non-3GPP (§7.9)); §8.2.1.2 (PMIP S5/S8 variant: Alt A = SGW GW Control Session → PBU → PCEF Modification → PBA; Alt B = PBU → PCEF Modification → PBA, lower jitter by separating binding from PCRF round-trip); §8.2.1.3 (UTRAN/GERAN variant: SGSN instead of MME; Activate PDP Context; S4-based Create Session Request).

---

## Coverage Status

**IN PROGRESS** — §4.1–§4.13 complete (chunks 3b-1, 3b-2). §5.1–§5.13 complete (chunk 3b-5). §6.1–§6.8 complete (chunk 3b-4). §7.1–§7.11 complete (chunks 3b-2, 3b-3). §8.1–§8.2 complete (chunk 3b-3).
Remaining: §9–§16 (additional procedures, HRPD, TWAN multi-connection, etc.).
