---
title: "SIP/SDP Profile for IMS (TS 24.229 §4)"
type: protocol
tags: [SIP, SDP, IMS, trust-domain, ICID, IOI, P-headers, security, overload, restoration, II-NNI]
sources: [ts_124229v160600p.pdf]
updated: 2026-04-18
---

# SIP/SDP Profile for IMS

3GPP TS 24.229 is the stage-3 specification that defines **how SIP and SDP are used within the IM CN subsystem**. §4 General establishes the conformance profile — roles, addressing, security, routing, trust domain, charging correlation, and cross-cutting mechanisms — that all subsequent per-node procedures in §5 build upon.

---

## 1. Node SIP Roles (§4.1)

Each IMS functional entity maps to a SIP role. The role determines which IETF RFC requirements apply.

| Functional Entity | SIP Role | Notes |
|---|---|---|
| UE | UA | Full UA role; additional UE-specific requirements in §5.1 |
| P-CSCF | Proxy (+ B2BUA for security) | Proxy role per RFC 3261; B2BUA for Gm security; IMS-ALG functions in §6.7 |
| I-CSCF | Proxy | Loose routing per RFC 3261 |
| S-CSCF | Proxy + UA (specific cases) | UA role: S-CSCF-initiated dialog release, providing messaging (MESSAGE), Transit Function |
| MGCF | UA | Per §5.5 |
| BGCF | Proxy | Per §5.6 |
| AS (terminating UA mode) | UA | Per TS 23.218 §9.1.1.1 + §5.7.2 |
| AS (originating UA mode) | UA | Per TS 23.218 §9.1.1.2 + §5.7.3 |
| AS (SIP proxy mode) | Proxy | Per TS 23.218 §9.1.1.3 + §5.7.4 |
| AS (3PCC mode) | UA | Per TS 23.218 §9.1.1.4 + §5.7.5 |
| MRFC | UA | Per §5.8 |
| MRB (inline-aware) | UA | Per §5.8A |
| MRB (inline-unaware) | Proxy | Per §5.8A |
| IBCF | Proxy (+ IMS-ALG) | IMS-ALG mode = B2BUA for address rewriting; §5.10 |
| E-CSCF | Proxy (+ UA for emergencies) | UA role when providing final response per RFC 6442 or RFC 3323 |
| LRF | UA | Per §5.12 |
| ISC gateway function | Proxy (+ IMS-ALG) | Per §5.13 |

> **Key rule**: Proxy-role entities that process SIP must use loose routing (RFC 3261 §16) and must not modify message bodies unless stated in this specification or a referenced RFC (NOTE 5 §4.1).

---

## 2. URI and Address Assignments (§4.2)

1. **I-CSCF URIs**: I-CSCFs are allocated SIP URIs (e.g. `sip:pcscf.home1.net`). A single SIP URI may be shared across multiple physical I-CSCFs via DNS SRV load-balancing.
2. **IP Addresses**: IM CN subsystem entities support IPv4-only, IPv6-only, or dual-stack. Determined by IP-CAN type:
   - GPRS/EPS IP-CAN → TS 23.221 §5.1
   - cdma2000 IP-CAN → §M.2.2.1 of TS 24.229
   - 5GS IP-CAN → TS 23.501
3. **Private User Identity (IMPI)**: Allocated by home network operator. Stored in ISIM; if no ISIM but USIM present, derived per §5.1.1.1A. If neither present (IMC), provisioned in IMC.
4. **Public User Identity (IMPU)**: One or more SIP URI and/or tel URIs. At least one must be a SIP URI. Available in ISIM or derived from USIM.
5. **tel URI companion rule**: For each tel URI, there is always a companion SIP URI with the same user part and `user=phone` parameter hosted by the home IMS domain.
6. **Shared PUI**: A public user identity may be simultaneously registered from multiple UEs with different private user identities and different contact addresses.
7. **Instance ID**: Required if UE supports GRUU (RFC 5627) or multiple registrations; conforms to RFC 5627 [93] and RFC 5626 [92].
8. **ICSI assignment**: UEs are assigned ICSI URNs (coded as `urn:urn-7:3gpp-service.*`) for IMS communication services they support; signalled in REGISTER via §7.2A.8.
9. **E-CSCFs**: Allocated multiple SIP URIs; the SIP URI configured in the P-CSCF/AS/IBCF for reaching E-CSCF is distinct from the URI assigned to the EATF.

