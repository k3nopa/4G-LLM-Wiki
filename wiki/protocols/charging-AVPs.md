---
title: "3GPP Charging AVP Reference (Selective)"
type: protocol
tags: [charging, Diameter, AVP, Rf, Ro, offline-charging, online-charging, MSCC, IMS-Information, PS-Information, ICID]
sources: [ts32299.md]
updated: 2026-04-12
---

# 3GPP Charging AVP Reference (Selective)

Source: 3GPP TS 32.299 v16.2.0 §7 — "Summary of used Attribute Value Pairs"

This page covers the key AVPs for both offline (Rf/ACR–ACA) and online (Ro/CCR–CCA) charging. The full dictionary contains ~250 3GPP-specific AVPs; only those relevant to 4G EPC and IMS charging are described here. For protocol flows and message formats see [Rf-offline-charging.md](Rf-offline-charging.md) and [Ro-online-charging.md](Ro-online-charging.md).

Vendor-Id for all 3GPP-specific AVPs: **10415**.

---

## 1. IETF Diameter AVPs Reused by 3GPP (§7.1)

Table 7.1.0.1 lists the standard IETF Diameter AVPs that 3GPP uses in ACR/ACA (offline) and CCR/CCA (online) messages. AVPs marked "M" (mandatory) must be supported; "Oc/Om" are conditionally/optionally present.

### Key IETF AVPs

| AVP Name | Code | Type | Used in | Notes |
|---|---|---|---|---|
| Accounting-Input-Octets | 363 | Unsigned64 | ACR | Uplink octets for the data container recording interval |
| Accounting-Output-Octets | 364 | Unsigned64 | ACR | Downlink octets for the data container recording interval |
| Acct-Application-Id | 259 | Unsigned32 | ACR/ACA | Value = **3** (per RFC 6733 via TS 29.230) |
| Auth-Application-Id | 258 | Unsigned32 | CCR/CCA | Value = **4** (per RFC 4006 via TS 29.230) |
| Called-Station-Id | 30 | UTF8String | CCR | Carries the **APN** the UE is connected to |
| Event-Timestamp | 55 | Time | ACR/CCR | Time when the chargeable event is received at the CTF |
| Multiple-Services-Credit-Control | 456 | Grouped | CCR/CCA | Core online charging container — see §7.1.9 below |
| Rating-Group | 432 | Unsigned32 | CCR/CCA | Charging key per TS 23.203; unique per Diameter CC session |
| Result-Code | 268 | Unsigned32 | ACA/CCA | Diameter result; 3GPP adds new values — see §7.1.11 below |
| Experimental-Result | 297 | Grouped | ACA/CCA | Vendor-Id=10415; 3GPP permanent failure 5011 = FEATURE_UNSUPPORTED |
| Service-Context-Id | 461 | UTF8String | CCR (M) | Identifies middle-tier TS — see §7.1.12 below |
| Service-Identifier | 439 | Unsigned32 | CCR/CCA | Service sub-key within a Rating-Group per middle-tier TS |
| Used-Service-Unit | 446 | Grouped | CCR | Consumed units reported to OCF — see §7.1.14 below |
| User-Name | 1 | UTF8String | CCR | NAI format per RFC 6733 |
| Vendor-Id | 266 | Unsigned32 | — | 10415 = 3GPP (IANA-registered) |
| User-Equipment-Info | 458 | Grouped | CCR/ACA | {Type, Value}: IMEISV as hex (PS) or decimal (IMS) |
| Session-Id | 263 | UTF8String | All | Diameter session correlation (M in all messages) |
| Subscription-Id | 443 | Grouped | CCR/ACR | {Type, Data}: Type 0=MSISDN, 1=IMSI |
| Termination-Cause | 295 | Enumerated | CCR | Reason for CCR[TERMINATION] |
| Validity-Time | 448 | Unsigned32 | CCA | OCF-set quota expiry time (seconds) |

---

### §7.1.9 Multiple-Services-Credit-Control (MSCC) AVP — Code 456

The key container for per-rating-group quota management in online charging. ABNF:

```
<Multiple-Services-Credit-Control> ::=
    < AVP Header: 456 >
    [ Granted-Service-Unit ]
    [ Requested-Service-Unit ]
  * [ Used-Service-Unit ]
  * [ Service-Identifier ]
    [ Rating-Group ]
  * [ G-S-U-Pool-Reference ]
    [ Validity-Time ]
    [ Result-Code ]
    [ Final-Unit-Indication ]
    [ Time-Quota-Threshold ]
    [ Volume-Quota-Threshold ]
    [ Unit-Quota-Threshold ]
    [ Quota-Holding-Time ]
    [ Quota-Consumption-Time ]
  * [ Reporting-Reason ]
    [ Trigger ]
    [ PS-Furnish-Charging-Information ]
    [ Refund-Information ]
  * [ AF-Correlation-Information ]
  * [ Envelope ]
    [ Envelope-Reporting ]
    [ Time-Quota-Mechanism ]
  * [ Service-Specific-Info ]
    [ QoS-Information ]
  * [ Announcement-Information ]
    [ 3GPP-RAT-Type ]
    [ Related-Trigger ]
```

---

### §7.1.11 Result-Code AVP — Code 268 (3GPP Extensions)

3GPP adds the following values beyond the standard RFC 6733 set:

**Transient Failures (4xxx):**

| Code | Name | Meaning |
|---|---|---|
| 4010 | DIAMETER_END_USER_SERVICE_DENIED | OCF denies due to subscription/balance restrictions |
| 4011 | DIAMETER_CREDIT_CONTROL_NOT_APPLICABLE | Service is free or falls to offline; no CC needed |
| 4012 | DIAMETER_CREDIT_LIMIT_REACHED | Account cannot cover the requested service |

**Permanent Failures (5xxx):**

| Code | Name | Meaning |
|---|---|---|
| 5003 | DIAMETER_AUTHORIZATION_REJECTED | Terminate the specific service; may also blacklist a Rating-Group |
| 5030 | DIAMETER_USER_UNKNOWN | IMSI/MSISDN not found in OCF |
| 5031 | DIAMETER_RATING_FAILED | Rating not possible — insufficient input, unrecognized AVP combination, or unknown Rating-Group; Failed-AVP MUST be included |

---

### §7.1.12 Service-Context-Id AVP — Code 461

UTF8String format: `"extensions".MNC.MCC."Release"."service-context"@"domain"`

Key 3GPP values for the `service-context@domain` portion:

| Value | Service |
|---|---|
| `32251@3gpp.org` | PS domain charging (TS 32.251) |
| `32260@3gpp.org` | IMS charging (TS 32.260) |
| `32275@3gpp.org` | MMTel service charging (TS 32.275) |
| `32270@3gpp.org` | MMS service charging |
| `32271@3gpp.org` | LCS charging |
| `32254@3gpp.org` | Exposure Function API charging |

The `"Release"` field indicates the 3GPP release (e.g. "12" for Release 12). MNC.MCC identifies the operator's service-specific extensions. At minimum, `"Release"."service-context"@"domain"` must be present.

---

### §7.1.14 Used-Service-Unit (USU) AVP — Code 446

Reports consumed units in CCR[UPDATE/TERMINATION]. ABNF:

```
<Used-Service-Unit> ::=
    < AVP Header: 446 >
    [ Reporting-Reason ]
    [ CC-Time ]
    [ CC-Total-Octets ]
    [ CC-Input-Octets ]
    [ CC-Output-Octets ]
    [ CC-Service-Specific-Units ]
  * [ Event-Charging-TimeStamp ]
```

---

## 2. 3GPP-Specific AVPs — Session and Subscriber Identity (§7.2)

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| 3GPP-IMSI | 1 | UTF8String | IMSI of the subscriber (ACR/CCR) |
| 3GPP-Charging-Id | 2 | OctetString | Charging ID derived from GTP TEID |
| 3GPP-RAT-Type | 21 | OctetString | Radio access type (UTRAN=1, GERAN=2, WLAN=3, E-UTRAN=6, etc.) |
| 3GPP-User-Location-Info | 22 | OctetString | TAI+ECGI for E-UTRAN access |
| 3GPP-MS-TimeZone | 23 | OctetString | UE timezone at time of charging |
| 3GPP-IMSI-MCC-MNC | 8 | UTF8String | IMSI-derived MCC+MNC |
| 3GPP-SGSN-MCC-MNC | 18 | UTF8String | Serving network MCC+MNC |
| User-Session-Id | 830 | UTF8String | IMS SIP Call-ID (IMS charging correlation) |
| IMS-Charging-Identifier | 841 | UTF8String | **ICID** — globally unique IMS charging ID generated by the first IMS node touching the session (P-CSCF or AS); correlates Rf/Ro records across all CSCFs for one session |
| PDN-Connection-Charging-ID | 2050 | Unsigned32 | Charging ID for additional PDN connections |

---

