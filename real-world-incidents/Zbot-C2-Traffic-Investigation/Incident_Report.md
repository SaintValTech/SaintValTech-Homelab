# Cybersecurity Incident Report — Malware Detection & Network Traffic Analysis

**Author:** Emeka Valentine Ogbu (SaintValTech)  
**Date of Investigation:** October 15–18, 2025  
**Environment:** Security Onion (Suricata, Strelka), pfSense Firewall, UDM Pro, VMware-based SOC Lab

---

## Executive Summary

Between October 15–18, 2025, Security Onion detected high-severity malware-related alerts from Suricata IDS indicating potential Trojan/Bifrose (Zbot/Fragtor) activity. Alerts pointed to outbound HTTP traffic from a seemingly internal IP (`192.168.3.65`) to external IPs, including `188.72.243.72` (Webzilla B.V., Netherlands).

Strelka extracted the suspicious executable:


VirusTotal confirmed it as a Trojan/Backdoor (Bifrose/Zbot) with **65/72 AV engines** detecting malicious behavior.

Wireshark PCAP analysis of the mirrored traffic identified additional HTTP requests to external web hosts and tracking domains, but no malicious executable downloads or internal device compromise was observed.

The investigation concluded that the detected traffic originated from SPAN-mirrored WAN traffic, not an internal infection. Security Onion successfully identified the malicious file, confirming the SOC’s detection capability.

---

## Detection Details

| Field                | Value                                                                 |
|---------------------|-----------------------------------------------------------------------|
| Alert Source         | Suricata (Security Onion)                                             |
| Rule Triggered       | ET MALWARE Zbot POST Request to C2                                     |
| Severity             | High                                                                   |
| Category             | Malware Command and Control Activity                                   |
| Source IP            | 192.168.3.65 (unassigned internal address)                             |
| Destination IP       | 188.72.243.72 (Webzilla B.V., Netherlands)                             |
| Destination Port     | 80 (HTTP)                                                              |
| Protocol             | TCP                                                                    |
| Detected File        | `/nsm/strelka/staging/HTTP-FYpTVB33entBpgOilk-9e04a788281c727566873d9df263aec1.exe` |
| File Size            | 579.6 KB                                                               |
| SHA256               | `282f463d7cdec977d675231279365dd890f1480dafa9d88473d5d47574e007b0`    |
| VirusTotal Detection | 65/72 AV engines flagged as Trojan/Backdoor (Bifrose/Zbot)            |
| Behavioral Flags     | detect-debug-environment, persistence, spreader, overlay, UPX-packed  |
| Rule Set             | Emerging Threats (ET)                                                  |

---

## Investigation Steps

### Step 1: Initial Alert Review
- Suricata IDS flagged multiple Zbot POST requests.
- The source IP (`192.168.3.65`) was unassigned in pfSense DHCP, firewall logs, and UDM Pro device lists.

### Step 2: SPAN Port Validation
- Verified SPAN mirrors pfSense LAN trunk (all VLANs) to Security Onion bond0 interface.
- Some alerts captured mirrored WAN ingress/egress traffic or broadcast artifacts.

### Step 3: Wireshark PCAP Analysis
- Exported PCAP via SCP for inspection.
- Observed HTTP GETs to external hosts:
  - `65.61.151.116` (www.genevalab.com) — multiple image and HTML requests
  - `74.125.19.127` (www.google-analytics.com) — analytics tracking cookies (`__utma`, `__utmb`, `__utmc`, `__utmz`)
- User-Agent spoofed as MSIE 6.0 — indicative of scripted requests or legacy traffic.
- MAC addresses identified from Cisco OUI, not matching internal hosts.
- No executable downloads were seen from Wireshark traffic.

### Step 4: VirusTotal Analysis
- File hash `282f463d7cdec977d675231279365dd890f1480dafa9d88473d5d47574e007b0` flagged by 65/72 AV engines.
- File identified as Trojan/Backdoor (Bifrose/Zbot).
- Behaviors observed: persistence, spreading capability, debug-environment detection, packed with UPX.

### Step 5: Firewall Verification
- pfSense confirmed egress restrictions and RFC1918 filtering were active.
- No internal hosts initiated traffic to the malicious IPs.

---

## Indicators of Compromise (IOCs)

| IOC Type                | Value                                                               |
|-------------------------|---------------------------------------------------------------------|
| Malicious File          | HTTP-FYpTVB33entBpgOilk-9e04a788281c727566873d9df263aec1.exe       |
| SHA256                  | `282f463d7cdec977d675231279365dd890f1480dafa9d88473d5d47574e007b0` |
| Malicious Domains/IPs   | 188.72.243.72 (Webzilla B.V.)                                      |
| Suspicious External Hosts | 65.61.151.116, 74.125.19.127                                     |
| User-Agent              | MSIE 6.0 (spoofed)                                                 |

---

## Assessment and Likely Scenario
- Source IP (`192.168.3.65`) was not an active internal host.
- Alerts resulted from mirrored WAN traffic via SPAN port — no internal compromise.
- Security Onion detected the malicious file as part of lab simulations and validation.
- **Risk Assessment:** Low — detection mechanism validated.

---

## Recommended Remediation & Prevention

1. **Traffic Monitoring & Correlation**
   - Continue correlating IDS alerts with firewall and endpoint logs before declaring an infection.
2. **SPAN Configuration**
   - Regularly verify SPAN mirror points to avoid false positives.
3. **Malware Handling**
   - Keep extracted malicious files quarantined.
   - Use VirusTotal and sandbox analysis for confirmation.
4. **Endpoint & Network Hygiene**
   - Maintain egress filtering and RFC1918 enforcement.
   - Update SIEM rulesets and IDS signatures to reduce false positives.
5. **Documentation**
   - Maintain detailed logs and screenshots for audits or purple team demonstrations.

---

## Supporting Evidence
1. Security Onion GUI: Suricata alert (Zbot POST request).
2. Wireshark PCAP: HTTP GET/POST analysis with source/destination IPs, timestamps, and MAC addresses.
3. VirusTotal Report: 65/72 AV engines detected Trojan/Backdoor.
4. pfSense Logs: No internal host traffic to malicious IPs.
5. SPAN Configuration: VLANs mirrored to bond0 interface.

---

## Final Verdict

| Classification           | Status                                  |
|-------------------------|----------------------------------------|
| Incident Type           | False Positive / External Traffic      |
| Malware Detection       | True Positive (Detected by Strelka)    |
| Internal Compromise     | Not Confirmed                           |
| Network Containment     | Secure                                  |
| Investigation Outcome   | Closed — No internal infection detected |
