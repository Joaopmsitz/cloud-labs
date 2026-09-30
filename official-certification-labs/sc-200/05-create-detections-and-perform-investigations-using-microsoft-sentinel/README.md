# Create Detections and Perform Investigations using Microsoft Sentinel

Hands-on lab focused on creating detections, automating incident response, simulating attacks, investigating security incidents, applying UEBA and ASIM, building workbooks, and integrating Microsoft Sentinel with Azure DevOps.

This lab was completed as part of the **Microsoft Security Operations Analyst (SC-200)** learning path.

---

## Objective

Develop practical experience with the Microsoft Sentinel detection and investigation lifecycle:

* Create and configure **SOAR Playbooks**
* Automate incident response with **Automation Rules**
* Create **Scheduled Analytics Rules**
* Write and refine **KQL detection queries**
* Configure **Entity and User Behavior Analytics (UEBA)**
* Work with **Anomaly Analytics Rules**
* Connect Windows systems through **Azure Arc** and **Azure Monitor Agent (AMA)**
* Simulate persistence, privilege escalation, and DNS-based C2 activity
* Create custom detections based on Windows security events
* Investigate incidents and their related entities
* Use **ASIM parsers** for normalized security data
* Build and customize **Microsoft Sentinel Workbooks**
* Export and deploy analytics content through **Azure DevOps**

---

## Environment

| Component                 | Purpose                                      |
| ------------------------- | -------------------------------------------- |
| Microsoft Sentinel        | SIEM and security analytics                  |
| Microsoft Defender XDR    | Security operations and investigation portal |
| Azure Monitor Agent (AMA) | Security event collection                    |
| Azure Arc                 | Connect non-Azure Windows systems            |
| Log Analytics             | Security event storage and querying          |
| KQL                       | Detection and investigation queries          |
| Logic Apps                | Playbook automation                          |
| UEBA                      | Entity behavior analysis                     |
| ASIM                      | Security event normalization                 |
| Azure Workbooks           | Security visualization and dashboards        |
| Azure DevOps              | Analytics rule source control and deployment |

---

# 1. Playbooks and Automation

The first stage focused on implementing automated response capabilities in Microsoft Sentinel.

### Playbook

A Logic App was deployed as a Microsoft Sentinel playbook using the **Sentinel SOAR Essentials** solution.

The playbook was configured to support incident-response workflows and was connected to the appropriate Microsoft Sentinel permissions.

### Automation Rule

An automation rule was created to trigger when a new incident is generated.

The rule evaluates incident tactics including:

* Reconnaissance
* Execution
* Persistence
* Command and Control
* Exfiltration
* Pre-Attack

When the defined conditions are met, the automation rule executes the ransomware-response playbook.

### Concepts practiced

* Microsoft Sentinel SOAR
* Logic Apps
* Playbooks
* Automation Rules
* Incident-triggered automation
* Security orchestration

---

# 2. Scheduled Analytics Rules

A scheduled analytics rule was created from the **New CloudShell User** template.

The rule was configured with:

| Setting         | Configuration   |
| --------------- | --------------- |
| Severity        | Medium          |
| Query frequency | Every 5 minutes |
| Lookback period | 1 day           |
| Event grouping  | Single alert    |

The rule was also configured with an automation rule that assigns generated incidents to the appropriate SOC analyst.

### Investigation workflow

The rule was designed to detect Azure Activity events generated when Cloud Shell resources are provisioned.

The relevant Activity Log operations included:

* `List Storage Account Keys`
* `Update Storage Account Create`

This demonstrated how Azure Activity data can be used as an input for Sentinel analytics and incident generation.

---

# 3. UEBA and Anomaly Detection

Microsoft Sentinel **User and Entity Behavior Analytics (UEBA)** was reviewed and verified as enabled.

The connected data sources were reviewed to understand how Sentinel uses behavioral information to identify anomalous activity involving users and entities.

### Anomaly Analytics

The anomaly analytics configuration was also explored.

The lab demonstrated the distinction between:

* **Production** anomaly rules
* **Flighting** anomaly rules

Flighting rules allow anomaly detection parameters such as the anomaly score threshold to be tested before being promoted to production.

A Flighting rule was duplicated and its anomaly threshold was modified for demonstration purposes before being removed as part of the lab cleanup.

### Concepts practiced

* UEBA
* Behavioral analytics
* Anomaly detection
* Production rules
* Flighting rules
* Anomaly score thresholds

---

# 4. Detection Preparation

Before creating custom detections, a Windows server was connected to Azure using **Azure Arc**.

The Azure Arc agent was validated using:

```cmd
azcmagent show
```

The machine was then connected to Microsoft Sentinel through the **Windows Security Events via AMA** data connector.

## Data Collection Rule

A Data Collection Rule (DCR) was created and associated with the Windows server.

