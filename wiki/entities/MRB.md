---
title: "MRB (Media Resource Broker)"
type: entity
tags: [IMS, MRB, MRFC, AS, media, Query-mode, In-Line-mode, Rc, Mr, Mr', Cr, SLA, QoS]
sources: [ts_123218v170000p.pdf]
updated: 2026-04-10
---

# MRB — Media Resource Broker

**Spec reference:** 3GPP TS 23.218 §13

Related pages: [MRF](MRF.md) · [TAS](TAS.md) ·
[AS Interaction Modes](../concepts/AS-interaction-modes.md) ·
[IMS Reference Points](../interfaces/IMS-reference-points.md)

---

## Role (§13.0)

The MRB is a functional entity responsible for:
1. **Collection**: gathering published information about available MRF (MRFC/MRFP) resources
2. **Supply**: providing appropriate MRF information to consuming entities (primarily Application Servers)
3. **Selection**: assigning specific MRFC resources to calls on behalf of applications

MRB enables a **pool of heterogeneous MRF resources** to be shared across multiple
heterogeneous applications. It abstracts the MRFC selection from the AS, allowing
the AS to request "conference resource for 10 participants" without needing to know
which specific MRFC to contact.

**Selection criteria** MRB considers:
- Specific characteristics of media resources required (codec, mixing capacity, etc.)
- Identity of the requesting application
- Fair-share allocation rules across different applications
- Per-application or per-subscriber SLA / QoS criteria
- Capacity models of particular MRF resources
- Whether a visited network MRB can be used

An MRB instance may operate in **both Query and In-Line modes simultaneously**. An AS
may also use Query mode for some calls and In-Line mode for others.

---

## Mode 1: Query Mode (§13.1)

```mermaid
graph LR
    TF[Transit\nFunction] -- ISC --> AS
    AS -- Rc --> MRB
    AS -- Cr --> MRFC
    AS -- Mr' direct --> MRFC
    AS -- ISC --> SCSCF[S-CSCF]
    SCSCF -- Mr --> MRFC
    MRFC -- Mp H.248 --> MRFP
```

**Flow:**

```mermaid
sequenceDiagram
    participant AS
    participant MRB
    participant MRFC

    AS->>MRB: Rc: Request MRF resources\n(specify required attributes)
    MRB->>MRB: Select suitable MRFC\nbased on criteria
    MRB-->>AS: Rc: Response with MRFC address(es)
    AS->>MRFC: Set up dialog\n(via S-CSCF ISC+Mr OR direct Mr')
    Note over MRFC: Resource in use
    AS->>MRB: Rc: Done with resource
    MRB->>MRFC: Return resource to idle pool
```

**Key properties:**
- AS controls when resource is released (notifies MRB explicitly)
- AS can request resources for multiple calls in advance (single Rc request/response)
- AS can incrementally request more or release resources it no longer needs
- Control packages (media commands) supported over Cr between AS and MRFC

---

## Mode 2: In-Line Mode (§13.2)

```mermaid
graph LR
    TF[Transit\nFunction] -- ISC --> AS
    AS -- Mr' --> MRB
    AS -- Cr --> MRFC
    MRB -- Mr' --> MRFC
    MRB -- ISC --> SCSCF[S-CSCF]
    SCSCF -- Mr --> MRFC
    MRFC -- Mp H.248 --> MRFP
```

**Flow:**

```mermaid
sequenceDiagram
    participant AS
    participant MRB
    participant MRFC

    AS->>MRB: Establish dialog\n(via Mr' direct OR via S-CSCF ISC+Mr)
    Note over MRB: Request contains info about\nKind of MRF required
    MRB->>MRB: Select appropriate MRFC
    MRB->>MRFC: Forward request\n(via Mr' direct OR S-CSCF ISC+Mr)
    Note over MRFC: Resource allocated
    MRFC-->>MRB: Response
    MRB-->>AS: Response
    Note over AS,MRFC: Subsequent dialog messages traverse MRB\n(+ S-CSCF if it was in path)
    Note over AS,MRFC: Control packages (Cr) flow directly\nAS ↔ MRFC, bypassing MRB
    AS->>MRB: BYE (session release)
    Note over MRB: Infers resource release\nfrom BYE
```

**Key properties:**
- MRB is **in the SIP signaling path** between AS and MRFC
- AS cannot distinguish which MRFC was selected (MRB abstracts it)
- All in-dialog SIP messages traverse MRB (and S-CSCF if applicable)
- Control packages (Cr) bypass MRB — go directly AS↔MRFC
- MRB infers release from BYE — no explicit release notification needed (unlike Query mode)

---

## Mode Comparison

| Aspect | Query Mode | In-Line Mode |
|---|---|---|
| AS role | AS selects, controls resource lifecycle | AS unaware of which MRFC selected |
| MRB in signaling path | No — only Rc request/response | Yes — all SIP messages traverse MRB |
| Resource release trigger | Explicit AS notification to MRB via Rc | Implicit — MRB sees BYE in path |
| Multi-call advance reservation | Yes — single Rc request for multiple calls | No |
| Control packages (Cr) | AS ↔ MRFC directly | AS ↔ MRFC directly (bypasses MRB) |
| AS→MRFC path after selection | Via S-CSCF (Mr) or direct (Mr') | Via MRB (which uses Mr' or S-CSCF Mr) |

---

## MRB Knowledge of MRF Resources (§13.3)

Information MRB must maintain:

| Information | Description |
|---|---|
| Available resources and attributes | Current and future; includes planned/unplanned downtime, scheduled add/remove |
| Fair-share rules | Allocation policies across competing applications |
| Capacity models | Per-MRFC capacity characteristics |
| Future reservations | Pre-booked resources (e.g. for conferencing or anticipated traffic spikes) |

MRB acquires this information via:
- **Operations interfaces** (O&M systems, provisioning)
- **Direct MRB–MRFC interface** (MRFC publishes resource info to MRB via Cr)

---

## Interfaces

| Interface | Peer | Purpose |
|---|---|---|
| **Rc** | AS | Query mode: AS requests MRF resources; MRB responds with MRFC addresses |
| **Mr'** | AS or MRFC | In-Line mode: AS→MRB session setup; MRB→MRFC forwarding (SIP-based) |
| **Cr** | MRFC | MRFC publishes resource information to MRB; media control |
| **Mr** | S-CSCF | Alternative path (via S-CSCF ISC + Mr) when not using direct Mr' |

---

## Inter-MRB

An MRB may use the services of another (visited network) MRB, using Mr, Mr', and Rc
interfaces as appropriate. This supports visited-network MRF resource selection for
roaming scenarios.
