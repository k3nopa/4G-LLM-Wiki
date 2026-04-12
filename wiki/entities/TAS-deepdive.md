---
title: "TAS Deep-Dive — Telephony Application Server Comprehensive Reference"
type: entity
tags: [TAS, IMS, AS, MMTEL, ISC, Sh, Mr, Ut, B2BUA, VoLTE, call-forwarding, conference, MRFC, iFC, deep-dive]
sources: [ts_123228v150600p.pdf, ts_123218v150600p.pdf]
updated: 2026-04-11
---

# TAS Deep-Dive — Telephony Application Server

**Base entity page:** [TAS.md](TAS.md)
**Spec references:** TS 23.228 §4.13, §5.5–§5.8; TS 23.218 §9, Annexes B/C; TS 24.173 (MMTEL services)

---

## Architectural Position

The TAS is the **telephony service execution engine** of IMS. It is triggered by the S-CSCF's iFC evaluation and applies MMTEL supplementary services (call forwarding, barring, hold, conference, etc.) before returning control to the S-CSCF for final routing. It is the only node with access to subscriber telephony preferences (via Sh) and the only node that directly controls media resources (via Mr to MRFC). It can operate in all five AS modes depending on the service being applied.

```mermaid
graph LR
    subgraph "S-CSCF"
        SCSCF["S-CSCF\n(iFC evaluation)"]
    end
    subgraph "TAS"
        TAS_node["TAS\n(MMTEL services\n+ B2BUA logic)"]
    end
    subgraph "Data"
        HSS["HSS"]
    end
    subgraph "Media"
        MRFC["MRFC"]
    end
    subgraph "UE Self-Service"
        UE["UE (XCAP)"]
    end
    SCSCF -->|"ISC (SIP)"| TAS_node
    TAS_node -->|"ISC (SIP)"| SCSCF
    TAS_node -->|"Sh (Diameter)"| HSS
    TAS_node -->|"Mr (SIP)"| MRFC
    UE -->|"Ut (HTTP/XCAP)"| TAS_node
```

---

## Complete Interface Table

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| **ISC** | S-CSCF | SIP (TCP/TLS) | Bidirectional | Receive iFC-triggered requests; return to S-CSCF via ODI; originate new requests (originating UA mode) |
| **Sh** | HSS | Diameter (Sh app) | Bidirectional | Read/write subscriber telephony service data; subscribe to change notifications |
| **Mr** | MRFC | SIP | TAS → MRFC | Request conference, announcement, and transcoding media resources |
| **Mr'** | MRFC | SIP | TAS → MRFC | Direct MRFC access bypassing S-CSCF (used for transcoding, conference in B2BUA mode) |
| **Ut** | UE | HTTP(S) / XCAP | UE → TAS | Subscriber self-service: configure call forwarding targets, barring, preferences |
| **Dh** | SLF | Diameter (Dh app) | TAS → SLF | Multi-HSS: locate correct HSS for Sh queries |
| **Cr** | MRFC | Media control | TAS → MRFC | Control channel for media server operations (per RFC 6230 / MEDIACTRL) |

---

## Sh Interface — Subscriber Data Access

The Sh interface is TAS's primary data source for service decisions. It contains all subscriber telephony preferences.

| Data Element | Read/Write | Purpose |
|---|---|---|
| Call Forwarding Unconditional (CFU) target | R/W | If CFU active: forward all calls to this URI |
| Call Forwarding No Reply (CFNRy) target + timer | R/W | Forward on no answer after N seconds |
| Call Forwarding Busy (CFB) target | R/W | Forward when UE reports busy (486) |
| Call Forwarding Not Reachable (CFNRc) target | R/W | Forward when UE unreachable (480, 408) |
| Call Barring flags | R/W | OIR/BOIC/BAIC/etc. barring status per service |
| CLIR preference (permanent/temporary) | R/W | Default CLIR mode for originating calls |
| Call Waiting status | R/W | Whether call waiting is active |
| Message Waiting Indication flags | R/W | Pending voicemail count (read by MWI service) |
| MSISDN | R | UE's phone number (used in P-Asserted-Identity insertion) |
| IMPU list | R | All registered public identities |

