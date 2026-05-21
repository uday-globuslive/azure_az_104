# Module 14 - Azure Landing Zones & Governance
## (Management Groups, Subscriptions, RBAC, Policies, Hub-Spoke)

> **Beginner Note**: Imagine you're setting up a new office building for a company. You need floors, departments, security, access cards, rules for each area, and a common reception. Azure Landing Zone is exactly that — the structured foundation for your entire Azure environment before any applications move in.

---

## Learning Objectives
By the end of this module, you will be able to:
- Understand what an Azure Landing Zone is and why it matters
- Design Management Group hierarchies
- Manage Subscriptions (Prod/Non-Prod)
- Apply Azure Policy for governance
- Configure RBAC correctly
- Set up Hub-Spoke network topology
- Implement monitoring across Landing Zones
- Explain Landing Zones in interviews confidently

---

## 14.1 What is an Azure Landing Zone?

### Simple Analogy

```
BUILDING A HOUSE vs BUILDING AN ENTERPRISE

WITHOUT Landing Zone (bad):
- Build rooms randomly
- No wiring plan
- No security plan
- No shared services (plumbing, electricity)
- Hard to add more rooms later
→ CHAOS, security gaps, high cost

WITH Landing Zone (good):
- Foundation laid first
- Electricity, plumbing, internet all planned
- Security (locks, access cards) designed upfront
- Rooms built on solid foundation
- Easy to add rooms (workloads) later
→ ORGANIZED, secure, scalable
```

### Landing Zone = Ready Environment

A Landing Zone is a **pre-configured, secure Azure environment** that includes:

```
┌────────────────────────────────────────────────────────────────────┐
│                      Azure Landing Zone                             │
│                                                                     │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│   │   GOVERNANCE    │  │   IDENTITY      │  │   NETWORKING    │  │
│   │                 │  │                 │  │                 │  │
│   │ Management      │  │ Azure AD        │  │ Hub VNet        │  │
│   │ Groups          │  │ RBAC            │  │ Spoke VNets     │  │
│   │ Subscriptions   │  │ PIM             │  │ VPN/ExpressRoute│  │
│   │ Azure Policy    │  │ MFA             │  │ Firewall        │  │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘  │
│                                                                     │
│   ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│   │   SECURITY      │  │   OPERATIONS    │  │   PLATFORM      │  │
│   │                 │  │                 │  │                 │  │
│   │ Defender for    │  │ Azure Monitor   │  │ Shared Services │  │
│   │ Cloud           │  │ Log Analytics   │  │ DNS             │  │
│   │ Key Vault       │  │ Alerts          │  │ Image Gallery   │  │
│   │ DDoS Protection │  │ Backup          │  │ Update Mgmt     │  │
│   └─────────────────┘  └─────────────────┘  └─────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
```

### Why Landing Zone Matters (Interview Context)

When a company migrates 200+ applications to Azure:
- Without Landing Zone: every team does their own thing → security gaps, no control, high cost
- With Landing Zone: common foundation → consistent security, governance, networking for all 200+ apps

---

## 14.2 Management Groups

### What are Management Groups?

Management Groups are **containers that organize your Azure subscriptions**. Think of them as folders that group subscriptions together so you can apply policies and access control to all of them at once.

### Management Group Hierarchy

```
ROOT MANAGEMENT GROUP (tenant level - top of everything)
│
├── Platform (your shared infrastructure)
│   ├── Identity Subscription (Azure AD DS, etc.)
│   ├── Management Subscription (monitoring, logs)
│   └── Connectivity Subscription (hub network, VPN, DNS)
│
├── Landing Zones (your actual workloads)
│   ├── Production
│   │   ├── Prod-App1-Subscription
│   │   ├── Prod-App2-Subscription
│   │   └── Prod-App3-Subscription
│   └── Non-Production
│       ├── Dev-Subscription
│       ├── Test-Subscription
│       └── UAT-Subscription
│
└── Sandbox (for experiments and learning)
    └── Sandbox-Subscription
```

### Why Management Groups?

