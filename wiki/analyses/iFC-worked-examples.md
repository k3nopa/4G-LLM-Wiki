---
title: "iFC Worked Examples: Call Forwarding, MRFC Services, Voicemail"
type: analysis
tags: [IMS, iFC, AS, call-forwarding, conference, announcement, transcoding, voicemail, MRFC, B2BUA, proxy, redirect, S-CSCF]
sources: [ts_123218v170000p.pdf]
updated: 2026-04-10
---

# iFC Worked Examples: Call Forwarding, MRFC Services, Voicemail

From TS 23.218 Annexes B and C (informative). These examples illustrate the normative
procedures from §6 and §9 in concrete service scenarios.

Related pages: [IM Call Model](../concepts/IM-call-model.md) ·
[AS Interaction Modes](../concepts/AS-interaction-modes.md) ·
[MRF](../entities/MRF.md) · [MRB](../entities/MRB.md) ·
[S-CSCF](../entities/S-CSCF.md)

---

## Example 1: iFC Chain Triggering (Annex C)

**Scenario**: User has two FCs — FC-X triggers AS1, FC-Y triggers AS2.

```mermaid
sequenceDiagram
    participant UE
    participant SCSCF as S-CSCF\n(FC-X→AS1, FC-Y→AS2)
    participant AS1
    participant AS2
    participant Dest

    UE->>SCSCF: 1. INVITE
    Note over SCSCF: Evaluate FC-X: SPT match
    SCSCF->>AS1: 2. INVITE (ODI inserted)
    Note over AS1: Service logic executes\nMay modify request
    AS1->>SCSCF: 3. INVITE (ODI returned, possibly modified)
    Note over SCSCF: Evaluate FC-Y: SPT match
    SCSCF->>AS2: 4a. INVITE (ODI inserted)
    Note over AS2: Service logic executes\nMay modify request
    AS2->>SCSCF: 5a. INVITE (ODI returned)
    Note over SCSCF: No more FCs matching
    SCSCF->>Dest: 6a. INVITE (normal routing)
```

**Key insight**: S-CSCF re-evaluates remaining FCs every time the request returns from
an AS. FC-Y fires only after AS1 has returned the request — AS1 cannot suppress FC-Y
evaluation (only Record-Route denial can do that).

If no further FCs match at step 4b, S-CSCF routes directly to destination (step 4b/6a).

---

## Example 2: Call Forwarding (CFonCLI) — Annex B.1

