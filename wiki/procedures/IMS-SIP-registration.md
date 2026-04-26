---
title: "IMS SIP Registration — Stage-3 Procedures (UE / P-CSCF / S-CSCF)"
type: procedure
tags: [SIP, IMS, registration, IMS-AKA, IPsec, P-CSCF, S-CSCF, REGISTER, 401, AKA, GRUU, Service-Route, integrity-protected, SIP-digest]
sources: [ts_124229v160600p.pdf]
updated: 2026-04-18
---

# IMS SIP Registration — Stage-3 Procedures

**Spec**: 3GPP TS 24.229 v16.6.0 §5.1.1, §5.2.2, §5.4.1
**Related pages**: [IMS-AKA-registration.md](IMS-AKA-registration.md) (TS 33.203 SA setup), [IMS-registration.md](IMS-registration.md) (TS 23.228 high-level flow), [SIP-IMS-profile.md](../protocols/SIP-IMS-profile.md) (§4 General profile), [IMS-access-security.md](../concepts/IMS-access-security.md), [IMS-auth-alternatives.md](../concepts/IMS-auth-alternatives.md)

This page captures the normative stage-3 SIP/SDP procedures for IMS registration — the exact headers constructed, security association state transitions, and per-node data stored — from all three node perspectives: **UE** (§5.1.1), **P-CSCF** (§5.2.2), and **S-CSCF** (§5.4.1). Focus is on IMS AKA (the mandatory mechanism); SIP Digest and TLS variants are noted where they differ.

---

## Overview: Two-Phase Registration

IMS registration requires two REGISTER exchanges when IMS AKA is used:

| Phase | REGISTER | Status | Purpose |
|---|---|---|---|
| 1 (Unprotected) | Sent without IPsec SA; Authorization: nonce="" response="" | → 401 Unauthorized | S-CSCF challenges with RAND+AUTN, provides IK+CK for P-CSCF to set up SAs |
| 2 (Protected) | Sent through IPsec SA; Authorization: response=RES | → 200 OK | S-CSCF verifies RES against XRES; registration bindings created |

For SIP Digest and NASS/GPRS-IMS-Bundled authentication, only the second phase applies (there is no "protected" REGISTER for bundled mechanisms — the procedures in §5.4.1.2.1 apply to all REGISTERs).

---

## Full IMS AKA Registration Sequence

