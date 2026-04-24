# Module 10 - Azure Monitoring

## Learning Objectives
By the end of this module, you will be able to:
- Understand Azure Monitor architecture and capabilities
- Configure and analyze Azure Metrics
- Work with Azure Log Analytics
- Create and manage Azure Alerts
- Implement Application Insights
- Generate Backup reports
- Configure Recovery Services Vault
- Set up VM Backups

---

## 10.1 Azure Monitor

### What is Azure Monitor?
Azure Monitor is a comprehensive monitoring solution that collects, analyzes, and acts on telemetry from cloud and on-premises environments.

#### 🏢 Real-World Monitoring Use Cases:

**E-Commerce Platform Observability:**
```
Company: ShopFast (Online retail, 2M daily visitors)
Challenge: Detect and resolve issues before customers notice

Monitoring Stack:
┌────────────────────────────────────────────────────────────────┐
│                    Observability Architecture                   │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Application Insights (APM):                                    │
│  ├── Request duration: Alert if P95 > 500ms                    │
│  ├── Failure rate: Alert if > 1% errors                        │
│  ├── Dependency tracking: SQL, Redis, external APIs            │
│  └── User analytics: Page views, user flows                    │
│                                                                 │
│  Infrastructure Metrics:                                        │
│  ├── VM CPU: Alert if > 85% for 5 minutes                      │
│  ├── VM Memory: Alert if > 90% used                            │
│  ├── Disk IOPS: Alert if queue depth > 30                      │
│  └── Network: Alert if packet loss > 0.1%                      │
│                                                                 │
│  Log Analytics (Centralized Logs):                              │
│  ├── Application logs: Errors, warnings, traces                │
│  ├── Security logs: Failed logins, suspicious activity         │
│  ├── Audit logs: Who changed what                              │
│  └── Retention: 90 days hot, 2 years archive                   │
│                                                                 │
│  Action Groups:                                                 │
│  ├── P1 (Critical): PagerDuty + Phone + SMS + Email           │
│  ├── P2 (Warning): Teams channel + Email                       │
│  └── P3 (Info): Email only (weekly digest)                     │
└────────────────────────────────────────────────────────────────┘

Real Incident Response:
Scenario: Checkout page slow during flash sale
├── 10:00 AM: Metric alert fires (response time > 2s)
├── 10:01 AM: Auto-scale adds 5 more web instances
├── 10:02 AM: AI detects SQL dependency degradation
├── 10:03 AM: DBA receives alert, finds blocking query
├── 10:05 AM: Query killed, performance restored
└── MTTD: 1 minute, MTTR: 5 minutes (zero customer escalations)
```

**Financial Trading Platform:**
```
Company: TradeFast Securities
Requirement: 99.999% uptime, <50ms latency

Advanced Monitoring Setup:
┌────────────────────────────────────────────────────────────────┐
│  Real-Time Trading Monitoring                                   │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Custom Metrics (pushed every second):                         │
│  ├── orders_per_second: Current trading volume                 │
│  ├── order_latency_p99: 99th percentile latency                │
│  ├── matching_engine_queue: Pending orders                     │
│  └── market_data_lag: Delay from exchange                      │
│                                                                 │
│  Alert Thresholds (aggressive):                                 │
│  ├── P99 latency > 30ms: CRITICAL (1 minute to respond)       │
│  ├── Queue depth > 100: WARNING (investigate)                  │
│  ├── Market data lag > 5ms: CRITICAL (halt trading)           │
│  └── Any system unavailable: CRITICAL (immediate call)         │
│                                                                 │
│  Dashboards (Wall-mounted screens):                             │
│  ├── Real-time: Trading volume, latency heatmap               │
│  ├── Capacity: System resources vs. market hours              │
│  └── Incidents: Active alerts, recent changes                  │
│                                                                 │
│  SLA Tracking:                                                  │
│  └── Monthly report: 99.997% achieved (27 minutes downtime)   │
└────────────────────────────────────────────────────────────────┘
```

**Healthcare System Compliance Monitoring:**
```
Company: HealthCare Network (15 hospitals)
Requirement: HIPAA compliance + operational monitoring

Compliance Monitoring:
┌────────────────────────────────────────────────────────────────┐
│  Azure Sentinel + Log Analytics                                 │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Security Monitoring (HIPAA Required):                          │
│  ├── Failed login attempts: Alert if > 5 in 10 minutes        │
│  ├── Privilege escalation: Any admin role assignment          │
│  ├── PHI access: Track every patient record access            │
│  └── Data export: Alert on large data downloads               │
│                                                                 │
│  KQL Query Examples:                                            │
│  // Detect after-hours PHI access                               │
│  PatientRecordAccess                                            │
│  | where TimeGenerated between (datetime(22:00)..datetime(06:00))│
│  | where AccessType == "View" or AccessType == "Export"        │
│  | summarize count() by UserId, bin(TimeGenerated, 1h)        │
│  | where count_ > 50                                            │
│                                                                 │
│  // Failed login brute force detection                          │
│  SigninLogs                                                     │
│  | where ResultType != "0"                                      │
│  | summarize FailedAttempts=count() by UserPrincipalName, IPAddress│
│  | where FailedAttempts > 10                                    │
│                                                                 │
│  Compliance Reports (Auto-generated):                           │
│  ├── Weekly: Access audit report                               │
│  ├── Monthly: Security incidents summary                        │
│  └── Quarterly: HIPAA compliance attestation                   │
└────────────────────────────────────────────────────────────────┘
```

