# 🧠 Zbot C2 Malware Incident Investigation

**Author:** Emeka Valentine Ogbu (SaintValTech)  
**Date:** October 15–18, 2025  
**Environment:** Security Onion (Suricata, Strelka), pfSense Firewall, UDM Pro, VMware-based SOC Lab  

This folder contains a full incident report of a malware detection and analysis exercise conducted in my SOC homelab.  
The purpose was to validate detection and investigation workflows for Command & Control (C2) activity using Security Onion.

### Summary
- Detected: Zbot/Bifrose Trojan HTTP POST traffic to `188.72.243.72 (Webzilla B.V.)`
- Source IP appeared internal but unassigned (mirror artifact)
- Malware file: `HTTP-FYpTVB33entBpgOilk-9e04a788281c727566873d9df263aec1.exe`
- Verified via VirusTotal: 65/72 detections (Trojan/Backdoor)
- Final Verdict: **False Positive (External Traffic Observed)**  
- SOC Detection Capability: **Validated**

### Tools Used
- **Security Onion** (Suricata, Strelka)
- **pfSense Firewall**
- **Wireshark**
- **VirusTotal**
- **VMware SOC Lab**

📄 [Read Full Report](./Incident_Report.md)
