---
title: "PCC Rule Types — PCC, QoS, ADC Rules and Associated Policy Information"
type: concept
tags: [PCC, PCRF, PCEF, BBERF, TDF, Gx, Gxx, Sd, QoS, ADC, usage-monitoring, APN-AMBR, spending-limits, traffic-steering, NBIFOM, TS23203]
sources: [ts23203.md]
updated: 2026-04-19
---

# PCC Rule Types — PCC, QoS, ADC Rules and Associated Policy Information

TS 23.203 §6.3–§6.12 defines the full information models for the three rule types (PCC rules, QoS rules, ADC rules) and the session-scoped policy information that accompanies them. This page serves as the normative reference for all PCC rule schemas.

---

## 1. PCC Rule (§6.3)

A PCC rule is the unit of policy and charging enforcement at the [PCEF](../entities/PCEF.md). The [PCRF](../entities/PCRF.md) provisions PCC rules to the PCEF over the [Gx reference point](../concepts/PCC-architecture.md).

### 1.1 Dynamic vs Predefined PCC Rules

| Type | Definition | How activated |
|---|---|---|
| **Dynamic** | Constructed by the PCRF and sent in full via Gx | PCRF sends full rule information in Gx CCR/RAR |
| **Predefined** | Pre-configured directly in PCEF; known to the PCRF by name | PCRF sends the rule identifier over Gx; PCEF activates the local rule |

A predefined PCC rule can only be activated if there is no UE-provided traffic mapping information related to the IP-CAN bearer. A predefined rule can be activated for multiple IP-CAN bearers in multiple IP-CAN sessions simultaneously (within one access point scope). A predefined rule is bound to one and only one IP-CAN bearer per IP-CAN session.

### 1.2 Active PCC Rule Effects

