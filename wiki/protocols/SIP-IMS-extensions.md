---
title: "IMS SIP Extensions — P-Headers, Timer Tables, Feature Tags and Feature-Capability Indicators"
type: protocol
tags: [SIP, IMS, P-headers, timers, feature-tags, Feature-Caps, ICSI, IARI, P-Charging-Vector, integrity-protected, mediasec, GRUU, reg-await-auth]
sources: [ts_124229v160600p.pdf]
updated: 2026-04-18
---

# IMS SIP Extensions — §7 Reference

**Spec**: 3GPP TS 24.229 v16.6.0 §7.2.13–§7.2.20, §7.2A, §7.7, §7.8, §7.9, §7.9A
**Related pages**: [SIP-IMS-profile.md](SIP-IMS-profile.md) (§4 General + trust domain), [IMS-SIP-registration.md](../procedures/IMS-SIP-registration.md) (registration procedures)

This page documents the SIP header field definitions, parameter extensions, timer values, media feature tags, and feature-capability indicators defined in TS 24.229 §7. It is a normative reference; for procedures using these headers see [IMS-SIP-registration.md](../procedures/IMS-SIP-registration.md) and [SIP-IMS-profile.md](SIP-IMS-profile.md).

---

## §7.2 New P-Headers Defined in TS 24.229 (§7.2.13–§7.2.20)

### Resource-Share (§7.2.13)

**Purpose**: Carries resource sharing information between P-CSCF and Application Servers, enabling media streams of multiple sessions involving the same UE to share IP-CAN resources.

**Applicability**: Within and between administrative domains (trust domain not required for traversal, but contains no user/topology-sensitive data).

**Values**:

| Value | Meaning | Inserted by |
|---|---|---|
| `supported` | Sender requests resource sharing option information for sessions involving the UE identified by `+sip.instance` | P-CSCF (in REGISTER) |
| `media-sharing` | AS determined one or more media streams can be subject to resource sharing | AS (in INVITE req/resp) |
| `no-media-sharing` | No media streams are subject to resource sharing | AS or P-CSCF (in req/resp) |

**Parameters** (with `media-sharing`):
- `origin`: `"session-initiator"` (AS serves UE sending INVITE) or `"session-receiver"` (AS serves UE receiving INVITE)
- `rules`: one rule per m-line — `new-sharing-key` (mandatory, identifies media stream), optional `existing-sharing-key-list` (keys already in use in other sessions; for forked INVITEs), `directionality` (`UL`/`DL`/`UL-DL`)
- `timestamp`: monotonically increasing counter per UE; P-CSCF uses the highest timestamp to determine which rules to apply (discards older rules); reset to 0 when no UE sessions active

**ABNF**: `resource-share = "Resource-Share" HCOLON r-s-param`; values: `r-s-supported` / `r-s-no-media-sharing` / `r-s-media-sharing` / `r-s-other`

### Service-Interact-Info (§7.2.14)

**Purpose**: Informational transport of service interaction information between Application Servers to allow service conflict avoidance. Carried only within the trust domain; removed at trust boundary; UE not expected to receive.

**Values**: `executed-service` (identity of service executed) and/or `avoid-service` (identity of conflicting service).

**ABNF**: `Service-Interact-Info = "Service-Interact-Info" HCOLON executed-service-params*(COMMA executed-service-params)`; `service-id = token / quoted-string`

### Cellular-Network-Info (§7.2.15)

**Purpose**: Used by a UE accessing IMS via a non-cellular IP-CAN (e.g., WLAN for emergency calls) to convey the cellular radio access network cell identity of the cellular network the UE most recently camped on. Enables location-based emergency routing even when UE is on WLAN.

**Applicability**: Trust domain only; removed when forwarding to untrusted domain. Present whenever `P-Access-Network-Info` is present. UE not expected to receive.