### Sh Message Flow for Service Data Fetch

```mermaid
sequenceDiagram
    participant SCSCF as S-CSCF
    participant TAS
    participant HSS

    SCSCF->>TAS: ISC INVITE (iFC triggered, originating)
    TAS->>HSS: Sh UDR (IMPU, Data-Reference: IFC_related_data)
    HSS-->>TAS: Sh UDA (service data: CFB=active, target=+441234)
    TAS->>TAS: Apply service logic\n(check call forwarding, barring, etc.)
    TAS-->>SCSCF: ISC INVITE (returned via ODI — no forward needed)\nor 302 Redirect / re-INVITE to forward target
```

**Caching:** TAS typically caches Sh data per subscriber to avoid Sh round-trip on every call. TAS subscribes to Sh change notifications (SNR) so HSS pushes updates when subscriber modifies preferences via Ut/XCAP.

### Sh Notification Subscription

```mermaid
sequenceDiagram
    participant TAS
    participant HSS
    participant UE

    TAS->>HSS: Sh SNR (IMPU, Data-Reference list, subscribe all changes)
    HSS-->>TAS: Sh SNA (subscribed)
    Note over UE: UE changes CFU target via XCAP (Ut)
    UE->>TAS: XCAP PUT /call-forwarding/unconditional/target = +447890
    TAS->>HSS: Sh PUR (IMPU, updated CFU target)
    HSS-->>TAS: Sh PUA
    HSS->>TAS: Sh PNR (IMPU, updated data — in case another system changed it)
    TAS-->>HSS: Sh PNA
```

---

## AS Operating Modes — Detailed

### Mode 1: SIP Proxy

TAS is transparent in the signaling path. Used for: CLIR (strip From header), P-Asserted-Identity manipulation, call barring checks.

```mermaid
sequenceDiagram
    participant SCSCF as S-CSCF
    participant TAS
    SCSCF->>TAS: INVITE (ODI in Route)
    TAS->>TAS: Check barring — allowed\nRemove P-Asserted-Identity (CLIR)
    TAS-->>SCSCF: INVITE (via ODI route, modified headers)
    Note over SCSCF: Resume iFC chain
```

### Mode 2: B2BUA — Call Hold / Conference

TAS creates two independent SIP dialogs: one toward the S-CSCF/UE side, one toward the media resource or second party.

```mermaid
sequenceDiagram
    participant UE1
    participant SCSCF as S-CSCF
    participant TAS
    participant MRFC

    UE1->>SCSCF: re-INVITE (a=inactive — hold request)
    SCSCF->>TAS: re-INVITE (ISC, iFC triggered)
    TAS->>TAS: Recognize hold request\nLeg 1: confirm hold with UE1\nLeg 2: inject hold music from MRFC
    TAS->>MRFC: INVITE (Mr' — request hold tone)
    MRFC-->>TAS: 200 OK (SDP: MRFP RTP address)
    TAS-->>SCSCF: 200 OK re-INVITE (SDP: a=inactive toward UE1)
    SCSCF-->>UE1: 200 OK (held)
    Note over TAS,MRFC: Hold music flows UE1 ← MRFP
```

### Mode 3: Redirect (302)

TAS sends 302 Moved Temporarily toward S-CSCF. Used for call forwarding (UA redirect mode).

```mermaid
sequenceDiagram
    participant SCSCF as S-CSCF
    participant TAS

    SCSCF->>TAS: INVITE (terminating, iFC triggered)
    TAS->>TAS: CFB check: UE busy (received 486)
    TAS-->>SCSCF: 302 Moved Temporarily\nContact: sip:+441234@voicemail.ims
    SCSCF->>SCSCF: Re-route INVITE to Contact URI
```

