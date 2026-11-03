# Perform Threat Hunting in Microsoft Sentinel

Hands-on threat hunting lab focused on proactively identifying Command and Control (C2) activity, creating hunting queries, investigating relationships through Microsoft Sentinel Hunting Graph, executing KQL jobs against the Data Lake, mapping hunting coverage to MITRE ATT&CK, and using Jupyter Notebooks with Visual Studio Code for advanced security analysis.

This lab was completed as part of the **Microsoft Security Operations Analyst (SC-200)** learning path.

---

## Objective

Develop practical experience with proactive threat hunting in Microsoft Sentinel by:

* Creating KQL hunting queries
* Hunting for PowerShell-based C2 activity
* Creating incidents directly from hunting results
* Using entity mapping during threat hunting
* Exploring relationships through Hunting Graph
* Running KQL jobs against the Microsoft Sentinel Data Lake
* Creating custom hunting tables
* Reviewing MITRE ATT&CK hunting coverage
* Creating and executing multi-query hunts
* Working with Microsoft Sentinel Data Lake Notebooks
* Integrating Microsoft Sentinel with Visual Studio Code
* Using Jupyter Notebooks for security analysis
* Exploring Microsoft Sentinel data through the MCP integration
* Using GitHub Copilot to assist with threat-hunting workflows

---

# Environment

| Component                    | Purpose                                     |
| ---------------------------- | ------------------------------------------- |
| Microsoft Sentinel           | SIEM and threat hunting                     |
| Microsoft Defender XDR       | Advanced hunting and incident investigation |
| Microsoft Sentinel Data Lake | Large-scale security data exploration       |
| KQL                          | Threat hunting and data analysis            |
| Azure Arc                    | Connect non-Azure systems                   |
| Azure Monitor Agent          | Windows telemetry collection                |
| SecurityEvent                | Windows security event data                 |
| Hunting Graph                | Relationship and attack-path exploration    |
| MITRE ATT&CK                 | Threat hunting and coverage mapping         |
| Visual Studio Code           | Notebook development environment            |
| Jupyter Notebooks            | Advanced security analysis                  |
| Microsoft Sentinel MCP       | Data Lake interaction from VS Code          |
| GitHub Copilot               | AI-assisted analysis and query development  |

---

# 1. Threat Hunting Preparation

The lab environment used a Windows server connected to Microsoft Sentinel through **Azure Arc** and the **Windows Security Events via AMA** data connector.

The Windows server was configured to send security telemetry to Sentinel through a Data Collection Rule collecting:

```text
All Security Events
```

This telemetry was later used as the primary source for threat hunting.

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
    ├── SecurityEvent
    ├── Advanced Hunting
    ├── Data Lake
    └── Threat Hunting
```

---

# 2. Command and Control Simulation

A controlled DNS-based Command and Control simulation was executed on the Windows server.

The simulation generated randomized DNS TXT queries with variable timing and subdomains in order to produce telemetry representative of DNS-based C2 activity.

The generated events were intentionally used as hunting data rather than representing a real-world compromise.

### Threat behavior simulated

```text
PowerShell
    │
    ▼
DNS TXT Requests
    │
    ▼
Randomized Subdomains
    │
    ▼
C2-like DNS Traffic
    │
    ▼
Security Telemetry
    │
    ▼
Threat Hunting
```

The generated telemetry was subsequently queried through Microsoft Sentinel.

---

# 3. PowerShell C2 Hunting Query

The first hunting exercise focused on identifying PowerShell processes and their command-line parameters.

The following KQL query was used:

```kql
let lookback = 2d;
SecurityEvent
| where TimeGenerated >= ago(lookback)
| where EventID == 4688 and Process =~ "powershell.exe"
| extend PwshParam = trim(@"[^/\\]*powershell(.exe)+" , CommandLine)
| project TimeGenerated, Computer, SubjectUserName, PwshParam
```

### Detection logic

The query searches for:

* Windows process creation events
* Event ID `4688`
* PowerShell execution
* PowerShell command-line parameters
* Activity occurring within the previous two days

The results were reviewed to identify the simulated execution of:

```text
-file c2.ps1
```

This allowed the simulated C2 activity to be identified through process telemetry.

---

# 4. Creating an Incident from Hunting Results

The identified PowerShell C2 activity was linked directly to a Microsoft Defender XDR incident.

The hunting result was converted into an incident with the following characteristics:

| Setting         | Configuration                               |
| --------------- | ------------------------------------------- |
| Alert title     | PowerShell C2 Hunt                          |
| Severity        | High                                        |
| Category        | Command and Control                         |
| MITRE technique | T1094 — Custom Command and Control Protocol |
| Impacted asset  | Device / Hostname                           |
| Source          | Advanced Hunting                            |

Entity mapping was configured so that the affected device could be associated with the incident.

### Hunting-to-Incident workflow

```text
SecurityEvent
      │
      ▼
