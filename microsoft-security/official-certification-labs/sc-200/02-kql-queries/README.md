# 🔎 SC-200 — Microsoft Sentinel KQL

## 🎯 Objective

Practice Kusto Query Language (KQL) for security log analysis, threat hunting, data aggregation, visualization, and multi-table investigation in Microsoft Defender XDR.

The lab covers filtering and searching log data, aggregation, visualization, joins, string and JSON manipulation, and reusable KQL functions.

---

## 🏗️ Environment

| Component           | Configuration              |
| ------------------- | -------------------------- |
| Security Platform   | Microsoft Defender XDR     |
| Query Language      | Kusto Query Language (KQL) |
| Primary Data        | `SecurityEvent_CL`         |
| Authentication Data | `SigninLogs_CL`            |
| Additional Data     | `App*` / `Sec*` tables     |
| Lab Platform        | Microsoft Learn            |

---

## ⚙️ KQL Exercises

### 1. Basic Queries

Started by querying the `SecurityEvent_CL` table and exploring the available log data.

Basic filtering was performed using `search` and `where`:

```kql
SecurityEvent_CL
| where TimeGenerated > ago(5d)
| where EventID_s == 4624
| where AccountType_s =~ "user"
```

Also practiced filtering multiple event IDs:

```kql
SecurityEvent_CL
| where TimeGenerated > ago(5d)
| where EventID_s in (4624, 4625)
```

---

### 2. Variables and Dynamic Data

Used `let` statements to define reusable values, lists, and query results.

Example:

```kql
let timeOffset = 1d;
let discardEventID = 4688;

SecurityEvent_CL
| where TimeGenerated > ago(timeOffset*60)
| where TimeGenerated < ago(timeOffset)
| where EventID_s != discardEventID
```

Also created dynamic lists for filtering specific accounts:

```kql
let suspiciousAccounts = datatable(account: string) [
    @"NA\timadmin",
    @"NT AUTHORITY\SYSTEM"
];

SecurityEvent_CL
| where TimeGenerated > ago(7d)
| where Account_s in (suspiciousAccounts)
```

---

### 3. Data Aggregation with `summarize`

Used `summarize` to aggregate security events and identify activity patterns.

Example:

```kql
SecurityEvent_CL
| where TimeGenerated > ago(5d)
| where EventID_s == 4688
| summarize count() by Computer
```

Practiced:

* `count()`
* `dcount()`
* `arg_max()`
* `arg_min()`
* `make_list()`
* `make_set()`

Also compared query results based on the order of operations in the KQL pipeline.

For example, filtering before or after `arg_max()` produces different results:

```kql
SecurityEvent_CL
| where EventID_s == 4624
| summarize arg_max(TimeGenerated, *) by Account_s
```

---

### 4. Security Detection Logic

Built a query to identify accounts generating disabled-account authentication failures across multiple applications:

```kql
let timeframe = 30d;
let threshold = 1;

SigninLogs_CL
| where TimeGenerated >= ago(timeframe)
| where ResultDescription has "User account is disabled"
| summarize applicationCount = dcount(AppDisplayName_s) by UserPrincipalName_s, IPAddress
| where applicationCount >= threshold
```

This demonstrated how KQL can be used to transform raw authentication logs into detection-oriented queries.

---

### 5. Data Visualization

Used the `render` operator to visualize query results.

Example — account activity:

```kql
SecurityEvent_CL
| where TimeGenerated > ago(5d)
| summarize count() by Account_s
| render columnchart
```

Time-based visualization:

```kql
SecurityEvent_CL
| where TimeGenerated > ago(5d)
| summarize count() by bin(TimeGenerated, 1m)
| render timechart
```

---

### 6. Multi-Table Queries

Practiced combining data from multiple tables using `union` and `join`.

Example:

```kql
SecurityEvent_CL
| union SigninLogs_CL
```

Also used table wildcards:

```kql
union Sec*
| summarize count() by Type
```

A multi-table `join` was used to correlate logon and logoff activity for the same account:

```kql
SecurityEvent_CL
| where EventID_s == 4624
| summarize LogOnCount = count() by Account_s
| project LogOnCount, Account_s
| join kind=inner (
    SecurityEvent_CL
    | where EventID_s == 4634
    | summarize LogOffCount = count() by Account_s
    | project LogOffCount, Account_s
) on Account_s
```

---

### 7. String and JSON Data

Practiced extracting and transforming data stored in structured and unstructured string fields.

Used `extract()` to retrieve account names:

```kql
SecurityEvent_CL
| where EventID_s == 4672 and AccountType_s == "User"
| extend Account_Name = extract(
    @"^(.*\\)?([^@]*)(@.*)?$",
    2,
    tolower(Account_s)
)
| summarize LoginCount = count() by Account_Name
| where Account_Name != ""
| where LoginCount < 10
```

Also worked with JSON authentication data using `parse_json()`:

```kql
SigninLogs_CL
| extend AuthDetails = parse_json(AuthenticationDetails_s)
| extend AuthMethod = AuthDetails[0].authenticationMethod
| extend AuthResult = AuthDetails[0]["authenticationStepResultDetail"]
| project AuthMethod, AuthResult, AuthDetails
```

Practiced multi-value operations with:

* `mv-expand`
* `mv-apply`

---

### 8. Reusable KQL Functions

Created a reusable KQL function from a saved query and tested it through its function alias.

Example:

```kql
PrivLogins
```

This demonstrated how frequently used queries can be saved and reused during security investigations and hunting activities.

---

## 📸 Screenshots

<img width="1024" height="766" alt="image" src="https://github.com/user-attachments/assets/00524bf8-bf94-4a20-82eb-f42e766873b5" />

---

## ✅ Result

Completed hands-on KQL exercises covering:

* Log searching and filtering
* `where` and `search`
* Variables and dynamic tables with `let`
* Data aggregation with `summarize`
* Security detection logic
* `arg_max()` and `arg_min()`
* Data visualization with `render`
* Multi-table queries with `union` and `join`
* String extraction and parsing
* JSON manipulation
* Multi-value expansion
* Reusable KQL functions

The lab provided practical experience writing KQL queries for security monitoring, investigation, and threat hunting scenarios in Microsoft security environments.