## 3. 3GPP-Specific AVPs — Service Information (§7.2)

### 3.1 Service-Information AVP — Code 873

Top-level grouped AVP that wraps service-domain-specific information. Present in ACR and CCR. Defined in TS 32.299; content defined per middle-tier TS.

```
Service-Information ::= < AVP Header: 873 >
    [ PS-Information ]
    [ IMS-Information ]
    [ MMS-Information ]
    [ LCS-Information ]
    [ PoC-Information ]
    [ MBMS-Information ]
    [ MMTel-Information ]
    [ ProSe-Information ]
    [ VCS-Information ]
    [ Application-Server-Information ]
    ...
```

### 3.2 PS-Information AVP — Code 874

Carries EPS/PDP context data for PS-domain charging (TS 32.251):

```
PS-Information ::= < AVP Header: 874 >
    [ 3GPP-Charging-Id ]
    [ PDN-Connection-Charging-ID ]
    [ Node-Id ]
    [ 3GPP-PDP-Type ]
    [ PDP-Address ]
    [ PDP-Address-Prefix-Length ]
    [ Dynamic-Address-Flag ]
    [ QoS-Information ]
  * [ SGSN-Address ]
    [ GGSN-Address ]
    [ SGW-Address ]
    [ CG-Address ]
    [ 3GPP-GPRS-Negotiated-QoS-Profile ]
    [ 3GPP-SGSN-MCC-MNC ]
    [ 3GPP-GGSN-MCC-MNC ]
    [ 3GPP-NSAPI ]
    [ 3GPP-Selection-Mode ]
    [ 3GPP-Charging-Characteristics ]
  * [ 3GPP-User-Location-Info ]
    [ 3GPP-MS-TimeZone ]
    [ 3GPP-RAT-Type ]
    [ PS-Furnish-Charging-Information ]
    [ PDP-Context-Type ]
    [ Called-Station-Id ]       ← APN
    ...
```

### 3.3 IMS-Information AVP — Code 876

Carries SIP session data for IMS charging (TS 32.260):

```
IMS-Information ::= < AVP Header: 876 >
    [ Event-Type ]
    [ SIP-Method ]
    [ Role-Of-Node ]
    [ Node-Functionality ]
    [ User-Session-Id ]
    [ Outgoing-Session-Id ]
  * [ Session-Priority ]
    [ Calling-Party-Address ]
    [ Called-Party-Address ]
    [ Called-Identity-Change ]
    [ Number-Portability-Routing-Information ]
    [ Carrier-Select-Routing-Information ]
    [ Alternate-Charged-Party-Address ]
  * [ Requested-Party-Address ]
  * [ Associated-URI ]
    [ Time-Stamps ]
  * [ Application-Server-Information ]
  * [ Inter-Operator-Identifier ]
    [ IMS-Charging-Identifier ]
  * [ SDP-Session-Description ]
  * [ SDP-Media-Component ]
    [ Served-Party-IP-Address ]
    [ Server-Capabilities ]
  * [ Trunk-Group-Id ]
    [ Bearer-Service ]
  * [ Service-Id ]
  * [ Service-Specific-Info ]
    [ Message-Body ]
    [ Cause-Code ]
    [ Reason-Header ]
  * [ Access-Network-Information ]
    [ Cellular-Network-Information ]
    [ IMS-Communication-Service-Identifier ]
    [ IMS-Application-Reference-Information ]
    [ Online-Charging-Flag ]
    [ Real-Time-Tariff-Information ]
    [ Account-Expiration ]
    [ Initial-IMS-Charging-Identifier ]
  * [ NNI-Information ]
    [ From-Address ]
    [ IMS-Emergency-Indicator ]
    [ IMS-Visited-Network-Identifier ]
  * [ Access-Transfer-Information ]
    [ Related-IMS-Charging-Identifier ]
    [ Related-IMS-Charging-Identifier-Node ]
    [ Route-Header-Received ]
    [ Route-Header-Transmitted ]
  * [ Instance-Id ]
    [ Reason-Header ]
    [ VoLTE-Information ]
    ...
```

---

## 4. 3GPP-Specific AVPs — IMS Role and Routing Identity (§7.2)

