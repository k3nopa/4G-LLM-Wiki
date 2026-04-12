---
title: "SRVCC Procedures from UTRAN(HSPA) (§6.3)"
type: procedure
tags: [SRVCC, UTRAN, HSPA, SGSN, handover, CS-domain, GERAN, IMS, bearer-splitting]
sources: [ts23216.md]
updated: 2026-04-12
---

# SRVCC Procedures from UTRAN(HSPA) (§6.3)

SRVCC from UTRAN(HSPA) mirrors the [E-UTRAN procedures](SRVCC-from-E-UTRAN.md) structurally, but the PS-side control plane node is the **SGSN** (not the MME), and the SRVCC is triggered by the **source UTRAN(HSPA)** sending a Relocation Required to the SGSN (not Handover Required to MME).

Key difference in voice bearer identification: UTRAN(HSPA) uses **UMTS traffic classes** (not QCI) — the voice PS bearer is identified by `Traffic Class = Conversational` AND `Source Statistics Descriptor = 'speech'`.

---

## Attach/Service Request/PS HO Enablement (§6.3.1–§6.3.1B)

| Procedure | SRVCC addition |
|---|---|
| GPRS Attach / RAU | UE includes SRVCC capability in "MS Network Capability". HSS provides STN-SR and C-MSISDN in subscription data to SGSN. SGSN includes "SRVCC operation possible" in RANAP Common ID |
| Emergency Attach | UE includes SRVCC capability in Emergency Attach Request; SGSN stores for emergency SRVCC operation |
| Service Request | SGSN includes "SRVCC operation possible" in RANAP Common ID |
| Intra-UTRAN SRNS Relocation | Source SGSN sends MS Classmark 2/3, STN-SR, C-MSISDN, ICS Indicator, Supported Codec IE to target SGSN/MME. Target SGSN includes in RANAP Common ID; target MME includes in S1-AP Handover Request |

> **STN-SR roaming rule**: If a roaming subscriber's HPLMN does not allow SRVCC in the VPLMN, HSS does not include STN-SR and C-MSISDN in the SGSN subscription data.

---

## Scenario Matrix

| Scenario | Section | Target | PS HO | DTM |
|---|---|---|---|---|
| SRVCC UTRAN(HSPA) → GERAN (no DTM) | §6.3.2.1 | GERAN | No | No |
| SRVCC UTRAN(HSPA) → GERAN/UTRAN (DTM no HO / UTRAN no PS HO) | §6.3.2.1A | GERAN/UTRAN | No | With DTM |
| SRVCC UTRAN(HSPA) → UTRAN or GERAN (with DTM HO) | §6.3.2.2 | UTRAN or GERAN+DTM | Yes | Yes |

---

## §6.3.2.1 — SRVCC from UTRAN(HSPA) to GERAN without DTM support (24 steps)