```mermaid
sequenceDiagram
    participant UE
    participant P-CSCF
    participant I-CSCF
    participant S-CSCF
    participant HSS

    Note over UE,HSS: Phase 1 — Unprotected REGISTER (no IPsec SA yet)
    UE->>P-CSCF: REGISTER (From/To=IMPU, Auth: nonce="", Security-Client, P-Access-Network-Info)
    Note over P-CSCF: Insert: integrity-protected=absent<br/>Path (IMS flow token + "ob")<br/>P-Charging-Vector (icid + orig-ioi type1)<br/>P-Visited-Network-ID<br/>Store temporary SA (reg-await-auth lifetime)
    P-CSCF->>I-CSCF: REGISTER (routed to home network)
    I-CSCF->>HSS: UAR (query: User-Authorization-Request)
    HSS-->>I-CSCF: UAA (S-CSCF address or capabilities)
    I-CSCF->>S-CSCF: REGISTER
    Note over S-CSCF: Identify user (To→IMPU, Auth→IMPI)<br/>Query HSS via MAR for auth vectors<br/>Check P-Visited-Network-ID<br/>Check IP address in Via
    S-CSCF->>HSS: MAR (Multimedia-Auth-Request)
    HSS-->>S-CSCF: MAA (RAND, AUTN, XRES, IK, CK, AV)
    S-CSCF-->>I-CSCF: 401 Unauthorized (WWW-Authenticate: nonce=RAND+AUTN, ik=IK, ck=CK, alg=AKAv1-MD5)
    Note over S-CSCF: Store RAND; start reg-await-auth timer
    I-CSCF-->>P-CSCF: 401
    Note over P-CSCF: Delete temp SA<br/>Strip ik/ck from WWW-Authenticate<br/>Insert Security-Server (ipsec-3gpp, algorithms)<br/>Set up new temporary SA (reg-await-auth lifetime)
    P-CSCF-->>UE: 401 (WWW-Authenticate: nonce=RAND+AUTN, alg, Security-Server)

    Note over UE,HSS: Phase 2 — Protected REGISTER (over IPsec SA)
    Note over UE: Verify AUTN (MAC, SQN range)<br/>Derive CK, IK from RAND<br/>Set up temporary IPsec SAs<br/>Compute RES
    UE->>P-CSCF: REGISTER [protected] (Auth: response=RES, Security-Verify=Security-Server content)
    Note over P-CSCF: integrity-protected = "yes"<br/>Promote temp SA → newly established SA<br/>Forward (same Call-ID)
    P-CSCF->>I-CSCF: REGISTER
    I-CSCF->>S-CSCF: REGISTER
    Note over S-CSCF: Match Call-ID to 401<br/>Stop reg-await-auth timer<br/>Verify RES vs XRES (RFC 3310/RFC 4169)<br/>SAR to HSS (server assignment)<br/>Store service profiles, iFCs, PUIs<br/>Build preloaded Route from Path<br/>Bind expiry to contact
    S-CSCF->>HSS: SAR (Server-Assignment-Request)
    HSS-->>S-CSCF: SAA (user profile, iFCs, restoration info)
    S-CSCF-->>I-CSCF: 200 OK (P-Associated-URI, Service-Route, Contact+GRUUs, Feature-Caps, P-Charging-Vector)
    I-CSCF-->>P-CSCF: 200 OK
    Note over P-CSCF: Temp SA → newly established SA (taken into use)<br/>Delete previously existing SA immediately<br/>Store Service-Route, P-Associated-URI, CFA, term-ioi<br/>Send SUBSCRIBE (reg event) to S-CSCF
    P-CSCF-->>UE: 200 OK
    Note over UE: Promote temp SA (SIP lifetime = max(prev lifetime, reg-expiry+30s))<br/>Store P-Associated-URI (default PUI = first URI)<br/>Store Service-Route (preloaded Route for non-REGISTER)<br/>Store pub-gruu / temp-gruu<br/>Store Feature-Caps ICSI values
```

---

## UE Procedures (§5.1.1)

### Phase 1: Initial REGISTER headers (§5.1.1.2.1)

| Header | Value / Rule |
|---|---|
| `Request-URI` | Home network SIP URI domain (e.g. `sip:example.com`) |
| `From` / `To` | Public user identity (IMPU) being registered |
| `Contact` | UE IP address or FQDN; `+sip.instance` if GRUU/multiple-registrations; protected-server-port in hostport if SA available |
| `Via` | UE IP; `rport` parameter for NAT detection; protected-server-port in sent-by (UDP) if SA available |
| `Expires` | 600,000 seconds (maximum) |
| `Supported` | `path`; optionally `gruu`, `outbound` |
| `P-Access-Network-Info` | Access technology type and access-specific info |
| `Security-Client` | List of supported security mechanisms with IPsec parameters; `mediasec` for media plane security |
| `Authorization` (IMS AKA) | `username`=IMPI; `realm`=home domain; `uri`=SIP URI of home domain; `nonce=""`; `response=""` |

### Phase 1: On 401 Unauthorized with IMS AKA (§5.1.1.5.1)

1. Extract RAND and AUTN from `nonce` parameter of `WWW-Authenticate`
2. Verify MAC (from AUTN); check SQN is in valid range
3. Check `Security-Server` header: validate algorithm matches `Security-Client`
4. Derive CK and IK from RAND per TS 33.203
5. Set up **temporary IPsec SAs** using algorithm negotiated with P-CSCF; SIP-level SA lifetime = reg-await-auth timer
6. Send **protected REGISTER** (same `Call-ID` as the 401) with:
   - `Authorization`: `username`=IMPI, `realm`=received, `nonce`=received, `response`=RES, `algorithm`=received, `uri`=home domain
   - `Security-Client`: same IPsec params as initial request + new SA parameters for 2 new SA pairs
   - `Security-Verify`: content of `Security-Server` from the 401

**Resynchronisation (SQN out of range):** Include `auts` parameter in `Authorization` header; no `response` parameter is processed by S-CSCF.

### Phase 2: On 200 OK (§5.1.1.6.1, IMS AKA)