```
APPLYING POLICY TO ONE MANAGEMENT GROUP AFFECTS ALL SUBSCRIPTIONS BELOW:

Policy: "Only allowed Azure regions = UK South, UK West"
Applied to: "Landing Zones" Management Group
Result: ALL subscriptions under Landing Zones are restricted to UK regions

WITHOUT Management Groups:
Apply same policy to 20 subscriptions separately = 20 configurations
Apply same RBAC to 20 subscriptions separately = 20 configurations

WITH Management Groups:
Apply once to Management Group = ALL subscriptions inherit it
```

### Management Group Rules

| Rule | Detail |
|------|--------|
| **Max depth** | 6 levels (root + 5 levels) |
| **Max per directory** | 10,000 management groups |
| **Each subscription** | Can only be in ONE management group |
| **Root** | All subscriptions and groups are under root |
| **Inheritance** | Policies flow down the hierarchy |

### Creating Management Groups

```powershell
# Connect to Azure
Connect-AzAccount

# Create top-level management group
New-AzManagementGroup `
    -GroupName "LandingZones" `
    -DisplayName "Landing Zones"

# Create child management groups
New-AzManagementGroup `
    -GroupName "Production" `
    -DisplayName "Production" `
    -ParentId "/providers/Microsoft.Management/managementGroups/LandingZones"

New-AzManagementGroup `
    -GroupName "NonProduction" `
    -DisplayName "Non-Production" `
    -ParentId "/providers/Microsoft.Management/managementGroups/LandingZones"

New-AzManagementGroup `
    -GroupName "Sandbox" `
    -DisplayName "Sandbox" `
    -ParentId "/providers/Microsoft.Management/managementGroups/LandingZones"

# Move subscription into a management group
New-AzManagementGroupSubscription `
    -GroupName "Production" `
    -SubscriptionId "your-subscription-id"
```

```bash
# Azure CLI
az account management-group create \
    --name "LandingZones" \
    --display-name "Landing Zones"

az account management-group create \
    --name "Production" \
    --display-name "Production" \
    --parent "/providers/Microsoft.Management/managementGroups/LandingZones"

# Move subscription to management group
az account management-group subscription add \
    --name "Production" \
    --subscription "your-subscription-id"
```

---

## 14.3 Azure Subscriptions

### What is a Subscription?

A subscription is a **billing and management boundary** in Azure. Think of it as a separate account/ledger that contains resources and has its own bill.

```
SUBSCRIPTION = Container for:
├── Resources (VMs, databases, storage)
├── Billing (who pays)
├── Quotas (limits on resources)
└── Access control (who can manage)
```

### Why Multiple Subscriptions?

```
SMALL COMPANY (one subscription might work):
└── Company Subscription
    ├── Dev resources
    ├── Prod resources
    └── All teams

LARGE ENTERPRISE (multiple subscriptions needed):
├── Prod-AppA-Subscription     → separate billing, separate blast radius
├── Prod-AppB-Subscription     → if one sub has an issue, others unaffected
├── Dev-Subscription           → devs can experiment without touching prod
├── Management-Subscription    → centralized logging and monitoring
└── Connectivity-Subscription  → networking hub, VPN, DNS
```

### Subscription Design Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **By environment** | Prod/Dev/Test/UAT | Separate environments |
| **By application** | App1/App2/App3 | Large separate apps |
| **By business unit** | Finance/HR/Sales | Department billing |
| **By geography** | EastUS/WestEU | Compliance, data residency |

### Subscription Limits (Why Multiple Subscriptions Matter)

| Resource | Limit per Subscription |
|----------|----------------------|
| VNets | 1,000 |
| Public IP addresses | 1,000 |
| Virtual Machines | 25,000 |
| Resource Groups | 980 |

> When you hit limits, you need a new subscription!

### Managing Subscriptions

```powershell
# List all subscriptions
Get-AzSubscription

# Switch to a specific subscription
Set-AzContext -SubscriptionId "your-subscription-id"

# Get current subscription
Get-AzContext | Select-Object SubscriptionName, SubscriptionId

