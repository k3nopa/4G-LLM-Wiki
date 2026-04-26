---
title: "PCEF — Policy and Charging Enforcement Function"
type: entity
tags: [PCEF, PGW, PCC, Gx, SDF, QoS, charging, gating, EPC, TS23203]
sources: [ts23203.md]
updated: 2026-04-19
---

# PCEF — Policy and Charging Enforcement Function

**Spec reference:** 3GPP TS 23.203 §6.2.2
**Plane:** User plane (data path) + Control plane (Diameter Gx)
**Layer:** EPC core — PCC enforcement, resides in the [PGW](PGW.md)

## Role

The PCEF is the enforcement point of the [PCC architecture](../concepts/PCC-architecture.md). It resides in the Gateway (PGW for EPC) and executes every PCC decision received from the [PCRF](PCRF.md) via Gx. It is responsible for:

- **SDF detection** — classifying every IP packet into a service data flow
- **Gate control** — blocking or passing SDFs per PCRF instructions
- **QoS enforcement** — mapping PCC rules to EPS bearer QoS parameters
- **Charging** — volume/time/event measurement and reporting (Gy for online, Gz for offline)
- **Application detection** — detecting application traffic when TDF is not deployed (optional PCEF-based ADC)
- **Traffic steering** — marking or routing packets per steering policy
- **Bearer Binding Function (BBF)** — binds PCC rules to IP-CAN bearers when GTP is used on S5/S8

---

## SDF Detection

### SDF Template

An **SDF template** is a set of SDF filters that identifies a service data flow. An SDF filter can contain one or more of:

| Filter component | Description |
|---|---|
| IP 5-tuple | Source/destination IP address (or prefix) + port (range) + protocol |
| IPSec SPI | Security Parameter Index (for IPSec traffic) |
| IPv6 flow label | 20-bit flow label from IPv6 header |
| ToS/Traffic Class + mask | Differentiated Services Code Point matching |
| Application identifier | Identifies DPI-detected application (used instead of 5-tuple for ADC) |

Alternatively, a **wildcard SDF filter** can match all traffic not matched by more specific rules.

### Downlink SDF Detection

For downlink (network → UE) packets on a given IP-CAN session, the PCEF evaluates **all activated SDF templates** across all bearers, in order of **SDF precedence** (lower value = higher priority). The first matching template determines the treatment (PCC rule applied).

### Uplink SDF Detection

For uplink (UE → network) packets on a given bearer, the PCEF evaluates only the **SDF templates of PCC rules bound to that specific bearer**, in precedence order.

### Default Rule

If no SDF filter matches, the packet is handled by the default catch-all rule (typically bound to the default bearer).

---

## Measurement (Charging Data Collection)

The PCEF measures traffic for charging purposes. Measurement method per PCC rule:

| Method | What is measured |
|---|---|
| Volume | Octets (uplink + downlink, or separately) |
| Duration | Time the SDF gate is open and data is flowing |
| Volume + Duration | Both simultaneously |
| Event | Count of specific events (e.g. session starts) |
| No measurement | No charging data collected |

### Measurement Granularity

| Level | Keyed by | Use |
|---|---|---|
| Bearer + Charging Key | Per (EPS bearer, rating group) | Standard charging record |
| Bearer + Charging Key + Service Identifier | Per (EPS bearer, rating group, service ID) | Service-level reporting |
| Monitoring Key | Per monitoring key | Usage monitoring control (PCRF-driven) |

**Inactivity Detection Time (IDT):** The PCRF may set an IDT on time-based or volume+time PCC rules. If no packets are observed for the IDT duration, the PCEF stops the time measurement clock until traffic resumes. This prevents charging for idle sessions.

---

## QoS Enforcement

The PCEF enforces QoS per IP-CAN bearer. It calculates bearer-level QoS from the individual PCC rules bound to that bearer:

| QoS parameter | Calculation rule |
|---|---|
| **GBR** of a bearer | Sum of GBR values of all **active** GBR PCC rules bound to that bearer |
| **MBR** of a bearer | Highest MBR among all PCC rules bound to that bearer |
| **APN-AMBR** | Enforced at PCEF; limits aggregate non-GBR traffic for the entire APN |

### Resource Sharing

When multiple PCC rules sharing the same SDF direction are grouped with a **sharing indication**:
- Shared GBR = highest GBR among the sharing rules (not the sum)
- Shared MBR = highest MBR among the sharing rules

### QCI and ARP Assignment