**access-type values**: `3GPP-GERAN`, `3GPP-UTRAN-FDD`, `3GPP-UTRAN-TDD`, `3GPP-E-UTRAN-FDD`, `3GPP-E-UTRAN-TDD`, `3GPP-E-UTRAN-ProSe-UNR`, `3GPP-NR-FDD`, `3GPP-NR-TDD`, `3GPP-NR-U-FDD`, `3GPP-NR-U-TDD`, `3GPP2-1X`, `3GPP2-1X-HRPD`, `3GPP2-UMB`, `3GPP2-1X-Femto`

**Cell identity parameters**:
- `cgi-3gpp`: GERAN CGI = MCC(3) + MNC(2/3) + LAC(4hex) + CI(4hex)
- `utran-cell-id-3gpp`: UTRAN/E-UTRAN/NR = MCC + MNC + TAC + ECI(7hex)
- `ci-3gpp2`: cdma2000 SID+NID+PZID+BASE_ID (14 hex chars)
- `cell-info-age`: relative time in seconds since cell info was collected by UE

### Priority-Share (§7.2.16)

**Purpose**: Instructs the access gateway (via P-CSCF) whether to share the same bearer for sessions regardless of session priority. Removes the need to set up separate bearers per priority level when acceptable.

**Values**: `"allowed"` / `"not-allowed"`

**Applicability**: Within or between administratively trusted domains. Present whenever SDP/application MIME body applicable (per RFC 3261). If AS UA uses the info, it removes the header before forwarding.

### Response-Source (§7.2.17)

**Purpose**: Identifies the SIP functional entity that generated an error response. Allows receiving entities to decide which procedure to invoke in response to a failure (e.g. restoration triggers).

**URN namespace**: `urn:3gpp:fe`

**ABNF**: `Response-Source = "Response-Source" HCOLON source-info`; `source-urn = "fe" EQUAL LAQUOT source-urn-val RAQUOT`

**fe-id values** (functional entity identifiers):

| fe-id | Entity |
|---|---|
| `ue` | UE |
| `p-cscf` | P-CSCF |
| `i-cscf` | I-CSCF |
| `s-cscf` | S-CSCF |
| `e-cscf` | E-CSCF |
| `mgcf` | MGCF |
| `bgcf` | BGCF |
| `ibcf` | IBCF |
| `trf` | TRF |
| `atcf` | ATCF |
| `agcf` | AGCF |
| `mrfc` | MRFC |
| `lrf` | LRF |
| `msc-server` | MSC Server |
| `as` | Application Server |

**fe-param values**:
- `role`: `mmtel-as` / `scc-as` / `ip-sm-gw` / `pf-mcptt-server` / `cf-mcptt-server` / `ncf-mcptt-server` / `cms` / `gms` / `tads` / `iua` / `msc-server-ics`
- `side`: `orig` / `term` / `transit`

Example: `fe=<urn:3gpp:fe:p-cscf.orig>` identifies the originating P-CSCF.

May be removed when received from outside trust domain (network policy).

### Attestation-Info (§7.2.18)

**Purpose**: Informs downstream nodes about the attestation level performed on an originating identity (STIR/SHAKEN). Used by originating network nodes (S-CSCF, AS, entry IBCF) when attesting caller identity. Informational — does not change UA behavior per RFC 3261.

**Values**: `"A"` / `"B"` / `"C"` per RFC 8588 (STIR attestation levels: full attestation / partial / gateway)

**ABNF**: `Attestation-Info = "Attestation-Info" HCOLON attestation-level / generic-param`

UE not expected to receive. Does not contain sensitive information.

### Origination-Id (§7.2.19)

**Purpose**: Carries a UUID identifying the node that attested the originating identity (companion to `Attestation-Info`). Allows downstream nodes to audit which specific network entity performed attestation.

**Format**: UUID per RFC 4122.

**ABNF**: `Origination-Id = "Origination-Id" HCOLON originator / token`; `originator = UUID`

UE not expected to receive. May be removed before transporting to untrusted entities.

### Additional-Identity (§7.2.20)

**Purpose**: Conveys an additional originating or target identity for multi-identity service. Allows a UE to request use of a secondary identity for an originating request, or allows the network to inform a terminating UE which identity was used to contact it.

