# Project 1 — Enterprise SOC Homelab & Network Security Architecture

## Objective

The objective of this project was to build a realistic security-focused lab environment that could support:

- enterprise-style network segmentation
- firewall policy enforcement
- centralized visibility across endpoints and network traffic
- SIEM and IDS-based detection workflows
- secure remote access
- incident simulation and validation
- repeatable documentation of security operations processes

This environment serves as the foundational platform for later projects involving Active Directory hardening, Microsoft Sentinel SOAR, Wazuh integrations, TheHive case management, and Zero Trust access design.

---

## Environment Overview

The homelab was designed to reflect a layered enterprise network with clear separation between user traffic, lab systems, and security tooling.

### Core Infrastructure

- pfSense (Netgate 2100) as the primary firewall and router
- Cisco CBS350 switch for VLAN assignment and SPAN / port mirroring
- UDM Pro for family / personal network segmentation
- Proxmox / virtualization hosts for Windows, Linux, and security workloads
- Ubuntu-based SIEM / monitoring hosts
- Windows Server and Windows client systems
- Security Onion, Wazuh, and Microsoft security tooling

### Network Segmentation Model

The environment was segmented using VLANs to separate traffic and reduce risk between functions.

#### Example segmentation

- VLAN 10 — UDM Pro / family and personal device network
- VLAN 20 — homelab / testing systems
- VLAN 30 — SIEM / monitoring infrastructure

This design allowed traffic isolation while still enabling controlled monitoring and inter-VLAN analysis where needed.

---

## Architecture Design

The architecture was built around the principle that monitoring infrastructure should be isolated from general-purpose lab activity and personal traffic.

### Key design principles

- firewall-centric control using pfSense
- traffic separation through VLANs
- mirrored packet visibility for network monitoring
- dedicated monitoring segment for SOC tooling
- remote access through secure VPN methods
- centralized log collection and analysis
- realistic detection and incident simulation workflows

### Traffic Visibility Strategy

A SPAN / mirror port on the Cisco switch was configured to feed monitored traffic to the security monitoring stack. This allowed Security Onion and related tools to inspect packet metadata and support network detection use cases without placing the monitoring stack inline.

### Remote Access Design

Remote access was implemented using secure methods such as:

- OpenVPN
- WireGuard
- Cloudflare-supported workflows where relevant later in the overall architecture

This ensured secure access to lab systems without directly exposing internal platforms unnecessarily.

---

## Core Components Implemented

### 1. pfSense Firewall Baseline

pfSense served as the primary network security control point.

Implemented functions included:

- VLAN-aware routing
- NAT
- DNS services
- DHCP
- firewall rule management
- VPN access support

pfSense was used as the authoritative control plane for segmentation, policy enforcement, and network security visibility.

---

### 2. pfBlockerNG DNSBL & IP Reputation Controls

pfBlockerNG was enabled to strengthen DNS-based and IP-based security filtering at the network edge.

#### DNSBL feeds enabled

- StevenBlack_ADs
- OpenPhish
- URLhaus
- Turkey_High_Risk

#### IPv4 deny feeds enabled

- CINS_army_v4
- ET_TOR_All_v4
- ET_Block_v4
- ET_Comp_v4
- ISC_Block_v4
- Spamhaus_Drop_v4
- Abuse_SSLBL_v4
- Abuse_Feodo_C2_v4

These controls provided baseline protection against known malicious or suspicious domains and IPs while also creating a real operational environment for troubleshooting false positives and service dependency issues.

---

### 3. VLAN Segmentation

The network was segmented into dedicated functional zones to reduce attack surface and better simulate enterprise environments.

#### Example use cases

- personal / family network separation
- homelab testing isolation
- dedicated SIEM / monitoring subnet
- protected security tooling placement

This segmentation model supported both operational realism and safer testing of lab activity.

---

### 4. Cisco Switching & SPAN Port Monitoring

The Cisco switch was used not just for VLAN assignment, but also for visibility.

Implemented capabilities included:

- VLAN trunking between infrastructure components
- isolated access ports by function
- SPAN / mirror configuration to feed network telemetry into monitoring tools

This allowed packet-level visibility without requiring inline disruption of production-style traffic paths.

---

### 5. Virtualization & Lab Services