KQL Hunting Query
      │
      ▼
Suspicious Result
      │
      ▼
Link to Incident
      │
      ▼
Entity Mapping
      │
      ▼
Incident Investigation
```

This demonstrated how proactive hunting can transition into the standard SOC incident-response workflow.

---

# 5. Microsoft Sentinel Hunting Graph

The **Hunting Graph** was used to investigate relationships between users, resources, and security entities.

The predefined scenario:

```text
Users with access to Sensitive Data
```

was used to investigate access relationships involving a sensitive storage account.

The graph was used to identify:

* User accounts
* Storage resources
* Relationships between entities
* Critical users
* Discovery sources
* Defender for Cloud relationships

The graph also allowed individual entities to be expanded to reveal additional relationships.

### Investigation model

```text
User
 │
 ├── Access
 │
 ▼
Sensitive Storage Account
 │
 ├── Discovery Source
 ├── Resource Details
 └── Related Entities
```

This provided practical experience with graph-based security investigation rather than relying exclusively on tabular query results.

---

# 6. Data Lake KQL Jobs

The lab introduced **KQL jobs** for running queries against Microsoft Sentinel Data Lake data.

A KQL job was created to search for the same PowerShell-based C2 behavior identified during the initial hunt.

The query was extended to aggregate the results:

```kql
let lookback = 2d;
SecurityEvent
| where TimeGenerated >= ago(lookback)
| where EventID == 4688 and Process =~ "powershell.exe"
| extend PwshParam = trim(@"[^/\\]*powershell(.exe)+" , CommandLine)
| project TimeGenerated, Computer, SubjectUserName, PwshParam
| summarize min(TimeGenerated), count()
    by Computer, SubjectUserName, PwshParam
```

### KQL Job configuration

The job was configured to:

* Run against the Sentinel Data Lake
* Execute once
* Write results to an Analytics-tier custom table
* Process the hunting query
* Make the resulting data available for further investigation

The generated table followed the Microsoft Sentinel KQL custom table convention:

```text
C2ATTACKHUNT_<identifier>_KQL_CL
```

After the job completed successfully, the resulting table was queried through Advanced Hunting.

---

# 7. MITRE ATT&CK Hunting Coverage

The MITRE ATT&CK section of Microsoft Sentinel was used to evaluate available hunting coverage.

The lab demonstrated how MITRE ATT&CK can be used to identify:

* Techniques with hunting queries
* Simulated coverage
* Existing hunting content
* Potential detection or hunting gaps

The **Account Manipulation** technique was selected as an example.

Associated hunting queries were reviewed and cloned into a new hunt.

### Hunt workflow

```text
MITRE ATT&CK Technique
          │
          ▼
Available Hunting Queries
          │
          ▼
Select Queries
          │
          ▼
Create Hunt
          │
          ▼
Define Hypothesis
          │
          ▼
Run Queries
          │
          ▼
Review Results
          │
          ▼
Validate / Invalidate Hypothesis
```

This demonstrated the concept of **hypothesis-driven threat hunting**, where a hunter defines an assumption about potentially malicious behavior and uses multiple queries to validate or invalidate it.

---

# 8. Hunting with Data Lake Notebooks

The second exercise introduced Microsoft Sentinel Data Lake Notebooks.

The lab demonstrated how notebooks can extend Sentinel's native hunting capabilities by combining:

* KQL
* Python
* Jupyter Notebooks
* Data visualization
* External data sources
* Machine learning techniques

Visual Studio Code was used as the notebook development environment.

---

# 9. Visual Studio Code Integration

The following Microsoft-published extensions were installed and configured:

* Python
* Jupyter
* GitHub Copilot
* Microsoft Sentinel

This provided an environment for interacting with Sentinel Data Lake data directly from Visual Studio Code.

### Development environment

```text
Visual Studio Code
       │
       ├── Python
       ├── Jupyter
       ├── GitHub Copilot
       └── Microsoft Sentinel
              │
              ▼
       Sentinel Data Lake
              │
              ▼
         SecurityEvent