- **Originating side**: UE or network entity inserts the alternative identity to be used; network can remove, ignore, or use it
- **Terminating side**: inserted to inform the terminating UE which identity was the contacted target

**Applicability**: Within trust domain; removed at trust domain boundary.

**ABNF**: `Additional-Identity = "Additional-Identity" HCOLON id-spec / token`; `id-spec = name-addr *(SEMI id-param)`

---

## §7.2A Extensions to Existing SIP Headers

### §7.2A.1 WWW-Authenticate extension (IMS AKA keying material)

New auth-param parameters added to `WWW-Authenticate` in 401 responses for REGISTER:

| Parameter | Syntax | Meaning |
|---|---|---|
| `ik` | `"ik" EQUAL LDQUOT *(HEXDIG) RDQUOT` | Integrity Key (IK) for P-CSCF IPsec SA setup |
| `ck` | `"ck" EQUAL LDQUOT *(HEXDIG) RDQUOT` | Cipher Key (CK) for P-CSCF IPsec SA setup |

**Operation**: S-CSCF appends `ik` and `ck` to `WWW-Authenticate`. P-CSCF stores both values and **removes them** before forwarding the 401 to the UE. The UE derives its own CK/IK directly from RAND using TS 33.203 procedures.

### §7.2A.2 Authorization header — `integrity-protected` parameter

New `dig-resp` parameter in the `Authorization` header field of REGISTER requests. Inserted by P-CSCF; read by S-CSCF to determine whether to challenge.

| Value | Meaning |
|---|---|
| `"yes"` | REGISTER received by P-CSCF is IPsec-SA-protected + IMS AKA |
| `"no"` | REGISTER not protected by IPsec SA + IMS AKA used (initial unprotected REGISTER) |
| `"tls-pending"` | REGISTER received over TLS; TLS session not yet bound to private identity (SIP Digest + TLS) |
| `"tls-yes"` | REGISTER received over TLS; TLS session already bound to private identity |
| `"ip-assoc-pending"` | REGISTER does not map to existing IP association but contains challenge response (SIP Digest without TLS) |
| `"ip-assoc-yes"` | REGISTER maps to an existing IP association (SIP Digest without TLS) |
| `"auth-done"` | REGISTER sent from trusted entity that has already authenticated the identities (e.g. MSC Server enhanced for ICS); S-CSCF shall skip authentication |
| `"tls-connected"` | REGISTER issued by UE over TLS session established prior to registration + IMS AKAv2 used (e.g. WebRTC over IMS per TS 24.371) |

**NOTE**: Value `"yes"` is also used when an initial REGISTER with existing IPsec SA contains an Authorization header with a challenge response — the IPsec SA itself implicitly authenticates the UE. In the TLS case, TLS alone is insufficient (value = `"tls-pending"` for initial REGISTER with challenge response).

### §7.2A.3 `tokenized-by` header field parameter

Appended by the IBCF when Topology Hiding (THIG) is active to encrypted strings within SIP header fields. Value = domain name of the network that performed the encryption.

**ABNF**: `tokenized-by-param = "tokenized-by" EQUAL hostname`

### §7.2A.4 P-Access-Network-Info (PANI) extensions

3GPP-specific extensions (Table 7.2A.4) to the `P-Access-Network-Info` header defined in RFC 7315:

| Extension parameter | Meaning |
|---|---|
| `daylight-saving-time` | Quoted string: DST adjustment ("+01", "+02") |
| `UE-local-IP-address` | UE local IP address (from IPsec outer header on S2b) |
| `UDP-source-port` | UDP source port for IKEv2 messages via ePDG |
| `TCP-source-port` | TCP source port for firewall traversal tunnel |
| `ePDG-IP-address` | ePDG IP address (IKEv2 tunnel endpoint) |
| `network-provided` | Indicates content was inserted by P-CSCF, S-CSCF, AS, MSC Server — not UE-provided |

Extended `access-class` values: `untrusted-non-3GPP-VIRTUAL-EPC`, `VIRTUAL-no-PS`, `WLAN-no-PS`, `3GPP-WLAN`, `3GPP-NR`, `3GPP-NR-U`

