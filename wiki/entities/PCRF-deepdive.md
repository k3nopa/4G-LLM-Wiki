---
title: "PCRF Deep-Dive — Policy and Charging Rules Function Comprehensive Reference"
type: entity
tags: [PCRF, EPC, IMS, Gx, Rx, Gxc, Gxa, Gxb, S9, Np, Sp, PCC, QoS, VoLTE, bearer-binding, roaming, deep-dive]
sources: [ts_123401v150400p.pdf, ts_123228v150600p.pdf, ts_123402v150400p.pdf]
updated: 2026-04-11
---

# PCRF Deep-Dive — Policy and Charging Rules Function

**Base entity page:** [PCRF.md](PCRF.md)
**Spec references:** TS 23.203 (primary); TS 23.401 §4.4.7; TS 23.228 §5.4.5; TS 23.402 §4.3/§4.10/§5/§6/§7

---

## Architectural Position

The PCRF is the **policy decision point** of the EPC and IMS. It receives service descriptions from Application Functions (via Rx) and subscriber policy from the SPR, correlates them with active IP-CAN sessions (via Gx), and delivers PCC rules to enforcement points (PGW/PCEF, SGW/BBERF, ePDG/TWAN). In the non-3GPP domain it also terminates Gxa (trusted access) and Gxc (PMIP S5/S8 BBERF). In roaming, two PCRFs coordinate via S9.

```mermaid
graph LR
    subgraph "Application Functions"
        PCSCF["P-CSCF (AF)"] -->|Rx| HPCRF
        AS_AF["Other AF"] -->|Rx| HPCRF
    end
    subgraph "3GPP Enforcement"
        PGW["PGW (PCEF)"] -->|Gx| HPCRF
        SGW_BBERF["SGW (BBERF, PMIP)"] -->|Gxc| HPCRF
    end
    subgraph "Non-3GPP Enforcement"
        TWAN["TWAN (BBERF)"] -->|Gxa| HPCRF
        ePDG["ePDG"] -->|Gxb partial| HPCRF
    end
    subgraph "Policy Sources"
        SPR[("SPR\n(Subscriber Profile)")] -->|Sp| HPCRF
        RCAF["RCAF\n(RAN Congestion)"] -->|Np| HPCRF
    end
    subgraph "Roaming"
        VPCRF["V-PCRF\n(VPLMN)"] -->|S9| HPCRF
        VPCRF -->|Gx| VPGW["V-PGW (PCEF)"]
        VPCRF -->|Rx| VAF["Visited AF"]
    end
    HPCRF["H-PCRF\n(HPLMN)"]
```

---

## Complete Interface Table

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| **Gx** | PGW (PCEF) | Diameter (Gx app) | Bidirectional | Push PCC rules to PCEF; receive IP-CAN events, RAT/location, usage |
| **Gxc** | SGW (BBERF) | Diameter (Gxc app) | Bidirectional | PMIP S5/S8: GW Control Session with BBERF; deliver QoS rules |
| **Gxa** | TWAN (trusted non-3GPP BBERF) | Diameter (Gxa app) | Bidirectional | Trusted non-3GPP: GW Control Session with TWAN BBERF |
| **Gxb** | ePDG (untrusted non-3GPP) | Diameter (Gxb app) | Bidirectional | Untrusted non-3GPP: partially specified in Rel-15; PCC to ePDG |
| **Rx** | AF (P-CSCF, AS, other) | Diameter (Rx app) | Bidirectional | Receive media/service descriptions; authorize QoS; gate control |
| **S9** | V-PCRF ↔ H-PCRF | Diameter (S9 app) | Bidirectional | Roaming: H-PCRF ↔ V-PCRF policy coordination (local breakout) |
| **Sp** | SPR (Subscriber Profile Repository) | Internal/vendor | PCRF ← SPR | Fetch subscriber policy data: allowed QoS, charging keys, service profiles |
| **Np** | RCAF (RAN Congestion AF) | Diameter (Np app) | RCAF → PCRF | Receive RAN user-plane congestion information for policy decisions |

