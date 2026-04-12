---
title: "SRVCC Procedures from E-UTRAN and 5G-SRVCC (§6.1–§6.2, §6.5)"
type: procedure
tags: [SRVCC, vSRVCC, 5G-SRVCC, E-UTRAN, NG-RAN, MME-SRVCC, AMF, handover, CS-domain, 1xCS, GERAN, UTRAN, IMS, bearer-splitting]
sources: [ts23216.md]
updated: 2026-04-12
---

# SRVCC Procedures from E-UTRAN

Covers two major variants anchored at the [MME](../entities/MME-deepdive.md):
- **§6.1** — E-UTRAN to 3GPP2 1xCS (via S102 tunnel)
- **§6.2** — E-UTRAN to 3GPP UTRAN/GERAN and vSRVCC (via Sv interface)

For the concept overview and entity descriptions see [SRVCC](../concepts/SRVCC.md).

---

## Part 1: E-UTRAN to 3GPP2 1xCS SRVCC (§6.1)

### Attach/Service Request/PS HO enablement

| Procedure | SRVCC addition |
|---|---|
| E-UTRAN Attach / TAU | UE includes SRVCC capability in "UE Network Capability" IE. MME stores this. MME includes "SRVCC operation possible" in S1-AP Initial Context Setup Request |
| Emergency Attach | UE includes SRVCC capability in Emergency Attach Request; MME stores for emergency SRVCC operation |
| Service Request | MME includes "SRVCC operation possible" in S1-AP Initial Context Setup Request |
| S1-based intra-E-UTRAN HO | Target MME includes "SRVCC operation possible" in S1-AP Handover Request |
| X2-based intra-E-UTRAN HO | Source eNB includes "SRVCC operation possible" in X2-AP Handover Request |

### Call flow: E-UTRAN to 1xCS (19 steps)

```mermaid
sequenceDiagram
    participant UE as 1xCS SRVCC UE
    participant ENB as eNB (E-UTRAN)
    participant MME
    participant SGW as S-GW/P-GW
    participant IWS as 1xCS IWS
    participant MSC as 1xRTT MSC
    participant CS as 1xRTT CS access

    Note over UE,CS: 1. Ongoing VoIP session over IMS/E-UTRAN access leg
    UE ->> ENB: 2. Measurement Reports
    ENB ->> ENB: 3. Handover decision (inter-technology → cdma2000 1xRTT)
    ENB ->> UE: 4. HO from EUTRA Preparation Request\n(3G1x Overhead Parameters, RAND value)
    UE ->> ENB: 5. UL Handover prep transfer\n(MEID, 1x Origination)\n[emergency: Request-Type="emergency handover", MEID included]
    ENB ->> MME: 6. UL S1 cdma2000 Tunnelling\n(MEID, RAND, 1x Origination, Reference CellID)\n+ CDMA2000 HO Required Indication IE
    MME ->> IWS: 7. S102 Direct Transfer\n("1x Air Interface Signalling (Origination)"\ncontaining MEID + RAND + 1x Origination Message)
    Note over IWS,MSC: 8. 1xRTT traffic assignment + 3GPP2 1xCS\nSession Transfer procedures per X.S0042
    IWS ->> MME: 9. S102 Direct Transfer\n(1x msg + Handover Direction indicator:\nsuccess=include HO Direction msg,\nfailure=failure indication)
    MME ->> ENB: 10. DL S1 cdma2000 Tunnelling\n(1x message + CDMA2000 HO Status IE\nbased on handover indicator from IWS)
    alt Success
        ENB ->> UE: 11. Mobility from EUTRA Command\n(embedded 1xRTT HO Direction message)
    else Failure
        ENB ->> UE: 11. DL Information Transfer\n(embedded 1xRTT failure message)
    end
    UE ->> CS: 12. UE retunes to 1xRTT radio;\ntraffic channel acquisition (1xRTT CS access / BSS)
    UE ->> CS: 13. 1xRTT handoff completion message
    CS ->> MSC: 14. Handoff done; IWS–MSC resources may release
    Note over UE,CS: 15. Ongoing voice call over CS access leg (1xRTT access)
    ENB ->> MME: 16. S1 UE Context Release Request\n(Cause: HO from E-UTRAN to 1xRTT)
    MME ->> SGW: 17. MME deactivates GBR bearers\n(MME-initiated Dedicated Bearer Deactivation)\nPS-to-CS HO indicator → P-GW for voice bearer\nNon-GBR bearers: Suspend Notification to S-GW\nS-GW marks UE suspended; P-GW discards packets
    ENB ->> ENB: 18. S1 UE Context released per TS 23.401
    opt Emergency session after HO
        MME ->> MME: 19. Subscriber Location Report\n(Reference Cell ID → GMLC at source side)\nper TS 23.271
    end
```

