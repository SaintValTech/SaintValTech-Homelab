# Project 6 — Multi-Source SOAR Integration with Sentinel, Security Onion, Wazuh, Shuffle, TheHive, and Cortex

## Objective

The objective of this project was to build a structured multi-source SOAR pipeline that could route cloud, endpoint, and network detections into a shared analyst workflow without collapsing everything into uncontrolled case creation.

This project was designed to:

- ingest Microsoft Sentinel incidents into TheHive
- ingest Wazuh alerts into TheHive through Shuffle
- route high-risk Security Onion detections into the same downstream workflow
- apply severity-based routing so not every signal becomes a formal case
- preserve alert-first triage instead of opening a case for every event
- support deduplication and stable object creation
- keep orchestration logic outside the SIEM and outside the case platform
- avoid exposing TheHive API directly to the internet
- create a reusable backbone for future correlation across cloud, endpoint, and network telemetry

This project reflects a real SOC design principle: detection, orchestration, enrichment, and case management should be connected, but they should not all be performed by the same tool.

---

## Environment Overview

This project connected multiple platforms already present in the SaintValTech SOC environment.

### Core Platforms

- Microsoft Sentinel for incident generation and Microsoft-native automation workflows
- Azure Logic Apps for Microsoft-native SOAR
- Security Onion for network detection and high-risk alerts
- Wazuh for endpoint and SIEM alerting, and as an intermediate alerting layer
- Shuffle for internal SOAR orchestration
- TheHive for alert and case management
- Cortex for enrichment workflows

### Supporting Security Context

- TheHive exposed securely through Cloudflare Tunnel and Cloudflare Access for analyst use
- internal east-west API communication used for backend platform-to-platform interactions
- SOC subnet segmentation in place
- self-signed / internal TLS decisions applied intentionally where appropriate inside the controlled lab

### Architectural Goal

The goal was not just to move alerts between tools. The goal was to build a sane operational model where:

- Sentinel handles incidents
- Security Onion generates high-signal network detections
- Wazuh acts as both an endpoint alert source and an intermediate alerting layer
- Shuffle orchestrates internal alert forwarding
- TheHive stores alerts and cases
- Cortex enriches objects later
- analysts work from a clean case-management layer

---

## Problem Statement

SOC environments become noisy and unmanageable when every source sends everything directly into a case management platform without decision logic, normalization, or orchestration.

### Common failure patterns this project aimed to avoid

- creating a case for every low-value signal
- duplicating analyst work across tools
- exposing backend APIs directly for convenience
- hardcoding routing logic into the wrong layer
- building enrichment into detection platforms instead of orchestration
- failing to deduplicate repeated alerts
- mixing analyst UI exposure with backend API traffic
- fragmenting endpoint, cloud, and network alerts into separate, disconnected workflows

The project therefore aimed to establish a more mature pattern:

- route signals intentionally
- create alerts first
- escalate to cases only when justified
- keep orchestration in the SOAR layer
- protect the case platform from unnecessary exposure
- consolidate high-signal detections from multiple sources into a shared analyst workflow

---

## Architecture Overview

### High-Level Design

#### Microsoft-native branch

Microsoft Sentinel  
→ Azure Logic App  
→ severity decision  
→ TheHive Case API or TheHive Alert API

#### Internal SOC telemetry branch

Security Onion  
→ selected high-risk detections  
→ Wazuh  
→ Shuffle  
→ TheHive Alert API

#### Direct Wazuh branch

Wazuh  
→ JSON webhook  
→ Shuffle  
→ TheHive Alert API

#### Enrichment layer

TheHive  
→ Cortex analyzers

High-signal telemetry from multiple sources is normalized and routed into TheHive through controlled orchestration layers, rather than allowing every platform to create cases independently.

A key design principle in this project was:

**Decision logic belongs in the orchestration layer, not in the case platform itself.**

---

## Core Components Implemented

## 1. Sentinel to TheHive Integration

The first major workflow connected Microsoft Sentinel incidents to TheHive using Azure Logic Apps.

### Trigger Used

- Microsoft Sentinel — When an incident is created or updated

This trigger supported:

- scheduled analytics rules
- NRT rules
- Defender-derived incidents
- incident-centric automation

---

## 2. Severity-Based Routing Logic

A condition was added in the Logic App to decide whether a Sentinel incident should become a **TheHive case** or a **TheHive alert**.

### Routing model

- **High / Critical** → create **Case** in TheHive
- **Medium / Low / Informational** → create **Alert** in TheHive

This prevented analyst overload and aligned with real Tier 1 SOC triage models.

---

## 3. TheHive Case Creation Path

