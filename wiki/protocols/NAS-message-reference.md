---
title: "NAS Message Reference (Stage 3) — §7 Error Handling + §8 Message Definitions"
type: protocol
tags: [NAS, EMM, ESM, message-definitions, error-handling, stage-3, EPS]
sources: [ts_124301v170600p.pdf]
updated: 2026-04-20
---

# NAS Message Reference (Stage 3)

Stage-3 error handling rules (§7) and message functional definitions (§8) for all EMM and ESM NAS messages. Source: 3GPP TS 24.301 v17.6.0.

See also:
- [protocols/NAS-EMM-protocol.md](NAS-EMM-protocol.md) — §4–§5.4 (NAS architecture, security, EMM procedures)
- [procedures/NAS-attach.md](../procedures/NAS-attach.md), [NAS-detach.md](../procedures/NAS-detach.md), [NAS-TAU.md](../procedures/NAS-TAU.md)
- [procedures/NAS-service-request.md](../procedures/NAS-service-request.md)
- [procedures/NAS-ESM-procedures.md](../procedures/NAS-ESM-procedures.md) — §6 ESM procedure descriptions

---

## §7 — Error Handling

### §7.1 Precedence Order

Clauses §7.2–§7.8 are applied in order. Each clause applies only if not already handled by a prior clause. Compliance is mandatory for the UE; implementation-dependent for the network.

### §7.2 — Message Too Short

If a received message is too short to contain all mandatory IEs, the message is **silently ignored**.

### §7.3 — Unknown PTI / EBI

#### §7.3.1 — Unknown PTI (ESM)

| Scenario | UE or Network Action |
|----------|---------------------|
| Network receives PDN CONNECTIVITY REQUEST / PDN DISCONNECT REQUEST / BEARER RESOURCE ALLOCATION REQUEST / BEARER RESOURCE MODIFICATION REQUEST with unknown PTI | Reject with ESM cause **#81** (invalid PTI) |
| UE receives ACTIVATE DEFAULT/DEDICATED/MODIFY with unknown PTI | If retransmission detected → ACCEPT; else → REJECT cause **#47** |
| UE receives most reject messages with bad PTI | Ignore |

#### §7.3.2 — Unknown EBI (ESM)

| Scenario | Action |
|----------|--------|
| Network receives PDN CONNECTIVITY/DISCONNECT/BEARER RESOURCE messages with unknown EBI | Reject with ESM cause **#43** (invalid EBI) |
| UE receives ACTIVATE DEFAULT/DEDICATED/MODIFY with unassigned EBI | REJECT cause **#43** |
| UE receives DEACTIVATE with unknown EBI | ACCEPT (silently) |
| UE receives most reject messages with bad EBI | Ignore |

### §7.4 — Unknown or Unforeseen Message Type

| Condition | Response |
|-----------|----------|
| Unknown message type | EMM STATUS or ESM STATUS, cause **#97** (message type non-existent or not implemented) |
| Message type exists but incompatible with protocol state | EMM STATUS or ESM STATUS, cause **#98** (message type not compatible with protocol state) |

### §7.5 — Mandatory IE Missing or Incorrect

#### §7.5.3 ESM mandatory IE errors

| Message | If mandatory IE error | Response |
|---------|-----------------------|----------|
| ACTIVATE DEFAULT/DEDICATED/MODIFY BEARER CONTEXT REQUEST | Missing mandatory IE | REJECT cause **#96** (mandatory IE incorrect) |
| DEACTIVATE EPS BEARER CONTEXT REQUEST | Missing mandatory IE | ACCEPT (ignoring error) |
| PDN CONNECTIVITY/DISCONNECT REJECT / BEARER RESOURCE ALLOCATION/MODIFICATION REJECT | Error in received reject | Treat IE as absent; ESM STATUS cause **#96** |

### §7.6 — Unknown or Unforeseen IEIs

- **Unknown IEI in non-imperative part**: ignore the IE and continue processing
- **Out-of-sequence IEs**: ignore and continue
- **Repeated IEs**: use the first occurrence; ignore subsequent copies

### §7.7 — Non-Imperative IE Errors

- Syntactically incorrect optional IEs: treat IE as **absent**
- Conditional IE errors: send EMM STATUS or ESM STATUS, cause **#100** (conditional IE error)

### §7.8 — Semantically Incorrect Message