**Key step annotations:**
- **Step 7**: MME selects 1xCS IWS based on Reference CellID received from eNB (§5.3.3.1.2 — local configuration in MME).
- **Step 8 & 9 are independent**: 3GPP2 1xCS procedures (step 8) and 3GPP S102 procedures (step 9) are separate; they do not depend on each other's ordering.
- **Step 17 (bearer handling)**: The MME stores in the UE context that the UE is in suspended status. All preserved non-GBR bearers are marked as suspended in S-GW and P-GW. P-GW discards packets received for the suspended UE.
- **Emergency**: For emergency SRVCC, the "VDN" parameter in 3GPP2 X.S0042 corresponds to the STN-SR parameter defined in TS 23.237. The 1xCS emergency STN-SR is specified by 3GPP2.

### PS resumption after CS call ends

If the UE returns to E-UTRAN after the CS voice call terminates, the UE resumes PS service by sending **TAU** to the MME. The MME informs S-GW and P-GW(s) to resume the suspended bearers. Resumption in S-GW and P-GW should be done by implicit resume via Modify Bearer Request (if triggered by TAU/Service Request). Explicit resume via Resume Notification message is used if Modify Bearer Request is not triggered.

---

## Part 2: E-UTRAN to 3GPP UTRAN/GERAN (v)SRVCC (§6.2)

### Attach/Service Request/PS HO enablement

| Procedure | SRVCC addition |
|---|---|
| E-UTRAN Attach / TAU | UE includes (v)SRVCC capability in "MS Network Capability". HSS provides STN-SR, C-MSISDN, vSRVCC flag to MME via S6a. MME includes "SRVCC operation possible" in S1-AP Initial Context Setup Request (only if STN-SR + C-MSISDN present) |
| Emergency Attach | UE includes SRVCC capability in Emergency Attach Request |
| Service Request | MME includes "SRVCC operation possible" in S1-AP Initial Context Setup Request |
| S1-based HO | Source MME sends MS Classmark 2/3, STN-SR, C-MSISDN, ICS Indicator, Supported Codec IE to target MME/SGSN. Target MME includes "SRVCC operation possible" in S1-AP HO Request; target SGSN includes in RANAP Common ID |
| X2-based HO | Source eNB includes "SRVCC operation possible" in X2-AP HO Request |
| Connected mode SRVCC capability update | MME sends updated "SRVCC operation possible" to eNB via existing UE context modification |
| Dedicated Bearer for vSRVCC (§6.2.1C) | PCRF marks video bearer with vSRVCC indicator; P-GW and S-GW include "vSRVCC indicator" in Create Bearer Request / Update Bearer Request |

> **STN-SR provisioning rule**: If STN-SR is present in HSS subscription, UE is SRVCC subscribed. If a roaming subscriber's HPLMN does not allow SRVCC in the VPLMN, HSS does not include STN-SR and C-MSISDN. If STN-SR or C-MSISDN are absent, the MME shall NOT set "SRVCC operation possible".

### Scenario matrix

| Scenario | Section | Target | PS HO | DTM | vSRVCC |
|---|---|---|---|---|---|
| SRVCC E-UTRAN → GERAN (no DTM) | §6.2.2.1 | GERAN | No | No | No |
| SRVCC E-UTRAN → GERAN/UTRAN (DTM no HO / UTRAN no PS HO) | §6.2.2.1A | GERAN/UTRAN | No | With DTM | No |
| SRVCC E-UTRAN → UTRAN/GERAN (PS HO + DTM HO) | §6.2.2.2 | UTRAN or GERAN+DTM | Yes | Yes | No |
| vSRVCC E-UTRAN → UTRAN (PS HO) | §6.2.2.3 | UTRAN | Yes | — | Yes |
| vSRVCC E-UTRAN → UTRAN (no PS HO) | §6.2.2.4 | UTRAN | No | — | Yes |