```mermaid
sequenceDiagram
    participant UE
    participant RNC as Source UTRAN (HSPA)
    participant SGSN as Source SGSN
    participant MSC as MSC Server/MGW
    participant TMSC as Target MSC
    participant TBSS as Target BSS
    participant TSGSN as Target SGSN
    participant GW as GGSN/S-GW/P-GW
    participant IMS

    UE ->> RNC: 1. Measurement Reports
    RNC ->> SGSN: 2+3. Relocation Required\n(Target ID, Source→Target Transparent Container\nwith "old BSS to new BSS IE",\nSRVCC HO Indication = CS only,\nUE not available for PS at target)
    Note over SGSN: 4. Bearer Splitting:\nTraffic Class=Conversational AND SSD='speech'\n= voice bearer; split voice from non-voice
    SGSN ->> MSC: 5. SRVCC PS to CS Request\n(IMSI, Target ID, STN-SR, C-MSISDN,\nSource→Target Transparent Container,\nMM Context, Source SAI,\n[emergency: Emergency Indication + equipment ID],\n[if available: Authenticated IMSI + C-MSISDN])
    MSC ->> TMSC: 6. Prepare HO Request (BSSMAP encapsulated)
    TMSC ->> TBSS: 7. HO Request / Acknowledge
    TMSC ->> MSC: 8. Prepare HO Response\n(Target→Source Transparent Container)
    MSC ->> MSC: 9. Establish circuit (ISUP IAM + ACM)
    MSC ->> IMS: 10. Initiation of Session Transfer\n(ISUP IAM with STN-SR [non-emerg]\nor E-STN-SR + equipment ID [emerg])
    Note over IMS: 11. Session Transfer + remote end update:\nVoIP downlink switched to CS access leg SDP
    IMS -->> IMS: 12. Release of IMS access leg (per TS 23.237)
    MSC ->> SGSN: 13. SRVCC PS to CS Response\n(Target→Source Transparent Container)
    SGSN ->> RNC: 14. Relocation Command\n(Target→Source Transparent Container,\nvoice component info only)
    RNC ->> UE: 15. Handover Command
    UE ->> TBSS: 16. UE tunes to GERAN
    TBSS ->> TMSC: 17. HO Detection → Handover Detection; UE sends HO Complete
    TSGSN ->> SGSN: 18. Suspend procedure:\nTLLI+RAI from GUTI (TS 23.003);\nGn/Gp SGSN: Suspend Request → Source SGSN;\nS4-SGSN: Suspend Notification → Source SGSN;\nSource SGSN → Suspend Response/Acknowledge
    TBSS ->> TMSC: 19. Handover Complete
    TMSC ->> MSC: 20. SES (Handover Complete); speech circuit connected
    TMSC ->> MSC: 21. ISUP Answer
    MSC ->> SGSN: 22. SRVCC PS to CS Complete Notification
    SGSN ->> MSC: 22. SRVCC PS to CS Complete Acknowledge
    Note over SGSN,GW: 22a. Bearer handling:\nGn/Gp SGSN: deactivate voice PDP Contexts;\nsuspend non-voice (background/interactive)\nby setting max bitrate → 0 kbit/s\nS4-SGSN: MS-and-SGSN Bearer Deactivation\nfor voice; PS-to-CS HO indicator → P-GW;\nnon-GBR: Suspend Notification → S-GW
    SGSN ->> RNC: 22b. Iu Release Command\nSource RNC releases resources;\nsends Iu Release Complete
    opt Non-emergency: TMSI reallocation
        MSC ->> UE: 23a. TMSI Reallocation (via target MSC)
        MSC ->> MSC: 23b. MAP Update Location → HSS/HLR
    end
    opt Emergency location continuity
        SGSN -->> MSC: 24. Subscriber Location Report\n→ GMLC (source or target side)
    end
```

**Differences from E-UTRAN §6.2.2.1**:

| Aspect | E-UTRAN (§6.2.2.1) | UTRAN(HSPA) (§6.3.2.1) |
|---|---|---|
| Trigger node | MME | SGSN |
| Trigger message | Handover Required | Relocation Required |
| Voice bearer identification | QCI=1 | Traffic Class=Conversational + SSD='speech' |
| CS Security key derivation | From E-UTRAN/EPS domain key per TS 33.401 | From UTRAN(HSPA)/EPS domain key per TS 33.102; CS Security key sent in MM Context |
| Source SAI | Not explicitly specified | SGSN sets Source SAI = Serving Area Identifier received from source RNC |
| Post-HO Gn/Gp SGSN bearer handling | N/A (S-GW/P-GW) | Deactivates voice PDP Contexts; sets non-voice max bitrate → 0 kbit/s |
| Post-HO S4-SGSN bearer handling | MME-initiated Bearer Deactivation | SGSN-initiated Bearer Deactivation; S-GW releases all RNC-related info |
| PS resumption | TAU to MME | RAU to SGSN (Gn/Gp: GTPv1 via GGSN/PGW; S4: TAU to MME also possible) |

**Step 5 (SRVCC PS to CS Request) — MM Context content**:
- CS Security key derived by SGSN from UTRAN(HSPA)/EPS domain key per TS 33.102.
- Includes IMSI + C-MSISDN (from HSS subscription at UTRAN(HSPA) attach).
- Source SAI is set to the Serving Area Identifier received from the source RNC.

**Step 22a — Gn/Gp SGSN bearer handling detail**:
- Voice PDP Contexts: deactivated.
- Non-voice PDP Contexts using background or interactive traffic class: preserved but maximum bitrate set to 0 kbit/s (effectively suspended). For PDP Contexts using streaming or conversational traffic class NOT used for voice: preserved with max bitrate → 0 kbit/s.