---

## Diameter Messages — Gx Interface

### IP-CAN Session Lifecycle

| Message | Direction | Trigger |
|---|---|---|
| CCR-Initial (IP-CAN Session Establishment) | PGW → PCRF | PDN connection created; PCEF registers with PCRF |
| CCA-Initial | PCRF → PGW | Initial PCC rules installed; default bearer QoS, gates, APN-AMBR |
| CCR-Update (IP-CAN Session Modification) | PGW → PCRF | RAT type change, ULI change, bearer binding event, usage threshold crossed, AN-GW change |
| CCA-Update | PCRF → PGW | Updated PCC rules; may add/remove/modify rules in response to event |
| CCR-Terminate (IP-CAN Session Termination) | PGW → PCRF | PDN connection deleted |
| CCA-Terminate | PCRF → PGW | Acknowledges; PCRF removes session state |
| RAR (Re-Auth-Request) — PCC Rules Provision | PCRF → PGW | PCRF-initiated rule push (e.g. Rx-triggered VoLTE bearer, operator policy change) |
| RAA (Re-Auth-Answer) | PGW → PCRF | Acknowledges rule installation; may include result/binding status |
| ASR (Abort-Session-Request) | PCRF → PGW | PCRF terminates IP-CAN session (e.g. subscription revoked) |
| ASA (Abort-Session-Answer) | PGW → PCRF | Confirms session abort |

### Key CCR-Update Event Triggers

| Event | PCRF Response |
|---|---|
| RAT type change (e.g. LTE → WLAN) | Re-evaluate policies; update APN-AMBR, QoS rules |
| UE location change (new cell/TA) | Check location-based policy; update if applicable |
| Bearer binding event (UL/DL gate change) | Update gate status in PCC rule |
| Usage threshold crossed | Trigger online charging refresh or apply throttle rule |
| AN-GW (SGW) change | Update GW Control Session binding; re-provision QoS |

---

## Diameter Messages — Gxc Interface (PMIP / BBERF)

The Gxc interface mirrors Gx but targets the SGW (BBERF) instead of the PGW. It carries **QoS Rules** (not full PCC rules — no charging parameters at the BBERF).

| Message | Direction | Purpose |
|---|---|---|
| CCR-Initial (GW Control Session Establishment) | SGW → PCRF | BBERF registers: IMSI, APN, RAT type, UE location, requested IP-CAN type |
| CCA-Initial | PCRF → SGW | QoS Rules installed at BBERF (event triggers, APN-AMBR, default bearer QoS) |
| CCR-Update (GW Control + QoS Rules Request) | SGW → PCRF | RAT/location change, resource request from UE (§5.5), HO re-establishment |
| CCA-Update | PCRF → SGW | Updated QoS Rules; BBERF generates TFT and drives bearer update to MME |
| CCR-Terminate (GW Control Session Termination) | SGW → PCRF | PDN disconnection or detach |
| CCA-Terminate | PCRF → SGW | Acknowledges |
| RAR (QoS Rules Provision) | PCRF → SGW | Dedicated bearer trigger: PCRF pushes QoS rule to BBERF |
| RAA | SGW → PCRF | Acknowledges; BBERF generates TFT and sends Create Bearer Request to MME |

**Gx vs Gxc distinction:**

| Aspect | Gx (PGW/PCEF) | Gxc (SGW/BBERF) |
|---|---|---|
| Carries PCC rules? | Yes (full: SDF filter + QoS + charging) | No — QoS rules only (no charging params) |
| Charging enforcement? | Yes (PCEF enforces charging) | No (SGW has no charging role in PMIP mode) |
| TFT generation? | PGW generates from PCC rule filters | SGW generates from QoS rule (BBERF role) |
| Session type | IP-CAN Session | GW Control Session |
| PCRF also sends to PGW? | Yes (B.2 PCC Rules Provision after bearer up) | PGW still receives B.2 separately |

