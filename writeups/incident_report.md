# Incident Report — NetSupport Manager RAT Infection
**Date of Incident:** 2026-02-28  
**Report Author:** Kiante  
**Exercise Source:** malware-traffic-analysis.net  
**PCAP File:** 2026-02-28-traffic-analysis-exercise.pcap  

---

## Executive Summary

A Security Information and Event Management (SIEM) system generated signature alerts for NetSupport Manager RAT activity originating from internal host `10.2.28.88`. The C2 server at `45.131.214.85` was contacted over TCP port 443 beginning at 19:55 UTC on 2026-02-28. Packet capture analysis confirmed the infected host, identified the user account, and documented C2 beaconing behavior consistent with NetSupport Manager RAT.

---

## Environment Details

| Field | Value |
|---|---|
| LAN Segment | 10.2.28.0/24 |
| Domain | easyas123.tech |
| AD Environment | EASYAS123 |
| Domain Controller | 10.2.28.2 — EASYAS123-DC |
| Gateway | 10.2.28.1 |

---

## Victim Details

| Field | Value |
|---|---|
| IP Address | 10.2.28.88 |
| MAC Address | 00:19:d1:b2:4d:ad |
| Hostname | DESKTOP-TEYQ2NR |
| User Account | brolf |
| Full Name | Becka Rolf |

---

## Indicators of Compromise (IOCs)

| Type | Value | Notes |
|---|---|---|
| C2 IP | 45.131.214.85 | NetSupport Manager RAT C2 server |
| C2 Port | 443 | TCP — disguised as HTTPS |
| HTTP Endpoint | /fakeu... | RAT beaconing POST endpoint |
| User-Agent | NetSupport RAT default | Malware-specific user agent |
| Internal IP | 10.2.28.88 | Infected Windows host |
| MAC Address | 00:19:d1:b2:4d:ad | Intel NIC on infected host |

---

## Timeline of Events

| Time (UTC) | Event |
|---|---|
| 19:55:00 | Infected host initiates TCP SYN to 45.131.214.85:443 |
| 19:55:00 | TCP handshake completed — C2 connection established |
| 19:55:00 | HTTP POST beaconing begins to /fakeu endpoint |
| 19:55:45 | Regular beaconing continues at consistent intervals |
| Ongoing | NetSupport RAT maintains persistent C2 channel |

---

## Analysis Methodology

### Step 1 — C2 Traffic Isolation
Applied Wireshark display filter `ip.addr == 45.131.214.85` to isolate all traffic between the internal network and the known C2 server. This immediately identified `10.2.28.88` as the infected host based on it being the only internal IP communicating with the C2.

**Finding:** 550 packets exchanged between `10.2.28.88` and `45.131.214.85`, including HTTP POST requests to a `/fakeu` endpoint — consistent with NetSupport RAT beaconing behavior.

### Step 2 — Host Identification via NBNS
Applied filter `nbns` to capture NetBIOS Name Service registration packets. The infected host broadcast its name to the network segment.

**Finding:** Hostname `DESKTOP-TEYQ2NR` identified from NBNS registration packets. MAC address `00:19:d1:b2:4d:ad` confirmed from Ethernet frame headers.

### Step 3 — User Account via Kerberos
Applied filter `kerberos.CNameString` to extract Active Directory authentication data. Expanded the AS-REQ packet details to locate the CNameString field.

**Finding:** User account `brolf` identified from Kerberos AS-REQ `cname` field authenticating against the `EASYAS123` domain.

### Step 4 — Full Name via SAMR
Used Edit → Find Packet with string search for `Rolf` across packet details. Located a SAMR QueryUserInfo response from the domain controller.

**Finding:** Full name `Becka Rolf` found in SAMR `QueryUserInfo` response — field: `samr.samr_UserInfo21.full_name`.

---

## Beaconing Analysis

The infected host exhibited classic RAT beaconing behavior:
- Regular interval HTTP POST requests to `45.131.214.85`
- Traffic tunneled over port 443 to blend with HTTPS traffic
- Lightweight packets consistent with C2 check-in rather than data exfiltration
- Connection maintained persistently throughout the capture window

---

## Recommended Remediation

1. **Isolate** — Immediately remove `DESKTOP-TEYQ2NR` (10.2.28.88) from the network
2. **Block** — Add `45.131.214.85` to firewall blocklist at perimeter and endpoint level
3. **Investigate** — Review Becka Rolf's recent activity and email for initial infection vector (phishing likely)
4. **Reimage** — Wipe and reimage the infected host — do not attempt to clean in place
5. **Credential Reset** — Reset `brolf` account credentials and review for lateral movement
6. **Scan** — Run EDR scan across `10.2.28.0/24` segment for additional infections
7. **Review Logs** — Pull SIEM logs for `45.131.214.85` across all internal hosts for the past 30 days

---

## Tools Used

- Wireshark 4.4.14
- Parrot OS (analysis environment)
- malware-traffic-analysis.net (PCAP source)
- VirusTotal / AbuseIPDB (IOC validation)

---

## Skills Demonstrated

- Network traffic capture and analysis
- Malware C2 traffic identification
- Wireshark display filter proficiency
- NBNS, Kerberos, and SAMR protocol analysis
- Incident report documentation
- IOC extraction and documentation
