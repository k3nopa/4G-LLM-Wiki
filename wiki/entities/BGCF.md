---
title: "BGCF (Breakout Gateway Control Function)"
type: entity
tags: [IMS, BGCF, PSTN, SIP, routing, MGCF, Mi, Mj, Mk]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# BGCF — Breakout Gateway Control Function

**Spec reference:** 3GPP TS 23.228 §4.11

## Role

The BGCF is a **SIP server that selects the point of PSTN/CS network breakout** for IMS-originated sessions destined for the PSTN or CS domain. It receives sessions from S-CSCF (Mi) and decides whether to break out locally (via MGCF, interface Mj) or route to another network's BGCF (interface Mk).

---

## Functions

| Function | Description |
|---|---|
| **Breakout point selection** | Determines whether PSTN breakout is local or in another network |
| **Route to MGCF** (local) | Routes session to MGCF via Mj; MGCF converts SIP → ISUP |
| **Route to peer BGCF** (remote) | Routes session to BGCF in another PLMN or network via Mk |
| **Route to IMS-ALG** | Routes toward external IP networks via Mx (IBT/IMS-ALG) |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **Mi** | S-CSCF | SIP | Receive sessions destined for PSTN |
| **Mj** | MGCF | SIP | Local breakout: pass session to MGCF |
| **Mk** | BGCF (peer network) | SIP | Remote breakout: route to another BGCF |
| **Mx** | IMS-ALG / IBT | SIP | Route toward external IP multimedia networks |

---

## Call Routing Decision

```
S-CSCF ──Mi──► BGCF
                │
                ├── Local breakout? ──Mj──► MGCF ──► PSTN
                │
                └── Remote network? ──Mk──► BGCF(peer) ──► MGCF ──► PSTN
```

The BGCF uses the dialed number (Request-URI tel URI or SIP URI) to determine routing:
- If the number belongs to a local PSTN trunk → use MGCF locally
- If the number belongs to another carrier's network → forward to that network's BGCF

---

## Relationship with MGCF

The **MGCF** (Media Gateway Control Function) is the actual SIP↔ISUP signaling converter and controls the **MGW** (Media Gateway) for RTP↔TDM transcoding. The BGCF selects which MGCF to use; the MGCF then takes over the session.

---

## Related Pages
- [S-CSCF](S-CSCF.md) — sends sessions to BGCF via Mi
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