---

## Diameter Messages — Gxa Interface (Trusted Non-3GPP)

Gxa is functionally identical to Gxc — it connects the PCRF to the trusted access BBERF (TWAN/WLAN gateway).

| Message | Direction | Purpose |
|---|---|---|
| CCR-Initial (GW Control Session Establishment) | TWAN → PCRF | Trusted non-3GPP attach: BBERF registers with PCRF |
| CCA-Initial | PCRF → TWAN | QoS Rules provision for trusted access |
| CCR-Update | TWAN → PCRF | Location/RAT change, UE resource request (§6.7) |
| CCA-Update | PCRF → TWAN | Updated QoS rules |
| CCR-Terminate | TWAN → PCRF | Trusted non-3GPP detach |
| RAR | PCRF → TWAN | PCRF-initiated rule push |

---

## Diameter Messages — Rx Interface

The Rx interface is how Application Functions (primarily P-CSCF) describe media sessions to the PCRF. This is the IMS ↔ EPC policy bridge.

| Message | Direction | Trigger |
|---|---|---|
| AAR (AA-Request) | AF → PCRF | Session setup or modification: deliver media component descriptor (SDP-derived) |
| AAA (AA-Answer) | PCRF → AF | Authorization result; may include IP flow information (gate status, QoS) |
| RAR (Re-Auth-Request) | PCRF → AF | PCRF notifies AF of bearer change (e.g. access type change, congestion) |
| RAA (Re-Auth-Answer) | AF → PCRF | Acknowledges notification |
| STR (Session-Termination-Request) | AF → PCRF | Session ends; AF instructs PCRF to remove Rx session and associated PCC rules |
| STA (Session-Termination-Answer) | PCRF → AF | Acknowledges; PCRF removes rules from PGW via Gx RAR |
| ASR (Abort-Session-Request) | PCRF → AF | PCRF initiates session teardown (e.g. bearer lost) |
| ASA (Abort-Session-Answer) | AF → PCRF | Acknowledges |

### Rx Media Component Descriptor (AAR Payload)

The P-CSCF derives this from the SDP offer/answer negotiated between endpoints:

| Field | Meaning | Maps to PCC |
|---|---|---|
| Media-Component-Number | Identifies this media stream | SDF rule group |
| Media-Type | audio / video / data | QCI selection |
| Max-Requested-Bandwidth-UL/DL | Codec bandwidth | GBR UL/DL in PCC rule |
| Flow-Description | IP 5-tuple (src/dst IP, port, protocol) | SDF filter in PCC rule |
| Flow-Status | ENABLED / DISABLED / REMOVED | Gate open/close |
| AF-Charging-Identifier | P-CSCF charging ID for this session | Linked to IMS CDR |
| Media-Sub-Component | Per-flow component (RTP + RTCP separately) | Separate TFT entries |

---

## Core PCRF Function: PCC Rule Generation

When the PCRF correlates an Rx session with a Gx session, it synthesizes PCC rules:

```mermaid
flowchart TD
    RxAAR["Rx AAR received\n(media component descriptor)"] --> CorrelateIP["Correlate with Gx session\n(match UE IP address from AAR\nto active IP-CAN session)"]
    CorrelateIP --> SPRlookup["Fetch subscriber policy from SPR\n(allowed QoS, charging keys, service profile)"]
    SPRlookup --> QoSmapping["Map media type → QCI\n(audio → QCI=1, video → QCI=2,\nIMS signaling → QCI=5)"]
    QoSmapping --> TFTgen["Build SDF filters from\nFlow-Description (5-tuples)\n+ RTCP companion flows"]
    TFTgen --> Charging["Add charging parameters:\nrating group, reporting level,\nonline/offline flags"]
    Charging --> PCCrule["PCC Rule =\nSDF filter + QoS + gate + charging"]
    PCCrule --> GxRAR["Send Gx RAR to PGW\n(install PCC rule)"]
    GxRAR --> GxcRAR["If PMIP mode:\nSend Gxc RAR to SGW\n(QoS rules only — no charging)"]
    GxRAR --> RxAAA["Send Rx AAA\n(authorize AF session)"]
```