| Action | Detail |
|---|---|
| Promote temporary SAs | SIP-level lifetime = max(previous-SA-lifetime, registration-expiry + 30s) |
| Store default PUI | First URI in `P-Associated-URI` list |
| Store Service-Route | Values saved as preloaded `Route` header for all subsequent non-REGISTER requests |
| Store GRUUs | `pub-gruu` parameter (stable GRUU per PUI+instance); `temp-gruu` cookie (temporary GRUU) — both from `Contact` header in 200 OK |
| Store ICSI values | `+g.3gpp.icsi-ref` from `Feature-Caps` header |

---

## P-CSCF Procedures (§5.2.2)

### Phase 1: Processing the unprotected REGISTER (§5.2.2.1)

| Action | Detail |
|---|---|
| Store temporary SA | Lifetime = reg-await-auth timer; bound to UE port from `Via` header |
| Set `rport` / `received` | Copy actual source IP+port into `Via` for NAT traversal |
| Insert `Path` header | Own SIP URI (IMS flow token) + `ob` parameter for multiple registrations |
| Insert `P-Charging-Vector` | New `icid-value` (globally unique); `orig-ioi` = type 1 IOI (visited network identifier) |
| Insert `P-Visited-Network-ID` | Visited network identifier |
| Remove `mediasec` | Strip `mediasec` parameter from `Security-Client` before forwarding to S-CSCF |
| Route | Visited network (via IBCF if present) → home network entry point → I-CSCF |
| ATCF | If required by local policy, insert `Route` header with ATCF URI |

**integrity-protected value for Phase 1:**
- IMS AKA: absent (no SA, no response in Authorization) — inserted by P-CSCF when forwarding
- SIP Digest (no IP assoc): absent / `ip-assoc-pending` (challenge response present) / `ip-assoc-yes` (IP assoc exists)
- SIP Digest + TLS: `tls-pending` (TLS not yet bound to private identity) / `tls-yes` (authenticated TLS)

### Phase 1: On 401 Unauthorized (§5.2.2.2 IMS AKA)

1. Delete existing temporary SA
2. **Remove** `ck` and `ik` parameters from `WWW-Authenticate` before forwarding to UE
3. Insert `Security-Server` header: ipsec-3gpp mechanism, IKE algorithms per TS 33.203, static parameter list
4. Set up **new temporary SA** with reg-await-auth timer lifetime
5. Forward 401 **unprotected** to UE

### Phase 2: On 200 OK (§5.2.2.1, IMS AKA)

**SA state transition:**

| Scenario | Temporary SA | Previously Existing SA |
|---|---|---|
| Initial registration (200 OK) | → Newly established SA (taken into use immediately) | Deleted immediately |
| Re-registration (200 OK) | → Newly established SA; old SA used for the REGISTER that initiated re-auth continues | All other old SAs deleted immediately |

**Data stored from 200 OK:**

| Item | Detail |
|---|---|
| `Service-Route` list | Saved bound to contact address and SA; used as preloaded `Route` in subsequent originating non-REGISTER requests |
| `P-Associated-URI` | All registered public user identities; default PUI = first URI |
| `Charging-Function-Addresses` | CDF/OCF addresses for charging |
| `term-ioi` | Type 1 IOI = home network identifier |
| NAT keep-alive | If `keep` parameter in `Via` received → start keep-alive towards UE |
| `outbound` option | If `outbound` in `Require` → bidirectional flow; if absent → route per RFC 3261 |

### Subscription to registration event package (§5.2.3)

After the **first REGISTER per private user identity** (IMPI), P-CSCF sends a `SUBSCRIBE` to the S-CSCF for the `reg` event package:
- `Expires` = value higher than the registration expiry
- If roaming and `iotl` supported: append `iotl="visitedA-homeA"` URI parameter to `Request-URI`

---

## S-CSCF Procedures (§5.4.1)

### Phase 1: Unprotected REGISTER (§5.4.1.2.1)

