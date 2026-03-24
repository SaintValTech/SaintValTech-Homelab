# SaintValTech Homelab

This repository documents the enterprise-style SOC and security engineering lab behind the SaintValTech portfolio.

It combines practical work across network security architecture, identity hardening, SIEM visibility, Microsoft security operations, Zero Trust access design, incident response, and multi-source SOAR integration.

The repository includes both:
- **flagship portfolio projects** that represent the main SaintValTech security architecture story
- **supporting implementation notes and operational documentation** that show how specific components were configured, tested, and maintained

---

## Flagship Portfolio Projects

The following six projects represent the core portfolio structure built from this homelab:

1. **Enterprise SOC Homelab & Network Security Architecture**  
   Segmented security lab architecture built with pfSense, VLANs, Cisco switching, packet visibility, SIEM tooling, and controlled trust boundaries.

2. **Domain Controller Hardening & Hybrid Identity Foundation**  
   Tier-0 Active Directory hardening, Group Policy security baseline, audit logging, Azure AD Connect Sync, and Hybrid Azure AD Join.

3. **Microsoft Sentinel SOAR Pipeline & Incident Automation**  
   Incident-driven Microsoft Sentinel automation using analytics rules, Azure Logic Apps, and validated incident-to-playbook execution.

4. **Defender for Endpoint Onboarding & Connectivity Troubleshooting in a Filtered Enterprise Network**  
   Troubleshooting Microsoft Defender for Endpoint onboarding and connectivity enforcement in a DNS-filtered environment protected by pfBlockerNG.

5. **Zero Trust Exposure of TheHive Using Cloudflare Tunnel, Access, and Microsoft Entra ID**  
   Secure publication of TheHive without inbound firewall exposure using Cloudflare Tunnel, Cloudflare Access, Microsoft Entra ID, and MFA.

6. **Multi-Source SOAR Integration with Sentinel, Security Onion, Wazuh, Shuffle, TheHive, and Cortex**  
   Cross-platform SOC workflow design connecting cloud, endpoint, and network detections into a structured alert and case management pipeline.

These project narratives are being organized under the `docs/` directory as the primary portfolio documentation layer.

---

## Supporting Documentation and Implementation Notes

In addition to the six flagship projects, this repository also contains supporting technical documentation and working notes from the broader homelab environment.

These materials remain important because they show the underlying implementation work behind the portfolio.

### `firewall/`
Firewall, VLAN, VPN, and network security implementation notes, including pfSense-related configuration work.

### `endpoint/`
Endpoint security, onboarding, Defender-related setup, and host-level implementation notes.

### `siem/`
Security monitoring, SIEM, log collection, and detection-related implementation notes.

### `real-world-incidents/`
Documented incident response work based on real-world security events and investigations.

### `docs/images/`
Supporting image placeholders and screenshots used in lab documentation.

### `docs/purple-team-lab.md`
Earlier purple-team-oriented lab documentation that may later be expanded, reorganized, or absorbed into the broader project structure.

---

## Repository Structure

- `docs/`  
  Main structured project documentation, including the six flagship SaintValTech portfolio projects.

- `firewall/`  
  Firewall, segmentation, VPN, and network security notes.

- `endpoint/`  
  Endpoint configuration and host security notes.

- `siem/`  
  Detection, SIEM, and monitoring documentation.

- `real-world-incidents/`  
  Incident response and investigation write-ups.

---

## Documentation Approach

This repository is not just a collection of isolated tool notes.

It is a documentation-driven security lab and portfolio that shows:
- how the environment was designed
- how security decisions were made
- how detections and workflows were validated
- how problems were troubleshot
- how multiple platforms were integrated into a usable SOC workflow

The emphasis is on operational realism, structured documentation, and architecture-level thinking rather than isolated demos.

---

## Technologies Used Across the Lab

- pfSense
- pfBlockerNG
- Cisco CBS350
- VLAN segmentation
- Wazuh
- Security Onion
- Zeek
- Suricata
- Microsoft Sentinel
- Microsoft Defender for Endpoint
- Microsoft Defender XDR
- Microsoft Entra ID
- Azure Logic Apps
- TheHive
- Cortex
- Shuffle
- Cloudflare Tunnel
- Cloudflare Access
- Windows Server
- Windows 10/11
- Ubuntu / Linux
- Kali Linux

---

## Purpose

This repository serves as:

1. A technical portfolio for recruiters, hiring managers, and collaborators
2. A structured record of hands-on security engineering and SOC project work
3. A practical reference for documenting enterprise-style cybersecurity lab design

---

## Website

SaintValTech portfolio website:  
[https://saintvaltech.com](https://saintvaltech.com)

---

## License

This project is licensed under the MIT License.