Extended `access-type` values: `3GPP-E-UTRAN-ProSe-UNR`, `3GPP-NR-FDD`, `3GPP-NR-TDD`, `3GPP-NR-U-FDD`, `3GPP-NR-U-TDD`, `IEEE-802.11ac`, `DVB-RCS2`

**Additional coding rules (§7.2A.4.3)**: For E-UTRAN, `utran-cell-id-3gpp` = MCC(3) + MNC(2/3) + TAC(4/6hex) + ECI(7hex). For WLAN access (IEEE-802.11 variants), `i-wlan-node-id` = AP MAC address (no delimiter). For NR, TAC = 6hex digits + NCI = 9hex digits (with optional NID for SNPN).

### §7.2A.5 P-Charging-Vector extensions

3GPP-specific extensions (Table 7.2A.5) for access-network-specific charging correlation info in `P-Charging-Vector`:

**IOI encoding (Table 7.2A.5A)**:
- Type 1 IOI string prefix: `%x54.79.70.65.20.31` = "Type 1"
- Type 3 IOI string prefix: `%x54.79.70.65.20.33` = "Type 3"
- Type 2 IOI: no string prefix; raw IOI value

**Special parameters**:
- `loopback`: Indicates loopback path (e.g. via TRF in local breakout); allows intermediate nodes like TRF to generate CDRs
- `fe-identifier`: List of IM CN functional entities (by `fe-addr`) and/or AS addresses (by `as-addr`) with `ap-id`; AS can appear multiple times with different `ap-id` values per application

**Access-network-charging-info per IP-CAN type**:

| IP-CAN | Parameters tracked |
|---|---|
| GPRS | `ggsn` (GGSN addr), `auth-token`, `pdp-sig` (yes/no), `gcid` (GPRS Charging ID, hex), `flow-id` (PDP contexts: SID+bearer pairs in SDP order) |
| EPC via WLAN | `pdg` (placeholder; no extensions in this release) |
| xDSL | `bras` (BRAS addr), `auth-token`, `dsl-bearer-sig` (yes/no), `dslcid` (DSL Charging ID, hex), `flow-id` |
| DOCSIS | `bcid` (48-char hex Billing Correlation ID; globally unique within DOCSIS cable system) |
| cdma2000 | `icn-bcp` + `itid` (itc-sig, itc-id, flow-id2) per cdma2000 bearer |
| EPS | `pdngw` (P-GW addr), `eps-sig` (yes/no), `ecid` (EPS Charging ID, hex), `flow-id` per EPS bearer |
| Ethernet | `ip-edge` (IP Edge Node address per ETSI ES 282 001) |
| Fiber | `ip-edge` (IP Edge Node address) |
| 5GS | `smf` (SMF addr), `5gscid` (5GS Charging ID, hex), `flow-id` per 5GS PDU session |

### §7.2A.6 `orig` URI parameter

URI parameter appended to the address of the S-CSCF, I-CSCF, or IBCF by Application Servers when initiating requests on behalf of a user.

**Semantics**:
- Appended to S-CSCF address → S-CSCF runs **originating** services
- Appended to I-CSCF address → I-CSCF runs **originating** procedures
- IBCF preserves `orig` in topmost Route header

**ABNF**: `orig = "orig"` (simple URI parameter token)

### §7.2A.7 Security header extensions — `mediasec` parameter

New parameter `mediasec` for `Security-Client`, `Security-Server`, and `Security-Verify` header fields (per RFC 3329). Labels a security mechanism entry as applicable to the **media plane**, not the signalling plane.

**Media plane security mechanisms** (Table 7.2A.7.2-2):

| Token | Description |
|---|---|
| `sdes-srtp` | End-to-access-edge media security using SRTP with SDES key negotiation |
| `msrp-tls` | End-to-access-edge media security for MSRP using TLS and certificate fingerprints |
| `bfcp-tls` | End-to-access-edge media security for BFCP using TLS and certificate fingerprints |
| `udptl-dtls` | End-to-access-edge media security for UDPTL using DTLS and certificate fingerprints |

