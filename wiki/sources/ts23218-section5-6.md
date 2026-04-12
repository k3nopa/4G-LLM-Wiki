---
title: "Source: TS 23.218 — IM Call Model, S-CSCF Architecture, AS Interaction Modes, MRB"
type: source
tags: [TS-23.218, IMS, S-CSCF, iFC, SPT, call-model, transit-function, ICSI, IARI, AS, ISC, MRFC, MRB, B2BUA, charging, ICID, IOI, voicemail, transcoding, conference]
sources: [ts_123218v170000p.pdf]
updated: 2026-04-10
---

# Source: 3GPP TS 23.218 v17.0.0 — FULLY INGESTED

**Full title:** IP Multimedia (IM) session handling; IM Call Model; Stage 2
**Version:** 17.0.0 (Release 17)
**PDF:** `raw/papers/ts_123218v170000p.pdf`
**Sections ingested:** Complete — §4–§13 + Annexes A/B/C (PDF pp 12–70)

---

## Scope of Ingested Content

TS 23.218 defines the **IM Call Model** — the normative framework governing how the
S-CSCF determines which Application Servers to invoke for a given SIP request, in
what order, and under what conditions. It is the specification for the iFC evaluation
engine and the S-CSCF functional architecture.

---

## §4 — Abbreviations

Standard 3GPP abbreviation table. No new concepts beyond CLAUDE.md glossary.
Key additions: ODI (Original Dialog Identifier), TIC (Transit Invocation Criteria),
TAS (Telephony Application Server), SCIM (Service Capability Interaction Manager).

---

## §5 — General Description of the IM Call Model

### §5.1 — Service Provision Architecture

Defines the IMS service plane topology:

- **ISC interface**: SIP-based interface between S-CSCF and all AS types (SIP AS,
  IM-SSF, OSA SCS, SCIM). All AS invocations go via ISC.
- **Media resources**: AS reaches MRFC either direct (Mr) or via MRB In-Line mode
  (Rc → MRB → Mr'/Cr → MRFC).
- **ISC gateway function (§5.1.2A)**: THIG + Screening + IMS-ALG/TrGw for inter-network
  ISC; hides topology from third-party ASes; provides SIP screening and NAT/IPv6 translation.
- **IBCF (§5.1.3)**: border control on Mr/Mr'/Cr/Mm/Ici interfaces.

### §5.1A — ICSI

ICSI = IMS Communication Service Identifier. Identifies the IMS service (e.g. MMTel
voice). UE-provided = unauthenticated (stripped by S-CSCF). S-CSCF inserts
authenticated ICSI after iFC validation. Usable as SPT in iFC.

### §5.1B — IARI

IARI = IMS Application Reference Identifier. Identifies the terminating application.
Absent → default application assumed. Usable as SPT in terminating iFC.

### §5.2 — Filter Criteria and SPTs

#### §5.2.1 — SPTs
Six types: SIP Method, Registration Type, Header Presence, Header Content / Request-URI,
Direction (UE-orig/term for registered/unregistered), Session Description (SDP).
Bar check precedes all SPT evaluation. REGISTER = always UE-originating.

#### §5.2.2 — Filter Criteria
Each FC: AS address + priority (unique/user) + trigger point (SPTs with AND/OR/NOT) +
default handling (SESSION_CONTINUED | SESSION_TERMINATED) + optional service info +
include-REGISTER flags.

#### §5.2.3 — S-CSCF iFC Processing Algorithm
Priority-ordered evaluation. ODI added before forwarding to AS. AS returns with same ODI.
AS 4xx → abandon lower FCs. AS denial via Record-Route (not iFC). Priority list locked
until request exits S-CSCF via Mw.

#### §5.2.4–5.2.5 — Transit Invocation Criteria / Transit Function
Same algorithm as §5.2.3 applied to network-configured TIC (not user-specific iFC).
Transit Function has no Registrar.

---

## §6 — S-CSCF Architecture and Handling Procedures

### §6.1 — Functional Models

**S-CSCF (§6.1.1):** ILCM + OLCM + Combined I/OLSM + ILSM + OLSM + Registrar + Notifier.
Each dialog leg can act independently as SIP proxy, redirect server, or UA.

**Transit Function (§6.1.2):** Same minus Registrar.

### §6.2 — S-CSCF Interfaces

| Interface | Peer | Protocol |
|---|---|---|
| Mw | P-CSCF, I-CSCF, S-CSCF | SIP |
| ISC | AS (all types) | SIP |
| Cx | HSS | Diameter |
| MRB (In-Line) | MRB | ISC-like |
| Mr | MRFC | SIP |

### §6.3 — Registration Handling

Authenticate (Cx MAR/MAA) → download iFC (Cx SAR/SAA) → validate ICSI/IARI →
evaluate iFC on REGISTER → third-party REGISTER to each matched AS (or AS subscribes
to reg event package instead) → apply default handling on AS failure.

### §6.4 — Originating Sessions

**Registered (§6.4.1):** Determine served user → bar check → ICSI validation → evaluate
originating iFC → normal routing.

**Unregistered (§6.4.2):** Fetch profile from HSS first (SAR, UNREGISTERED_USER) →
same as registered. After last FC: normal routing (does not reject unlike terminating).

### §6.5 — Terminating Sessions

**Registered (§6.5.1):** GRUU→IMPU mapping → bar check → ICSI/IARI validation →
evaluate terminating iFC → if AS changes Request-URI: option a (re-route), b (switch
to originating iFC), or c (continue terminating iFC then route to new URI) → route to UE.

**Unregistered (§6.5.2):** Fetch profile from HSS → same as registered. If no terminating
iFC matches: **reject session** (unlike originating unregistered which continues to
normal routing).

### §6.5A — Transit Function Handling

Same SPT/TIC evaluation algorithm as S-CSCF but applies to transit (non-registered)
requests using locally configured TIC. No subscriber context.

---

## §6.6–§9 Summary (Chunk 3a-2)

### §6.6 — S-CSCF Session Release
Two modes: proxy (pass-through BYE) and initiate (S-CSCF generates BYE to all dialog
parties simultaneously with independent CSeq numbers).

### §6.7 — S-CSCF Subscription/Notification
Reg event package (RFC 3680): entities subscribe to S-CSCF; NOTIFY carries implicitly
registered IMPUs, GRUUs, ICSIs, IARIs per contact.

### §6.8 — S-CSCF IMS Charging
ICID (from P-CSCF), IOI (operator identity), charging function addresses (from HSS via
Cx). S-CSCF stores and propagates; removes IP-CAN charging info when crossing network
boundaries. CDR generated per session.

### §6.8A — Transit Function Charging
Same parameters; charging addresses locally configured; IOI replaced with own when
forwarding to AS.

### §6.9 — S-CSCF Subscriber Data
Application Server Subscription Information = all iFC in user service profile.
iFC components: AS addr, default handling, trigger point, priority, service info,
include-REGISTER-req/resp flags. Authentication data via Cx.

### §7 — HSS Functional Requirements
Stores data for S-CSCFs (Cx), IM-SSF ASes (Si/Sh), SIP ASes (Sh). Interfaces:
Cx (HSS↔CSCF), Sh (HSS↔AS; also allows AS to activate/deactivate own iFC per subscriber),
Si (HSS↔CSE MAP), Si/Sh (HSS↔IM-SSF).

### §8 — MRFC Functional Requirements
MRFC capabilities: tones/announcements, ad-hoc conferences, transcoding. Interfaces:
Mr (MRFC↔S-CSCF, SIP), Cr (AS↔MRFC, media control + resource fetch), Mr' (direct
AS/MRB↔MRFC without S-CSCF). MRFC supports offer/answer with preconditions on all services.