### QCI Mapping (Voice/Video/Signaling)

| Media Type | QCI | Characteristic |
|---|---|---|
| VoLTE RTP (voice) | 1 | GBR, conversational, 100ms delay budget |
| Video call RTP | 2 | GBR, conversational, 150ms delay budget |
| IMS signaling (SIP) | 5 | Non-GBR, IMS signaling, default bearer |
| Video streaming | 4 | GBR, real-time video |
| Default internet | 9 | Non-GBR, best-effort |

---

## Roaming Architecture

### Non-Roaming / Home-Routed Roaming

```mermaid
graph LR
    PCSCF["P-CSCF"] -->|Rx| HPCRF["H-PCRF\n(HPLMN)"]
    PGW["PGW (PCEF)\n(HPLMN)"] -->|Gx| HPCRF
```

Single PCRF. No S9. P-CSCF and PGW both in HPLMN (or P-CSCF in VPLMN with Rx to H-PCRF).

### Local Breakout Roaming

```mermaid
graph LR
    VPCSCF["V-P-CSCF\n(VPLMN)"] -->|Rx| VPCRF["V-PCRF\n(VPLMN)"]
    VPGW["V-PGW (PCEF)\n(VPLMN)"] -->|Gx| VPCRF
    VPCRF -->|S9| HPCRF["H-PCRF\n(HPLMN)"]
    HPCSCF["H-P-CSCF\n(HPLMN)"] -->|Rx| HPCRF
```

In local breakout, the V-PCRF is the enforcement coordinator:
- V-PCRF terminates Gx from V-PGW and Rx from visited AF
- V-PCRF requests policy authorization from H-PCRF via S9
- H-PCRF applies home subscriber policy; V-PCRF applies visited network constraints
- H-PCRF can also receive Rx from home AF and push rules toward V-PCRF via S9

### S9 Diameter Messages

| Message | Direction | Purpose |
|---|---|---|
| CCR-Initial (S9 session) | V-PCRF → H-PCRF | Establish S9 session; carry IP-CAN session info |
| CCA-Initial | H-PCRF → V-PCRF | Home policy returned (PCC rules / QoS rules) |
| CCR-Update | V-PCRF → H-PCRF | Notify home of Rx event, bearer change, location change |
| CCA-Update | H-PCRF → V-PCRF | Updated home policy |
| CCR-Terminate | V-PCRF → H-PCRF | Session ends |
| RAR | H-PCRF → V-PCRF | H-PCRF pushes new rules (Rx-triggered from home AF) |
| RAA | V-PCRF → H-PCRF | Acknowledges |

---

## VoLTE Bearer Trigger — End-to-End Flow

The PCRF's most important procedure: translating a SIP call setup into a GBR dedicated bearer.

