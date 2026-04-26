---
title: "CUPS Parameter Reference (Sx Rule Types)"
type: concept
tags: [CUPS, Sx, PDR, FAR, URR, QER, parameters, session-context, PFD, usage-report]
sources: [ts23214]
updated: 2026-04-19
---

# CUPS Parameter Reference (Sx Rule Types)

**Source:** 3GPP TS 23.214 v16.1.0 §7

This page provides the normative attribute-level definitions for all four Sx rule types and the Usage Report structure. Each rule type is installed by the CP function in the UP function during Sx session management procedures. Applicability columns: **SGW** = SGW-U/SGW-C, **PGW** = PGW-U/PGW-C (PDN GW), **TDF** = TDF-U/TDF-C.

See [CUPS Architecture](CUPS-architecture.md) for the conceptual description of each rule type, and [Sx Session Management](../procedures/Sx-session-management.md) for the procedures by which rules are installed.

---

## 1. Session Context (§7.2)

The session context comprises the session related parameters plus all PDRs, URRs, QERs, and FARs that share the same Session ID.

| Attribute | Description | SGW | PGW | TDF |
|---|---|---|---|---|
| Session ID | Uniquely identifies the session. For SGW: one PDN connection; for PGW: one PDN connection + IP-CAN session; for TDF: one TDF session or TDF in unsolicited reporting mode | X | X | X |

---

## 2. Packet Detection Rule (PDR) (§7.3)

The PDR contains information required to classify a packet arriving at the UP function. There is at least one PDR per direction (UL and DL). Multiple PDRs with different precedences can exist per session; UP evaluates them in precedence order.

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| Session ID | Identifies the session associated to this PDR | | X | X | X |
| Rule ID | Unique identifier for this PDR | | X | X | X |
| Precedence | Determines evaluation order across all PDRs | Lower value = higher priority | | X | X |
| Source interface | "access side" (UL), "core side" (DL), "CP function" (from CP via Sx-u), "SGi-LAN" (from SGi-LAN service function) | Identifies packet origin direction | X | X | X |
| UE IP address | IPv4 address and/or IPv6 prefix with length | Used to match DL packets at PGW; also UL match at PGW when combined with SDF/App ID | | X | X |
| Network instance | Identifies the network instance for the incoming packet | Required when PGW/TDF-U handles multiple APNs with overlapping IPs, or UP nodes connect to peers in different IP domains | X | X | X |
| Local F-TEID | The local F-TEID at this UP function | Used to match GTP-U encapsulated packets by TEID | X | X | |
| List of SDF Filters | Service Data Flow filter(s) | IP 5-tuple or prefix-based filter | | X | X |
| Application ID | Application identifier for traffic detection | Used for application-based detection (ADC/PFD) | | X | X |
| Outer header removal | Instructs UP to remove one or more outer headers (e.g. IP+UDP+GTP) | Any extension header shall be stored for the packet | X | X | |
| Forwarding Action Rule ID | References the FAR to apply to matched packets | | X | X | X |
| List of URR IDs | References zero or more URRs — each identifies a measurement action | | X | X | X |
| List of QER IDs | References zero or more QERs — each identifies a QoS enforcement action | | X | X | X |

### Detection field combinations per node and direction

| Scenario | Detection fields used |
|---|---|
| SGW UL | Local F-TEID |
| SGW DL | Local F-TEID |
| PGW UL | Local F-TEID + UE IP address + SDF filter / Application ID |
| PGW DL | UE IP address + SDF filter / Application ID |
| TDF UL (solicited) | UE IP address + SDF filter / Application ID |
| TDF DL (solicited) | UE IP address + SDF filter / Application ID |
| TDF UL/DL (unsolicited) | Application ID only |

---

## 3. Usage Reporting Rule (URR) (§7.4)

The URR defines how a packet shall be accounted (what to measure) and when/how to report the measurements to the CP function.

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| Session ID | Identifies the session | | X | X | X |
| Rule ID | Unique identifier | | X | X | X |
| Reporting triggers | One or more events that cause the UP function to generate and send a usage report | See trigger list below | X | X | X |
| Periodic measurement threshold | Time point (e.g. time-of-day) when a periodic report is sent | Used for offline charging, usage monitoring, Quota-Idle-Timeout | X | X | X |
| Volume measurement threshold | UL / DL / total byte-count threshold | | X | X | X |
| Time measurement threshold | Duration threshold (in seconds) | | X | X | X |
| Event measurement threshold | Number of locally-defined events | | | X | X |
| Inactivity detection time | Period with no packets before time measurement stops | Timer restarted at end of each transmitted packet | | X | X |
| Event based reporting | Pointer to locally configured event policy for triggering usage reports | | | X | X |
| Linked URR ID | Points to one or more other URR IDs | Enables generation of a combined Usage Report for multiple URRs (see §5.2.2.4, TS 29.244) | | X | X |
| Measurement method | Indicates how to measure: volume, duration, combined volume/duration, or event | | X | X | X |
| Measurement information | Conditions for measurement | Used to: measure before QoS enforcement; pause/activate measurement (PGW Pause of Charging); reduce reporting for application start/stop events | X | X | X |