Follow any foreseen reaction specified for the message. If none exists, send EMM STATUS or ESM STATUS with cause **#95** (semantically incorrect message).

---

## §8.1 — Message Structure Overview

Every NAS message consists of IEs with the following presence categories and formats:

| Presence | Meaning |
|----------|---------|
| M | Mandatory |
| C | Conditional |
| O | Optional |

| Format | Description |
|--------|-------------|
| V | Value only (fixed length, half-octet or whole octets) |
| LV | Length + Value |
| T | Type only (1-octet IEI) |
| TV | Type + Value |
| TLV | Type + Length + Value |
| LV-E | Length (2 octets) + Value |
| TLV-E | Type + Length (2 octets) + Value |

**Significance levels**: local (both sides may ignore), dual (both sides must process), access (only in specific access mode).

**NOTE**: The SECURITY PROTECTED NAS MESSAGE wrapper contains: Protocol discriminator + Security header type + Message authentication code (4 oct) + Sequence number (1 oct) + NAS message.

---

## §8.2 — EPS Mobility Management (EMM) Messages

### Message Quick-Reference Table

| §  | Message | Direction | Significance |
|----|---------|-----------|-------------|
| 8.2.1 | ATTACH ACCEPT | net → UE | dual |
| 8.2.2 | ATTACH COMPLETE | UE → net | dual |
| 8.2.3 | ATTACH REJECT | net → UE | dual |
| 8.2.4 | ATTACH REQUEST | UE → net | dual |
| 8.2.5 | AUTHENTICATION FAILURE | UE → net | dual |
| 8.2.6 | AUTHENTICATION REJECT | net → UE | dual |
| 8.2.7 | AUTHENTICATION REQUEST | net → UE | dual |
| 8.2.8 | AUTHENTICATION RESPONSE | UE → net | dual |
| 8.2.9 | CS SERVICE NOTIFICATION | net → UE | dual |
| 8.2.10 | DETACH ACCEPT | both | dual |
| 8.2.11 | DETACH REQUEST | both | dual |
| 8.2.12 | DOWNLINK NAS TRANSPORT | net → UE | dual |
| 8.2.13 | EMM INFORMATION | net → UE | **local** |
| 8.2.14 | EMM STATUS | both | **local** |
| 8.2.15 | EXTENDED SERVICE REQUEST | UE → net | dual |
| 8.2.16 | GUTI REALLOCATION COMMAND | net → UE | dual |
| 8.2.17 | GUTI REALLOCATION COMPLETE | UE → net | dual |
| 8.2.18 | IDENTITY REQUEST | net → UE | dual |
| 8.2.19 | IDENTITY RESPONSE | UE → net | dual |
| 8.2.20 | SECURITY MODE COMMAND | net → UE | dual |
| 8.2.21 | SECURITY MODE COMPLETE | UE → net | dual |
| 8.2.22 | SECURITY MODE REJECT | UE → net | dual |
| 8.2.23 | SECURITY PROTECTED NAS MESSAGE | both | dual |
| 8.2.24 | SERVICE REJECT | net → UE | dual |
| 8.2.25 | SERVICE REQUEST | UE → net | dual |
| 8.2.26 | TRACKING AREA UPDATE ACCEPT | net → UE | dual |
| 8.2.27 | TRACKING AREA UPDATE COMPLETE | UE → net | dual |
| 8.2.28 | TRACKING AREA UPDATE REJECT | net → UE | dual |
| 8.2.29 | TRACKING AREA UPDATE REQUEST | UE → net | dual |
| 8.2.30 | UPLINK NAS TRANSPORT | UE → net | dual |
| 8.2.31 | DOWNLINK GENERIC NAS TRANSPORT | net → UE | dual |
| 8.2.32 | UPLINK GENERIC NAS TRANSPORT | UE → net | dual |
| 8.2.33 | CONTROL PLANE SERVICE REQUEST | UE → net | dual |
| 8.2.34 | SERVICE ACCEPT | net → UE | dual |

---

### §8.2.1–8.2.4 — Attach Messages

**ATTACH ACCEPT** (net → UE): Mandatory IEs: EPS attach result, T3412, TAI list, ESM message container. Key optional IEs: GUTI, T3402 (backoff), T3423 (ISR), Equivalent PLMNs, Emergency number list, EPS network feature support, T3324 (PSM active timer), Extended DRX parameters, DCN-ID, T3448 (CP congestion), T3447 (Service Gap Control), Extended emergency number list, Ciphering key data, UE radio capability ID, WUS assistance, NB-S1 DRX, IMSI offset.

