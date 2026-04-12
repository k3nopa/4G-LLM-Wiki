---
title: "P-CSCF Deep-Dive — Proxy-CSCF Comprehensive Reference"
type: entity
tags: [P-CSCF, IMS, SIP, Gm, Mw, Rx, IPsec, AF, VoLTE, preconditions, emergency, SigComp, deep-dive]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-11
---

# P-CSCF Deep-Dive — Proxy Call Session Control Function

**Base entity page:** [P-CSCF.md](P-CSCF.md)
**Spec references:** TS 23.228 §4.6, §5.1–§5.4, §5.10–§5.11

---

## Architectural Position

The P-CSCF is the **UE's sole entry point into IMS** and the **IMS's sole entry point toward EPC bearer policy**. It sits between two worlds: the SIP/IMS domain (where it is a proxy) and the EPC policy domain (where it is an Application Function). Every IMS signaling message to and from the UE passes through it; every QoS request toward the EPC passes through its Rx interface.

```mermaid
graph LR
    subgraph "UE"
        UE_SIP["UE SIP stack\n(IMS client)"]
    end
    subgraph "P-CSCF"
        PCSCF["P-CSCF\n(SIP Proxy + AF)"]
    end
    subgraph "IMS Core"
        ICSCF["I-CSCF"]
        SCSCF["S-CSCF"]
        TAS["TAS / AS"]
    end
    subgraph "EPC Policy"
        PCRF["PCRF"]
    end
    UE_SIP -->|"Gm (SIP over IPsec)"| PCSCF
    PCSCF -->|"Mw (SIP)"| ICSCF
    PCSCF -->|"Mw (SIP)"| SCSCF
    PCSCF -->|"Rx (Diameter)"| PCRF
```

**P-CSCF location:** Always in the **visited PLMN** (VPLMN) — even for home-routed roaming. The UE's SIP traffic always exits locally to the VPLMN P-CSCF; only the IMS core (I/S-CSCF) may be in the HPLMN.

---

## Complete Interface Table

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| **Gm** | UE | SIP (UDP/TCP) + IPsec ESP | Bidirectional | UE ↔ IMS: all SIP registration and session messages; protected by IPsec SA |
| **Mw** | I-CSCF, S-CSCF | SIP (TCP/TLS) | Bidirectional | Forward SIP within IMS core; route REGISTER to I-CSCF, sessions to S-CSCF |
| **Rx** | PCRF | Diameter (Rx app) | Bidirectional | AF session: deliver media descriptions; authorize/gate QoS bearers; receive bearer event notifications |

---

## P-CSCF Discovery

The UE must discover its P-CSCF address before IMS registration. Three mechanisms:

```mermaid
flowchart TD
    attach["EPS Attach / PDN Connectivity\n(IMS APN)"] --> PCO["PCO (Protocol Config Options)\nin Activate Default EPS Bearer\n→ PGW provides P-CSCF address"]
    attach --> DHCP["DHCP on IMS APN\n→ P-CSCF FQDN in option 120"]
    DHCP --> DNS["DNS NAPTR/SRV\n→ resolve FQDN to IP"]
    PCO --> use["UE uses P-CSCF address\nfor SIP REGISTER"]
    DNS --> use
```

PCO is the most common in LTE deployments — the PGW returns P-CSCF address(es) directly in the bearer setup response. DHCP/DNS provides a fallback and is used in more complex WLAN/IMS deployments.

---

## SIP Messages Handled

### Registration

| SIP Message | Direction | P-CSCF Action |
|---|---|---|
| REGISTER | UE → P-CSCF → I-CSCF | Add `P-Visited-Network-ID`, `P-Access-Network-Info`; forward to I-CSCF via Mw |
| 401 Unauthorized | I-CSCF → P-CSCF → UE | Forward IMS AKA challenge; save auth params for IPsec SA negotiation |
| REGISTER (with auth) | UE → P-CSCF → S-CSCF | Forward authenticated REGISTER; IPsec SA established at this point |
| 200 OK (REGISTER) | S-CSCF → P-CSCF → UE | Forward; extract `P-Associated-URI` list; store S-CSCF address for routing |
| de-REGISTER | UE → P-CSCF → S-CSCF | Forward de-registration; clear IPsec SA; clear Rx session if active |