---

## 3. Transport Mechanisms (§4.2A)

- No mandatory transport protocol above RFC 3261 §18 except access-technology-specific annexes.
- **Exception**: UE and IM CN subsystem entities shall transport SIP messages longer than **1300 bytes** via TCP procedures of RFC 3261 §18.1.1 (even if discovered MTU > 1500 bytes).
- **SIP ports**: Initial REGISTER: per §5.1.1.2 / §5.2.2. All other requests/responses: on the protected ports per TS 33.203.
- **Emergency exception**: If UE lacks credentials, UE and P-CSCF may use non-protected ports for non-REGISTER messages.

---

## 4. Security Mechanisms (§4.2B)

### 4.1 Signalling Security

TS 33.203 defines the security architecture. TS 24.229 specifies the SIP-layer mechanisms, summarised in Table 4-1:

| Mechanism | Authentication | Integrity | IPsec SA | UE Support | Network Support |
|---|---|---|---|---|---|
| IMS AKA + IPsec ESP | IMS AKA | IPsec ESP | Yes (RFC 3329) | Mandatory (UICC) | Mandatory P-CSCF, I-CSCF, S-CSCF |
| IMS AKA + HTTP Digest AKAv2 (no IPsec) | IMS AKA | TLS session | No | Mandatory (WIC/UICC) | Mandatory eP-CSCF |
| SIP digest without TLS | SIP digest | None | No | Optional | Optional P/I/S-CSCF |
| SIP digest + TLS | SIP digest | TLS session | Yes | Optional | Optional P/I/S-CSCF |
| NASS-IMS bundled auth | N/A | None | No | No UE support | Optional P/I/S-CSCF |
| GPRS-IMS-Bundled auth | N/A | None | No | No UE support | Optional I/S-CSCF |
| Trusted node authentication | N/A | None | No | No UE support | Optional I/S-CSCF |
| SIP over TLS + client cert | TLS client cert | TLS session | No | Mandatory (external attached network) | Optional IBCF, P-CSCF |

> **NOTE 1**: WebRTC TLS is optional and used only with SIP digest authentication. For WebRTC, TLS may be combined with IMS AKA using HTTP Digest AKAv2 without IPsec SA.

> **NOTE 2**: Network domain security between IM CN subsystem entities uses TS 33.210 (IP layer NDS). TS 33.328 covers media plane security.

### 4.2 Media Security (§4.2B.2)

Table 4-2 summarises media security mechanisms. Key options:

| Mechanism | Media | UE Support | P-CSCF (IMS-ALG) |
|---|---|---|---|
| End-to-access-edge using SDES | RTP | RFC 3329 + Table A.317/34,36,37 SDP | Required |
| End-to-access-edge MSRP using TLS + cert | MSRP | RFC 3329 + Table A.317/40,40A,51,37A | Required |
| End-to-access-edge BFCP using TLS + cert | BFCP | RFC 4583 + cert | Required |
| End-to-access-edge UDPTL using DTLS + cert | UDPTL | RFC 7345 + cert | Required |
| End-to-end SDES | RTP | Table A.317/34,36 SDP extensions | Not applicable |
| End-to-end KMS | RTP | Table A.317/34,35 SDP extensions | Not applicable |

> MGCF has no media security support (CS domain interworking). PSAPs not expected to support end-to-end media security.

---

## 5. Routing Principles (§4.3)

All IM CN subsystem entities apply **loose routing** per RFC 3261 §16. I-CSCF, IBCF, S-CSCF, E-CSCF shall use RFC 3261 loose routing even when interacting with strict-router-only networks.

---

