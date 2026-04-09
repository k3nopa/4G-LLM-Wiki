---
title: "PCRF — Policy and Charging Rules Function"
type: entity
tags: [PCRF, EPC, policy, charging, Gx, Rx, Diameter, QoS, IMS]
sources: [ts_123401v150400p.pdf]
updated: 2026-04-08
---

# PCRF — Policy and Charging Rules Function

**Spec reference:** 3GPP TS 23.401 §4.4.7; detailed in TS 23.203
**Plane:** Control plane (Diameter signaling only)
**Layer:** EPC core — policy brain

## Role

The PCRF is the policy decision point of the EPC. It decides QoS and charging rules for each UE's IP-CAN session and pushes those rules to the PGW (via Gx). It also receives session information from Application Functions (e.g. IMS P-CSCF via Rx) and translates media description into bearer-level QoS decisions — this is the key mechanism for VoLTE dedicated bearer establishment.

> PCRF functions are fully specified in **3GPP TS 23.203**.

## Variants

### Non-Roaming
Single **H-PCRF** per UE IP-CAN session:
- Terminates **Gx** (to PGW/PCEF)
- Terminates **Rx** (to AF — typically P-CSCF for IMS)

### Roaming — Home Routed
Same as non-roaming (PGW in HPLMN, only H-PCRF).

### Roaming — Local Breakout
Two PCRFs per IP-CAN session:
- **V-PCRF** (in VPLMN): terminates Gx (to V-PGW) + S9 + Rx (for visited AF)
- **H-PCRF** (in HPLMN): terminates Rx (for home services) + S9 (to V-PCRF)
- S9 carries policy/charging control information between H-PCRF and V-PCRF

## Interfaces

| Interface | Peer | Protocol | Direction | Purpose |
|---|---|---|---|---|
| Gx | PGW (PCEF) | Diameter | Bidirectional | Push PCC rules; receive events/usage from PGW |
| Rx | AF (P-CSCF) | Diameter | Bidirectional | Receive session info from IMS; authorize QoS |
| S9 | V-PCRF ↔ H-PCRF | Diameter | Bidirectional | Roaming policy coordination |
| Sp | SPR | — | — | Subscriber Profile Repository (subscription data) |
| Np | RCAF | — | Receive | RAN User Plane Congestion Information |

## Key Workflow: VoLTE Dedicated Bearer

1. UE registers with IMS → P-CSCF established as proxy
2. UE sends SIP INVITE → P-CSCF sends **AAR** (AA-Request) on **Rx** to PCRF with SDP media info
3. PCRF correlates Rx session with active Gx session (matching UE IP)
4. PCRF creates PCC rule: QCI=1 (GBR conversational voice), GBR = codec bandwidth
5. PCRF sends **RAR** (Re-Auth-Request) on **Gx** to PGW with new PCC rule
6. PGW initiates dedicated bearer establishment toward MME → eNodeB → UE
7. VoLTE RTP media flows on the new GBR bearer

## PCC Rules
A PCC rule contains:
- Service Data Flow (SDF) filter (5-tuple or application ID)
- QoS parameters: QCI, ARP, GBR UL/DL, MBR UL/DL
- Charging parameters: rating group, reporting level, metering method
- Gating status (open/closed)
- Precedence

## Related Pages
- [PGW](PGW.md) — enforcement point via Gx
- [P-CSCF](P-CSCF.md) — IMS application function via Rx
- [Reference points](../interfaces/reference-points.md)
