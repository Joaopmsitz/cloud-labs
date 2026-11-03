# 🛡️ SC-200 — Connect Logs to Microsoft Sentinel

## 🎯 Objective

Connect multiple data sources to Microsoft Sentinel using Microsoft Sentinel Data Connectors.

The lab focuses on collecting security and activity logs from Microsoft Defender for Cloud, Azure resources, Windows machines, Linux hosts, and Microsoft Defender XDR.

The exercises also provide hands-on experience with Azure Monitor Agent (AMA), Azure Arc, Data Collection Rules (DCRs), CEF, Syslog, and the integration between Microsoft Sentinel and Microsoft Defender XDR.

## 🏗️ Environment

| Component          | Configuration                   |
| ------------------ | ------------------------------- |
| Security Platform  | Microsoft Sentinel              |
| Security Portal    | Microsoft Defender XDR          |
| Cloud Platform     | Microsoft Azure                 |
| Agents             | Azure Monitor Agent (AMA)       |
| Hybrid Management  | Azure Arc                       |
| Windows Collection | Windows Security Events via AMA |
| Linux Collection   | CEF via AMA / Syslog via AMA    |
| Configuration      | Data Collection Rules (DCR)     |
| Lab Platform       | Microsoft Learn / Skillable     |

## ⚙️ Data Connectors

### 1. Microsoft Defender for Cloud

Accessed the **Microsoft Defender for Cloud** solution through the Microsoft Sentinel Content Hub.

The **Tenant-based Microsoft Defender for Cloud** Data Connector was reviewed and verified as connected.

The connector uses the:

```text
SecurityAlert
```

table for Microsoft Defender for Cloud alert data.

The lab also demonstrated that Microsoft Defender for Cloud alerts can be streamed through Microsoft Defender XDR.

---

### 2. Azure Activity

Configured and reviewed the **Azure Activity** Data Connector.

The connector was verified as connected and the `AzureActivity` table was reviewed through the table management configuration.

The lab also covered:

* Analytics tier data retention
* Azure Activity configuration
* UEBA configuration
* Analytics rules
* Hunting queries
* Workbook integration

UEBA was verified as enabled for Azure Activity.

---

## 🪟 Windows Log Collection

### 3. Azure Windows Virtual Machine

Created a Windows 11 Enterprise virtual machine in Azure and configured it as a log source for Microsoft Sentinel.

The VM was connected to the **Windows Security Events via AMA** Data Connector.

A **Data Collection Rule (DCR)** was created and associated with the Windows virtual machine.

Configuration included:

```text
Agent: Azure Monitor Agent (AMA)
Data Connector: Windows Security Events via AMA
Collection: All Security Events
```

The DCR was successfully created and associated with the Azure Windows VM.

---

### 4. Azure Arc — Windows Server

Connected the non-Azure Windows server to Azure using **Azure Arc**.

The connection was performed using:

```text
azcmagent connect
```

The connection status was verified using:

```text
azcmagent show
```

The agent status was confirmed as:

```text
Connected
```

The Azure Arc-enabled Windows server was subsequently added as a resource to the existing Data Collection Rule.

This demonstrated how Microsoft Sentinel can collect security events from Windows systems outside of Azure.

---

## 🐧 Linux Log Collection

### 5. Common Event Format (CEF) via AMA

Connected a Linux host to Azure using Azure Arc and configured the **Common Events Format (CEF) via AMA** Data Connector.

A dedicated Data Collection Rule was created for the Linux host.

The configuration collected:

```text
LOG_WARNING
```

The Azure Monitor Agent was deployed through the Data Collection Rule.

The CEF collector was already prepared on the Linux host.

The Linux host was validated using:

```bash
netstat -lnptv
```

The `rsyslog` / `syslog-ng` service was verified as listening on:

```text
TCP/UDP 514
```

CEF events can be queried through the:

```text
CommonSecurityLog
```

table.

---

### 6. Syslog via AMA

A second Linux host was connected to Azure using Azure Arc.

The **Syslog via AMA** Data Connector was then configured.

A separate Data Collection Rule was created and associated with the Linux host.

The configuration collected:

```text
LOG_WARNING
```

The Azure Monitor Agent and AMA Forwarder were used for log collection.

The Linux host was validated using:

```bash
netstat -lnptv
```

The `rsyslog` / `syslog-ng` service was verified as listening on port:

```text
514
```

Syslog events can be queried through the:

```text
Syslog
```

table.

---

## 🔗 Microsoft Defender XDR Integration

### 7. Connect Microsoft Sentinel to Defender XDR

The final exercise explored the integration between Microsoft Sentinel and Microsoft Defender XDR.

Two integration scenarios were simulated:

```text
Microsoft Sentinel Workspace
            ↓
      Defender XDR
```

and:

```text
Existing Microsoft Sentinel Workspace
            ↓
      Defender XDR
```

The interactive guides demonstrated the process of onboarding Sentinel workspaces into Microsoft Defender XDR.

The exercise also introduced the Microsoft Sentinel capabilities available directly through the Microsoft Defender portal as part of the unified security operations experience.

---

## 📊 Important Tables

| Data Source                  | Data Connector                  | Primary Table                 |
| ---------------------------- | ------------------------------- | ----------------------------- |
| Microsoft Defender for Cloud | Tenant-based Defender for Cloud | `SecurityAlert`               |
| Azure Activity               | Azure Activity                  | `AzureActivity`               |
| Windows Security Events      | Windows Security Events via AMA | Windows security event tables |
| Linux CEF                    | CEF via AMA                     | `CommonSecurityLog`           |
| Linux Syslog                 | Syslog via AMA                  | `Syslog`                      |

## 🔄 Data Collection Architecture

```text
                         Microsoft Sentinel
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
     Defender for Cloud   Azure Activity      Defender XDR
             │                  │                  │
             ▼                  ▼                  ▼
      SecurityAlert       AzureActivity       XDR Data
             
             ┌──────────────────┴──────────────────┐
             │                                     │
             ▼                                     ▼
      Windows Systems                         Linux Systems
             │                                     │
             ▼                                     ├── CEF
      Azure Monitor Agent                       │      │
             │                                  │      ▼
             ▼                                  │ CommonSecurityLog
          DCR                                   │
                                                └── Syslog
                                                     │
                                                     ▼
                                                   Syslog
```

## 🔐 Technologies Practiced

* Microsoft Sentinel Data Connectors
* Microsoft Defender for Cloud
* Microsoft Defender XDR
* Azure Activity
* Azure Monitor Agent (AMA)
* Azure Arc
* Data Collection Rules (DCR)
* Windows Security Events
* Common Event Format (CEF)
* Syslog
* `SecurityAlert`
* `AzureActivity`
* `CommonSecurityLog`
* `Syslog`
* UEBA

## 📸 Screenshots

<img width="1038" height="874" alt="image" src="https://github.com/user-attachments/assets/21c4152d-2ff0-4bf1-82c9-ea74d851be39" />


## ✅ Result

Completed the **Connect logs to Microsoft Sentinel** lab covering Learning Path 07.

The lab provided hands-on experience connecting and managing multiple log sources in Microsoft Sentinel, including:

* Microsoft Defender for Cloud
* Azure Activity
* Azure Windows virtual machines
* Azure Arc-enabled Windows servers
* Linux hosts
* CEF via AMA
* Syslog via AMA
* Microsoft Defender XDR

The lab also provided practical experience with **Azure Monitor Agent, Azure Arc, Data Collection Rules, security event collection, CEF, Syslog, and Microsoft Sentinel–Defender XDR integration**.