### Mode 4: B2BUA — Conference (Multi-party)

```mermaid
sequenceDiagram
    participant UE1
    participant UE2
    participant UE3
    participant SCSCF as S-CSCF
    participant TAS
    participant MRFC

    Note over TAS: UE1 invites UE2, TAS invoked as originating B2BUA
    TAS->>MRFC: INVITE (Mr' — create conference bridge, conf-ID=X)
    MRFC-->>TAS: 200 OK (SDP: bridge RTP)
    TAS-->>UE1: 200 OK (SDP: bridge RTP) [Call-ID 1]
    TAS->>SCSCF: INVITE UE2 [Call-ID 2] (originating UA mode)
    UE2-->>TAS: 200 OK
    TAS->>MRFC: INVITE (Mr' — add UE2 to conf-ID=X) [Call-ID 4]
    Note over TAS: Same conf-ID reused → MRFC adds party to bridge
    TAS->>SCSCF: INVITE UE3 [Call-ID 5] (add third party)
    UE3-->>TAS: 200 OK
    TAS->>MRFC: INVITE (Mr' — add UE3 to conf-ID=X) [Call-ID 6]
    Note over MRFC: Three-party audio bridge active
```

### Mode 5: Originating UA — MWI / Click-to-Dial

TAS generates a new SIP request without being triggered by an incoming dialog:

```mermaid
sequenceDiagram
    participant TAS
    participant SCSCF as S-CSCF
    participant UE

    Note over TAS: Event: new voicemail deposited\n(triggered by voicemail AS notification)
    TAS->>SCSCF: SUBSCRIBE sip:+1234@home.ims\n(originating UA mode — no incoming INVITE)
    SCSCF->>SCSCF: Evaluate originating iFCs\n(SUBSCRIBE, Direction=Orig)
    SCSCF->>UE: SUBSCRIBE (via P-CSCF)
    UE-->>SCSCF: 200 OK
    TAS->>UE: NOTIFY (Message-Waiting: yes; Messages-Waiting: 2)
    UE-->>TAS: 200 OK
    Note over UE: MWI indicator shown on phone
```

---

## MMTEL Service Logic

### Call Forwarding Decision Tree

```mermaid
flowchart TD
    incoming["Terminating INVITE\nfrom S-CSCF (iFC triggered)"] --> sh["Fetch Sh data:\nCFU/CFB/CFNRy/CFNRc status + targets"]
    sh --> cfu{CFU active?}
    cfu -->|Yes| fwd_cfu["Forward to CFU target\n(302 or re-INVITE)"]
    cfu -->|No| forward["Forward INVITE to UE\n(return to S-CSCF via ODI → route to UE)"]
    forward --> ue_resp{UE Response?}
    ue_resp -->|"486 Busy"| cfb{CFB active?}
    ue_resp -->|"No reply (timer)"| cfnry{CFNRy active?}
    ue_resp -->|"408/480 Unreachable"| cfnrc{CFNRc active?}
    ue_resp -->|"200 OK"| connected["Call connected"]
    cfb -->|Yes| fwd_cfb["Forward to CFB target"]
    cfb -->|No| reject["Return 486 to caller"]
    cfnry -->|Yes| fwd_cfnry["Forward to CFNRy target"]
    cfnry -->|No| timeout["Return 408 to caller"]
    cfnrc -->|Yes| fwd_cfnrc["Forward to CFNRc target"]
    cfnrc -->|No| unreach["Return 480 to caller"]
```

### Call Barring Checks (Originating)

On an originating INVITE:
1. Fetch Sh barring flags: BOIC (Barring Outgoing International Calls), BOIC-exHC (except Home Country), BAIC (Barring All Incoming), OIR (Originating Identification Restriction)
2. Check dialed number against barring rules
3. If barred: return 403 Forbidden toward S-CSCF (which returns it to UE)
4. If allowed: return INVITE to S-CSCF via ODI