For High and Critical incidents, the Logic App created a case in TheHive using the TheHive Case API.

### Design characteristics

- native case schema used
- structured metadata included
- case template support referenced where applicable
- severity normalized on the TheHive side
- tagging used to preserve source and automation context

This preserved a clean escalation path for truly important incidents.

---

## 4. TheHive Alert Creation Path

For Medium, Low, and Informational incidents, the Logic App created alerts in TheHive instead of cases.

This preserved visibility without generating unnecessary formal case objects.

This also allowed later analyst-driven promotion if an alert turned out to warrant investigation.

---

## 5. Logic App Execution Reliability

A critical implementation detail involved making sure the Logic App continued collecting evidence and metadata regardless of which branch succeeded or failed.

### Run-after design

Post-condition actions were configured to continue after:

- succeeded
- failed
- skipped
- timed out

This is a production-grade SOAR pattern and prevents silent workflow termination.

---

## Security Onion and Wazuh Alert Pipeline

## 6. Security Onion Integration Through Wazuh

In addition to direct Wazuh endpoint alerts, selected high-risk detections from Security Onion were routed into the same workflow by forwarding those signals into Wazuh first, then sending them onward to Shuffle and TheHive.

This created a unified internal alerting path:

**Security Onion → Wazuh → Shuffle → TheHive**

### Why this design mattered

- avoided exposing Security Onion directly to downstream case-management workflows
- allowed Wazuh to act as an intermediate alert normalization layer
- preserved a single orchestration path into Shuffle
- reduced fragmentation between endpoint and network-driven alerts
- made it easier to consolidate high-signal detections into TheHive for analyst triage

---

## 7. Wazuh Alert Ingestion Objective

The second major workflow focused on forwarding Wazuh alerts into TheHive using Shuffle as an internal SOAR layer.

### Design principles

- Wazuh should not write directly to TheHive
- Shuffle should serve as the orchestrator
- TheHive should receive alerts, not immediate cases
- deduplication should be preserved
- backend API communication should stay internal

---

## 8. Final Internal SOC Branch Architecture

Final workflow model:

**Security Onion → selected high-risk alerts → Wazuh → JSON webhook → Shuffle (self-hosted Orborus runtime) → HTTP POST → TheHive Alert API**

Wazuh endpoint detections also followed the same downstream pattern into Shuffle and TheHive.

### Operational characteristics

- Shuffle is the only orchestrator in this branch
- TheHive receives alerts first
- Cloudflare exposure is for UI access, not backend API traffic
- all backend integration remains internal to the SOC subnet
- Security Onion network detections and Wazuh endpoint detections converge into a shared analyst workflow

---

## Runtime Troubleshooting and Remediation

## 9. Initial Runtime Problem

Although API design and authentication appeared correct, the Shuffle workflow initially failed in a misleading way.

### Symptoms observed

- HTTP actions hung indefinitely
- executions were aborted by the cleanup bot
- direct `curl` and `wget` from the host succeeded
- API calls from within Shuffle failed

At first, the issue looked like:

- bad API authentication
- Cloudflare interference
- TheHive API failure
- TLS problems

---

## 10. Root Cause: Orborus Proxy Inheritance

The actual cause was that the Shuffle Orborus runtime inherited a default HTTP proxy setting.

### Effect of the bad proxy state

- outbound HTTP traffic from Shuffle was routed to a non-existent proxy
- API calls hung
- execution behavior failed
- cleanup termination occurred

This was an important lesson:

**workflow logic can be correct while the automation runtime itself is broken.**

---

## 11. Runtime Remediation

The Orborus container was recreated with proxy environment variables explicitly disabled.

### Remediation outcome

- no invalid upstream proxy
- correct internal east-west API communication
- successful HTTP actions to TheHive
- stable execution path

---

## 12. Direct API Validation Before SOAR Execution

Before trusting Shuffle again, the TheHive Alert API was validated directly using a manual `curl` request.

### This confirmed

- TheHive API was reachable
- API credentials were valid
- the internal network path was correct
- the failure was not caused by TheHive

This is an important troubleshooting pattern for SOAR work: validate the API outside the orchestration engine first.

---

## 13. Final Shuffle Workflow Design

The final Shuffle workflow was intentionally minimal.

### Workflow model

Webhook  
→ HTTP POST to TheHive Alert API

### Design characteristics

- `sourceRef` used for deduplication
- mapped execution fields used instead of raw pasted JSON
- alert metadata normalized for TheHive
- source and tagging preserved for later analyst context

This kept the workflow clean and easier to troubleshoot.

---

## TLS and Internal Trust Decision

