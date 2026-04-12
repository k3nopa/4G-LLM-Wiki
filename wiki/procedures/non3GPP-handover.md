---
title: "Non-3GPP ↔ 3GPP Handover Without Optimization (S2b/S2a → E-UTRAN)"
type: procedure
tags: [non-3GPP, handover, ePDG, S2b, S2a, E-UTRAN, UTRAN, GERAN, PMIP, GTP, PGW-reuse, MME, SGW, HSS, TS-23.402]
sources: [ts_123402v150300p.pdf]
updated: 2026-04-10
---

# Non-3GPP ↔ 3GPP Handover Without Optimization

**Spec reference:** 3GPP TS 23.402 §8 (v15.3.0)

Related pages: [ePDG](../entities/ePDG.md) · [PGW](../entities/PGW.md) · [SGW](../entities/SGW.md) ·
[MME](../entities/MME.md) · [HSS](../entities/HSS.md) ·
[Non-3GPP Access Architecture](../concepts/non-3GPP-access-architecture.md) ·
[S2b Attach/Detach](S2b-attach.md) · [EPS Attach](EPS-attach.md)

---

## Overview (§8.1)

"Handover without optimization" means the UE transfers from non-3GPP access (untrusted S2b or
trusted S2a via ePDG/TWAN) to 3GPP E-UTRAN (or UTRAN/GERAN) — or vice versa — without
using handover preparation signaling between the two access networks. The key goals are:

1. **IP address continuity** — UE retains same PDN IP address(es) by reusing the same PGW
2. **No simultaneous multi-access** — the UE transitions cleanly; both accesses are not active at
   the same time during the handover
3. **Charging continuity** — same Charging ID preserved across access change

### Multi-PDN Handover Rules

- A UE may have multiple PDN connections active over non-3GPP access
- When handing over to 3GPP E-UTRAN, **only one PDN connection per APN** can be handled by
  the Attach(Handover) procedure
- The UE must establish connectivity for remaining PDN connections after the Attach procedure
  using UE-requested PDN Connectivity (§5.10)
- For 3GPP → non-3GPP: the UE establishes non-3GPP connectivity per-PDN; HSS provides
  existing PGW identity to ensure the same PGW is reused

### Direction Asymmetry

| Direction | Mechanism |
|---|---|
| Non-3GPP → 3GPP E-UTRAN | Attach with "Handover" indication; MME coordinates PGW reuse via HSS |
| 3GPP → Non-3GPP | ePDG/TWAN establishes S2b/S2a bearing to existing PGW; HSS returns PGW identity |

---

## Non-3GPP → E-UTRAN Handover, GTP S5/S8 (§8.2.1.1)

### 18-Step Procedure

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant SGW
    participant PGW
    participant PCRF
    participant HSS
    participant ePDG

    Note over UE,ePDG: UE has active PDN connection via untrusted non-3GPP (ePDG S2b)

    UE->>eNB: Step 1: UE discovers E-UTRAN (cell search)
    UE->>MME: Step 2: Attach Request (IMSI/GUTI, PDN addr preservation, Handover indication, APN)
    MME->>HSS: Step 3: Authentication + Location Update\n(Update Location Request)
    HSS-->>MME: Step 4: Update Location Answer\n(subscription data + PGW identity for active PDN)
    Note over MME,HSS: HSS returns identity of PGW currently serving the non-3GPP session

    MME->>SGW: Step 5: Create Session Request\n(IMSI, APN, PDN type, Handover Indication,\nSGW TEID for C-plane, EPS Bearer QoS)
    SGW->>PGW: Step 6: Create Session Request\n(IMSI, APN, RAT Type=E-UTRAN,\nHandover Indication, SGW address/TEID,\nEPS Bearer QoS, APN-AMBR)
    Note over PGW: Handover Indication: PGW does NOT assign new IP — reuses existing PDN addr
    PGW->>PCRF: Step 7 (opt): PCEF-Initiated IP-CAN Session Modification\n(access type change: non-3GPP→E-UTRAN)
    PCRF-->>PGW: Updated PCC rules for E-UTRAN
    PGW-->>SGW: Step 8: Create Session Response\n(PGW TEID for C/U plane, PDN addr (same),\nEPS Bearer QoS, APN-AMBR, Charging ID (same))
    SGW-->>MME: Step 9: Create Session Response\n(SGW TEID for U-plane, EPS Bearer QoS)

    MME->>eNB: Step 10: Initial Context Setup Request\n(EPS Bearer setup, SGW address/TEID)
    eNB-->>UE: Step 11: Attach Accept + Activate Default Bearer Request
    UE-->>eNB: Step 12: Attach Complete + Activate Default Bearer Accept
    eNB-->>MME: Step 13: Initial Context Setup Response\n(eNB address/TEID for U-plane)

    MME->>SGW: Step 14: Modify Bearer Request\n(eNB address/TEID, ARP, Handover Indication)
    SGW->>PGW: Step 15: Modify Bearer Request\n(SGW address/TEID, Handover Indication)
    Note over PGW: Step 15: PGW switches downlink tunnel from ePDG to SGW
    Note over PGW: PGW may apply deferred PCC rule changes from step 7
    PGW-->>SGW: Step 16: Modify Bearer Response
    SGW-->>MME: Step 17: Modify Bearer Response

    Note over UE,PGW: UE now sends/receives data over E-UTRAN (S1 + S5 bearers)
    Note over UE,ePDG: Step 18: PGW-initiated resource deactivation on ePDG side (§7.9 GTP)
    PGW->>ePDG: Delete Bearer Request (Linked EPS Bearer ID)
    ePDG->>UE: IKEv2 INFORMATIONAL (release)
    ePDG-->>PGW: Delete Bearer Response