**ATTACH COMPLETE** (UE → net): Mandatory IEs: ESM message container (LV-E). Sent after bearer context activates.

**ATTACH REJECT** (net → UE): Mandatory IEs: EMM cause. Optional: ESM message container, T3346 (congestion backoff), T3402, Extended EMM cause.

**ATTACH REQUEST** (UE → net): Mandatory: EPS attach type, NAS KSI, EPS mobile identity, UE network capability, ESM message container. ~40 optional IEs covering: Old GUTI/P-TMSI sig, last TAI, DRX parameters, MS classmarks 2/3, Supported Codecs, voice domain preference, Device properties, T3324, Extended DRX, UE additional security capability, UE status, N1 UE NC (5GS interworking), WUS assistance, NB-S1 DRX, IMSI offset. ESM information transfer flag triggers ESM INFORMATION REQUEST sub-procedure before attach completes.

---

### §8.2.5–8.2.8 — Authentication Messages

**AUTHENTICATION FAILURE** (UE → net): Mandatory: EMM cause. Optional: Authentication failure parameter (IEI=30, TLV 16 oct) — AUTS value, included **only** if cause = #21 (synch failure).

**AUTHENTICATION REJECT** (net → UE): No IEs beyond message type. UE must abort all activities, mark USIM invalid.

**AUTHENTICATION REQUEST** (net → UE): Mandatory: NAS KSI_ASME (1/2 oct), RAND (16 oct, V), AUTN (LV 17 oct).

**AUTHENTICATION RESPONSE** (UE → net): Mandatory: Authentication response parameter (LV 5-17 oct) — RES from USIM (4-16 oct) or RES* for 5G AKA (16 oct).

---

### §8.2.9 — CS Service Notification

Sent by network to UE when a CS paging with CS call indicator arrives via SGs and a NAS signalling connection is already established. Mandatory: Paging identity. Optional: CLI (IEI=60), SS Code (IEI=61), LCS indicator (IEI=62), LCS client identity (IEI=63).

---

### §8.2.10–8.2.11 — Detach Messages

**DETACH ACCEPT**: Two variants, same format (just message type):
- Network → UE: for UE-originating detach (network confirms)
- UE → network: for UE-terminated detach (UE confirms)

**DETACH REQUEST**:
- UE-originating (UE → net): Mandatory: Detach type (1/2 oct) + NAS KSI (1/2 oct) + EPS mobile identity (LV 5-12 oct)
- Network-initiated (net → UE): Mandatory: Detach type (1/2 oct) + Spare half octet. Optional: EMM cause (IEI=53, TV 2 oct) — e.g. cause #2 (IMSI detach)

---

### §8.2.12–8.2.13 — NAS Transport and EMM Information

**DOWNLINK NAS TRANSPORT** (net → UE): Mandatory: NAS message container (LV 3-252 oct). Carries encapsulated SMS, LCS, SS messages.

**EMM INFORMATION** (net → UE, **local significance**): All optional: Full name for network (IEI=43), Short name (IEI=45), Local time zone (IEI=46, TV 2 oct), Universal time and local time zone (IEI=47, TV 8 oct), Network daylight saving time (IEI=49, TLV 3 oct). UE may ignore this message entirely.

---

### §8.2.14 — EMM STATUS

Sent by either side at any time to report error conditions (§7). Both directions, **local significance**. Mandatory: EMM cause (V 1 oct).

---

### §8.2.15 — Extended Service Request

Purpose: CSFB/1xCS fallback or packet services requiring more info than SERVICE REQUEST. Mandatory: Service type (1/2 oct) + NAS KSI (1/2 oct) + M-TMSI (Mobile identity LV 6 oct). Conditional: CSFB response (IEI=B-, TV 1 oct) — included only for MT CSFB. Optional: EPS bearer context status (IEI=57), Device properties (IEI=D-), UE request type (IEI=29, TLV 3 oct — MUSIM), Paging restriction (IEI=28, TLV 3-5 oct — MUSIM).

---

### §8.2.16–8.2.17 — GUTI Reallocation

**GUTI REALLOCATION COMMAND** (net → UE): Mandatory: GUTI (EPS mobile identity LV 12 oct). Optional: TAI list (IEI=54, TLV 8-98 oct), DCN-ID (IEI=65, TLV 4 oct), UE radio capability ID (IEI=66, TLV 3-n), UE radio capability ID deletion indication (IEI=B-, TV 1 oct).

