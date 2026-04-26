---
title: "IMS Charging Information — Parameter Reference"
type: concept
tags: [charging, IMS, IMS-Information, Service-Information, CDR, Ro, Rf, Nchf, offline, online, converged, parameters, AVP]
sources: [ts_132260v170300p.pdf]
updated: 2026-04-13
---

# IMS Charging Information — Parameter Reference

This page documents the IMS-specific charging parameters defined in 3GPP TS 32.260 §6.3–§6.4: the **Service Information** wrapper, the **IMS Information** block (the ~50-field IMS-specific container embedded in every Rf/Ro charging message), and the **IMS Charging Information** block used in converged (Nchf) charging.

See [IMS charging architecture](IMS-charging-architecture.md) for the high-level offline/online/converged framework, [IMS offline charging flows](../procedures/IMS-offline-charging-flows.md) for CDR generation flows, and [IMS online charging flows](../procedures/IMS-online-charging-flows.md) for Ro procedure flows.

---

## 1. Service Information Structure (§6.3.1.1)

The `Service Information` IE (Om) wraps all service-specific charging data sent in Rf (Charging Data Request) and Ro (Debit/Reserve Units Request) messages. For IMS, its primary component is `IMS Information`.

**Table 6.3.1.1.1 — Service Information components:**

| IE | Category | Description | Nodes that provide it |
|---|---|---|---|
| **Service Information** | Om | Top-level container for all 3GPP-specific parameters | All |
| Subscriber Identifier | Om | Public User Identity/Identities (Public User ID) for offline charging | Not IBCF, I-CSCF, MGCF, BGCF, TF |
| **IMS Information** | Om | IMS-specific parameters (see §2 below) | All |
| PS Information | Oc | PS-domain parameters (TS 32.251) — GGSN Address, User Location, etc. | All |
| — GGSN Address | Oc | IP address of node that generated the Access Charging ID | Not I-CSCF, MGCF, BGCF, IBCF |
| — User Location Info | Oc | Network-provided UE location for 3GPP accesses | Not MGCF, BGCF, TF |
| — MS Time Zone | Oc | Time zone offset (15-minute steps) where UE currently resides | Not MGCF, BGCF, TF |
| — Subscriber Equipment Number | Oc | IMEI identifying the UE (P-CSCF, S-CSCF, AS) | P-CSCF, S-CSCF, AS |
| — 3GPP PS Data Off Status | Oc | UE's Data Off status (Activated/Deactivated, per TS 23.228) | AS |
| VCS Information | Oc | Voice call over CS parameters (TS 32.276) | MGCF, AS |
| — ISUP Cause | Oc | Reason call was released (PSTN-side cause) | MGCF |
| — VLR Number | Oc | E.164 address of VLR serving the user | AS |
| — MSC Address | Oc | E.164 address of MSC that generated the network call reference number | AS |

---

## 2. IMS Information (§6.3.1.2)

The `IMS Information` grouped IE is the core IMS-charging-specific block. It is provided by **all** IMS NEs in both Rf and Ro messages, except where noted in the "Nodes" column. The node-type column abbreviations: SC = S-CSCF, PC = P-CSCF, IC = I-CSCF, EC = E-CSCF, MR = MRFC, MC = MGCF, BG = BGCF, AS = AS, IB = IBCF, TF = TF, TR = TRF, AT = ATCF.

**Table 6.3.1.2.1 — IMS Information fields:**