```

### Key Step Details

**Step 2 — Attach with Handover Indication:**
- UE indicates this is a handover (not fresh attach) via "active flag" and handover indication
- UE includes PDN address from current non-3GPP session (for verification)
- UE indicates the APN for which it wants to hand over

**Step 4 — HSS Returns Existing PGW Identity:**
- The HSS subscription context contains the PGW identity (FQDN or IP) of the PGW currently
  serving the UE via non-3GPP
- MME uses this to select the **same PGW** — the critical step for IP continuity
- If HSS has no PGW identity (e.g. UE had no non-3GPP session), normal attach proceeds

**Step 6 — Create Session Request with Handover Indication:**
- `Handover Indication` flag tells PGW: "do not allocate new IP address"
- PGW recognizes existing PDN context (same IMSI + APN) and creates a new GTP session reusing
  the existing PDN IP address and Charging ID
- `RAT Type = E-UTRAN` signals the access type change for charging/policy

**Step 7 — Optional PCEF-Initiated IP-CAN Session Modification:**
- PGW notifies PCRF of access type change (non-3GPP → E-UTRAN)
- PCRF may apply different QoS/PCC rules for LTE vs WLAN
- Any new PCC rules are "deferred" by PGW until step 15 (after tunnel switch)

**Step 15 — Tunnel Switch (Critical):**
- After receiving Modify Bearer Request with eNB address, PGW switches the **downlink data
  path** from ePDG tunnel (S2b) to SGW tunnel (S5)
- This is the moment the handover occurs — data starts flowing via 3GPP
- Deferred PCC rule changes (from step 7) are applied here

**Step 18 — Non-3GPP Resource Deactivation:**
- After tunnel switch, the ePDG-PGW S2b bearer is now redundant
- PGW initiates resource deactivation toward ePDG (§7.9, Delete Bearer or Binding Revocation)
- ePDG releases the IKEv2/IPsec tunnel with UE
- UE need not perform explicit ePDG detach — it is handled by PGW initiation

---

## PMIP S5/S8 Variant (§8.2.1.2)

When S5/S8 uses **PMIPv6** instead of GTP, steps 6–9 are replaced:

### Alternative A (Standard)

```mermaid
sequenceDiagram
    participant MME
    participant SGW
    participant PGW
    participant PCRF

    MME->>SGW: GW Control Session Establishment Request\n(IMSI, APN, EPS Bearer QoS, Handover Indication)
    SGW->>PGW: Proxy Binding Update\n(MN NAI, APN, Access Type=E-UTRAN,\nHandover Indicator, APN-AMBR)
    PGW->>PCRF: IP-CAN Session Modification (access change)
    PCRF-->>PGW: Updated PCC rules
    PGW-->>SGW: Proxy Binding Ack\n(PDN addr (same), GRE key, Charging ID (same))
    SGW-->>MME: GW Control Session Establishment Response
```

### Alternative B (Lower Jitter)

Alt B reorders steps to minimize the time between PBU and data switchover:
- SGW sends PBU immediately (before PCEF modification)
- PGW receives PBU → sends PBA (binding established, data can flow)
- PCEF IP-CAN Modification follows after the PBA

> **Why Alt B:** In high-speed mobility scenarios, the gap between PBU and PBA must be
> minimized. Alt B avoids waiting for PCRF round-trip before establishing the new binding,
> reducing packet loss during handover.

---

## UTRAN / GERAN Variant (§8.2.1.3)

When the 3GPP target is **UTRAN (3G)** or **GERAN (2G)** rather than E-UTRAN:

| E-UTRAN step | UTRAN/GERAN equivalent |
|---|---|
| MME | SGSN (3G/2G) |
| Attach Request (NAS) | Attach Request (GPRS) |
| Initial Context Setup (S1-AP) | Activate PDP Context (Iu/Gb) |
| S5 GTP session | Gn/S4 GTP session |

**Key difference:** PDP Context Activation replaces EPS Bearer setup. SGSN sends
`Create Session Request` (S4-SGSN) or `Create PDP Context Request` (Gn-SGSN) to SGW/PGW
with the same `Handover Indication` flag to trigger PGW reuse.

---

## Remaining PDN Connections After Handover

After the initial Attach(Handover) for one PDN connection, the UE must reconnect any
remaining PDN connections that were active on the non-3GPP side:

1. UE-requested PDN Connectivity (TS 23.401 §5.10) for each additional APN
2. Each PDN connectivity request also uses Handover Indication
3. MME/HSS coordinates to reuse the same PGW for each APN
4. PGW deactivates the corresponding non-3GPP (ePDG/S2b) session for each APN after the
   new 3GPP session is established

> **Constraint:** If UE had two PDN connections to the same APN (different PDN GWs), the
> 3GPP side can only accommodate one at a time. The second would require a fresh PDN
> connectivity without handover indication.

---

## IP Continuity Guarantee

The mechanism ensuring IP address continuity across non-3GPP → 3GPP handover:

```mermaid
flowchart TD
    A[HSS stores: UE → PGW identity per APN] -->|UpdateLocation step 4| B[MME receives PGW identity]
    B --> C[MME routes Create Session Request to same PGW]
    C --> D[PGW recognizes Handover Indication]
    D --> E[PGW creates new GTP binding WITHOUT allocating new IP]
    E --> F[UE retains same PDN IP address]
    F --> G[Applications unaware of access change]
```

The HSS is the authoritative record keeper of which PGW serves which APN for a given UE.
This record is written by the PGW (via AAA/HSS update) at initial attach and updated at
each handover, PDN connect/disconnect event.