**Service**: UE2 has Call Forwarding on CLI (forward calls from UE1's number to UE3).
**AS mode**: Redirect Server (Mode 1) or Proxy (Mode 3) depending on variant.

### Variant A: UE-Redirect (B.1.3)

AS acts as **Redirect Server** (Mode 1). AS sends 302 Moved Temporarily. UE1 itself
re-issues INVITE to UE3.

```mermaid
sequenceDiagram
    participant UE1
    participant S1 as S-CSCF UE1
    participant I2 as I-CSCF UE2
    participant S2 as S-CSCF UE2
    participant AS as CF AS
    participant UE3net as UE3 Home Net

    UE1->>S1: 1. INVITE (to UE2)
    S1->>I2: 2. INVITE
    I2->>S2: 5. INVITE (HSS lookup done)
    S2->>AS: 6. INVITE (iFC match — CF service)
    Note over AS: CLI check: UE1 number matches\nForward to UE3
    AS-->>S2: 7. 302 Moved Temporarily (Contact: UE3)
    S2-->>I2: 9. 302
    I2-->>S1: 10. 302
    S1-->>UE1: 13. 302
    UE1->>S1: 15. INVITE (to UE3, new request)
    S1->>UE3net: 16. INVITE
    Note over UE1,UE3net: 17. Bearer establishment and call setup
```

AS is no longer in path after the 302 — clean handoff.

### Variant B: S-CSCF Redirect (B.1.4)

AS acts as **SIP Proxy** (Mode 3). AS notifies all parties call is being forwarded via
181, then modifies Request-URI and returns INVITE to S-CSCF.

```mermaid
sequenceDiagram
    participant UE1
    participant S2 as S-CSCF UE2
    participant AS as CF AS (Proxy)
    participant UE3net as UE3 Home Net

    S2->>AS: 6. INVITE
    AS->>S2: 7a-7d. 181 Call Is Being Forwarded\n(notifies UE1 in path)
    Note over AS: Proxy mode: modifies Request-URI to UE3\nreturns INVITE to S-CSCF
    AS->>S2: 8. INVITE (Request-URI = UE3)
    S2->>UE3net: 9. INVITE (routed via I-CSCF UE3)
    Note over UE1,UE3net: 14. Bearer establishment and call setup
```

**Difference from Variant A**: 181 rings back through the entire path to UE1 (steps
7a–7d), so UE1 sees the "forwarding" notification. UE1 does NOT need to re-issue INVITE.

---

## Example 3: Announcement on UE-Originating Session (Annex B.2.1)

**Service**: UE originates call; destination unreachable; AS plays announcement.
**AS mode**: B2BUA (Mode 4, Routing B2BUA).

```mermaid
sequenceDiagram
    participant UE as UE (calling)
    participant AS as AS (B2BUA)
    participant SCSCF as S-CSCF
    participant MRFC

    UE->>SCSCF: 1. INVITE [Call-ID 1]
    SCSCF->>AS: 3. INVITE [Call-ID 1] (iFC match)
    Note over AS: Service logic: proceed
    AS->>SCSCF: 4. INVITE [Call-ID 2] (toward dest)
    SCSCF-->>AS: 6. Session Failure (can't route)
    AS->>SCSCF: 7. ACK [Call-ID 2]
    Note over AS: Service logic: play announcement
    AS->>SCSCF: 9. INVITE (SDP-A) [Call-ID 3]\n(request announcement from MRFC)
    SCSCF->>MRFC: 10. INVITE (SDP-A) [Call-ID 3]
    MRFC-->>SCSCF: 11. 200 OK (SDP-M) — resource allocated
    SCSCF-->>AS: 12. 200 OK (SDP-M)
    AS->>UE: 13. 183 Session Progress (SDP-M)\nvia S-CSCF [Call-ID 1]
    Note over AS,MRFC: Preconditions + QoS (steps 14-25)
    AS-->>UE: 25. 200 OK [Call-ID 1]
    SCSCF->>MRFC: 26. ACK [Call-ID 3]
    MRFC->>MRFC: 28. Play announcement
    UE->>SCSCF: 29. ACK [Call-ID 1]
```

**Key pattern**: AS uses two Call-IDs. Call-ID 2 is the attempted outgoing call (fails).
Call-ID 3 is the MRFC announcement leg. B2BUA correlates both with Call-ID 1 (the
original UE-AS dialog). ACK on Call-ID 3 (step 26) triggers MRFC to start playing.

---

## Example 4: Ad-Hoc Conference (Annex B.2.2)

**Service**: UE-1 requests to bring UE-2 and UE-3 into conference.
**AS mode**: B2BUA (Mode 4). MRFC provides conference bridge.

Call-ID assignment:
- Call-ID 1: UE-1 → AS (original request: INVITE MPTY)
- Call-ID 2: AS → MRFC (conference identifier assigned for UE-2 party)
- Call-ID 3: AS → UE-2 (re-INVITE to bring UE-2 into conference bridge)
- Call-ID 4: AS → MRFC (same conference identifier for UE-3 party)
- Call-ID 5: AS → UE-3 (re-INVITE for UE-3)
- Call-ID 6: AS → MRFC (same conference identifier for UE-1 party)

```mermaid
sequenceDiagram
    participant UE1
    participant AS
    participant SCSCF as S-CSCF
    participant MRFC
    participant UE2
    participant UE3

    UE1->>SCSCF: 1. INVITE MPTY [1]
    SCSCF->>AS: 3. INVITE [1]
    Note over AS: Service logic: start conference
    AS->>SCSCF: 5. INVITE [2] (to MRFC, conference-id, UE-2 SDP)
    SCSCF->>MRFC: 6. INVITE [2]
    MRFC-->>AS: 8. 200 OK [2] — conference resource allocated
    Note over MRFC: Path established: UE-2 ↔ MRFP
    AS->>UE2: 9-13. re-INVITE [3] — bring UE-2 into bridge
    AS->>SCSCF: 18. INVITE [4] (same conference-id, UE-3 SDP)
    SCSCF->>MRFC: 19. INVITE [4]
    AS->>UE3: 22-26. re-INVITE [5] — bring UE-3 into bridge
    AS->>SCSCF: 31. INVITE [6] (same conference-id, UE-1 SDP)
    SCSCF->>MRFC: 32. INVITE [6]
    AS-->>UE1: 35-36. 200 OK [1]
    Note over MRFC: 37. Paths established: all UEs ↔ MRFP
```

**Key insight**: same **conference identifier** is reused in all MRFC INVITEs (Call-IDs
2, 4, 6). MRFC uses this identifier to add each party to the same conference bridge.
First INVITE creates the conference; subsequent INVITEs add participants.

---

## Example 5: Transcoding (Annex B.2.3)

**Service**: Called UA returns 606 Not Acceptable (codec mismatch); AS invokes MRFC
for transcoding.
**AS mode**: B2BUA (Mode 4).

Two variants depending on whether called UA provides codec info in 606:

### Variant A: Called UA Indicates Codec (B.2.3.1)

```mermaid
sequenceDiagram
    participant UE as UE (calling)
    participant AS
    participant SCSCF as S-CSCF
    participant MRFC
    participant CUA as Called UA

    AS->>SCSCF: 5. INVITE (UE SDP) [2]
    SCSCF->>CUA: 6. INVITE [2]
    CUA-->>SCSCF: 7. 606 Not Acceptable\n(UA SDP — acceptable codec indicated)
    Note over AS: Service logic: use MRFC for transcoding
    AS->>SCSCF: 12. INVITE (UA SDP) [3] → MRFC
    MRFC-->>AS: 15. 200 OK [3] — transcoding resource (UA codec side)
    AS->>CUA: 18. INVITE (UA SDP) [4]
    CUA-->>AS: 20. 200 OK [4]
    AS->>SCSCF: 26. INVITE (UE SDP) [5] → MRFC
    MRFC-->>AS: 29. 200 OK [5] — transcoding resource (UE codec side)
    Note over MRFC: Transcodes between UE codec and UA codec
    Note over AS,UE: Normal call establishment continues
```

Three dialogs to MRFC: [3] for called UA codec side, [5] for calling UE codec side.
MRFP interconnects both transcoding legs.

### Variant B: Called UA Provides No SDP (B.2.3.2)

If 606 contains no SDP, AS must query MRFC for supported codecs first:
1. AS sends INVITE (no SDP) to MRFC [3] → MRFC responds with 183 (MRF SDP — codec list)
2. AS sends that codec list in INVITE to called UA [4]
3. Called UA accepts specific codec → PRACK negotiation
4. AS establishes transcoding (rest same as Variant A)

---

## Example 6: Voicemail — Out-of-Coverage Message Recording (Annex B.3.1)

**Service**: UE is unregistered; incoming call forwarded to voicemail AS.
**AS mode**: Terminating UA (Mode 1). Triggered via unregistered terminating iFC.
**iFC trigger**: Default Filter Criteria for unregistered user → Voicemail AS.

```mermaid
sequenceDiagram
    participant Caller
    participant SCSCF as S-CSCF\n(serving called UE)
    participant AS as AS (Voicemail)

    Caller->>SCSCF: 1. INVITE (for unregistered UE)
    Note over SCSCF: iFC match: default FC → Voicemail
    SCSCF->>AS: 2. INVITE
    AS->>AS: Start voicemail application\n(checks subscriber profile via Sh)
    AS-->>SCSCF: 3. 183 Session Progress (SDP)
    Note over Caller,AS: 5-8: PRACK/200 OK exchange
    Note over Caller,AS: 9: QoS Establishment / Resource Reservation
    Note over Caller,AS: 10-15: UPDATE/200 OK (preconditions)
    AS-->>Caller: 15. 200 OK (call accepted)
    Note over AS: 18. Play announcement\n(UE powered off / out of coverage)
    Note over Caller: 19. Leave message
    Caller->>SCSCF: 20. BYE
    SCSCF->>AS: 21. BYE
    AS-->>Caller: 23. 200 OK
```

**Terminating UA pattern**: AS responds with 183 (preconditions), accepts call with
200 OK, then controls media session. Caller's BYE goes through S-CSCF to AS.

---

## Example 7: Voicemail — On-Registration Message Notification (Annex B.3.2)

**Service**: UE registers; voicemail AS detects stored messages; AS calls UE back.
**AS mode**: Originating UA (Mode 2). Triggered by third-party REGISTER.

```mermaid
sequenceDiagram
    participant UE
    participant SCSCF as S-CSCF
    participant AS as AS (Voicemail)

    UE->>SCSCF: 1-4. REGISTER (401 challenge → authenticated REGISTER → 200 OK)
    Note over SCSCF: iFC on REGISTER → third-party REGISTER to Voicemail AS
    SCSCF->>AS: 5. REGISTER (third-party)
    AS-->>SCSCF: 6. 200 OK
    Note over AS: 7. Downloads subscriber data via Sh\nDetects recorded messages
    AS->>SCSCF: 7. INVITE (to UE — AS as originating UA)
    SCSCF->>UE: 8. INVITE
    Note over UE,AS: 9-29: Session establishment with preconditions\n(183 + PRACK + UPDATE + 180 Ringing + 200 OK)
    AS-->>UE: 29. ACK
    Note over AS: 30. Play announcement + messages
    UE->>SCSCF: 31. BYE
    SCSCF->>AS: 32. BYE
    AS-->>UE: 34. 200 OK
```

**Originating UA pattern**: AS generates INVITE without being triggered by an incoming
SIP request. The trigger is the third-party REGISTER event. AS downloads subscriber
profile via Sh to determine that messages are waiting.

---

## Pattern Summary

| Example | AS Mode | Dialogs | MRFC? | iFC Trigger |
|---|---|---|---|---|
| iFC chain | Proxy (Mode 3) | 1 | No | Multiple FCs in sequence |
| CFonCLI (redirect) | Redirect Server | 1 | No | Terminating, registered |
| CFonCLI (proxy) | SIP Proxy | 1 | No | Terminating, registered |
| Announcement | Routing B2BUA | 3 | Yes (announcement) | Originating |
| Ad-hoc conference | Routing B2BUA | 6 | Yes (conference) | Originating |
| Transcoding | Routing B2BUA | 4–5 | Yes (transcoding) | Originating |
| Voicemail deposit | Terminating UA | 1 | No | Unregistered terminating |
| Voicemail playback | Originating UA | 1 | No | REGISTER trigger |