| AVP Name | Code | Type | Values / Notes |
|---|---|---|---|
| Role-Of-Node | 829 | Enumerated | **0=Originating, 1=Terminating, 2=Proxy** (for PS domain); identifies the IMS node's role in the SIP session |
| Node-Functionality | 862 | Enumerated | **0=S-CSCF, 1=P-CSCF, 2=I-CSCF, 3=MRFC, 4=MGCF, 5=BGCF, 6=AS, 7=IBCF, 8=SGW, 9=PGW, 10=HSGW, 11=E-CSCF, 14=TRF, 15=TF, 16=ATCF** |
| Originating-IOI | 839 | UTF8String | Inter-Operator Identifier for the originating network side (IMS charging correlation across operators) |
| Terminating-IOI | 840 | UTF8String | IOI for the terminating network side |
| Transit-IOI-List | 2701 | Grouped | List of transit operator identifiers for transit charging |
| Calling-Party-Address | 831 | UTF8String | SIP URI / TEL-URI of the calling party |
| Called-Party-Address | 832 | UTF8String | SIP URI / TEL-URI of the called party |
| SIP-Request-Timestamp | 834 | Time | Time of the SIP request at the charging node |
| SIP-Response-Timestamp | 835 | Time | Time of the SIP response at the charging node |
| Access-Network-Information | 1263 | OctetString | Content of SIP `P-Access-Network-Info` header; access-specific encoding (GERAN=CGI, UTRAN=LAI+CI, E-UTRAN=TAI+ECGI) |

---

## 5. 3GPP-Specific AVPs — SDP / Media Components (§7.2)

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| SDP-Media-Component | 843 | Grouped | One per media line; contains SDP-Media-Name, SDP-Media-Description(s), AF-Charging-Identifier, Media-Initiator-Flag, SDP-TimeStamps, etc. |
| SDP-Session-Description | 842 | UTF8String | Session-level SDP line (one AVP per `a=`, `b=`, etc.) |
| SDP-Media-Description | 845 | UTF8String | Media-level SDP attribute line |
| SDP-TimeStamps | 1273 | Grouped | {SDP-Offer-Timestamp, SDP-Answer-Timestamp} |
| SDP-Answer-Timestamp | 1275 | Time | Time of SDP answer — used to detect media start |
| SDP-Offer-Timestamp | 1274 | Time | Time of SDP offer |
| SDP-Media-Name | 844 | UTF8String | `m=` line value (e.g. "audio 49152 RTP/AVP 98") |

---

## 6. 3GPP-Specific AVPs — Online Quota Management (§7.2)

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| Quota-Holding-Time (QHT) | 871 | Unsigned32 | Seconds; idle timeout — CTF sends CCR[UPDATE] with Reporting-Reason=QHT when no traffic for this duration |
| Quota-Consumption-Time (QCT) | 881 | Unsigned32 | Time-based parallel volume consumption window; see Quota-Consumption-Time AVP in MSCC |
| Time-Quota-Threshold | 868 | Unsigned32 | Remaining seconds before CCR[UPDATE] is triggered (pre-emptive re-authorization) |
| Volume-Quota-Threshold | 869 | Unsigned32 | Remaining octets before CCR[UPDATE] (pre-emptive re-authorization) |
| Unit-Quota-Threshold | 1226 | Unsigned32 | Remaining service-specific units before CCR[UPDATE] |
| Reporting-Reason | 872 | Enumerated | Reason for USU report in CCR[UPDATE/TERMINATION] — see table below |
| Trigger | 1264 | Grouped | `* [ Trigger-Type ]` — list of events that triggered re-authorization |
| Trigger-Type | 870 | Enumerated | Type of re-authorization trigger — see table below |
| Time-Quota-Mechanism | 1270 | Grouped | `{ Time-Quota-Type } [ Base-Time-Interval ]` — controls CTP/DTP combinational quota |
| Quota-Indicator | 3912 | Enumerated | 0=QUOTA_NEEDED, 1=QUOTA_NOT_NEEDED — online charging quota control indicator |

### Reporting-Reason Values

| Value | Name | Meaning |
|---|---|---|
| 0 | THRESHOLD | Volume/time/unit quota threshold crossed |
| 1 | QHT | Quota Holding Time expiry — no traffic |
| 2 | FINAL | Last CCR (TERMINATION) |
| 3 | QUOTA_EXHAUSTED | Granted quota fully consumed |
| 4 | VALIDITY_TIME | Validity-Time from last CCA expired |
| 5 | OTHER_QUOTA_TYPE | Reporting due to different quota type exhaustion |
| 6 | RATING_CONDITION_CHANGE | RAT-type, QoS, location etc. changed |
| 7 | FORCED_REAUTHORISATION | OCF-initiated re-auth via RAR |
| 8 | POOL_EXHAUSTED | G-S-U-Pool exhausted |
| 9 | UNUSED_QUOTA_TIMER | Unused-Quota-Timer expired |