1. **Identify user**: public identity from `To` header; private identity from `Authorization` header (if absent, derive from IMPU by removing SIP scheme, URI parameters, port, and `To` header parameters)
2. **Cx query**: MAR to HSS to obtain authentication vectors (RAND, AUTN, XRES, IK, CK); if Line-Identifiers received over Cx, compare with `P-Access-Network-Info` `dsl-location`/`eth-location`/`fiber-location` — match → authenticated; no match → 403
3. **Check `P-Visited-Network-ID`**: identify visited network
4. **IP address check**: compare `received`/`sent-by` in `Via` with stored IP or prefix; mismatch → query HSS; persistent mismatch → 403
5. **Store RAND** for future resynchronisation detection
6. **Start** `reg-await-auth` timer
7. **Send 401** (unprotected, via P-CSCF to UE)

### Phase 1: 401 WWW-Authenticate content (§5.4.1.2.1A — IMS AKA)

| Parameter | Value |
|---|---|
| `realm` | Globally unique name of the S-CSCF |
| `nonce` | RAND + AUTN + optional server-specific data |
| `algorithm` | `AKAv1-MD5` (default); `AKAv2-SHA-256` if UE requested and S-CSCF supports |
| `ik` | IK (Integrity Key) — for P-CSCF to set up SAs |
| `ck` | CK (Cipher Key) — for P-CSCF to set up SAs |

_The P-CSCF strips `ik` and `ck` before forwarding to UE — these parameters are for P-CSCF only._

For **SIP Digest** (§5.4.1.2.1B): `WWW-Authenticate` per RFC 2617 with `realm`, `nonce` (generated by S-CSCF), `algorithm` (default `MD5`), `qop` (default `auth`).

### Phase 2: Protected REGISTER with IMS AKA (§5.4.1.2.2)

**If `reg-await-auth` timer is running:**

1. Verify `Call-ID` matches the 401 challenge — only proceed if they match
2. Stop `reg-await-auth` timer
3. Check `Authorization` header contains:
   - `username` = IMPI
   - `algorithm` = `AKAv1-MD5` (or `AKAv2-SHA-256` if TLS + AKAv2)
   - `response` = RES (authentication challenge response)
4. Verify RES against expected XRES (calculated using RFC 3310 for AKAv1, RFC 4169 for AKAv2)
5. Only proceed if RES matches XRES

**If no authentication ongoing (re-registration with `integrity-protected`=`yes`):**
- S-CSCF may require re-authentication for any REGISTER request
- If re-auth needed: proceed as for unprotected REGISTER (from step 3 of §5.4.1.2.1)

**After successful verification — registration binding update:**

| Step | Action |
|---|---|
| 4A | If no `reg-id` and same IMPI + old contact already registered (unexpired): terminate old dialogs (480), NOTIFY subscribers of old PUI deregistration, delete old bindings |
| 4B | If `reg-id` present + "outbound" in Supported/Path: if first time with this instance-id → create new registration flow; if same instance-id previously registered → refresh or replace flow |
| 5 | SAR to HSS (Server-Assignment-Request, REGISTRATION type) |
| 5a | Store list of PUIs (registered + implicit-registered + wildcarded; each = barred or non-barred) |
| 5b | Store all service profiles + iFCs (initial Filter Criteria for registered + common parts; unregistered part retained for when user deregisters) |
| 5c | If S-CSCF restoration supported: store restoration info from HSS |
| 5d | If PCRF-based P-CSCF restoration supported: store user profiles including IMSI |
| 6 | Update registration bindings: bind all contact info (header params + URI params, except pub-gruu/temp-gruu) to each non-barred PUI |
| 7 | Build preloaded Route from `Path` header: preserve order; bind to contact address of UE or to registration flow+contact (multiple registration); new list replaces old list |
| 8 | Bind registration expiry to contact (or registration flow); may reduce to minimum allowed; may return 423 if UE's expiry too short |
| 9 | Store `icid-value` from `P-Charging-Vector` |
| 9A | Store `orig-ioi` (type 1 IOI — identifies visited network) from `P-Charging-Vector` |
| 10 | Send 200 OK per §5.4.1.2.2F |

After 200 OK: S-CSCF sends **third-party REGISTER** to each AS that matches the iFCs for the REGISTER event.

### Phase 2: Protected REGISTER with SIP Digest (§5.4.1.2.2A)

- `integrity-protected` header field = `tls-pending` / `tls-yes` / `ip-assoc-pending` / `ip-assoc-yes`
- S-CSCF verifies response = H(A1) per RFC 2617; `stale` = `true` if nonce is valid but expired
- On success + `tls-pending`/`tls-yes` → associate registration with local state `"tls-protected"`
- 200 OK includes `Authentication-Info` with `nextNonce`, `qop`, `rspauth`, `cnonce`, `nonce-count`

