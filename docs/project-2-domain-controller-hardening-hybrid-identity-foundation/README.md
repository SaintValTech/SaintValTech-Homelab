# Project 2 — Domain Controller Hardening & Hybrid Identity Foundation

## Objective

The objective of this project was to build and harden the identity foundation of the SaintValTech SOC lab by securing a Windows Domain Controller and extending that identity layer into Microsoft Entra ID.

This project was designed to support:

- enterprise-style Active Directory security baselines
- Tier-0 identity protection
- audit-ready logging and monitoring
- secure Kerberos, LDAP, SMB, and PowerShell configuration
- hybrid identity integration with Microsoft Entra ID
- Hybrid Azure AD Join readiness
- Microsoft Defender and Sentinel identity-aware security workflows

This project built the identity control plane that later supported Defender for Endpoint onboarding, Sentinel automation, and Zero Trust access design.

---

## Environment Overview

The project centered on the primary Domain Controller used in the lab.

### Domain Controller Details

- **Hostname:** CORP-DC01
- **Domain:** `corp.saintvaltech.com`
- **Role:** Primary Domain Controller
- **Security classification:** Tier-0 identity infrastructure

The Domain Controller served as the foundation for:

- Active Directory authentication and authorization
- Group Policy enforcement
- audit logging and event generation
- Azure AD Connect Sync
- Hybrid Azure AD Join
- Microsoft security integrations
- future identity-aware detection and access control workflows

---

## Project Scope

This project had two tightly connected phases:

1. **Domain Controller hardening baseline**
2. **Hybrid identity foundation with Azure AD Connect and Hybrid Azure AD Join**

Together, these phases transformed the Domain Controller from a basic AD server into a hardened identity platform connected to Microsoft’s cloud identity stack.

---

## Phase 1 — Domain Controller Hardening Baseline

The first part of the project focused on applying a structured security baseline to the Domain Controller using Group Policy and Windows security controls.

### GPO Used

- **GPO Name:** `GPO - Domain Controller Security Baseline`
- **Linked OU:** `01 - Domain Controllers`

The baseline focused on reducing attack surface, improving protocol security, and increasing visibility for SOC monitoring.

---

## Hardening Areas Implemented

### 1. Kerberos Hardening

Kerberos policy was configured to enforce a stronger and more controlled authentication baseline.

#### Configured settings included

- enforce user logon restrictions
- maximum lifetime for service ticket
- maximum lifetime for user ticket
- maximum lifetime for user ticket renewal
- maximum tolerance for computer clock synchronization

#### Validation goal

- confirm Kerberos ticket behavior
- confirm AES-based ticketing rather than weak legacy fallback

This improved the authentication posture of the Domain Controller and aligned it with enterprise identity expectations.

---

### 2. NTLM Hardening

NTLM was restricted to reduce reliance on weaker legacy authentication methods.

#### Configured settings included

- LAN Manager authentication level set to NTLMv2-only
- deny incoming NTLM traffic
- deny outgoing NTLM traffic where appropriate

This was important because unnecessary NTLM support expands attack surface and weakens identity security.

---

### 3. LDAP Signing

LDAP signing requirements were enforced on the Domain Controller.

#### Goal

- require signed LDAP traffic
- reduce exposure to unsigned directory communications
- strengthen directory service integrity

This improved the security of Active Directory communications and aligned with a more secure directory baseline.

---

### 4. SMB Hardening

SMB signing requirements were configured to strengthen server and client communication integrity.

#### Configured areas included

- SMB client signing always enabled
- SMB server signing always enabled
- legacy or weaker negotiation behavior reduced where appropriate

This hardened file and service communication behavior on the Domain Controller.

---

### 5. Advanced Audit Policy

Advanced Audit Policy was enabled to support meaningful SIEM and SOC visibility.

#### Audit categories covered

- Account Logon
- Logon / Logoff
- Account Management
- Directory Service Access
- Directory Service Changes
- Object Access
- Privilege Use
- Policy Change

#### Operational purpose

- generate useful security telemetry
- improve detection potential in Wazuh, Sentinel, and related tools
- support investigations with structured log evidence

This was one of the most important parts of the project because hardening without visibility is incomplete.

---

### 6. PowerShell Logging

PowerShell logging was enabled to improve visibility into script execution and administrative activity.

#### Logging areas included

- Script Block Logging
- script block invocation logging
- Module Logging
- PowerShell Transcription

This provided stronger detection coverage for PowerShell-based activity on the Domain Controller.

---

### 7. User Rights Assignment and Tier-0 Lockdown

User rights were reviewed and restricted to reduce unnecessary privilege exposure on the Domain Controller.

#### Focus areas included

- local logon rights
- RDP logon rights
- deny logon rules for low-trust or local account scenarios
- restricting sensitive rights such as debugging and security log control

This reinforced the principle that Domain Controllers should be tightly administered and not treated like normal servers.

---

### 8. Windows Defender Firewall Baseline

The Domain Profile firewall baseline was reviewed and enforced.

#### Configured behavior

- firewall on
- inbound block by default
- outbound allow by default
- logging of dropped packets
- logging of successful connections