Each PCC rule carries a [QCI](../concepts/QCI-characteristics.md) and [ARP](../concepts/QCI-characteristics.md). The BBF uses QCI + ARP to bind the rule to an existing bearer or initiate a new bearer. Rules with the same QCI and ARP value are bound to the same bearer.

---

## Gate Control

The PCEF implements a **gate** per SDF direction (uplink / downlink) for each PCC rule:
- **Gate open** — packets matching the SDF filter are allowed to pass
- **Gate closed** — packets matching the SDF filter are blocked (dropped by PCEF)

Gate status is set by the PCRF in the PCC rule. The PCRF closes gates when:
- AF session is terminated (e.g. SIP BYE received at P-CSCF)
- Usage limit exceeded
- Subscriber credit exhausted (online charging)
- 3GPP PS Data Off is active for non-exempt traffic

---

## Bearer Binding Function (BBF) — GTP Mode

When GTP is used on S5/S8, the BBF resides in the PCEF (PGW). The BBF performs [step 3 of the binding mechanism](../concepts/PCC-architecture.md#binding-mechanism-611):

1. Receives activated PCC rules from the PCRF
2. For each PCC rule, selects an existing IP-CAN bearer with matching QCI + ARP, or triggers new bearer establishment
3. Notifies PCRF of the bearer binding result
4. Ensures GBR/MBR enforcement per bearer (see above)

In **PMIP mode** (S5/S8 PMIP), the BBF moves to the [BBERF](SGW.md) in the SGW; the PCEF then has no BBF.

---

## Application Detection and Control (ADC) — PCEF-based

When a [TDF](TDF.md) is not deployed, the PCEF may perform application detection:
- Uses **application identifiers** in PCC rules (DPI-based matching)
- Reports application start/stop events to PCRF via Gx event triggers
- Applies enforcement (gating, redirect, bandwidth limitation) per ADC rule received on Gx

---

## Traffic Steering

The PCEF may receive **traffic steering policy identifiers** from the PCRF (via Gx). Based on the identifier, the PCEF:
- Applies local steering policy (the policy content is local configuration — not sent by PCRF)
- May mark packets with appropriate DSCP or routing tags
- May forward traffic to (S)Gi-LAN service chains

Applicable only in non-roaming or home-routed scenarios.

---

## 3GPP PS Data Off

When a UE activates 3GPP PS Data Off (a UE feature blocking mobile data except exempt services):
- PCEF receives event trigger from PCRF
- PCRF closes the gates of all PCC rules for non-exempt SDFs, or modifies/deactivates the corresponding PCC rules
- Exempt SDFs (IMS signalling, emergency) continue normally
- If PCRF is not deployed, the PCEF uses predefined PCC rules to enforce this behavior

---

## Interfaces

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| **Gx** | [PCRF](PCRF.md) | Diameter | Bidirectional | Receive PCC rules; report events, usage, bearer bindings |
| **Gy** | [OCS](../entities/OCS.md) | Diameter (Ro) | Bidirectional | Online credit control for SDF-based charging (TS 32.251) |
| **Gz** | OFCS | GTP' / Rf | Outbound | Offline charging data records for SDFs (TS 32.240) |
| **Gw** | [PFDF](../entities/PFDF.md) | — | Receive | Packet Flow Description updates for application detection |
| **S5/S8** | [SGW](SGW.md) | GTP-U / PMIP | Bidirectional | User plane tunnel to/from serving gateway |

---

## Event Triggers Reported to PCRF

The PCEF contains an **Event Reporting Function (ERF)** that detects and reports:

| Trigger | Subscription required? |
|---|---|
| PLMN change | PCRF |
| QoS change | PCRF |
| QoS exceeds authorization | PCRF |
| Traffic mapping info change | Always (mandatory) |
| Location change (cell/area/CN) | PCRF |
| UE presence in Presence Reporting Area change | PCRF |
| Out of credit | PCRF |
| UE IP address change | Always (mandatory) |
| Usage report | PCRF |
| Start/stop of application traffic detection | PCRF |
| SRVCC CS-to-PS handover | PCRF |
| Credit management session failure | PCRF |
| Addition/removal of access to IP-CAN session | Always (mandatory) |

---

## Related Pages

- [PCRF](PCRF.md) — policy decision point; issues PCC rules to PCEF via Gx
- [PGW](PGW.md) — gateway node that hosts the PCEF function
- [PCC Architecture](../concepts/PCC-architecture.md) — full entity/reference-point overview
- [QCI Characteristics](../concepts/QCI-characteristics.md) — QCI table and ARP definition
- [SGW](SGW.md) — hosts BBERF in PMIP mode
