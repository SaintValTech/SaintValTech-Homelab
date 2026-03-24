# Project 4 — Defender for Endpoint Onboarding & Connectivity Troubleshooting in a Filtered Enterprise Network

## Objective

The objective of this project was to validate Microsoft Defender for Endpoint health on a Windows Server / Domain Controller inside a filtered enterprise-style lab and troubleshoot a configuration enforcement issue reported by Microsoft security tooling.

This project was designed to:

- validate Microsoft Defender for Endpoint health on a Windows Server / Domain Controller
- troubleshoot a Microsoft portal configuration enforcement error
- determine whether the issue was caused by DNS, firewall, endpoint state, or onboarding failure
- restore required Microsoft Defender connectivity without weakening the lab’s filtering posture
- document a reliable server-specific validation method for Microsoft Defender for Endpoint
- confirm Defender for Identity sensor health where applicable
- create an operational troubleshooting reference for filtered enterprise networks

This project reflects a real-world scenario where endpoint security tooling must coexist with aggressive DNS and edge filtering controls.

---

## Environment Overview

The affected system existed inside a segmented, security-filtered environment.

### Affected System

- **Host:** CORP-DC01
- **Role:** Domain Controller
- **Platform:** Windows Server
- **Function:** Tier-0 identity infrastructure and Microsoft security onboarding target

### Network Context

The device existed behind a filtered architecture that included:

- pfSense at the edge
- pfBlockerNG DNSBL and IP reputation controls
- segmented VLAN design
- controlled outbound security filtering
- enterprise-style DNS handling through internal infrastructure

### Security Stack Context

The affected server was expected to support:

- Microsoft Defender Antivirus
- Microsoft Defender for Endpoint
- Microsoft Defender for Identity sensor operations
- Microsoft Sentinel ingestion
- hybrid identity workflows via Microsoft Entra ID

---

## Problem Statement

After onboarding activity, Microsoft security tooling indicated that the device had a configuration enforcement or connectivity issue.

This created an ambiguous state:

- onboarding appeared partially complete
- core services were running locally
- general internet connectivity existed
- portal health was not clean
- policy and telemetry confidence could not be assumed

The project therefore became a full operational investigation into whether the problem was:

- a failed onboarding
- a DNS resolution issue
- a TCP/443 reachability issue
- a service-state issue
- a validation-method misunderstanding
- or some combination of the above

---

## Initial Validation

### 1. Defender Local Health Checks

The first stage focused on confirming the local state of Defender on the Windows Server.

#### Commands used

- `Get-MpComputerStatus`
- `Get-Service Sense`

#### Observed indicators

- Antivirus enabled
- Real-time protection enabled
- behavior monitoring enabled
- signatures current
- Sense service running
- tamper protection enabled via ATP
- host recognized as a virtual machine

Initial conclusion: the local Microsoft Defender stack appeared healthy.

---

### 2. Baseline Internet Validation

General outbound connectivity was tested to determine whether the device had a broad internet access problem.

#### Example validation

- `Invoke-WebRequest https://www.microsoft.com`

The request succeeded, confirming that general outbound HTTPS was functional.

---

## Troubleshooting Workflow

### 3. Endpoint Connectivity Testing

The next phase focused on specific Microsoft Defender-related endpoints.

#### Example tests performed

- `Invoke-WebRequest https://winatp-gw-cus.microsoft.com/ping`
- `Invoke-WebRequest https://winatp-cus.microsoft.com/ping`
- `Test-NetConnection endpoint.security.microsoft.com -Port 443`
- `Test-NetConnection security.microsoft.com -Port 443`
- `Test-NetConnection login.microsoftonline.com -Port 443`
- `Test-NetConnection enterpriseregistration.windows.net -Port 443`

#### Key findings

- some Microsoft endpoints were reachable
- some requests returned HTTP 404, which still indicated network reachability
- at least one critical endpoint failed with remote name could not be resolved
- `endpoint.security.microsoft.com` failed DNS resolution
- general Microsoft services like `security.microsoft.com` and `login.microsoftonline.com` were reachable

---

### 4. Sense Service Handling

The Sense service was checked to ensure that the MDE sensor component was present and active.

- `Get-Service Sense`

The Sense service was running. Attempts to restart it were not the correct primary fix path and were also complicated by service protection and privilege boundaries.

---

## Root Cause Analysis

The primary root cause was identified as:

**DNS-based filtering interfering with resolution of required Microsoft Defender for Endpoint cloud endpoints.**

### Evidence supporting this conclusion

- general internet access worked
- some Microsoft security endpoints were reachable
- specific Defender-related hostnames failed DNS resolution
- the environment used pfBlockerNG DNSBL
- post-whitelist rebuild behavior aligned with restored resolution expectations
- the portal error described connectivity enforcement issues, which matched the DNS findings

In enterprise or semi-enterprise environments, security filtering can accidentally block cloud telemetry, service discovery, or Microsoft security platform dependencies. This was a real operational dependency conflict.

---

## Remediation

### pfBlockerNG DNSBL Review

Because the network edge used pfBlockerNG with multiple DNSBL and IP reputation feeds, the remediation path focused on reviewing filtering behavior and whitelisting required domains.