### Trigger-Type Values (selected)

| Value | Meaning |
|---|---|
| 1 | Change in SGSN IP address |
| 2 | Change in QoS |
| 3 | Change in Location (RAI/TAI) |
| 4 | Change in RAT |
| 10 | Change in QoS Traffic Class |
| 12 | Change in Location (CGI/SAI) |
| 14 | Change in Location (ECGI/TAI) |
| 20 | Change in UE timezone |

---

## 7. 3GPP-Specific AVPs — Tariff and AoC (§7.2)

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| Tariff-Time-Change | 451 | Time | Point in time when the tariff changes (e.g. peak to off-peak); triggers new USU/GSU pair in MSCC |
| Tariff-Information | 2060 | Grouped | Detailed tariff data: [ Current-Tariff ] * [ Tariff-Time-Change ] [ Next-Tariff ] |
| Remaining-Balance | 2021 | Grouped | `{ Value-Digits } [ Exponent ]` — subscriber balance after deduction (in CCA) |
| AoC-Information | 2054 | Grouped | Advice of Charge: `[ AoC-Cost-Information ] [ Tariff-Information ] [ AoC-Subscription-Information ]` |
| AoC-Cost-Information | 2053 | Grouped | `[ Accumulated-Cost ] * [ Incremental-Cost ] [ Currency-Code ]` |
| AoC-Request-Type | 2055 | Enumerated | 0=AoC_NOT_REQUESTED, 1=AoC_FULL, 2=AoC_COST_ONLY, 3=AoC_TARIFF_ONLY |
| AoC-Service-Type | 2313 | Enumerated | 0=NONE, 1=AOC-S (service start/stop), 2=AOC-D (during), 3=AOC-E (end) |
| Offline-Charging | 1278 | Grouped | Used in CCA → instructs CTF on offline charging parameters when both online and offline are active |

---

## 8. 3GPP-Specific AVPs — SRVCC / Access Transfer (§7.2)

These AVPs support [SRVCC](../../concepts/SRVCC.md) charging correlation when a VoIP session transfers between PS and CS access:

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| Access-Transfer-Information | 2709 | Grouped | Carries access transfer data in IMS-Information; contains Access-Transfer-Type, * Access-Network-Information, Related-IMS-Charging-Identifier, Change-Time |
| Access-Transfer-Type | 2710 | Enumerated | **0=PS-to-CS, 1=CS-to-PS, 2=PS-to-PS, 3=CS-to-CS** |
| Related-IMS-Charging-Identifier | 2711 | UTF8String | ICID of the related (pre-transfer) IMS session — enables billing correlation across SRVCC |
| Related-IMS-Charging-Identifier-Node | 2712 | Address | IP address of the node that generated the related ICID |
| Access-Network-Info-Change | 4401 | Grouped | Subsequent access network changes during session: `* [ Access-Network-Information ] [ Cellular-Network-Information ] [ Change-Time ]` |

---

## 9. 3GPP-Specific AVPs — AF Correlation (§7.2)

| AVP Name | Code | Type | Notes |
|---|---|---|---|
| AF-Correlation-Information | 1276 | Grouped | Links Ro charging to the Gx/Rx PCC flow: `{ AF-Charging-Identifier } * [ Flows ]`; the ICID as known to the AF is the AF-Charging-Identifier; received by P-GW from PCRF via Gx |

---

## 10. AVP Code Range Allocation (§7.2 table NOTE)

TS 29.230 reserves the following AVP code ranges for 3GPP TS 32.299:

| Release | Code Range |
|---|---|
| Pre-Rel-8 | 800–899, 1200–1299 |
| Rel-8 | 2000–2199 |
| Rel-9 | 2300–2399 |
| Rel-10 | 2600–2699 |
| Rel-11 | 2700–2799 |
| Rel-12 | 3400–3499 |
| Rel-13 | 3900–3999 |
| Rel-14 | 4400–4499 |
| Rel-15 | 1300–1399 |

This range partitioning ensures AVP code uniqueness across releases.

---

## See Also

- [charging-architecture.md](../concepts/charging-architecture.md) — CTF/CDF/OCF/CGF roles, offline/online scenarios
- [Rf-offline-charging.md](Rf-offline-charging.md) — ACR/ACA message formats and flows
- [Ro-online-charging.md](Ro-online-charging.md) — CCR/CCA message formats, IEC/ECUR/SCUR flows, quota management