**P-CSCF stores after successful registration:**
- UE's IPsec SA parameters (SPI, keys, ports)
- S-CSCF address (for routing subsequent session requests)
- Registered IMPU(s) for this UE
- Registration expiry timer

### Session (INVITE / BYE / UPDATE / PRACK)

| SIP Message | Direction | P-CSCF Action |
|---|---|---|
| INVITE | UE → P-CSCF → S-CSCF | Parse SDP offer; send Rx AAR (gate=DISABLED); forward INVITE |
| 183 Session Progress | S-CSCF → P-CSCF → UE | Parse SDP answer; update Rx AAR with final 5-tuple; open gate if preconditions met |
| PRACK | UE → P-CSCF → S-CSCF | Forward; part of reliable provisional response (100rel) |
| UPDATE | UE → P-CSCF → S-CSCF | SDP renegotiation; may trigger Rx AAR update |
| 200 OK (INVITE) | S-CSCF → P-CSCF → UE | Forward; send Rx AAR update (gate=ENABLED) if preconditions complete |
| ACK | UE → P-CSCF → S-CSCF | Forward; session established |
| BYE | UE/network → P-CSCF | Forward; send Rx STR → PCRF removes PCC rules → PGW deletes GBR bearer |
| CANCEL | UE → P-CSCF → S-CSCF | Forward; send Rx STR (abort pending authorization) |
| re-INVITE | Either → P-CSCF | Forward; update Rx session with new SDP if media changes |

---

## IPsec Security Architecture

The P-CSCF is the IMS security gateway. During IMS registration it negotiates IPsec SAs with the UE to protect all subsequent SIP signaling.

```mermaid
sequenceDiagram
    participant UE
    participant PCSCF as P-CSCF

    Note over UE,PCSCF: Step 1 — REGISTER with Security-Client header
    UE->>PCSCF: REGISTER\nSecurity-Client: ipsec-3gpp;\n  alg=hmac-sha-1-96; ealg=aes-cbc;\n  spi-c=12345; spi-s=67890;\n  port-c=5100; port-s=5101

    Note over UE,PCSCF: Step 2 — Forward challenge (IMS AKA via S-CSCF/HSS)
    PCSCF->>UE: 401 Unauthorized\nSecurity-Server: ipsec-3gpp;\n  alg=hmac-sha-1-96; ealg=aes-cbc;\n  spi-c=11111; spi-s=22222;\n  port-c=4500; port-s=4501;\nWWW-Authenticate: IMS AKA challenge

    Note over UE,PCSCF: Both sides derive keys from IMS AKA (CK+IK)\nFour SAs established: UE→PCSCF (protect+unprotect), PCSCF→UE (protect+unprotect)
    Note over UE,PCSCF: Step 3 — Protected REGISTER
    UE->>PCSCF: REGISTER (via IPsec ESP)\nAuthorization: IMS AKA response
    PCSCF->>UE: 200 OK (via IPsec ESP)

    Note over UE,PCSCF: All subsequent SIP on this port pair is ESP-protected
```

**Four IPsec SAs per UE-P-CSCF pair:**
- UE client port → P-CSCF server port: UE-originated messages (protected)
- P-CSCF client port → UE server port: P-CSCF-originated messages (protected)
- Plus two unprotected SAs used only for the initial un-authenticated REGISTER

**Key derivation:** Keys are derived from CK + IK received from S-CSCF (which got them from HSS via MAA). The UE independently derives the same keys from its USIM. Neither UE nor P-CSCF needs to exchange keys explicitly.

---

## AF Role — Rx Interface Behavior