### Reporting trigger values

| Trigger | SGW | PGW | TDF |
|---|---|---|---|
| Detection of 1st DL packet on bearer (paging trigger) | X | | |
| Start of traffic detection without app instance/SDF filter | | X | X |
| Stop of traffic detection without app instance/SDF filter | | X | X |
| Start of traffic detection with app instance identifier and deduced SDF filter | | X | X |
| Stop of traffic detection with app instance identifier and deduced SDF filter | | X | X |
| Deletion of last PDR for a URR | X | X | X |
| Periodic measurement threshold reached | X | X | X |
| Volume measurement threshold reached | X | X | X |
| Time measurement threshold reached | X | X | X |
| Event measurement threshold reached | | X | X |
| Immediate report requested | X | X | X |
| Measurement of incoming UL traffic | | X | |
| Measurement of discarded DL traffic | X | | |

---

## 4. Forwarding Action Rule (FAR) (§7.5)

The FAR defines how to forward a packet that has been matched by a PDR, including encapsulation, decapsulation, and forwarding destination.

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| Session ID | Identifies the session | | X | X | X |
| Rule ID | Unique identifier | | X | X | X |
| Network instance | Identifies the network instance for the outgoing packet | Same multi-domain use cases as in PDR | X | X | X |
| Destination interface | "access side" (DL to UE), "core side" (UL to PGW/internet), "CP function" (forward to CP via Sx-u), "SGi-LAN" (to SGi-LAN service function) | | X | X | X |
| Outer header creation | Instructs UP to add IP+UDP+GTP header; contains F-TEID of peer entity (eNB, SGW, PGW, CP function) | Extension headers stored during removal are added here | X | X | |
| Send end marker packet(s) | Instructs UP to construct and send end marker packets as described in §5.8.1 | Sent together with outer header creation parameter for the new F-TEID | X | X | |
| Transport level marking | UL/DL DiffServ Code Point (DSCP) setting | | X | X | |
| Forwarding policy | Reference to preconfigured forwarding treatment for FMSS or HTTP redirection; contains TSP ID or Redirect Destination | TSP ID action is enforced _before_ outer header creation actions | | X | X |
| Container for header enrichment | Information used by UP for header enrichment | UL direction only | | X | X |
| Buffer Control Information | How UP function shall perform buffering | See TS 29.244 §5.2.4 | X | | |
| Delay Downlink Packet Notification Information | D parameter (delay DDN) | See TS 29.244 §5.9.3 | X | | |
| Extended buffering Information | DL Data Buffer Expiration Time + DL Suggested Packet Count | See TS 29.244 §5.9.3 | X | | |

---

## 5. QoS Enforcement Rule (QER) (§7.6)

The QER defines bit-rate limiting and packet marking for QoS. All PDRs that reference the same QER share the same QoS resources (e.g. the same MBR gate).

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| Session ID | Identifies the session | | X | X | X |
| Rule ID | Unique identifier | | X | X | X |
| QoS Enforcement Rule correlation ID | Identity allowing UP to correlate multiple QERs across sessions for APN-AMBR enforcement for the same UE and APN | PGW-C provides same correlation ID for all non-GBR PDRs of all PDN connections to the same APN | | X | |
| Gate status UL/DL | Instructs UP to pass (open) or block (close) the flow; "close after measurement report" for termination action "discard" | | | X | X |
| Maximum bitrate (MBR) | UL/DL maximum bitrate | May contain: APN-AMBR; TDF session MBR; Bearer MBR; SDF MBR | | X | X |
| Guaranteed bitrate (GBR) | UL/DL guaranteed bitrate | Bearer GBR | X | X | |
| Down-link flow level marking | DL packet marking | PGW: SCI marking in GTP extension header (service indication towards GERAN per TS 23.060); TDF: DSCP marking for application indication | | X | X |
| Packet rate | Number of packets per time interval | UL/DL packet rate for Serving PLMN Rate Control and APN Rate Control (CIoT EPS per TS 23.401) | | X | |