---

### §6.2.2.1 — SRVCC from E-UTRAN to GERAN (no DTM support) — 24 steps

This is the baseline SRVCC flow: CS-only handover, no parallel PS handover, no DTM. The Suspend procedure at step 18 deactivates all PS bearers.

```mermaid
sequenceDiagram
    participant UE
    participant EUTRAN as Source E-UTRAN
    participant MME as Source MME
    participant MSC as MSC Server/MGW
    participant TMSC as Target MSC
    participant TBSS as Target BSS
    participant SGSN as Target SGSN
    participant SGW as S-GW/P-GW
    participant IMS

    UE ->> EUTRAN: 1. Measurement Reports
    EUTRAN ->> MME: 2+3. HO Decision → Handover Required\n(Target ID, Source→Target Transparent Container\nwith "old BSS to new BSS info IE",\nSRVCC HO Indication = CS only,\nUE not available for PS at target)
    Note over MME: 4. Bearer Splitting:\nQCI=1 voice split from non-voice PS bearers
    MME ->> MSC: 5. SRVCC PS to CS Request\n(IMSI, Target ID, STN-SR, C-MSISDN,\nSource→Target Transparent Container,\nMM Context, optional priority/ARP,\n[emergency: Emergency Indication + equipment ID])
    MSC ->> TMSC: 6. Prepare HO Request\n(BSSMAP encapsulated; SAI = default configured in MSC\nidentifying source=E-UTRAN; priority if applicable)
    TMSC ->> TBSS: 7. HO Request / Acknowledge
    TMSC ->> MSC: 8. Prepare HO Response\n(Target→Source Transparent Container)
    MSC ->> MSC: 9. Establish circuit (ISUP IAM + ACM)\nbetween target MSC and MGW
    MSC ->> IMS: 10. Initiation of Session Transfer\n(ISUP IAM with STN-SR [non-emerg]\nor E-STN-SR + equipment ID [emerg])
    Note over IMS: 11. Session Transfer + update remote end:\nVoIP downlink switched to CS access leg SDP
    IMS -->> IMS: 12. Release of IMS access leg (per TS 23.237)
    MSC ->> MME: 13. SRVCC PS to CS Response\n(Target→Source Transparent Container)
    MME ->> EUTRAN: 14. Handover Command\n(Target→Source Transparent Container,\nvoice component info only)
    EUTRAN ->> UE: 15. Handover from E-UTRAN Command
    opt Secondary RAT usage reporting
        EUTRAN -->> MME: 15a. Secondary RAT data usage report
    end
    UE ->> TBSS: 16. UE tunes to GERAN
    TBSS ->> TMSC: 17. HO Detection → Handover Detection message;\nUE sends HO Complete via BSS
    SGSN ->> MME: 18. Suspend procedure (TS 23.060 §6.2.1.1.2):\nTLLI+RAI derived from GUTI; Target SGSN\nsends Suspend Notification → Source MME;\nSource MME returns Suspend Acknowledge
    TBSS ->> TMSC: 19. Handover Complete
    TMSC ->> MSC: 20. SES (Handover Complete); speech circuit connected
    TMSC ->> MSC: 21. ISUP Answer
    MSC ->> MME: 22. SRVCC PS to CS Complete Notification
    MME ->> MSC: 22. SRVCC PS to CS Complete Acknowledge
    Note over MME,SGW: 22a. Bearer deactivation:\nMME deactivates voice + GBR bearers\n(MME-initiated Dedicated Bearer Deactivation)\nPS-to-CS HO indicator → P-GW for voice bearer\nNon-GBR bearers: Suspend Notification → S-GW\nS-GW releases S1-U bearers; marks UE suspended
    MME ->> EUTRAN: 22b. Release resources (S1 signalling + UE resources)\nSource eNB releases resources
    opt Non-emergency: TMSI reallocation
        MSC ->> UE: 23a. TMSI Reallocation (via target MSC,\nif IMSI authenticated but unknown in VLR)
        MSC ->> IMS: 23b. MAP Update Location → HSS/HLR\n(not UE-initiated; Update Location not\nperformed for emergency sessions)
    end
    opt Emergency session location continuity
        MME ->> MME: 24. Subscriber Location Report\n(MSC Server identity → GMLC\nat source or target side per TS 23.271)
    end
```