The P-CSCF is the primary **Application Function (AF)** in IMS. Its Rx interactions are tightly coupled to SIP message processing:

### Rx Session Lifecycle

```mermaid
sequenceDiagram
    participant UE
    participant PCSCF as P-CSCF (AF)
    participant PCRF
    participant PGW

    Note over UE,PGW: SIP INVITE received — SDP offer
    PCSCF->>PCRF: Rx AAR\n(Framed-IP-Address=UE-IP,\nMedia-Component: audio QCI=1 GBR=64k,\nFlow-Description: 5-tuple,\nFlow-Status=DISABLED)
    PCRF-->>PCSCF: Rx AAA (authorized)
    Note over UE,PCSCF: 183 / PRACK / UPDATE — SDP answer confirmed
    PCSCF->>PCRF: Rx AAR update\n(Flow-Status=ENABLED, final 5-tuple)
    PCRF->>PGW: Gx RAR (PCC rule: QCI=1 GBR=64k, gate=OPEN)
    PGW-->>PCRF: Gx RAA
    PCRF-->>PCSCF: Rx AAA (gate open, resources committed)
    Note over UE,PGW: RTP flows on GBR bearer
    Note over UE,PCSCF: BYE received
    PCSCF->>PCRF: Rx STR (session terminated)
    PCRF->>PGW: Gx RAR (remove PCC rule)
    PGW->>PGW: Delete GBR bearer
    PCRF-->>PCSCF: Rx STA
```

### Rx AAR Content (Derived from SDP)

| SDP Field | Rx AVP | Meaning |
|---|---|---|
| `m=audio 49152 RTP/AVP 98` | Media-Type=AUDIO | Voice stream |
| `b=AS:64` | Max-Requested-Bandwidth-UL/DL=64kbps | GBR value |
| `c=IN IP4 192.0.2.1` + port | Flow-Description (5-tuple) | SDF filter for TFT |
| RTCP port (RTP port + 1) | Media-Sub-Component (RTCP flow) | Separate sub-TFT entry |
| Codec (AMR-WB, G.711) | Used for QCI selection logic | → QCI=1 for voice |

**RTCP flows:** P-CSCF always adds a sub-component for the RTCP port (RTP port + 1, bidirectional) in addition to the RTP component. This ensures RTCP control traffic gets the same QCI treatment.

### P-CSCF Response to PCRF Notifications

The PCRF may send Rx RAR to the P-CSCF to notify of bearer events:

| PCRF → P-CSCF Notification | P-CSCF Action |
|---|---|
| Bearer lost (RAT change, congestion) | Trigger SIP re-INVITE or UPDATE to renegotiate media |
| Access type change (e.g. LTE → WLAN) | Notify UE via SIP of changed QoS characteristics |
| Bearer released (PCRF-initiated) | Trigger SIP session teardown (BYE) |

---

## P-Header Insertion

The P-CSCF adds 3GPP-specific SIP headers to all forwarded messages:

| Header | Value | Purpose |
|---|---|---|
| `P-Visited-Network-ID` | VPLMN identity string | Tells I-CSCF/S-CSCF which visited network the UE is in |
| `P-Access-Network-Info` | Access type + location (3GPP-E-UTRAN-FDD; UTRAN-cell-id; GERAN-cell-id) | Access type and UE location for charging and policy |
| `P-Associated-URI` | List of registered IMPUs | Inserted in 200 OK to REGISTER; tells UE all its registered public identities |
| `P-Called-Party-ID` | Terminating IMPU | Added on terminating side to identify which IMPU was dialed |
| `P-Charging-Vector` | ICID + IOI | IMS charging correlation ID; P-CSCF inserts initial ICID on originating side |
| `P-Early-Media` | Authorization flag | Controls early media (ringback tone) before 200 OK |

---

## Procedure Participation

### 1. IMS Initial Registration