```mermaid
sequenceDiagram
    participant UE
    participant PCSCF as P-CSCF (AF)
    participant PCRF
    participant PGW as PGW (PCEF)
    participant SGW
    participant MME

    Note over UE,PCSCF: SIP INVITE — SDP offer (codec, ports)
    UE->>PCSCF: SIP INVITE (SDP: audio RTP 5-tuple, G.711, 64kbps)
    PCSCF->>PCRF: Rx AAR (media-component: audio, MBR=64kbps, 5-tuple, gate=DISABLED)
    PCRF->>PCRF: Correlate UE IP → active Gx session\nMap audio → QCI=1, GBR=64kbps\nBuild SDF filter from 5-tuple\nGate DISABLED (preconditions)
    PCRF-->>PCSCF: Rx AAA (authorized, gate=DISABLED)
    Note over UE,PCSCF: SIP 183 / PRACK / UPDATE preconditions...
    Note over PCSCF: SDP answer received — final 5-tuple known
    PCSCF->>PCRF: Rx AAR update (gate=ENABLED, final SDP 5-tuple)
    PCRF->>PGW: Gx RAR (PCC rule: QCI=1, GBR=64k UL+DL, TFT=RTP 5-tuple, gate=OPEN)
    PGW-->>PCRF: Gx RAA (accepted)
    PGW->>SGW: Create Bearer Request (EPS Bearer QoS=QCI1/GBR64k, TFT, Linked Bearer ID)
    SGW->>MME: Create Bearer Request
    MME->>UE: NAS Activate Dedicated Bearer Context Request
    UE-->>MME: NAS Activate Dedicated Bearer Context Accept
    MME-->>SGW: Create Bearer Response (EPS Bearer ID, eNB TEID)
    SGW-->>PGW: Create Bearer Response (SGW TEID)
    PGW->>PCRF: Gx CCR-Update (bearer binding complete, EPS Bearer ID)
    PCRF-->>PGW: Gx CCA-Update
    PCRF-->>PCSCF: Rx RAR (resources reserved)
    PCSCF-->>PCRF: Rx RAA
    Note over UE,PCSCF: SIP 200 OK / ACK — RTP flows on QCI=1 bearer
```

### Gate Control (Preconditions)

The PCRF implements two-phase gate control for IMS preconditions:

1. **Phase 1 (gate DISABLED):** Rx AAR arrives with gate closed — PCRF creates PCC rule but with gate=CLOSED. Bearer is established but traffic is blocked.
2. **Phase 2 (gate ENABLED):** After SDP answer exchange confirms media path — Rx AAR update with gate=ENABLED. PCRF sends Gx RAR with gate=OPEN. PGW opens gate; RTP flows.

This prevents media from flowing before both endpoints have confirmed their RTP parameters.

---

## PCRF Session Correlation

The PCRF must correlate Rx sessions (from P-CSCF) with Gx sessions (from PGW) to install rules on the correct IP-CAN session.

```mermaid
flowchart TD
    RxAAR["Rx AAR\n(UE IP address,\nAF session ID)"] --> lookup["Look up active Gx sessions\nby UE IP address"]
    lookup --> found{Match found?}
    found -->|Yes| correlate["Bind Rx session to Gx session\nInstall PCC rules on that PCEF"]
    found -->|No| store["Store Rx session\nWait for Gx CCR-Initial\n(UE may not have PDN yet)"]
    GxCCR["Gx CCR-Initial\n(UE IP address)"] --> lookup2["Look up stored Rx sessions\nby UE IP address"]
    lookup2 --> found2{Pending Rx?}
    found2 -->|Yes| correlate2["Bind and install rules\nincluding any pending Rx-derived rules"]
    found2 -->|No| newSession["New Gx session\nApply default policy only"]
    correlate --> install["Gx RAR: install PCC rules\nRx AAA: authorize AF session"]
    correlate2 --> install
```

**Correlation key:** UE's IP address (from Rx AAR Framed-IP-Address and Gx CCR UE-IP-Address AVPs). In IPv6 the prefix is used.

---

## PCC Rule Structure

A PCC rule is the atomic unit of PCRF output. Each rule specifies enforcement for one service data flow:

```mermaid
erDiagram
    PCC_RULE {
        string rule_name
        int precedence
        string rule_status
    }
    SDF_FILTER {
        string src_IP_prefix
        string dst_IP_prefix
        int src_port_range
        int dst_port_range
        int protocol_ID
        string direction
    }
    QoS_PARAMETERS {
        int QCI
        int ARP_priority
        bool ARP_preemption_capability
        bool ARP_preemption_vulnerability
        int GBR_UL_kbps
        int GBR_DL_kbps
        int MBR_UL_kbps
        int MBR_DL_kbps
    }
    CHARGING_PARAMETERS {
        string rating_group
        string service_ID
        string reporting_level
        string metering_method
        bool online_charging
        bool offline_charging
    }
    GATE_STATUS {
        bool UL_gate_open
        bool DL_gate_open
    }
    PCC_RULE ||--o{ SDF_FILTER : "has"
    PCC_RULE ||--|| QoS_PARAMETERS : "has"
    PCC_RULE ||--|| CHARGING_PARAMETERS : "has"
    PCC_RULE ||--|| GATE_STATUS : "has"
```

---

## PCRF Session State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle : PCRF started
    Idle --> GxActive : CCR-Initial from PGW\n(IP-CAN session established)
    GxActive --> GxActive : CCR-Update (events)\nRAR/RAA (rule push)\nRx AAR binding
    GxActive --> RxBound : Rx AAR correlated\n(AF session bound to Gx session)
    RxBound --> RxBound : Rx AAR updates (gate, SDP change)\nCCR-Updates from PCEF\nRAR/RAA for rule push
    RxBound --> GxActive : Rx STR (AF session ends)\nPCC rules removed from PGW
    GxActive --> Idle : CCR-Terminate (PDN disconnected)\nor ASR sent (PCRF-initiated teardown)
    RxBound --> Idle : CCR-Terminate\n(Rx STR + Gx terminate)
```

One PCRF session per PDN connection (Gx). Multiple Rx sessions may bind to a single Gx session (e.g. multiple simultaneous VoLTE calls on the same PDN connection).

---

## PCRF in Non-3GPP Access

The PCRF's role expands across all access types. The key difference is **which BBERF** it talks to on Gxa/Gxc:

| Access Type | BBERF Interface | PCC Enforcement Split |
|---|---|---|
| 3GPP LTE (GTP S5/S8) | None — PGW is sole enforcement point | PCRF → Gx → PGW only |
| 3GPP LTE (PMIP S5/S8) | Gxc → SGW (BBERF) | PCRF → Gxc → SGW (QoS rules) + Gx → PGW (PCC rules, B.2 after bearer up) |
| Trusted non-3GPP (S2a) | Gxa → TWAN (BBERF) | PCRF → Gxa → TWAN (QoS rules) + Gx → PGW (PCC rules) |
| Untrusted non-3GPP (S2b) | Gxb → ePDG (partial, Rel-15) | PCRF → Gx → PGW (primary path); Gxb toward ePDG not fully standardized |

**Key constraint:** When a UE has simultaneous 3GPP + non-3GPP connections (MAPCON), the PCRF maintains separate Gx sessions per PDN connection per access. PCC rules are delivered independently to each PCEF.

---

## Failure and Overload Behavior

```mermaid
stateDiagram-v2
    [*] --> Normal
    Normal --> GxDown_at_PGW : PGW loses Gx connection
    GxDown_at_PGW --> Normal : Reconnected — CCR re-sent
    GxDown_at_PGW --> GxDown_at_PGW : PGW applies local CCF (default rules)\nExisting bearers may be kept or terminated per operator policy
    Normal --> RxDown_at_PCSCF : P-CSCF loses Rx connection
    RxDown_at_PCSCF --> Normal : Reconnected
    RxDown_at_PCSCF --> RxDown_at_PCSCF : New VoLTE calls cannot get GBR bearer\nPCRF cannot authorize media QoS\nCalls may fall back to default bearer
    Normal --> S9Down : S9 to H/V-PCRF fails (roaming)
    S9Down --> Normal : Reconnected
    S9Down --> S9Down : V-PCRF may apply local breakout defaults\nH-PCRF cannot enforce home policy\nCalls continue with best-effort QoS
    Normal --> Overloaded : High Diameter request rate
    Overloaded --> Normal : Load decreases
    Overloaded --> Overloaded : Return DIAMETER_TOO_BUSY\nApply Diameter overload control (RFC 7683)