| IE | Cat | Description | Nodes |
|---|---|---|---|
| Event Type | Oc | SIP Method + SIP "Event" header content + "expires" header when present | All |
| Node Functionality | M | Function of the reporting node (S-CSCF, P-CSCF, I-CSCF, etc.) | All |
| Role of Node | Om | Whether node serves Originating or Terminating party | Not MRFC |
| User Session ID | Om | SIP Call-ID of the session. If B2BUA: incoming dialog's Call-ID | All |
| Outgoing Session ID | Oc | B2BUA outgoing dialog's Call-ID | AS only |
| Session Priority | Oc | Session priority level | All |
| Calling Party Address | Om | SIP/Tel URI of calling party (Public User ID or Public Service ID) | All |
| Called Party Address | Om | SIP/Tel URI of called party. For registration: Public User ID under registration | All |
| Number Portability routing information | Oc | After DNS/ENUM number portability query | SC, IC, AS, MC, BG, TF, TR |
| Carrier Select routing information | Oc | After DNS/ENUM carrier selection query | SC, IC, AS, MC, BG, TF, TR |
| Alternate Charged Party Address | Oc | AS-substituted address to be charged (replaces calling party for billing) | AS only |
| Requested Party Address | Oc | Original destination before forwarding; only if different from Called Party Address | SC, PC, EC, AS, MR, TF, TR, AT |
| Called Asserted Identity | Oc | Final asserted identity from SIP 2xx response | SC, EC, AS, MR, TF, TR, AT |
| Called Identity Change | Oc | Terminating identity change event + timestamp (SIP UPDATE/RE-INVITE) | SC, PC, EC, AS, TF, TR, AT |
| Called Identity Change Time Stamp | Oc | Time of SIP UPDATE/RE-INVITE with changed terminating identity | SC, PC, EC, AS, TF, TR, AT |
| Called Identity | Oc | Changed terminating identity received in SIP UPDATE/RE-INVITE | SC, PC, EC, AS, TF, TR, AT |
| Associated URI | Oc | Non-barred public user identity linked to IMPU under registration | SC, PC, IC, IB |
| Time Stamps | Oc | SIP Request time + SIP Response time | All |
| Application Server Information | Oc | SIP URIs of addressed ASes + called-party number (E.164) | SC, EC, AS, MR, TF, TR |
| Inter Operator Identifier | Oc | Network neighbours (originating + terminating IOIs) via SIP signalling; may occur multiple times | All |
| IMS Charging Identifier | Om | ICID generated by this IMS node for the SIP session | All |
| Related IMS Charging Identifier | Oc | ICID of the session being accessed in access transfer cases | PC, AS, AT |
| Related IMS Charging Identifier Generation Node | Oc | Identity of node that generated the Related ICID | PC, AS, AT |
| Transit IOI List | Oc | Involved transit networks via SIP signalling; may occur multiple times. From AS: inbound or outbound from S-CSCF | Not EC, not IC, not AT |
| Early Media Description | Oc | Session/media parameters active before final SIP INVITE answer; multi-valued | Not IC, not BG |
| SDP Session Description | Oc | SDP "attribute-line" (=c=, b=, k=, a=, etc.) | Not IC, not BG |
| SDP Media Component | Oc | Grouped field with one media component's sub-fields; multiple per session | Not IC, not BG |
| Served Party IP Address | Oc | P-CSCF's IP contact address of the calling or called party | PC only |
| Server Capabilities | Oc | S-CSCF server capabilities (TS 29.229) | IC only |
| Trunk Group ID | Oc | Incoming and outgoing PSTN trunk legs | MC only |
| Bearer Service | Oc | Bearer service used for PSTN leg | MC only |
| Service Id | Oc | Service identifier; for conferences: conference-ID | MR only |
| Service Specific Info | Oc | AS-provided service data; includes TAD Identifier (CS or PS access type for SRVCC) | AS only |
| Message Bodies | Oc | SIP Message body metadata (Content-Type, Length, Disposition, Originator). Only if present in triggering SIP message | All (note 2) |
| Access Network Information | Oc | Content of SIP P-Access-Network-Info header | Not TF, not TR |
| Additional Access Network Information | Oc | Content of additional SIP P-Access-Network-Info header | Not TF, not TR |
| Cellular Network Information | Oc | SIP Cellular-Network-Info header; used when UE employs non-cellular IP-CAN (untrusted WLAN) to relay last-camped radio cell | Not TF, not TR |
| Access Network Info Change | Oc | Grouped field: subsequent P-Access-Network-Info changes + timestamps | SC, PC, EC, IB, MR, AT |
| IMS Communication Service ID | Oc | IMS communication service identifier from P-Asserted-Service (upstream from S-CSCF) or Feature-Caps header "+g.3gpp.icsi-ref" | SC, PC, EC, IB, AT, TR, AS |
| IMS Application Reference ID | Oc | IMS application reference identifier from SIP Request (note 5: only present if triggered on SIP REGISTER 200 OK) | PC, SC |
| Cause Code | Oc | Cause value | All |
| Reason Header | Oc | SIP Reason header in BYE or CANCEL; may occur multiple times; reliability not guaranteed if SIP/CANCEL originates outside operator trust domain | All |
| Real Time Tariff Information | Oc | Tariff/add-on charge received (RTTI XML body) | SC, IB, MC, AS |
| Online Charging Flag | Oc | Indicates OC Request was based on ECF address from P-Charging-Function-Addresses. **Note: no proof that online charging action was actually taken** | SC, AS, MR |
| Account Expiration | Oc | Subscriber account expiration date/time of day | OCS only (in response) |
| Initial IMS Charging Identifier | Oc | ICID generated by the IMS node for the **initial** SIP session (for IMS service continuity access transfer) | AS, AT |
| NNI Information | Oc | NNI used for interconnection and roaming (Session Direction, NNI Type, Relationship Mode, Neighbour Node Address). May occur multiple times | PC, SC, BG, AS, IB, AT, TF, TR |
| From Address | Om | SIP From header content | Not TR |
| IMS Emergency Indication | Oc | Flags if this is an IMS emergency registration or session | PC, SC, IC |
| IMS Visited Network Identifier | Oc | Content of SIP P-Visited-Network-ID header | PC, SC, AS |
| SIP Route header received | Oc | Topmost route header in received initial SIP INVITE or non-session SIP MESSAGE | PC, SC, IB, TR, AT, TF |
| SIP Route header transmitted | Oc | Route header in transmitted initial SIP INVITE or non-session SIP MESSAGE | PC, SC, IC, AT, TF |
| Instance Id | Oc | Uniquely identifies the device (fixed or mobile) of the served user | PC, SC, AS |
| TAD Identifier | Oc | Access network type (CS or PS) through which the session shall be terminated (SRVCC context) | AS only |
| FE Identifier List | Oc | IM CN subsystem functional entity addresses and/or AS and application identifiers where the IM CN subsystem FE creates charging information for the related CDR | All |