When a PCC rule is active at the PCEF:
- The SDF template is used for service data flow detection (downlink and uplink)
- Usage data for the SDF is recorded (see [PCEF measurement](../entities/PCEF.md#measurement))
- Policies associated with the rule are invoked (gating, QoS, charging)
- If the rule uses an application detection filter: start/stop of application traffic is reported to the PCRF if requested

### 1.3 Deferred Activation/Deactivation

A PCC rule may carry:
- A **deferred activation time** — rule is inactive until this time
- A **deferred deactivation time** — rule becomes inactive at this time (even if still active)
- Both — rule is inactive before activation time, active between the two times, inactive after deactivation time

Rules with deferred times can only be used for bearers **without** traffic mapping information (i.e., not bearer-bound with UE TFT). Deferred modification does not apply to changes to QoS or SDF filter information.

### 1.4 PCC Rule Field Table (Table 6.3)

| Field | Description |
|---|---|
| **Rule identifier** | Uniquely identifies the PCC rule within the IP-CAN session |
| **SDF template** | Set of SDF filters (5-tuple, IPSec SPI, IPv6 flow label, ToS+mask) **or** Application identifier for DPI-based matching |
| **Precedence** | Order of SDF filter evaluation; lower value = higher priority |
| **Mute for notification** | Whether start/stop application traffic notifications shall be muted to the PCRF |
| **Charging key** | Rating group; used by OCS/OFCS to determine tariff |
| **Service identifier** | Identifies one or more services for charging; most granular service level identifier |
| **Sponsor Identifier** | Identifies the 3rd-party sponsor for sponsored data connectivity (from AF via Rx) |
| **Application Service Provider Identifier** | Identifies the ASP delivering the sponsored service |
| **Charging method** | Online, offline, or neither (no end-user charging) |
| **Measurement method** | Volume, duration, volume+duration, or event |
| **AF Record Information** | Information from the AF needed for charging record correlation |
| **Service identifier level reporting** | Whether separate usage reports shall be generated per service identifier |
| **Gate status** | Open (packets pass) or Closed (packets dropped) per SDF direction (UL/DL) |
| **QCI** | [QoS Class Identifier](QCI-characteristics.md) — determines packet forwarding treatment |
| **UL maximum bitrate** | Maximum UL bitrate authorized for the SDF |
| **DL maximum bitrate** | Maximum DL bitrate authorized for the SDF |
| **UL guaranteed bitrate** | Guaranteed UL bitrate (GBR rules only) |
| **DL guaranteed bitrate** | Guaranteed DL bitrate (GBR rules only) |
| **UL sharing indication** | Groups UL GBR rules sharing the same resource (max GBR = highest in group) |
| **DL sharing indication** | Groups DL GBR rules sharing the same resource |
| **Redirect** | Whether detected SDF traffic shall be redirected to a controlled address |
| **Redirect Destination** | Target address for redirected traffic |
| **ARP** | Allocation and Retention Priority — priority level (1–15), pre-emption capability, pre-emption vulnerability |
| **Bind to Default Bearer** | Forces this rule to be bound to the default bearer |
| **PS to CS session continuity** | Marks this rule as a vSRVCC candidate for PCEF to apply CS-continuity treatment (TS 23.216) |
| **User Location Report** | PCEF shall include user location info when reporting for this rule |
| **UE Timezone Report** | PCEF shall include UE timezone when reporting for this rule |
| **Monitoring key** | Groups PCC rules for usage monitoring; multiple rules may share the same key |
| **Exclusion from session monitoring** | This SDF shall be excluded from the IP-CAN session-level usage monitoring |
| **Traffic steering policy identifier(s)** | Reference to pre-configured traffic steering policy at PCEF/(S)Gi-LAN |
| **Allowed Access Type (NBIFOM)** | IP-CAN type(s) to be used for this flow (3GPP and/or WLAN) in NBIFOM scenarios |
| **Routing Rule ID (NBIFOM)** | Reference to the NBIFOM routing rule (§6.12) for this flow |
| **UL Maximum Packet Loss Rate** | Maximum tolerated UL packet loss rate |
| **DL Maximum Packet Loss Rate** | Maximum tolerated DL packet loss rate |

---

## 2. QoS Rule (§6.5)

A QoS rule enables service data flow detection and QoS control at the [BBERF](../entities/SGW.md) (SGW in PMIP mode). QoS rules are **derived from PCC rules** by the PCRF and sent to the BBERF via Gxx.

> Key invariant: Every active PCC rule in the PCEF must have a corresponding active QoS rule in the BBERF. The PCRF ensures this by generating a corresponding QoS rule for each PCC rule when a BBERF is present.

### 2.1 QoS Rule Field Table (Table 6.5)

| Field | Category | PCRF may modify on active rule? |
|---|---|---|
| Rule identifier | Mandatory | No |
| Precedence | Mandatory | Yes |
| Service data flow template | Mandatory | Yes |
| QoS class identifier (QCI) | Mandatory | Yes |
| UL maximum bitrate | Conditional (if in PCC rule) | Yes |
| DL maximum bitrate | Conditional | Yes |
| UL guaranteed bitrate | Conditional | Yes |
| DL guaranteed bitrate | Conditional | Yes |
| UL sharing indication | — | No |
| DL sharing indication | — | No |
| ARP | Conditional (if in PCC rule) | Yes |
| PS to CS session continuity | Conditional | No |
| User Location Report | Conditional | Yes |
| UE Timezone Report | Conditional | Yes |

> Note: The BBERF only supports SDF templates consisting of a set of SDF filters (not Application identifier).

### 2.2 QoS Rule Operations

- **Activation**: PCRF sends full QoS rule info to BBERF via Gxx
- **Modification**: PCRF may modify any active QoS rule field marked Yes above
- **Deactivation**: PCRF deactivates via Gxx; all active QoS rules on a bearer are automatically deactivated at IP-CAN bearer termination

---

## 3. ADC Rule (§6.8)

An ADC (Application Detection and Control) rule identifies an application and defines enforcement actions for that application's traffic. ADC rules operate over:
- **Sd** (PCRF ↔ TDF) — solicited application reporting and enforcement at TDF
- **St** (PCRF ↔ TSSF) — traffic steering control at the TSSF

### 3.1 ADC Rule Types

| Type | Description |
|---|---|
| **Predefined** | Pre-configured at TDF/TSSF; constant; not modifiable by PCRF |
| **Dynamic** | Some parameters can be provided and modified by PCRF (Table 6.8) |

At most one ADC rule can be active per application identifier value at any time.

**Precedence**: When multiple ADC rules match the same application traffic, the rule with highest precedence (lowest value) determines enforcement, reporting, monitoring, and charging. Dynamic ADC rules take precedence over predefined rules with the same precedence value.

### 3.2 ADC Rule Field Table (Table 6.8)

| Field | Category | PCRF modifiable | Reference point |
|---|---|---|---|
| ADC Rule identifier | Mandatory | No | Sd, St |
| Application detection (application name) | — | — | Sd, St |
| Precedence | Optional | Yes | Sd, St |
| **Application identifier** | Conditional | No | Sd, St |
| **Service data flow filter list** | Conditional | No | Sd, St |
| Mute for notification | Optional | No | Sd |
| Monitoring key | Optional | Yes | Sd |
| Exclusion from session monitoring | Optional | Yes | Sd |
| Gate status | Optional | Yes | Sd |
| UL maximum bitrate | Optional | Yes | Sd |
| DL maximum bitrate | Optional | Yes | Sd |
| Redirect | Optional | Yes | Sd |
| Redirect Destination | Conditional | Yes | Sd |
| DSCP value | Optional | Yes | Sd |
| Charging key | Optional | Yes | Sd |
| Service identifier | Optional | Yes | Sd |
| Sponsor Identifier | Conditional | Yes | — |
| Application Service Provider Identifier | Conditional | Yes | Sd |
| Charging method | Conditional | No | Sd |
| Measurement method | Optional | Yes | Sd |
| Service identifier level reporting | Optional | Yes | Sd |
| **Traffic steering policy identifier(s)** | Optional | Yes | Sd, St |

> Notes:
> - Either **Application identifier** (DPI-based) or **Service data flow filter list** must be present; not both simultaneously.
> - DSCP value: TDF marks downlink packets of detected application traffic with this DSCP value (used for (S)Gi-LAN steering).
> - Application identifier can include encrypted traffic detection configuration (Annex X).

### 3.3 ADC Rule Operations (§6.8.2)

- **Activation over Sd**: PCRF sends ADC rule identifier to TDF; may include usage monitoring and enforcement control for dynamic ADC rules
- **Active ADC rule effects**: application matching traffic is detected; start/stop reported to PCRF (if not muted and requested); monitoring and enforcement applied
- **Deactivation**: PCRF deactivates at any time; TDF session termination deactivates all associated ADC rules
- **Deferred activation/deactivation**: same mechanism as for PCC rules (single deferred activation time or deactivation time or both)

---

## 4. IP-CAN Session and Bearer Policy Information (§6.4)

Session-scoped policy attributes provisioned by the PCRF to the PCEF (and BBERF if applicable), separate from individual PCC/QoS rules.

### Table 6.4: IP-CAN Bearer and Session Related Policy Information

| Attribute | Description | Scope | PCRF modifiable |
|---|---|---|---|
| **Charging information** | OFCS and/or OCS addresses | IP-CAN session | No (overrides predefined) |
| **Default charging method** | Charging method for PCC rules where method is omitted | IP-CAN session | No |
| **Event triggers** | Events that cause PCEF to re-request PCC rules | IP-CAN session | Yes |
| **Authorized QoS per bearer** | Authorized QoS for a specific IP-CAN bearer (QCI, GBR, MBR) | IP-CAN bearer | Yes |
| **Authorized MBR per QCI** | Authorized MBR for all bearers of a given QCI (network-initiated mode) | IP-CAN session | Yes |
| **Revalidation time limit** | Time within which PCEF shall request a fresh set of PCC rules | IP-CAN session | Yes |
| **PRA Identifier(s)** | Presence Reporting Area(s) to monitor for UE entry/exit | IP-CAN session | Yes |
| **List(s) of PRA elements** | Elements comprising each UE-dedicated PRA | IP-CAN session | Yes |
| **Default NBIFOM access** | IP-CAN type for traffic not matching any routing rule | IP-CAN session | Yes (only at access addition) |

> Note on Charging information: Received at initial interaction with PCEF; supersedes Primary OFCS/OCS and Secondary OFCS/OCS address in the charging characteristics profile.

> Note on Revalidation time limit: PCEF triggers a PCC rules re-request to the PCRF when this timer expires. Useful for policy refresh without event trigger.

---

## 5. TDF Session Policy Information (§6.4a)

Provisioned by the PCRF to the TDF via Sd at TDF session establishment.

### Table 6.4a: TDF Session Related Policy Information

| Attribute | Description | PCRF modifiable |
|---|---|---|
| Charging Characteristics | Controls TDF online/offline charging behaviour | No |
| Charging information | OFCS and/or OCS addresses | No |
| Default charging method | Default method for ADC rules with omitted charging method | No |
| Event trigger | Events causing TDF to re-request ADC rules | Yes |
| Maximum downlink bit rate | Maximum DL bit rate per TDF session | Yes |
| Maximum uplink bit rate | Maximum UL bit rate per TDF session | Yes |
| ADC Revalidation time limit | Time within which TDF shall trigger an ADC rules re-request | Yes |

> Note: PCRF should set Maximum DL bit rate = DL APN-AMBR to avoid DL packet discard at PCEF when TDF performs charging.

---

## 6. APN-AMBR Control (§6.4b)

APN-AMBR limits the aggregate non-GBR traffic for all IP-CAN sessions of a UE to the same APN.

### Table 6.4b-1: APN Related Policy Information

| Attribute | Description | Scope |
|---|---|---|
| **Authorized APN-AMBR** | APN-AMBR for all IP-CAN sessions of the UE to the same APN | All IP-CAN sessions for same UE + APN |
| **Subsequent APN-AMBR** | APN-AMBR to apply after the APN-AMBR change time | Same scope |
| **APN-AMBR change time** | When the PCEF shall apply the Subsequent APN-AMBR | Same scope |

Rules:
- PCRF may provide up to four instances of Subsequent APN-AMBR (with different change times; values must differ and not be too close in time)
- Authorized APN-AMBR may be conditional (per RAT type and/or IP-CAN type); PCEF applies conditional APN-AMBR only when conditions match; otherwise applies the unconditional APN-AMBR
- Conditional APN-AMBR is **not** applied for NBIFOM PDN connections
- PCEF communicates changed APN-AMBR to the UE

---

## 7. Usage Monitoring Control (§6.6)

Usage monitoring enables the PCRF to track traffic volume and/or time at the PCEF or TDF, keyed by a monitoring key.

### Table 6.6: Usage Monitoring Control Information

| Information | Category | Description |
|---|---|---|
| **Monitoring key** | Mandatory | Groups services sharing a common usage allowance; any number of PCC/ADC rules may share the same key |
| **Volume threshold** | Optional | Volume after which PCEF/TDF shall report to PCRF (triggers "Usage threshold reached") |
| **Time threshold** | Optional | Resource time after which PCEF/TDF shall report to PCRF |
| **Monitoring time** | Optional | Time at which PCEF/TDF shall re-apply the volume/time threshold |
| **Subsequent Volume threshold** | Optional, Conditional | Volume threshold for the period **after** the Monitoring time |
| **Subsequent Time threshold** | Optional, Conditional | Time threshold for the period after the Monitoring time |
| **Inactivity Detection Time** | Optional, Conditional | Stops time measurement if no packets received for this duration |

> Note: Subsequent thresholds are only applicable when Monitoring time is provided. Inactivity Detection Time is only applicable for Time threshold.

### 7.1 Usage Monitoring Operations (§6.6.2)

**Conditions for monitoring to be active:**

| Level | PCEF active condition | TDF active condition |
|---|---|---|
| IP-CAN session level | Active IP-CAN session + volume/time threshold provided for the session | Active TDF session + volume/time threshold provided |
| Monitoring key level | ≥1 PCC rule activated for the IP-CAN session associated with that monitoring key | ≥1 ADC rule activated at TDF associated with that monitoring key |

> Important: The PCRF should not monitor the same traffic via both PCC rules and ADC rules simultaneously — this avoids double-counting.

---

## 8. Policy Decisions Based on Spending Limits (§6.9)

The PCRF may make PCC decisions based on policy counter status maintained in the OCS (via Sy):

1. PCRF sends an **Initial Spending Limit Report Request** to OCS at IP-CAN session establishment
2. OCS returns current status of requested policy counters; may also return pending statuses with future activation times
3. PCRF subscribes to **spending limit reporting** — OCS notifies PCRF on threshold crossings (e.g. "daily spending limit of $2 reached")
4. PCRF uses counter status as input to policy decisions:
   - Change QoS (e.g. downgrade APN-AMBR when budget nearly exhausted)
   - Modify PCC/QoS/ADC rules (apply gating, change charging conditions)
5. At IP-CAN session termination: PCRF sends **Final Spending Limit Report Request** to OCS

Counter identifiers and charging keys may have 1:1 or 1:many relationships — services sharing the same charging key may be subject to different policy counters.

---

## 9. Traffic Steering Control Information (§6.11)

Traffic steering allows the PCRF to route specific application or service data flows through (S)Gi-LAN service functions.

### Table 6.11: Traffic Steering Control Information

| Component | In ADC rule (Sd/St) | In PCC rule (Gx) |
|---|---|---|
| Rule Name | ADC Rule Identifier | Rule identifier |
| Description of Traffic | Application Identifier or SDF filter list | Service Data Flow Template |
| Traffic steering policy identifier(s) | Traffic steering policy identifier(s) | Traffic steering policy identifier(s) |
| Precedence | Precedence | Precedence |

The **Traffic Steering Policy Identifier** is a reference to a pre-configured steering policy at the PCEF/TDF/TSSF. The policy content (service function chain, routing tags) is locally defined — the PCRF only sends the identifier.

### 9.1 Interfaces

| Interface | Direction | Purpose |
|---|---|---|
| Gx | PCRF → PCEF | Traffic steering identifiers embedded in PCC rules |
| Sd | PCRF → TDF | Traffic steering identifiers embedded in ADC rules |
| St | PCRF → TSSF | Traffic steering control information provisioned as ADC rules over St |

### 9.2 Operations over St (§6.11.2)

- **Provisioning**: PCRF provides traffic steering policy identifiers, service data flow description, and precedence to TSSF; TSSF uses APN to determine the PDN
- **Modification**: PCRF may update steering policy identifiers, precedence, or SDF description
- **Removal**: PCRF may remove traffic steering control information at any time

---

## 10. NBIFOM Routing Rule (§6.12)

NBIFOM (Network-Based IP Flow Mobility) routing rules associate IP-CAN types with specific IP flows when a UE has simultaneous 3GPP and WLAN connectivity.

### Table 6.12: NBIFOM Routing Rule Information

| Field | Category | PCEF modifiable |
|---|---|---|
| Routing Rule identifier | Mandatory | No (assigned by PCEF) |
| Routing information | — | — |
| Routing Rule Priority | Mandatory | Yes |
| Routing Filter | Mandatory | Yes |
| Routing Access Information | Mandatory | Yes |

- **Routing Filter**: same format as SDF filter (clause 6.2.2.2); may be a wildcard
- **Routing Access Information**: IP-CAN type (3GPP or WLAN) to use for matching flows
- **Routing Rule Priority**: order in which rules are evaluated; derived from UE Binding Update priority (TS 23.161)

### 10.1 NBIFOM Routing Rule Operations (§6.12.2)

At creation:
1. PCEF provides routing rule info to PCRF via Gx
2. PCRF checks if a PCC rule with matching SDF template exists; if so, updates the PCC rule's Allowed Access Type to the Routing Access Information
3. Otherwise, PCRF creates a new PCC rule with: same parameters as the matching rule, Allowed Access Type set to the Routing Access Information, and installs it in the PCEF

Modification:
- Routing Filter change → PCRF updates SDF filter or precedence of corresponding PCC rule
- Routing Access Information change → PCRF updates Allowed Access Type in PCC rule

Removal:
- If the PCC rule was created for this routing rule → PCRF removes the PCC rule
- Otherwise → PCRF removes only the Allowed Access Type from the PCC rule

---

## Rule Type Summary

| Feature | PCC Rule | QoS Rule | ADC Rule |
|---|---|---|---|
| Interface | Gx (PCEF) | Gxx (BBERF) | Sd/St (TDF/TSSF) |
| Generated by | PCRF | PCRF (derived from PCC rules) | PCRF / TDF (pre-configured) |
| Detection method | SDF filters or Application ID | SDF filters only | Application ID or SDF filters |
| Charging control | Yes (Gy/Gz at PCEF) | No | Yes (Gyn/Gzn at TDF) |
| Gate control | Yes | No | Yes |
| QoS enforcement | Yes | Yes | No (BW limiting only) |
| Redirect | Yes | No | Yes |
| Traffic steering | Yes | No | Yes |
| Usage monitoring | Yes (monitoring key) | No | Yes (monitoring key) |
| Deferred timing | Yes | No | Yes |

---

## Related Pages

- [PCC Architecture](PCC-architecture.md) — entity overview and reference points
- [PCRF](../entities/PCRF.md) — generates all rule types
- [PCEF](../entities/PCEF.md) — enforces PCC rules
- [QCI Characteristics](QCI-characteristics.md) — QCI/ARP reference
- [EPS Bearer](EPS-bearer.md) — bearer model and bearer binding
