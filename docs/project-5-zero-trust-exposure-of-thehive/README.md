# Project 5 — Zero Trust Exposure of TheHive Using Cloudflare Tunnel, Access, and Microsoft Entra ID

## Objective

The objective of this project was to securely expose TheHive to authorized users without weakening the network security posture of the SOC environment.

Specifically, the project aimed to:

- eliminate inbound port forwarding for TheHive
- avoid public IP exposure of the platform
- enforce identity-aware access controls before application login
- integrate access authentication with Microsoft Entra ID
- require MFA at the access perimeter
- preserve TheHive’s placement inside a protected SOC subnet
- create a cleaner and more defensible architecture than traditional reverse proxy or NAT exposure

TheHive also needed to support secure integration with **Microsoft Sentinel SOAR workflows**, enabling Logic Apps to send incident and alert data into TheHive as part of the broader case management pipeline.

---

## Environment Overview

### Platform Being Exposed

- **Application:** TheHive
- **Role:** SOC case management and incident response platform
- **Function in lab:** central analyst workspace for alerts, cases, and workflow coordination

### Internal Placement

TheHive was deployed inside a hardened SIEM / SOC subnet behind pfSense with strict segmentation and no requirement for direct public inbound access.

### Supporting Components

- pfSense as edge firewall
- Cloudflare Tunnel (`cloudflared`) for outbound-only service publication
- Cloudflare Access for Zero Trust access enforcement
- Microsoft Entra ID for identity and MFA
- Cloudflare DNS for public hostname handling

### Protected Hostname

- `thehive.saintvaltech.com`

---

## Problem Statement

Initial attempts to expose TheHive through traditional methods introduced complexity and unnecessary risk.

### Problems with Traditional Exposure

- required inbound WAN port forwarding
- introduced NAT and firewall troubleshooting complexity
- increased attack surface
- created edge cases around DNS rebinding and reverse path behavior
- complicated certificate and HTTPS handling
- conflicted with pfSense hardening and non-standard management port configuration
- made the architecture harder to explain in a modern SOC context

The platform needed to be accessible to authorized analysts while preserving the integrity of the SOC subnet and avoiding unnecessary exposure at the network edge.

In addition, TheHive needed to be reachable in a secure and reliable way for **Microsoft Sentinel SOAR workflows**, allowing Logic Apps to forward incidents and alerts into TheHive without relying on traditional inbound firewall exposure or fragile NAT-based publishing.

This led to the decision to replace traditional exposure with a **Zero Trust publication model**.

---

## Design Decision

Cloudflare Tunnel was selected because it allowed TheHive to be published through **outbound-only connectivity** from the internal host to Cloudflare, eliminating the need for inbound firewall openings.

### Benefits of This Design

- no public IP exposure of TheHive
- no WAN port forwarding
- no NAT rules
- no NAT reflection problems
- no inbound firewall exceptions required
- TLS handled through Cloudflare edge services
- access controlled before the application is reached
- better alignment with Zero Trust and MSSP-grade access patterns

Security philosophy: the platform should be reachable, but not directly exposed. Access control should happen at the identity and edge layer before the application itself is presented.

---

## Final Architecture

### High-Level Flow

User browser  
→ Cloudflare Edge over HTTPS  
→ Cloudflare Access policy  
→ Cloudflare Tunnel  
→ `cloudflared` running on TheHive VM  
→ TheHive bound to localhost

### Security Properties of the Final Design

- no inbound traffic reaches pfSense for this application
- TheHive is not bound to LAN or WAN interfaces
- access is mediated by Cloudflare Access
- authentication is handled by Microsoft Entra ID
- MFA is enforced before the application is reached
- TheHive remains inside the SOC subnet and hidden from direct exposure

---

## Core Components Implemented

### 1. TheHive Localhost Binding

The first hardening step was to ensure TheHive was not directly reachable from the network.

#### Relevant configuration intent

- `play.server.http.address = "127.0.0.1"`
- `play.server.http.port = 9000`

This guaranteed that TheHive could not be reached directly over LAN or WAN and that Cloudflare Tunnel became the only intended access path.

---

### 2. Cloudflare Tunnel Installation

`cloudflared` was installed on the TheHive host to act as the outbound tunnel client.

This created and maintained the secure outbound tunnel between the internal TheHive VM and Cloudflare’s edge network.

---

### 3. Cloudflare Tunnel Authentication and Creation

The tunnel client was authenticated with Cloudflare and a dedicated tunnel was created for the service.

#### Workflow included

- authenticating `cloudflared` with the Cloudflare account
- creating a tunnel for TheHive
- generating tunnel credentials
- storing tunnel metadata securely on the host

---

### 4. Tunnel Configuration

A tunnel configuration file mapped the external hostname to the internal local service.

### Configuration intent included

- tunnel name / ID
- credentials file reference
- ingress rule for `thehive.saintvaltech.com`
- service target pointing to `http://127.0.0.1:9000`
- fallback HTTP 404 route

