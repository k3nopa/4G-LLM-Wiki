---
title: "MME — Mobility Management Entity"
type: entity
tags: [MME, EPC, control-plane, NAS, mobility, authentication, bearer]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# MME — Mobility Management Entity

**Spec reference:** 3GPP TS 23.401 §4.4.2, §4.3.5, §4.3.7
**Plane:** Control plane only (no user plane traffic)
**Layer:** EPC core

## Role

The MME is the primary control plane node of the EPC. It is the UE's anchor point for NAS signaling, authentication, session management signaling, and mobility management. Every LTE UE has exactly one serving MME at any point in time.

## Functions (from TS 23.401 §4.4.2)

| Function | Description |
|---|---|
| NAS signaling | Termination point for all NAS messages from UE (via eNodeB relay) |
| NAS security | NAS integrity + ciphering; selects NAS security algorithms |
| Inter-CN signaling | S3 toward SGSN for 2G/3G↔LTE inter-RAT mobility |
| UE reachability (idle) | Paging coordination; knows UE at TAI list granularity in ECM-IDLE |
| TAI list management | Allocates Tracking Area Identity lists to UEs (avoids frequent TAU) |
| TAI → timezone | Maps UE location (TAI) to timezone; signals timezone change to PGW |
| PGW selection | Selects PGW for UE's PDN connections (DNS-based, APN-based) |
| SGW selection | Selects Serving GW for UE |
| MME selection | Selects target MME during handover with MME change |
| SGSN selection | Selects SGSN for handover to 2G/3G networks |
| Authentication | Runs AKA procedure; fetches authentication vectors from HSS via S6a |
| Authorization | Verifies UE is authorized to access requested services/APNs |
| Bearer management | Dedicated bearer establishment, modification, deletion signaling |
| Lawful Interception | LI of signaling traffic |
| Warning messages | Selects appropriate eNodeBs for distributing PWS messages |
| RFSP Index | Provides RAT/Frequency Selection Priority index to eNodeB (from HSS subscription) |
| CIoT CP optimization | For NB-IoT/LTE-M: transports user data in NAS (no S1-U); header compression (ROHC); acts as local mobility anchor |
| Presence Reporting | Reports UE presence in Presence Reporting Areas on PCC request |
| Overload control | Sends OVERLOAD START/STOP to eNodeBs via S1-AP; rejects NAS requests with back-off timers |
| Load balancing | Weight Factor advertised to eNodeBs for proportional MME selection |
| Load re-balancing | Moves UEs to other MMEs in pool via TAU with release cause |

## Interfaces

| Interface | Peer | Protocol | Direction/Purpose |
|---|---|---|---|
| S1-MME | eNodeB | S1-AP | NAS relay, E-RAB setup/modify/release, paging, handover |
| S6a | HSS | Diameter | Update-Location, authentication vectors, subscription data |
| S11 | SGW | GTPv2-C | Create/Modify/Delete Session, Create/Delete Bearer |
| S10 | MME (peer) | GTPv2 | UE context transfer during inter-MME handover or relocation |
| S3 | SGSN | GTP | Inter-RAT context transfer (2G/3G ↔ LTE) |
| S13 | EIR | — | UE identity check (IMEI verification) |
| S7a | CSS | — | CSG subscription data for roaming UEs |
| Nq | RCAF | — | Receives RAN congestion info |
| SGs | MSC/VLR | — | CS Fallback + SMS over SGs (see TS 23.272) |

## State Machines Owned

### EMM State (per UE context in MME)
```
EMM-DEREGISTERED ──Attach Accept──────────────► EMM-REGISTERED
                  ◄─Detach / All bearers released──────────────
                  ◄─Attach Reject / TAU Reject──────────────────
```
- **EMM-DEREGISTERED**: no valid location/routing; UE not reachable; partial UE context may exist
- **EMM-REGISTERED**: location known at TAI granularity; at least 1 active PDN connection

### ECM State (per UE context in MME)
```
ECM-IDLE ──Attach/TAU/Service/Detach Request──► ECM-CONNECTED
         ◄──S1 Release Procedure─────────────────────────────
```
- **ECM-IDLE**: no S1-MME/S1-U; MME pages UE; UE context exists in MME but not eNodeB
- **ECM-CONNECTED**: S1-MME established; UE location known to eNB granularity

## Timers (key ones)
| Timer | Purpose |
|---|---|
| Mobile Reachability Timer | Started when UE enters ECM-IDLE; ~= UE's periodic TAU timer; expiry → suspect UE unreachable |
| Implicit Detach Timer | Started when Mobile Reachability Timer expires; expiry → implicit detach |
| Active Timer | If MME allocated Active Time (PSM); tracks PSM active window |

## Overload Behavior (§4.3.7.4)
When overloaded, MME can:
- Send `OVERLOAD START` to eNodeBs via S1-AP (with Traffic Load Reduction Indication %)
- eNodeBs reject RRC connection requests for non-emergency/non-MPS services
- MME rejects NAS requests with MM back-off timer
- MME rejects ESM requests with SM back-off timer
- Recovery: `OVERLOAD STOP` messages to eNodeBs

## Load Balancing (§4.3.7.2)
- MME Pool Area: group of MMEs serving same set of eNodeBs
- Weight Factor: broadcast to eNodeBs via S1-AP; eNB selects MME proportionally
- Load re-balancing: MME initiates S1 Release with cause "load balancing TAU required" → UE performs TAU → new MME selected

## Key Context Stored per UE
- IMSI, GUTI, S-TMSI
- EMM state, ECM state
- TAI list allocated to UE
- Security context (NAS keys, algorithm, NAS COUNT)
- EPS bearer contexts (EBI, QoS params)
- SGW address + TEID
- Subscribed RFSP Index + RFSP Index in use
- IMS voice over PS Sessions indicator (from HSS)
- UE capability (dual connectivity, NR, CIoT, etc.)

## Related Pages
- [SGW](SGW.md) — user plane peer via S11
- [HSS](HSS.md) — subscription/auth data via S6a
- [PCRF](PCRF.md) — no direct interface; PCRF interacts with PGW/AF
- [EPS bearer model](../concepts/EPS-bearer.md)
- [EMM/ECM states](../concepts/EMM-ECM-states.md)
- [Reference points](../interfaces/reference-points.md)