# Create management group subscription assignment
New-AzManagementGroupSubscription `
    -GroupName "Production" `
    -SubscriptionId "prod-subscription-id"
```

---

## 14.4 Azure Policy

### What is Azure Policy?

Azure Policy is a **rule engine** for Azure. You define rules (policies) and Azure enforces them automatically.

```
REAL-WORLD ANALOGY:

Company HR Policy:
Rule: "All employees must complete security training"
Effect: New employee joins → HR reminds them → Must complete before getting full access

Azure Policy:
Rule: "All VMs must have encryption enabled"
Effect: New VM created without encryption → Azure blocks it or flags it
```

### Policy Concepts

| Term | Explanation |
|------|-------------|
| **Policy Definition** | The rule itself (written in JSON) |
| **Policy Assignment** | Applying the rule to a scope |
| **Policy Initiative** | Group of policies (policy set) |
| **Scope** | Where it applies (MG, subscription, RG, resource) |
| **Effect** | What happens when rule is triggered |

### Policy Effects

| Effect | What Happens |
|--------|-------------|
| **Deny** | Block the resource creation/modification |
| **Audit** | Allow but flag as non-compliant (log it) |
| **AuditIfNotExists** | Audit if a related resource doesn't exist |
| **DeployIfNotExists** | Automatically deploy missing resources |
| **Append** | Add settings to the resource |
| **Modify** | Modify resource properties |
| **Disabled** | Policy is turned off |

### Example Policies

```
Policy 1: "Allowed Locations"
Rule: Resources can only be created in "UK South" or "UK West"
Effect: Deny
Result: Any attempt to create resource in East US is BLOCKED

Policy 2: "Require Tags on Resources"
Rule: All resources must have "Environment" and "Owner" tags
Effect: Deny
Result: Resource without required tags is BLOCKED

Policy 3: "Monitor VMs without Antimalware"
Rule: VMs should have antimalware extension installed
Effect: AuditIfNotExists
Result: VM without extension is FLAGGED in compliance report

Policy 4: "Auto-install Log Analytics Agent"
Rule: VMs should have Log Analytics agent
Effect: DeployIfNotExists
Result: VM created → Policy automatically installs the agent
```

### Creating Azure Policy

```powershell
# List built-in policy definitions
Get-AzPolicyDefinition | Where-Object {$_.Properties.DisplayName -like "*tag*"} |
    Select-Object Name, @{N="DisplayName";E={$_.Properties.DisplayName}} |
    Format-Table

# Assign a built-in policy: "Require a tag on resources"
$policy = Get-AzPolicyDefinition | Where-Object {
    $_.Properties.DisplayName -eq "Require a tag on resources"
}

New-AzPolicyAssignment `
    -Name "RequireEnvironmentTag" `
    -Scope "/subscriptions/your-sub-id" `
    -PolicyDefinition $policy `
    -PolicyParameterObject @{ tagName = "Environment" } `
    -Description "All resources must have an Environment tag"
```

### Custom Policy Definition

```json
{
  "mode": "All",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "not": {
            "field": "location",
            "in": ["uksouth", "ukwest"]
          }
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

```powershell
# Create custom policy definition
New-AzPolicyDefinition `
    -Name "AllowedLocationsUK" `
    -DisplayName "Only allow UK regions" `
    -Description "Resources must be in UK South or UK West" `
    -Policy '{"mode":"All","policyRule":{"if":{"allOf":[{"field":"location","notIn":["uksouth","ukwest"]}]},"then":{"effect":"deny"}}}' `
    -Mode "All"
```

### Policy Initiatives (Policy Sets)

Group multiple policies together and assign them as one.

```powershell
# Create an initiative (group of policies)
$policies = @(
    @{
        policyDefinitionId = "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62"  # Require tag
        parameters = @{ tagName = @{ value = "Environment" } }
    },
    @{
        policyDefinitionId = "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62"
        parameters = @{ tagName = @{ value = "Owner" } }
    }
)

New-AzPolicySetDefinition `
    -Name "CompanyBaselineGovernance" `
    -DisplayName "Company Baseline Governance" `
    -Description "Baseline policies for all workloads" `
    -PolicyDefinition (ConvertTo-Json $policies -Depth 5)
```