```mermaid
sequenceDiagram
    participant UE
    participant PCSCF as P-CSCF
    participant ICSCF as I-CSCF
    participant SCSCF as S-CSCF

    UE->>PCSCF: REGISTER sip:operator.com\nFrom: sip:+1234@operator.com\nSecurity-Client: ipsec-3gpp params
    PCSCF->>PCSCF: Add P-Visited-Network-ID\nAdd P-Access-Network-Info
    PCSCF->>ICSCF: REGISTER (via Mw)
    Note over ICSCF,SCSCF: UAR/UAA with HSS → S-CSCF selected
    ICSCF->>SCSCF: REGISTER
    SCSCF->>PCSCF: 401 Unauthorized (IMS AKA challenge)
    PCSCF->>UE: 401 Unauthorized\nSecurity-Server: P-CSCF ipsec params
    Note over UE,PCSCF: IPsec SA negotiated (4 SAs)
    UE->>PCSCF: REGISTER (via IPsec, with AKA response)
    PCSCF->>ICSCF: REGISTER
    ICSCF->>SCSCF: REGISTER
    Note over SCSCF: SAR/SAA with HSS → service profile downloaded
    SCSCF->>ICSCF: 200 OK
    ICSCF->>PCSCF: 200 OK (P-Associated-URI list)
    PCSCF->>PCSCF: Store S-CSCF address\nStore IPsec SA\nStart registration timer
    PCSCF->>UE: 200 OK (P-Associated-URI, registration expiry)
```

### 2. VoLTE MO Call (Originating)

P-CSCF receives INVITE from UE, triggers Rx AAR (gate disabled), forwards to S-CSCF. On SDP answer (183/200 OK), updates Rx session, enables gate. On BYE, sends Rx STR.

**Key P-CSCF actions:**
1. Parse SDP offer → extract media type, bandwidth, IP/port
2. Rx AAR with gate=DISABLED (preconditions phase)
3. Forward INVITE to S-CSCF (add `P-Charging-Vector` with fresh ICID)
4. On 183 with SDP answer: update Rx session with final 5-tuple
5. On preconditions complete: Rx AAR update gate=ENABLED
6. On 200 OK ACK: session established
7. On BYE: Rx STR → bearer released

### 3. VoLTE MT Call (Terminating)

