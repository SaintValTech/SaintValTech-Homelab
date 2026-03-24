# Project 3 — Microsoft Sentinel SOAR Pipeline & Incident Automation

## Objective

The objective of this project was to build and validate a Microsoft-native SOAR foundation that could turn security detections into incidents and trigger automated workflows through Azure Logic Apps.

This project was designed to prove that the SaintValTech environment could support:

- incident-driven automation in Microsoft Sentinel
- analytics-rule-based incident creation
- playbook execution through Azure Logic Apps
- managed identity-based permissions
- reliable Defender-to-Sentinel signal flow
- deterministic testing of SOAR behavior
- an operationally trustworthy Microsoft-native automation backbone

Before integrating third-party tooling such as TheHive, Shuffle, Wazuh, Security Onion, or Cortex, the goal was to confirm that the Microsoft security stack could create, route, and automate incidents correctly on its own.

---

## Environment Overview

This project was built inside the Microsoft security portion of the SaintValTech environment.

### Core Components

- Microsoft Sentinel
- Microsoft Defender XDR
- Microsoft Defender for Endpoint
- Microsoft Entra ID
- Azure Logic Apps
- Log Analytics Workspace: `LAW-Production-SOC`
- Resource Group: `RG-Production-SOC`

### Endpoint Sources Used During Testing

- macOS endpoint onboarded to Defender for Endpoint
- Windows Server endpoint previously onboarded and validated
- Microsoft-native connectors active in Sentinel

### Scope Constraint

This project intentionally focused on Microsoft-native signals only.

The validation phase did **not** depend on:

- Wazuh
- Security Onion
- TheHive
- Cortex
- Shuffle
- other third-party telemetry sources

This design decision was important because it isolated the Microsoft automation pipeline and ensured that any success or failure could be attributed to the Microsoft stack itself.

---

## Architecture Role

This project established the Microsoft-native detection-to-automation path for the SaintValTech SOC architecture.

### High-Level Flow

Endpoint telemetry  
→ Microsoft Defender XDR  
→ Microsoft Sentinel  
→ Analytics rule  
→ Incident creation  
→ Automation rule  
→ Azure Logic App playbook execution

This created the first fully validated SOAR backbone in the lab.

---

## Core Components Implemented

### 1. Defender for Endpoint Validation

Before automation could be trusted, endpoint telemetry had to be confirmed.

Validation activities included:

- reviewing Defender health on Windows Server
- validating endpoint telemetry on macOS
- confirming alerts in Microsoft Defender
- confirming visibility inside connected Microsoft tools

This ensured that the automation pipeline began with real signal generation rather than assumptions.

---

### 2. Sentinel Connector Review

Connected Microsoft security sources were reviewed to confirm ingestion and alert availability.

Examples included:

- Microsoft Defender for Endpoint
- Microsoft Defender XDR
- Microsoft Defender for Identity
- Microsoft Entra ID
- Microsoft Entra ID Protection
- Microsoft Defender for Cloud Apps
- Microsoft Defender Threat Intelligence
- Security Events where applicable

This step was important because SOAR cannot function cleanly if connectors are misread, incomplete, or assumed to be inactive.

---

### 3. Core Logic App Creation

A Sentinel-compatible Logic App was created to act as the automation target for incident workflows.

#### Configuration highlights

- Hosting model: Consumption
- Tenancy model: Multi-tenant
- Workflow type: Stateful
- Resource Group: `RG-Production-SOC`
- Region: Canada East
- Playbook name: `PB-Incident-Core-SOAR`
- Trigger: Microsoft Sentinel — When an incident is created
- Authentication model: Managed Identity

This created the execution engine for incident-driven automation.

---

### 4. Sentinel Playbook Permissions

A key issue during implementation involved Sentinel’s permission model for running playbooks.

Remediation included:

- removing existing resource group permission assignment
- re-adding permission for `RG-Production-SOC`
- validating playbook visibility and execution state
- confirming managed identity readiness

This was an important operational lesson because the Sentinel permission interface can be misleading. Actual execution success depends on resource permissions and managed identity configuration, not just what the UI appears to show.

---

### 5. Automation Rule Creation

An incident-driven automation rule was created to standardize what happens when a Sentinel incident is created.

#### Rule details

- Name: `AR-Incident-Core-SOAR`
- Trigger: When incident is created

#### Action flow

- add tags
- assign owner
- normalize severity where needed
- run playbook

A key lesson here was that action order matters. The playbook was intentionally placed last to preserve cleaner workflow execution.

---

### 6. Guaranteed Incident Rule Concept

Controlled analytics rules were created to remove uncertainty around whether Sentinel was actually capable of creating new incidents on demand.

