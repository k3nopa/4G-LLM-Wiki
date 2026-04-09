---
title: "TS 23.228 §4 — IMS Architecture & Functional Entities"
type: source
tags: [IMS, CSCF, P-CSCF, I-CSCF, S-CSCF, TAS, BGCF, MRF, HSS, identity, IMPI, IMPU, GRUU, SIP, roaming]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# Source: 3GPP TS 23.228 §4 — IMS Architecture & Functional Entities

**Spec:** TS 23.228 v15.6.0  
**PDF pages read:** 27–66 (spec pages 26–65)  
**Section covered:** §4 — IP Multimedia Subsystem

---

## Architecture Overview

IMS is a service layer overlay on top of the EPC (or any IP-CAN). It provides:
- SIP-based multimedia session control
- Service logic via Application Servers (AS)
- Tight integration with EPC policy (Rx→PCRF→Gx→PGW)
- Identity management separate from access (IMPI/IMPU vs MSISDN/IMSI)

The IMS sits "above" the IP transport; it does not itself provide connectivity. An IMS subscriber needs both an EPC PDN connection (via an IMS APN) and IMS registration.

---

## Functional Entities Covered

### CSCF Family
- **P-CSCF** (Proxy-CSCF): UE's entry point into IMS. SIP proxy + security. AF on Rx.
- **I-CSCF** (Interrogating-CSCF): PLMN entry for inbound. S-CSCF assignment via HSS Cx.
- **S-CSCF** (Serving-CSCF): Registrar + session routing engine. ISC to ASes. Central to all sessions.

### Other Core Nodes
- **BGCF** (Breakout Gateway Control Function): Routes sessions toward PSTN/CS via MGCF.
- **MRFC** (Multimedia Resource Function Controller): Controls conference/tone resources.
- **MRFP** (Multimedia Resource Function Processor): Media plane for MRFC. Mixes RTP streams.
- **TAS** (Telephony Application Server): Executes telephony services (call hold, CLIP, conferencing) via ISC.
- **HSS**: Subscriber home; Cx interface to S-CSCF/I-CSCF; Sh interface to ASes.
- **ENUM/DNS**: Telephone number → SIP URI translation. Used during session routing.
- **IP-SM-GW**: Routes SMS over IMS when UE registers with tel URI.

---

## Reference Points Identified

| Interface | Between | Protocol | Purpose |
|---|---|---|---|
| **Gm** | UE ↔ P-CSCF | SIP | UE registration, sessions |
| **Mw** | P-CSCF ↔ I-CSCF, P-CSCF ↔ S-CSCF, I-CSCF ↔ S-CSCF | SIP | Intra-IMS SIP |
| **ISC** | S-CSCF ↔ AS | SIP | Service control; iFC trigger |
| **Cx** | S-CSCF ↔ HSS, I-CSCF ↔ HSS | Diameter | S-CSCF assignment, profile download, auth |
| **Sh** | AS ↔ HSS | Diameter | AS access to subscriber data |
| **Mg** | MGCF → I-CSCF | SIP | PSTN→IMS inbound call entry |
| **Mi** | S-CSCF → BGCF | SIP | Route to PSTN breakout |
| **Mj** | BGCF → MGCF | SIP | BGCF routes to MGCF |
| **Mk** | BGCF → BGCF | SIP | Inter-PLMN BGCF routing |
| **Mx** | BGCF ↔ IBT (IMS-ALG) | SIP | Inter-IMS (external network) |
| **Mr** | S-CSCF ↔ MRFC | SIP | Conference/media resource control |
| **Mp** | MRFC → MRFP | H.248 | MRFC controls MRFP media |
| **Ma** | I-CSCF → AS | SIP | I-CSCF directly terminates to AS |
| **Mm** | IMS ↔ external IP network | SIP | External SIP network interop |
| **Rx** | P-CSCF → PCRF | Diameter | AF session info → policy |
| **Ut** | UE ↔ AS | HTTP(S)/XCAP | Subscriber-managed AS configuration |