---

## 14.5 RBAC (Role-Based Access Control) in Landing Zones

### RBAC in Landing Zone Context

```
Management Group Level (broadest):
└── Assign: "Reader" to all managers (can see everything)

Subscription Level:
└── Assign: "Contributor" to dev team (can do anything in their sub)

Resource Group Level:
└── Assign: "VM Contributor" to ops team (can only manage VMs)

Resource Level (narrowest):
└── Assign: "Key Vault Secrets User" to a specific app
```

### Landing Zone RBAC Design

| Scope | Role | Who |
|-------|------|-----|
| Root Management Group | Owner | Cloud Platform Team only |
| Platform Management Group | Contributor | Platform Engineers |
| Production MG | Reader | All developers |
| Production MG | Contributor | Platform team only |
| Dev/Test Subscription | Contributor | Dev teams |
| Prod Subscription | Reader + specific roles | App teams |

### Common Roles in Enterprise

| Role | What They Can Do |
|------|----------------|
| **Global Admin** (Azure AD) | Everything in Azure AD |
| **Owner** | Everything + assign access |
| **Contributor** | Create/delete resources, no access control |
| **Reader** | View everything, change nothing |
| **Network Contributor** | Manage networking resources |
| **VM Contributor** | Manage VMs only |
| **Security Admin** | Manage security settings |
| **Billing Reader** | View billing information |

### PIM (Privileged Identity Management)

PIM provides **just-in-time** privileged access — you don't have admin access all the time; you request it when needed.

```
WITHOUT PIM:
User has "Owner" role permanently
→ Any compromise of user account = full access to everything

WITH PIM:
User has no standing privilege
User needs Owner access → requests it → gets approved → gets access for 1 hour → access expires
→ Compromise of account = no access without approval
```

```powershell
# PIM is configured via Azure AD Portal or Azure CLI
# Navigate to: Azure AD → Identity Governance → Privileged Identity Management

# Using PowerShell (requires AzureADPreview module)
# Activate eligible role
Enable-AzureADMSPrivilegedRoleAssignment `
    -ProviderId "aadRoles" `
    -ResourceId "tenant-id" `
    -RoleDefinitionId "owner-role-id" `
    -SubjectId "user-object-id" `
    -Type "UserAdd" `
    -AssignmentState "Active" `
    -schedule (New-Object Microsoft.Open.MSGraph.Model.GovernanceSchedule) `
    -Reason "Emergency production issue"
```

---

## 14.6 Hub-Spoke in Landing Zone Context

### Full Landing Zone Network Design

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    Landing Zone Network Architecture                      │
│                                                                           │
│   CONNECTIVITY SUBSCRIPTION (managed by platform team)                   │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │                          HUB VNET                                 │  │
│   │                       (10.0.0.0/16)                               │  │
│   │                                                                    │  │
│   │  GatewaySubnet    AzureFirewallSubnet  AzureBastionSubnet         │  │
│   │  10.0.0.0/27      10.0.1.0/26          10.0.2.0/26               │  │
│   │  ┌───────────┐    ┌───────────┐         ┌───────────┐             │  │
│   │  │VPN/ER GW  │    │  Azure    │         │  Bastion  │             │  │
│   │  │           │    │  Firewall │         │           │             │  │
│   │  └───────────┘    └───────────┘         └───────────┘             │  │
│   │         │                │                    │                   │  │
│   │         │          (inspects all traffic)      │                  │  │
│   └─────────┼──────────────┬─┼────────────────────┼───────────────────┘ │
│             │              │ │                    │                      │
│             │         VNet │ │ Peering            │                      │
│    ┌────────▼──────┐  ┌────▼─▼──────────┐  ┌─────▼───────────────┐     │
│    │ ON-PREMISES   │  │  SPOKE VNET 1   │  │   SPOKE VNET 2      │     │
│    │ 192.168.0.0/24│  │  (10.1.0.0/16)  │  │   (10.2.0.0/16)     │     │
│    │               │  │                 │  │                      │     │
│    │  DC, ERP, etc │  │  Production     │  │  Dev/Test            │     │
│    │               │  │  App workloads  │  │  workloads           │     │
│    └───────────────┘  └─────────────────┘  └──────────────────────┘     │
└──────────────────────────────────────────────────────────────────────────┘
```

### Key Networking Decisions

| Decision | Guidance |
|----------|---------|
| **How many spokes?** | One per subscription or application team |
| **Hub location** | Same region as majority of workloads |
| **Firewall** | Azure Firewall Premium for enterprise |
| **DNS** | Private DNS Resolver in Hub |
| **Connectivity** | VPN for dev, ExpressRoute for prod |

### Setting Up Hub-Spoke (Landing Zone)

```powershell
#============================================
# 1. CREATE CONNECTIVITY SUBSCRIPTION RESOURCES
#============================================