---

## 3. Offline Per-Node Operation Type Matrix (§6.3.2)

The offline Charging Data Request (Rf) carries the IMS Information in different operation types per node. Operation types: **S** (Start) / **I** (Interim) / **S** (Stop) / **E** (Event) → notation `SISE`. A dash (`-`) means that operation type is not applicable for that field+node.

**Node operation type support:**

| Node Type | Supported Operation Types |
|---|---|
| S-CSCF | S/I/S/E (session + event) |
| E-CSCF | S/I/S/E (session + event) |
| P-CSCF | S/I/S/E (session + event) |
| I-CSCF | E only (event CDR only) |
| MRFC | S/I/S (session only, no Event) |
| MGCF | S/I/S/E (session + event) |
| BGCF | E only (event CDR only) |
| AS | S/I/S/E (session + event) |
| IBCF | S/I/S/E (session + event) |
| TF | S/I/S/E (session + event) |
| TRF | S/I/S/E (session + event) |
| ATCF | S/I/S/E (session + event) |

**Key per-field operation type rules (selected from Table 6.3.2.1):**

| IE | S-CSCF | I-CSCF | MRFC | AS | MGCF |
|---|---|---|---|---|---|
| Operation Interval | SI-- | - | SI-- | SI-- | SI-- |
| Outgoing Session ID | S--E | - | - | S--E | - |
| Calling Party Address | SISE | E | SIS | SISE | SISE |
| Called Party Address | SISE | E | SIS | SISE | SISE |
| Server Capabilities | - | E | - | - | - |
| Service Id (conference) | - | - | SIS | - | - |
| Trunk Group ID / Bearer Service | - | - | - | - | SISE |
| TAD Identifier | - | - | - | SISE | - |
| Online Charging Flag | SI-E | - | SIS | SI-E | - |
| Related IMS Charging Identifier | - | - | - | SISE | - |
| Access Network Info Change | -I-- | - | -I-- | -I-- | - |
| Message Bodies | SISE | - | - | SISE | SISE |
| IMS Emergency Indication | SISE | E | - | - | - |
| SDP Session Description | S-E | S-E | S-E | S-E | - |
| SDP Media Component | S-E | S-E | S-E | S-E | - |