### CLIR (Calling Line Identification Restriction)

```mermaid
sequenceDiagram
    participant SCSCF as S-CSCF
    participant TAS

    SCSCF->>TAS: INVITE (P-Asserted-Identity: sip:+1234@home.ims, ODI in Route)
    TAS->>HSS: Sh UDR (CLIR preference)
    HSS-->>TAS: Sh UDA (CLIR=permanent)
    TAS->>TAS: CLIR active → remove P-Asserted-Identity\nAdd Privacy: id header
    TAS-->>SCSCF: INVITE (P-Asserted-Identity removed, Privacy: id — via ODI)
```

### Call Waiting

When UE is in an active call (has a SIP dialog in `confirmed` state) and a second INVITE arrives:
1. TAS detects: UE has active dialog
2. If CW enabled: forward INVITE to UE (UE rings while first call active)
3. UE sends HOLD on first call, accepts second via re-INVITE

---

## Voicemail Integration

TAS is typically co-located with or tightly coupled to the Voicemail AS. Two key flows:

### Voicemail Deposit (No-Reply Forward)

1. Terminating INVITE arrives at S-CSCF
2. iFC routes to TAS
3. TAS forwards to UE — no answer (CFNRy timer expires)
4. TAS re-routes to Voicemail AS (terminating UA mode)
5. Caller hears greeting → leaves message
6. Voicemail AS notifies TAS of new message

### Voicemail MWI (On-Registration)

Triggered by third-party REGISTER iFC:
1. UE registers → S-CSCF evaluates REGISTER iFCs
2. iFC `Registration-Type=INITIAL` matches → S-CSCF forks third-party REGISTER to TAS
3. TAS fetches subscriber Sh data: checks `Messages-Waiting` count
4. If voicemails waiting: TAS generates NOTIFY (originating UA) toward UE
5. UE displays MWI indicator

---

## Transcoding via MRFC (B2BUA)

When called UA cannot match calling UA's codec:

```mermaid
sequenceDiagram
    participant UE1
    participant SCSCF as S-CSCF
    participant TAS
    participant MRFC

    UE1->>SCSCF: INVITE (SDP: AMR-WB only)
    SCSCF->>TAS: INVITE (iFC triggered, transcoding service)
    TAS->>TAS: Detect codec mismatch\nNeed transcoding
    TAS->>MRFC: INVITE (Mr' — AMR-WB side) [Call-ID 3]
    MRFC-->>TAS: 200 OK (SDP: MRFP RTP addr for AMR-WB)
    TAS->>TAS: Query MRFC for codec list [Call-ID 4 — no SDP]
    MRFC-->>TAS: 200 OK (supported codec list including G.711)
    TAS-->>UE1: 183 Session Progress (SDP: MRFP RTP addr for AMR-WB)
    Note over TAS: Route INVITE to called UA with G.711 SDP
    TAS->>SCSCF: INVITE UE2 (SDP: G.711) [Call-ID 5]
    SCSCF->>UE1: Note via TAS B2BUA handling...
    Note over TAS,MRFC: MRFP bridges AMR-WB ↔ G.711 in real-time
```

---

## TAS Data Model

