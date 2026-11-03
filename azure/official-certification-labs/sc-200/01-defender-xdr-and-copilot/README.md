# 🛡️ SC-200 — Microsoft Defender XDR & Defender for Endpoint

## 🎯 Objective

Implement and validate Microsoft Defender security operations capabilities through Microsoft Purview Audit and Microsoft Defender for Endpoint.

The lab covers device onboarding, RBAC, device groups, detection testing, attack simulation, incident investigation, and IP investigation.

---

## 🏗️ Environment

| Component         | Configuration                   |
| ----------------- | ------------------------------- |
| Security Platform | Microsoft Defender XDR          |
| Endpoint Security | Microsoft Defender for Endpoint |
| Audit             | Microsoft Purview Audit         |
| Test Device       | WIN1                            |
| Operating System  | Windows 10/11                   |
| User Group        | `sg-IT`                         |
| Device Group      | `Regular`                       |
| RBAC Role         | `Tier 1 Support`                |

---

## 🏛️ Architecture

```text
                         Microsoft Defender XDR
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             Purview Audit             Defender for Endpoint
                                                │
                                                ▼
                                         Device Onboarding
                                                │
                                                ▼
                                              WIN1
                                                │
                                  ┌─────────────┴─────────────┐
                                  │                           │
                                  ▼                           ▼
                           Detection Test             Attack Simulation
                                  │                           │
                                  └─────────────┬─────────────┘
                                                ▼
                                       Incident Investigation
                                                │
                                                ▼
                                        Evidence & Response
                                                │
                                                ▼
                                         IP Investigation
```

---

## ⚙️ Implementation

### 1. Microsoft Purview Audit

Enabled audit recording through the Microsoft Purview portal:

**Purview → Solutions → Audit → Start recording user and admin activity**

Audit recording may take some time to become available after activation.

If audit ingestion was not enabled, I verified and enabled it through Exchange Online PowerShell:

```powershell
Install-Module -Name ExchangeOnlineManagement
Connect-ExchangeOnline

Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled

Set-AdminAuditLogConfig -UnifiedAuditLogIngestionEnabled $true

Get-AdminAuditLogConfig | FL UnifiedAuditLogIngestionEnabled

Disconnect-ExchangeOnline
```

If required by the lab environment:

```powershell
Enable-OrganizationCustomization
```

---

### 2. Microsoft Defender for Endpoint

Initialized Microsoft Defender for Endpoint through:

**Microsoft Defender XDR → Settings → Device discovery**

Configured **Standard discovery** for the environment.

---

### 3. Device Onboarding

Configured Windows device onboarding through:

**Settings → Endpoints → Onboarding**

Used the **Defender deployment tool** to onboard `WIN1`.

The deployment package was generated from the Defender portal, extracted on the endpoint, and executed locally.

After entering the generated access key, the onboarding process was completed and the device became available in Defender for Endpoint.

---

### 4. RBAC Configuration

Created a custom role for Tier 1 support personnel.

**Role:** `Tier 1 Support`

**Security Operations permissions:**

* All read and manage permissions

**Assignment:**

* Assignment name: `Tier 1 Support`
* User/data group: `sg-IT`

Configured through:

**Settings → Microsoft Defender XDR → Permissions and Roles**

---

### 5. Device Group

Created the `Regular` device group under:

**Settings → Endpoints → Device groups**

Configuration:

| Setting           | Value            |
| ----------------- | ---------------- |
| Name              | `Regular`        |
| Remediation level | Full remediation |
| OS                | Windows 10/11    |
| User access       | `sg-IT`          |

---

### 6. Detection Test

Verified that `WIN1` was successfully onboarded under:

**Assets → Devices**

Then executed the Microsoft Defender for Endpoint detection test from the onboarding page using an elevated Command Prompt.

The generated alert was:

```text
[TestAlert] Suspicious PowerShell commandline
```

The alert was investigated through:

**Investigations & response → Incidents & alerts → Alerts**

Reviewed the alert timeline, details, and recommended actions.

The related incident was:

```text
Execution incident on one endpoint
```

The incident was tagged as:

```text
Simulation
```

and classified as:

```text
Informational
└── Expected activity
    └── Security testing
```

---

### 7. Attack Simulation

Performed a simulated multi-stage attack on `WIN1`.

Opened an elevated PowerShell session and navigated to the lab files:

```powershell
cd C:\Users\Admin\Desktop\Allfiles
```

Executed the attack simulation:

```powershell
.\AttackScript.ps1
```

Selected:

```text
R
```

to run the simulation once.

The script generated simulated attack activity, including process execution, simulated command-and-control communication, and other behaviors designed to trigger Defender for Endpoint detections.

---

### 8. Incident Investigation

The simulation generated the following incident:

```text
Multi-stage incident involving Defense evasion & Discovery on one endpoint
```

Investigated the incident through:

**Investigations & response → Incidents & alerts → Incidents**

Reviewed the:

* Attack story
* Incident graph
* Alerts
* Assets
* Investigations
* Evidence and Response
* Summary

The **Attack story** was used to visualize the sequence of events and understand how the individual alerts were connected within the incident.

---

### 9. IP Investigation

Within **Evidence and Response**, reviewed the IP addresses associated with the incident.

Opened the observed IP address and reviewed:

* Overview
* Incidents & alerts
* Observed in organizations

This provided additional context for the network indicator involved in the simulated attack.

---

## 📸 Screenshot

<img width="1104" height="813" alt="Microsoft Defender XDR Lab" src="https://github.com/user-attachments/assets/f5d0b8b6-471f-4bea-b43c-34de771e7592" />

---

## ✅ Result

Successfully configured Microsoft Purview Audit and Microsoft Defender for Endpoint, onboarded `WIN1`, implemented RBAC and device grouping, generated security detections, simulated an attack, and investigated the resulting multi-stage incident through Microsoft Defender XDR.