**GUTI REALLOCATION COMPLETE** (UE → net): No IEs beyond message type.

---

### §8.2.18–8.2.19 — Identification

**IDENTITY REQUEST** (net → UE): Mandatory: Identity type (1/2 oct) + Spare half octet.

**IDENTITY RESPONSE** (UE → net): Mandatory: Mobile identity (LV 4-10 oct) — IMSI, IMEI, IMEISV, TMSI, or no identity.

---

### §8.2.20–8.2.22 — Security Mode Control

**SECURITY MODE COMMAND** (net → UE): Mandatory: Selected NAS security algorithms (V 1 oct), NAS KSI (1/2 oct), Spare half octet, Replayed UE security capabilities (LV 3-6 oct). Optional: IMEISV request (IEI=C-, TV 1 oct), Replayed nonce_UE (IEI=55, TV 5 oct), Nonce_MME (IEI=56, TV 5 oct), Hash_MME (IEI=4F, TLV 10 oct — required during attach/TAU if ATTACH/TAU REQUEST lacked integrity), Replayed UE additional security capability (IEI=6F, TLV 6 oct), UE radio capability ID request (IEI=37, TLV 3 oct).

**SECURITY MODE COMPLETE** (UE → net): Optional: IMEISV (IEI=23, TLV 11 oct — if IMEISV was requested), Replayed NAS message container (IEI=79, TLV-E 3-n — if Hash_MME mismatch), UE radio capability ID (IEI=66, TLV 3-n — if requested).

**SECURITY MODE REJECT** (UE → net): Mandatory: EMM cause (V 1 oct). Causes: #23 (UE security capabilities mismatch) or #24 (security mode rejected).

---

### §8.2.23 — Security Protected NAS Message

Wrapper for any protected NAS message. Both directions. Structure: Protocol discriminator + Security header type + Message authentication code (V 4 oct) + Sequence number (V 1 oct) + NAS message (V 1-n oct). Note: SECURITY PROTECTED NAS MESSAGE and SERVICE REQUEST are not plain NAS messages and shall not appear as inner messages.

---

### §8.2.24 — Service Reject

Mandatory: EMM cause (V 1 oct). Conditional: T3442 (IEI=5B, TV 2 oct) — included if cause = #39 (CS service temporarily not available). Optional: T3346 value (IEI=5F, TLV 3 oct — MM congestion control), T3448 value (IEI=6B, TLV 3 oct — CP data congestion).

---

### §8.2.25 — Service Request

Non-standard L3 structure (does not use the standard security header + message type layout). Mandatory: KSI and sequence number (V 1 oct — NAS KSI [3 bits] + 5-bit sequence number = 5 LSBs of NAS COUNT), Message authentication code short (V 2 oct — Short MAC computed over NAS COUNT).

---

### §8.2.26 — Tracking Area Update Accept

Mandatory: EPS update result (1/2 oct) + Spare half octet. Large set of optional IEs:

| IEI | IE | Format | Length |
|-----|----|--------|--------|
| 5A | T3412 value | TV | 2 |
| 50 | GUTI | TLV | 13 |
| 54 | TAI list | TLV | 8-98 |
| 57 | EPS bearer context status | TLV | 4 |
| 13 | Location area identification (combined TAU) | TV | 6 |
| 23 | MS identity (TMSI for combined TAU) | TLV | 7-10 |
| 53 | EMM cause (EPS-only success in combined TAU) | TV | 2 |
| 17 | T3402 | TV | 2 |
| 59 | T3423 (ISR timer) | TV | 2 |
| 4A | Equivalent PLMNs | TLV | 5-47 |
| 34 | Emergency number list | TLV | 5-50 |
| 64 | EPS network feature support | TLV | 3-4 |
| F- | Additional update result | TV | 1 |
| 5E | T3412 extended value | TLV | 3 |
| 6A | T3324 value (PSM active timer) | TLV | 3 |
| 6E | Extended DRX parameters | TLV | 3 |
| 65 | DCN-ID | TLV | 4 |
| 6B | T3448 value | TLV | 3 |
| 6C | T3447 value (Service Gap Control) | TLV | 3 |
| 7C | Ciphering key data | TLV-E | 35-2291 |
| 66 | UE radio capability ID | TLV | 3-n |
| 35/36 | Negotiated WUS assistance / NB-S1 DRX | TLV | 3-n |
| 38 | Negotiated IMSI offset (MUSIM) | TLV | 4 |
| 37 | EPS additional request result | TLV | 3 |

