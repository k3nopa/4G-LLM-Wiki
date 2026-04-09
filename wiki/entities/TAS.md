---
title: "TAS (Telephony Application Server)"
type: entity
tags: [IMS, TAS, AS, application-server, ISC, iFC, VoLTE, MMTEL, telephony, services]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# TAS — Telephony Application Server

**Spec reference:** 3GPP TS 23.228 §4.13; TS 23.218 (iFC/service control detail)

## Role

The TAS is an **IMS Application Server** that executes telephony supplementary services on behalf of IMS subscribers. It is the most important AS in a VoLTE deployment. It communicates with the S-CSCF via the **ISC interface** (SIP), triggered by iFC evaluation.

Operators typically implement TAS as the **MMTEL AS** (TS 24.173), supporting the full IMS Multimedia Telephony (MMTel) feature set.

---

## Services Provided

| Service | Description |
|---|---|
| **CLIP/CLIR** | Calling/Connected Line Identification Presentation/Restriction |
| **Call Forwarding** | Unconditional (CFU), No Reply (CFNRy), Busy (CFB), Not Reachable (CFNRc) |
| **Call Hold / Resume** | SIP re-INVITE with a=inactive/sendrecv |
| **Call Waiting** | Allow second incoming call while first is active |
| **Call Barring** | Outgoing/incoming call restrictions |
| **Conference** | Multi-party calling; TAS as conference focus; controls MRFC |
| **Voicemail** | Forward to voicemail AS on no-reply/busy |
| **DTMF / IVR** | Interact with MRFP for DTMF relay |
| **Explicit Communication Transfer (ECT)** | Blind/attended transfer |
| **Message Waiting Indication (MWI)** | NOTIFY to UE of waiting voicemails |

---

## AS Modes (per TS 23.218)

The TAS can operate in multiple modes depending on the service and iFC configuration:

| Mode | Description |
|---|---|
| **Terminating UA** | TAS terminates the SIP dialog (e.g. voicemail — records media) |
| **Originating UA** | TAS generates new SIP dialogs (e.g. click-to-dial, MWI NOTIFY) |
| **SIP proxy** | TAS transparently forwards; may modify headers (e.g. CLIR: remove From) |
| **B2BUA** | TAS fully controls both sides; used for call hold, transfer, conference |
| **3rd party call control** | TAS establishes sessions between two UEs without being in the media path |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **ISC** | S-CSCF | SIP | Receive iFC-triggered requests; return control to S-CSCF |
| **Sh** | HSS | Diameter | Read/write subscriber telephony service data (call forwarding targets, barring flags) |
| **Mr** | MRFC | SIP | Request conference/announcement resources |
| **Ut** | UE | HTTP(S)/XCAP | Subscriber self-service configuration of supplementary services |

---

## ISC Interaction — iFC Triggering

```
UE──INVITE──►S-CSCF
              │ Evaluate iFC:
              │  Priority=1, SPT: Direction=Orig → AS=sip:tas.ims.operator.com
              │
              │──ISC INVITE──►TAS
              │               │ (apply CLIR, call barring, call forwarding checks)
              │◄──ISC INVITE──│ (with ODI token in Route header)
              │  continue iFC chain
              ▼
           route to terminating side
```

The ODI (Original Dialog Identifier) is an opaque token the S-CSCF inserts in the Route header when sending to the AS. The AS must include this in its outgoing Route to return control to the S-CSCF at the correct iFC chain position.

---

## Call Forwarding Flow (CFB)

```
Terminating INVITE arrives at S-CSCF
  │ iFC: terminating, INVITE → TAS
  │──ISC INVITE──►TAS
  │               │ Check: UE busy? Yes.
  │               │ CFB target: +4412345678
  │◄──302 Moved Temporarily (Contact: cfb-target)
  │  or TAS re-INVITEs to forward target (B2BUA)
```

---

## Relationship with HSS (Sh)

TAS uses Sh (Diameter) to:
- Read subscriber service data: CFU target, CFB/CFNRy/CFNRc flags, barring lists
- Write subscriber data if UE updates via Ut (XCAP)
- Subscribe to data change notifications (HSS pushes Sh SNR when data changes)

---

## Related Pages
- [S-CSCF](S-CSCF.md) — triggers TAS via ISC; evaluates iFCs
- [MRF](MRF.md) — TAS requests media resources via Mr
- [HSS](HSS.md) — TAS reads service data via Sh
- [IMS Identity Model](../concepts/IMS-identity-model.md) — service profile, iFC
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