Media mechanisms can be applied on-the-fly (independently of signalling mechanism choice) and can be supported independently. Any of the listed mechanisms may be applied when a media stream is started, or media may be set up without security. Example: `Security-Client: sdes-srtp;mediasec`

### §7.2A.8 IMS Communication Service Identifier (ICSI)

**Format**: URN coded as `urn:urn-7:3gpp-service.ims.icsi.<name>`

**Usage contexts**:
1. Tag-value in `g.3gpp.icsi-ref` media feature tag (§7.9.2, RFC 3840) — percent-encoded URN characters not in RFC 3840 tag-value syntax
2. Feature-capability indicator value in `+g.3gpp.icsi-ref` (§7.9A.2, RFC 6809) Feature-Caps header
3. Value of `P-Preferred-Service` or `P-Asserted-Service` headers (RFC 6050)

**Subclass relationship**: ICSI subclasses use dots. Check for parent ICSI returns true if subclass is present. E.g., checking for `urn:urn-7:3gpp-service.ims.icsi.mmtel` returns true when `urn:urn-7:3gpp-service.ims.icsi.mmtel.hd-video` is present.

**Example**: `g.3gpp.icsi-ref="urn%3Aurn-7%3A3gpp-service.ims.icsi.mmtel"` (percent-encoded in feature tag context)

### §7.2A.9 IMS Application Reference Identifier (IARI)

**Format**: URN coded as `urn:urn-7:3gpp-application.ims.iari.<name>`

**Usage**: Tag-value in `g.3gpp.iari-ref` media feature tag (§7.9.3, RFC 3840).

**Example**: `g.3gpp.iari-ref="urn%3Aurn-7%3A3gpp-application.ims.iari.game-v1"`

### §7.2A.10 `phone-context` tel URI parameter coding rules

Per-IP-CAN coding rules for the `phone-context` tel URI parameter when access network info is available:

| IP-CAN | phone-context value |
|---|---|
| GPRS | `<MCC>.<MNC>.gprs.<home-domain>` |
| EPS | `<MCC>.<MNC>.eps.<home-domain>` |
| 5GS | `<MCC>.<MNC>.5gs.<home-domain>` |
| WLAN (EPC) | `<SSID>.<AP-MAC>.i-wlan.<home-domain>` or `<AP-MAC>.i-wlan.<home-domain>` |
| xDSL | `<dsl-location>.xdsl.<home-domain>` |
| Ethernet | `<eth-location>.ethernet.<home-domain>` |
| Fiber | `<fiber-location>.fiber.<home-domain>` |
| cdma2000 | `<subnet-id>.<home-domain>` |
| No access info | `geo-local.<home-domain>` |

### §7.2A.12 CPC and OLI tel URI parameters

Calling Party's Category (`cpc`) and Originating Line Information (`oli`) URI parameters for tel URIs in `P-Asserted-Identity`:

**CPC values**:

| Value | Meaning |
|---|---|
| `ordinary` | Caller identified, no special features |
| `test` | Test call (maintenance procedure) |
| `operator` | Call from operator position |
| `payphone` | Calling station is a payphone |
| `unknown` | CPC could not be ascertained |
| `mobile-hplmn` | Mobile device in home PLMN |
| `mobile-vplmn` | Mobile device in visited PLMN |
| `emergency` | Emergency service call |

**OLI**: 2-digit decimal code (North American Numbering Plan Administration). Not populated at originating UE; passed only by IM CN subsystem entities per trust domain rules.

### §7.2A.13 `sos` SIP URI parameter

URI parameter in the Contact header of a REGISTER request, indicating **emergency registration**.

**S-CSCF behavior when `sos` present**:
1. Do not apply barring to the public user identity being registered
2. Do not apply initial filter criteria to requests destined for the emergency-registered contact address
3. Preserve existing emergency contact address if new Contact URI matches a previously registered URI (no overwrite)

### §7.2A.14 P-Associated-URI extension