**Step 22a — S4-SGSN bearer handling detail**:
- Initiates MS-and-SGSN Initiated Bearer Deactivation for voice bearer (per TS 23.060).
- PS-to-CS HO indicator notified to P-GW for voice bearer deactivation.
- If dynamic PCC: P-GW may interact with PCRF per TS 23.203.
- Non-GBR bearers: preserved via Suspend Notification to S-GW. S-GW releases all RNC-related information (address and TEIDs) for the UE if Direct Tunnel is established; sends Suspend Notification to P-GW.

### PS resumption after CS call ends

**Gn/Gp SGSN**: UE sends RAU to SGSN → SGSN resumes PDP Contexts per TS 23.060.  
**S4-SGSN**: Same; additionally informs S-GW and P-GW to resume suspended bearers via Modify Bearer Request (implicit resume if triggered by RAU/Service Request; explicit via Resume Notification otherwise).

---

## §6.3.2.1A — SRVCC with DTM/no DTM HO or UTRAN without PS HO

Identical to §6.3.2.1 **except**:
- **Step 18 (Suspend) is NOT performed** (source SGSN only deactivates voice bearers).
- At end of procedure, **remaining PS resources re-established** when UE performs RAU per TS 23.060.
- Target SGSN may deactivate PDP Contexts that cannot be established.

---

## §6.3.2.2 — SRVCC from UTRAN(HSPA) to UTRAN or GERAN with DTM HO support

Structurally identical to [§6.2.2.2](SRVCC-from-E-UTRAN.md) (CS+PS parallel HO). Key SGSN-specific additions:

```mermaid
sequenceDiagram
    participant UE
    participant RNC as Source UTRAN (HSPA)
    participant SGSN as Source SGSN
    participant MSC as MSC Server/MGW
    participant TMSC as Target MSC
    participant TSGSN as Target SGSN
    participant TRNS as Target RNS/BSS
    participant GW as S-GW/P-GW/GGSN
    participant IMS

    UE ->> RNC: 1. Measurement Reports
    Note over RNC,SGSN: 2a. For SRVCC to UTRAN:\nRNC → SGSN: SRVCC CS KEYS REQUEST\n2b. SGSN → RNC: SRVCC CS KEYS RESPONSE\n(Integrity Protection Key IE,\nEncryption Key IE, SRVCC Information IE)
    RNC ->> SGSN: 3. Relocation Required\n(Target ID, Source RNC→Target RNC Transparent Container,\nSRVCC HO Indication = CS+PS HO)
    Note over SGSN: 4. Bearer Splitting:\nTraffic Class=Conversational + SSD='speech' = voice\nVoice → MSC Server (CS path)\nNon-voice PS bearers → target SGSN (PS path)
    par CS path
        SGSN ->> MSC: 5a. SRVCC PS to CS Request\n(IMSI, Target ID, STN-SR, C-MSISDN,\nSource→Target Transparent Container,\nMM Context, PDP Context)
        MSC ->> TMSC: 5b. Prepare HO Request
        TRNS -->> TSGSN: 5c. Reloc/HO Request (CS)
        TRNS ->> TMSC: 7+8a. Reloc/HO Req Ack
        TMSC ->> MSC: 8b. Prepare HO Response
        MSC ->> MSC: 8c. Establish circuit (ISUP IAM + ACM)
    and PS path
        SGSN ->> TSGSN: 6a. Forward Relocation Request\n(Source→Target Transparent Container,\nMM Context, PDP Context,\nbearing info for all non-voice bearers;\nS4-variant: PDN Connections IE;\nGn/Gp variant: PDP Contexts)
        TSGSN ->> TRNS: 6b. Reloc/HO Request (PS)
        TRNS ->> TSGSN: 7a. Reloc/HO Req Ack
        TSGSN ->> SGSN: 7b. Forward Reloc Response
    end
    MSC ->> IMS: 9. Session Transfer (STN-SR or E-STN-SR)
    Note over IMS: 10+11. Session Transfer complete; IMS access leg released
    MSC ->> SGSN: 12. SRVCC PS to CS Response\n(Target→Source Transparent Container)
    SGSN ->> RNC: 13. Relocation Command\n(synchronises CS+PS containers;\nGERAN target: source RNC receives SRVCC Info IE with NONCE IE)
    RNC ->> UE: 14+15. HO from UTRAN Command
    UE ->> TRNS: 16. UE tunes to UTRAN/GERAN; HO Detection
    par CS completion
        TRNS ->> TMSC: 17a. Reloc/HO Complete
        TMSC ->> MSC: 17b. SES (Handover Complete)
        MSC ->> MSC: 17c. ANSWER
        MSC ->> SGSN: 17d. PS to CS Complete Notification/Ack
        SGSN ->> GW: 17e. Delete voice bearer\n(sets PS-to-CS HO indicator; PCC if dynamic)
        opt Non-emergency
            MSC ->> UE: 17f. TMSI Reallocation
            MSC ->> MSC: 17g. UpdateLoc → HSS/HLR
        end
    and PS completion
        TRNS ->> TSGSN: 18a. Reloc/HO Complete
        TSGSN ->> SGSN: 18b. Forward Reloc Complete; SGSN → Ack to TSGSN
        TSGSN ->> GW: 18c. Update bearer (GGSN/S-GW/P-GW)
        SGSN ->> GW: 18d. Delete Session Request\n(S4-SGSN → S-GW; Gn/Gp → GGSN)
        SGSN ->> RNC: 18e. Iu Release Command; RNC → Iu Release Complete
    end
    opt Emergency location continuity
        SGSN -->> MSC: 19. Subscriber Location Report → GMLC
    end
```