# Hub VNet
$hubVNet = New-AzVirtualNetwork `
    -Name "vnet-hub-connectivity" `
    -ResourceGroupName "rg-connectivity" `
    -Location "uksouth" `
    -AddressPrefix "10.0.0.0/16"

# Add required subnets
$subnetConfigs = @(
    @{ Name = "GatewaySubnet";           Prefix = "10.0.0.0/27" },
    @{ Name = "AzureFirewallSubnet";     Prefix = "10.0.1.0/26" },
    @{ Name = "AzureBastionSubnet";      Prefix = "10.0.2.0/26" },
    @{ Name = "RouteServerSubnet";       Prefix = "10.0.3.0/27" },
    @{ Name = "SharedServicesSubnet";    Prefix = "10.0.4.0/24" }
)

foreach ($s in $subnetConfigs) {
    Add-AzVirtualNetworkSubnetConfig `
        -Name $s.Name `
        -VirtualNetwork $hubVNet `
        -AddressPrefix $s.Prefix
}
$hubVNet | Set-AzVirtualNetwork
Write-Host "Hub VNet created"

#============================================
# 2. CREATE AZURE FIREWALL IN HUB
#============================================
$fwPIP = New-AzPublicIpAddress `
    -Name "pip-hub-firewall" `
    -ResourceGroupName "rg-connectivity" `
    -Location "uksouth" `
    -AllocationMethod Static `
    -Sku Standard

$fwIPConfig = New-AzFirewallIpConfiguration `
    -Name "FW-IPConfig" `
    -PublicIpAddress $fwPIP `
    -Subnet (Get-AzVirtualNetworkSubnetConfig -Name "AzureFirewallSubnet" -VirtualNetwork (Get-AzVirtualNetwork -Name "vnet-hub-connectivity" -ResourceGroupName "rg-connectivity"))

$firewall = New-AzFirewall `
    -Name "fw-hub" `
    -ResourceGroupName "rg-connectivity" `
    -Location "uksouth" `
    -IpConfiguration $fwIPConfig `
    -SkuTier "Premium"

$firewallPrivateIP = $firewall.IpConfigurations[0].PrivateIpAddress
Write-Host "Firewall Private IP: $firewallPrivateIP"

#============================================
# 3. CREATE SPOKE VNETS
#============================================
$spoke1VNet = New-AzVirtualNetwork `
    -Name "vnet-spoke-production" `
    -ResourceGroupName "rg-production-network" `
    -Location "uksouth" `
    -AddressPrefix "10.1.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "snet-web" -VirtualNetwork $spoke1VNet -AddressPrefix "10.1.1.0/24"
Add-AzVirtualNetworkSubnetConfig -Name "snet-app" -VirtualNetwork $spoke1VNet -AddressPrefix "10.1.2.0/24"
Add-AzVirtualNetworkSubnetConfig -Name "snet-data" -VirtualNetwork $spoke1VNet -AddressPrefix "10.1.3.0/24"
$spoke1VNet | Set-AzVirtualNetwork

#============================================
# 4. PEER HUB ↔ SPOKE
#============================================
# Hub → Spoke
Add-AzVirtualNetworkPeering `
    -Name "peer-hub-to-prod" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "vnet-hub-connectivity" -ResourceGroupName "rg-connectivity") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "vnet-spoke-production" -ResourceGroupName "rg-production-network").Id `
    -AllowForwardedTraffic `
    -AllowGatewayTransit

# Spoke → Hub
Add-AzVirtualNetworkPeering `
    -Name "peer-prod-to-hub" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "vnet-spoke-production" -ResourceGroupName "rg-production-network") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "vnet-hub-connectivity" -ResourceGroupName "rg-connectivity").Id `
    -AllowForwardedTraffic `
    -UseRemoteGateways