**Step 10 (Session Transfer) detail**:
- **Non-emergency**: MSC Server sends ISUP IAM with STN-SR to IMS. If SRVCC with priority → IMS handles with priority. Priority in SIP Session Transfer = mapped from ARP in step 5. IMS priority indicator should equal original IMS session priority over PS.
- **Emergency**: MSC Server uses locally configured E-STN-SR and includes equipment identifier. IMS Service Continuity or Emergency IMS Service Continuity per TS 23.237.
- Step 10 can begin as soon as step 8 (Prepare HO Response) is received — does not wait for step 9.
- Steps 11 and 12 are independent of step 13.

> **CAMEL caveat (NOTE 2 in §6.2.2.2)**: If the MSC Server uses an ISUP interface for non-emergency sessions, session transfer initiation may fail if the subscriber profile includes CAMEL triggers that are not available prior to handover (see TS 23.292 §7.3.2.1.3). When CAMEL triggers are available and a local anchor transfer function is used (see TS 23.237), CAMEL triggers other than those in TS 23.292 §7.3.2.1.3 are not used during the transfer.

---

### §6.2.2.1A — SRVCC with DTM/no DTM HO support or UTRAN without PS HO

Identical to §6.2.2.1 **except**:
- **Step 18 (Suspend) is NOT performed** (SGSN only deactivates bearers for voice).
- Source MME **only deactivates voice bearers** (not all GBR bearers) at step 22a.
- At end of procedure, **remaining PS resources are re-established** when UE performs RAU/TAU.

---

### §6.2.2.2 — SRVCC from E-UTRAN to UTRAN/GERAN with PS HO + DTM HO support

The CS and PS handover paths proceed **in parallel**. Both paths are triggered simultaneously by the source MME.

```mermaid
sequenceDiagram
    participant UE
    participant EUTRAN as Source E-UTRAN
    participant MME as Source MME
    participant MSC as MSC Server/MGW
    participant TMSC as Target MSC
    participant TSGSN as Target SGSN
    participant TRNS as Target RNS/BSS
    participant SGW as S-GW/P-GW
    participant IMS

    UE ->> EUTRAN: 1. Measurement Reports
    EUTRAN ->> MME: 2+3. HO Required\n(SRVCC HO Indication = CS+PS HO)
    Note over MME: 4. Bearer Splitting: voice QCI=1 → CS path;\nall other PS bearers → PS path
    par CS path
        MME ->> MSC: 5a. SRVCC PS to CS Request
        MSC ->> TMSC: 5b. Prepare HO Request
        TRNS -->> TSGSN: 5c. Reloc/HO Request (CS)
        TRNS ->> TMSC: 7a+8a. Reloc/HO Req Ack
        TMSC ->> MSC: 8b. Prepare HO Response
        MSC ->> MSC: 8c. Establish circuit (ISUP)
    and PS path
        MME ->> TSGSN: 6a. Forward Relocation Request\n(Source→Target Transparent Container,\nMM Context, PDN Connections IE\nexcl. voice bearer; S4-SGSN variant: PDN Connections IE;\nGn/Gp variant: PDP Contexts)
        TSGSN ->> TRNS: 6b. Reloc/HO Request (PS)
        TRNS ->> TSGSN: 7a. Reloc/HO Req Ack
        TSGSN ->> MME: 7b. Forward Reloc Response\n(Target→Source Transparent Container)
    end
    MSC ->> IMS: 9. Session Transfer (STN-SR / E-STN-SR)
    Note over IMS: 10+11. Session Transfer complete;\nIMS access leg released
    MSC ->> MME: 12. SRVCC PS to CS Response\n(Target→Source Transparent Container)
    MME ->> EUTRAN: 13. Handover Command\n(synchronises both CS and PS containers;\ntarget BSS receives same CS+PS resource allocation)
    EUTRAN ->> UE: 14. HO from E-UTRAN Command
    UE ->> TRNS: 15+16. UE tunes to UTRAN/GERAN; HO Detection
    par CS completion
        TRNS ->> TMSC: 17a. Reloc/HO Complete
        TMSC ->> MSC: 17b. SES (HO Complete)
        MSC ->> MSC: 17c. ANSWER (ISUP)
        MSC ->> MME: 17d. PS to CS Complete Notification/Ack
        MME ->> SGW: 17e. Delete voice bearer (GBR deactivation)
        opt Non-emergency
            MSC ->> UE: 17f. TMSI Reallocation
            MSC ->> MSC: 17g. UpdateLoc → HSS/HLR
        end
    and PS completion
        TRNS ->> TSGSN: 18a. Reloc/HO Complete
        TSGSN ->> MME: 18b. Forward Reloc Complete
        MME ->> TSGSN: 18b. Forward Reloc Complete Ack
        TSGSN ->> SGW: 18c. Update bearer (S-GW/P-GW)
        MME ->> SGW: 18d. Delete Session Request (S-GW)
        MME ->> EUTRAN: 18e. Release Resources\n(S1 signalling + UE resources)
    end
    opt Emergency location continuity
        MME -->> MME: 19. Subscriber Location Report → GMLC
    end
```