### §9 — Generic SIP AS Session Handling
AS functional model: AS-ILCM + AS-OLCM + Application Logic.
**5 modes of operation**: (1) Terminating UA/redirect, (2) Originating UA, (3) SIP proxy,
(4) Routing/Initiating B2BUA, (5) Not involved.
AS interfaces: ISC, Sh, Dh, Cr, Rc. Sh allows AS to activate/deactivate own iFC per user.
AS subscriber data: service key, STP, service scripts.
AS session procedures: originating/terminating/registration/release/charging handling.
Charging: AS propagates ICID/IOI; originating UA AS generates ICID if none received.

## §13 + Annexes B/C Summary (Chunk 3a-3)

### §13 — Media Resource Broker (MRB)
MRB pools heterogeneous MRFC resources for multiple applications. Two modes:
- **Query mode**: AS queries MRB via Rc with required attributes; MRB returns MRFC
  address; AS directly establishes dialog with MRFC (via Mr or Mr'); AS explicitly
  notifies MRB when done
- **In-Line mode**: MRB in SIP path between AS and MRFC; AS sends request to MRB
  (Mr' or S-CSCF ISC+Mr); MRB selects MRFC and forwards; subsequent in-dialog
  messages traverse MRB; MRB infers release from BYE
- Selection criteria: resource characteristics, application identity, SLA/QoS, capacity,
  future reservations, visited network MRB
- §13.3: MRB learns resource info via O&M interfaces or direct MRB↔MRFC interface

### Annex A — Scalability/Deployment (informative)
Hierarchical AS architecture patterns; scalability concern with long AS chains on
signaling path; SIP AS as gateway to external ASes outside IM CN subsystem.

### Annex B — Worked Examples (informative)
Call forwarding (CFonCLI): redirect-server mode (302 + UE re-issues INVITE) vs
SIP-proxy mode (181 Call-Is-Being-Forwarded + AS modifies Request-URI).
MRFC services: announcement (B2BUA, 30-step, Call-IDs 1+2+3), ad-hoc conference
(B2BUA, 41-step, 6 Call-IDs, same conference identifier reused for each party),
transcoding (B2BUA, two variants: 53-step with codec in 606, 31-step with no SDP in 606).
Voicemail: out-of-coverage recording (terminating UA, unregistered iFC, 23-step),
on-registration playback (originating UA, third-party REGISTER trigger, 34-step).

### Annex C — iFC Triggering Example (informative)
Two-AS chain: FC-X→AS1, FC-Y→AS2. S-CSCF evaluates FC-X on incoming INVITE → forwards
to AS1 → AS1 returns request → S-CSCF evaluates FC-Y → forwards to AS2 → AS2 returns
→ S-CSCF routes to destination. Confirms: each AS return re-triggers FC evaluation.

## Coverage Status
**FULLY INGESTED** — no remaining sections.
