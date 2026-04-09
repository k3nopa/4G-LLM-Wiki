---
title: "S-CSCF (Serving-CSCF)"
type: entity
tags: [IMS, S-CSCF, SIP, registrar, iFC, ISC, service-control, Cx, routing, VoLTE]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# S-CSCF — Serving Call Session Control Function

**Spec reference:** 3GPP TS 23.228 §4.8

## Role

The S-CSCF is the **central routing and service execution engine of IMS**. It is simultaneously:
- The SIP **registrar** for all IMS subscribers (owns registration state)
- The **session routing function** — all originating and terminating sessions pass through it
- The **iFC evaluation engine** — determines which Application Servers to invoke
- The **Cx client** — downloads subscriber service profiles from HSS

Every IMS subscriber is served by exactly one S-CSCF at a time. The S-CSCF name is stored in the HSS.

---

## Functions

| Function | Description |
|---|---|
| **Registrar** | Processes REGISTER requests; stores contact URIs; manages registration expiry |
| **Download service profile** | On REGISTER, downloads subscriber's service profile (iFCs) from HSS via Cx SAR/SAA |
| **iFC evaluation** | For every initial SIP dialog, evaluates iFCs in priority order; forks to matching ASes via ISC |
| **Originating routing** | After iFC chain, routes INVITE toward terminating side (ENUM/DNS, BGF, or BGCF) |
| **Terminating routing** | For terminating sessions, routes to UE contact URI (via P-CSCF) |
| **SIP B2BUA or proxy** | Acts as SIP proxy or B2BUA depending on service requirements |
| **Authentication** | Verifies IMS AKA authentication during registration (AUTN check, RES/XRES comparison) with HSS challenge |
| **Charging** | Generates CDRs; interfaces with charging systems |
| **SIP timer management** | Enforces registration expiry; de-registers UEs that miss re-registration |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **Mw** | P-CSCF, I-CSCF | SIP | Receive REGISTER/INVITE; return responses |
| **ISC** | AS (TAS, MMTEL, etc.) | SIP | Trigger service logic; fork request to AS |
| **Cx** | HSS | Diameter | SAR/SAA (profile download), MAR/MAA (auth), RTR/RTA (network de-reg), PPR/PPA (profile push) |
| **Mi** | BGCF | SIP | Route sessions toward PSTN breakout |
| **Mr** | MRFC | SIP | Request conference/announcement resources |
| **Mm** | External IP network | SIP | Route sessions toward external SIP networks |

---

## Registration Flow

```
UE──REGISTER──►P-CSCF──►I-CSCF──Cx UAR──►HSS
                                  ◄──UAA──
                         I-CSCF──►S-CSCF
                                   │Cx SAR──►HSS
                                   │◄──SAA (service profile)
                                   │Cx MAR──►HSS (auth challenge)
                                   │◄──MAA (AUTN, XRES, CK, IK)
                                   │──401 Unauthorized──►UE (AUTN)
                                   │
                         UE──REGISTER (with RES)──►
                                   │ verify RES==XRES
                                   │Cx SAR (re-register)──►HSS
                                   │◄──SAA
                                   └──200 OK──►UE
```

---

## iFC Evaluation (Service Control)

On every **initial** SIP dialog (new INVITE, REGISTER, MESSAGE, SUBSCRIBE, etc.):

1. S-CSCF holds the subscriber's **service profile** (downloaded from HSS during registration)
2. Service profile contains a list of **iFCs** ordered by priority (lower = higher priority)
3. Each iFC has:
   - **Priority** (integer)
   - **Service Point Triggers (SPTs)**: conditions on SIP method, headers, body, session direction (orig/term), request-URI
   - **Application Server URI** (SIP URI)
   - **Default handling**: continue (if AS unresponsive) or terminate session
4. S-CSCF evaluates each iFC in priority order
5. If SPT conditions match → **fork request to AS via ISC**
6. After AS returns (or if AS is a proxy and routes back), continue to next iFC

### iFC Matching Example
```
iFC priority=1:
  SPT: Method=INVITE, Direction=Originating
  AS: sip:mmtel.ims.operator.com
  DefaultHandling: CONTINUE

→ All originating INVITEs go through MMTEL AS (call hold, CLIP, CLIR, etc.)
```

### AS Return Routing
AS returns request to S-CSCF via Route header containing the **ODI** (Original Dialog Identifier) — an S-CSCF-internal token that lets it resume the iFC chain at the correct position.

---

## Session Routing (Post-iFC)

After iFC chain completes:

**Originating:**
- S-CSCF resolves destination: ENUM/DNS lookup on Request-URI
- If tel URI → ENUM → SIP URI → route to terminating S-CSCF (via I-CSCF or directly)
- If PSTN number → route to BGCF (Mi)
- If IMS subscriber in same PLMN → route directly to their S-CSCF

**Terminating:**
- S-CSCF has registration binding → route to UE's P-CSCF
- If UE not registered → return 480 Temporarily Unavailable (or trigger PSTN fallback)

---

## Capabilities

S-CSCF instances have **mandatory and optional capabilities** defined by the operator. During S-CSCF assignment (I-CSCF via Cx), the HSS returns required capabilities, and the I-CSCF matches against available S-CSCFs. This allows:
- Specialized S-CSCFs for high-value subscribers
- Geographic/load-balanced pools
- Feature-specific S-CSCFs (e.g. only some support IMS conferencing)

---

## Key Timers

| Timer | Purpose |
|---|---|
| Registration Expiry | Negotiated in REGISTER (Contact: expires=). S-CSCF de-registers if UE misses refresh. |
| Network Initiated De-registration | HSS can push RTR (Registration Termination Request) via Cx; S-CSCF de-registers all contacts |

---

## Related Pages
- [P-CSCF](P-CSCF.md) — sends REGISTER/INVITE to S-CSCF
- [I-CSCF](I-CSCF.md) — assigns S-CSCF; routes inbound
- [TAS](TAS.md) — primary ISC peer for telephony services
- [HSS](HSS.md) — service profile, auth, Cx
- [IMS Identity Model](../concepts/IMS-identity-model.md) — IMPU, iFC, service profile
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
