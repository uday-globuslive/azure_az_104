# Module 11 - Azure Migration (On-Premises to Azure)

## Learning Objectives
By the end of this module, you will be able to:
- Understand the Azure Migration framework and methodology
- Explain the 5 stages of cloud migration (Cloud Adoption Framework)
- Plan and execute on-premises to Azure migration
- Use Azure Migrate service for discovery and assessment
- Choose the right migration strategy (6 R's)
- Migrate VMs, databases, and web apps to Azure
- Understand post-migration optimization

---

## 11.1 Why Migrate to Azure?

### Business Drivers

| Driver | Description |
|--------|-------------|
| **Cost Reduction** | Eliminate hardware refresh cycles and datacenter costs |
| **Scalability** | Scale resources up/down on demand |
| **Disaster Recovery** | Built-in geo-redundancy and backup |
| **Security** | Microsoft manages physical security, compliance |
| **Innovation** | Access to AI, ML, analytics services |
| **Agility** | Faster deployment of new services |

### Common Migration Scenarios

```
┌─────────────────────────────────────────────────────────────────┐
│              Common On-Premises to Azure Migrations              │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Physical    │  │  VMware/     │  │  Windows     │          │
│  │  Servers     │  │  Hyper-V VMs │  │  Workloads   │          │
│  │  ──────────► │  │  ──────────► │  │  ──────────► │          │
│  │  Azure VMs   │  │  Azure VMs   │  │  Azure VMs   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  SQL Server  │  │  Web Apps    │  │  File        │          │
│  │  On-prem     │  │  IIS         │  │  Servers     │          │
│  │  ──────────► │  │  ──────────► │  │  ──────────► │          │
│  │  Azure SQL   │  │  Azure App   │  │  Azure Files │          │
│  │              │  │  Service     │  │              │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11.2 Cloud Adoption Framework (CAF)

### What is CAF?
Microsoft's Cloud Adoption Framework is a collection of documentation, implementation guidance, and tools to help organizations accelerate cloud adoption.

### The 5 Stages of Migration (CAF)

```
┌──────────────────────────────────────────────────────────────────────┐
│                  Cloud Adoption Framework Stages                      │
│                                                                       │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐            │
│   │ STAGE 1 │──►│ STAGE 2 │──►│ STAGE 3 │──►│ STAGE 4 │──►STAGE 5  │
│   │ STRATEGY│   │  PLAN   │   │  READY  │   │ MIGRATE │   MANAGE   │
│   └─────────┘   └─────────┘   └─────────┘   └─────────┘            │
│                                                                       │
│   Define        Build        Prepare       Execute     Optimize      │
│   Motivation    Migration    Azure         Migration   & Govern      │
│   & Goals       Backlog      Landing       Plan                      │
│                              Zone                                     │
└──────────────────────────────────────────────────────────────────────┘
```

### Stage 1 - Strategy

| Activity | Description |
|----------|-------------|
| **Motivations** | Cost savings, datacenter exit, risk reduction |
| **Business Outcomes** | Define measurable goals |
| **Business Case** | Financial justification |
| **First Adoption Project** | Choose pilot project |

Key Questions:
- Why are we moving to cloud?
- What does success look like?
- What is the ROI?

### Stage 2 - Plan

| Activity | Description |
|----------|-------------|
| **Digital Estate Assessment** | Inventory all workloads |
| **Initial Organization Alignment** | Define cloud team structure |
| **Skills Readiness Plan** | Training requirements |
| **Migration Backlog** | Prioritized list of workloads |

### Stage 3 - Ready (Landing Zone)

An Azure Landing Zone is a pre-configured environment with:
- Management groups and subscriptions
- Azure Policy baseline
- Role-based access control
- Network topology
- Logging and monitoring

```
┌─────────────────────────────────────────────────────────┐
│                    Azure Landing Zone                    │
│                                                          │
│   ┌─────────────────────────────────────────────────┐   │
│   │            Management Group Hierarchy            │   │
│   │   Root → Platform → Workload → Sandbox          │   │
│   └─────────────────────────────────────────────────┘   │
│                                                          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│   │  Identity &  │  │  Network     │  │  Management  │  │
│   │  Access      │  │  (Hub-Spoke) │  │  (Monitor,   │  │
│   │  (RBAC, PIM) │  │              │  │   Backup)    │  │
│   └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Stage 4 - Migrate

Execute migration using tools like Azure Migrate, Database Migration Service, etc.

### Stage 5 - Manage

Post-migration operations:
- Monitor workloads
- Optimize costs
- Govern with Azure Policy
- Ensure security compliance

---

## 11.3 The 6 R's Migration Strategies

### What are the 6 R's?
These are the six approaches to migrating workloads to the cloud. Choosing the right strategy depends on the workload type, business requirements, and time constraints.

```
┌──────────────────────────────────────────────────────────────────────┐
│                      6 R's Migration Strategies                       │
│                                                                       │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐         │
│  │  1. REHOST     │  │  2. REFACTOR   │  │  3. REARCHITECT│         │
│  │  (Lift & Shift)│  │  (Minor change)│  │  (Re-design)   │         │
│  │                │  │                │  │                │         │
│  │  Fastest       │  │  Some code     │  │  Rebuild for   │         │
│  │  No code change│  │  change needed │  │  cloud-native  │         │
│  └────────────────┘  └────────────────┘  └────────────────┘         │
│                                                                       │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐         │
│  │  4. REBUILD    │  │  5. REPLACE    │  │  6. RETIRE     │         │
│  │  (Rewrite)     │  │  (SaaS/PaaS)   │  │  (Decommission)│         │
│  │                │  │                │  │                │         │
│  │  Start fresh   │  │  Use existing  │  │  Decommission  │         │
│  │  cloud-native  │  │  SaaS solution │  │  unused apps   │         │
│  └────────────────┘  └────────────────┘  └────────────────┘         │
└──────────────────────────────────────────────────────────────────────┘
```

### Detailed Explanation

#### 1. Rehost (Lift & Shift)
- **What**: Move workload as-is to Azure
- **When**: Quick migration, legacy apps, mainframe
- **Tool**: Azure Migrate
- **Example**: Move on-prem VM → Azure VM (same config)
- **Pros**: Fast, low risk, no code changes
- **Cons**: Not optimized for cloud, same cost as on-prem

#### 2. Refactor
- **What**: Minor code changes to fit PaaS
- **When**: App can run on managed services with small changes
- **Tool**: App Service Migration Assistant
- **Example**: IIS Web App → Azure App Service
- **Pros**: Better performance, managed infrastructure
- **Cons**: Some development work required

#### 3. Rearchitect
- **What**: Redesign the application for cloud
- **When**: App needs to scale, modernization needed
- **Tool**: Azure Kubernetes Service, Microservices
- **Example**: Monolithic app → Microservices on AKS
- **Pros**: Full cloud-native benefits
- **Cons**: High effort, significant development time

#### 4. Rebuild
- **What**: Rewrite the application from scratch
- **When**: Existing app is outdated, new requirements
- **Tool**: Azure Functions, Logic Apps, API Management
- **Example**: Legacy ERP → Cloud-native serverless app
- **Pros**: Modern architecture, fully optimized
- **Cons**: Most expensive, longest timeline

#### 5. Replace
- **What**: Replace with SaaS solution
- **When**: Commercial software exists for the purpose
- **Tool**: Microsoft 365, Dynamics 365, third-party SaaS
- **Example**: Custom CRM → Dynamics 365 / Salesforce
- **Pros**: No management overhead
- **Cons**: Loss of customization

#### 6. Retire
- **What**: Decommission unused or redundant applications
- **When**: App has no business value or is duplicate
- **Tool**: Assessment tools to identify unused apps
- **Example**: Old reporting system replaced by Power BI
- **Pros**: Reduces costs, simplifies portfolio
- **Cons**: May affect users who still use it

### Decision Matrix

| Factor | Rehost | Refactor | Rearchitect | Rebuild | Replace | Retire |
|--------|--------|----------|-------------|---------|---------|--------|
| Speed | ★★★★★ | ★★★★ | ★★ | ★ | ★★★ | ★★★★★ |
| Cloud benefit | ★★ | ★★★ | ★★★★ | ★★★★★ | ★★★★ | N/A |
| Cost (upfront) | Low | Medium | High | Very High | Low | Low |
| Risk | Low | Low | Medium | High | Medium | Low |

---

## 11.4 Azure Migrate Service

### What is Azure Migrate?
Azure Migrate is a centralized hub for migrating on-premises servers, databases, web apps, and virtual desktops to Azure.

### Azure Migrate Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                          Azure Migrate Hub                             │
│                                                                        │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                         Tools                                   │  │
│  │  ┌──────────────────┐        ┌──────────────────┐              │  │
│  │  │   Discovery &    │        │   Migration      │              │  │
│  │  │   Assessment     │        │   Execution      │              │  │
│  │  │                  │        │                  │              │  │
│  │  │  • Server Assess │        │  • Server Migrate│              │  │
│  │  │  • Database Assess        │  • Database Mig. │              │  │
│  │  │  • Web App Assess│        │  • Web App Mig.  │              │  │
│  │  └──────────────────┘        └──────────────────┘              │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                              │                                         │
│                              ▼                                         │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                  Azure Migrate Appliance                        │  │
│  │              (Deployed On-Premises)                             │  │
│  │                                                                  │  │
│  │   Discovers VMs, databases, web apps, dependencies             │  │
│  └────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────┘
```

### Azure Migrate Appliance

A lightweight virtual appliance you deploy on-premises to:
- Discover VMware VMs, Hyper-V VMs, and physical servers
- Collect metadata (config, performance)
- Send data to Azure Migrate project

### Setting Up Azure Migrate

#### Step 1: Create Azure Migrate Project

```powershell
# Register provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.Migrate"

# Create resource group
New-AzResourceGroup -Name "Migration-RG" -Location "East US"

# Create Migrate project (via Portal or REST API)
# Azure Portal: All Services → Azure Migrate → Create Project
```

```bash
# Azure CLI
az group create --name "Migration-RG" --location "eastus"

az migrate project create \
    --resource-group "Migration-RG" \
    --name "MyMigrationProject" \
    --location "eastus"
```

#### Step 2: Deploy Azure Migrate Appliance

```
For VMware:
1. Download OVA template from Azure Portal
2. Deploy as VM in vCenter
3. Configure appliance with Azure credentials
4. Start discovery

For Hyper-V:
1. Download VHD template from Azure Portal
2. Create VM in Hyper-V Manager
3. Configure appliance
4. Start discovery

For Physical Servers:
1. Download installer from Azure Portal
2. Install on a Windows Server
3. Configure and start discovery
```

#### Step 3: Discover and Assess

```
Discovery Process:
On-prem VMs → Appliance → (over HTTPS) → Azure Migrate → Assessment Report

Assessment Report shows:
- Recommended Azure VM sizes
- Estimated monthly costs
- Readiness status (Ready/Conditionally Ready/Not Ready)
- Dependency map
```

---

## 11.5 Step-by-Step On-Premises to Azure Migration

### Full Migration Workflow

```
┌────────────────────────────────────────────────────────────────────┐
│             Complete On-Premises to Azure Migration Journey         │
│                                                                     │
│  PHASE 1: PREPARE              PHASE 2: ASSESS                     │
│  ┌─────────────────────┐       ┌─────────────────────┐            │
│  │ • Inventory workloads│       │ • Run Azure Migrate  │            │
│  │ • Define scope       │       │ • Analyze reports    │            │
│  │ • Set up Azure sub.  │       │ • Identify blockers  │            │
│  │ • Create landing zone│       │ • Plan VM sizes      │            │
│  │ • Set up connectivity│       │ • Estimate costs     │            │
│  └─────────────────────┘       └─────────────────────┘            │
│                                                                     │
│  PHASE 3: MIGRATE              PHASE 4: OPTIMIZE                   │
│  ┌─────────────────────┐       ┌─────────────────────┐            │
│  │ • Replication setup  │       │ • Resize VMs         │            │
│  │ • Test migrations    │       │ • Reserved instances │            │
│  │ • Cutover planning   │       │ • Azure Hybrid Benefit│           │
│  │ • Execute cutover    │       │ • Enable monitoring  │            │
│  │ • Validate & test    │       │ • Implement backup   │            │
│  └─────────────────────┘       └─────────────────────┘            │
└────────────────────────────────────────────────────────────────────┘
```

### Phase 1: Prepare

#### 1.1 Inventory & Discovery

```powershell
# Run on-prem to collect server inventory
Get-ADComputer -Filter * -Properties * | 
    Select-Object Name, OperatingSystem, IPv4Address |
    Export-Csv "ServerInventory.csv"

# Collect running services and applications
Get-Service | Where-Object {$_.Status -eq "Running"} | 
    Select-Object Name, DisplayName, Status |
    Export-Csv "Services.csv"

# Disk usage
Get-WmiObject Win32_LogicalDisk | 
    Select-Object DeviceID, Size, FreeSpace |
    Export-Csv "DiskInventory.csv"
```

#### 1.2 Azure Subscription Setup

```powershell
# Connect to Azure
Connect-AzAccount

# Create management group structure
New-AzManagementGroup -GroupName "Production" -DisplayName "Production"

# Create subscriptions (requires billing account access)
# Then organize under management groups

# Set up hub-spoke network
New-AzResourceGroup -Name "Network-Hub-RG" -Location "East US"
New-AzVirtualNetwork `
    -Name "Hub-VNet" `
    -ResourceGroupName "Network-Hub-RG" `
    -Location "East US" `
    -AddressPrefix "10.0.0.0/16"
```

### Phase 2: Assessment

#### 2.1 VM Assessment Results

```
Azure Migrate Assessment Report Example:

Server Name    | On-Prem Size | Recommended Azure Size | Monthly Cost | Readiness
──────────────────────────────────────────────────────────────────────────────────
WebServer01    | 4 CPU, 16 GB | Standard_D4s_v3       | $140/month   | Ready
SQLServer01    | 8 CPU, 64 GB | Standard_E8s_v4        | $450/month   | Ready
AppServer01    | 2 CPU, 8 GB  | Standard_D2s_v3        | $70/month    | Ready
LegacyApp01    | 4 CPU, 8 GB  | Standard_D4s_v3        | $140/month   | Conditionally Ready
```

#### 2.2 Readiness Status

| Status | Meaning |
|--------|---------|
| **Ready** | Can be migrated as-is |
| **Conditionally Ready** | Minor issues, can migrate with fixes |
| **Not Ready** | Significant blockers, needs rework |
| **Unknown** | Insufficient data |

### Phase 3: Migration - VM Migration (Rehost)

#### Step 1: Set Up Replication

```
Azure Portal: Azure Migrate → Servers → Migrate
→ Replicate → Select Source (VMware/Hyper-V/Physical)
→ Select VMs to migrate
→ Target settings (Resource Group, VNet, etc.)
→ Start replication (takes time based on data size)
```

#### Step 2: Test Migration

```
IMPORTANT: Always test before cutting over!

Azure Portal: Azure Migrate → Replicating machines
→ Select VM → Test Migration
→ Verify VM boots and works correctly
→ Test application connectivity
→ Validate database connections
→ Run smoke tests
→ Clean up test migration
```

#### Step 3: Execute Cutover

```
Cutover Checklist:
□ Notify stakeholders of maintenance window
□ Schedule low-traffic window
□ Take final backup
□ Stop on-premises VM (to prevent data divergence)
□ Execute cutover in Azure Migrate
□ Update DNS records to point to Azure VM IP
□ Update any firewall rules
□ Validate application functionality
□ Monitor for 24-48 hours
□ Decommission on-premises VM after validation
```

```powershell
# Update DNS after migration
# Update A record to point to new Azure VM IP

$newIP = (Get-AzPublicIpAddress -ResourceGroupName "Migration-RG" -Name "WebServer01-pip").IpAddress
Write-Host "New Azure VM IP: $newIP"
Write-Host "Update DNS A record for webserver01.contoso.com to $newIP"
```

---

## 11.6 Database Migration

### Azure Database Migration Service (DMS)

Azure DMS migrates databases to Azure with minimal downtime.

### Supported Migration Paths

| Source | Target |
|--------|--------|
| SQL Server | Azure SQL Database |
| SQL Server | Azure SQL Managed Instance |
| SQL Server | SQL Server on Azure VM |
| Oracle | Azure Database for PostgreSQL |
| MySQL | Azure Database for MySQL |
| PostgreSQL | Azure Database for PostgreSQL |
| MongoDB | Azure Cosmos DB |

### SQL Server Migration Process

```
┌──────────────────────────────────────────────────────────────────┐
│                   SQL Server Migration Flow                       │
│                                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐   │
│  │  Assess  │───►│  Plan    │───►│  Migrate │───►│  Cutover │   │
│  │          │    │          │    │          │    │          │   │
│  │ SKA tool │    │ Target   │    │ Initial  │    │ Stop     │   │
│  │ Azure    │    │ Azure SQL│    │ Full load│    │ source DB│   │
│  │ Assess.  │    │ vs MI vs │    │ CDC sync │    │ Cutover  │   │
│  │          │    │ SQL VM   │    │ (ongoing)│    │ to Azure │   │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

### Choosing SQL Target

| Scenario | Recommended Target |
|----------|-------------------|
| Full compatibility required | SQL Server on Azure VM |
| Managed service, most features | Azure SQL Managed Instance |
| Simple apps, cost-effective | Azure SQL Database |
| Hyperscale, very large DBs | Azure SQL Hyperscale |

### Running SQL Assessment

```powershell
# Install Data Migration Assistant (DMA)
# Download: https://www.microsoft.com/download/details.aspx?id=53595

# Or use Azure Database Migration Service
# 1. Create DMS Instance
az dms create \
    --resource-group "Migration-RG" \
    --name "MyDMS" \
    --location "eastus" \
    --sku-name "GeneralPurpose_4vCores"

# 2. Create migration project
az dms project create \
    --resource-group "Migration-RG" \
    --service-name "MyDMS" \
    --name "SQLMigrationProject" \
    --source-platform "SQL" \
    --target-platform "SQLDB" \
    --location "eastus"
```

### Online vs Offline Migration

| Type | Description | Downtime |
|------|-------------|----------|
| **Offline** | Stop source, migrate, start target | Hours |
| **Online** | CDC replication until cutover | Minutes |

---

## 11.7 Web Application Migration

### App Service Migration Assistant

Microsoft provides a free tool to migrate IIS web applications to Azure App Service.

### Supported Scenarios

| Source | Target |
|--------|--------|
| IIS on Windows Server | Azure App Service |
| .NET Web Apps | Azure App Service |
| PHP on IIS | Azure App Service |

### Migration Steps

```
1. Run App Service Migration Assistant on source server
2. Tool analyzes IIS configuration and compatibility
3. Review readiness report and fix issues
4. Select target App Service Plan
5. Tool deploys and configures App Service
6. Update DNS to point to App Service URL
```

### Containerization for Legacy Apps

```powershell
# Use App Containerization tool for container migration
# Supports: ASP.NET, Java web apps
# Target: AKS or Azure App Service (container)

# General process:
# 1. Install Azure Migrate: App Containerization tool
# 2. Discover web apps on IIS
# 3. Tool builds Docker image
# 4. Push to Azure Container Registry
# 5. Deploy to AKS or App Service Container
```

---

## 11.8 Migration Tools Summary

### Azure Native Tools

| Tool | Purpose |
|------|---------|
| **Azure Migrate** | Server discovery, assessment, migration |
| **Azure Database Migration Service** | Database migrations |
| **App Service Migration Assistant** | IIS → App Service |
| **Azure Data Box** | Offline data transfer (large datasets) |
| **AzCopy** | Data transfer to Azure Storage |
| **Azure Site Recovery** | DR + migration |

### Third-Party Tools (ISV Partners)

| Tool | Vendor | Use Case |
|------|--------|----------|
| Carbonite Migrate | Carbonite | Physical/virtual migrations |
| CloudEndure | AWS (now AWS MGN) | Lift & shift |
| Movere | Microsoft | Discovery/assessment |
| Turbonomic | IBM | Optimization + migration |

---

## 11.9 Cost Optimization After Migration

### Azure Hybrid Benefit

Save up to 85% by reusing existing Windows Server and SQL Server licenses.

```powershell
# Apply Azure Hybrid Benefit to existing VM
$vm = Get-AzVM -ResourceGroupName "Production-RG" -Name "WebServer01"
$vm.LicenseType = "Windows_Server"
Update-AzVM -ResourceGroupName "Production-RG" -VM $vm

Write-Host "Azure Hybrid Benefit applied to WebServer01"
```

### Reserved Instances

Commit to 1 or 3 years to save up to 72% vs pay-as-you-go.

| Term | Discount | Best For |
|------|----------|----------|
| 1 year | ~40% | Stable workloads |
| 3 year | ~60-72% | Long-term stable workloads |

### Right-Sizing

```powershell
# Check CPU/Memory utilization to right-size
$insights = Get-AzMetric `
    -ResourceId $vm.Id `
    -MetricName "Percentage CPU" `
    -StartTime (Get-Date).AddDays(-30) `
    -EndTime (Get-Date) `
    -TimeGrain 01:00:00 `
    -AggregationType Average

$avgCPU = ($insights.Data | Measure-Object -Property Average -Average).Average
Write-Host "Average CPU over 30 days: $([math]::Round($avgCPU, 2))%"
Write-Host "If < 20%, consider downsizing the VM"
```

---

## 11.10 Post-Migration Checklist

```
POST-MIGRATION VALIDATION:
□ Application accessible from expected locations
□ All services started and running
□ Database connections working
□ Authentication working (AD, Azure AD)
□ SSL certificates valid
□ DNS resolving correctly
□ Monitoring configured (Azure Monitor)
□ Backup configured (Recovery Services Vault)
□ Alerts set up for critical metrics
□ Auto-shutdown configured for non-prod
□ Resource locks applied to critical resources
□ RBAC roles assigned correctly
□ NSG rules verified
□ Log Analytics connected
□ Cost alerts configured

ON-PREMISES CLEANUP (after 30-day validation):
□ Decommission on-prem VMs
□ Reclaim storage and licenses
□ Update CMDB / asset management
□ Archive or dispose of hardware
□ Update documentation
```

---

## Interview Tips - Migration Topics

### Commonly Asked Interview Questions

**Q1: What is Lift & Shift migration?**
> Moving workloads from on-prem to Azure as-is, without code changes. Uses Azure Migrate. Fastest approach but doesn't take full advantage of cloud.

**Q2: What are the 6 R's?**
> Rehost, Refactor, Rearchitect, Rebuild, Replace, Retire. Each represents a different level of cloud adoption depth.

**Q3: What tool would you use to migrate VMware VMs to Azure?**
> Azure Migrate with the Server Migration tool. Deploy the Azure Migrate Appliance in vCenter for discovery, assess readiness, replicate VMs, test migration, then cutover.

**Q4: How would you migrate a SQL Server database with minimal downtime?**
> Use Azure Database Migration Service (DMS) in online migration mode. It performs an initial full load then uses Change Data Capture (CDC) to keep databases in sync until cutover.

**Q5: What is an Azure Landing Zone?**
> A pre-configured Azure environment with management groups, RBAC, networking (hub-spoke), policy baseline, and monitoring. It's the foundation for enterprise migration.

**Q6: How do you estimate Azure costs before migration?**
> Use Azure Migrate assessment report, Azure Pricing Calculator, and Total Cost of Ownership (TCO) Calculator.

**Q7: What is Azure Hybrid Benefit?**
> A licensing benefit that lets you use existing on-premises Windows Server and SQL Server licenses on Azure, saving up to 85% on licensing costs.

---

## Hands-on Exercises

### Exercise 1: Create Azure Migrate Project and Simulate Assessment

```powershell
# Step 1: Register providers
Register-AzResourceProvider -ProviderNamespace "Microsoft.Migrate"
Register-AzResourceProvider -ProviderNamespace "Microsoft.OffAzure"

# Step 2: Create resource group
New-AzResourceGroup -Name "Migration-Lab-RG" -Location "East US"

# Step 3: Explore Azure Migrate in Portal
Write-Host "Navigate to Azure Portal > Azure Migrate > Servers, databases and web apps"
Write-Host "Click 'Create project' and fill in:"
Write-Host "  - Subscription: Your subscription"
Write-Host "  - Resource Group: Migration-Lab-RG"
Write-Host "  - Project Name: MyMigrationProject"
Write-Host "  - Geography: United States"
Write-Host ""
Write-Host "After creation, explore:"
Write-Host "  - Discover: Deploy appliance to discover on-prem VMs"
Write-Host "  - Assess: Create assessment from discovered VMs"
Write-Host "  - Migrate: Replicate and migrate VMs"
```

### Exercise 2: Use Azure TCO Calculator

```
1. Navigate to https://azure.microsoft.com/en-us/pricing/tco/calculator/
2. Define workloads (Servers, Databases, Storage, Networking)
3. Adjust assumptions (Energy, Labor, Currency)
4. View 5-year TCO comparison
5. Download the report
```

### Exercise 3: Enable Azure Hybrid Benefit

```powershell
# Connect to Azure
Connect-AzAccount

# List VMs without Hybrid Benefit
Get-AzVM | Where-Object {$_.LicenseType -ne "Windows_Server"} |
    Select-Object Name, ResourceGroupName |
    Format-Table

# Apply Hybrid Benefit to a VM
$vm = Get-AzVM -ResourceGroupName "Migration-Lab-RG" -Name "TestVM"
$vm.LicenseType = "Windows_Server"
Update-AzVM -ResourceGroupName "Migration-Lab-RG" -VM $vm

Write-Host "Hybrid Benefit applied - you're now saving on Windows licensing!"
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Migration Drivers | Cost, agility, scalability, innovation |
| CAF Stages | Strategy → Plan → Ready → Migrate → Manage |
| 6 R's | Rehost, Refactor, Rearchitect, Rebuild, Replace, Retire |
| Azure Migrate | Discovery, assessment, replication, cutover |
| Database Migration | DMS for SQL, offline vs online migration |
| Web App Migration | App Service Migration Assistant, containerization |
| Post-Migration | Validate, monitor, backup, optimize costs |

---

## Additional Resources

- [Azure Migrate Documentation](https://docs.microsoft.com/azure/migrate/)
- [Cloud Adoption Framework](https://docs.microsoft.com/azure/cloud-adoption-framework/)
- [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/)
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)
- [TCO Calculator](https://azure.microsoft.com/pricing/tco/calculator/)
- [App Service Migration Assistant](https://appmigration.microsoft.com/)