This gave the Domain Controller a stronger host-level network baseline while preserving controlled server functionality.

---

## Validation of the Domain Controller Baseline

Validation focused on proving that the baseline was not just configured, but actually applied.

### Example validation methods

- `gpupdate /force`
- `gpresult /r`
- `auditpol /get /category:*`
- `klist`
- registry queries for LDAP and SMB values
- PowerShell policy validation commands

### Validation goals

- confirm the baseline GPO was applied
- confirm audit settings were active
- confirm Kerberos behavior aligned with the intended configuration
- confirm LDAP signing and SMB signing values
- confirm PowerShell logging policy presence

This transformed the baseline from a checklist into an auditable security configuration.

---

## Phase 2 — Hybrid Identity Foundation

After securing the on-prem identity layer, the next phase extended the Domain Controller environment into Microsoft Entra ID.

The purpose of this phase was to prepare the environment for:

- Hybrid Azure AD Join
- Microsoft Defender for Endpoint
- Conditional Access support
- Microsoft Sentinel identity-aware workflows
- identity-driven Zero Trust architecture

---

## Hybrid Identity Components Implemented

### 1. Active Directory Forest Validation

The Active Directory forest and domain were reviewed before synchronization work began.

#### Validation areas included

- forest health
- domain identity
- DNS and Global Catalog readiness
- UPN suffix review
- tenant and domain verification readiness

This ensured the on-prem identity layer was stable enough to synchronize safely.

---

### 2. Microsoft 365 / Entra ID Tenant Validation

The Microsoft tenant side was checked to confirm that:

- the SaintValTech domain existed
- the domain was verified
- tenant administration was functional
- Entra ID was accessible
- the environment was ready for sync operations

---

### 3. Azure AD Connect Sync Installation

Azure AD Connect was installed using an enterprise-style hybrid identity setup approach.

#### Features enabled

- Password Hash Sync
- Seamless SSO
- synchronization scheduler support

A browser authentication issue during setup was identified and addressed using registry-based alternate browser handling.

This was an important operational step because it reflects the kind of troubleshooting required in real hybrid deployments.

---

### 4. Initial Synchronization

Initial synchronization was triggered and reviewed to confirm the connector was functioning correctly.

#### Validation focus

- sync scheduler enabled
- regular sync interval confirmed
- initial synchronization completed
- users created on-prem appeared in Entra ID

This proved that the identity pipeline between on-prem Active Directory and Microsoft Entra ID was working.

---

### 5. Hybrid Azure AD Join

After synchronization was functioning, Hybrid Azure AD Join was configured.

#### Result

- on-prem devices remained domain joined
- devices could also register in Microsoft Entra ID
- the Domain Controller identity path was extended into the hybrid model
- future device-aware Microsoft security controls became possible

This was a critical step because it connected the lab’s identity layer to Microsoft’s cloud security ecosystem.

---

## Validation of the Hybrid Identity Phase

Validation focused on confirming that synchronization and hybrid registration were real, not assumed.

### Validation areas included

- Azure AD Connect scheduler state
- synchronization cycle status
- user appearance in Entra ID
- Hybrid Azure AD Join completion state
- `dsregcmd /status` output
- portal-side device presence
- domain join plus Azure AD join coexistence

### Operational outcome

The environment moved from being a local-only Active Directory deployment to a hybrid identity platform capable of supporting modern Microsoft security tooling.

---

## Security and Operational Value

This project demonstrates practical capability in:

- Active Directory hardening
- Domain Controller security baselining
- Group Policy security design
- audit policy implementation
- PowerShell logging configuration
- Kerberos, LDAP, SMB, and NTLM security controls
- hybrid identity planning
- Azure AD Connect deployment
- Hybrid Azure AD Join configuration
- Microsoft identity troubleshooting
- documentation-driven identity engineering

Operationally, this project matters because identity is the control plane for everything that followed.

Without a hardened and well-integrated identity layer, later work in Defender, Sentinel, TheHive exposure, and SOAR orchestration would be less trustworthy and less realistic.

---

## Tools & Technologies

- Windows Server
- Active Directory
- Group Policy
- Kerberos
- LDAP signing
- SMB signing
- NTLM hardening
- Windows Defender Firewall
- PowerShell logging
- Azure AD Connect
- Microsoft Entra ID
- Hybrid Azure AD Join
- Microsoft 365 tenant administration

---

## Skills Demonstrated

- Domain Controller hardening
- Group Policy security baseline design
- protocol security hardening
- audit logging strategy
- PowerShell visibility design
- identity troubleshooting
- hybrid identity integration
- Azure AD Connect configuration
- Hybrid Azure AD Join validation
- enterprise security documentation

---

## Outcome

This project established the hardened identity foundation of the SaintValTech SOC environment.

It secured the primary Domain Controller with an enterprise-style baseline, improved authentication and logging posture, and extended the environment into Microsoft Entra ID through Azure AD Connect and Hybrid Azure AD Join.

More importantly, it created a trustworthy identity layer that supported later work in Microsoft Defender, Microsoft Sentinel, Zero Trust access, and multi-platform SOC integration.