### 200 OK Construction (§5.4.1.2.2F)

The S-CSCF MUST include in the 200 OK:

| Header | Content |
|---|---|
| Path headers | Received Path header fields (list of P-CSCF + IBCF URIs) |
| `P-Associated-URI` | Distinct registered PUIs of this IMPI — first URI = default PUI; no barred PUIs; implicitly registered PUIs included; wildcarded PUIs not listed |
| `Service-Route` | **A** — S-CSCF SIP URI (unique per registration or per flow if multiple registrations), indicating subsequent requests via this route were sent by the UE or the registration flow; **B** — IBCF URI as topmost entry if network topology hiding required; **C** — `iotl=visitedA-homeA` SIP URI parameter appended if: S-CSCF supports RFC 7549, UE is roaming, P-CSCF not in home network, required by local policy |
| `P-Charging-Function-Addresses` | CDF/OCF addresses — only if P-CSCF is in same network as S-CSCF (determined from `P-Visited-Network-ID`) |
| `P-Charging-Vector` | `orig-ioi` = type 1 IOI (identifies sending/home network); `term-ioi` = previously received `orig-ioi` (type 1, identifies visited network); `icid-value` = previously received icid-value |
| `Contact` | All contact addresses for this PUI — all saved header params + URI params (except `pub-gruu` and `temp-gruu`) |
| `pub-gruu` | Per Contact with `+sip.instance`: public GRUU for this (PUI, instance-id) pair |
| `temp-gruu` | Per Contact without `bnc` URI parameter: newly assigned temporary GRUU |
| `temp-gruu-cookie` | If S-CSCF supports RFC 6140 and Contact has `bnc`: temp-gruu-cookie per RFC 6140 |
| `Feature-Caps` | `+g.3gpp.icsi-ref` ICSI values from service profile (those not needing explicit P-CSCF support) |
| `Require: outbound` | If received REGISTER had both `reg-id` and `+sip.instance`, and first Path URI has `ob` parameter |
| `Feature-Caps: +g.3gpp.verstat` | If home network supports calling number verification with signature/attestation |
| `Feature-Caps: +sip.607` | If home network supports 607 Unwanted response code |

---

## User-Initiated Deregistration (§5.4.1.4)

Triggered by UE sending REGISTER with `Expires: 0`.

**S-CSCF processing:**

1. Verify that the REGISTER corresponds to an existing registered contact or flow; if not → 481
2. For IMS AKA: require `integrity-protected` = `yes` in Authorization
3. For SIP Digest without TLS or with TLS: require `integrity-protected` = `tls-yes` or `ip-assoc-yes`
4. Release any INVITE dialogs involving this user's contact addresses or flows
5. Process `Contact` header:
   - `*` and no `reg-id`: remove binding between PUI (To header) and all contact addresses registered by this UE
   - `*` and `reg-id`/`+sip.instance` + multiple registration: remove binding between PUI and the specific registration flow
   - Specific Contact: remove that specific binding
6. Third-party REGISTER to each AS matching iFCs for REGISTER event
7. 200 OK: list all contacts (current + deregistered with `expires=0`)
8. If all PUIs deregistered: P-CSCF/UE SUBSCRIBE cancelled; S-CSCF may request HSS to release or keep S-CSCF assignment

---

## Network-Initiated Deregistration (§5.4.1.5)

Initiated by HSS (RTR) or internal S-CSCF event. S-CSCF can deregister:
- All contact addresses for a PUI
- Some contact addresses
- A specific contact address
- One or more registration flows (if multiple registration in use)

**Method**: S-CSCF sends a `NOTIFY` request to the UE/P-CSCF subscribers of the reg event package, setting `Subscription-State: terminated` in the dialog.

If active multimedia sessions still exist for the registered contact at deregistration time, S-CSCF releases some or all of those sessions (per §5.4.5.1.2) before deregistering.

---

## State Data Stored Per Node

### UE
- Default PUI (first URI in `P-Associated-URI`)
- Service-Route (preloaded `Route` for outgoing non-REGISTER requests)
- `pub-gruu` (per PUI+instance) and `temp-gruu` (per registration)
- Feature-Caps ICSI values (`+g.3gpp.icsi-ref`)
- SA parameters: ports, algorithms, SPI values, lifetime