#============================================
# 5. ROUTE TABLE - Force traffic through Firewall
#============================================
$routes = @(
    New-AzRouteConfig -Name "rt-default" -AddressPrefix "0.0.0.0/0" -NextHopType VirtualAppliance -NextHopIpAddress $firewallPrivateIP,
    New-AzRouteConfig -Name "rt-onprem" -AddressPrefix "192.168.0.0/16" -NextHopType VirtualAppliance -NextHopIpAddress $firewallPrivateIP
)

$routeTable = New-AzRouteTable `
    -Name "rt-spoke-production" `
    -ResourceGroupName "rg-production-network" `
    -Location "uksouth" `
    -Route $routes

Write-Host "Hub-Spoke with Firewall routing configured!"
```

---

## 14.7 Azure Policy in Landing Zones

### Landing Zone Policy Examples

```
POLICIES APPLIED AT ROOT MANAGEMENT GROUP (affect everything):
1. Require Tags: Environment, Owner, CostCenter
2. Enable Microsoft Defender for Cloud
3. Configure diagnostic settings to central Log Analytics

POLICIES APPLIED AT PRODUCTION MANAGEMENT GROUP:
1. Deny creation in non-approved regions (UK only)
2. Require HTTPS only on web apps
3. Require encryption on storage accounts
4. Deny public IP addresses without approval
5. Require VM backup policy

POLICIES APPLIED AT DEV/TEST MANAGEMENT GROUP:
1. Auto-shutdown VMs at 8 PM (cost control)
2. Allow more regions (devs may test in US)
3. Restrict expensive VM sizes
```

### Deploying Policy at Scale with ARM/Bicep

```bicep
// Bicep - Policy Initiative for Landing Zone
resource policySet 'Microsoft.Authorization/policySetDefinitions@2021-06-01' = {
  name: 'LandingZoneBaseline'
  properties: {
    displayName: 'Landing Zone Baseline Policies'
    description: 'Core governance policies for all landing zones'
    policyType: 'Custom'
    policyDefinitions: [
      {
        policyDefinitionId: '/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'
        parameters: {
          tagName: { value: 'Environment' }
        }
      }
      {
        policyDefinitionId: '/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c'
        parameters: {
          listOfAllowedLocations: { value: [ 'uksouth', 'ukwest' ] }
        }
      }
    ]
  }
}
```

---

## 14.8 Monitoring in Landing Zones

### Centralized Monitoring Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│               Centralized Monitoring (Management Subscription)      │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────┐     │
│   │              Log Analytics Workspace (Central)           │     │
│   │                                                          │     │
│   │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────┐  │     │
│   │   │Prod Logs │  │Dev Logs  │  │Platform  │  │Audit │  │     │
│   │   │          │  │          │  │Logs      │  │Logs  │  │     │
│   │   └──────────┘  └──────────┘  └──────────┘  └──────┘  │     │
│   └──────────────────────────────────────────────────────────┘     │
│                              ▲                                      │
│                    Diagnostic Settings                              │
│            ┌─────────────────┼────────────────┐                   │
│            │                 │                │                   │
│    ┌────────▼─────┐  ┌────────▼─────┐  ┌──────▼──────┐          │
│    │  Production  │  │     Dev      │  │  Platform   │          │
│    │  Subscription│  │  Subscription│  │ Subscription│          │
│    └──────────────┘  └──────────────┘  └─────────────┘          │
└────────────────────────────────────────────────────────────────────┘
```

### Setting Up Centralized Logging

