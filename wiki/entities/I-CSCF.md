---
title: "I-CSCF (Interrogating-CSCF)"
type: entity
tags: [IMS, I-CSCF, SIP, S-CSCF assignment, Cx, HSS, routing, THIG]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# I-CSCF — Interrogating Call Session Control Function

**Spec reference:** 3GPP TS 23.228 §4.7

## Role

The I-CSCF is the **entry point into a home IMS network** for inbound SIP traffic. Its primary purpose is **S-CSCF selection and query**: for any inbound REGISTER or session, it queries the HSS (via Cx) to find or assign an S-CSCF, then forwards the request to that S-CSCF.

The I-CSCF is optional as an intermediary in fully trusted inter-PLMN scenarios, but in practice it always exists as the THIG (Topology Hiding Inter-network Gateway) boundary.

---

## Functions

| Function | Description |
|---|---|
| **S-CSCF assignment** (REGISTER) | Queries HSS (Cx UAR) to determine which S-CSCF already serves the UE, or obtains capabilities to select one |
| **S-CSCF assignment** (session) | For incoming sessions: queries HSS (Cx LIR) to find the registered S-CSCF |
| **THIG** | Topology Hiding Inter-network Gateway — encrypts/strips Route and Record-Route headers before crossing PLMN boundary |
| **Load balancing** | Selects among multiple S-CSCFs satisfying capability requirements |
| **Entry for PSTN→IMS** | MGCF routes inbound PSTN calls to I-CSCF (Mg interface) |
| **Entry from external SIP** | External SIP networks reach IMS via I-CSCF (Mm interface) |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **Mw** | P-CSCF, S-CSCF | SIP | Inbound REGISTER/session forwarding |
| **Cx** | HSS | Diameter | S-CSCF query: UAR/UAA (REGISTER path), LIR/LIA (session path) |
| **Mg** | MGCF | SIP | PSTN inbound entry |
| **Mm** | External IP network | SIP | External SIP entry |
| **Ma** | AS | SIP | Direct termination to AS for certain service scenarios |

---

## Cx Procedures Used

| Procedure | Request/Answer | Trigger |
|---|---|---|
| **UAR/UAA** (User Authorization) | I-CSCF → HSS | REGISTER arrives; check if UE already has S-CSCF; if not, get capabilities for selection |
| **LIR/LIA** (Location Info) | I-CSCF → HSS | Session arrives for terminating user; find registered S-CSCF name |

### UAR/UAA Flow (REGISTER path)
```
P-CSCF──REGISTER──►I-CSCF──Cx UAR──►HSS
                            ◄──Cx UAA──
                   [select S-CSCF]
                   ──REGISTER──►S-CSCF
```
UAA returns either:
- S-CSCF name already assigned → route to it
- Capabilities required → I-CSCF picks S-CSCF from available pool

### LIR/LIA Flow (terminating session)
```
External──INVITE──►I-CSCF──Cx LIR──►HSS
                            ◄──Cx LIA──(S-CSCF address)
                   ──INVITE──►S-CSCF
```

---

## S-CSCF Selection

When HSS returns **mandatory and optional capabilities** (not a specific S-CSCF name):
1. I-CSCF consults locally provisioned list of S-CSCFs with their capabilities
2. Selects an S-CSCF that satisfies all mandatory capabilities (optional are best-effort)
3. May load-balance across equally capable S-CSCFs

Capabilities are integer lists with operator-defined semantics (e.g. capability=5 = "can handle forking", capability=7 = "supports IMS conferencing").

---

## THIG

When acting as THIG, the I-CSCF:
- Encrypts the `Route` path so external networks cannot observe internal IMS topology
- Strips internal `Record-Route` entries from responses going out of the PLMN
- Appears as a single opaque entry point to inter-PLMN SIP peers

---

## Related Pages
- [P-CSCF](P-CSCF.md) — sends REGISTER to I-CSCF
- [S-CSCF](S-CSCF.md) — receives REGISTER/INVITE from I-CSCF
- [HSS](HSS.md) — queried via Cx
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