The collection configuration was:

```text
All Security Events
```

This allowed Windows security events to be ingested into the Sentinel environment and queried through KQL.

### Architecture

```text
WINServer
    │
    ▼
Azure Arc
    │
    ▼
Azure Monitor Agent
    │
    ▼
Data Collection Rule
    │
    ▼
Microsoft Sentinel
    │
    ▼
SecurityEvent
```

---

# 5. Simulated Attack Scenarios

Three attack techniques were simulated on the connected Windows server to generate telemetry for detection engineering.

## 5.1 Registry Run Key Persistence

A Registry Run Key was created under:

```text
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

The technique simulated an attacker establishing persistence by configuring a program to execute when the user logs in.

### MITRE ATT&CK

```text
T1547.001 — Registry Run Keys / Startup Folder
```

---

## 5.2 Privilege Escalation

A new local user was created and added to the local Administrators group.

The simulation generated Windows security events associated with security group membership changes.

The relevant event was:

```text
Event ID: 4732
A member was added to a security-enabled local group
```

This telemetry was later used to build a detection for unauthorized additions to the local Administrators group.

---

## 5.3 DNS Command and Control Simulation

A PowerShell script was executed to generate a high volume of randomized DNS queries.

The objective was to simulate characteristics of DNS-based command-and-control traffic and generate telemetry for later threat-hunting activities.

The script remained running in the background so that the generated events could be used by subsequent security investigations.

---

# 6. Detection Engineering with KQL

The next stage focused on transforming raw security telemetry into actionable detections.

## 6.1 Registry Persistence Detection

The initial investigation searched the available security data for the persistence artifact:

```kql
search "temp\\startup.bat"
```

After identifying the relevant telemetry in the `SecurityEvent` table, the query was refined to identify execution of `reg.exe`:

```kql
SecurityEvent
| where Activity startswith "4688"
| where Process == "reg.exe"
| where CommandLine startswith "REG"
```

The query was then enhanced with entity information:

```kql
SecurityEvent
| where Activity startswith "4688"
| where Process == "reg.exe"
| where CommandLine startswith "REG"
| extend
    timestamp = TimeGenerated,
    HostCustomEntity = Computer,
    AccountCustomEntity = SubjectUserName
```

### Detection configuration

| Setting        | Configuration             |
| -------------- | ------------------------- |
| Detection type | Scheduled / NRT detection |
| Category       | Persistence               |
| Severity       | High                      |
| Entity         | Device                    |
| Identifier     | Hostname                  |
| Source         | `SecurityEvent`           |

Automated remediation actions were also configured to support investigation of the affected device.

---

# 7. Privilege Escalation Detection

The second detection focused on identifying accounts added to the local Administrators group.

The initial investigation searched for administrator-related events:

```kql
search "administrators"
| summarize count() by $table
```

The investigation was then narrowed to Event ID `4732`:

```kql
SecurityEvent
| where EventID == 4732
| where TargetAccount == "Builtin\\Administrators"
```

Because the event contains the added account's SID rather than directly exposing the username, a join was used to correlate the SID with the corresponding account:

```kql
SecurityEvent
| where EventID == 4732
| where TargetAccount == "Builtin\\Administrators"
| extend Acct = MemberSid, MachId = SourceComputerId
| join kind=leftouter (
    SecurityEvent
    | summarize count() by TargetSid, SourceComputerId, TargetUserName
    | project
        Acct1 = TargetSid,
        MachId1 = SourceComputerId,
        UserName1 = TargetUserName
) on $left.MachId == $right.MachId1, $left.Acct == $right.Acct1
| extend
    timestamp = TimeGenerated,
    HostCustomEntity = Computer,
    AccountCustomEntity = UserName1
```

### Detection configuration

| Setting         | Configuration        |
| --------------- | -------------------- |
| Detection type  | Analytics Rule       |
| Severity        | High                 |
| MITRE ATT&CK    | Privilege Escalation |
| Event ID        | 4732                 |
| Entity          | Account              |
| Entity          | Host                 |
| Query frequency | Every 5 minutes      |
| Lookback        | 1 day                |

An automation rule was also configured to execute the previously created Sentinel playbook when the incident is created.

---

# 8. Incident Investigation

After the detections generated incidents, the investigation workflow was performed directly in Microsoft Defender XDR.

The investigation included:

* Reviewing generated incidents
* Checking severity and status
* Assigning incidents to an analyst
* Creating incident tags
* Reviewing the Attack Story
* Executing available playbooks
* Reviewing mapped entities
* Creating incident tasks
* Reviewing the Activity Log
* Opening the investigation graph
* Investigating related alerts
* Reviewing entity timelines
* Reviewing related entities and alerts
* Classifying the incident

The investigation graph was used to correlate the affected host with related security activity.

The incident was ultimately classified as:

```text
True positive - suspicious activity
```

### Investigation workflow

```text
Detection
   │
   ▼