```powershell
# Create central Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "rg-management" `
    -Name "law-central-monitoring" `
    -Location "uksouth" `
    -Sku "PerGB2018" `
    -RetentionInDays 90

# Apply diagnostic settings policy via Azure Policy
# (forces all resources to send logs to central workspace)
$policyDef = Get-AzPolicyDefinition | Where-Object {
    $_.Properties.DisplayName -like "*diagnostic settings*Log Analytics*"
}

New-AzPolicyAssignment `
    -Name "DeployDiagnosticsToLogAnalytics" `
    -Scope "/providers/Microsoft.Management/managementGroups/LandingZones" `
    -PolicyDefinition $policyDef `
    -PolicyParameterObject @{
        logAnalytics = (Get-AzOperationalInsightsWorkspace -Name "law-central-monitoring" -ResourceGroupName "rg-management").ResourceId
    } `
    -AssignIdentity `
    -Location "uksouth"    # Required for DeployIfNotExists policies
```

### Monitoring Essentials in Landing Zone

| Component | Purpose |
|-----------|---------|
| **Log Analytics Workspace** | Central repository for all logs |
| **Azure Monitor** | Metrics, alerts, dashboards |
| **Microsoft Defender for Cloud** | Security posture, threat protection |
| **Activity Log** | Who did what, when (audit trail) |
| **Diagnostic Settings** | Route resource logs to Log Analytics |
| **Alerts** | Notify when something goes wrong |
| **Workbooks** | Custom dashboards/reports |

---

## 14.9 Key Vault in Landing Zones

### What is Azure Key Vault?

Key Vault is a **secure vault for secrets** — like a safe for your passwords, encryption keys, and certificates.

```
WITHOUT Key Vault (dangerous):
App code: connectionString = "Server=mydb;Password=MyP@ssword123"
→ Password in source code = SECURITY RISK
→ Password in config file = SECURITY RISK
→ Anyone with code access can see the password

WITH Key Vault (secure):
App code: secret = GetSecretFromKeyVault("db-connection-string")
→ Password stored in Key Vault (encrypted at rest)
→ App identity (Managed Identity) has access to vault
→ No passwords in code or config
→ Audit log of every access
```

### Key Vault Stores Three Things

| Type | Examples |
|------|---------|
| **Secrets** | Passwords, connection strings, API keys |
| **Keys** | Encryption keys for data protection |
| **Certificates** | SSL/TLS certificates |

### Managed Identity + Key Vault (Best Practice)

```
TRADITIONAL (username/password for apps):
App ──── uses username/password ────► Database
Problem: where do you store the password?

MANAGED IDENTITY (no password needed):
App (has Azure Managed Identity)
App → "I am App01, let me access Key Vault" → Azure verifies App01's identity
Azure → "Yes, App01 is allowed" → Returns secret
App → uses secret to connect to database
→ NO passwords stored anywhere!
```

```powershell
# Create Key Vault
New-AzKeyVault `
    -VaultName "kv-company-prod-001" `
    -ResourceGroupName "rg-security" `
    -Location "uksouth" `
    -Sku "Standard"

# Store a secret
Set-AzKeyVaultSecret `
    -VaultName "kv-company-prod-001" `
    -Name "db-connection-string" `
    -SecretValue (ConvertTo-SecureString "Server=mydb;Password=MyP@ss" -AsPlainText -Force)

# Enable Managed Identity on VM
$vm = Get-AzVM -ResourceGroupName "rg-production" -Name "vm-app01"
Update-AzVM -ResourceGroupName "rg-production" -VM $vm -IdentityType SystemAssigned

# Give VM access to Key Vault
$vmIdentity = (Get-AzVM -ResourceGroupName "rg-production" -Name "vm-app01").Identity.PrincipalId

Set-AzKeyVaultAccessPolicy `
    -VaultName "kv-company-prod-001" `
    -ObjectId $vmIdentity `
    -PermissionsToSecrets Get, List
```

---

## 14.10 Putting It All Together - Landing Zone Deployment

### Complete Landing Zone Checklist