**Notes on parallel paths (GERAN target)**:
- When target is GERAN, MME may receive two different Target-to-Source Transparent Containers: one from MSC Server ("New BSS to Old BSS Info" per TS 48.008) and one from SGSN ("Target BSS to Source BSS Transparent Container" per TS 48.018). Both must be included in the Handover Command to E-UTRAN.
- At step 6a, if target SGSN uses S4-based interaction with S-GW/P-GW: PDN Connections IE includes bearer info for all bearers except the voice bearer. If Gn/Gp: Forward Relocation Request includes PDP Contexts instead.

**Fallback to §6.2.2.1A**: If only the voice bearer relocation succeeds (PS bearers fail), the MME proceeds with step 13 after receiving SRVCC PS to CS Response from MSC Server (step 12), and both UE and MME continue as described in §6.2.2.1A.

---

### §6.2.2.3 — vSRVCC from E-UTRAN to UTRAN (with PS HO)

Structurally identical to §6.2.2.2. Key **differences** for video call continuity:

**Step 4 — Extended bearer splitting**:
- Split: QCI=1 (voice) → CS path via SRVCC
- Split: vSRVCC-marked PS bearer (video) → also CS path via vSRVCC
- Remaining non-voice/non-video PS bearers → PS path

**Step 5a — SRVCC PS to CS Request**:
- Includes **vSRVCC indicator** and received **vSRVCC flag** from HSS.

**Step 5b1 — SCC AS consultation**:
- MSC Server queries SCC AS ("last active session?") to determine if the session is voice-only or voice+video.
- SCC AS responds with "voice and video" → MSC Server proceeds with vSRVCC.

**Step 5c — BS30 bearer reservation at target RNS**:
- Target MSC requests BS30 (multimedia circuit bearer) resource at target UTRAN.
- Target RNS allocates CS radio resources for **both voice and video**.
- If BS30 reservation fails → vSRVCC abandoned (no fallback to SRVCC in this path).

**Step 9 — Session Transfer**:
- MSC Server sends STN-SR with **default predefined Codecs** (per TS 26.111) SDP for both voice and video.

**Step 10 — SCC AS vSRVCC detection**:
- SCC AS detects video SDP in the CS access leg update → performs vSRVCC HO.
- SCC AS sends remote leg update with CS SDP for voice+video. If video SDP is **missing** → SCC AS assumes SRVCC (voice only) and releases the video bearer from the remote leg.

**Step 17d1 — Bearer deactivation**:
- Source MME deactivates **both QCI=1 AND vSRVCC-marked PS bearers**.
- Sets PS-to-CS HO indicator to P-GW.

**After HO complete (step 17c)**:
- UE uses predefined Codecs initially.
- UE may start **3G-324M multimedia codec negotiation** for the CS video leg (per TS 26.267 / TR 26.911 / ITU-T H.324 Annex K).

---

### §6.2.2.4 — vSRVCC from E-UTRAN to UTRAN (no PS HO)

Same as §6.2.2.3 except **PS HO steps (6a/6b/7a/7b/18a-18e) are NOT performed**. At end of procedure, UE re-establishes PS resources by performing RAU per TS 23.060. The target SGSN may deactivate PDP Contexts that cannot be established.