Extends RFC 7315 to allow a SIP proxy to **remove** URIs from the `P-Associated-URI` header field. In standard RFC 7315, proxies cannot modify this header; TS 24.229 relaxes this constraint.

### §7.2A.17 `premium-rate` tel URI parameter

Identifies numbers belonging to a premium-rate category for operator barring in roaming scenarios.

**Values**: `"information"` / `"entertainment"`

**ABNF**: `premrate = premrate-tag "=" premrate-value`; placed in the Request-URI

### §7.2A.18 Reason header extension

Extended to include additional 3GPP-specific protocol values beyond those in RFC 3326. Protocol token extensions allow 3GPP-specific cause codes in the `Reason` header (e.g., for SRVCC and restoration scenarios).

---

## §7.7 SIP Timer Values (Table 7.7.1)

Three-column timer values: between IM CN subsystem elements / at the UE / at P-CSCF toward UE.

| Timer | Between IM CN elements | At UE | P-CSCF toward UE | Meaning |
|---|---|---|---|---|
| T1 | 500ms | 2s | 2s | RTT estimate; base retransmit interval |
| T2 | 4s | 16s | 16s | Max retransmit interval for non-INVITE requests and INVITE responses |
| T4 | 5s | 17s | 17s | Max duration message remains in network |
| Timer A | T1 | T1 | T1 | INVITE request retransmit interval (UDP) |
| Timer B | 64×T1 | 64×T1 | 64×T1 | INVITE transaction timeout |
| Timer C | >3min | >3min | >3min | Proxy INVITE transaction timeout |
| Timer D | >32s UDP / 0s TCP | >128s / 0s | >128s / 0s | Wait time for response retransmits |
| Timer E | T1 | T1 | T1 | Non-INVITE request retransmit interval |
| Timer F | 64×T1 | 64×T1 | 64×T1 | Non-INVITE transaction timeout |
| Timer G | T1 | T1 | T1 | INVITE response retransmit interval |
| Timer H | 64×T1 | 64×T1 | 64×T1 | Wait time for ACK receipt |
| Timer I | T4 UDP / 0s TCP | T4 / 0s | T4 / 0s | Wait time for ACK retransmits |
| Timer J | 64×T1 UDP / 0s TCP | 64×T1 / 0s | 64×T1 / 0s | Wait time for non-INVITE request retransmits |
| Timer K | T4 UDP / 0s TCP | T4 / 0s | T4 / 0s | Wait time for response retransmits |
| Timer L | 64×T1 | 64×T1 | 64×T1 | Wait time for accepted INVITE request retransmits |
| Timer M | 64×T1 | 64×T1 | 64×T1 | Wait time for retransmission of 2xx to INVITE or additional 2xx from other INVITE branches |
| Timer N | 64×T1 | 64×T1 | 64×T1 | Wait time for receipt of NOTIFY upon sending SUBSCRIBE |

**NOTE**: T1 between IM CN elements can be extended as a network option when the MRFC and controlling AS are under the same operator and the AS knows the MRFC implements a longer T1. T2 and T4 are also modified accordingly.

**IM CN subsystem T1 = 500ms** (faster than RFC 3261 default of 500ms per spec, matched here). Timer F = 64 × 500ms = 32s, but IM CN subsystem value is **128 seconds** (= 64 × 2s) when referenced in the reg-await-auth timer context — because reg-await-auth worst case = 2 × Timer F where Timer F at IM CN subsystem level uses the 64×T1 value applied at UE (2s × 64 = 128s).

---

## §7.8 IM CN Subsystem Timers (Table 7.8.1)