---

## Identity Model Summary

| Identity | Type | Usage |
|---|---|---|
| IMPI | Private (NAI format) | Authentication only; never in SIP Request-URI |
| IMPU | Public (SIP URI or tel URI) | Signaling, called/calling party |
| GRUU | Globally Routable UA URI | Targets a specific UE instance (P-GRUU persistent, T-GRUU temporary) |
| Wildcarded IMPU | Pattern (tel URI range) | Bulk registration — not individually signaled |

One IMPI can have multiple IMPUs. IMPUs can belong to an Implicit Registration Set — registering one registers all.

---

## Service Control Model (S-CSCF + iFC)

1. Each IMPU has a **service profile** in HSS
2. Service profile contains **iFCs** (Initial Filter Criteria)
3. Each iFC has a priority + **Service Point Triggers (SPTs)** (method, headers, body, direction)
4. S-CSCF evaluates iFCs sequentially on every "initial" SIP dialog
5. On SPT match → S-CSCF forks request to **AS via ISC**
6. AS can act as: originating UA, terminating UA, SIP proxy, B2BUA, 3rd party call control

AS returns control to S-CSCF via `Route` header with original dialog URI.

---

## Roaming Architectures

### Home Routed (standard)
```
UE → P-CSCF(VPLMN) → I-CSCF(HPLMN) → S-CSCF(HPLMN)
     P-CSCF: Rx → V-PCRF → S9 → H-PCRF
```

### Local Breakout (§4.15a)
```
UE → P-CSCF(VPLMN) → S-CSCF(HPLMN)  [P-CSCF directly to H-IMS via Mw]
     P-CSCF: Rx → V-PCRF → S9 → H-PCRF
```

---

## Key Findings

- P-CSCF is the **sole AF** for IMS on Rx; it maps SIP SDP → Rx AAR to authorize GBR bearer
- S-CSCF is the **central routing engine** — all sessions flow through it; it owns iFC evaluation
- I-CSCF's primary role is **S-CSCF selection** (Cx UAR/UAA); in LBO it may be bypassed
- S-CSCF capabilities are matched against required capabilities to pick the right S-CSCF
- GRUU: T-GRUU is `+sip.instance` based; P-GRUU is durable across registrations
- Wildcarded IMPU: allows carrier to register entire number range without per-IMPU signaling
- BGCF decides whether PSTN breakout is local (→MGCF via Mj) or another PLMN (→BGCF via Mk)
- MRFC/MRFP split: controller (SIP/H.248) vs media processor (RTP mixing, tone injection)
- IP-SM-GW: when UE registers with tel URI, SMS can be delivered via IMS to IP-SM-GW → SMS-GMSC

---

## Entities Documented

- [P-CSCF](../entities/P-CSCF.md)
- [I-CSCF](../entities/I-CSCF.md)
- [S-CSCF](../entities/S-CSCF.md)
- [BGCF](../entities/BGCF.md)
- [MRF (MRFC+MRFP)](../entities/MRF.md)
- [TAS](../entities/TAS.md)
- [HSS — IMS data](../entities/HSS.md) (updated)

## Concepts Documented

- [IMS Identity Model](../concepts/IMS-identity-model.md)

## Interfaces Documented

- [IMS Reference Points](../interfaces/IMS-reference-points.md)

---

## Questions Raised

- What triggers S-CSCF **re-assignment** (e.g. if S-CSCF fails)? HSS retains S-CSCF name; I-CSCF re-queries on next REGISTER.
- How does P-CSCF know which PCRF to address on Rx in a roaming scenario? V-PCRF address is provisioned or discovered via DNS in VPLMN.
- What is the GRUU re-allocation lifecycle for T-GRUU during re-registration?
- How does IP-SM-GW interwork with legacy SMS-SC — via SGd or S6c Diameter interfaces?