P-CSCF is in the SIP path for the terminating leg:
1. Receives INVITE from S-CSCF (Mw)
2. Parses SDP offer from remote party
3. Sends Rx AAR to PCRF (for terminating UE's PDN connection bearer)
4. Forwards INVITE to UE (via Gm, IPsec protected)
5. Relays 183, PRACK, UPDATE, 200 OK back toward originating side
6. On BYE: Rx STR

### 4. Session Release

```mermaid
sequenceDiagram
    participant UE
    participant PCSCF as P-CSCF
    participant SCSCF as S-CSCF
    participant PCRF

    UE->>PCSCF: BYE sip:session@...
    PCSCF->>SCSCF: BYE (forward via Mw)
    PCSCF->>PCRF: Rx STR (AF-Application-Identifier, session ID)
    PCRF-->>PCSCF: Rx STA
    Note over PCRF: Remove PCC rules → Delete GBR bearer via Gx RAR
    SCSCF-->>PCSCF: 200 OK (BYE)
    PCSCF-->>UE: 200 OK (BYE)
```

**P-CSCF-initiated session release (network side):** If the P-CSCF detects that the SIP signaling bearer is lost (e.g. IPsec SA expired, S1 release with no reachability), it sends Rx STR to PCRF to release the GBR bearer, and may send a BYE toward the network to terminate the IMS session.

### 5. Re-registration

UE sends periodic REGISTER before expiry. P-CSCF forwards; on 200 OK, resets registration timer. No new IPsec SA needed — existing SA reused.

### 6. Network-Initiated De-registration (RTR from HSS)

```mermaid
sequenceDiagram
    participant HSS
    participant SCSCF as S-CSCF
    participant PCSCF as P-CSCF
    participant UE

    HSS->>SCSCF: RTR (Registration-Termination-Request)
    SCSCF-->>HSS: RTA
    SCSCF->>PCSCF: de-REGISTER (SIP REGISTER with Expires:0)
    PCSCF->>UE: de-REGISTER (forward)
    UE-->>PCSCF: 200 OK
    PCSCF->>PCSCF: Clear IPsec SA\nClear S-CSCF route\nSend Rx STR for any active sessions
```

---

## Signaling Compression (SigComp)

P-CSCF supports optional SIP compression via **SigComp** (RFC 3486) to reduce over-the-air bandwidth consumption for SIP messages:

- UE signals SigComp support in REGISTER (`comp=sigcomp` in Via header)
- P-CSCF and UE negotiate a shared compression state machine
- Compressed SIP messages are 5-10x smaller than uncompressed
- Only applies to Gm (UE ↔ P-CSCF) — Mw uses full SIP
- Important for NB-IoT and legacy narrow-band access where SIP overhead is significant

---

## Emergency Call Handling

P-CSCF plays a special role in IMS emergency calls:

```mermaid
flowchart TD
    UE_INVITE["UE sends INVITE\nRequest-URI: urn:service:sos\nor emergency number"] --> detect["P-CSCF detects emergency\n(Request-URI or number match)"]
    detect --> located{"UE location\navailable?"}
    located -->|Yes| loc_hdr["Add P-Access-Network-Info\nwith UE location (cell ID)"]
    located -->|No| no_loc["Forward without location\nE-CSCF may request"]
    loc_hdr --> ecscf["Route to E-CSCF\n(Emergency-CSCF)"]
    no_loc --> ecscf
    ecscf --> psap["E-CSCF routes to PSAP\n(Public Safety Answering Point)"]
    detect --> rx_emerg["Rx AAR for emergency bearer\n(EPS emergency bearer, QCI=5 or higher priority)"]
```

- P-CSCF detects emergency URIs or numbers
- Inserts location (P-Access-Network-Info with ECGI/cell-ID)
- Routes toward E-CSCF (not I-CSCF)
- Triggers emergency bearer via Rx (separate from normal Gx session; uses emergency PDN if applicable)
- Works even for unauthenticated UEs (limited-service state)

---

## P-CSCF State per UE Registration

```mermaid
erDiagram
    UE_REGISTRATION {
        string IMPU
        string IMPI
        string contact_address
        string S_CSCF_address
        int registration_expiry
        timestamp registered_at
    }
    IPSEC_SA {
        int SPI_client
        int SPI_server
        int port_client
        int port_server
        string algorithm_integrity
        string algorithm_encryption
        string CK_hex
        string IK_hex
        timestamp expiry
    }
    RX_SESSION {
        string AF_session_ID
        string call_ID
        string UE_IP
        int media_component_number
        string media_type
        int GBR_UL_kbps
        int GBR_DL_kbps
        string flow_description_UL
        string flow_description_DL
        string gate_status
        string ICID
    }
    UE_REGISTRATION ||--|| IPSEC_SA : "has (4 SAs)"
    UE_REGISTRATION ||--o{ RX_SESSION : "has (one per active session)"
```

---

## Failure and Overload Behavior

```mermaid
stateDiagram-v2
    [*] --> Normal
    Normal --> RxDown : PCRF unreachable
    RxDown --> Normal : PCRF reconnected
    RxDown --> RxDown : New sessions proceed without GBR bearer\nP-CSCF may reject INVITE or forward without Rx AAR (operator policy)\nExisting sessions unaffected (Rx session already established)
    Normal --> IPsecFailed : UE IPsec SA expired or lost
    IPsecFailed --> Normal : UE re-registers (new IPsec SA)
    IPsecFailed --> IPsecFailed : P-CSCF sends BYE for active sessions\nSend Rx STR to PCRF\nClear UE registration state
    Normal --> SIPTimerExpiry : Registration timer expires (no re-REGISTER from UE)
    SIPTimerExpiry --> Normal : UE re-registers
    SIPTimerExpiry --> SIPTimerExpiry : P-CSCF initiates de-registration toward S-CSCF\nSend Rx STR for active sessions\nSend RTR-equivalent signaling
    Normal --> Overloaded : High SIP message rate
    Overloaded --> Normal : Load decreases
    Overloaded --> Overloaded : Apply SIP 503 Service Unavailable\nRetry-After header to rate-limit UE retries
```

---

## Configuration Parameters

| Parameter | Description |
|---|---|
| Gm IP address / port | SIP listening address for UE connections (port 5060 / 5061 TLS) |
| Mw peer addresses | I-CSCF and S-CSCF addresses for SIP forwarding |
| PCRF address (Rx) | Diameter realm/hostname for Rx sessions |
| IPsec algorithms | Supported integrity (HMAC-SHA-1) and encryption (AES-CBC, 3DES) algorithms |
| SPI allocation range | SPI values for IPsec SA negotiation |
| P-Visited-Network-ID string | Operator's network identity string (VPLMN identifier) |
| Registration timer | Default registration expiry (suggested to UE, typically 3600s) |
| SigComp support | Whether SIP compression is enabled |
| Emergency URI list | Numbers/URNs that trigger emergency call handling |
| E-CSCF address | Where to route emergency INVITEs |
| Rx timeout | How long to wait for PCRF AAA before proceeding (or rejecting) |
| Gate control mode | Whether to enforce 2-phase gate (precondition-based) or immediate |
| P-CSCF overload threshold | SIP message rate triggering 503 responses |

---

## Key Architectural Properties

| Property | Details |
|---|---|
| **Always in VPLMN** | P-CSCF is in the visited network; the IMS core (I/S-CSCF) may be in the HPLMN. This keeps the SIP/IPsec termination local |
| **IPsec termination** | P-CSCF is the only node that terminates the UE's IPsec SA — all other IMS nodes see plaintext SIP |
| **Single hop from UE** | UE uses P-CSCF as SIP outbound proxy for all requests; Route header set by P-CSCF points to S-CSCF |
| **Stateful for charging** | P-CSCF inserts P-Charging-Vector with ICID (IMS Charging ID) — correlates SIP dialog with CDRs across all IMS nodes |
| **Dual role** | Simultaneously a SIP proxy (for the IMS core) and a Diameter client (for the EPC PCRF). Neither role can function without the other in VoLTE |
| **No subscriber data** | P-CSCF has no access to HSS; it does not know the subscriber's iFC, IMPI, or service profile. It relies entirely on what S-CSCF includes in SIP headers |

---

## Cross-References

| Topic | Page |
|---|---|
| P-CSCF base entity | [entities/P-CSCF.md](P-CSCF.md) |
| I-CSCF (Mw peer, IMS entry) | [entities/I-CSCF.md](I-CSCF.md) |
| S-CSCF (Mw peer, registrar) | [entities/S-CSCF.md](S-CSCF.md) |
| PCRF (Rx policy peer) | [entities/PCRF.md](PCRF.md) |
| PCRF deep-dive | [entities/PCRF-deepdive.md](PCRF-deepdive.md) |
| IMS Identity Model | [concepts/IMS-identity-model.md](../concepts/IMS-identity-model.md) |
| IMS Registration | [procedures/IMS-registration.md](../procedures/IMS-registration.md) |
| IMS QoS bearer | [procedures/IMS-QoS-bearer.md](../procedures/IMS-QoS-bearer.md) |
| VoLTE MO call | [procedures/VoLTE-MO-call.md](../procedures/VoLTE-MO-call.md) |
| VoLTE MT call | [procedures/VoLTE-MT-call.md](../procedures/VoLTE-MT-call.md) |
| Session release | [procedures/session-release.md](../procedures/session-release.md) |
| IMS reference points | [interfaces/IMS-reference-points.md](../interfaces/IMS-reference-points.md) |
