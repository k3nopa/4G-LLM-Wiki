---
title: "VoLTE Mobile-Terminated Call"
type: procedure
tags: [VoLTE, IMS, termination, S-CSCF, P-CSCF, I-CSCF, MGCF, SIP, QoS, PCC, MT, roaming, PSI, SLF]
sources: [ts_123228v160500p.pdf]
updated: 2026-04-10
---

# VoLTE Mobile-Terminated Call

Covers TS 23.228 §5.7–5.8: all Termination procedure variants (MT#1, MT#2, MT#3, PSTN-T,
NI-T, AS-T#1–4), SLF-based HSS resolution (§5.8), mid-session signalling routing (§5.9),
and sessions without preconditions (§5.7a).

Related pages: [S-CSCF](../entities/S-CSCF.md) · [P-CSCF](../entities/P-CSCF.md) ·
[I-CSCF](../entities/I-CSCF.md) · [IMS QoS/Bearer](IMS-QoS-bearer.md) ·
[VoLTE MO Call](VoLTE-MO-call.md) · [Session Release](session-release.md) ·
[IMS Registration](IMS-registration.md)

---

## 1. Termination Procedure General Rules (§5.7.0)

Three architectural constants apply to **all** termination procedures:

1. **Signalling path fixed at registration.** S-CSCF knows the next hop P-CSCF address
   (from the registration procedure). P-CSCF knows the UE address. Both remain fixed for
   the life of the registration.

2. **P-CSCF always present.** Every terminating UE has an associated P-CSCF. The P-CSCF:
   - performs QoS resource authorization on Rx (AAR to PCRF)
   - handles MPS priority session detection and dynamic policy invocation
   - enables media flows on UE answer (gate open)
   - generates CDR for roaming case

3. **PSTN termination is a special case.** The MGCF acts as a SIP endpoint receiving
   sessions on behalf of the PSTN. It uses H.248 to control an MGW and sends IAM/ANM to
   the PSTN via SS7.

---

## 2. MT#1: Mobile Termination, Roaming (§5.7.1)

The UE is in a **visited network**. S-CSCF is in the home network; P-CSCF is in the
visited network. At registration, S-CSCF learned the P-CSCF address in the visited network.

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant S as S-CSCF (home)
    participant P as P-CSCF (visited)
    participant UE
    participant PCRF as PCRF/PCF

    Orig->>S: 1. INVITE (initial SDP offer, via S-S procedure)
    Note over S: 2. Validate service profile;<br/>invoke terminating iFC chain
    Note over S: 3. Look up next hop = P-CSCF in visited (from registration)
    S->>P: 3. INVITE
    Note over P: 4. MPS check: if priority session,<br/>derive session info → dynamic policy → AAR to PCRF<br/>Recall UE address from registration
    P->>UE: 4. INVITE
    UE-->>P: 5. Offer Response (SDP answer — supported media subset)
    P->>PCRF: 6. AAR (Authorize QoS Resources — SDP→IP flows)
    PCRF-->>P: AAA (PCC rules authorized)
    P-->>S: 7. Offer Response
    S-->>Orig: 8. Offer Response (per S-S procedure)

    Orig->>S: 9. Response Confirmation (Opt SDP)
    Note over P: If new SDP in step 9 → re-authorize at step 12
    S->>P: 10. Response Conf (may route via I-CSCF per operator config)
    P->>UE: 11. Response Conf
    UE-->>P: 12. Conf Ack (Opt SDP); if SDP changed P-CSCF re-authorizes
    Note over UE: 13. Resource Reservation<br/>(UE-initiated or IP-CAN-initiated after step 6)
    P-->>S: 14-15. Conf Ack → originator (may route via I-CSCF)

    Orig->>S: 16. Reservation Conf (resource reservation complete)
    S->>P: 17. Reservation Conf
    P->>UE: 18. Reservation Conf
    UE->>UE: 19. Alert User (incoming call ring)
    UE-->>P: 20. Reservation Conf
    P-->>S: 21. Reservation Conf
    S-->>Orig: 22. Reservation Conf

    UE-->>P: 23. Ringing (180)
    P-->>S: 24. Ringing
    S-->>Orig: 25. Ringing

    UE-->>P: 26. 200 OK (destination answers)
    P->>PCRF: 27. Enable Media Flows (gate open)
    UE->>UE: 28. Start Media
    P-->>S: 29-30. 200 OK → originator

    Orig->>S: 31. ACK
    S->>P: 32. ACK
    P->>UE: 33. ACK
```

### Step notes

| Step | Detail |
|---|---|
| 2 | S-CSCF validates GRUU/IMPU mapping; invokes **terminating** iFC chain (may fork to TAS/voicemail AS etc.) |
| 4 | P-CSCF priority: if INVITE contains MPS indication, P-CSCF derives session info and invokes dynamic policy. P-CSCF may also authorize resources at step 4 if originating side already has resources (no SDP answer needed first). |
| 6 | P-CSCF sends AAR on Rx with media component descriptors from UE's SDP answer. |
| 10 | Routing of Response Conf may go through I-CSCF depending on operator configuration. |
| 13 | Resource reservation: UE-initiated (Bearer Resource Allocation Request per TS 23.401) or IP-CAN-initiated (PCRF pushes PCC rules to PCEF). |
| 27 | Gate open: P-CSCF instructs PCRF/PCF to enable media flows — media can only flow after UE answers. |

---

## 3. MT#2: Mobile Termination, Home (§5.7.2)

The UE is in the **home network**. Both S-CSCF and P-CSCF are in the home network.
Structurally identical to MT#1 with one routing difference:

- Step 3: S-CSCF forwards INVITE to the P-CSCF **in the home network** (no visited-to-home hop)

```mermaid
graph LR
    subgraph "Home Network"
        S["S-CSCF"] --> P["P-CSCF"]
        P --> UE
    end
    Orig["Originating Network"] --> S
    P -.->|Rx| PCRF
```

All 33 steps are identical to MT#1 in structure. The only difference is the topology:
P-CSCF and UE are in the same home network as the S-CSCF.

---

## 4. MT#3: Mobile Termination, CS Domain Roaming (§5.7.2a)

Applies when the user has **both IMS and CS subscriptions** but is **unregistered for IMS**
(e.g. the device is a UMTS phone currently attached to CS only).

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant S as S-CSCF
    participant B as BGCF
    participant MGCF

    Orig->>S: 1. INVITE (via S-S procedure)
    Note over S: 2. Service Control:<br/>- Re-route to messaging service, OR<br/>- Route toward CS domain termination address (E.164)
    Note over S: 3. Destination is CS domain → pass to BGCF
    S->>B: INVITE (CS domain address)
    B->>MGCF: 4. INVITE → appropriate MGCF
    MGCF->>MGCF: 5. Normal PSTN-T procedure continues
```

Key notes:
- If no S-CSCF allocated for this user: route per §5.12.1 (unregistered IMS user handling)
- S-CSCF may invoke service logic that redirects the session (e.g. to voicemail) instead of
  routing to the CS domain
- Once the MGCF receives the INVITE, the flow continues as PSTN-T (§5.7.3)

---

## 5. PSTN-T: PSTN Termination (§5.7.3)

The MGCF receives the session from IMS and bridges to the PSTN via SS7 and H.248/MGW.

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant MGCF
    participant MGW
    participant PSTN

    Orig->>MGCF: 1. INVITE (initial SDP offer, via S-S procedure)
    MGCF->>MGW: 2. H.248 — pick outgoing channel + determine MGW media capabilities
    MGCF-->>Orig: 3. Offer Response (supported media subset, per S-S)
    Orig->>MGCF: 4. Response Confirmation (Opt SDP)
    MGCF->>MGW: 5. H.248 — modify connection; reserve resources for media
    MGCF-->>Orig: 6. Conf Ack (Opt SDP)
    MGW->>MGW: 7. Reserve resources
    Orig->>MGCF: 8. Reservation Conf
    MGCF->>PSTN: 9. IAM (Initial Address Message — destination info)
    MGCF-->>Orig: 10. Reservation Conf (per S-S)
    PSTN-->>MGCF: 11. ACM (Address Complete — alerting, optional)
    MGCF-->>Orig: 12. Ringing (provisional 180, via S-S)
    PSTN-->>MGCF: 13. ANM (Answer Message — destination answered)
    MGCF->>MGW: 14. H.248 — make MGW connection bi-directional
    MGCF-->>Orig: 15. 200 OK (via S-S)
    Orig->>MGCF: 16. ACK
```

Key properties:
- MGCF has no P-CSCF role — it is the S-CSCF equivalent; no Rx/PCRF interaction
- IAM is sent only after resource reservation is complete (step 9 after step 8)
- RLC (Release Complete) in PSTN-O corresponds to ACM/ANM in PSTN-T

---

## 6. NI-T: Non-IMS Termination to External SIP Client (§5.7.4)

The originating UE has IMS precondition capability but the external destination **does not**.
The session is set up in two phases:

### Phase 1 — Session with Inactive Media (Figure 5.19b)

```mermaid
sequenceDiagram
    participant UE
    participant P as P-CSCF
    participant S as S-CSCF
    participant Ext as External SIP Client

    UE->>P: 1. INVITE (Supported: precondition; all media inactive)
    P->>S: 2. INVITE
    S->>Ext: 3. INVITE (precondition info forwarded but ignored by Ext)
    Ext-->>S: 4. 200 OK (media inactive — Ext ignores preconditions)
    S-->>P: 5. 200 OK
    Note over P: 6. P-CSCF may authorize QoS resources (operator policy)
    P-->>UE: 7. 200 OK (media inactive)
    UE->>P: 8. ACK
    P->>S: 9. ACK
    S->>Ext: 10. ACK
    Note over UE: 11. UE initiates resource reservation<br/>(session established but media not yet flowing)
```

### Phase 2 — Media Activation (Figure 5.19c)

After resource reservation, UE activates media via re-INVITE:

```mermaid
sequenceDiagram
    participant UE
    participant P as P-CSCF
    participant S as S-CSCF
    participant Ext as External SIP Client

    UE->>P: 12. re-INVITE (media active)
    P->>S: 13. re-INVITE
    S->>Ext: 14. re-INVITE
    Ext-->>S: 15. 200 OK (media active)
    S-->>P: 16. 200 OK
    Note over P: 17. P-CSCF enables media flows (gate open)
    P-->>UE: 18. 200 OK (media active — session fully established)
    UE->>P: 19. ACK
    P->>S: 20. ACK
    S->>Ext: 21. ACK
```

Key property: The two-phase approach ensures the IMS UE can establish and authorize its EPS
bearer before media flows, even though the external client cannot participate in preconditions.

---

## 7. AS-T#1: PSI Based AS Termination — Direct (§5.7.5)

Session destined to a PSI; HSS provides the AS address **directly** to the I-CSCF.

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant I as I-CSCF
    participant HSS
    participant AS

    Orig->>I: 1. INVITE (destined to PSI e.g. sip:game1@home1.net)
    I->>HSS: 2-3. Cx Query (routing info for PSI)
    HSS-->>I: 4-5. Response (AS address hosting the PSI)
    I->>AS: 6-7. INVITE (forwarded directly to AS)
    AS-->>Orig: 8-9. 183 Session Progress (session setup continues)
```

---

## 8. AS-T#2: PSI Based AS Termination — Indirect via S-CSCF (§5.7.6)

Session destined to a PSI; HSS provides S-CSCF address; S-CSCF evaluates iFC to find AS.

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant I as I-CSCF
    participant HSS
    participant S as S-CSCF
    participant AS

    Orig->>I: 1. INVITE (destined to PSI e.g. sip:gamego@home1.net)
    I->>HSS: 2-3. Cx Query
    HSS-->>I: 4-5. S-CSCF address (or capabilities for new selection)
    I->>S: 6-7. INVITE (to S-CSCF)
    Note over S: 8. S-CSCF evaluates iFC for PSI → gets AS address
    S->>AS: 9. INVITE (to AS identified by filter criteria)
    AS-->>Orig: 10-12. Session Progress
```

---

## 9. AS-T#3: PSI Based AS Termination — DNS Routing (§5.7.7)

Session from an external network to a PSI whose subdomain resolves via DNS to an AS.

```mermaid
sequenceDiagram
    participant Ext as External Network
    participant I as I-CSCF
    participant DNS
    participant AS

    Ext->>I: 1. INVITE (sip:fungame1@as1.home1.net)
    Note over I: 2. I-CSCF checks domain list; finds match for as1.home1.net<br/>3. Queries DNS for IP address of as1.home1.net
    I->>DNS: 4. DNS query
    DNS-->>I: 5. IP address of AS
    I->>AS: 6-7. INVITE (to DNS-resolved AS address)
    AS-->>Ext: 8-9. Session Progress
```

Key: I-CSCF uses its configured domain list and DNS to route without HSS interaction.
This allows external networks to reach PSIs via subdomain-based routing.

---

## 10. AS-T#4: AS Termination Based on Service Logic (§5.7.8)

Session destined to an IMS user but the S-CSCF's iFC diverts it to an AS, which
**terminates the session itself** rather than forwarding to the UE.

```mermaid
sequenceDiagram
    participant Orig as Originating Network
    participant I as I-CSCF
    participant HSS
    participant S as S-CSCF
    participant AS

    Orig->>I: 1. INVITE (destined to user sip:user2@home2.net)
    I->>HSS: 2-3. Cx Query
    HSS-->>I: 4-5. S-CSCF address
    I->>S: 6-7. INVITE
    Note over S: 8. S-CSCF evaluates iFC for user → AS intercepts<br/>(e.g. call barring, unconditional forward, voicemail)
    S->>AS: 9. INVITE (AS terminates — does NOT forward to UE)
    AS-->>Orig: 10-12. Session setup continues (with AS as endpoint)
```

---

## 11. Sessions Without Preconditions (§5.7a)

**Use case:** Services without real-time QoS requirements before session activation —
instant messaging, presence, or calls where existing default bearers suffice.

**Key difference from §5.6/§5.7:** No dedicated IP-CAN bearer is required before
the session becomes active. Resource reservation can happen concurrently with or after
the session is answered.

### §5.7a.2: End-to-End Without Preconditions (Figure 5.19h)

Full cross-operator flow (UE#1 → P-CSCF#1 → S-CSCF#1 → I-CSCF#2 → S-CSCF#2 → P-CSCF#2 → UE#2):

```mermaid
sequenceDiagram
    participant UE1 as UE#1
    participant P1 as P-CSCF#1
    participant S1 as S-CSCF#1
    participant I2 as I-CSCF#2
    participant S2 as S-CSCF#2
    participant P2 as P-CSCF#2
    participant UE2 as UE#2

    UE1->>P1: 1. INVITE (SDP offer, no preconditions)
    Note over P1: 2. Media policy check (bandwidth vs PCRF limits)
    P1->>S1: 3. INVITE
    Note over S1: 4. Service control (originating iFC)
    S1->>I2: 5. INVITE
    I2->>I2: 6. HSS query (LIR → S-CSCF#2 address)
    I2->>S2: 7. INVITE
    Note over S2: 8. Service control (terminating iFC)
    S2->>P2: 9. INVITE
    Note over P2: 10. Media policy check
    P2->>UE2: 11. INVITE
    UE2-->>P2: 12-13. 180 Ringing (optional)
    P2-->>UE1: 14-17. Ringing propagated
    Note over UE2: 18. Reserve IP-CAN bearer for media<br/>(may happen before or after answering)
    UE2-->>P2: 19. 200 OK (Offer Response / SDP answer)
    Note over P2: 20. Authorize QoS resources (AAR on Rx, operator policy)
    P2-->>S2: 21. 200 OK
    S2-->>I2: 22. 200 OK
    I2-->>S1: 23. 200 OK
    S1-->>P1: 24. 200 OK
    Note over P1: 25. Authorize QoS resources (AAR on Rx, operator policy)
    P1-->>UE1: 26. 200 OK
    UE1->>P1: 27. ACK
    P1->>S1: 28. ACK
    S1->>I2: 29. ACK
    I2->>S2: 30. ACK
    S2->>P2: 31. ACK (→ UE#2)
    Note over UE1: 32. Reserve IP-CAN bearer for media<br/>(based on SDP answer from step 26)
```

**Comparison with precondition flows:**

| Feature | With preconditions (§5.6/5.7) | Without preconditions (§5.7a) |
|---|---|---|
| Bearer before ringing | Required | Not required |
| QoS authorization timing | Before Offer Response | After 200 OK (operator policy) |
| Ringing timing | After resource reservation confirmed | May happen immediately |
| Bearer reservation | Before 200 OK | Before or after 200 OK |
| Use case | VoLTE voice/video | IM, presence, best-effort calls |

---

## 12. SLF-Based HSS Resolution (§5.8)

When multiple independently addressable HSSs are deployed, the **SLF (Subscription Locator
Function)** resolves a user identity to the specific HSS holding that user's data.

### Interfaces

| Interface | Between | Operation |
|---|---|---|
| **Dx** | I-CSCF or S-CSCF ↔ SLF | `DX_SLF_QUERY` (identity) → `DX_SLF_RESP` (HSS name) |
| **Dh** | AS ↔ SLF | `DH_SLF_QUERY` (IMPU) → `DH_SLF_RESP` (HSS name) |

### SLF on REGISTER (§5.8.2)

```mermaid
sequenceDiagram
    participant P as P-CSCF
    participant I as I-CSCF
    participant SLF
    participant HSS

    P->>I: REGISTER
    Note over I: Case 1: I-CSCF queries SLF directly
    I->>SLF: DX_SLF_QUERY (IMPU from REGISTER)
    SLF->>SLF: Database lookup
    SLF-->>I: DX_SLF_RESP (HSS name)
    I->>HSS: Cx Query (correct HSS)
```

Case 2 (Figure 5.20a): I-CSCF forwards REGISTER to S-CSCF; S-CSCF queries SLF
(`DX_SLF_QUERY`) and resolves the correct HSS itself.

### SLF on UE INVITE (§5.8.3)

```mermaid
sequenceDiagram
    participant xCSCF as x-CSCF
    participant I as I-CSCF
    participant SLF
    participant HSS

    xCSCF->>I: INVITE
    I->>SLF: DX_SLF_QUERY (IMPU; E.164 converted to Tel URI per RFC 3966)
    SLF-->>I: DX_SLF_RESP (HSS name)
    I->>HSS: Cx Query
```

**E.164 note:** If the called identity is an E.164 number in `sip:+12345@home.net;user=phone`
format, I-CSCF must convert it to `tel:+12345` before sending `DX_SLF_QUERY`.

### SLF on AS Access to HSS (§5.8.4)

```mermaid
sequenceDiagram
    participant AS
    participant SLF
    participant HSS

    AS->>SLF: DH_SLF_QUERY (IMPU)
    SLF-->>AS: DH_SLF_RESP (HSS name)
    AS->>HSS: Sh message (SH_PULL / SH_UPDATE / SH_SUBS_NOTIF)
```

AS may cache the HSS name for subsequent Sh operations.

---

## 13. Mid-Session Signalling Routing (§5.9)

Four nodes **must** remain on the signalling path for the life of an established session:

| Node | Why it must stay in path |
|---|---|
| **P-CSCF (originating)** | CDR generation (roaming); force resource release at session end |
| **S-CSCF (originating)** | Service logic at session completion; CDR at termination |
| **S-CSCF (terminating)** | Service logic at session completion; CDR at termination |
| **P-CSCF (terminating)** | CDR generation (roaming); force resource release |

Additional rules:
- I-CSCFs may optionally remain in path if performing mid-session or session-clearing functions
- **All UE signalling** (re-INVITE, BYE, REFER, etc.) must be sent to the P-CSCF
- Session path is recorded via `Record-Route` headers during INVITE/200 OK

---

## Cross-References

| Topic | Page |
|---|---|
| Session Release (BYE, network-initiated, PSTN release) | [Session Release](session-release.md) |
| Session Hold/Resume | [Session Release](session-release.md) |
| VoLTE MO Call (MO#1, MO#2, PSTN-O) | [VoLTE MO Call](VoLTE-MO-call.md) |
| IMS QoS/PCC interactions (Rx, gate control) | [IMS QoS/Bearer](IMS-QoS-bearer.md) |
| S-CSCF iFC evaluation | [S-CSCF](../entities/S-CSCF.md) |
| I-CSCF: THIG, HSS query, S-CSCF selection | [I-CSCF](../entities/I-CSCF.md) |
| HSS Cx/Sh subscriber data | [HSS](../entities/HSS.md) |