```

---

## Configuration Parameters

| Parameter | Description |
|---|---|
| Gx realm / peer list | PGW addresses/realms for Gx sessions |
| Gxc realm | SGW addresses/realms for BBERF sessions (PMIP mode) |
| Gxa realm | TWAN addresses/realms for trusted non-3GPP sessions |
| Rx realm / AF whitelist | Authorized AF identities allowed to send AAR |
| SPR address | Subscriber Profile Repository endpoint (or co-located) |
| S9 peer (H-PCRF or V-PCRF) | Roaming peer address |
| Default PCC rules per APN | Rules applied at IP-CAN session establishment before Rx binding |
| QCI mapping table | Media-type → QCI mapping (configurable per operator) |
| GBR bandwidth allocation | Max GBR per session / per subscriber |
| Preemption policy | ARP preemption capability/vulnerability defaults |
| Gate control mode | Precondition-based (2-phase) vs immediate gate open |
| Rx session idle timeout | How long to wait for matching Gx session before discarding pending Rx |
| Congestion policy (Np) | Actions on RCAF congestion report: throttle / block / deprioritize |

---

## Key Architectural Properties

| Property | Details |
|---|---|
| **Policy decision point only** | PCRF makes decisions but does not enforce — it delegates to PCEF (PGW) and BBERF (SGW/TWAN) |
| **Session correlation is critical** | Rx → Gx binding by UE IP address is the mechanism that connects IMS call setup to EPS bearer modification |
| **Gx session per PDN connection** | Each UE PDN connection has its own Gx session; one UE may have multiple parallel Gx sessions |
| **Rx session per AF dialog** | One Rx session per SIP dialog leg at P-CSCF; one Rx session may add multiple PCC rules |
| **No user-plane involvement** | PCRF is purely Diameter signaling; it never sees RTP packets |
| **Cross-access visibility** | In MAPCON, PCRF correlates multiple Gx sessions for same UE on different access types simultaneously |
| **Home PLMN authority** | H-PCRF always has final authority on subscriber QoS policy; V-PCRF can only further restrict, not expand |

---

## Cross-References

| Topic | Page |
|---|---|
| PCRF base entity | [entities/PCRF.md](PCRF.md) |
| PGW (PCEF, Gx consumer) | [entities/PGW.md](PGW.md) |
| PGW deep-dive | [entities/PGW-deepdive.md](PGW-deepdive.md) |
| SGW (BBERF, Gxc) | [entities/SGW.md](SGW.md) |
| SGW deep-dive | [entities/SGW-deepdive.md](SGW-deepdive.md) |
| P-CSCF (AF, Rx) | [entities/P-CSCF.md](P-CSCF.md) |
| EPS bearer model | [concepts/EPS-bearer.md](../concepts/EPS-bearer.md) |
| Dedicated bearer lifecycle | [procedures/dedicated-bearer.md](../procedures/dedicated-bearer.md) |
| IMS QoS bearer | [procedures/IMS-QoS-bearer.md](../procedures/IMS-QoS-bearer.md) |
| VoLTE MO call | [procedures/VoLTE-MO-call.md](../procedures/VoLTE-MO-call.md) |
| VoLTE MT call | [procedures/VoLTE-MT-call.md](../procedures/VoLTE-MT-call.md) |
| PMIP S5/S8 procedures | [procedures/PMIP-S5S8-procedures.md](../procedures/PMIP-S5S8-procedures.md) |
| Trusted non-3GPP attach | [procedures/trusted-non3GPP-attach.md](../procedures/trusted-non3GPP-attach.md) |
| Non-3GPP architecture | [concepts/non-3GPP-access-architecture.md](../concepts/non-3GPP-access-architecture.md) |
| EPC reference points | [interfaces/reference-points.md](../interfaces/reference-points.md) |
| IMS reference points | [interfaces/IMS-reference-points.md](../interfaces/IMS-reference-points.md) |