## 6. Trust Domain (§4.4)

### 6.1 Definition

The trust domain for the IM CN subsystem comprises all functional entities **belonging to the same operator network**: P-CSCF, eP-CSCF, E-CSCF, I-CSCF, IBCF, S-CSCF, BGCF, MGCF, MRFC, MRB, EATF, ATCF, ISC gateway, and all ASes within the trust domain.

Entities in **networks with interconnect agreements** may also be in the trust domain. The PSAP is typically regarded as within the trust domain.

### 6.2 Trust Domain Applies to These Headers/Parameters

A trust domain boundary entity shall apply removal rules when SIP signalling crosses the trust boundary:

| Header/Parameter | Trust Domain Action |
|---|---|
| P-Asserted-Identity | Remove at boundary per RFC 3325; "id" priv-value not removed |
| P-Access-Network-Info | Remove at boundary per RFC 7315 |
| History-Info | Remove at boundary per RFC 7044 §10.1.2 |
| P-Asserted-Service | Remove at boundary per RFC 6050 |
| Resource-Priority | Remove when forwarding outside trust domain; strip from untrusted source |
| Reason (in response) | Remove at boundary per RFC 6432 |
| P-Profile-Key | Remove at boundary per RFC 5002 |
| P-Served-User | Remove at boundary per RFC 5502 |
| P-Private-Network-Indication | Remove when forwarding outside; strip from untrusted source |
| P-Early-Media | Remove at boundary per RFC 5009 |
| CPC / OLI URI parameters | Remove when crossing trust boundary |
| Feature-Caps | Remove all Feature-Caps from UEs and external networks outside trust domain |
| Priority (psap-callback value) | Remove based on local policy |
| iotl URI parameter | Remove when crossing trust boundary |
| Restoration-Info | Remove at boundary |
| Relayed-Charge | Remove at boundary |
| Service-Interact-Info | Remove at boundary |
| Cellular-Network-Info | Remove at boundary; strip from untrusted source |
| Response-Source | Remove at boundary per §7.2.17 |
| Attestation-Info | Remove at boundary per §7.2.18 |
| Origination-Id | Remove at boundary per §7.2.19 |
| Additional-Identity | Remove at boundary per §7.2.20 |

---

## 7. Charging Correlation Principles (§4.5)

The IM CN subsystem generates/retrieves five types of charging correlation information, all carried in the **P-Charging-Vector header field** (§7.2A.5 / RFC 7315):

| Correlation Data | P-Charging-Vector Parameter | Description |
|---|---|---|
| ICID | `icid-value` | IM CN subsystem Charging Identifier — session-level correlation across all IMS entities |
| ICID origin | `icid-generated-at` | Address of the entity that generated the ICID |
| Related ICID | `related-icid` | ICID used on SRVCC/dual-radio source access leg |
| Access network charging info | `access-network-charging-info` | Bearer-level data from IP-CAN (GGSN/PGW context info) |
| Orig IOI | `orig-ioi` | Originating operator network identifier |
| Term IOI | `term-ioi` | Terminating operator network identifier |
| Transit IOI | `transit-ioi` | Indexed list of transit operator network identifiers |
| FE Identifier | `fe-identifier` | IM CN subsystem functional entity address + AS application identifier |

### 7.1 ICID Generation Rules (Table 4-2A)

| ICID inserted in... | Initial/standalone SIP message | Subsequent SIP message |
|---|---|---|
| Any **request** | First IM CN subsystem entity inserts `icid-value` | First IM CN entity inserts `icid-value` from initial request's icid-value |
| Any **response** | First IM CN entity receiving request inserts `icid-value` from initial request or standalone | Same — uses icid-value from initial request |

**Per-entity ICID generation responsibilities:**
- **P-CSCF**: generates ICID for UE-originated calls; also generates a unique ICID per REGISTER (in a unique P-Charging-Vector instance, not passed to UE)
- **I-CSCF**: generates ICID for UE-terminated calls if none received in initial request
- **MGCF**: generates ICID for PSTN/PLMN-originated calls
- **AS**: generates ICID when acting as originating UA
- **MSC server**: generates ICID for ICS/SRVCC-originated calls