---

### §8.2.27–8.2.29 — TAU Complete / Reject / Request

**TAU COMPLETE** (UE → net): No IEs beyond message type. Sent only if GUTI was reallocated or new TMSI assigned.

**TAU REJECT** (net → UE): Mandatory: EMM cause. Optional: T3346 (IEI=5F, TLV 3 oct), Extended EMM cause (IEI=A-, TV 1 oct).

**TAU REQUEST** (UE → net): Mandatory: EPS update type (1/2 oct) + NAS KSI (1/2 oct) + Old GUTI (LV 12 oct). Large set of optional IEs including: Non-current native NAS KSI, GPRS ciphering key SN, Old P-TMSI signature, Additional GUTI, Nonce_UE, UE network capability, Last visited registered TAI, DRX parameters, UE radio capability info update needed, EPS bearer context status, MS network capability, Old LAI, TMSI status, MS classmark 2/3, Supported Codecs, Additional update type (SMS only/CIoT), Voice domain preference, Old GUTI type, Device properties, MS network feature support, TMSI based NRI container, T3324 (PSM), T3412 extended value, Extended DRX, UE additional security capability, UE status, N1 UE NC, UE radio capability ID availability, WUS assistance, NB-S1 DRX, Requested IMSI offset (MUSIM), UE request type (MUSIM), Paging restriction (MUSIM).

---

### §8.2.30–8.2.32 — NAS Transport (Uplink, Generic)

**UPLINK NAS TRANSPORT** (UE → net): Mandatory: NAS message container (LV 3-252 oct).

**DOWNLINK GENERIC NAS TRANSPORT** (net → UE): Mandatory: Generic message container type (V 1 oct) + Generic message container (LV-E 3-n oct). Optional: Additional information (IEI=65, TLV 3-n oct).

**UPLINK GENERIC NAS TRANSPORT** (UE → net): Same structure as downlink.

---

### §8.2.33 — Control Plane Service Request

NB-S1 CIoT specific. Mandatory: Control plane service type (1/2 oct) + NAS KSI (1/2 oct). Optional: ESM message container (IEI=78, TLV-E 3-n — piggybacked PDN CONNECTIVITY REQUEST), NAS message container (IEI=67, TLV 4-253 — for pending SMS in IDLE), EPS bearer context status (IEI=57), Device properties (IEI=D-), UE request type (IEI=29, MUSIM), Paging restriction (IEI=28, MUSIM).

---

### §8.2.34 — Service Accept

Sent by network in response to SERVICE REQUEST, EXTENDED SERVICE REQUEST, or CONTROL PLANE SERVICE REQUEST. Optional: EPS bearer context status (IEI=57, TLV 4 oct), T3448 value (IEI=6B, TLV 3 oct), EPS additional request result (IEI=37, TLV 3 oct).

---

## §8.3 — EPS Session Management (ESM) Messages

### Message Quick-Reference Table