| Timer | UE | P-CSCF | S-CSCF | Meaning |
|---|---|---|---|---|
| `reg-await-auth` | N/A | N/A | **4 minutes** | Guards receipt of next REGISTER during authentication. Started when S-CSCF sends 401. Worst case = 2 × Timer F = 2 × 128s = 256s < 4min. UE and P-CSCF use this value to set the SIP-level lifetime of temporary IPsec SAs. |
| `request-await-auth` | N/A | N/A | **4 minutes** | Same as reg-await-auth but for non-REGISTER requests requiring authentication (§5.4.3.6.1). |
| `emerg-reg` | 8–20s (configurable) | N/A | N/A | Supervises time from deciding to establish emergency session via IM CN to completion of emergency registration including IP-CAN procedures (§5.1.6.1). |
| `emerg-request` | 5–15s (configurable) | N/A | N/A | Supervises time for initial emergency service request (§5.1.6.8.1). |
| `NoVoPS-dereg` | 0–65535s (configurable) | N/A | N/A | Time UE waits before deregistering from IMS when VoPS-not-supported indication received from lower layers (Annexes B.3.1.0a, L.3.1.0a, U.3.1.0a, W.3.1.0a). |
| `emerg-non3gpp` | 5–20s (configurable) | N/A | N/A | Supervises time searching for usable 3GPP access to set up emergency call before attempting via non-3GPP access (Annexes R.2.2.6.1, W.2.2.6.1). |

---

## §7.9 Media Feature Tags (RFC 3840)

Media feature tags used in `Contact` and `Accept-Contact` headers for UE capability negotiation and forking.

| Tag | ASN.1 OID | Values | Purpose |
|---|---|---|---|
| `g.3gpp.icsi-ref` | 1.3.6.1.8.2.4 | ICSI URN (token, equality) | Indicates ICSI values supported by the agent. Used for forking to UE/application supporting a particular IMS communication service. Multiple values allowed. |
| `g.3gpp.iari-ref` | 1.3.6.1.8.2.5 | IARI URN (token, equality) | Indicates IARI values supported by the agent. Used for forking to UE/application supporting a particular IMS application. Multiple values allowed. |
| `g.3gpp.registration-token` | 1.3.6.1.8.2.27 | Quoted string (equality) | Unique token identifying the registration. Included by S-CSCF in third-party REGISTER. AS uses token to correlate originating requests and terminating responses with the correct registration. Token is unique per URI. |
| `g.3gpp.ps-data-off` | 1.3.6.1.8.2.35 | `"active"` / `"inactive"` | PS data off status reported by UE in Contact of REGISTER. `"active"` = PS data off activated by user; `"inactive"` = deactivated. |
| `g.3gpp.rlos` | (pending IANA) | Boolean | UE indicates support for Restricted Local Operator Service (RLOS) in REGISTER. |

---

## §7.9A Feature-Capability Indicators (Feature-Caps header, RFC 6809)

Feature-capability indicators used in the `Feature-Caps` header field for network entity capability advertisement (not negotiated, unlike media feature tags).

| Indicator | Value syntax | Purpose and context |
|---|---|---|
| `+g.3gpp.icsi-ref` | ICSI URN (fcap-value-list, one or more tokens) | ICSI values supported by the entity in standalone transactions or dialogs. S-CSCF includes in 200 OK to REGISTER to signal supported ICSIs from the user's service profile. |
| `+g.3gpp.trf` | `"<" SIP-URI ">"` | SIP URI of the TRF. Included in INVITE Feature-Caps in roaming scenarios to signal visited network supports Transit and Roaming Functionality (local breakout). |
| `+g.3gpp.loopback` | `"<" 1*(qdtext / quoted-pair) ">"` | Home network signals that an INVITE request is a loopback request to the visited network (for local breakout architecture). |
| `+g.3gpp.home-visited` | `"<" 1*(qdtext / quoted-pair) ">"` | Home network indicates it supports loopback to the identified visited network. Value = visited network identifier (same format as P-Visited-Network-ID). |
| `+g.3gpp.mrb` | `"<" SIP-URI ">"` | SIP URI of the visited network MRB. Signals MRB availability for multimedia resource allocation in roaming scenarios. |
| `+g.3gpp.registration-token` | Quoted string | Same as g.3gpp.registration-token media feature tag but in Feature-Caps. Identifies registration for current request/response in Feature-Caps; used by AS to identify registration. |
| `+g.3gpp.thig-path` | `"<" SIP-URI ">"` | SIP URI of the visited network IBCF that applied topology hiding to the Path header. Included in 200 OK to REGISTER to pass to P-CSCF the URI of the IBCF that performed THIG. |
| `+g.3gpp.priority-share` | fcap-value-list (one or more tokens) | Indicates priority sharing support. |
| `+g.3gpp.verstat` | N/A (no value) | Home network supports calling party number verification with signature and attestation (RFC 8224 / STIR). Included in 200 OK to REGISTER. |
| `+g.3gpp.anbr` | N/A (no value) | Network supports ANBR per TS 26.114. Included in 200 OK to REGISTER. |