**DevOps SRE Dashboard:**
```
Four Golden Signals Monitoring:

1. LATENCY (How long to serve requests?)
   KQL: requests | summarize percentiles(duration, 50, 95, 99) by bin(timestamp, 5m)
   Alert: P95 > 500ms

2. TRAFFIC (How many requests?)
   KQL: requests | summarize count() by bin(timestamp, 1m)
   Dashboard: Requests per minute trend

3. ERRORS (What's the failure rate?)
   KQL: requests | summarize failRate=countif(success==false)*100.0/count() by bin(timestamp, 5m)
   Alert: Error rate > 1%

4. SATURATION (How full is our system?)
   KQL: Perf | where ObjectName=="Processor" | summarize avg(CounterValue) by Computer
   Alert: CPU > 80% for 10 minutes
```

### Azure Monitor Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Azure Monitor                                   │
│                                                                              │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                        Data Sources                                 │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│   │  │ Azure    │  │ Azure    │  │ Operating│  │ Custom   │           │   │
│   │  │ Resources│  │ Tenant   │  │ System   │  │ Sources  │           │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │   │
│   └────────────────────────────────────────────────────────────────────┘   │
│                                    │                                         │
│                                    ▼                                         │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                        Data Platform                                │   │
│   │  ┌─────────────────────┐     ┌─────────────────────┐               │   │
│   │  │      Metrics        │     │       Logs          │               │   │
│   │  │  (Time-series DB)   │     │  (Log Analytics)    │               │   │
│   │  └─────────────────────┘     └─────────────────────┘               │   │
│   └────────────────────────────────────────────────────────────────────┘   │
│                                    │                                         │
│                                    ▼                                         │
│   ┌────────────────────────────────────────────────────────────────────┐   │
│   │                        Insights & Actions                           │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐           │   │
│   │  │Visualize │  │ Analyze  │  │  Alerts  │  │  Actions │           │   │
│   │  │(Workbooks│  │ (KQL)    │  │          │  │ (Runbook,│           │   │
│   │  │Dashboards│  │          │  │          │  │ Function)│           │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘           │   │
│   └────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Data Types in Azure Monitor

| Type | Description | Storage |
|------|-------------|---------|
| **Metrics** | Numerical values, time-series | Azure Monitor Metrics DB |
| **Logs** | Text, events, traces | Log Analytics workspace |
| **Traces** | Distributed tracing | Application Insights |

### Azure Monitor Data Sources

| Source | Data Type | Examples |
|--------|-----------|----------|
| **Platform Metrics** | Metrics | CPU, Memory, Disk |
| **Activity Logs** | Logs | Management operations |
| **Diagnostic Logs** | Logs | Resource-specific events |
| **Application Data** | Traces/Logs | App Insights telemetry |
| **Guest OS Data** | Metrics/Logs | VM agent data |

---

## 10.2 Azure Metrics

### What are Metrics?
Metrics are numerical values that describe aspects of a system at a particular time. They are collected at regular intervals and stored in a time-series database.

### Metric Properties

| Property | Description |
|----------|-------------|
| **Name** | Identifier for the metric |
| **Timestamp** | When collected |
| **Value** | Numerical measurement |
| **Dimensions** | Categorization attributes |

### Common Azure Metrics

| Resource Type | Example Metrics |
|---------------|-----------------|
| **Virtual Machines** | CPU percentage, Network In/Out, Disk IOPS |
| **Storage Accounts** | Transactions, Ingress/Egress, Availability |
| **SQL Database** | DTU percentage, Storage used, Deadlocks |
| **App Service** | Requests, Response time, HTTP errors |

### Viewing Metrics

```powershell
# Get available metrics for a resource
$resourceId = "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}"

Get-AzMetricDefinition -ResourceId $resourceId | Select-Object Name, Unit

# Get metric values
$endTime = Get-Date
$startTime = $endTime.AddHours(-1)

Get-AzMetric -ResourceId $resourceId `
    -MetricName "Percentage CPU" `
    -StartTime $startTime `
    -EndTime $endTime `
    -TimeGrain 00:05:00
```

```bash
# Azure CLI
az monitor metrics list \
    --resource "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}" \
    --metric "Percentage CPU" \
    --interval PT5M

# List available metrics
az monitor metrics list-definitions \
    --resource "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}"
```

### Metrics Explorer

Azure Metrics Explorer allows you to:
- Create charts from metrics
- Apply aggregations (avg, min, max, sum, count)
- Add filters and splitting
- Pin to dashboards

### Custom Metrics

```powershell
# Send custom metric using REST API
$uri = "https://monitoring.azure.com/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}/metrics"

$customMetric = @{
    time = (Get-Date).ToUniversalTime().ToString("o")
    data = @{
        baseData = @{
            metric = "CustomMetricName"
            namespace = "CustomNamespace"
            dimNames = @("Dimension1")
            series = @(
                @{
                    dimValues = @("DimensionValue1")
                    min = 10
                    max = 100
                    sum = 500
                    count = 10
                }
            )
        }
    }
} | ConvertTo-Json -Depth 10
```

---

## 10.3 Log Analytics

### What is Log Analytics?
Log Analytics is a tool in the Azure portal for editing and running log queries against data in Azure Monitor Logs.

### Log Analytics Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   Log Analytics Workspace                        │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                        Tables                                 ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    ││
│  │  │AzureActivity│  │Heartbeat│  │  Perf    │  │  Event   │    ││
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    ││
│  │  │Syslog    │  │ W3CIISLog│  │Exceptions│  │ Custom   │    ││
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    ││
│  └─────────────────────────────────────────────────────────────┘│
│                              │                                   │
│                              ▼                                   │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Kusto Query Language (KQL)                ││
│  │                                                               ││
│  │  AzureActivity | where TimeGenerated > ago(1h) | count       ││
│  │                                                               ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Creating Log Analytics Workspace

```powershell
# Create workspace
New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "Monitoring-RG" `
    -Name "MyLogAnalyticsWorkspace" `
    -Location "East US" `
    -Sku "PerGB2018"
```

```bash
# Azure CLI
az monitor log-analytics workspace create \
    --resource-group "Monitoring-RG" \
    --workspace-name "MyLogAnalyticsWorkspace" \
    --location "eastus" \
    --sku "PerGB2018"
```

### Kusto Query Language (KQL) Basics

```kusto
// Get recent sign-in logs
SigninLogs
| where TimeGenerated > ago(1d)
| project UserDisplayName, ClientAppUsed, Status
| limit 100

// Count events by source
Event
| summarize count() by Source
| order by count_ desc

// Get CPU performance data
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize AvgCPU = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
| render timechart

// Find failed logins
SecurityEvent
| where EventID == 4625
| summarize FailedAttempts = count() by Account, IPAddress
| order by FailedAttempts desc

// Azure Activity query
AzureActivity
| where OperationNameValue endswith "write" or OperationNameValue endswith "delete"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
```

### Common KQL Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `where` | Filter rows | `| where Status == "Failed"` |
| `project` | Select columns | `| project Name, Time` |
| `summarize` | Aggregate data | `| summarize count() by Type` |
| `order by` | Sort results | `| order by Count desc` |
| `join` | Combine tables | `| join kind=inner Table2 on Key` |
| `extend` | Add calculated column | `| extend Duration = End - Start` |
| `render` | Visualize results | `| render timechart` |

### Connecting Data Sources

```powershell
# Connect VM to Log Analytics
$workspaceId = (Get-AzOperationalInsightsWorkspace `
    -ResourceGroupName "Monitoring-RG" `
    -Name "MyLogAnalyticsWorkspace").CustomerId

$workspaceKey = (Get-AzOperationalInsightsWorkspaceSharedKey `
    -ResourceGroupName "Monitoring-RG" `
    -Name "MyLogAnalyticsWorkspace").PrimarySharedKey

# Install Log Analytics VM Extension (Windows)
Set-AzVMExtension `
    -ResourceGroupName "VM-RG" `
    -VMName "MyVM" `
    -Name "MicrosoftMonitoringAgent" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion "1.0" `
    -Settings @{"workspaceId" = $workspaceId} `
    -ProtectedSettings @{"workspaceKey" = $workspaceKey}
```

---

## 10.4 Azure Alerts

### What are Alerts?
Alerts proactively notify you when conditions are found in your monitoring data.

### Alert Components

```
┌─────────────────────────────────────────────────────────────────┐
│                        Alert Rule                                │
│                                                                  │
│   ┌─────────────────┐    ┌─────────────────┐                   │
│   │    Scope        │    │   Condition      │                   │
│   │  (Resource)     │    │  (Signal/Logic)  │                   │
│   └─────────────────┘    └─────────────────┘                   │
│                                │                                 │
│                                ▼                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                      Condition Met                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                │                                 │
│                                ▼                                 │
│   ┌─────────────────┐    ┌─────────────────┐                   │
│   │  Action Group   │    │    Severity     │                   │
│   │ (Notification)  │    │    (0-4)        │                   │
│   └─────────────────┘    └─────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘
```

### Alert Types

| Type | Based On | Use Case |
|------|----------|----------|
| **Metric Alerts** | Metric values | CPU > 80% |
| **Log Alerts** | Log Analytics queries | Error count > 10 |
| **Activity Log Alerts** | Activity log events | VM deallocated |
| **Service Health Alerts** | Azure service issues | Service outage |
| **Smart Detection** | AI-based anomalies | Performance degradation |

### Alert Severity Levels

| Severity | Name | Description |
|----------|------|-------------|
| Sev 0 | Critical | Immediate attention required |
| Sev 1 | Error | Requires attention soon |
| Sev 2 | Warning | May require attention |
| Sev 3 | Informational | Good to know |
| Sev 4 | Verbose | Detailed information |

### Creating Alert Rules

```powershell
# Create metric alert rule
$resourceId = "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}"

$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "Percentage CPU" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 80

Add-AzMetricAlertRuleV2 `
    -Name "HighCPUAlert" `
    -ResourceGroupName "Monitoring-RG" `
    -WindowSize 00:05:00 `
    -Frequency 00:01:00 `
    -TargetResourceId $resourceId `
    -Condition $condition `
    -ActionGroupId "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/microsoft.insights/actionGroups/NotifyOps" `
    -Severity 2
```

```bash
# Azure CLI - Create metric alert
az monitor metrics alert create \
    --name "HighCPUAlert" \
    --resource-group "Monitoring-RG" \
    --scopes "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}" \
    --condition "avg Percentage CPU > 80" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --action "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/microsoft.insights/actionGroups/NotifyOps" \
    --severity 2
```

### Creating Action Groups

```powershell
# Create action group
$emailReceiver = New-AzActionGroupReceiver `
    -Name "EmailOps" `
    -EmailReceiver `
    -EmailAddress "ops@contoso.com"

$smsReceiver = New-AzActionGroupReceiver `
    -Name "SMSOps" `
    -SmsReceiver `
    -CountryCode "1" `
    -PhoneNumber "5555555555"

Set-AzActionGroup `
    -ResourceGroupName "Monitoring-RG" `
    -Name "NotifyOps" `
    -ShortName "NotifyOps" `
    -Receiver $emailReceiver, $smsReceiver
```

### Action Types

| Action | Description |
|--------|-------------|
| Email/SMS/Push/Voice | Notify individuals |
| Azure Function | Run serverless code |
| Logic App | Run workflow |
| Webhook | Call external service |
| Automation Runbook | Run PowerShell script |
| ITSM | Create ticket |
| Event Hub | Stream to Event Hub |
| Secure Webhook | Call with Azure AD auth |

---

## 10.5 Application Insights

### What is Application Insights?
Application Insights is an extensible Application Performance Management (APM) service for monitoring live applications.

### Application Insights Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Application Insights                          │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Data Collection                       │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│   │
│   │  │ Requests │  │Dependencies│  │Exceptions│  │ Traces   ││   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘│   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│   │
│   │  │Page Views│  │ Events   │  │ Metrics  │  │Availability│   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘│   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    Analysis Tools                        │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│   │
│   │  │Application│  │ Failure  │  │Performance│  │Live      ││   │
│   │  │   Map     │  │ Analysis │  │           │  │ Metrics  ││   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘│   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Application Insights

```powershell
# Create Application Insights
New-AzApplicationInsights `
    -ResourceGroupName "App-RG" `
    -Name "MyAppInsights" `
    -Location "East US" `
    -WorkspaceResourceId "/subscriptions/{sub-id}/resourceGroups/Monitoring-RG/providers/Microsoft.OperationalInsights/workspaces/MyLogAnalyticsWorkspace"

# Get instrumentation key
$appInsights = Get-AzApplicationInsights `
    -ResourceGroupName "App-RG" `
    -Name "MyAppInsights"

$instrumentationKey = $appInsights.InstrumentationKey
$connectionString = $appInsights.ConnectionString
```

```bash
# Azure CLI
az monitor app-insights component create \
    --app "MyAppInsights" \
    --resource-group "App-RG" \
    --location "eastus" \
    --workspace "/subscriptions/{sub-id}/resourceGroups/Monitoring-RG/providers/Microsoft.OperationalInsights/workspaces/MyLogAnalyticsWorkspace"
```

### Key Telemetry Types

| Type | Description |
|------|-------------|
| **Request** | HTTP requests received |
| **Dependency** | Calls to external services |
| **Exception** | Caught/uncaught exceptions |
| **Trace** | Diagnostic log messages |
| **Event** | User actions, custom events |
| **Metric** | Performance measurements |
| **PageView** | Browser page loads |

### Application Map

```
         ┌─────────────────┐
         │   Web App       │
         │   (Frontend)    │
         └────────┬────────┘
                  │
    ┌─────────────┼─────────────┐
    │             │             │
    ▼             ▼             ▼
┌───────┐   ┌───────┐   ┌───────┐
│ API   │   │ Cache │   │ Queue │
│ App   │   │(Redis)│   │       │
└───┬───┘   └───────┘   └───────┘
    │
    ▼
┌───────┐
│  SQL  │
│  DB   │
└───────┘
```

### KQL Queries for Application Insights

```kusto
// Get failed requests
requests
| where success == false
| summarize count() by name, resultCode
| order by count_ desc

// Response time analysis
requests
| summarize avg(duration), percentile(duration, 95) by bin(timestamp, 1h)
| render timechart

// Dependency failures
dependencies
| where success == false
| summarize count() by name, resultCode
| order by count_ desc

// Exceptions by type
exceptions
| summarize count() by type
| order by count_ desc
| render piechart

// User sessions
pageViews
| summarize SessionCount = dcount(session_Id), PageViews = count() by bin(timestamp, 1d)
| render timechart
```

### Availability Tests

```powershell
# Create URL ping test
$webTest = @{
    SyntheticMonitorId = "MyWebTest"
    Name = "Homepage Check"
    Enabled = $true
    Frequency = 300  # 5 minutes
    Timeout = 30
    Kind = "ping"
    Locations = @(
        @{ Id = "us-fl-mia-edge" },
        @{ Id = "emea-nl-ams-azr" }
    )
    Configuration = @{
        WebTest = "<WebTest><Items><Request Method='GET' Url='https://myapp.com' /></Items></WebTest>"
    }
}
```

---

## 10.6 Backup Reports

### What are Backup Reports?
Backup Reports provide insights into backup health, protection status, and trends using Azure Monitor and Log Analytics.

### Enabling Backup Reports

1. **Configure Diagnostics Settings**
   - Navigate to Recovery Services Vault → Diagnostic settings
   - Send to Log Analytics workspace
   - Select categories: AzureBackupReport

```powershell
# Enable diagnostic settings for backup vault
$vault = Get-AzRecoveryServicesVault -Name "MyBackupVault"
$workspace = Get-AzOperationalInsightsWorkspace -Name "MyLogAnalyticsWorkspace" -ResourceGroupName "Monitoring-RG"

Set-AzDiagnosticSetting `
    -ResourceId $vault.ID `
    -WorkspaceId $workspace.ResourceId `
    -Enabled $true `
    -Category AzureBackupReport,CoreAzureBackup,AddonAzureBackupJobs,AddonAzureBackupAlerts,AddonAzureBackupPolicy,AddonAzureBackupStorage,AddonAzureBackupProtectedInstance
```

### Backup Report Views

| View | Shows |
|------|-------|
| **Summary** | Overall backup health |
| **Backup Items** | Protected items status |
| **Usage** | Storage consumption |
| **Jobs** | Backup job history |
| **Policies** | Policy compliance |
| **Optimize** | Cost optimization recommendations |

### Backup KQL Queries

```kusto
// Backup job status
CoreAzureBackup
| where OperationName == "Job"
| summarize count() by JobStatus
| render piechart

// Failed backups
AddonAzureBackupJobs
| where JobStatus == "Failed"
| project TimeGenerated, BackupItemUniqueId, JobFailureType, JobUniqueId
| order by TimeGenerated desc

// Backup storage trend
AddonAzureBackupStorage
| summarize StorageConsumedInMBs = sum(StorageConsumedInMBs) by bin(TimeGenerated, 1d)
| render timechart

// Protected instance count
AddonAzureBackupProtectedInstance
| summarize ProtectedInstanceCount = dcount(BackupItemUniqueId) by bin(TimeGenerated, 1d)
| render timechart
```

---

## 10.7 Recovery Services Vault

### What is Recovery Services Vault?
A management entity that stores backup data for Azure VMs, SQL databases, on-premises machines, and Azure Files.

### Recovery Services Vault Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                   Recovery Services Vault                        │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   Backup Items                           │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐│   │
│   │  │Azure VMs │  │Azure SQL │  │Azure Files│  │ On-prem  ││   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────┘│   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   Backup Policies                        │   │
│   │  • Schedule (Daily/Weekly/Monthly)                       │   │
│   │  • Retention (Days/Weeks/Months/Years)                   │   │
│   │  • Instant Restore                                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                   Storage Replication                    │   │
│   │  • LRS (Locally Redundant)                               │   │
│   │  • GRS (Geo-Redundant)                                   │   │
│   │  • ZRS (Zone-Redundant)                                  │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Recovery Services Vault

```powershell
# Create vault
New-AzRecoveryServicesVault `
    -ResourceGroupName "Backup-RG" `
    -Name "MyBackupVault" `
    -Location "East US"

# Set vault storage redundancy
$vault = Get-AzRecoveryServicesVault -Name "MyBackupVault"
Set-AzRecoveryServicesBackupProperty `
    -Vault $vault `
    -BackupStorageRedundancy GeoRedundant
```

```bash
# Azure CLI
az backup vault create \
    --resource-group "Backup-RG" \
    --name "MyBackupVault" \
    --location "eastus"

# Set storage redundancy
az backup vault backup-properties set \
    --resource-group "Backup-RG" \
    --name "MyBackupVault" \
    --backup-storage-redundancy GeoRedundant
```

### Storage Redundancy Options

| Option | Description | Use Case |
|--------|-------------|----------|
| **LRS** | 3 copies in one datacenter | Dev/Test |
| **GRS** | 6 copies across two regions | Production |
| **ZRS** | 3 copies across availability zones | High availability |

### Backup Policies

```powershell
# Create backup policy
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"

# Set schedule to daily at 10 PM
$schPol.ScheduleRunTimes.Clear()
$schPol.ScheduleRunTimes.Add((Get-Date "10:00 PM"))

# Set retention
$retPol.DailySchedule.DurationCountInDays = 30
$retPol.WeeklySchedule.DurationCountInWeeks = 12
$retPol.MonthlySchedule.DurationCountInMonths = 12
$retPol.YearlySchedule.DurationCountInYears = 5

# Create policy
New-AzRecoveryServicesBackupProtectionPolicy `
    -Name "DailyBackupPolicy" `
    -WorkloadType "AzureVM" `
    -RetentionPolicy $retPol `
    -SchedulePolicy $schPol `
    -VaultId $vault.ID
```

---

## 10.8 VM Backups

### What are VM Backups?
Azure Backup provides independent and isolated backups of Azure VMs with application-consistent snapshots.

### VM Backup Process

```
1. Backup Triggered
         │
         ▼
2. VM Extension Takes
   Application-Consistent Snapshot
         │
         ▼
3. Data Transferred to
   Recovery Services Vault
         │
         ▼
4. Recovery Point Created
```

### Enabling VM Backup

```powershell
# Set vault context
$vault = Get-AzRecoveryServicesVault -Name "MyBackupVault"
Set-AzRecoveryServicesVaultContext -Vault $vault

# Get backup policy
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DailyBackupPolicy"

# Enable backup on VM
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "MyVM"
Enable-AzRecoveryServicesBackupProtection `
    -ResourceGroupName "VM-RG" `
    -Name "MyVM" `
    -Policy $policy
```

```bash
# Azure CLI
az backup protection enable-for-vm \
    --resource-group "Backup-RG" \
    --vault-name "MyBackupVault" \
    --vm "/subscriptions/{sub-id}/resourceGroups/VM-RG/providers/Microsoft.Compute/virtualMachines/MyVM" \
    --policy-name "DailyBackupPolicy"
```

### Triggering On-Demand Backup

```powershell
# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $vault

# Get backup item
$backupItem = Get-AzRecoveryServicesBackupItem `
    -BackupManagementType "AzureVM" `
    -WorkloadType "AzureVM" `
    -Name "MyVM"

# Trigger backup
$job = Backup-AzRecoveryServicesBackupItem -Item $backupItem

# Wait for completion
Wait-AzRecoveryServicesBackupJob -Job $job
```

```bash
# Azure CLI
az backup protection backup-now \
    --resource-group "Backup-RG" \
    --vault-name "MyBackupVault" \
    --container-name "IaaSVMContainer;iaasvmcontainerv2;VM-RG;MyVM" \
    --item-name "VM;iaasvmcontainerv2;VM-RG;MyVM" \
    --retain-until "2024-12-31"
```

### Restoring VM

```powershell
# Get recovery point
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date

$recoveryPoints = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -StartDate $startDate `
    -EndDate $endDate

# Restore VM (Replace existing)
$restoreJob = Restore-AzRecoveryServicesBackupItem `
    -RecoveryPoint $recoveryPoints[0] `
    -StorageAccountName "restoreSA" `
    -StorageAccountResourceGroupName "Storage-RG" `
    -TargetResourceGroupName "VM-RG" `
    -UseOriginalStorageAccount

# Restore as new VM
$restoreJob = Restore-AzRecoveryServicesBackupItem `
    -RecoveryPoint $recoveryPoints[0] `
    -StorageAccountName "restoreSA" `
    -StorageAccountResourceGroupName "Storage-RG" `
    -TargetResourceGroupName "VM-RG" `
    -TargetVMName "RestoredVM" `
    -TargetVNetName "MyVNet" `
    -TargetVNetResourceGroup "Network-RG" `
    -TargetSubnetName "default"
```

### Restore Options

| Option | Description |
|--------|-------------|
| **Create new** | Create new VM from backup |
| **Replace existing** | Replace existing VM disks |
| **Restore disks** | Restore only disks |
| **Cross-region** | Restore to secondary region (GRS) |
| **File recovery** | Mount disks and copy files |

### File Recovery

```powershell
# Download recovery script
$script = Get-AzRecoveryServicesBackupRPMountScript `
    -RecoveryPoint $recoveryPoints[0]

# Execute script (mounts backup volume)
# Browse and copy required files
# Run script again with -Unmount to cleanup
```

---

## Hands-on Exercises

### Exercise 1: Configure Azure Monitor and Metrics

```powershell
# Create resources for monitoring
New-AzResourceGroup -Name "Monitoring-Lab-RG" -Location "East US"

# Create VM to monitor
$vmParams = @{
    ResourceGroupName = "Monitoring-Lab-RG"
    Name = "MonitoredVM"
    Location = "East US"
    Image = "Win2019Datacenter"
    Size = "Standard_DS2_v2"
    Credential = (Get-Credential -Message "Enter VM credentials")
}
New-AzVM @vmParams

# Explore metrics
$vmId = (Get-AzVM -Name "MonitoredVM" -ResourceGroupName "Monitoring-Lab-RG").Id

# List available metrics
Get-AzMetricDefinition -ResourceId $vmId | Select-Object Name, Unit | Format-Table

# Get CPU metrics for last hour
$now = Get-Date
$hourAgo = $now.AddHours(-1)

Get-AzMetric -ResourceId $vmId -MetricName "Percentage CPU" `
    -StartTime $hourAgo -EndTime $now -TimeGrain 00:05:00 |
    Select-Object -ExpandProperty Data | Format-Table

Write-Host "Navigate to Azure Portal > Monitor > Metrics to explore metrics visually"
```

### Exercise 2: Configure Log Analytics and Alerts

```powershell
# Create Log Analytics workspace
$workspace = New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -Name "MonitoringWorkspace" `
    -Location "East US" `
    -Sku "PerGB2018"

Write-Host "Workspace ID: $($workspace.CustomerId)"

# Get workspace key for agent configuration
$keys = Get-AzOperationalInsightsWorkspaceSharedKey `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -Name "MonitoringWorkspace"

Write-Host "Primary Key: $($keys.PrimarySharedKey)"

# Install VM extension for Log Analytics
$vmId = (Get-AzVM -Name "MonitoredVM" -ResourceGroupName "Monitoring-Lab-RG").Id

Set-AzVMExtension `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -VMName "MonitoredVM" `
    -Name "MicrosoftMonitoringAgent" `
    -Publisher "Microsoft.EnterpriseCloud.Monitoring" `
    -ExtensionType "MicrosoftMonitoringAgent" `
    -TypeHandlerVersion "1.0" `
    -Settings @{"workspaceId" = $workspace.CustomerId} `
    -ProtectedSettings @{"workspaceKey" = $keys.PrimarySharedKey}

# Create action group for alerts
$emailReceiver = New-AzActionGroupReceiver `
    -Name "EmailAdmin" `
    -EmailReceiver `
    -EmailAddress "admin@contoso.com"

Set-AzActionGroup `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -Name "AlertActionGroup" `
    -ShortName "AlertGrp" `
    -Receiver $emailReceiver

# Create metric alert
$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "Percentage CPU" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 80

$actionGroup = Get-AzActionGroup -ResourceGroupName "Monitoring-Lab-RG" -Name "AlertActionGroup"

Add-AzMetricAlertRuleV2 `
    -Name "HighCPUAlert" `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -WindowSize 00:05:00 `
    -Frequency 00:01:00 `
    -TargetResourceId $vmId `
    -Condition $condition `
    -ActionGroupId $actionGroup.Id `
    -Severity 2 `
    -Description "Alert when CPU exceeds 80%"

Write-Host "Alert rule created successfully"
```

### Exercise 3: Configure VM Backup with Recovery Services Vault

```powershell
# Create Recovery Services Vault
$vault = New-AzRecoveryServicesVault `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -Name "BackupVault" `
    -Location "East US"

Write-Host "Vault created: $($vault.Name)"

# Set storage redundancy
Set-AzRecoveryServicesBackupProperty `
    -Vault $vault `
    -BackupStorageRedundancy LocallyRedundant

# Set vault context
Set-AzRecoveryServicesVaultContext -Vault $vault

# Get default VM backup policy
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"

# Display policy details
Write-Host "Policy Name: $($policy.Name)"
Write-Host "Schedule: $($policy.SchedulePolicy)"
Write-Host "Retention: $($policy.RetentionPolicy)"

# Enable backup for VM
Enable-AzRecoveryServicesBackupProtection `
    -ResourceGroupName "Monitoring-Lab-RG" `
    -Name "MonitoredVM" `
    -Policy $policy

Write-Host "Backup enabled for MonitoredVM"

# Trigger on-demand backup
$backupItem = Get-AzRecoveryServicesBackupItem `
    -BackupManagementType "AzureVM" `
    -WorkloadType "AzureVM" `
    -Name "MonitoredVM"

$backupJob = Backup-AzRecoveryServicesBackupItem -Item $backupItem

Write-Host "Backup job triggered. Job ID: $($backupJob.JobId)"

# Monitor backup job status
do {
    $job = Get-AzRecoveryServicesBackupJob -JobId $backupJob.JobId
    Write-Host "Job Status: $($job.Status) - $($job.Operation)"
    Start-Sleep -Seconds 30
} while ($job.Status -eq "InProgress")

Write-Host "Backup job completed with status: $($job.Status)"

# List recovery points
$recoveryPoints = Get-AzRecoveryServicesBackupRecoveryPoint `
    -Item $backupItem `
    -StartDate (Get-Date).AddDays(-1) `
    -EndDate (Get-Date)

$recoveryPoints | Format-Table RecoveryPointType, RecoveryPointTime, RecoveryPointId
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Azure Monitor | Comprehensive monitoring for metrics, logs, and insights |
| Metrics | Time-series data, numerical values |
| Log Analytics | KQL queries, centralized log storage |
| Alerts | Proactive notifications with action groups |
| Application Insights | APM for application monitoring |
| Backup Reports | Insights into backup health via Log Analytics |
| Recovery Services Vault | Centralized backup management |
| VM Backups | Application-consistent VM protection |

---

## Review Questions

1. What is the difference between Azure Metrics and Log Analytics?
2. Explain the components of an alert rule.
3. What telemetry types does Application Insights collect?
4. What are the storage redundancy options for Recovery Services Vault?
5. Describe the VM backup and restore process.
6. How do you configure diagnostic settings for backup reports?

---

## Additional Resources

- [Azure Monitor Overview](https://docs.microsoft.com/azure/azure-monitor/overview)
- [Log Analytics Documentation](https://docs.microsoft.com/azure/azure-monitor/logs/log-analytics-overview)
- [Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)
- [Azure Backup Documentation](https://docs.microsoft.com/azure/backup/)
- [KQL Reference](https://docs.microsoft.com/azure/data-explorer/kusto/query/)

---

## 10.6 Advanced Monitoring and Observability (Expert Level)

### KQL Advanced Queries

```kql
// Find failed deployments in last 24 hours
AzureActivity
| where TimeGenerated > ago(24h)
| where OperationNameValue contains "deployments/write"
| where ActivityStatusValue == "Failed"
| project TimeGenerated, Caller, ResourceGroup, 
          OperationNameValue, ActivityStatusValue, 
          Properties = parse_json(Properties)
| extend ErrorCode = tostring(Properties.statusCode),
         ErrorMessage = tostring(Properties.statusMessage)
| order by TimeGenerated desc

// Memory and CPU correlation with percentiles
Perf
| where TimeGenerated > ago(1h)
| where ObjectName == "Processor" or ObjectName == "Memory"
| where CounterName == "% Processor Time" or CounterName == "% Used Memory"
| summarize 
    AvgValue = avg(CounterValue),
    P50 = percentile(CounterValue, 50),
    P95 = percentile(CounterValue, 95),
    P99 = percentile(CounterValue, 99)
    by Computer, ObjectName, CounterName, bin(TimeGenerated, 5m)
| order by TimeGenerated desc

// Anomaly detection for requests
let sensitivity = 1.5;
requests
| where timestamp > ago(24h)
| summarize count() by bin(timestamp, 5m)
| extend series_decompose_anomalies(count_, sensitivity)
| where series_decompose_anomalies_count__ad_flag == 1

// Cross-resource queries
union 
    (AzureDiagnostics | where ResourceType == "APPLICATIONGATEWAYS"),
    (AzureDiagnostics | where ResourceType == "AZUREFIREWALLS"),
    (AzureDiagnostics | where ResourceType == "LOADBALANCERS")
| where TimeGenerated > ago(1h)
| summarize count() by ResourceType, Category
```

### Application Insights Custom Telemetry

```csharp
// C# - Custom telemetry
public class OrderProcessor
{
    private readonly TelemetryClient _telemetry;

    public OrderProcessor(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }

    public async Task ProcessOrder(Order order)
    {
        using var operation = _telemetry.StartOperation<RequestTelemetry>("ProcessOrder");
        operation.Telemetry.Properties["OrderId"] = order.Id.ToString();
        operation.Telemetry.Properties["CustomerId"] = order.CustomerId;
        
        try
        {
            var stopwatch = Stopwatch.StartNew();
            
            // Track dependency call
            using (_telemetry.StartOperation<DependencyTelemetry>("PaymentService"))
            {
                await ProcessPayment(order);
            }
            
            // Track custom metric
            _telemetry.TrackMetric("OrderProcessingTimeMs", stopwatch.ElapsedMilliseconds);
            
            // Track custom event
            _telemetry.TrackEvent("OrderCompleted", new Dictionary<string, string>
            {
                { "OrderId", order.Id.ToString() },
                { "Amount", order.Total.ToString("C") }
            }, new Dictionary<string, double>
            {
                { "ItemCount", order.Items.Count }
            });
            
            operation.Telemetry.Success = true;
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
            operation.Telemetry.Success = false;
            throw;
        }
    }
}
```

### Alert Rules with Action Groups

```powershell
# Create Action Group
$emailReceiver = New-AzActionGroupReceiver `
    -Name "EmailOps" `
    -EmailReceiver `
    -EmailAddress "ops@company.com"

$smsReceiver = New-AzActionGroupReceiver `
    -Name "SMSOps" `
    -SmsReceiver `
    -CountryCode "1" `
    -PhoneNumber "5551234567"

$webhookReceiver = New-AzActionGroupReceiver `
    -Name "TeamsWebhook" `
    -WebhookReceiver `
    -ServiceUri "https://outlook.office.com/webhook/..."

$actionGroup = Set-AzActionGroup `
    -ResourceGroupName "Monitoring-RG" `
    -Name "CriticalAlerts" `
    -ShortName "Critical" `
    -Receiver @($emailReceiver, $smsReceiver, $webhookReceiver)

# Create metric alert for high CPU
$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "Percentage CPU" `
    -MetricNamespace "Microsoft.Compute/virtualMachines" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 90

Add-AzMetricAlertRuleV2 `
    -ResourceGroupName "Monitoring-RG" `
    -Name "HighCPU-Alert" `
    -Severity 2 `
    -WindowSize 00:05:00 `
    -Frequency 00:01:00 `
    -TargetResourceId $vm.Id `
    -Condition $condition `
    -ActionGroupId $actionGroup.Id

# Create log-based alert
$source = New-AzScheduledQueryRuleSource `
    -Query "exceptions | where timestamp > ago(5m) | summarize count() by problemId | where count_ > 10" `
    -DataSourceId $appInsights.Id `
    -QueryType ResultCount

$schedule = New-AzScheduledQueryRuleSchedule `
    -FrequencyInMinutes 5 `
    -TimeWindowInMinutes 5

$triggerCondition = New-AzScheduledQueryRuleTriggerCondition `
    -ThresholdOperator GreaterThan `
    -Threshold 0

$aznsActionGroup = New-AzScheduledQueryRuleAznsActionGroup `
    -ActionGroup $actionGroup.Id

New-AzScheduledQueryRule `
    -ResourceGroupName "Monitoring-RG" `
    -Name "HighExceptionRate" `
    -Location "eastus" `
    -Source $source `
    -Schedule $schedule `
    -Action $aznsActionGroup `
    -TriggerCondition $triggerCondition
```

### Azure Monitor Workbooks

```json
{
  "version": "Notebook/1.0",
  "items": [
    {
      "type": "query",
      "name": "Infrastructure Health",
      "query": "Heartbeat | summarize LastHeartbeat = max(TimeGenerated) by Computer | extend Status = iff(LastHeartbeat > ago(5m), 'Healthy', 'Unhealthy')",
      "visualization": "tiles",
      "tileSettings": {
        "showBorder": true,
        "titleContent": { "columnMatch": "Computer" },
        "subtitleContent": { "columnMatch": "Status" }
      }
    },
    {
      "type": "query", 
      "name": "Resource Utilization Trend",
      "query": "Perf | where ObjectName == 'Processor' | summarize avg(CounterValue) by bin(TimeGenerated, 5m), Computer | render timechart"
    }
  ]
}
```

### Azure Site Recovery

```powershell
# Create Recovery Services Vault
$vault = New-AzRecoveryServicesVault `
    -ResourceGroupName "DR-RG" `
    -Name "Enterprise-RSV" `
    -Location "westus2"

Set-AzRecoveryServicesBackupProperty `
    -Vault $vault `
    -BackupStorageRedundancy GeoRedundant

# Enable replication for VM (Azure to Azure)
$sourceVM = Get-AzVM -ResourceGroupName "Prod-RG" -Name "CriticalServer"

$replicationJob = New-AzRecoveryServicesAsrReplicationProtectedItem `
    -AzureToAzure `
    -AzureVmId $sourceVM.Id `
    -Name "CriticalServer-Replication" `
    -ProtectionContainerMapping $containerMapping `
    -RecoveryResourceGroupId $drResourceGroup.ResourceId `
    -RecoveryAzureNetworkId $drVNet.Id `
    -RecoveryAzureSubnetName "default" `
    -RecoveryVmName "CriticalServer-DR"

# Monitor replication health
Get-AzRecoveryServicesAsrReplicationProtectedItem `
    -ProtectionContainer $protectionContainer | 
    Select-Object FriendlyName, ProtectionState, ReplicationHealth, 
                  ActiveLocation, TestFailoverState

# Test failover
$protectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem `
    -ProtectionContainer $protectionContainer `
    -FriendlyName "CriticalServer"

Start-AzRecoveryServicesAsrTestFailoverJob `
    -ReplicationProtectedItem $protectedItem `
    -AzureVMNetworkId $testVNet.Id `
    -Direction PrimaryToRecovery
```

### Observability Best Practices

| Layer | Tools | Purpose |
|-------|-------|--------|
| **Infrastructure** | Azure Monitor, VM Insights | Resource health, performance |
| **Platform** | Container Insights, AKS diagnostics | Kubernetes monitoring |
| **Application** | Application Insights | APM, distributed tracing |
| **Security** | Defender for Cloud, Sentinel | Threat detection |
| **Cost** | Cost Management | Budget alerts, optimization |