### 7.2 IOI Types (§4.5.4)

Three IOI types defined in Table 4-2B:

| Type | Between | Carried in |
|---|---|---|
| **Type 1** | Visited and home network (P-CSCF ↔ S-CSCF; SCC AS ↔ ATCF) | REGISTER requests/responses + all session-related requests/responses |
| **Type 2** | Originating and terminating network (S-CSCF ↔ S-CSCF or MGCF; E-CSCF ↔ MGCF/IBCF for emergency) | All session-related and session-unrelated requests/responses |
| **Type 3** | Any network ↔ any AS; E-CSCF ↔ LRF or EATF; transit function ↔ AS | All session-related and session-unrelated requests/responses |

**IOI insertion rule (Table 4-2B):**
- In any **request**: sending network removes existing `orig-ioi`, inserts its own `orig-ioi`; does not insert `term-ioi`
- In any **response**: receiving network populates `term-ioi` from previously received `orig-ioi`; sets `orig-ioi` to `term-ioi` value from previous response (if received)

### 7.3 Transit IOI (§4.5.4A)

The Transit IOI identifies networks that **transit** the SIP request between originating and terminating networks. Carried in `transit-ioi` parameter (indexed). Inserted by transit networks (IBCF acting as entry/exit point per Table 4-3; additional routing functions in Annex I). Deleted by S-CSCF of home terminating network.

```
Example: P-Charging-Vector: icid-value="AyretyU0dm+6O2IrT5tAFrbHLso=023551024"; 
         orig-ioi=home1.net; transit-ioi="operatorA.1, void, operatorB.3"
```

### 7.4 Charging Function Addresses (§4.5.5)

CDF/OCF addresses retrieved from HSS via Cx interface, passed by S-CSCF to IMS entities in P-Charging-Function-Addresses header (RFC 7315). Parameters: `ccf` for CDF (offline); `ecf` for OCF (online). Multiple values for redundancy; first instance = primary.

- AS may retrieve charging function addresses from HSS via Sh interface
- P-Charging-Function-Addresses not passed to UE or visited network

### 7.5 FE Identifier (§4.5.8)

Each IM CN functional entity and AS generating charging events includes its address (`fe-addr`) and optionally an application identifier (`ap-id`) in the `fe-identifier` parameter. The last element of the operator domain:
1. Deletes `fe-identifier` parameter from received P-Charging-Vector
2. Adds its own `fe-addr` to `fe-identifier`

This enables the billing domain to compare CDR addresses against actual entity addresses.

---

## 8. Emergency Service — SIP Aspects (§4.7)

Three sources of emergency calls in IM CN subsystem:
1. **UE-generated**: UE detects emergency → procedures in §5.1.6
2. **AS-generated**: AS identifies an incoming request as emergency → routes to E-CSCF
3. **Enterprise network**: IBCF receives from peering enterprise network → routes to E-CSCF

Location provision mechanisms for emergency calls (§4.7.5):
- UE includes Geolocation header (RFC 6442)
- UE includes P-Access-Network-Info (cell ID)
- P-CSCF includes P-Access-Network-Info from IP-CAN info (Rx/Gx)
- LRF allocates location reference → provides to PSAP over Le interface
- S-CSCF includes P-Access-Network-Info from HSS
- E-CSCF → LRF request per §5.12.3.2 / §5.11.4.3

---

## 9. Tracing of Signalling (§4.8)

SIP signalling tracing configured via TS 32.422 (IMS level trace management object per TS 24.323). Trace starts with the initial request creating a dialog; stops when a pre-configured trigger occurs or the dialog ends (RFC 8497). Trace depth "maximum" logs all requests/responses; "minimum" logs initial request + 100 + ACK only.

---

## 10. Overlap Signalling (§4.9)

Two overlap signalling methods (both optional and network-policy-dependent):

