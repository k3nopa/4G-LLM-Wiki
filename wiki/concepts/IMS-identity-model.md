---
title: "IMS Identity Model"
type: concept
tags: [IMS, identity, IMPI, IMPU, GRUU, P-GRUU, T-GRUU, MSISDN, tel-URI, SIP-URI, implicit-registration, service-profile, iFC]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# IMS Identity Model

**Spec reference:** 3GPP TS 23.228 §4.3–§4.5

## Overview

IMS uses a layered identity model that separates **authentication identity** from **signaling identity** from **routing identity**. This is fundamentally different from CS networks where the MSISDN serves all three purposes.

---

## Core Identity Types

### IMPI — IMS Private User Identity

| Property | Value |
|---|---|
| Format | NAI (Network Access Identifier): `user@domain`, e.g. `+441234567890@ims.operator.com` |
| Used for | Authentication only — appears in Authorization header during registration |
| Routing | **Never** used in SIP Request-URI or public signaling |
| Storage | HSS; associated with one or more IMPUs |
| Scope | One per subscriber (one per UICC/ISIM) |

The IMPI is the equivalent of a username for IMS AKA authentication. It identifies the subscriber to the network for security purposes.

### IMPU — IMS Public User Identity

| Property | Value |
|---|---|
| Format | SIP URI (`sip:+441234567890@ims.operator.com`) or tel URI (`tel:+441234567890`) |
| Used for | Calling/called party ID in SIP signaling; contact in REGISTER |
| Visibility | Public — appears in From, To, P-Asserted-Identity headers |
| Scope | One IMPI can have **multiple IMPUs** (e.g. separate voice, video, business identities) |
| Registration | Must be registered to receive sessions |

#### Wildcarded IMPU
- Represents a **range of IMPUs** using a regular expression pattern
- Used for carrier-scale deployments: register an entire number range without per-UE signaling
- HSS stores one service profile for the entire wildcard range
- The S-CSCF handles routing to specific IMPUs matching the pattern

### GRUU — Globally Routable UA URI

A GRUU identifies a **specific UE instance** rather than just a subscriber. It combines the IMPU with a `gr` parameter:

```
sip:+441234567890@ims.operator.com;gr=urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6
```

| Type | Description | Lifetime |
|---|---|---|
| **P-GRUU** (Public) | Durable; tied to an IMPU registration at a specific UE; survives re-registration | Valid while IMPU is registered at that UE |
| **T-GRUU** (Temporary) | Changes with each re-registration; provides unlinkability; based on `+sip.instance` | Valid for one registration interval |

**Use case:** When a subscriber has multiple registered UEs (e.g. phone + tablet + softphone), a GRUU allows routing to a specific device. Without GRUU, the S-CSCF may fork to all registered contacts.

---

## Implicit Registration Set

An **Implicit Registration Set** is a group of IMPUs that are always registered together:
- Registering **any one IMPU** in the set automatically registers **all** IMPUs in the set
- De-registering one de-registers all
- The set has exactly **one service profile** in the HSS
- Allows a subscriber to have both tel URI and SIP URI automatically active with one REGISTER

```
IMPU Set: {sip:+441234567890@ims.operator.com, tel:+441234567890}
→ Register either one → both are active → both can receive calls
```

---

## Service Profile Structure

Each IMPU (or Implicit Registration Set) has a **service profile** in the HSS:

```
Service Profile
└── iFCs (Initial Filter Criteria) [ordered by priority]
    ├── iFC #1
    │   ├── Priority: 1
    │   ├── Service Point Triggers (SPTs)
    │   │   ├── Method: INVITE
    │   │   └── Direction: Originating
    │   ├── Application Server URI: sip:mmtel.ims.operator.com
    │   └── Default Handling: CONTINUE
    ├── iFC #2
    │   ├── Priority: 2
    │   ├── SPTs: Method: REGISTER
    │   ├── AS URI: sip:presence.ims.operator.com
    │   └── Default Handling: CONTINUE
    └── ...
```

Service profiles are **downloaded to S-CSCF** during registration (Cx SAR/SAA). They are stored in S-CSCF memory for the duration of the registration.

---

## Identity in SIP Headers

| Header | Identity Used | Direction |
|---|---|---|
| `From` | IMPU (originating) | UE → network |
| `To` | IMPU (terminating) | UE → network |
| `P-Asserted-Identity` | IMPU (network-verified) | Added by S-CSCF after authentication |
| `P-Preferred-Identity` | IMPU (UE preference) | UE → P-CSCF (hints which IMPU to assert) |
| `Authorization` | IMPI | Registration auth |
| `Contact` | GRUU or raw contact URI | Registration binding |

---

## MSISDN Relationship

| Identity | Layer | Example |
|---|---|---|
| IMSI | EPC auth | 234150123456789 |
| MSISDN | E.164 number | +441234567890 |
| IMPI | IMS auth | +441234567890@ims.operator.com |
| IMPU | IMS signaling | sip:+441234567890@ims.operator.com |

The MSISDN and IMPU often encode the same phone number in different formats. HSS links IMSI, IMPI, and IMPU(s) together.

---

## ENUM/DNS

Tel URIs (`tel:+441234567890`) are translated to SIP URIs via **ENUM** (Telephone NUMber mapping):
1. Phone number reversed and appended to `e164.arpa` domain
2. DNS NAPTR query returns SIP URI
3. S-CSCF uses this during session routing to translate tel URIs to routable SIP URIs

---

## Related Pages
- [S-CSCF](../entities/S-CSCF.md) — downloads/evaluates service profiles; applies GRUUs
- [HSS](../entities/HSS.md) — stores IMPI, IMPU, service profile
- [TAS](../entities/TAS.md) — reads service data via Sh; acts on iFC triggers
- [P-CSCF](../entities/P-CSCF.md) — handles P-Asserted-Identity, P-Preferred-Identity