**Step 2a-2b — SRVCC CS KEYS exchange (UTRAN target only)**:
- Before Relocation Required, for SRVCC to UTRAN: RNC initiates the **SRVCC Preparation** by sending SRVCC CS KEYS REQUEST to source SGSN.
- SGSN responds with SRVCC CS KEYS RESPONSE containing:
  - Integrity Protection Key IE
  - Encryption Key IE
  - SRVCC Information IE
- This is required so the UTRAN target cell has the correct CS security keys derived from the PS security context.

**Step 6a — Forward Relocation Request variants**:
- S4-SGSN (S-GW/P-GW): Forward Relocation Request contains **Source-to-Target Transparent Container, MM Context, PDP Context** with bearer info for all bearers except voice bearer. Security key handling per TS 33.401.
- Gn/Gp-SGSN (GGSN): Forward Relocation Request contains **PDP Contexts** (instead of PDN Connections IE) including bearer info for all bearers except voice bearer. Security key handling per TS 33.102.

**Step 13 — Relocation Command coordination**:
- Source SGSN synchronises both CS and PS prepared relocations.
- Sends single Relocation Command with Target-to-Source Transparent Container containing both CS and PS resource allocations.
- For GERAN target: source RNC shall receive **SRVCC Information IE with NONCE IE** (step 13 in TS 23.216 §6.3.2.2 note).

**Fallback to §6.3.2.1A**: If only voice bearer relocation succeeds (PS fails), SGSN proceeds with step 13 after receiving SRVCC PS to CS Response (step 12); both UE and SGSN continue as described in §6.3.2.1A.

---

## SGSN Bearer Splitting Summary

| Bearer type | UTRAN(HSPA) identification | Action |
|---|---|---|
| Voice PS bearer | Traffic Class = Conversational AND SSD = 'speech' | Send to MSC Server via Sv (SRVCC path) |
| vSRVCC video bearer | _Not applicable for UTRAN(HSPA) in this spec_ | — |
| Non-voice PS bearers (CS+PS HO) | All others | Forward Relocation Request to target SGSN (PS HO path) |
| Non-voice PS bearers (CS-only HO) | All others | Suspend or deactivate after step 22a |

---

## Cross-references

- [SRVCC concept](../concepts/SRVCC.md) — all variants, architecture, entities
- [SRVCC procedures from E-UTRAN](SRVCC-from-E-UTRAN.md) — §6.1–§6.2 (MME-based flows)
- [SRVCC CS-to-PS and 5G-SRVCC](SRVCC-CS-to-PS-and-5G.md) — §6.4–§6.5
- Source: [TS 23.216](../sources/ts23216.md)
