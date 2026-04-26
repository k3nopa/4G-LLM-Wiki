---
title: "PCRF — Policy and Charging Rules Function"
type: entity
tags: [PCRF, EPC, policy, charging, Gx, Rx, Diameter, QoS, IMS, TS23203, V-PCRF, H-PCRF, BBERF, usage-monitoring, MPS]
sources: [ts_123401v150400p.pdf, ts23203.md]
updated: 2026-04-19
---

# PCRF — Policy and Charging Rules Function

**Spec reference:** 3GPP TS 23.401 §4.4.7; detailed in TS 23.203
**Plane:** Control plane (Diameter signaling only)
**Layer:** EPC core — policy brain

## Role

The PCRF is the policy decision point of the EPC. It decides QoS and charging rules for each UE's IP-CAN session and pushes those rules to the PGW (via Gx). It also receives session information from Application Functions (e.g. IMS P-CSCF via Rx) and translates media description into bearer-level QoS decisions — this is the key mechanism for VoLTE dedicated bearer establishment.

> PCRF functions are fully specified in **3GPP TS 23.203**.

## Variants

### Non-Roaming
Single **H-PCRF** per UE IP-CAN session:
- Terminates **Gx** (to PGW/PCEF)
- Terminates **Rx** (to AF — typically P-CSCF for IMS)

### Roaming — Home Routed
Same as non-roaming (PGW in HPLMN, only H-PCRF).

### Roaming — Local Breakout
Two PCRFs per IP-CAN session:
- **V-PCRF** (in VPLMN): terminates Gx (to V-PGW) + S9 + Rx (for visited AF)
- **H-PCRF** (in HPLMN): terminates Rx (for home services) + S9 (to V-PCRF)
- S9 carries policy/charging control information between H-PCRF and V-PCRF

## Interfaces

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| Gx | PGW (PCEF) | Diameter | Bidirectional | Push PCC rules; receive events/usage from PGW |
| Rx | AF (P-CSCF) | Diameter | Bidirectional | Receive session info from IMS; authorize QoS |
| S9 | V-PCRF ↔ H-PCRF | Diameter | Bidirectional | Roaming policy coordination |
| Sp | SPR | — | — | Subscriber Profile Repository (subscription data) |
| Np | RCAF | — | Receive | RAN User Plane Congestion Information |

## Key Workflow: VoLTE Dedicated Bearer

1. UE registers with IMS → P-CSCF established as proxy
2. UE sends SIP INVITE → P-CSCF sends **AAR** (AA-Request) on **Rx** to PCRF with SDP media info
3. PCRF correlates Rx session with active Gx session (matching UE IP)
4. PCRF creates PCC rule: QCI=1 (GBR conversational voice), GBR = codec bandwidth
5. PCRF sends **RAR** (Re-Auth-Request) on **Gx** to PGW with new PCC rule
6. PGW initiates dedicated bearer establishment toward MME → eNodeB → UE
7. VoLTE RTP media flows on the new GBR bearer

## PCC Rules
A PCC rule contains:
- Service Data Flow (SDF) filter (5-tuple or application ID)
- QoS parameters: QCI, ARP, GBR UL/DL, MBR UL/DL
- Charging parameters: rating group, reporting level, metering method
- Gating status (open/closed)
- Precedence

---

## Detailed Functional Description (TS 23.203 §6.2.1)

### PCRF Input Sources

The PCRF makes PCC decisions by combining information from multiple sources:

| Source | Interface | Key information |
|---|---|---|
| **PCEF** | Gx | Subscriber ID (IMSI/MSISDN), IMEI(SV), UE IPv4/IPv6, bearer attributes, IP-CAN type and RAT type, UE timezone, UE location (e.g. CGI, TAI, ECGI), PDN connection ID, APN, PLMN, 3GPP PS Data Off status |
| **BBERF** | Gxx | Gateway Control Session: bearer attributes, IP-CAN type, UE location |
| **SPR** | Sp | Allowed services, max QoS, charging info per APN, IMS Signalling Priority, MPS subscription, subscriber spending limit profile, 3GPP PS Data Off exempt services, ADC profile configuration |
| **AF** | Rx | SDP media description (codec, bandwidth, IP addresses, ports), media type, AF Application Identifier, ICSI, emergency indicator, sponsored data reference, background data transfer reference ID |
| **OCS** | Sy | Policy counter status (spending limits); PCRF subscribes and receives threshold-crossing notifications |
| **RCAF** | Np | RAN User Plane Congestion Information (RUCI): IMSI, eNB ID / ECGI / SAI, APN, congestion level |

### Session Binding Decision Logic

When the PCRF receives an Rx session request from an AF, it performs session binding:

1. **Find the matching IP-CAN session** — using UE IPv4 address and/or IPv6 network prefix, UE identity (if provided), and APN/PDN information
2. **Associate AF session → IP-CAN session** — links the AF-provided service info to the correct Gx session
3. **Generate PCC rules** — creates or modifies dynamic PCC rules based on:
   - SDP media info (IP filters, bandwidth)
   - Subscription data from SPR
   - Operator policies