**Notes:**
1. Many fields only present if available in the CTF of the IMS node
2. Message Bodies only if included in the triggering SIP message
3. Service Information only if Charging Data Request triggered on a SIP message
4. SDP Media Component: TBD for IBCF

---

## 4. Online Per-Node Operation Type Matrix (§6.3.3)

The online Debit/Reserve Units Request (Ro) carries IMS Information per operation type: **I** (Initial) / **U** (Update) / **T** (Terminate) / **E** (Event) → notation `IUTE`.

**Online node support:**

| Node | Supported Operation Types |
|---|---|
| IMS-GWF | I/U/T/E |
| MRFC | I/U/T (no Event) |
| AS | I/U/T/E |

**Key per-field operation type rules (Table 6.3.3.1):**

| IE | IMS-GWF | MRFC | AS |
|---|---|---|---|
| Termination Cause | --T- | --T- | --T- |
| Requested Action | - | - | --E |
| Outgoing Session ID | - | - | IUTE |
| Session Priority | I--E | I-- | I--E |
| Called Asserted Identity | -U-E | -U-E | -U-E |
| Associated URI | --E | - | - |
| SDP Session Description | IU-- | IU-- | IU-- |
| SDP Media Component | IU-- | IU-- | IU-- |
| Calling Party Address | IUTE | IUT | IUTE |
| Called Party Address | IUTE | IUT | IUTE |
| IMS Communication Service ID | I--E | - | I--E |
| Tariff Information (RTTI) | -U-- | - | -U-- |
| Initial IMS Charging Identifier | - | - | IUTE |
| Access Transfer Information | - | - | -U-- |
| IMS Emergency Indication | - | - | - |
| 3GPP PS Data Off Status | - | - | E only |
| Message Bodies | IUTE | - | IUTE |

**Debit/Reserve Units Response (online) — IMS-specific fields:**

| IE | IMS-GWF | MRFC | AS |
|---|---|---|---|
| Account Expiration | - | - | IUTE |
| Announcement Information (in MUO) | IU-- | - | IU-- |
| Operation Failure Action | - | - | - |
| Extended Information | IUTE | IUT | IUTE |

---

## 5. IMS Charging Information for Converged Charging (§6.4.2)

In converged charging (Nchf), the `IMS Charging Information` block replaces the `IMS Information` inside `Service Information`. It is carried in the Charging Data Request/Response messages to/from the [CHF](../concepts/charging-architecture.md). The structure largely parallels §6.3.1.2 but is adapted for the 5G SBI context.

### 5.1 IMS Charging Information Structure (Table 6.4.2.2.1)

Key structural differences vs §6.3.1.2:

| Difference | §6.3.1.2 (Rf/Ro) | §6.4.2.2 (Nchf) |
|---|---|---|
| Node identity | Node Functionality (M) | IMS Node Functionality (Om) |
| User identity | User Name (in request header) | User Information → User Identifier (Oc) + User Equipment Info (Oc) |
| Subscriber address | Subscriber Identifier (Om, in Service Information) | Subscriber Identifier (Om, at top level of Charging Data Request) |
| PS info anchor | GGSN Address | Serving Node Address (Oc) — equivalent function |
| Location/timezone | User Location Info (Oc), MS Time Zone (Oc) | User Location Info (Oc), UE Time Zone (Oc) |
| Called asserted | Called Asserted Identity (singular) | Called Asserted Identities (plural, Oc) |
| Emergency indication | IMS Emergency Indication (Oc) | IMS Emergency Indication (Oc) — same |
| Access Transfer | Access Transfer Information (Oc) | Access Transfer Information (Oc) — IU-- only |