```

---

# 10. Microsoft Sentinel MCP Integration

The lab also introduced the **Microsoft Sentinel MCP integration** for Data Lake exploration.

The MCP server was configured in Visual Studio Code using the Sentinel Data Exploration endpoint.

This allowed the development environment to interact with Sentinel data through natural-language prompts and assisted query development.

Example hunting prompts included:

```text
Which tables are good to use for hunting malicious activities on devices?
```

```text
What columns within the SecurityEvent table are useful in hunting queries?
```

```text
Query the last 90 days of SecurityEvent data and summarize the top malicious activities.
```

The workflow demonstrated how natural-language interaction can assist security analysts in discovering relevant data sources, fields, and hunting approaches.

---

# 11. Jupyter Notebook Threat Hunting

The lab demonstrated the creation of a Jupyter Notebook based on findings and suggestions generated during the hunting workflow.

The notebook contained a combination of:

* Markdown cells
* Python code cells
* Security analysis
* Query results
* Hunting observations

The Microsoft Sentinel extension was also used to browse the available Data Lake tables.

The `SecurityEvent` schema was inspected directly from Visual Studio Code.

A Microsoft Sentinel sample notebook was also reviewed:

```text
01_GettingStartedwithSentineldatalake
```

This provided an introduction to working with Sentinel Data Lake data through notebooks.

---

# 12. Advanced Threat Hunting Workflow

The complete workflow practiced during this lab can be summarized as:

```text
Threat Intelligence
        │
        ▼
Hunting Hypothesis
        │
        ▼
KQL Query
        │
        ├───────────────┐
        ▼               ▼
Advanced Hunting    Data Lake
        │               │
        ▼               ▼
Hunting Results     KQL Job
        │               │
        └───────┬───────┘
                ▼
          Investigation
                │
        ┌───────┴────────┐
        ▼                ▼
     Incident         Hunt Graph
        │                │
        └───────┬────────┘
                ▼
        Hypothesis Validation
                │
        ┌───────┴────────┐
        ▼                ▼
     Confirmed         Invalidated
        │
        ▼
   Detection / Response
```

---

# 13. Key KQL Techniques Practiced

The lab reinforced several KQL techniques relevant to threat hunting:

### Time-based filtering

```kql
| where TimeGenerated >= ago(2d)
```

### Event filtering

```kql
| where EventID == 4688
```

### Case-insensitive process matching

```kql
| where Process =~ "powershell.exe"
```

### Command-line extraction

```kql
| extend PwshParam = trim(
    @"[^/\\]*powershell(.exe)+",
    CommandLine
)
```

### Result projection

```kql
| project
    TimeGenerated,
    Computer,
    SubjectUserName,
    PwshParam
```

### Aggregation

```kql
| summarize
    min(TimeGenerated),
    count()
    by Computer, SubjectUserName, PwshParam
```

These operations were combined to transform raw Windows telemetry into actionable hunting results.

---

# Technologies Practiced

* Microsoft Sentinel
* Microsoft Defender XDR
* Microsoft Sentinel Data Lake
* Advanced Hunting
* KQL
* KQL Jobs
* Hunting Graph
* MITRE ATT&CK
* Azure Arc
* Azure Monitor Agent
* Data Collection Rules
* Windows Security Events
* `SecurityEvent`
* Jupyter Notebooks
* Visual Studio Code
* Python
* Microsoft Sentinel MCP
* GitHub Copilot
* Threat Intelligence
* Command and Control Hunting
* Hypothesis-driven Threat Hunting

---

# Key Security Concepts

| Concept            | Application                                                   |
| ------------------ | ------------------------------------------------------------- |
| Threat Hunting     | Proactive investigation of suspicious behavior                |
| C2 Detection       | Hunting PowerShell and DNS-based command-and-control activity |
| Process Creation   | Event ID 4688 analysis                                        |
| KQL                | Query and transformation of security telemetry                |
| MITRE ATT&CK       | Technique mapping and hunting coverage                        |
| Hunting Graph      | Relationship-based investigation                              |
| Data Lake          | Large-scale security data exploration                         |
| KQL Jobs           | Scheduled/one-time processing of Data Lake data               |
| Notebooks          | Advanced analysis and visualization                           |
| MCP                | Natural-language interaction with Sentinel data               |
| Hypothesis Hunting | Validate or invalidate security assumptions                   |

---

# Result

Completed **Learning Path 09 — Perform Threat Hunting in Microsoft Sentinel**, covering both advanced hunting with Microsoft Sentinel and Data Lake Notebook-based hunting.

The lab provided hands-on experience with the complete proactive threat-hunting lifecycle:

**Threat Intelligence → Hypothesis → KQL → Hunt → Investigation → Validation → Detection/Response**

The exercises covered PowerShell-based C2 hunting, incident creation from hunting results, graph-based investigation, Data Lake KQL jobs, MITRE ATT&CK hunting coverage, Jupyter Notebooks, Visual Studio Code, Microsoft Sentinel MCP integration, and AI-assisted security analysis.

This lab expanded the practical use of Microsoft Sentinel beyond traditional SIEM detection by demonstrating how analysts can **proactively search for threats, investigate relationships, analyze large security datasets, and develop repeatable hunting workflows**.