4. **Push rules to PCEF** — via Gx RAR (Re-Auth-Request) or RAA (Re-Auth-Answer)

### V-PCRF and H-PCRF Roles

In **home-routed roaming**, the PCRF in the home network (H-PCRF) holds policy authority:
- H-PCRF receives Rx from home AF (e.g. home IMS P-CSCF)
- H-PCRF sends PCC decisions to V-PCRF via S9
- V-PCRF translates and enforces via Gx to V-PCEF and via Gxx to V-BBERF
- V-PCRF may apply additional visited operator QoS policies (within the constraints set by H-PCRF)

In **local-breakout roaming**, the V-PCRF has local control but H-PCRF supplies subscription-based policies via S9.

In **non-roaming**, a single PCRF (acting as H-PCRF) handles all interfaces.

### Multiple BBF Handling

During certain handover scenarios, multiple BBERF sessions may be linked to a single Gx session:

| BBF state | When it occurs |
|---|---|
| **Primary BBERF** | BBERF with same IP-CAN type as the PCEF; PCRF activates QoS rules on this one |
| **Non-primary BBERF** | BBERF from prior/target access; PCRF does not activate QoS rules but keeps context |

- PCRF tracks which QoS rules are activated on which BBERF
- When a non-primary BBERF becomes primary, PCRF activates the appropriate QoS rules
- The PCRF links BBFs to IP-CAN sessions using UE IPv4/IPv6, UE Identity, PDN Connection ID, and PDN ID

### Usage Monitoring Control

The PCRF may instruct the PCEF and/or TDF to monitor traffic volume or time:
- Assigns a **monitoring key** to one or more PCC rules (or to the entire IP-CAN session)
- PCEF/TDF track usage against a threshold set by the PCRF
- When threshold is reached, PCEF/TDF report to PCRF → PCRF makes a new policy decision
- Can apply to: individual SDFs, groups of SDFs (same monitoring key), or entire IP-CAN session
- PCRF can disable volume monitoring temporarily without deleting the PCC rule

### Subscriber Spending Limits (Sy)

The PCRF subscribes to **policy counter status** from the OCS via the Sy reference point:
- PCRF sends a Spending Limit Request to OCS, referencing the subscriber's spending limit profile
- OCS maintains counters (e.g. remaining data budget) and notifies PCRF when a threshold is crossed
- PCRF uses counter status to dynamically apply or remove PCC rules (e.g. block non-essential services when credit is low)

### Background Data Transfer (Nt via SCEF)

When a 3rd-party ASP requests future background data transfer via the SCEF:
1. SCEF sends a request to PCRF via Nt with desired time window and max aggregated bitrate
2. PCRF derives a transfer policy (time window + max aggregate bitrate + charging rate reference)
3. Transfer policy is stored in SPR with a **Reference ID**
4. When the AF initiates the actual session, it provides the Reference ID in the Rx AAR
5. PCRF retrieves the agreed policy and applies it to the IP-CAN session

### MPS — Multimedia Priority Service

When the Priority EPS Bearer Service is activated for a subscriber:
1. PCRF receives indication via Rx (from P-CSCF) or via subscription data (SPR)
2. PCRF elevates the ARP priority of the **default bearer** and the **IMS CN signalling bearer** (QCI 5)
3. This allows these bearers to pre-empt lower-priority bearers during congestion
4. ARP modification is sent to PCEF via Gx rule update

### IMS Emergency and RLOS

For IMS emergency calls:
- PCRF does not use SPR subscription data (local operator policies apply)
- S9 interface does not apply (emergency handled locally in VPLMN)
- PCRF assigns highest-priority ARP to emergency bearer
- After emergency call ends, PCRF starts an **inactivity timer**; if only default bearer QCI and IMS signalling QCI remain when timer expires, the IP-CAN session resources are released

RLOS (Restricted Local Operator Services): PCRF handles RLOS sessions similarly with local policies.

### 3GPP PS Data Off

A UE feature allowing the user to block mobile data except for exempt services:
- When activated, UE indicates to network → PCEF reports event trigger to PCRF
- PCRF closes gates or deactivates PCC rules for all non-exempt SDFs
- PCRF queries SPR for the subscriber's PS Data Off exempt services list
- Exempt services (IMS signalling, emergency, operator-defined) remain unaffected
- When deactivated, PCRF restores gates/rules for all blocked SDFs

---

## Related Pages
- [PGW](PGW.md) — enforcement point via Gx (hosts PCEF)
- [PCEF](PCEF.md) — enforcement function; receives PCC rules from PCRF
- [P-CSCF](P-CSCF.md) — IMS application function via Rx
- [PCC Architecture](../concepts/PCC-architecture.md) — full PCC entity/reference-point overview
- [QCI Characteristics](../concepts/QCI-characteristics.md) — QCI table and ARP parameters
- [SGW](SGW.md) — hosts BBERF in PMIP mode (Gxx)
- [Reference points](../interfaces/reference-points.md)
