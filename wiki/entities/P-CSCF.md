---
title: "P-CSCF (Proxy-CSCF)"
type: entity
tags: [IMS, P-CSCF, SIP, proxy, security, Rx, policy, VoLTE, AF]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# P-CSCF — Proxy Call Session Control Function

**Spec reference:** 3GPP TS 23.228 §4.6

## Role

The P-CSCF is the **UE's first and only contact point into the IMS**. Every SIP message from the UE passes through the P-CSCF; every SIP message toward the UE passes back through it. It acts simultaneously as:
- A SIP outbound proxy for the UE
- An IMS security gateway (IPsec SA with UE)
- The **Application Function (AF)** that drives EPC bearer policy via Rx

The P-CSCF is located in the **VPLMN** when roaming (both home-routed and local-breakout).

---

## Functions

| Function | Description |
|---|---|
| **SIP proxy** | Forwards REGISTER and session requests toward I-CSCF or S-CSCF |
| **IPsec security** | Establishes IPsec SAs with UE during registration (IMS AKA); protects all SIP messages |
| **SIP compression** | Optionally compresses SIP (SigComp) to reduce over-the-air signaling overhead |
| **P-header insertion** | Adds `P-Visited-Network-ID`, `P-Access-Network-Info`, `P-Associated-URI` headers |
| **AF on Rx** | Extracts media parameters from SDP; sends AAR to PCRF to authorize/establish GBR bearers |
| **PDG for S-CSCF** | Forwards responses back toward UE; strips/adds Route headers |
| **SDP inspection** | Parses SDP in INVITE to extract codec, bandwidth, IP/port for Rx AAR |
| **Emergency call handling** | Routes emergency INVITEs to E-CSCF; provides location info |
| **SIP timer management** | Handles SIP re-registration timers; performs de-registration if UE disappears |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **Gm** | UE | SIP (UDP/TCP/TLS) | UE → IMS registration and sessions |
| **Mw** | I-CSCF, S-CSCF | SIP | Forward SIP within IMS |
| **Rx** | PCRF | Diameter | Send AF session info to authorize QoS bearers |

---

## P-CSCF Discovery

The UE discovers its P-CSCF address via one of:
1. **DHCP** — P-CSCF FQDN/address in DHCP option on IMS APN
2. **DNS** — NAPTR/SRV lookup after DHCP gives FQDN
3. **PCO** (Protocol Configuration Options) in EPS Attach / PDN Connectivity — PGW provides P-CSCF address directly

The P-CSCF address is typically provisioned per-PLMN or derived from the UE's PGW in the VPLMN.

---

## Role in VoLTE Bearer Establishment

```
UE          P-CSCF          PCRF          PGW
 │──INVITE──►│                │             │
 │           │──Rx AAR───────►│             │
 │           │                │──Gx RAR────►│
 │           │                │◄──Gx RAA────│
 │           │◄──Rx AAA───────│             │
 │           │                │             │
 │           │──INVITE──►(S-CSCF)           │
```

1. P-CSCF receives INVITE with SDP (codec=AMR-WB, 24kbps)
2. Sends **Rx AAR** to PCRF: media-component-description, flow description, AF-application-id
3. PCRF grants: returns AAA; pushes PCC rule via Gx RAR to PGW
4. PGW installs GBR bearer (QCI=1, GBR=24kbps, ARP=1)
5. P-CSCF forwards INVITE to S-CSCF

On session teardown (BYE), P-CSCF sends **Rx STR** → PCRF removes PCC rule → PGW deletes GBR bearer.

---

## Security

- P-CSCF is the **IMS security boundary** toward the UE
- During IMS AKA:
  1. UE and P-CSCF negotiate an IPsec SA (ESP, AES or 3DES)
  2. SA protects SIP signaling on Gm interface
  3. SA uses port-based encapsulation (UE client port + server port pairs)
- P-CSCF does NOT decrypt IPsec for routing — it terminates the SA
- For THIG (Topology Hiding Inter-network Gateway): I-CSCF may strip topology headers before leaving the HPLMN; P-CSCF in VPLMN only sees what I-CSCF exposes

---

## Related Pages
- [I-CSCF](I-CSCF.md) — receives forwarded REGISTER from P-CSCF
- [S-CSCF](S-CSCF.md) — receives forwarded INVITE from P-CSCF
- [PCRF](PCRF.md) — policy peer on Rx
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
- [IMS Identity Model](../concepts/IMS-identity-model.md)