For some east-west internal SOAR communications, certificate verification was relaxed intentionally due to the environment being a tightly controlled internal SOC subnet.

This was not an accidental weakness. It was a documented risk decision.

### Security reasoning

- TLS encryption remained in place
- strict certificate verification was relaxed in a fully controlled private environment
- systems were inside the SOC VLAN
- segmentation and monitoring reduced exposure
- the trust decision was intentional and documented

This reflects a realistic lab-to-production boundary decision rather than a random shortcut.

---

## Cortex Role in the Architecture

Cortex was positioned as the enrichment engine, not the routing engine.

### Design principle

- routing belongs in Logic Apps, Shuffle, and orchestration logic
- enrichment belongs in Cortex analyzers
- post-alert and post-case workflows can later invoke enrichment cleanly

This kept the design modular and operationally sane.

---

## Correlation Value and Future Expansion

The architecture was intentionally designed to support correlation across:

- cloud detections from Sentinel
- endpoint detections from Wazuh
- network detections from Security Onion

By routing these sources into TheHive through controlled workflows, the project created a path toward:

- shared alert triage
- later enrichment with Cortex
- analyst-driven escalation
- future case correlation across telemetry types

This is what makes the design a **multi-source SOAR integration project**, not just a set of disconnected point-to-point integrations.

---

## Validation & Results

### Sentinel Branch Validation

- Sentinel incidents were created
- severity routing worked
- High / Critical incidents created cases
- lower-severity incidents created alerts
- Logic App executions completed successfully
- duplicate object sprawl was avoided by design

### Internal SOC Branch Validation

- Wazuh alerts reached Shuffle
- Shuffle created TheHive alerts successfully
- HTTP POST returned successful creation behavior
- alerts appeared in TheHive
- deduplication worked as intended
- reruns did not create unnecessary duplicates
- Security Onion high-risk alerts could be routed into the same downstream workflow through Wazuh

### Operational Validation

The combined design proved that:

- case routing logic works
- internal API orchestration works
- runtime debugging matters as much as logic design
- TheHive can act as the central analyst workspace
- multiple telemetry sources can feed a common triage layer without collapsing into noise

---

## Challenges Encountered

- routing logic complexity: deciding what becomes a case and what remains an alert
- avoiding duplication while preserving analyst usability
- distinguishing runtime failures from logic failures
- keeping tool roles separate instead of letting one platform do everything

A mature design required resisting the temptation to let one tool do everything.

### Final role separation

- Sentinel detects and generates incidents
- Security Onion generates high-risk network detections
- Wazuh acts as endpoint telemetry and an intermediate alerting layer
- Shuffle orchestrates internal alert forwarding
- Logic Apps orchestrate Microsoft-native flows
- TheHive stores and manages analyst objects
- Cortex enriches

---

## Security and Operational Value

This project demonstrates practical capability in:

- SOAR pipeline design
- cross-platform alert routing
- Sentinel-to-TheHive integration
- Security Onion-to-Wazuh-to-TheHive integration
- Wazuh-to-TheHive orchestration
- Logic App condition design
- Shuffle runtime troubleshooting
- API validation and testing
- alert-vs-case triage strategy
- deduplication design
- controlled internal trust decisions
- modular SOC workflow architecture

Operationally, this project matters because it moves beyond isolated tooling and creates an integrated analyst workflow model. Instead of forcing analysts to jump between multiple platforms with duplicated signals, the design centralizes triage and case handling while preserving separation between detection and orchestration.

---

## Tools & Technologies

- Microsoft Sentinel
- Azure Logic Apps
- Security Onion
- Wazuh
- Shuffle
- TheHive
- Cortex
- Cloudflare-secured TheHive UI exposure
- HTTP APIs
- webhook ingestion
- internal TLS / self-signed lab trust decisions

---

## Skills Demonstrated

- SOAR architecture design
- SIEM-to-case-platform integration
- workflow routing logic
- Logic App implementation
- Shuffle orchestration
- API troubleshooting
- deduplication planning
- analyst workflow design
- cross-platform integration
- enterprise-style documentation

---

## Outcome

This project successfully established a multi-source SOAR integration model in which Microsoft Sentinel, Security Onion, and Wazuh can all feed TheHive through appropriate orchestration layers, while preserving alert-first triage, severity-based escalation, deduplication, and modular enrichment design.

It also documented key operational lessons around runtime troubleshooting, internal API design, and the separation of detection, orchestration, and case management responsibilities.

The resulting architecture is extensible, operationally realistic, and well positioned for future enrichment, correlation, and response automation across cloud, endpoint, and network telemetry sources.