### P-CSCF
- Registration bindings: contact address → SA pair → (default PUI + P-Associated-URI list + Service-Route list)
- Charging-Function-Addresses (CDF/OCF)
- `term-ioi` (type 1, home network identifier)
- SA state: temporary → newly established (→ old, for re-registration)
- NAT keep-alive configuration (if `keep` in Via)

### S-CSCF
- PUI list: registered + implicitly registered + wildcarded, each barred/non-barred
- Service profiles: all iFCs (for registered and unregistered parts), ICSI/IARI values
- Preloaded Route list: bound to contact address or registration flow
- Registration expiry bound to contact/flow
- `icid-value` + `orig-ioi` (type 1) from P-Charging-Vector
- IP address of UE (from Via `received`/`sent-by`)
- Restoration info (if S-CSCF restoration supported) and IMSI (if PCRF-based P-CSCF restoration)
- RAND (from last 401, for resync detection)

---

## Abnormal Cases

### IMS AKA (§5.4.1.2.3A, §5.4.1.4.2)

| Condition | S-CSCF Action |
|---|---|
| RES does not match XRES | 403 Forbidden; registration state unchanged |
| Wrong `Call-ID` in protected REGISTER | Ignore (Call-IDs must match) |
| `auts` in Authorization (SQN out of range) | Fetch new auth vectors from HSS; either send new 401 or 403 (abandon) |
| UE resends within same auth session | Only 2 consecutive invalid challenges before S-CSCF sends new 401 with fresh challenge |
| `integrity-protected`=`yes` but no matching user at S-CSCF | 500 Internal Server Error (if S-CSCF restoration not supported) |

### SIP Digest (§5.4.1.2.3B, §5.4.1.4.4)

| Condition | S-CSCF Action |
|---|---|
| Auth challenge response mismatch | 403 Forbidden or rechallenging 401 |
| Invalid nonce | 401 with `stale=true` and fresh nonce |
| nonce-count mismatch | 401 with `stale=true` |
| No matching user at S-CSCF | 500 (if S-CSCF restoration not supported) |

### reg-await-auth timer expiry
- S-CSCF treats authentication as failed
- If UE was previously registered: stays registered until expiry (TS 33.203 §19 behavior)
- New initial REGISTER before expiry: stop timer, restart authentication with fresh 401

---

## Registration Flow Management (Multiple Registrations, RFC 5626)

When the multiple registrations mechanism is active (UE includes `reg-id` + `+sip.instance`):
- Each `reg-id` value identifies a separate registration flow
- P-CSCF inserts `ob` parameter in the `Path` header
- S-CSCF creates a new registration flow or refreshes/replaces an existing one based on matching `+sip.instance` and `reg-id`
- S-CSCF Service-Route is bound to the registration flow + associated contact address
- A 439 (First Hop Lacks Outbound Support) is returned if `ob` is absent from the first Path URI when `reg-id` is in the Contact

---

## Cross-References

| Topic | Page |
|---|---|
| IPsec SA setup detail (4-port model, SPI negotiation) | [IMS-AKA-registration.md](IMS-AKA-registration.md) |
| High-level IMS registration call flow (TS 23.228) | [IMS-registration.md](IMS-registration.md) |
| Cx MAR/SAR message content | [Cx-signalling-flows.md](Cx-signalling-flows.md) |
| S-CSCF iFC evaluation at registration | [IM-call-model.md](../concepts/IM-call-model.md) |
| SIP trust domain and P-header removal rules | [SIP-IMS-profile.md](../protocols/SIP-IMS-profile.md) |
| Security mechanisms and alternatives | [IMS-access-security.md](../concepts/IMS-access-security.md), [IMS-auth-alternatives.md](../concepts/IMS-auth-alternatives.md) |
| GRUU and identity model | [IMS-identity-model.md](../concepts/IMS-identity-model.md) |
| P-CSCF entity deep-dive | [P-CSCF-deepdive.md](../entities/P-CSCF-deepdive.md) |
| S-CSCF entity deep-dive | [S-CSCF-deepdive.md](../entities/S-CSCF-deepdive.md) |