```mermaid
erDiagram
    SUBSCRIBER_CONTEXT {
        string IMPU
        string IMPI
        string MSISDN
        timestamp sh_cache_fetched
    }
    CALL_FORWARDING {
        bool CFU_active
        string CFU_target
        bool CFB_active
        string CFB_target
        bool CFNRy_active
        string CFNRy_target
        int CFNRy_timer_seconds
        bool CFNRc_active
        string CFNRc_target
    }
    BARRING {
        bool BOIC_active
        bool BOIC_exHC_active
        bool BAIC_active
        bool BIC_Roam_active
        string PIN_code
    }
    CALL_PREFERENCES {
        string CLIR_mode
        bool CW_active
        bool ECT_active
        bool conference_active
    }
    MWI_STATUS {
        int messages_waiting_count
        int urgent_messages
        timestamp last_notified
    }
    ACTIVE_SESSION {
        string Call_ID
        string direction
        string remote_party_URI
        string AS_mode
        string MRFC_dialog_ID
        string conference_ID
    }
    SUBSCRIBER_CONTEXT ||--|| CALL_FORWARDING : "has"
    SUBSCRIBER_CONTEXT ||--|| BARRING : "has"
    SUBSCRIBER_CONTEXT ||--|| CALL_PREFERENCES : "has"
    SUBSCRIBER_CONTEXT ||--|| MWI_STATUS : "has"
    SUBSCRIBER_CONTEXT ||--o{ ACTIVE_SESSION : "has (one per call leg)"
```

---

## Procedure Participation Summary

| Procedure | TAS Role | Mode | Key Action |
|---|---|---|---|
| VoLTE MO call (originating) | Apply MMTEL originating services | SIP proxy or B2BUA | CLIR, call barring check, CLIP insertion |
| VoLTE MT call (terminating) | Apply MMTEL terminating services | SIP proxy or Redirect | CFU/CFB/CFNRy/CFNRc check; forward or pass through |
| Call hold / resume | Hold music injection | B2BUA | Fork to MRFC for hold tone; control both legs |
| Multi-party conference | Conference anchor | B2BUA | Establish MRFC bridge; add parties one by one via Mr' |
| IMS registration (third-party) | MWI check | Originating UA | Fetch Sh MWI count; send NOTIFY to UE if messages waiting |
| Call transfer (ECT) | Transfer controller | B2BUA | Attended: establish consultation call; transfer target; release consult |
| Network-initiated de-register | Session cleanup | — | Receive notification; tear down active sessions |
| Voicemail deposit | Call terminator | Terminating UA | Accept call, play greeting, record message |
| Voicemail playback | Session originator | Originating UA | INVITE to UE; play messages via MRFC |
| DTMF collection | IVR controller | B2BUA | Receive RFC 2833 / SIP INFO DTMF; interact with MRFP |

---

## Ut Interface — Subscriber Self-Service

The Ut interface lets UEs manage their own service settings via standard HTTP/XCAP:

```mermaid
flowchart TD
    UE_App["UE application\n(XCAP client)"] -->|"HTTPS PUT /xcap/call-forwarding/..."| TAS_XCAP["TAS XCAP server\n(Ut interface)"]
    TAS_XCAP --> validate["Validate request\n(auth, schema, permissions)"]
    validate --> shPUR["Sh PUR to HSS\n(update service data)"]
    shPUR --> HSS["HSS stores updated\nservice data"]
    TAS_XCAP -->|"200 OK"| UE_App
```

XCAP resource URIs (examples):
- `/xcap/call-forwarding/unconditional/forwarded-to/` — CFU target
- `/xcap/call-barring/outgoing-international/` — BOIC status
- `/xcap/communication-waiting/` — Call Waiting flag

TAS acts as the XCAP server and writes changes through to HSS via Sh PUR.

---

## Failure and Overload Behavior

```mermaid
stateDiagram-v2
    [*] --> Normal
    Normal --> ShDown : HSS Sh interface unreachable
    ShDown --> Normal : Sh reconnected
    ShDown --> ShDown : Use cached Sh data (if available)\nIf no cache: apply default service (no forwarding, no barring)\nNew calls may complete without supplementary services
    Normal --> MrDown : MRFC unreachable
    MrDown --> Normal : MRFC recovered
    MrDown --> MrDown : Conference calls fail\nHold music unavailable (hold still works — just silence)\nTranscoding unavailable
    Normal --> ISCError : S-CSCF ISC request malformed or unexpected
    ISCError --> Normal : Normal ISC flow resumes
    ISCError --> ISCError : TAS returns 500 or 480\nS-CSCF Default Handling applies:\nCONTINUE = call proceeds without service\nSESSION_TERMINATED = call fails
    Normal --> Overloaded : High ISC request rate
    Overloaded --> Normal : Load decreases
    Overloaded --> Overloaded : Return 503 on ISC\nS-CSCF Default Handling applies
```