| §  | Message | Direction | Significance |
|----|---------|-----------|-------------|
| 8.3.1 | ACTIVATE DEDICATED EPS BEARER CONTEXT ACCEPT | UE → net | dual |
| 8.3.2 | ACTIVATE DEDICATED EPS BEARER CONTEXT REJECT | UE → net | dual |
| 8.3.3 | ACTIVATE DEDICATED EPS BEARER CONTEXT REQUEST | net → UE | dual |
| 8.3.4 | ACTIVATE DEFAULT EPS BEARER CONTEXT ACCEPT | UE → net | dual |
| 8.3.5 | ACTIVATE DEFAULT EPS BEARER CONTEXT REJECT | UE → net | dual |
| 8.3.6 | ACTIVATE DEFAULT EPS BEARER CONTEXT REQUEST | net → UE | dual |
| 8.3.7 | BEARER RESOURCE ALLOCATION REJECT | net → UE | dual |
| 8.3.8 | BEARER RESOURCE ALLOCATION REQUEST | UE → net | dual |
| 8.3.9 | BEARER RESOURCE MODIFICATION REJECT | net → UE | dual |
| 8.3.10 | BEARER RESOURCE MODIFICATION REQUEST | UE → net | dual |
| 8.3.11 | DEACTIVATE EPS BEARER CONTEXT ACCEPT | UE → net | dual |
| 8.3.12 | DEACTIVATE EPS BEARER CONTEXT REQUEST | net → UE | dual |
| 8.3.12A | ESM DUMMY MESSAGE | both | dual |
| 8.3.13 | ESM INFORMATION REQUEST | net → UE | dual |
| 8.3.14 | ESM INFORMATION RESPONSE | UE → net | dual |
| 8.3.15 | ESM STATUS | both | dual |
| 8.3.16 | MODIFY EPS BEARER CONTEXT ACCEPT | UE → net | dual |
| 8.3.17 | MODIFY EPS BEARER CONTEXT REJECT | UE → net | dual |
| 8.3.18 | MODIFY EPS BEARER CONTEXT REQUEST | net → UE | dual |
| 8.3.18A | NOTIFICATION | net → UE | **local** |
| 8.3.19 | PDN CONNECTIVITY REJECT | net → UE | dual |
| 8.3.20 | PDN CONNECTIVITY REQUEST | UE → net | dual |
| 8.3.21 | PDN DISCONNECT REJECT | net → UE | dual |
| 8.3.22 | PDN DISCONNECT REQUEST | UE → net | dual |
| 8.3.23 | REMOTE UE REPORT | UE → net | dual |
| 8.3.24 | REMOTE UE REPORT RESPONSE | net → UE | dual |
| 8.3.25 | ESM DATA TRANSPORT | both | dual |

Note: All ESM messages carry EPS bearer identity (1/2 oct) and Procedure transaction identity (PTI, 1 oct) in the header.

---

### §8.3.1–8.3.3 — Dedicated Bearer Activation

**ACTIVATE DEDICATED EPS BEARER CONTEXT ACCEPT** (UE → net): Optional: PCO (IEI=27, TLV 3-253), NBIFOM container (IEI=33, TLV 3-257), Extended PCO (IEI=7B, TLV-E 4-65538).

**ACTIVATE DEDICATED EPS BEARER CONTEXT REJECT** (UE → net): Mandatory: ESM cause (V 1 oct). Optional: PCO, NBIFOM container, Extended PCO.

**ACTIVATE DEDICATED EPS BEARER CONTEXT REQUEST** (net → UE): Mandatory: Linked EBI (1/2 oct) + Spare + EPS QoS (LV 2-14) + TFT (LV 2-256). Optional: Transaction identifier, Negotiated QoS, Negotiated LLC SAPI, Radio priority, Packet flow identifier, PCO (IEI=27), WLAN offload indication (IEI=C-), NBIFOM container (IEI=33), Extended PCO (IEI=7B), Extended EPS QoS (IEI=5C, TLV 12 oct — for rates exceeding EPS QoS max values).

---

### §8.3.4–8.3.6 — Default Bearer Activation

**ACTIVATE DEFAULT EPS BEARER CONTEXT ACCEPT** (UE → net): Optional: PCO (IEI=27), Extended PCO (IEI=7B).

**ACTIVATE DEFAULT EPS BEARER CONTEXT REJECT** (UE → net): Mandatory: ESM cause. Optional: PCO, Extended PCO.

**ACTIVATE DEFAULT EPS BEARER CONTEXT REQUEST** (net → UE): Mandatory: EPS QoS (LV 2-14), APN (LV 2-101), PDN address (LV 6-14). Optional IEs include:
- Transaction identifier, Negotiated QoS, LLC SAPI, Radio priority, Packet flow ID (GERAN/UTRAN mobility)
- APN-AMBR (IEI=5E, TLV 4-8)
- ESM cause (IEI=58 — used when allocated PDN type differs from requested)
- PCO (IEI=27) / Extended PCO (IEI=7B)
- Connectivity type (IEI=B- — LIPA PDN indicator)
- WLAN offload indication (IEI=C-)
- NBIFOM container (IEI=33)
- Header compression configuration (IEI=66, TLV 5-257 — for CIoT control-plane optimization)
- Control plane only indication (IEI=9- — this PDN connection is control-plane CIoT only)
- Serving PLMN rate control (IEI=6E, TLV 4 — max ESM DATA TRANSPORT messages per 6-min interval)
- Extended APN-AMBR (IEI=5F, TLV 8 — for rates exceeding APN-AMBR field limits)

---

### §8.3.7–8.3.10 — Bearer Resource Management