### Microsoft Defender Domain Whitelisting

#### Core Defender-related endpoints

- `.wdatp.com`
- `winatp-gw.microsoft.com`
- `winatp-gw-cus.microsoft.com`
- `winatp-cus.microsoft.com`
- `endpoint.security.microsoft.com`
- `.security.microsoft.com`

#### Telemetry / pipeline-related domains

- `browser.pipe.aria.microsoft.com`
- `.aria.microsoft.com`
- `.events.data.microsoft.com`

#### Shared Microsoft service dependencies

- `.bing.com`
- `.msn.com`
- `.windows.com`

Allowing only the obvious primary endpoint is often not enough. Microsoft security products depend on multiple service, telemetry, and supporting domains.

### DNSBL Rebuild and Service Reload

After whitelist updates, DNSBL was rebuilt and Unbound / DNSBL services were restarted through pfBlockerNG. This refreshed the filtering layer cleanly and allowed subsequent validation to proceed.

---

## Validation After Remediation

### DNS Validation

- `Resolve-DnsName endpoint.security.microsoft.com`
- `nslookup endpoint.security.microsoft.com`

Expected result: previously failing Defender-related domains now resolve successfully.

### TCP/443 Validation

- `Test-NetConnection endpoint.security.microsoft.com -Port 443`

Expected result: DNS and outbound HTTPS path both restored.

### Local Service Validation

- `Get-Service Sense`
- `Get-MpComputerStatus`

Expected state:

- Sense running
- Defender enabled
- real-time protection enabled
- signatures current

### Portal-Side Validation

Final authority for health confirmation is the Microsoft security portal. Expected improvements included:

- connectivity enforcement error clears
- device reports healthy / up to date state
- telemetry and policy handling normalize

---

## Server-Specific Validation Insight

### Why OnboardingState Was Misleading

One of the most important lessons in this project was that Windows Server does not expose onboarding status in the same way as Windows client devices.

- `Get-MpComputerStatus | Select OnboardingState`

A blank `OnboardingState` on Windows Server is not, by itself, proof of failed onboarding.

### Correct Validation Model for Servers

Validation should instead focus on:

- Sense service state
- Defender health indicators
- DNS and endpoint connectivity tests
- portal device visibility and health
- Defender for Identity sensor status where relevant

---

## Defender for Identity Sensor Findings

The environment also included Defender for Identity sensor validation.

### Observed findings

- sensor page and workflows were reviewed
- the Domain Controller sensor appeared present
- service status showed running
- health showed healthy
- version data appeared current

Where the sensor showed running and healthy state, Defender for Identity was already in a good operational condition.

### ADCS / ADFS “Unable to Determine” Interpretation

A related portal observation involved “Unable to determine” status for ADCS / ADFS-related visibility.

In this case, the most likely explanation was that those roles were not installed on the server.

This is not automatically an error. It can simply mean the role is not deployed or not applicable in the current architecture.

---

## Challenges Encountered

- one security control impaired another: DNSBL filtering interfered with cloud security tooling
- distinguishing local service health from cloud onboarding completion
- separating DNS failures from transport failures
- interpreting Microsoft portal health correctly in a server environment
- avoiding misleading client-style validation assumptions on Windows Server

Without proper interpretation, the environment could easily have been misdiagnosed as a bad onboarding, broken Sense service, or failed internet access problem.

---

## Security and Operational Value

This project demonstrates practical capability in:

- Microsoft Defender for Endpoint troubleshooting
- Windows Server security validation
- DNS and network dependency analysis
- pfBlockerNG DNSBL operations
- Microsoft endpoint whitelisting strategy
- interpreting cloud security platform health correctly
- resolving conflicts between network security controls and endpoint tooling
- Defender for Identity validation
- portal-to-host troubleshooting workflow

Operationally, this project matters because successful onboarding is not just about installing an agent or seeing a running service. Real success depends on cloud connectivity, correct DNS behavior, endpoint health, portal-side validation, and architecture-aware interpretation of status indicators.

---

## Tools & Technologies

- Microsoft Defender for Endpoint
- Microsoft Defender Antivirus
- Sense service
- Microsoft security portal
- Microsoft Defender for Identity
- pfSense
- pfBlockerNG
- DNSBL
- Unbound DNS Resolver
- Windows Server
- PowerShell
- Test-NetConnection
- Resolve-DnsName
- Microsoft Entra / hybrid identity environment

---

## Skills Demonstrated

- endpoint onboarding troubleshooting
- Microsoft security stack validation
- DNS troubleshooting
- pfBlockerNG whitelist design
- filtered network operations
- cloud endpoint dependency analysis
- service-state interpretation
- server-specific security validation
- technical documentation
- incident-style root cause analysis

---

## Outcome

This project successfully identified and remediated a Microsoft Defender for Endpoint connectivity enforcement issue caused by DNS-based filtering in the lab’s security architecture. It also established a more accurate and operationally sound validation method for Microsoft Defender on Windows Server, avoiding misleading client-side assumptions such as blank OnboardingState interpretation.

The result was a healthier, more trustworthy Microsoft security posture for the Domain Controller and a documented troubleshooting pattern that can be applied to filtered enterprise environments in the future.