---

## Configuration Parameters

| Parameter | Description |
|---|---|
| ISC listening address | SIP URI for S-CSCF to route ISC requests to this TAS |
| S-CSCF realm | SIP domain/realm of the S-CSCF pool (for originating UA mode routing) |
| HSS Sh realm | Diameter realm/hostname for Sh queries |
| SLF address | SLF hostname (multi-HSS deployments for Dh) |
| MRFC address (Mr') | MRFC SIP URI for direct media resource requests |
| Sh cache TTL | How long to cache subscriber service data before re-fetching |
| Sh subscription mode | Subscribe-all (push) vs on-demand UDR per call |
| CFNRy timer default | Default no-reply timer (seconds) before forwarding |
| XCAP base URI | Base path for Ut XCAP resource tree |
| MWI poll interval | How often to check for new voicemail if no push notification |
| Conference media profile | Default codec / MRFC profile for conference bridges |
| Barring PIN enforcement | Whether PIN is required for barring activation/deactivation |
| Default CLIR | Permanent/temporary CLIR default if Sh data unavailable |

---

## Key Architectural Properties

| Property | Details |
|---|---|
| **Service data owner** | TAS is the runtime consumer of subscriber service data; HSS is the persistent store. The Sh interface bridges them |
| **Closest to subscriber intent** | TAS applies per-subscriber service preferences; all other IMS nodes apply operator-level routing logic |
| **Multi-mode flexibility** | TAS operates in all 5 AS modes within a single call (e.g. proxy for CLIR, then B2BUA for hold, then redirect for forwarding) |
| **Media controller** | TAS is the only node above the MRFC that issues SIP requests to MRFC (via Mr/Mr'). MRFP is controlled exclusively via H.248 from MRFC |
| **XCAP server** | TAS serves as the XCAP server for telephony service settings, translating HTTP XCAP operations into Sh PUR toward HSS |
| **Stateful per-call** | TAS maintains per-call leg state for B2BUA sessions (conference IDs, hold state, transfer state) |

---

## Cross-References

| Topic | Page |
|---|---|
| TAS base entity | [entities/TAS.md](TAS.md) |
| S-CSCF (ISC triggering) | [entities/S-CSCF.md](S-CSCF.md) |
| S-CSCF deep-dive | [entities/S-CSCF-deepdive.md](S-CSCF-deepdive.md) |
| HSS (Sh data source) | [entities/HSS.md](HSS.md) |
| HSS deep-dive | [entities/HSS-deepdive.md](HSS-deepdive.md) |
| MRFC/MRFP (media resources) | [entities/MRF.md](MRF.md) |
| MRB (resource broker) | [entities/MRB.md](MRB.md) |
| IMS Identity Model | [concepts/IMS-identity-model.md](../concepts/IMS-identity-model.md) |
| IM Call Model (iFC/ODI) | [concepts/IM-call-model.md](../concepts/IM-call-model.md) |
| AS interaction modes | [concepts/AS-interaction-modes.md](../concepts/AS-interaction-modes.md) |
| iFC worked examples | [analyses/iFC-worked-examples.md](../analyses/iFC-worked-examples.md) |
| VoLTE MO call | [procedures/VoLTE-MO-call.md](../procedures/VoLTE-MO-call.md) |
| VoLTE MT call | [procedures/VoLTE-MT-call.md](../procedures/VoLTE-MT-call.md) |
| IMS registration | [procedures/IMS-registration.md](../procedures/IMS-registration.md) |
| Session release | [procedures/session-release.md](../procedures/session-release.md) |
| IMS reference points | [interfaces/IMS-reference-points.md](../interfaces/IMS-reference-points.md) |