| Method | Mechanism | When Used |
|---|---|---|
| **In-dialog** | INFO requests carry additional digits; new INVITEs on 404/484 | Before early dialog established |
| **Multiple-INVITE** | INVITEs with same Call-ID and From tag carry incremental digits | Per TS 29.163 |

Only one method used within one IM CN subsystem. **Deterministic routing rule (§4.9.3.2)**: Multiple-INVITE method — entity receiving a new INVITE outside existing dialog with same Call-ID and From shall route it to the same next-hop as the previous INVITE.

---

## 11. Dialog Correlation (§4.10)

- Standard mechanism: Call-ID + From/To tags (RFC 3261)
- **Problem**: B2BUAs modify Call-ID → breaks correlation across the path
- **Solution**: Session-ID header (RFC 7989) carries a globally unique session identifier unchanged across B2BUAs; usage specified in Annex A
- CONF AS: 3PTY conference INVITEs include Session-ID in the URI list to CONF AS per TS 24.147

---

## 12. Priority Mechanisms (§4.11)

Uses RFC 4412 Resource-Priority header. Three authorization approaches:

1. **Subscription-based**: P-CSCF receives priority info with UE contact changes (from HSS/profile); validates Resource-Priority against stored allowed values; forwards to AS for final authorization
2. **Database-based** (dial-string): UE indicates priority via special dialing; P-CSCF marks request for temporary priority; AS provides final authorization
3. **Database-based** (REGISTER-based): UE uses special dialing; request routed to S-CSCF and AS for authorization; backwards indication confirms specific priority

**Priority namespace "ets"** (Emergency Telecommunications Service): Regulatory requirement across several administrations. No UE-level requirement on conformance to any specific priority scheme.

---

## 13. Overload Control (§4.12)

Two mechanisms (not mutually exclusive):

| Mechanism | Standard | Mode | Scope |
|---|---|---|---|
| **Feedback-based** | RFC 7339 | Per-hop Via header; loss-based algorithm default | Adjacent hop only |
| **Load filter event package** | RFC 7200 | Load filter subscription to AS; distribute load filter rules | Network-wide (hop-by-hop or wider) |

**Priority ordering (last to drop)**:
1. Emergency calls — exempt to a configured threshold even under overload
2. Priority calls (multimedia priority service) — exempt until further exemption would cause network instability
3. Mid-dialog messages — higher priority than initial SIP requests; last to drop before initial requests

Operation across operators requires bilateral agreement.

---

## 14. II-NNI Traversal Scenario (§4.13)

The signalling path between a calling and called user may span one or more II-NNI legs (traffic legs). Each leg is an **II-NNI traversal scenario** with distinct characteristics.

### 14.1 Identification — `iotl` Parameter

The `iotl` SIP URI parameter (RFC 7549) in Request-URI or Route headers identifies the II-NNI traversal scenario type:
- Included by the **start** of the II-NNI traversal scenario (P-CSCF, S-CSCF, BGCF, or SCC AS)
- Removed by the **end** of the II-NNI traversal scenario (S-CSCF, TRF, P-CSCF)
- Absence of `iotl` → non-roaming default scenario assumed

```
Example Route header with iotl:
Route: <sip:ibcf-vA1.visited-A.net;lr>,<sip:home-abc@scscf-hA1.home-A.net;lr;iotl=visitedA-homeA>
```

### 14.2 Security at II-NNI Boundary (§4.13.3)

IBCF acting as entry point for dialog-creating SIP requests from outside the trust domain:
- Shall remove any `iotl` URI parameter per §4.4.15
- May assume default II-NNI traversal scenario type OR use trusted SIP elements to determine scenario type

---

## 15. Restoration Procedures (§4.14)

Two restoration scenarios:

### 15.1 P-CSCF Restoration (§4.14.2)

Triggered when a neighbouring entity (S-CSCF, IBCF) detects P-CSCF unreachability. Two variants:

