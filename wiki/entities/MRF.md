---
title: "MRF (MRFC + MRFP)"
type: entity
tags: [IMS, MRF, MRFC, MRFP, conference, media, H.248, SIP, Mr, Mp]
sources: [ts_123228v150600p.pdf]
updated: 2026-04-08
---

# MRF — Multimedia Resource Function (MRFC + MRFP)

**Spec reference:** 3GPP TS 23.228 §4.12

## Role

The MRF provides **in-network media processing** for IMS sessions. It is split into two components following the ITU-T BICC/H.248 decomposition model:

| Component | Role |
|---|---|
| **MRFC** | Controller: SIP signaling + H.248 control of MRFP |
| **MRFP** | Processor: actual RTP mixing, tone injection, transcoding, recording |

---

## MRFC Functions

| Function | Description |
|---|---|
| **SIP endpoint** | Appears as a SIP UAS/UAC; receives Mr from S-CSCF or TAS |
| **Conference controller** | Creates/manages conference resources on MRFP |
| **Announcement controller** | Triggers MRFP to play tones or announcements |
| **Transcoding control** | Instructs MRFP to transcode between codecs |
| **H.248 master** | Sends H.248 commands to MRFP to allocate ports, mix streams, inject tones |

## MRFP Functions

| Function | Description |
|---|---|
| **RTP mixer** | Mixes multiple RTP streams for conference; sends each participant a mix of others |
| **Tone generator** | Generates DTMF, ringback, busy tones, custom announcements |
| **Transcoder** | Converts between codecs (e.g. AMR-WB ↔ G.711) |
| **Media recorder** | Can record RTP streams (e.g. for voicemail) |
| **Bearer termination** | Terminates RTP/RTCP; connects to user media bearers |

---

## Interfaces

| Interface | Peer | Protocol | Purpose |
|---|---|---|---|
| **Mr** | S-CSCF or TAS (AS) | SIP | Request media resources; S-CSCF sends INVITE to MRFC for conference leg |
| **Mp** | MRFP | H.248 (MEGACO) | MRFC controls MRFP: create/modify/delete connections, play tones |

---

## Conference Call Flow (simplified)

```
Participant A ──INVITE──► S-CSCF
                           │──Mr INVITE──► MRFC
                           │              │──Mp H.248──► MRFP (allocate port)
                           │◄──200 OK─────│
                           ◄──200 OK──── S-CSCF (SDP: MRFP RTP port)
A sends RTP ────────────────────────────────────────────► MRFP

(Repeat for Participants B, C)

MRFP mixes A+B+C streams and sends each participant mixed stream.
```

---

## TAS Interaction

For IMS conferencing via TAS (AS via ISC), the TAS acts as the focus (B2BUA). TAS initiates legs to the MRFC via the Mr interface (through S-CSCF or directly). The MRFC manages the mixer bridge via H.248 on Mp.

---

## Related Pages
- [S-CSCF](S-CSCF.md) — sends Mr requests to MRFC
- [TAS](TAS.md) — may control conferencing via Mr
- [IMS Reference Points](../interfaces/IMS-reference-points.md)