Alert
   │
   ▼
Incident
   │
   ├── Alerts
   ├── Entities
   ├── Attack Story
   └── Activity
          │
          ▼
   Investigation Graph
          │
          ▼
   Classification
```

---

# 9. ASIM Registry Event Parsers

The lab also introduced the **Advanced Security Information Model (ASIM)**.

The Registry Event parser for Microsoft Windows was located through the Advanced Hunting schema and its KQL implementation was reviewed.

The parser follows the Registry Event normalization schema and provides a normalized way to query Windows registry activity.

The parser used in the lab followed the versioned naming convention:

```text
_Im_RegistryEvent_MicrosoftWindowsEvent*
```

The parser was loaded into the Advanced Hunting editor and executed to validate the normalized results.

The exercise focused on Windows Registry Event **Event ID 4657**.

### Concepts practiced

* ASIM
* Normalized security data
* Registry Event schema
* Parser functions
* KQL abstraction
* Event ID 4657

---

# 10. Microsoft Sentinel Workbooks

The lab introduced Sentinel Workbooks for security visualization and operational monitoring.

## Azure Activity Workbook

The built-in **Azure Activity** workbook template was explored and saved as a custom workbook.

The visualization for the Activities column was modified to use a:

```text
Heatmap
```

This demonstrated how existing Sentinel workbook templates can be adapted to improve data visualization.

## Custom Workbook

A new workbook was also created from scratch.

The workbook included:

* Custom Markdown title
* `SecurityEvent` queries
* Time chart visualization
* Grid visualization
* Custom item widths
* Refresh configuration

The final layout used:

```text
25% — Time chart
75% — SecurityEvent grid
```

This provided practical experience creating dashboards for security monitoring.

---

# 11. Azure DevOps Integration

The final exercise focused on managing Sentinel content through source control.

## Export Analytics Rule

The previously created analytics rule was exported from Microsoft Sentinel as:

```text
Azure_Sentinel_analytic_rule.json
```

The exported file was reviewed as an Azure Resource Manager template.

## Azure DevOps Repository

An Azure DevOps project and repository were created to centralize Sentinel content.

The exported analytics rule was uploaded to the repository and committed to the `main` branch.

## Microsoft Sentinel Repository Integration

Microsoft Sentinel was then connected to the Azure DevOps repository through:

```text
Microsoft Sentinel
        │
        ▼
Repositories
        │
        ▼
Azure DevOps
        │
        ▼
Repository / main
        │
        ▼
Sentinel Content Deployment
```

All available content types were selected and the deployment status was verified as successful.

The repository connection was then removed as part of the lab cleanup.

---

# Technologies Practiced

* Microsoft Sentinel
* Microsoft Defender XDR
* Microsoft Defender for Cloud
* Azure Monitor Agent
* Azure Arc
* Log Analytics
* KQL
* Logic Apps
* Sentinel Playbooks
* Automation Rules
* Analytics Rules
* Scheduled Queries
* UEBA
* Anomaly Detection
* ASIM
* Azure Workbooks
* Azure Activity Logs
* Windows Security Events
* Azure DevOps
* MITRE ATT&CK

---

# Key Tables and Data Sources

| Table / Source                  | Purpose                                            |
| ------------------------------- | -------------------------------------------------- |
| `SecurityEvent`                 | Windows security event detection and investigation |
| Azure Activity                  | Azure resource and subscription activity           |
| Windows Security Events via AMA | Windows telemetry collection                       |
| Registry Event ASIM parser      | Normalized registry activity                       |
| Microsoft Sentinel Incidents    | Investigation and response workflow                |

---

# Detection Lifecycle

The complete workflow practiced in this lab can be summarized as:

```text
Data Collection
      │
      ▼
KQL Investigation
      │
      ▼
Detection Engineering
      │
      ▼
Analytics Rule
      │
      ▼
Alert
      │
      ▼
Incident
      │
      ├──────────────┐
      ▼              ▼
Entity Mapping    Automation
      │              │
      ▼              ▼
Investigation    Playbook
      │
      ▼
Attack Story / Investigation Graph
      │
      ▼
Classification
```

---

# Result

Completed the **Learning Path 8 — Create detections and perform investigations using Microsoft Sentinel** lab, covering the complete detection and investigation workflow in Microsoft Sentinel.

The hands-on activities included building automated response capabilities, creating KQL-based detections, simulating attack techniques, investigating generated incidents, mapping entities, using UEBA and ASIM, creating security workbooks, and integrating Sentinel analytics content with Azure DevOps.

The lab provided practical experience moving from **raw security telemetry to detection, incident investigation, automated response, and security visualization**.
