---
title: "HSS — Home Subscriber Server"
type: entity
tags: [HSS, EPC, IMS, subscriber-data, authentication, Diameter, S6a, Cx, Sh, IMPI, IMPU, iFC]
sources: [ts_123401v150400p.pdf, ts_123228v150600p.pdf]
updated: 2026-04-08
---

# HSS — Home Subscriber Server

**Spec reference:** 3GPP TS 23.401 §4.2 (S6a), §4.3.5–4.3.6 (subscription data usage); IMS detail in TS 23.228
**Plane:** Control plane (Diameter only)
**Layer:** EPC + IMS — master subscriber database

## Role

The HSS is the central subscriber database for both EPC and IMS. It stores authentication credentials, subscription profiles, and service configuration for each subscriber. It is the authoritative source: the MME fetches subscriber data during Attach; the S-CSCF fetches IMS service profiles during IMS Registration.

> Full HSS specification is in **3GPP TS 29.272** (S6a/S6d) and **3GPP TS 29.228/29.229** (Cx interface for IMS).

## Data Stored (EPC context)

| Data | Description |
|---|---|
| IMSI | International Mobile Subscriber Identity — primary key |
| Authentication vectors | RAND, XRES, AUTN, KASME (EPS AKA); pre-generated or on-demand |
| Subscription profile | Allowed APNs, default APN, QoS profile per APN (QCI, ARP, APN-AMBR, UE-AMBR) |
| RFSP Index | RAT/Frequency Selection Priority — passed to MME for eNB RRM |
| Roaming permissions | Visited PLMN allowed/restricted lists |
| Access restrictions | Barring of specific RATs, areas |
| IMS voice over PS indicator | Whether UE is configured for IMS voice; passed to MME |
| Registered MME identity | Which MME currently serves the subscriber; updated on each Attach/TAU |
| Homogenous IMS voice support | Indication for T-ADS routing decisions |

## Data Stored (IMS context — from TS 23.228)

| Data | Description |
|---|---|
| **IMPI** | IMS Private User Identity (NAI format); used for IMS AKA authentication |
| **IMPU(s)** | One or more IMS Public User Identities (SIP URI or tel URI) per subscriber |
| **Implicit Registration Set** | Group of IMPUs registered together; one service profile for the set |
| **Service Profile** | Per-IMPU or per-set: ordered list of iFCs (Initial Filter Criteria) with SPTs and AS URIs |
| **Registered S-CSCF** | FQDN/address of the S-CSCF currently serving this subscriber; set on SAR, cleared on de-reg |
| **S-CSCF capabilities required** | Mandatory and optional capability sets for S-CSCF selection by I-CSCF |
| **IMS auth data** | IMS AKA vectors (RAND, AUTN, XRES, CK, IK) or SIP Digest credentials |
| **Wildcarded IMPU** | Range-based IMPU pattern; one service profile for bulk number range |
| **Charging info** | Default charging function addresses for IMS CDRs |

### Cx Interface — IMS Key Procedures

**User Authorization Request (UAR) / Answer (UAA):**
- I-CSCF → HSS when REGISTER arrives
- HSS returns: existing S-CSCF name (if already registered), or required capabilities for S-CSCF selection
- Also checks if the visited network is authorized

**Server Assignment Request (SAR) / Answer (SAA):**
- S-CSCF → HSS after authentication success (or de-registration)
- Assignment type: REGISTRATION, RE_REGISTRATION, UNREGISTRATION, etc.
- HSS stores S-CSCF name; returns full service profile (all iFCs for subscriber's IMPUs)

**Multimedia Auth Request (MAR) / Answer (MAA):**
- S-CSCF → HSS during IMS AKA
- HSS generates and returns IMS auth vectors: RAND, AUTN, XRES, CK, IK

**Location Info Request (LIR) / Answer (LIA):**
- I-CSCF → HSS for terminating sessions
- Returns the registered S-CSCF name serving the terminating IMPU

**Registration Termination Request (RTR) / Answer (RTA):**
- HSS → S-CSCF to initiate network-side de-registration
- Reason: subscription change, administrative, roaming constraints

**Push Profile Request (PPR) / Answer (PPA):**
- HSS → S-CSCF to push updated service profile without re-registration
- Triggered when operator changes iFC configuration in HSS

### Sh Interface — AS Access to Subscriber Data

The Sh interface allows Application Servers (TAS, voicemail AS, etc.) to:
- **Read** subscriber service data: call forwarding targets, barring flags, MSISDN
- **Write** subscriber service data: updated by XCAP (UE self-service via Ut)
- **Subscribe** to change notifications: HSS pushes Sh SNR when data changes
- AS identifies itself to HSS; HSS enforces what data the AS is permitted to access

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| S6a | MME | Diameter | Authentication, subscription download, location update/cancel |
| S6d | SGSN | Diameter | Same as S6a but toward SGSN |
| Cx | S-CSCF / I-CSCF | Diameter | IMS registration (UAR, SAR, MAR, LIR, RTR, PPR) |
| Sh | AS (TAS, voicemail AS, etc.) | Diameter | AS read/write subscriber service data; change notifications |
| MAP/Gr | legacy SGSN | SS7/MAP | Legacy 2G/3G (pre-Diameter) |

## S6a Key Procedures (EPC)

**Update Location Request (ULR) / Answer (ULA):**
- MME → HSS on Attach or inter-MME TAU
- HSS updates registered MME; returns subscription data

**Authentication Information Request (AIR) / Answer (AIA):**
- MME → HSS to fetch authentication vectors (EPS-AV) for AKA
- HSS generates and returns requested number of EPS-AVs

**Cancel Location Request (CLR) / Answer (CLA):**
- HSS → old MME when UE attaches to new MME
- Old MME detaches UE and deletes UE context

**Purge UE Request (PUR) / Answer (PUA):**
- MME → HSS when UE is implicitly detached (unreachable)

**Notify Request (NOR) / Answer (NOA):**
- MME → HSS to report IMS voice over PS session support status, RAT type

## Notes
- HSS is not in the user plane at all
- In roaming, the HSS is always in the HPLMN; the MME in VPLMN queries it over S6a
- HSS is shared between EPC and IMS — same node stores both EPS subscriber data and IMS service profiles

## Related Pages
- [MME](MME.md) — primary EPC consumer via S6a
- [PCRF](PCRF.md) — may query SPR (which may be co-located with HSS) via Sp
- [S-CSCF](S-CSCF.md) — downloads service profile via Cx; authenticates UE
- [I-CSCF](I-CSCF.md) — queries HSS for S-CSCF assignment via Cx
- [TAS](TAS.md) — reads/writes subscriber data via Sh
- [IMS Identity Model](../concepts/IMS-identity-model.md)
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
- [EPC Reference Points](../interfaces/reference-points.md)