The environment hosted multiple virtual machines to support both IT and SOC use cases.

Examples included:

- Windows Server for AD, DNS, and DHCP roles
- Windows 10/11 endpoints
- Linux hosts
- Kali / Kali Purple
- Ubuntu servers
- vulnerable test systems
- security tooling hosts

This created a multi-platform environment for log generation, endpoint monitoring, service testing, and attack simulation.

---

### 6. Security Monitoring Stack

The monitoring environment was built to support both host-based and network-based visibility.

#### Key platforms

- Wazuh for SIEM / endpoint visibility
- Security Onion for IDS / NSM workflows
- Zeek for network metadata
- Suricata for signature-based detection
- Microsoft Defender / Sentinel integrations in related projects

The result was a layered visibility model combining logs, alerts, telemetry, and packet-level evidence.

---

## Implementation Highlights

### Segmentation & Routing

The environment was configured so that each major trust boundary had a defined purpose, and firewall rules controlled communication between networks. This created a realistic operational model for:

- east-west traffic control
- monitoring policy validation
- troubleshooting connectivity issues
- validating detection coverage

### Monitoring Placement

Monitoring systems were deliberately isolated from general user and lab traffic while still receiving mirrored network data and system logs. This reflected a security architecture approach where monitoring infrastructure is protected rather than casually colocated.

### DNS & Security Filtering

DNSBL and IP reputation controls were enabled to simulate a realistic defensive posture. This also created opportunities to validate how security controls can interfere with cloud and telemetry services when not properly whitelisted.

### Operational Documentation

The environment was documented as an evolving enterprise-style platform, allowing later projects to inherit a stable and explainable foundation. This is important because the value of a SOC lab is not only in building it, but in being able to explain and defend the architecture.

---

## Challenges Encountered

As with real environments, the project included operational complexity beyond simple setup.

### Key challenges

- balancing secure filtering with application and cloud service dependencies
- managing segmentation without breaking necessary communication paths
- handling mirrored traffic visibility correctly
- ensuring SIEM and IDS tools received the right telemetry sources
- maintaining clear separation between lab workloads and monitoring systems
- supporting remote administration without weakening the edge posture

These challenges were important because they made the project operationally realistic rather than purely academic.

---

## Validation & Testing

Validation focused on confirming that the environment was not just deployed, but working as intended.

### Validation activities included

- verifying VLAN assignment and isolation
- confirming pfSense routing, DNS, and DHCP behavior
- validating pfBlockerNG feed activation and DNSBL updates
- confirming mirror / SPAN traffic delivery to monitoring tools
- reviewing logs and detections in Wazuh and Security Onion
- testing remote access and administrative connectivity
- confirming endpoint visibility across Windows and Linux systems

This validated the environment as a usable SOC and IT operations platform rather than a static lab build.

---

## Security and Operational Value

This project demonstrates practical capability in:

- network segmentation
- firewall administration
- DNS-based filtering
- SIEM / IDS architecture
- packet visibility design
- secure remote access
- documentation-driven infrastructure building

Operationally, it shows an understanding that security infrastructure must be:

- segmented
- observable
- controllable
- explainable
- supportable over time

That is the difference between installing security tools and building a usable security operations environment.

---

## Tools & Technologies

- pfSense
- pfBlockerNG
- Cisco CBS350
- VLANs
- NAT
- DNS
- DHCP
- OpenVPN
- WireGuard
- Wazuh
- Security Onion
- Zeek
- Suricata
- Proxmox
- Windows Server
- Windows 10/11
- Linux / Ubuntu
- Kali Linux
- Cloudflare services where applicable in later phases

---

## Skills Demonstrated

- firewall configuration
- VLAN segmentation
- network troubleshooting
- DNS and DHCP administration
- IDS / NSM visibility design
- SIEM integration
- packet monitoring architecture
- remote access configuration
- environment hardening
- infrastructure documentation
- security operations lab design

---

## Outcome

This project established the technical foundation for the entire SaintValTech SOC portfolio. It created the network, segmentation, monitoring, and visibility architecture required to support later work in identity hardening, hybrid Microsoft security, SOAR automation, TheHive integration, and multi-source incident workflows.

More importantly, it demonstrated the ability to design and operate a realistic enterprise-style lab that supports both IT operations and security operations use cases.