---

### 5. DNS Routing Through Tunnel

The existing traditional DNS model was replaced with Cloudflare-managed tunnel routing.

Cloudflare DNS now resolves the public TheHive hostname to the Cloudflare Tunnel route rather than to a public IP or manually forwarded service.

---

### 6. Tunnel as a Persistent Service

`cloudflared` was installed and enabled as a system service to ensure the tunnel starts automatically, survives reboots, and behaves like a production service.

---

## Zero Trust Access Control

### 7. Cloudflare Access Application

Once the tunnel path existed, Cloudflare Access was used to enforce identity-aware access controls.

#### Application characteristics

- application type: self-hosted
- protected application URL: `thehive.saintvaltech.com`

---

### 8. Microsoft Entra ID Integration

Microsoft Entra ID was configured as the identity provider for Cloudflare Access.

Users attempting to access TheHive are redirected through Microsoft identity authentication before access is granted.

---

### 9. Access Policy Enforcement

Cloudflare Access policy was configured to restrict access based on organizational identity and authentication requirements.

#### Policy goals

- allow only approved SaintValTech identities
- require OTP / MFA or equivalent strong authentication

This creates layered control:

- identity verification at Cloudflare
- MFA enforcement
- application authentication inside TheHive

---

## Authentication Flow

Final login flow:

User  
→ `thehive.saintvaltech.com`  
→ Cloudflare Access  
→ Microsoft Entra ID  
→ MFA  
→ session grant from Cloudflare  
→ TheHive login page  
→ TheHive application authentication

### Separation of Duties

- **Microsoft Entra ID** → identity and MFA
- **Cloudflare Access** → perimeter access enforcement
- **TheHive** → application-level authorization and use

---

## Validation & Testing

### External Access Validation

- phone / cellular network access
- external browser access from outside the LAN

Result: TheHive was reachable externally without any public inbound exposure of the internal service host.

### Internal Access Validation

Internal access was also validated where appropriate to ensure the solution worked cleanly through the intended hostname and access flow.

### Security Validation

- no WAN ports opened for TheHive
- no NAT rules required
- no public IP exposure of TheHive
- no inbound firewall exceptions required
- MFA enforced before application access
- TheHive remained bound to localhost only
- SOC subnet placement preserved
- TLS handled at Cloudflare edge

---

## Challenges Encountered

- traditional exposure models created HTTPS redirection issues, certificate handling difficulties, and DNS rebinding conflicts
- older NAT-based publication patterns were operationally fragile for a sensitive SOC service
- the challenge was not merely making TheHive reachable, but making it reachable without undermining segmentation and firewall posture

The final design solved that tension by making publication outbound-only and access identity-driven.

---

## Relationship to the Wider SOC Stack

This project also clarified the role of TheHive relative to other tools in the SaintValTech environment.

- **TheHive** → analyst workspace and case management
- **Sentinel** → detection and response orchestration
- **Wazuh / Security Onion** → telemetry and alert generation
- **Shuffle** → automation broker
- **Cloudflare Access** → Zero Trust entry point

TheHive should be accessible to analysts, but supporting telemetry systems and backend components should not all be exposed the same way.

---

## Security and Operational Value

This project demonstrates practical capability in:

- Zero Trust service publication
- Cloudflare Tunnel deployment
- Cloudflare Access configuration
- Microsoft Entra ID integration
- MFA-enforced perimeter access
- localhost-only application hardening
- secure DNS-based publication design
- eliminating NAT-based exposure for sensitive internal services
- SOC platform access architecture

Operationally, this project matters because SOC tools like TheHive are high-value platforms. Exposing them carelessly introduces unnecessary risk. This project shows an understanding that analyst access should be controlled, identity-aware, auditable, minimally exposed, and architecturally clean.

---

## Tools & Technologies

- TheHive
- Cloudflare Tunnel
- `cloudflared`
- Cloudflare Access
- Microsoft Entra ID
- Cloudflare DNS
- pfSense
- localhost service binding
- MFA
- self-hosted application publication controls

---

## Skills Demonstrated

- Zero Trust architecture design
- secure external service publication
- Cloudflare Tunnel deployment
- identity-aware access enforcement
- Entra ID integration
- MFA access control
- SOC platform hardening
- service exposure minimization
- infrastructure documentation
- security architecture reasoning

---

## Outcome

This project successfully exposed TheHive to authorized users without introducing inbound firewall exposure, NAT complexity, or public application exposure. By combining Cloudflare Tunnel, Cloudflare Access, and Microsoft Entra ID, the platform was transformed from an internally isolated service into a securely accessible SOC workspace protected by identity-aware Zero Trust controls.

The result is a cleaner, safer, and more portfolio-worthy architecture than traditional port forwarding, and it reflects how sensitive analyst platforms should be published in modern security environments.