| Trigger | Restoration Type | Mechanism |
|---|---|---|
| UDM/HSS-based | UE-terminating requests | S-CSCF sends `Restoration-Info` header to HSS; HSS/UDM notifies UE's MME/AMF |
| PCF/PCRF-based | UE-terminating INVITE | S-CSCF includes IMSI in `Restoration-Info` header; IBCF selects alternative P-CSCF |
| P-CSCF restart (lost bindings) | Home network | S-CSCF/I-CSCF rejects via 404; triggers UDM/HSS-based restoration |

### 15.2 S-CSCF Restoration (§4.14.3)

- P-CSCF informs UE about S-CSCF failure in **504 (Server Time-out) response** containing 3GPP IM CN subsystem XML body (§7.6)
- S-CSCF self-trigger: if S-CSCF cannot retrieve profile from HSS (e.g. after restart), sends 504 + XML body to UE → UE performs initial registration
- I-CSCF can re-select S-CSCF if previously selected S-CSCF unavailable
- IBCF entry-point: can trigger UE re-registration by including XML body in 504 response

---

## 16. Resource Sharing (§4.15)

Resource sharing allows two or more sessions to share the same uplink/downlink media resources. The P-CSCF determines potential resource sharing; if deferred to AS:
- P-CSCF includes `Resource-Share` header (§7.2.13) in REGISTER and third-party REGISTER to AS
- AS responds with updated resource sharing rules via `Resource-Share` header in responses
- Updated rules applied to all sessions sharing resources

---

## 17. Priority Sharing (§4.16)

Priority sharing allows two or more sessions with different priority to share the same bearer. Determination deferred to AS in home network:
- P-CSCF includes `g.3gpp.priority-share` feature-capability indicator (§7.9A.10) in Feature-Caps header in REGISTER
- AS responds with `Priority-Share` header (§7.2.16) in responses with value `allowed` or `not-allowed`

---

## 18. 3GPP PS Data Off (§4.17)

When 3GPP PS data off is active:
- IP packets for non-exempt services are blocked across EPS/GPRS/5GS IP-CAN
- UE includes `g.3gpp.ps-data-off` media feature tag (§7.9.8) in **all** REGISTER requests over EPS, GPRS, and 5GS IP-CAN
- UE re-registers every time PS data off status changes or list of exempt services changes
- AS handling a service checks the flag; if active and service not exempt, blocks downlink IP packets from reaching UE

---

## 19. Dynamic Service Interaction (§4.18)

Allows ASes executing services in the same IMS session to avoid conflicting interactions. Mechanism:
- AS includes `Service-Interact-Info` header (§7.2.14) with identities of executed services and services to avoid
- AS receiving this header takes it into account before executing its service
- Trust domain boundary: `Service-Interact-Info` removed at boundary per §4.4.18

---

## 20. Restricted Local Operator Services (RLOS) (§4.19)

RLOS are operator-defined services offered to UEs using an EPS IP-CAN. Available when:
- UE is successfully registered using IMS AKA or GPRS-IMS bundled authentication, **or**
- UE attempted registration that was rejected with 403 (Forbidden)

RLOS offered only for UE-originating case. Available to both own subscribers and roaming subscribers. Detailed procedures in Annex L.

---

## Cross-References

- Trust domain enforcement details: each node's §5 subclause
- P-Charging-Vector full syntax: [§7.2A.5](../protocols/SIP-IMS-extensions.md) / RFC 7315
- Security mechanism details (IMS AKA, IPsec SA): [concepts/IMS-access-security.md](../concepts/IMS-access-security.md)
- Emergency session flows: [procedures/IMS-emergency-session.md](../procedures/IMS-emergency-session.md)
- ICID/IOI at charging layer: [concepts/IMS-charging-architecture.md](../concepts/IMS-charging-architecture.md)
- II-NNI interface: [interfaces/II-NNI.md](../interfaces/II-NNI.md)
- IBCF entity: [entities/IBCF.md](../entities/IBCF.md)
- P-CSCF entity: [entities/P-CSCF.md](../entities/P-CSCF.md)
- S-CSCF entity: [entities/S-CSCF.md](../entities/S-CSCF.md)