---

### §6.2.3 — Returning back to E-UTRAN

Once the CS voice call terminates (UE still in CS domain), existing mechanisms per TS 23.401 and TS 23.060 move the UE back to E-UTRAN, e.g., by prioritizing E-UTRAN availability via MSC indication to GERAN/UTRAN over the RR connection release. The BSC/RNC can consider E-UTRAN availability in their cell selection logic. See also TS 23.272 for CSFB return path.

---

## Common PS Bearer Splitting Logic

The PS bearer splitting function in the MME (for E-UTRAN→UTRAN/GERAN) works as follows:

```mermaid
flowchart TD
    A[Handover Required received\nwith SRVCC HO Indication] --> B{SRVCC HO Indication type?}
    B --> |CS only\n(GERAN no DTM / UTRAN no PS HO)| C[Split voice bearer only\ntoward MSC Server via Sv]
    B --> |CS+PS HO\n(UTRAN or GERAN+DTM)| D[Split voice bearer\ntoward MSC Server via Sv\nAND split non-voice bearers\ntoward target SGSN via\nForward Relocation Request]
    C --> E{vSRVCC?}
    D --> E
    E --> |No| F[Voice: QCI=1 only]
    E --> |Yes| G[Voice: QCI=1\nVideo: vSRVCC-marked bearer\nRemainder: non-voice PS bearers]
```

**Voice bearer identification**:
- E-UTRAN path: QCI=1 is the unique identifier (guaranteed by PCRF enforcement per §5.3.8)
- The MME uses the SRVCC HO Indication from E-UTRAN and the QCI value to decide which bearer goes via Sv

**Priority detection**:
- MME detects SRVCC with priority if the ARP of the EPS bearer used for IMS signalling indicates it
- Priority indication is included in the SRVCC PS to CS Request message to MSC Server
- MSC Server maps ARP → CS priority level based on operator policy and local configuration
- IMS priority indicator in Session Transfer should match the original IMS session priority over PS

---

## Part 3: 5G-SRVCC from NG-RAN to 3GPP UTRAN (§6.5)

5G-SRVCC (Release 16) handles SRVCC for UEs on 5G NR (NG-RAN) handing over to 3GPP UTRAN CS. The unique structural element is **MME_SRVCC** — a protocol bridge between the 5GC AMF (N26) and the 3G MSC Server (Sv). The MSC Server-side PS-CS procedure is identical to E-UTRAN→UTRAN without PS HO (§6.2.2.1A).

### Enabling Procedures

| Procedure | 5G-SRVCC addition |
|---|---|
| 5G Registration (§6.5.1) | UE includes 5G-SRVCC capability in "UE Network Capability" Registration Request. UDM/HSS includes STN-SR and C-MSISDN in subscription. AMF includes "5G-SRVCC operation possible" in N2 Request |
| Service Request (§6.5.2) | AMF includes "5G-SRVCC operation possible" in N2 Request |
| Intra-5GS PS HO (§6.5.3) | Source AMF sends MS Classmark 2, STN-SR, C-MSISDN, Supported Codec IE to target AMF; target AMF includes "5G-SRVCC operation possible" in N2 HO Request |

### §6.5.4 — 5G-SRVCC from NG-RAN to 3GPP UTRAN (18 steps)