All other fields (ICID, IOI, SDP Session/Media, Time Stamps, Application Server Information, etc.) are present with the same semantics.

### 5.2 Converged Charging Data Request — Per-Operation Matrix (Table 6.4.2.3.1)

Single node column (generic IMS Node), operation types IUTE:

| IE | Operation |
|---|---|
| Session Identifier | -UTE (not at Initial) |
| Subscriber Identifier | IUTE |
| Retransmission Indicator | IUT- |
| Notify URI | IU-- |
| Triggers | -UT- |
| Multiple Unit Usage | IUT- |
| Requested Unit | IU-- |
| Used Unit Container | -UT- |
| IMS Charging Information | IUTE |
| Role of Node | IUTE |
| Early Media Description | I--E |
| SDP Session Description | IU-- |
| SDP Media Component | IU-- |
| Session Priority | I--E |
| NNI Information | I-TE |
| IMS Communication Service ID | -U-- |
| Access Transfer Information | IU-- |
| Cause Code | --TE |
| Reason Header | --TE |
| 3GPP PS Data Off Status | ---E |
| TAD Identifier | IU-- |
| Related IMS Charging Identifier | IUTE |
| Serving Node Address | IU-- |
| VLR Number / MSC Address | I--E |

### 5.3 Converged Charging Data Response — Per-Operation Matrix (Table 6.4.2.3.2)

| IE | Operation |
|---|---|
| Session Identifier | IUTE |
| Invocation Timestamp | IUTE |
| Invocation Result | IUTE |
| Invocation Sequence Number | IUTE |
| Session Failover | I--- |
| Triggers | - (not present in response) |
| Multiple Unit information | I--E |
| — Granted Unit | IU-- |
| — Validity Time | IU-- |
| — Final Unit Indication | IU-- |
| — Result Code | IU-E |

### 5.4 Formal Parameter References

- CDR abstract syntax and encoding for Rf: **TS 32.298**
- Charging event parameter definitions (Ro): **TS 32.299**
- CHF CDR parameters (Nchf): **TS 32.298**
- IMS charging resources attributes: **TS 32.291**

---

## 6. Key Design Notes

**ICID (IMS Charging Identifier)** — generated by the first IMS node that processes the SIP transaction. All subsequent nodes receive it via SIP P-Charging-Vector header and include it in their CDR/Ro message to enable correlation across all per-node records for a single session. See [IMS charging architecture](IMS-charging-architecture.md).

**FE Identifier List** — enables the CDF to find all CDRs related to a given IM CN subsystem transaction. Each node lists the addresses of other NEs that are generating CDRs for the same session, allowing off-line correlation without requiring a centralized session store.

**Online Charging Flag** — the presence of this flag in the CDR does NOT prove that online charging was carried out. It only indicates that a P-Charging-Function-Addresses header was received with an ECF address. Whether the IMS-GWF or AS actually performed Ro credit control is a separate matter.

**From Address (Om, not in TRF)** — mandatory for all nodes except TRF, reflecting that the From header is preserved end-to-end in SIP but the Transit Function may process messages without a visible From header in transit contexts.

---

## Cross-References

- [IMS charging architecture](IMS-charging-architecture.md) — ICID, IOI, offline/online/converged paradigms, trigger conditions
- [IMS offline charging flows](../procedures/IMS-offline-charging-flows.md) — how CDRs are generated using these parameters
- [IMS online charging flows](../procedures/IMS-online-charging-flows.md) — how Ro messages are triggered using these parameters
- [IMS converged charging flows](../procedures/IMS-converged-charging-flows.md) — Nchf CHF-based charging
- [IMS CDR field reference](../protocols/IMS-CDR-field-reference.md) — per-node CDR field tables (§6.1.3)
- [charging AVPs](../protocols/charging-AVPs.md) — Diameter AVP reference (TS 32.299 §7)
- [Ro online charging protocol](../protocols/Ro-online-charging.md) — CCR/CCA message format