**BEARER RESOURCE ALLOCATION REJECT** (net → UE): Mandatory: ESM cause. Optional: PCO, Back-off timer value (IEI=37, TLV 3 — not included if cause = #65 max bearers reached), Re-attempt indicator (IEI=6B — not included if cause = #26 insufficient resources), NBIFOM, Extended PCO.

**BEARER RESOURCE ALLOCATION REQUEST** (UE → net): Mandatory: Linked EBI (1/2 oct) + Spare + Traffic flow aggregate (LV 2-256) + Required traffic flow QoS (LV 2-14). Optional: PCO, Device properties, NBIFOM, Extended PCO, Extended EPS QoS.

**BEARER RESOURCE MODIFICATION REJECT** (net → UE): Mandatory: ESM cause. Optional: PCO, Back-off timer value, Re-attempt indicator, NBIFOM, Extended PCO.

**BEARER RESOURCE MODIFICATION REQUEST** (UE → net): Mandatory: EPS bearer identity for packet filter (1/2 oct) + Spare + Traffic flow aggregate (LV 2-256). Optional: Required traffic flow QoS (IEI=5B, TLV 3-15), ESM cause (IEI=58 — for resource release), PCO (IEI=27), Device properties, NBIFOM (IEI=33), Header compression configuration (IEI=66 — for ROHC re-negotiation or CIoT inter-system), Extended PCO (IEI=7B), Extended EPS QoS (IEI=5C).

---

### §8.3.11–8.3.12A — Bearer Deactivation and Dummy

**DEACTIVATE EPS BEARER CONTEXT ACCEPT** (UE → net): Optional: PCO (IEI=27), Extended PCO (IEI=7B).

**DEACTIVATE EPS BEARER CONTEXT REQUEST** (net → UE): Mandatory: ESM cause. Optional: PCO (IEI=27), T3396 value (IEI=37, TLV 3 — APN back-off timer, included if cause = #26 insufficient resources), WLAN offload indication (IEI=C- — NOT included if deactivating all bearers of a PDN), NBIFOM (IEI=33), Extended PCO (IEI=7B).

**ESM DUMMY MESSAGE** (both): No IEs beyond message type. Sent in ESM message container within ATTACH REQUEST if UE does not request any PDN connection during attach.

---

### §8.3.13–8.3.15 — ESM Information and Status

**ESM INFORMATION REQUEST** (net → UE): No IEs beyond message type. Triggers ESM INFORMATION RESPONSE to obtain security-protected APN or PCO.

**ESM INFORMATION RESPONSE** (UE → net): Optional: APN (IEI=28, TLV 3-102), PCO (IEI=27, TLV 3-253), Extended PCO (IEI=7B). PCO and Extended PCO are mutually exclusive depending on UE/network support.

**ESM STATUS** (both): Mandatory: ESM cause (V 1 oct). Reports error conditions listed in §7.

---

### §8.3.16–8.3.18A — Bearer Modification

**MODIFY EPS BEARER CONTEXT ACCEPT** (UE → net): Optional: PCO (IEI=27), NBIFOM (IEI=33), Extended PCO (IEI=7B).

**MODIFY EPS BEARER CONTEXT REJECT** (UE → net): Mandatory: ESM cause. Optional: PCO, NBIFOM, Extended PCO.

**MODIFY EPS BEARER CONTEXT REQUEST** (net → UE): All IEs optional: New EPS QoS (IEI=5B, TLV 3-15), TFT (IEI=36, TLV 3-257), New QoS (IEI=30), Negotiated LLC SAPI (IEI=32), Radio priority (IEI=8-), Packet flow ID (IEI=34), APN-AMBR (IEI=5E, TLV 4-8), PCO (IEI=27), WLAN offload indication (IEI=C-), NBIFOM (IEI=33), Header compression configuration (IEI=66 — for CIoT ROHC re-negotiation), Extended PCO (IEI=7B), Extended APN-AMBR (IEI=5F), Extended EPS QoS (IEI=5C).

**NOTIFICATION** (net → UE, **local significance**): Mandatory: Notification indicator (LV 2 oct). Carries events relevant to upper layers, e.g. SRVCC handover cancel.

---

### §8.3.19–8.3.22 — PDN Connectivity / Disconnect

**PDN CONNECTIVITY REJECT** (net → UE): Mandatory: ESM cause. Optional: PCO, Back-off timer value (IEI=37), Re-attempt indicator (IEI=6B), NBIFOM, Extended PCO. Back-off timer not included if cause is one of: #28/50/51/54/57/58/61/65.

**PDN CONNECTIVITY REQUEST** (UE → net): Mandatory: Request type (1/2 oct) + PDN type (1/2 oct). Optional: ESM information transfer flag (IEI=D-, TV 1 — triggers ESM INFORMATION REQUEST during attach), APN (IEI=28, TLV 3-102 — not included in ATTACH REQUEST or for emergency/RLOS), PCO (IEI=27), Device properties (IEI=C-), NBIFOM (IEI=33), Header compression configuration (IEI=66 — if IPv4/v6/v4v6 + CIoT CP + header compression supported), Extended PCO (IEI=7B).

**PDN DISCONNECT REJECT** (net → UE): Mandatory: ESM cause. Optional: PCO (IEI=27), Extended PCO (IEI=7B).

**PDN DISCONNECT REQUEST** (UE → net): Mandatory: Linked EBI (1/2 oct) + Spare half octet. Optional: PCO (IEI=27), Extended PCO (IEI=7B).

---

### §8.3.23–8.3.25 — Remote UE and Data Transport

**REMOTE UE REPORT** (UE → net): Sent by ProSe UE-to-network relay. Optional: Remote UE Context Connected (IEI=79, TLV-E 3-65538), Remote UE Context Disconnected (IEI=7A, TLV-E 3-65538), ProSe Key Management Function address (IEI=6F, TLV 3-19).

**REMOTE UE REPORT RESPONSE** (net → UE): No IEs beyond message type. Acknowledges REMOTE UE REPORT.

**ESM DATA TRANSPORT** (both): Mandatory: User data container (LV-E 2-n oct). Optional: Release assistance indication (IEI=F-, TV 1 oct) — UE may include to signal no further UL and/or DL data expected (helps network release the NAS signalling connection promptly after CP data transfer).

---

## Message Type Code Tables (§9.8)

### Table 9.8.1 — EMM Message Type Codes (bits 7-1, bit 8=0, bit 7=1)

| Hex | Message |
|-----|---------|
| 41 | Attach request |
| 42 | Attach accept |
| 43 | Attach complete |
| 44 | Attach reject |
| 45 | Detach request |
| 46 | Detach accept |
| 48 | TAU request |
| 49 | TAU accept |
| 4A | TAU complete |
| 4B | TAU reject |
| 4C | Extended service request |
| 4D | Control plane service request |
| 4E | Service reject |
| 4F | Service accept |
| 50 | GUTI reallocation command |
| 51 | GUTI reallocation complete |
| 52 | Authentication request |
| 53 | Authentication response |
| 54 | Authentication reject |
| 5C | Authentication failure |
| 55 | Identity request |
| 56 | Identity response |
| 5D | Security mode command |
| 5E | Security mode complete |
| 5F | Security mode reject |
| 60 | EMM status |
| 61 | EMM information |
| 62 | Downlink NAS transport |
| 63 | Uplink NAS transport |
| 64 | CS Service notification |
| 68 | Downlink generic NAS transport |
| 69 | Uplink generic NAS transport |

### Table 9.8.2 — ESM Message Type Codes (bits 8-7=11)

| Hex | Message |
|-----|---------|
| C1 | Activate default EPS bearer context request |
| C2 | Activate default EPS bearer context accept |
| C3 | Activate default EPS bearer context reject |
| C5 | Activate dedicated EPS bearer context request |
| C6 | Activate dedicated EPS bearer context accept |
| C7 | Activate dedicated EPS bearer context reject |
| C9 | Modify EPS bearer context request |
| CA | Modify EPS bearer context accept |
| CB | Modify EPS bearer context reject |
| CD | Deactivate EPS bearer context request |
| CE | Deactivate EPS bearer context accept |
| D0 | PDN connectivity request |
| D1 | PDN connectivity reject |
| D2 | PDN disconnect request |
| D3 | PDN disconnect reject |
| D4 | Bearer resource allocation request |
| D5 | Bearer resource allocation reject |
| D6 | Bearer resource modification request |
| D7 | Bearer resource modification reject |
| D9 | ESM information request |
| DA | ESM information response |
| DB | Notification |
| DC | ESM dummy message |
| E8 | ESM status |
| E9 | Remote UE report |
| EA | Remote UE report response |
| EB | ESM data transport |