```mermaid
sequenceDiagram
    participant UE
    participant NGRAN as NG-RAN
    participant AMF
    participant SMF
    participant MMESRVCC as MME_SRVCC
    participant MSC as MSC Server/MGW
    participant TRNC as Target RNC/UTRAN
    participant IMS

    Note over UE,IMS: Step 1: UE has established PDU session for IMS voice (SCC AS-anchored)
    NGRAN ->> AMF: Step 2: Voice-triggered HO
    NGRAN ->> AMF: Step 3: HO Required\n(Target ID=UTRAN RNC-ID,\nGeneric Source→Target Transparent Container,\n5G-SRVCC HO Indication)
    AMF ->> MMESRVCC: Step 4: AMF selects MME_SRVCC\n(based on target RNC ID / operator config)
    AMF ->> MMESRVCC: Step 5: Forward Relocation Request\n(IMSI, Target ID, STN-SR, C-MSISDN,\nMM Context, Generic Source→Target Transparent Container,\n5G-SRVCC HO Indication, Supported Codec IE,\nMS ClassMark 2, Emergency Indication+Equipment ID if emergency)
    Note over MMESRVCC,IMS: Step 6: MME_SRVCC initiates PS-CS handover toward MSC Server\n(Steps 5–13 of §6.2.2.1A: CS HO preparation + IMS Session Transfer via STN-SR)
    MMESRVCC ->> AMF: Step 7: Forward Relocation Response\n(Target→Source Transparent Container)
    AMF ->> NGRAN: Step 8: HO Command
    NGRAN ->> UE: Step 9: HO Command
    UE ->> TRNC: Step 10: UE tunes to target UTRAN cell
    TRNC ->> MSC: Step 11: HO Detection at target RNS
    UE ->> TRNC: Step 12: Handover Complete via target RNS\n(MSC Server/MGW can send/receive voice data)
    MSC ->> MMESRVCC: Step 13: SRVCC PS to CS Completion
    MMESRVCC ->> AMF: Step 14: Forward Relocation Complete
    AMF ->> MMESRVCC: Step 15: Forward Relocation Complete ACK\n(AMF releases UE context related to MME_SRVCC)
    MMESRVCC ->> MSC: Step 16: PS to CS Complete ACK\n(MME_SRVCC removes stored UE context;\nMSC Server removes context related to MME_SRVCC)
    AMF ->> SMF: Step 17: PDU Session Release (all PDU sessions)\n(due to PS-to-CS HO for 5G-SRVCC per TS 23.502)
    opt Emergency session
        AMF ->> AMF: Step 18: Source AMF or MSC Server →\nSubscriber Location Report to GMLC\n(per TS 23.273/23.271)
    end
```

**Key step annotations:**

- **Step 4 (MME_SRVCC selection)**: AMF selects an MME_SRVCC that has Sv connectivity to the MSC Server serving the target RNC ID. Selection is operator-configured.
- **Step 5 (Forward Relocation Request)**: Carries the 5G-SRVCC HO Indication flag (tells MME_SRVCC this is a 5G-SRVCC, not a standard inter-RAT HO). Also carries STN-SR and C-MSISDN from UDM/HSS subscription. Emergency indication and equipment identifier are included if the session is an emergency session. Authenticated IMSI and C-MSISDN are also included if available.
- **Step 6 (MME_SRVCC reuses §6.2.2.1A)**: The MSC Server-side processing is identical to an E-UTRAN→UTRAN SRVCC without PS HO. MME_SRVCC is the protocol adapter — the MSC Server has no knowledge it is dealing with 5G access.
- **Step 17 (all PDU sessions released)**: Critical 5G-SRVCC distinction. After the UE moves to 3G CS, it cannot maintain 5G PS connectivity. All PDU sessions are released by AMF per TS 23.502.
- **IMS continuity post-HO**: Described in TS 23.292 §13; when the UE accesses a UTRAN cell due to 5G-SRVCC, IMS service continuity procedures apply.

### §6.5.5 — Emergency PDU Session for 5G-SRVCC

Emergency PDU session establishment per TS 23.502 with the following 5G-SRVCC-specific additions:
- AMF includes "5G-SRVCC operation possible" indication in the N2 Request (if not already sent).
- After the emergency PDU session is **released**, AMF **restores the SRVCC indication** to the same status as prior to emergency PDU session establishment.

---

## Cross-references

- [SRVCC concept](../concepts/SRVCC.md) — all variants, architecture principles, entity descriptions, CS-to-PS flows, HO failure, security
- [SRVCC procedures from UTRAN(HSPA)](SRVCC-from-UTRAN-HSPA.md) — §6.3 SGSN-based flows
- [SRVCC CS-to-PS and 5G-SRVCC procedures](SRVCC-CS-to-PS-and-5G.md) — §6.4–§6.5
- [MME deep-dive](../entities/MME-deepdive.md) — PS bearer splitting function, Sv interface detail
- [PCRF deep-dive](../entities/PCRF-deepdive.md) — QCI=1 enforcement, vSRVCC bearer marking
- [HSS deep-dive](../entities/HSS-deepdive.md) — STN-SR, C-MSISDN, vSRVCC flag provisioning
- Source: [TS 23.216](../sources/ts23216.md)