---

## 6. Usage Report (§7.7)

The UP function sends a Usage Report to the CP function to report measurements for an active URR, or to report application traffic detection events for an active PDR. Reports may be generated repeatedly as long as a valid trigger applies; a **final** usage report is sent when a URR is removed or all PDR references to it are deleted.

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| Session ID | Identifies the session | PDN connection (SGW/PGW) or TDF session (TDF) | X | X | X |
| Rule ID | Identifies the PDR or URR that triggered the report | PDR Rule ID for: 1st DL packet detection (SGW); start/stop of traffic detection. URR Rule ID for all other triggers | X | X | X |
| Reporting trigger | Identifies the event that triggered the report | Full trigger list in §7.4 above | X | X | X |
| Start time | Timestamp when collection of measurement started | Not sent when trigger is start/stop of traffic detection | X | X | X |
| End time | Timestamp when measurement information was generated | Not sent when trigger is start/stop of traffic detection | X | X | X |
| Measurement information | Volume/time/events measured for this URR | For 1st DL packet: includes DSCP value. For start/stop with app instance + deduced SDF: includes app instance ID + deduced SDF filter (+ deduced UE IP for TDF unsolicited). Not sent for start/stop without app instance + deduced SDF | X | X | X |

---

## 7. Functional Description — How CP Constructs Rule Sets (§7.8)

The CP function provides Sx parameters at three aggregation levels. This explains _how_ the rule types above are combined to realize the full CUPS session.

### 7.1 PDN Connection / TDF Session Level Context (§7.8.2)

To realize session-level reporting and MBR enforcement, CP provides **per session**:

- **One URR** describing the reporting requirement for the entire PDN connection or TDF session
- **One QER** with APN-AMBR value (PGW case) or TDF session MBR (TDF case)
- CP includes the URR Rule ID and QER Rule ID in each PDR activated for the PDN connection/TDF session

For APN-AMBR enforcement across multiple PDN connections to the same APN:
- CP uses the **same QER Rule ID** (same QER correlation ID) in each PDR for non-GBR traffic across all PDN connections to that APN
- ARP-based admission control remains exclusively at CP — it is never sent to UP

### 7.2 Bearer Level Context (§7.8.3)

Per active bearer, CP provides:

- **One or two FARs** — one for DL (both SGW and PGW), one for UL (SGW only)
- **One URR** for bearer-level charging/reporting
- **One QER** for bearer MBR/GBR enforcement

CP associates each PDR to the appropriate bearer by embedding the Rule IDs (FAR, URR, QER) in the PDR. **CP never sends the EPS Bearer ID to the UP function** — it maintains the Bearer ID ↔ Sx rule mapping internally and handles all bearer binding decisions.

### 7.3 Measurement Key Level Context (§7.8.4)

For charging and monitoring, CP provides **one URR per Charging Key and/or Monitoring Key**:

- CP maintains the mapping between each Charging Key / Monitoring Key applicable to an SDF and the URR Measurement Key on Sx
- CP may provide a **single combined URR** for a Charging Key + Monitoring Key pair when their measurement requirements completely overlap
- To associate a PDR's traffic with one or more measurement keys, CP includes a reference (URR Rule ID) to each applicable URR in the PDR

---

## 8. PFD Management Parameters (§7.9.1)

PFD management request attributes (node-level, not session-level; applicable to PGW/TDF only):

| Attribute | Description | Comment | SGW | PGW | TDF |
|---|---|---|---|---|---|
| PFD(s) | Extension to application detection filter | Provisioning PFD(s) for an App ID replaces ALL existing PFDs in PGW-U/TDF-U for that App ID | | X | X |
| Application ID | Application identifier the PFD(s) are associated with | Provisioning an App ID _without_ PFD(s) removes all stored PFDs for that App ID | | X | X |

**Key rules:**
- Multiple Application IDs can be managed in a single PFD management request message
- If two CP functions push different PFDs for the same App ID to the same UP function, the **last received** PFDs win (conflict risk in multi-CP deployments — see §5.11.4)

---

## Cross-References

- [CUPS Architecture](CUPS-architecture.md) — PDR/FAR/URR conceptual description, detection scenarios, forwarding scenarios
- [Sx Session Management](../procedures/Sx-session-management.md) — procedures by which these rules are installed/modified/removed
- [Sx Interface](../interfaces/Sx.md) — interface summary and message types
- [PCC Architecture](PCC-architecture.md) — how PCRF-driven PCC rules map to PDR/FAR/URR/QER
- TS 29.244 — wire-level encoding of all parameters (PFCP protocol)
