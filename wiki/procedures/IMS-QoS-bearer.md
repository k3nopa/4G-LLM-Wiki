---
title: "IMS QoS and Bearer Interaction"
type: procedure
tags: [IMS, QoS, PCC, PCRF, P-CSCF, bearer, Rx, SDP, preconditions, VoLTE, media-flow]
sources: [ts_123228v160500p.pdf]
updated: 2026-04-10
---

# IMS QoS and Bearer Interaction

Covers TS 23.228 §5.4.5–5.4.9 and §5.4.12: the mechanisms by which IMS sessions interact with
the underlying IP-CAN (4G EPS bearer layer) through PCC (Policy and Charging Control) to authorize,
reserve, enable, and release QoS resources for media streams.

Related pages: [P-CSCF](../entities/P-CSCF.md) · [PCRF](../entities/PCRF.md) ·
[EPS Bearer Model](../concepts/EPS-bearer.md) · [IMS Registration](IMS-registration.md) ·
[IMS Reference Points](../interfaces/IMS-reference-points.md)

---

## 1. Bearer Establishment with Pre-Alerting (§5.4.6.3)

When QoS-assured preconditions are used (§5.4.8), bearer establishment occurs *before* alerting
the called party. The flow (Figure 5.7) has three main phases:

```mermaid
sequenceDiagram
    participant UE_O as Originating UE
    participant P_O as P-CSCF (orig)
    participant S_O as S-CSCF (orig)
    participant S_T as S-CSCF (term)
    participant P_T as P-CSCF (term)
    participant UE_T as Terminating UE
    participant PCRF as PCRF/PCF

    Note over UE_O,UE_T: Phase 1 — Session Initiation
    UE_O->>P_O: INVITE (SDP offer, preconditions)
    P_O->>PCRF: AAR (authorize QoS resources, Rx)
    PCRF-->>P_O: AAA
    P_O->>S_O: INVITE
    S_O->>S_T: INVITE (via I-CSCF if cross-network)
    S_T->>P_T: INVITE
    P_T->>UE_T: INVITE

    Note over UE_O,UE_T: Phase 2 — Pre-Alerting / Resource Reservation
    UE_T-->>P_T: 183 Session Progress (SDP answer, precond not met)
    P_T->>PCRF: AAR (authorize terminating QoS)
    PCRF-->>P_T: AAA
    P_T-->>S_T: 183
    S_T-->>S_O: 183
    S_O-->>P_O: 183
    P_O-->>UE_O: 183

    Note over UE_O,PCRF: IP-CAN bearer creation (UE ↔ PCEF ↔ PCRF gate open)
    PCRF->>P_O: RAR (enable media flow)
    P_O-->>PCRF: RAA
    UE_O->>P_O: PRACK / UPDATE (preconditions met)
    P_O->>S_O: PRACK / UPDATE
    S_O->>S_T: PRACK / UPDATE
    S_T->>P_T: PRACK / UPDATE
    P_T->>UE_T: PRACK / UPDATE

    Note over UE_O,UE_T: Phase 3 — Alerting and Answer
    UE_T-->>P_T: 180 Ringing
    P_T-->>UE_O: 180 Ringing (via S-CSCFs)
    UE_T-->>P_T: 200 OK
    P_T-->>UE_O: 200 OK (via S-CSCFs)
    UE_O->>UE_T: ACK
```

Key property: the IP-CAN bearer is established and the gate is opened **before** the called UE
starts ringing, ensuring media can flow immediately at answer.

---

## 2. PCC Interactions Overview (§5.4.7.0)

Eight defined interactions between IMS signalling plane and the IP-CAN bearer plane:

| # | Interaction | Direction | Trigger |
|---|---|---|---|
| 1 | **Authorize QoS Resources** | P-CSCF → PCRF (Rx AAR) | SDP in SIP INVITE/200 OK/UPDATE |
| 2 | **Resource Reservation** | UE-initiated or network-initiated | UE bearer request or PCRF push |
| 3 | **Enable Media Flows** | PCRF → PCEF (gate open) | Both endpoints confirmed media OK |
| 4 | **Disable Media Flows** | PCRF → PCEF (gate close) | Session hold, call waiting, etc. |
| 5 | **Revoke Authorization** | PCRF → PCEF | IMS session release |
| 6 | **IP-CAN Bearer Release Indication** | PCEF → PCRF → P-CSCF | Bearer dropped in network |
| 7 | **Authorization of Bearer Modification** | PCEF ↔ PCRF | UE QoS change request |
| 8 | **Indication of Bearer Modification** | PCEF → PCRF → P-CSCF | MBR drops to 0 kbit/s |

**PCEF gate function:** The PCEF maintains a logical gate per media flow. Gate open = media
passes; gate closed = media blocked. The PCRF controls gates via Gx; the P-CSCF drives PCRF via Rx (4G) or N5 (5G).

**SDP-to-authorization derivation:** The [P-CSCF](../entities/P-CSCF.md) inspects SDP bodies
in SIP messages and derives IP flow descriptors (5-tuple: src/dst IP, src/dst port, protocol)
plus bandwidth parameters (b=AS, b=RS, b=RR lines) to construct the `AAR` to the PCRF.

---

## 3. Authorize QoS Resources (§5.4.7.1)

The [P-CSCF](../entities/P-CSCF.md) sends an `AAR` (AA-Request) on the **Rx** interface to the
[PCRF](../entities/PCRF.md) each time SDP is negotiated or re-negotiated:

- Triggered by: INVITE (offer), 200 OK (answer), UPDATE, re-INVITE
- Content: media component descriptors derived from SDP (codec, bandwidth, IP addresses, ports)
- PCC authorizes **each SIP session independently** — including parallel sessions from Call Waiting

**Authorization expression:**
- Maximum IP resource limits (bandwidth caps per flow direction)
- IP destination address and port restrictions (the authorized 5-tuple envelope)

The PCRF responds with `AAA` (AA-Answer) containing the PCC rules (QCI, ARP, GBR/MBR values)
that will be installed on the corresponding EPS dedicated bearer.

> A single SIP session may result in multiple media components (audio + video + BFCP), each
> mapped to a separate IP flow and potentially a separate EPS bearer.

---

## 4. Resource Reservation with PCC (§5.4.7.1a)

Two reservation models:

```mermaid
graph TD
    A{Reservation Model}
    A -->|UE-initiated| B["UE sends Bearer Resource Allocation Request<br/>(TS 23.401 §5.4.5)"]
    B --> C["PCEF requests authorization from PCRF (Gx CCR)"]
    C --> D["PCRF validates: request ≤ sum of authorized IP resources"]
    D -->|OK| E["Bearer created / modified"]
    D -->|Reject| F["Bearer request denied to UE"]

    A -->|Network-initiated| G["PCRF pushes PCC rules to PCEF (Gx RAR)"]
    G --> H["PCEF creates/modifies bearer toward UE"]
    H --> I["UE-side acceptance or rejection"]
```

**Validation rule:** The QoS requested by the UE for the bearer must not exceed the *sum* of
authorized IP resources for all media components mapped to that bearer.

---

## 5. Enabling Media Flows (§5.4.7.2)

The PCRF opens the gate for a media flow when:
- Both endpoints have confirmed the media path (preconditions met, or no preconditions used)
- The IMS session progresses to the ringing or answer phase

**Forked sessions:** When a SIP request is forked to multiple terminating UEs simultaneously,
the gate-open decision uses a logical OR — if *any* of the forked legs is active, the gate is open.

---

## 6. Disabling Media Flows (§5.4.7.3)

Gate is closed (media blocked) when:
- Session is placed on hold (`a=inactive` or `a=sendonly` in re-INVITE)
- Call Waiting: active session gates are managed to prevent media leakage to held sessions
- Network-policy-driven temporary disablement

The P-CSCF detects the hold condition from the SDP change and sends an updated `AAR` or
`STR`/session update to the PCRF, which then issues `RAR` to PCEF to close the gate.

---

## 7. Revoke Authorization (§5.4.7.4)