This followed a deliberate operational principle:

**A SOAR pipeline should be validated with deterministic tests, not assumptions.**

This was especially important because alerts and incidents do not behave identically inside Sentinel.

---

### 7. Scheduled Guaranteed Incident Rule

An initial scheduled analytics rule was created around Defender-derived signals.

This confirmed that:

- alerts were being ingested
- rule logic was functioning
- incidents could be created through analytics logic

However, visual testing was complicated by:

- alert grouping
- incident reuse
- schedule windows
- delayed evaluation timing

This made the platform appear inconsistent at first, even though its behavior was normal.

---

### 8. Near Real-Time Rule Design

To remove scheduling ambiguity, the project moved to a Near Real-Time design.

#### Example rule characteristics

- Name: `NRT-Test-Immediate-Incident`
- Rule type: Near Real Time
- Query source: `SecurityAlert`
- Incident creation: enabled
- Re-open closed incidents: disabled

NRT rules proved far more effective for validation because they reduced waiting time and gave clearer feedback.

---

### 9. EICAR-Based Endpoint Validation

A controlled endpoint test was used to generate a safe, known detection signal.

#### Validation sequence

- Defender detected the EICAR test file
- an alert was created in Microsoft Defender
- Sentinel ingested the alert
- the NRT rule matched the event
- a Sentinel incident was created
- the automation rule triggered
- the playbook execution path was validated

This provided strong evidence that the end-to-end pipeline was operational.

---

## Validation and Results

Validation focused on proving the full Microsoft-native incident pipeline.

### Confirmed outcomes

- endpoint telemetry generated a valid detection
- Microsoft Defender created an alert
- Sentinel ingested the signal
- analytics rule logic matched the event
- a Sentinel incident was created
- the incident triggered the automation rule
- the playbook executed through the configured SOAR path

This confirmed that the core detection-to-automation workflow functioned correctly.

---

## Key Operational Lessons

### Alerts vs Incidents

One of the most important lessons from this project was the distinction between alerts and incidents.

Alerts do not always create fresh visible incidents.

Additional complexities included:

- incidents may be grouped
- repeated tests may enrich existing workflows
- analytics success is not always immediately obvious without reviewing the right history and logic

This was a critical architectural lesson because many SOAR misunderstandings come from confusing alert behavior with incident behavior.

---

### Grouping and Reuse Behavior

Sentinel’s grouping and incident reuse behavior initially made the environment appear unreliable.

In reality, the platform was behaving as designed.

The lesson was that:
- automation design must account for incident grouping behavior
- testing methodology matters as much as rule logic
- SOC operators must understand platform behavior before diagnosing failure

---

### Alert-Based vs Incident-Based SOAR

Another major design decision in this project was rejecting alert-based SOAR as the primary model.

Reasons included:

- higher duplication risk
- inconsistent operational clarity
- weaker alignment with how Sentinel is actually used in mature SOC workflows

The final decision was to adopt **incident-based SOAR** as the correct model.

This became a core design principle for later integrations.

---

## Security and Operational Value

This project demonstrates practical capability in:

- Microsoft Sentinel analytics rule creation
- SOAR workflow design
- Azure Logic App deployment
- incident-driven automation architecture
- Defender-to-Sentinel validation
- managed identity usage
- permission troubleshooting
- deterministic SOC pipeline testing
- incident lifecycle understanding
- Microsoft-native security operations design

Operationally, this project matters because it created a trusted automation backbone for the environment.

Without this proof, later integrations with TheHive, Wazuh, Security Onion, Shuffle, and Cortex would have been built on uncertainty.

---

## Tools and Technologies

- Microsoft Sentinel
- Microsoft Defender XDR
- Microsoft Defender for Endpoint
- Microsoft Entra ID
- Azure Logic Apps
- Managed Identity
- Log Analytics Workspace
- NRT analytics rules
- automation rules
- EICAR testing

---

## Skills Demonstrated

- SIEM analytics rule creation
- SOAR workflow design
- Sentinel incident automation
- Microsoft security stack integration
- Logic App deployment
- cloud permission troubleshooting
- incident lifecycle understanding
- controlled detection validation
- operational testing methodology
- SOC workflow design

---

## Outcome

This project successfully validated the Microsoft-native SOAR foundation of the SaintValTech SOC environment.

It proved that:

- incidents can be created reliably in Sentinel
- incidents can trigger automation rules
- playbooks can execute correctly through managed identity and Azure Logic Apps

More importantly, it established an incident-centric automation model that now serves as the trusted base for later integrations involving TheHive, Wazuh, Shuffle, Security Onion, Cortex, and multi-source case workflows.