### IMS-ALG Media Capability Indicators (Table 7.9A.4)

Included by P-CSCF in REGISTER and 200 OK to REGISTER to indicate which media types the IMS-ALG (P-CSCF media proxy) supports. If **none** of these indicators are present, it is assumed all media types are supported.

| Indicator | Meaning |
|---|---|
| `sip.audio` | IMS-ALG supports audio as streaming media |
| `sip.application` | IMS-ALG supports application as streaming media |
| `sip.data` | IMS-ALG supports data as streaming media |
| `sip.video` | IMS-ALG supports video as streaming media |
| `sip.control` | IMS-ALG supports control as streaming media |
| `sip.text` | IMS-ALG supports text as streaming media |
| `sip.message` | IMS-ALG supports message as streaming media |
| `sip.ice` | IMS-ALG supports ICE |
| `sip.app-subtype` | IMS-ALG supports MIME application subtypes for streaming media |
| `sip.fax` | IMS-ALG supports T.38 fax protocol or passthrough |

---

## §7.10 Reg-Event Package Extensions

### §7.10.2 Wildcarded Public User Identity transport

XML extension to the reg event package (RFC 3680) `<registration>` element: adds `<wildcardedIdentity>` sub-element (namespace `urn:3gpp:ns:extRegExp:1.0`). Used by S-CSCF in NOTIFY to convey the wildcard pattern of a wildcarded IMPU alongside a specific representative IMPU in the `aor` attribute.

### §7.10.3 Policy transport extension

XML extension to reg event package for transporting P-CSCF policies (namespace `urn:3gpp:ns:extRegInfo:1.0`):
- `<rph>` child of `<actions>`: RFC 4412 resource priority namespace (`ns`) + allowed value (`val`) — defines permitted priority level per UE
- `<privSender>`: P-CSCF acts as privileged sender (passes UE identity for all calls)
- `<pni>`: P-Private-Network-Indication handling — `fwd` (forward if received) or `ins` (insert in all requests); `domain` attribute = PNI domain URI
- `<privSenderPNI>`: P-CSCF allows UE to make private calls, passes identities only for private network traffic

---

## §7.11 URNs Defined in TS 24.229

### Country-specific emergency service URN

Format: `urn:service:sos.country-specific.<ISO-3166-1-alpha-2>.<national-code>`

- Used when an IANA-registered service URN (RFC 5031) is not available for a specific national emergency service type
- Example: `urn:service:sos.country-specific.xy.567` (emergency number 567 in country "xy")
- Usable wherever `urn:service:sos.*` URNs are allowed

### ICSI value for RLOS

`urn:urn-7:3gpp-service.ims.icsi.rlos` — device supports Restricted Local Operator Service.

---

## §7.12 Info Packages and MIME Types

### DTMF info package (`infoDtmf`)

- **MIME type**: `application/session-info`
- **Content-Disposition**: `signal; handling=optional`
- **Usage**: SIP INFO method within an established dialog; transports DTMF digits as `SubsequentDigit: HCOLON phonedigits` (hex digits + `*` + `#`)
- **Dual use**: Also used for overlap dialling (additional digits not previously sent)

### g.3gpp.current-location-discovery info package

- **Info package name**: `g.3gpp.current-location-discovery`
- **MIME type**: `application/vnd.3gpp.current-location-discovery+xml`
- **Content-Disposition**: `info-package`
- **Purpose**: Allows one UA participating in an INVITE dialog to request current location information from the other UA via SIP INFO (used in emergency sessions for location updates during the dialog). Authorization by the user is required before the UA provides location information (unless emergency session).