At IMS session release (SIP BYE), the P-CSCF sends `STR` (Session-Termination-Request) on Rx.
The PCRF then:
1. Revokes all PCC rules associated with the session
2. Instructs PCEF to remove the corresponding dedicated EPS bearer(s) (via Gx RAR)
3. Gates are closed; authorized resources are freed

---

## 8. IP-CAN Bearer Release Indication (§5.4.7.5)

When a bearer is released by the **network** (not triggered by IMS signalling):

```mermaid
sequenceDiagram
    participant PCEF
    participant PCRF
    participant P_CSCF as P-CSCF
    participant S_CSCF as S-CSCF
    participant UE

    PCEF->>PCRF: CCR (bearer released, Gx)
    PCRF->>P_CSCF: RAR (bearer release notification, Rx)
    P_CSCF-->>PCRF: RAA
    P_CSCF->>S_CSCF: BYE or re-INVITE (session modification/release)
    S_CSCF->>UE: SIP signalling propagated
```

The P-CSCF maps the lost bearer back to the IMS session(s) it was serving. Depending on
whether the bearer was the *only* bearer or a redundant one, the P-CSCF may:
- Trigger IMS session release (§5.10.3.1) via BYE
- Trigger session modification via re-INVITE (downgrade codec/bandwidth)

---

## 9. Authorization of IP-CAN Bearer Modification (§5.4.7.6)

When the UE requests a bearer modification (e.g. bandwidth change):

1. PCEF checks if new QoS is within already-authorized limits
2. If yes: bearer modification proceeds without PCRF consultation
3. If no (or insufficient info): PCEF requests updated authorization from PCRF (`CCR`)
4. PCRF may need to consult P-CSCF if the modification implies a session change

When the **IMS session** is modified (re-INVITE / UPDATE with new SDP):
- P-CSCF sends updated `AAR` to PCRF with new media component descriptors
- PCRF updates PCC rules; PCEF may modify or create/delete bearers accordingly

---

## 10. Indication of IP-CAN Bearer Modification (§5.4.7.7)

Special case: if a bearer's **MBR (Maximum Bit Rate) drops to 0 kbit/s**:

- PCEF reports to PCRF via `CCR` with modified QoS
- PCRF forwards indication to P-CSCF via `RAR`
- P-CSCF evaluates whether the IMS session can continue
- If the session cannot tolerate zero bandwidth: P-CSCF initiates session release (BYE)

This handles network congestion or resource withdrawal scenarios where the bearer still
exists structurally but can no longer carry the media.

---

## 11. Resource Sharing for Concurrent Sessions (§5.4.7.8)

When a UE has multiple simultaneous IMS sessions (e.g. active call + call on hold):

**Uplink/downlink tagging:** The P-CSCF assigns *resource sharing tags* to media components.
Components with the **same tag** are allowed to share the same IP-CAN bearer resources.

Rules:
- Emergency sessions: **never** participate in resource sharing
- Gate management: when resource sharing is active, all but one session's gates are closed
  (prevents double-counting of bandwidth)
- The active session's gate is open; the held session's gate is closed

```mermaid
graph LR
    UE --> B1[Bearer — shared tag X]
    B1 --> MC1[Media Component: Call A active]
    B1 --> MC2[Media Component: Call B on hold]
    MC1 -->|gate open| PCEF
    MC2 -->|gate closed| PCEF
```

---

## 12. Priority Sharing for Concurrent Sessions (§5.4.7.9)

The P-CSCF can send a **priority sharing indicator** to the PCRF on Rx, allowing the PCRF
to assign the *same bearer priority* (ARP) across multiple concurrent sessions.

Use case: Mission Critical Push-To-Talk (MCPTT) and similar services where multiple parallel
sessions must compete equally for radio resources rather than one pre-empting another.

---

## 13. QoS-Assured Preconditions (§5.4.8)

QoS preconditions ensure an IMS session is **not completed** (no ringing, no answer) until
the required IP-CAN bearer is established and has sufficient resources.

Three cases:

| Case | Description |
|---|---|
| **a** | Both endpoints require precondition confirmation before alerting. Bearer must be established at both ends before 180 Ringing is generated. |
| **b** | Both endpoints indicate preconditions met in the *initial* exchange (fast path — bearer already exists, e.g. second call). Alerting may proceed immediately. |
| **c** | No preconditions used. Session proceeds without waiting for bearer confirmation (best-effort). |

**Segmented resource reservation:** Each endpoint independently confirms its own local
resources are reserved. The originating side confirms when its UL bearer is up; the
terminating side confirms when its DL bearer is up. Both confirmations are required for Case a.

SIP signalling for preconditions uses the `Supported: precondition` and `Require: precondition`
headers, with `P-Early-Media` and `183 Session Progress` carrying interim SDP.

---

## 14. Event and Information Distribution (§5.4.9)

The S-CSCF or AS can push service information to UE endpoints via SIP event notifications:

```mermaid
sequenceDiagram
    participant UE
    participant P_CSCF as P-CSCF
    participant S_CSCF as S-CSCF
    participant AS

    UE->>P_CSCF: SUBSCRIBE (event package)
    P_CSCF->>S_CSCF: SUBSCRIBE
    S_CSCF->>AS: SUBSCRIBE (routed via iFC)
    AS-->>S_CSCF: 200 OK
    S_CSCF-->>P_CSCF: 200 OK
    P_CSCF-->>UE: 200 OK

    AS->>S_CSCF: NOTIFY (event data)
    S_CSCF->>P_CSCF: NOTIFY
    P_CSCF->>UE: NOTIFY
    UE-->>P_CSCF: 200 OK
    P_CSCF-->>S_CSCF: 200 OK
    S_CSCF-->>AS: 200 OK
```

Event packages used in IMS: `reg` (registration state), `dialog` (dialog state for call
waiting/hold), `presence`, `message-summary` (voicemail MWI), `conference`.

---

## 15. Public Service Identity (PSI) Routing (§5.4.12)

A **Public Service Identity (PSI)** is an IMPU that identifies a service rather than an
individual subscriber (e.g. a conference bridge, voicemail deposit, or emergency number).

### PSI Types

| Type | Example | Notes |
|---|---|---|
| Distinct PSI | `sip:conf-123@operator.com` | Exact match in HSS |
| Wildcarded PSI | `sip:conf-.*@operator.com` | Regex in HSS; matches range of identities |
| Subdomain-based PSI | `sip:conf.operator.com` | Routed by DNS subdomain rules |

### PSI Routing — Originating Side

1. S-CSCF evaluates iFC for the originating user
2. If a Filter Criterion matches the destination PSI: session routed to the AS that owns the PSI
3. AS handles the service

### PSI Routing — Terminating Side

Two sub-cases:
- **HSS routing:** HSS stores the AS address for the PSI. I-CSCF queries HSS (`LIR`); HSS
  returns direct AS address (no S-CSCF assignment needed)
- **S-CSCF routing:** PSI assigned to an S-CSCF; normal terminating iFC evaluation applies

### PSI Configuration in HSS

The HSS stores PSIs as IMPUs in service profiles. For wildcarded PSIs, the HSS applies
regex matching against the incoming identity to determine which AS handles the request.

---

## 16. Session Flow Taxonomy (§5.4a / Table 5.2)

TS 23.228 defines a structured taxonomy of session flows to enable precise specification of
which path a SIP session takes:

```mermaid
graph TD
    subgraph Origination
        MO1["MO#1 — Originating roaming<br/>(UE in visited network, S-CSCF in home)"]
        MO2["MO#2 — Originating home<br/>(UE and S-CSCF in home network)"]
        PSTN_O["PSTN-O — Originating from PSTN<br/>(MGCF entry)"]
        AS_O["AS-O — AS-originated<br/>(AS generates session on behalf of user)"]
        NI_O["NI-O — Network-initiated origination<br/>(IM-SSF, SCC-AS)"]
    end

    subgraph ServingToServing
        SS1["S-S#1 — Different operators<br/>(I-CSCF#2 as home entry)"]
        SS2["S-S#2 — Same operator<br/>(direct S-CSCF to S-CSCF)"]
        SS3["S-S#3 — PSTN termination, same network<br/>(BGCF within network)"]
        SS4["S-S#4 — PSTN termination, different network<br/>(BGCF to another BGCF)"]
    end

    subgraph Termination
        MT1["MT#1 — Terminating roaming<br/>(UE in visited, S-CSCF in home)"]
        MT2["MT#2 — Terminating home<br/>(UE and S-CSCF in home)"]
        MT3["MT#3 — CS domain roaming<br/>(VMSC/GMSC interwork)"]
        AS_T["AS-T#1-4 — AS-terminated variants"]
        PSTN_T["PSTN-T — Originating from PSTN termination"]
        NI_T["NI-T — Network-initiated termination"]
    end
```

### S-S#1 Flow: Different Network Operators (§5.5.1)

The most complete serving-to-serving case (Figure 5.10, ~40 steps):

```mermaid
sequenceDiagram
    participant UE_O as Originating UE
    participant P_O as P-CSCF#1
    participant S_O as S-CSCF#1
    participant I_T as I-CSCF#2
    participant HSS_T as HSS#2
    participant S_T as S-CSCF#2
    participant P_T as P-CSCF#2
    participant UE_T as Terminating UE

    UE_O->>P_O: INVITE (SDP offer)
    P_O->>S_O: INVITE (service control, iFC evaluation)
    Note over S_O: Originating iFC → AS chain (if configured)
    S_O->>I_T: INVITE (routed to terminating operator)
    I_T->>HSS_T: LIR (location query for terminating IMPU)
    HSS_T-->>I_T: LIA (S-CSCF#2 address)
    I_T->>S_T: INVITE
    Note over S_T: Terminating iFC → AS chain (if configured)
    S_T->>P_T: INVITE
    P_T->>UE_T: INVITE

    UE_T-->>P_T: 183 Session Progress (SDP answer)
    P_T-->>S_T: 183
    S_T-->>I_T: 183
    I_T-->>S_O: 183
    S_O-->>P_O: 183
    P_O-->>UE_O: 183

    Note over UE_O,UE_T: Resource reservation / preconditions (PRACK/200 PRACK/UPDATE)

    UE_T-->>P_T: 180 Ringing
    P_T-->>UE_O: 180 Ringing (via I-CSCF, S-CSCFs)

    UE_T-->>P_T: 200 OK (SDP confirmation)
    P_T-->>S_T: 200 OK
    S_T-->>I_T: 200 OK
    I_T-->>S_O: 200 OK
    S_O-->>P_O: 200 OK
    P_O-->>UE_O: 200 OK

    UE_O->>P_O: ACK
    P_O->>S_O: ACK
    S_O->>I_T: ACK
    I_T->>S_T: ACK
    S_T->>P_T: ACK
    P_T->>UE_T: ACK

    Note over UE_O,UE_T: Media flowing on established EPS bearers
```

Key characteristics of S-S#1:
- I-CSCF#2 is the topological hiding boundary between operators
- The I-CSCF performs an `LIR` to HSS#2 to find the assigned S-CSCF#2
- Both S-CSCFs evaluate their respective terminating/originating iFC chains independently
- SDP offer/answer follows the INVITE/183/UPDATE/200-OK exchange (may be deferred)

---

## Cross-References

| Topic | Page |
|---|---|
| P-CSCF as Rx AF | [P-CSCF](../entities/P-CSCF.md) |
| PCRF Gx/Rx policy engine | [PCRF](../entities/PCRF.md) |
| EPS dedicated bearer creation | [EPS Bearer Model](../concepts/EPS-bearer.md) |
| IMS Registration (P-CSCF discovery, S-CSCF assignment) | [IMS Registration](IMS-registration.md) |
| VoLTE MO Call full flow | [VoLTE MO Call](VoLTE-MO-call.md) |
| VoLTE MT Call full flow | [VoLTE MT Call](VoLTE-MT-call.md) |
| IMS reference points (Gm, Mw, Rx, ISC) | [IMS Reference Points](../interfaces/IMS-reference-points.md) |