```
PHASE 1: MANAGEMENT STRUCTURE
□ Create Root Management Group
□ Create Platform MG
│   ├── Connectivity Subscription
│   ├── Identity Subscription
│   └── Management Subscription
□ Create Landing Zones MG
│   ├── Production MG
│   └── Non-Production MG
□ Create Sandbox MG

PHASE 2: GOVERNANCE
□ Apply Azure Policies (regions, tags, security baseline)
□ Configure RBAC for each management group
□ Enable Microsoft Defender for Cloud
□ Set up Azure Monitor and Log Analytics
□ Configure Activity Log alerts

PHASE 3: NETWORKING (Connectivity Subscription)
□ Create Hub VNet
□ Deploy Azure Firewall
□ Deploy VPN Gateway or ExpressRoute Gateway
□ Deploy Azure Bastion
□ Configure Private DNS Zones
□ Set up Azure DNS Resolver

PHASE 4: SECURITY (Identity Subscription)
□ Configure Azure AD
□ Enable PIM for privileged roles
□ Set up MFA for all users
□ Configure Conditional Access policies
□ Deploy Key Vault for platform secrets

PHASE 5: MANAGEMENT SUBSCRIPTION
□ Create central Log Analytics Workspace
□ Configure diagnostic settings (via Policy)
□ Set up Azure Monitor Workbooks
□ Configure backup policies
□ Set up update management

PHASE 6: FIRST WORKLOAD LANDING ZONE
□ Create Spoke VNet in workload subscription
□ Peer to Hub VNet
□ Apply route table (force through firewall)
□ Assign RBAC to app team
□ Deploy resources
```

---

## Summary

| Component | Purpose |
|-----------|---------|
| **Landing Zone** | Pre-built foundation for cloud workloads |
| **Management Groups** | Hierarchical organization of subscriptions |
| **Subscriptions** | Billing/isolation boundary |
| **Azure Policy** | Enforce rules automatically |
| **RBAC** | Control who can do what |
| **Hub-Spoke** | Shared networking with centralized control |
| **Key Vault** | Secure secret management |
| **Log Analytics** | Centralized monitoring |
| **PIM** | Just-in-time privileged access |

---

## Interview Answers

**Q: What is an Azure Landing Zone?**
> An Azure Landing Zone is a pre-configured environment that provides the foundation for cloud workloads. It includes management groups for hierarchy, subscriptions for isolation, RBAC for access control, Azure Policy for governance, Hub-Spoke networking for connectivity, Key Vault for secrets, and centralized monitoring. It ensures all workloads start on a consistent, secure, and well-governed foundation.

**Q: What is the difference between Management Groups and Subscriptions?**
> Management Groups are containers to organize subscriptions hierarchically — you apply policies and RBAC to a management group and they inherit down to all subscriptions. Subscriptions are the billing and isolation boundary where resources actually live. You can have thousands of subscriptions organized under a few management groups.

**Q: How does Azure Policy work?**
> Azure Policy defines rules (written in JSON) and assigns them to scopes (management groups, subscriptions, resource groups). When someone tries to create or modify a resource, Azure checks all applicable policies. Effects can be Deny (block it), Audit (allow but flag), or DeployIfNotExists (automatically fix it). Policies inherit down the management group hierarchy.

**Q: What is Hub-Spoke in a Landing Zone?**
> Hub-Spoke is a network topology where a central Hub VNet contains shared services (Azure Firewall, VPN/ExpressRoute Gateway, Bastion, DNS), and Spoke VNets connect to it via VNet peering. All spoke traffic routes through the Hub Firewall for inspection. It allows workloads in spokes to use shared connectivity without each needing their own gateway.

---

## Additional Resources

- [Azure Landing Zones Documentation](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/)
- [Management Groups](https://docs.microsoft.com/azure/governance/management-groups/)
- [Azure Policy](https://docs.microsoft.com/azure/governance/policy/)
- [Enterprise Scale Landing Zone](https://github.com/Azure/Enterprise-Scale)
- [Key Vault](https://docs.microsoft.com/azure/key-vault/)
- [PIM Documentation](https://docs.microsoft.com/azure/active-directory/privileged-identity-management/)
