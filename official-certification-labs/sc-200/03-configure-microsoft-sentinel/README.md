# 🛡️ SC-200 — Configure Microsoft Sentinel Environment

## 🎯 Objective

Configure and manage a Microsoft Sentinel environment by creating watchlists, managing threat intelligence indicators, and configuring log retention settings.

The lab focuses on core Microsoft Sentinel configuration tasks used to support security monitoring, threat hunting, and operational requirements.

---

## 🏗️ Environment

| Component           | Configuration                 |
| ------------------- | ----------------------------- |
| Security Platform   | Microsoft Sentinel            |
| Security Portal     | Microsoft Defender XDR        |
| Query Language      | Kusto Query Language (KQL)    |
| Threat Intelligence | Microsoft Threat Intelligence |
| Primary Table       | `SecurityEvent_CL`            |
| Lab Platform        | Microsoft Learn               |

---

## ⚙️ Sentinel Configuration

### 1. Watchlist

Created a Microsoft Sentinel watchlist containing high-value hosts.

The CSV file contained the following structure:

```csv
Hostname
Host1
Host2
Host3
Host4
Host5
```

Watchlist configuration:

| Setting     | Value              |
| ----------- | ------------------ |
| Name        | `HighValueHostsJP` |
| Description | `High Value Hosts` |
| Alias       | `HighValueHosts`   |
| Search Key  | `Hostname`         |
| Source      | Local CSV          |

The watchlist can be accessed in KQL using:

```kql
_GetWatchlist('HighValueHosts')
```

The `Hostname` column is used to reference the host values.

---

### 2. Threat Intelligence Indicator

Created a threat intelligence indicator through Microsoft Defender's Threat Intelligence management interface.

Configuration:

| Setting        | Value                                  |
| -------------- | -------------------------------------- |
| Object Type    | `Indicator`                            |
| Observable     | Domain name                            |
| Indicator Type | `malicious-activity`                   |
| Valid From     | Lab execution date                     |
| Description    | `This domain is known to be malicious` |

The indicator was created as a domain-based threat intelligence object and subsequently queried through **Advanced Hunting**.

---

### 3. Advanced Hunting

Used **Microsoft Defender Advanced Hunting** to query the threat intelligence data created during the lab.

The threat intelligence data can be queried through the `ThreatIntelIndicators` table.

Example:

```kql
ThreatIntelIndicators
| project ObservableValue
```

This allows the observable value associated with the indicator to be isolated from the other threat intelligence fields.

---

### 4. Log Retention

Configured the retention settings for the `SecurityEvent` table.

| Tier            | Retention |
| --------------- | --------- |
| Analytics       | 90 days   |
| Total retention | 90 days   |
| Data lake       | 180 days  |

The Analytics retention period was changed to **90 days**, while the Data lake retention setting was verified at **180 days**.

---

## 📸 Screenshots

<img width="1019" height="762" alt="image" src="https://github.com/user-attachments/assets/cf2086c9-3b7b-4d04-817e-8ea0d0aecd2c" />

<img width="1126" height="768" alt="image" src="https://github.com/user-attachments/assets/c03a258f-89a8-499f-9c2d-59090bba97bf" />


---

## ✅ Result

Completed the Microsoft Sentinel configuration lab covering:

* Watchlist creation and CSV ingestion
* Watchlist search keys and aliases
* KQL access to watchlists
* Threat intelligence indicator creation
* Domain-based observables
* Threat intelligence querying with Advanced Hunting
* `ThreatIntelIndicators` data
* Analytics log retention configuration
* Data lake retention configuration

The lab provided hands-on experience configuring Microsoft Sentinel components used for security monitoring, threat intelligence, threat hunting, and log management.
