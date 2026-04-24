# Module 13 - Azure Landing Zones and Enterprise Architecture (Complete Guide: Beginner to Expert)

## Learning Objectives
By the end of this module, you will be able to:
- Understand Azure Landing Zone concepts and architecture
- Design and implement Hub & Spoke network architectures
- Configure enterprise-scale Azure environments
- Implement network connectivity patterns
- Design multi-subscription architectures
- Apply governance and security at scale
- Deploy Landing Zones using Infrastructure as Code
- Understand Cloud Adoption Framework (CAF)

---

## 13.1 Introduction to Azure Landing Zones

### What is an Azure Landing Zone?
An Azure Landing Zone is a fully configured environment for hosting workloads in Azure. It represents the output of a multi-subscription Azure environment following best practices for scalability, security, governance, networking, and identity.

### Why Landing Zones Matter

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Without Landing Zone                                     │
│                                                                              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐         │
│  │ Team A   │ │ Team B   │ │ Team C   │ │ Team D   │ │ Team E   │         │
│  │ SubA     │ │ SubB     │ │ SubC     │ │ SubD     │ │ SubE     │         │
│  │ ────────│ │ ────────│ │ ────────│ │ ────────│ │ ────────│         │
│  │ • Own    │ │ • Own    │ │ • Own    │ │ • Own    │ │ • Own    │         │
│  │   VNet   │ │   VNet   │ │   VNet   │ │   VNet   │ │   VNet   │         │
│  │ • Own    │ │ • Own    │ │ • Own    │ │ • Own    │ │ • Own    │         │
│  │   Rules  │ │   Rules  │ │   Rules  │ │   Rules  │ │   Rules  │         │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘         │
│                                                                              │
│  Problems:                                                                   │
│  • No consistency  • Security gaps  • Management overhead  • No governance  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                      With Landing Zone                                       │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    Enterprise Scale Foundation                         │  │
│  │  ┌────────────────────────────────────────────────────────────────┐   │  │
│  │  │  Management  │  Connectivity  │  Identity  │  Security         │   │  │
│  │  └────────────────────────────────────────────────────────────────┘   │  │
│  │                                                                        │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │  │
│  │  │ Team A   │ │ Team B   │ │ Team C   │ │ Team D   │ │ Team E   │   │  │
│  │  │ Landing  │ │ Landing  │ │ Landing  │ │ Landing  │ │ Landing  │   │  │
│  │  │ Zone     │ │ Zone     │ │ Zone     │ │ Zone     │ │ Zone     │   │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  Benefits:                                                                   │
│  • Consistent governance  • Centralized security  • Scalable  • Compliant  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Landing Zone Components

| Component | Description | Key Services |
|-----------|-------------|--------------|
| **Identity** | Authentication & authorization | Azure AD, RBAC |
| **Network** | Connectivity & segmentation | VNets, Firewalls, VPN |
| **Governance** | Policies & compliance | Azure Policy, Blueprints |
| **Security** | Protection & monitoring | Defender, Sentinel |
| **Management** | Operations & visibility | Monitor, Log Analytics |

#### 🏢 Real-World Landing Zone Use Cases:

**Enterprise Cloud Migration (Fortune 500):**
```
Company: GlobalManufacturing Inc. (50,000 employees, 30 countries)
Challenge: Migrate 500 applications over 3 years

Landing Zone Architecture:
┌─────────────────────────────────────────────────────────────────────────┐
│                     Root Management Group                                │
│                     (Tenant-wide policies)                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Platform Management Group                                       │   │
│  │  └── Policies: Require tags, allowed locations, require HTTPS   │   │
│  │                                                                   │   │
│  │  ├── Identity Subscription                                       │   │
│  │  │   └── Azure AD Connect, Conditional Access policies           │   │
│  │  │                                                                │   │
│  │  ├── Connectivity Subscription                                   │   │
│  │  │   └── Hub VNet, ExpressRoute (10Gbps to data centers)        │   │
│  │  │   └── Azure Firewall (Premium) - $3,500/month                 │   │
│  │  │   └── 4 VPN tunnels to remote offices                        │   │
│  │  │                                                                │   │
│  │  └── Management Subscription                                     │   │
│  │      └── Log Analytics (2 year retention, 5TB ingest/month)     │   │
│  │      └── Automation accounts for patching                        │   │
│  │      └── Azure Monitor workspace                                 │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Landing Zones Management Group                                  │   │
│  │  └── Policies: Inherit from Platform + application-specific     │   │
│  │                                                                   │   │
│  │  ├── Production (Child MG)                                       │   │
│  │  │   └── Strict policies, change control required               │   │
│  │  │   ├── SAP Subscription (ERP system)                          │   │
│  │  │   ├── CRM Subscription (Salesforce integration)              │   │
│  │  │   ├── E-Commerce Subscription (customer-facing)              │   │
│  │  │   └── Data Platform Subscription (analytics)                 │   │
│  │  │                                                                │   │
│  │  └── Non-Production (Child MG)                                   │   │
│  │      └── Relaxed policies, cost controls                        │   │
│  │      ├── Development Subscription (auto-shutdown 7PM)           │   │
│  │      └── Staging Subscription (pre-prod testing)                │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │  Sandbox Management Group                                        │   │
│  │  └── Innovation, POCs (no connectivity to production)           │   │
│  │  └── Monthly budget: $5,000 (auto-delete if exceeded)           │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘

Migration Results (Year 1):
├── Applications migrated: 150
├── Subscriptions created: 25 (organized, not sprawl)
├── Security incidents: 0 (policies prevented misconfigurations)
├── Cost savings: 30% vs. on-prem (reserved instances, auto-shutdown)
└── Developer productivity: +40% (self-service landing zones)
```

**Regulated Industry (Banking):**
```
Company: SecureBank (PCI-DSS, SOX compliance required)
Challenge: Meet compliance while enabling innovation

Compliance-First Landing Zone:
┌─────────────────────────────────────────────────────────────────────────┐
│  Governance Policies Applied:                                            │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Network Isolation (PCI Requirement):                                    │
│  ├── Policy: Deny public IP on VMs                                      │
│  ├── Policy: Require NSG on subnets                                     │
│  ├── All traffic through Azure Firewall (inspection)                    │
│  └── Private endpoints mandatory for PaaS services                      │
│                                                                          │
│  Data Protection (SOX Requirement):                                      │
│  ├── Policy: Storage encryption with customer-managed keys              │
│  ├── Policy: SQL TDE enabled                                            │
│  ├── Policy: Deny public blob access                                    │
│  └── Key Vault with HSM backing                                         │
│                                                                          │
│  Identity (Both):                                                        │
│  ├── Conditional Access: MFA for all Azure admin                        │
│  ├── PIM: Just-in-time access for production                            │
│  ├── Maximum privileged accounts: 10 (enforced)                         │
│  └── Access reviews: Quarterly automated                                │
│                                                                          │
│  Monitoring (Audit Trail):                                               │
│  ├── All activity logs to immutable storage                             │
│  ├── Retention: 7 years (legal requirement)                             │
│  ├── Azure Sentinel for threat detection                                │
│  └── Weekly compliance reports auto-generated                           │
│                                                                          │
│  Automated Compliance Check:                                             │
│  └── Daily scan: 500 resources checked                                  │
│  └── Non-compliant resources auto-remediated where possible            │
│  └── Report to CISO dashboard                                           │
└─────────────────────────────────────────────────────────────────────────┘

Audit Results:
├── PCI-DSS audit: Passed (first time)
├── SOX audit: No findings
├── Compliance score: 98% (Azure Security Benchmark)
└── Time to demonstrate compliance: 2 days (was 2 weeks)
```

**Startup Scale-Up:**
```
Company: TechStartup (Growing from 10 to 500 employees)
Challenge: Build governance without slowing innovation

Lightweight Landing Zone (Grows with you):
┌─────────────────────────────────────────────────────────────────────────┐
│  Phase 1: MVP (10 people)                                               │
│  ├── Single subscription                                                 │
│  ├── Simple Hub-Spoke (1 hub, 2 spokes: prod/dev)                      │
│  ├── Basic policies (tags, regions)                                     │
│  └── Cost: $200/month in governance overhead                            │
├─────────────────────────────────────────────────────────────────────────┤
│  Phase 2: Growth (100 people)                                           │
│  ├── Added subscriptions for new products                               │
│  ├── Implemented PIM for production access                              │
│  ├── Azure Firewall for centralized inspection                         │
│  └── Cost: $2,000/month in governance                                   │
├─────────────────────────────────────────────────────────────────────────┤
│  Phase 3: Scale (500 people)                                            │
│  ├── Full Enterprise-Scale deployment                                   │
│  ├── SOC 2 compliance required (customer request)                       │
│  ├── Multiple regions (DR requirement)                                  │
│  └── Cost: $15,000/month in governance (0.5% of total spend)           │
└─────────────────────────────────────────────────────────────────────────┘

Key Decision: Start simple, add governance as needed
Result: Never blocked by "enterprise" overhead
```
| **Automation** | Deployment & operations | ARM, Bicep, Terraform |

### Landing Zone Design Principles

| Principle | Description |
|-----------|-------------|
| **Subscription democratization** | Subscriptions as unit of management and scale |
| **Policy-driven governance** | Azure Policy for guardrails |
| **Single control and management plane** | Azure Resource Manager |
| **Application-centric migration** | Organize around applications |
| **Azure-native design** | Leverage native constructs |

---

## 13.2 Cloud Adoption Framework (CAF)

### What is CAF?
The Microsoft Cloud Adoption Framework (CAF) is a comprehensive guide for organizations adopting Azure. It provides end-to-end guidance for cloud adoption.

### CAF Phases

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Cloud Adoption Framework Lifecycle                        │
│                                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │ STRATEGY │→│   PLAN   │→│  READY   │→│  ADOPT   │→│  GOVERN  │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│       │             │             │             │             │             │
│       │             │             │             │             │             │
│  Define         Create        Set up         Migrate/       Manage         │
│  motivations    adoption      landing        Innovate       operations     │
│  & outcomes     plan          zones                         & compliance   │
│                                                                              │
│                         ┌──────────┐                                        │
│                         │  MANAGE  │                                        │
│                         │ (Ongoing)│                                        │
│                         └──────────┘                                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

### CAF Ready Phase - Landing Zones

The Ready phase focuses on preparing Azure environment through Landing Zones:

| Step | Activities |
|------|------------|
| **1. Azure Setup** | Configure Azure AD, subscriptions, regions |
| **2. Landing Zone** | Deploy Enterprise-scale or Start-small |
| **3. Expand** | Add more landing zones for workloads |

---

## 13.3 Enterprise-Scale Architecture

### Enterprise-Scale Overview

Enterprise-Scale is Microsoft's recommended approach for large organizations, providing a prescriptive architecture for organizations at scale.

### Enterprise-Scale Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Enterprise-Scale Architecture                         │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                    Root Management Group                               │  │
│  │                    (Tenant Root Group)                                │  │
│  └───────────────────────────────┬───────────────────────────────────────┘  │
│                                  │                                           │
│         ┌────────────────────────┼────────────────────────┐                 │
│         ▼                        ▼                        ▼                 │
│  ┌──────────────┐       ┌──────────────┐       ┌──────────────┐           │
│  │  Platform    │       │  Landing     │       │  Sandboxes   │           │
│  │              │       │  Zones       │       │              │           │
│  └──────┬───────┘       └──────┬───────┘       └──────────────┘           │
│         │                      │                                            │
│    ┌────┼────┐            ┌────┼────┐                                      │
│    ▼    ▼    ▼            ▼    ▼    ▼                                      │
│ ┌────┐┌────┐┌────┐    ┌────┐┌────┐┌────┐                                  │
│ │Mgmt││Conn││Iden│    │Corp││Onln││ LZ │                                  │
│ │    ││    ││    │    │    ││    ││ N  │                                  │
│ └────┘└────┘└────┘    └────┘└────┘└────┘                                  │
│                                                                              │
│  Legend:                                                                     │
│  • Mgmt = Management     • Corp = Corp-connected LZ                        │
│  • Conn = Connectivity   • Onln = Online LZ                                │
│  • Iden = Identity       • LZ N = Additional Landing Zones                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Management Group Hierarchy

```
Tenant Root Group
│
├── Platform
│   ├── Management (Log Analytics, Automation, Monitor)
│   ├── Connectivity (Hub VNet, Firewall, ExpressRoute, VPN)
│   └── Identity (Domain Controllers, Azure AD Connect)
│
├── Landing Zones
│   ├── Corp (Internal workloads, connected to Hub)
│   │   ├── Subscription: HR System
│   │   ├── Subscription: Finance App
│   │   └── Subscription: ERP System
│   │
│   └── Online (Public-facing, internet access)
│       ├── Subscription: E-commerce
│       ├── Subscription: Customer Portal
│       └── Subscription: Public APIs
│
├── Sandboxes (Development, no production connectivity)
│   ├── Subscription: Developer 1
│   └── Subscription: Developer 2
│
└── Decommissioned (Archived subscriptions)
```

### Platform Landing Zone Components

#### 1. Management Subscription

```
┌─────────────────────────────────────────────────────────────────┐
│                  Management Subscription                         │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Management Resource Group                   │   │
│  │                                                          │   │
│  │  ┌───────────────────┐  ┌───────────────────┐           │   │
│  │  │  Log Analytics    │  │  Automation       │           │   │
│  │  │  Workspace        │  │  Account          │           │   │
│  │  │  • All logs       │  │  • Runbooks       │           │   │
│  │  │  • Diagnostics    │  │  • DSC            │           │   │
│  │  │  • Security logs  │  │  • Updates        │           │   │
│  │  └───────────────────┘  └───────────────────┘           │   │
│  │                                                          │   │
│  │  ┌───────────────────┐  ┌───────────────────┐           │   │
│  │  │  Azure Monitor    │  │  Microsoft        │           │   │
│  │  │  • Alerts         │  │  Sentinel         │           │   │
│  │  │  • Dashboards     │  │  • SIEM           │           │   │
│  │  │  • Workbooks      │  │  • SOAR           │           │   │
│  │  └───────────────────┘  └───────────────────┘           │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

#### 2. Connectivity Subscription

```
┌─────────────────────────────────────────────────────────────────┐
│                  Connectivity Subscription                       │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     Hub VNet                             │   │
│  │                 (10.0.0.0/16)                            │   │
│  │                                                          │   │
│  │  ┌───────────────────┐  ┌───────────────────┐           │   │
│  │  │  Azure Firewall   │  │  Gateway Subnet   │           │   │
│  │  │  Subnet           │  │  (VPN/ExpressRoute│           │   │
│  │  │  10.0.0.0/26      │  │  10.0.1.0/27)     │           │   │
│  │  └───────────────────┘  └───────────────────┘           │   │
│  │                                                          │   │
│  │  ┌───────────────────┐  ┌───────────────────┐           │   │
│  │  │  Bastion Subnet   │  │  DNS Servers      │           │   │
│  │  │  10.0.2.0/27      │  │  Subnet           │           │   │
│  │  │                   │  │  10.0.3.0/24      │           │   │
│  │  └───────────────────┘  └───────────────────┘           │   │
│  │                                                          │   │
│  │  ┌────────────────────────────────────────────────────┐ │   │
│  │  │                   Azure Firewall                    │ │   │
│  │  │  • Network Rules  • Application Rules  • DNAT      │ │   │
│  │  └────────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                DDoS Protection Plan                      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              ExpressRoute / VPN Gateway                   │   │
│  │                  (On-premises connectivity)               │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 13.4 Hub and Spoke Network Architecture

### What is Hub and Spoke?
Hub and Spoke is a network topology where a central hub VNet connects to multiple spoke VNets. The hub contains shared services and provides connectivity to on-premises.

### Hub and Spoke Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Hub and Spoke Architecture                              │
│                                                                              │
│                        ┌──────────────────┐                                 │
│                        │   On-Premises    │                                 │
│                        │   Data Center    │                                 │
│                        └────────┬─────────┘                                 │
│                                 │                                            │
│                       ExpressRoute/VPN                                       │
│                                 │                                            │
│  ┌──────────────────────────────┴──────────────────────────────────┐       │
│  │                          HUB VNET                                │       │
│  │                        (10.0.0.0/16)                             │       │
│  │                                                                  │       │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │       │
│  │  │ Firewall │  │ Gateway  │  │  DNS     │  │ Bastion  │        │       │
│  │  │ Subnet   │  │ Subnet   │  │  Servers │  │ Subnet   │        │       │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │       │
│  │                                                                  │       │
│  └──────────────────────────┬───────────────────────────────────────┘       │
│                              │                                               │
│              ┌───────────────┼───────────────┬───────────────┐             │
│              │               │               │               │             │
│      VNet Peering     VNet Peering    VNet Peering    VNet Peering        │
│              │               │               │               │             │
│              ▼               ▼               ▼               ▼             │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐ │
│  │  SPOKE 1     │   │  SPOKE 2     │   │  SPOKE 3     │   │  SPOKE 4     │ │
│  │  HR App      │   │  Finance App │   │  Web Tier    │   │  Data Tier   │ │
│  │ 10.1.0.0/16  │   │ 10.2.0.0/16  │   │ 10.3.0.0/16  │   │ 10.4.0.0/16  │ │
│  │              │   │              │   │              │   │              │ │
│  │ [VM] [VM]    │   │ [VM] [DB]    │   │ [App] [App]  │   │ [SQL] [SQL]  │ │
│  └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘ │
│                                                                              │
│                             Azure Firewall Routes All Traffic               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Hub VNet Configuration

```powershell
# Create Hub VNet
$hubVnet = New-AzVirtualNetwork `
    -Name "Hub-VNet" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -AddressPrefix "10.0.0.0/16"

# Add subnets
$azureFirewallSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "AzureFirewallSubnet" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.0.0/26"

$gatewaySubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "GatewaySubnet" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.1.0/27"

$bastionSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "AzureBastionSubnet" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.2.0/27"

$dnsSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "DnsServers" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.3.0/24"

$sharedServicesSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "SharedServices" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.4.0/24"

$hubVnet | Set-AzVirtualNetwork
```

### Spoke VNet Configuration

```powershell
# Create Spoke VNet
$spokeVnet = New-AzVirtualNetwork `
    -Name "Spoke-HR-VNet" `
    -ResourceGroupName "HR-RG" `
    -Location "East US" `
    -AddressPrefix "10.1.0.0/16"

# Add subnets
$webSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "Web-Subnet" `
    -VirtualNetwork $spokeVnet `
    -AddressPrefix "10.1.1.0/24"

$appSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "App-Subnet" `
    -VirtualNetwork $spokeVnet `
    -AddressPrefix "10.1.2.0/24"

$dataSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "Data-Subnet" `
    -VirtualNetwork $spokeVnet `
    -AddressPrefix "10.1.3.0/24"

$spokeVnet | Set-AzVirtualNetwork
```

### VNet Peering Configuration

```powershell
# Hub to Spoke peering
Add-AzVirtualNetworkPeering `
    -Name "Hub-to-Spoke-HR" `
    -VirtualNetwork $hubVnet `
    -RemoteVirtualNetworkId $spokeVnet.Id `
    -AllowGatewayTransit `
    -AllowForwardedTraffic

# Spoke to Hub peering
Add-AzVirtualNetworkPeering `
    -Name "Spoke-HR-to-Hub" `
    -VirtualNetwork $spokeVnet `
    -RemoteVirtualNetworkId $hubVnet.Id `
    -UseRemoteGateways `
    -AllowForwardedTraffic
```

### User Defined Routes (UDR) for Traffic Through Firewall

```powershell
# Create route table
$routeTable = New-AzRouteTable `
    -Name "Spoke-RouteTable" `
    -ResourceGroupName "HR-RG" `
    -Location "East US"

# Add route to send all traffic through firewall
$route = Add-AzRouteTableRoute `
    -Name "ToFirewall" `
    -RouteTable $routeTable `
    -AddressPrefix "0.0.0.0/0" `
    -NextHopType "VirtualAppliance" `
    -NextHopIpAddress "10.0.0.4"  # Azure Firewall private IP

# Associate with spoke subnets
$spokeVnet = Get-AzVirtualNetwork -Name "Spoke-HR-VNet" -ResourceGroupName "HR-RG"
Set-AzVirtualNetworkSubnetConfig `
    -Name "Web-Subnet" `
    -VirtualNetwork $spokeVnet `
    -AddressPrefix "10.1.1.0/24" `
    -RouteTable $routeTable

$spokeVnet | Set-AzVirtualNetwork
```

---

## 13.5 Azure Firewall Configuration

### Azure Firewall in Hub

```powershell
# Create public IP for firewall
$firewallPip = New-AzPublicIpAddress `
    -Name "AzureFirewall-PIP" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -Sku "Standard" `
    -AllocationMethod "Static"

# Create Azure Firewall
$firewall = New-AzFirewall `
    -Name "AzureFirewall" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -VirtualNetworkName "Hub-VNet" `
    -PublicIpName "AzureFirewall-PIP" `
    -Sku "AZFW_VNet" `
    -SkuTier "Standard"
```

### Firewall Policy

```powershell
# Create firewall policy
$policy = New-AzFirewallPolicy `
    -Name "EnterpriseFirewallPolicy" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -ThreatIntelMode "Alert"

# Create rule collection group
$ruleCollectionGroup = New-AzFirewallPolicyRuleCollectionGroup `
    -Name "DefaultApplicationRuleCollectionGroup" `
    -Priority 100 `
    -FirewallPolicyName "EnterpriseFirewallPolicy" `
    -ResourceGroupName "Connectivity-RG"
```

### Network Rules

```powershell
# Allow Azure services
$networkRule1 = New-AzFirewallPolicyNetworkRule `
    -Name "AllowAzureServices" `
    -Protocol "Any" `
    -SourceAddress "10.0.0.0/8" `
    -DestinationAddress "AzureCloud" `
    -DestinationPort "*"

# Allow DNS
$networkRule2 = New-AzFirewallPolicyNetworkRule `
    -Name "AllowDns" `
    -Protocol "UDP" `
    -SourceAddress "*" `
    -DestinationAddress "168.63.129.16" `
    -DestinationPort "53"

$networkRuleCollection = New-AzFirewallPolicyNetworkRuleCollection `
    -Name "NetworkRules" `
    -Priority 100 `
    -ActionType "Allow" `
    -Rule $networkRule1, $networkRule2
```

### Application Rules

```powershell
# Allow Windows Update
$appRule1 = New-AzFirewallPolicyApplicationRule `
    -Name "AllowWindowsUpdate" `
    -SourceAddress "10.0.0.0/8" `
    -Protocol "https:443" `
    -TargetFqdn "*.update.microsoft.com", "*.windowsupdate.com"

# Allow Azure DevOps
$appRule2 = New-AzFirewallPolicyApplicationRule `
    -Name "AllowAzureDevOps" `
    -SourceAddress "10.0.0.0/8" `
    -Protocol "https:443" `
    -TargetFqdn "*.dev.azure.com", "*.visualstudio.com"

$appRuleCollection = New-AzFirewallPolicyApplicationRuleCollection `
    -Name "ApplicationRules" `
    -Priority 200 `
    -ActionType "Allow" `
    -Rule $appRule1, $appRule2
```

---

## 13.6 ExpressRoute and VPN Connectivity

### ExpressRoute Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      ExpressRoute Connectivity                               │
│                                                                              │
│  ┌──────────────────┐                       ┌──────────────────────────┐   │
│  │   On-Premises    │                       │       Azure Region        │   │
│  │   Data Center    │                       │                           │   │
│  │                  │                       │  ┌──────────────────────┐ │   │
│  │  ┌────────────┐  │                       │  │     Hub VNet         │ │   │
│  │  │ Edge Router│  │                       │  │                      │ │   │
│  │  └─────┬──────┘  │                       │  │  ┌────────────────┐  │ │   │
│  │        │         │                       │  │  │ ExpressRoute   │  │ │   │
│  └────────┼─────────┘                       │  │  │ Gateway        │  │ │   │
│           │                                  │  │  └────────┬───────┘  │ │   │
│           │      ┌─────────────────────┐    │  │           │          │ │   │
│           │      │  ExpressRoute       │    │  │      Peered to      │ │   │
│           └──────┤  Circuit            ├────┼──┤      Spokes         │ │   │
│                  │  (Provider Edge)    │    │  │                      │ │   │
│                  └─────────────────────┘    │  └──────────────────────┘ │   │
│                                              │                           │   │
│                  Microsoft Edge              │  ┌──────────────────────┐ │   │
│                  (Azure Peering Location)    │  │    Spoke VNets       │ │   │
│                                              │  │    (Workloads)       │ │   │
│                                              │  └──────────────────────┘ │   │
│                                              └──────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Creating ExpressRoute Gateway

```powershell
# Create ExpressRoute Gateway
$gwPip = New-AzPublicIpAddress `
    -Name "ExpressRoute-GW-PIP" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -AllocationMethod "Static" `
    -Sku "Standard"

$hubVnet = Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Connectivity-RG"
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $hubVnet

$gwIpConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "GatewayIpConfig" `
    -SubnetId $gwSubnet.Id `
    -PublicIpAddressId $gwPip.Id

$expressRouteGw = New-AzVirtualNetworkGateway `
    -Name "ExpressRoute-Gateway" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -IpConfigurations $gwIpConfig `
    -GatewayType "ExpressRoute" `
    -GatewaySku "ErGw1AZ"
```

### VPN Gateway for Backup

```powershell
# Create VPN Gateway (as backup to ExpressRoute)
$vpnGw = New-AzVirtualNetworkGateway `
    -Name "VPN-Gateway" `
    -ResourceGroupName "Connectivity-RG" `
    -Location "East US" `
    -IpConfigurations $gwIpConfig `
    -GatewayType "Vpn" `
    -VpnType "RouteBased" `
    -GatewaySku "VpnGw1AZ" `
    -EnableBgp $true `
    -Asn 65515
```

---

## 13.7 Azure Policy for Governance

### Enterprise Policies

```json
// Require resource group tags
{
    "mode": "All",
    "policyRule": {
        "if": {
            "allOf": [
                {
                    "field": "type",
                    "equals": "Microsoft.Resources/subscriptions/resourceGroups"
                },
                {
                    "anyOf": [
                        {
                            "field": "[concat('tags[', 'Environment', ']')]",
                            "exists": "false"
                        },
                        {
                            "field": "[concat('tags[', 'CostCenter', ']')]",
                            "exists": "false"
                        },
                        {
                            "field": "[concat('tags[', 'Owner', ']')]",
                            "exists": "false"
                        }
                    ]
                }
            ]
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

```json
// Allowed locations
{
    "mode": "All",
    "parameters": {
        "allowedLocations": {
            "type": "Array",
            "metadata": {
                "displayName": "Allowed locations",
                "description": "The list of allowed locations for resources."
            }
        }
    },
    "policyRule": {
        "if": {
            "allOf": [
                {
                    "field": "location",
                    "notIn": "[parameters('allowedLocations')]"
                },
                {
                    "field": "location",
                    "notEquals": "global"
                }
            ]
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

### Policy Initiative (Policy Set)

```powershell
# Create policy initiative
$policySetDef = @"
{
    "properties": {
        "displayName": "Enterprise Governance Initiative",
        "description": "Collection of policies for enterprise governance",
        "policyDefinitions": [
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/allowed-locations",
                "parameters": {
                    "listOfAllowedLocations": {
                        "value": ["eastus", "eastus2", "westus2"]
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/require-tag-rg"
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/allowed-vm-skus",
                "parameters": {
                    "listOfAllowedSKUs": {
                        "value": ["Standard_D2s_v3", "Standard_D4s_v3", "Standard_B2s"]
                    }
                }
            }
        ]
    }
}
"@

New-AzPolicySetDefinition `
    -Name "EnterpriseGovernance" `
    -PolicyDefinition $policySetDef `
    -ManagementGroupName "root-mg"
```

---

## 13.8 Landing Zone Deployment with Infrastructure as Code

### Bicep Template for Landing Zone

```bicep
// landing-zone.bicep
targetScope = 'subscription'

@description('The location for the landing zone resources')
param location string = 'eastus'

@description('The name of the workload')
param workloadName string

@description('The environment (dev, test, prod)')
@allowed([
  'dev'
  'test'
  'prod'
])
param environment string

@description('The hub VNet resource ID for peering')
param hubVNetId string

@description('The address prefix for the spoke VNet')
param spokeAddressPrefix string

@description('Tags for all resources')
param tags object = {
  Environment: environment
  Workload: workloadName
  ManagedBy: 'Bicep'
}

// Variables
var resourceGroupName = 'rg-${workloadName}-${environment}'
var vnetName = 'vnet-${workloadName}-${environment}'

// Resource Group
resource rg 'Microsoft.Resources/resourceGroups@2021-04-01' = {
  name: resourceGroupName
  location: location
  tags: tags
}

// Spoke VNet Module
module spokeVnet 'modules/vnet.bicep' = {
  name: 'deploy-spoke-vnet'
  scope: rg
  params: {
    vnetName: vnetName
    location: location
    addressPrefix: spokeAddressPrefix
    subnets: [
      {
        name: 'snet-web'
        addressPrefix: cidrSubnet(spokeAddressPrefix, 24, 0)
      }
      {
        name: 'snet-app'
        addressPrefix: cidrSubnet(spokeAddressPrefix, 24, 1)
      }
      {
        name: 'snet-data'
        addressPrefix: cidrSubnet(spokeAddressPrefix, 24, 2)
      }
    ]
    tags: tags
  }
}

// VNet Peering to Hub
module vnetPeering 'modules/peering.bicep' = {
  name: 'deploy-vnet-peering'
  scope: rg
  params: {
    spokeVnetName: spokeVnet.outputs.vnetName
    hubVNetId: hubVNetId
  }
}

// Network Security Groups
module nsgs 'modules/nsg.bicep' = {
  name: 'deploy-nsgs'
  scope: rg
  params: {
    location: location
    workloadName: workloadName
    environment: environment
    tags: tags
  }
}

// Route Table (to force traffic through hub firewall)
module routeTable 'modules/route-table.bicep' = {
  name: 'deploy-route-table'
  scope: rg
  params: {
    location: location
    routeTableName: 'rt-${workloadName}-${environment}'
    firewallPrivateIp: '10.0.0.4'  // Hub firewall IP
    tags: tags
  }
}

// Key Vault
module keyVault 'modules/keyvault.bicep' = {
  name: 'deploy-keyvault'
  scope: rg
  params: {
    keyVaultName: 'kv-${workloadName}-${environment}'
    location: location
    tags: tags
  }
}

// Storage Account
module storage 'modules/storage.bicep' = {
  name: 'deploy-storage'
  scope: rg
  params: {
    storageAccountName: 'st${workloadName}${environment}'
    location: location
    tags: tags
  }
}

// Outputs
output resourceGroupName string = rg.name
output vnetId string = spokeVnet.outputs.vnetId
output keyVaultName string = keyVault.outputs.keyVaultName
```

### VNet Module

```bicep
// modules/vnet.bicep
param vnetName string
param location string
param addressPrefix string
param subnets array
param tags object

resource vnet 'Microsoft.Network/virtualNetworks@2021-05-01' = {
  name: vnetName
  location: location
  tags: tags
  properties: {
    addressSpace: {
      addressPrefixes: [
        addressPrefix
      ]
    }
    subnets: [for subnet in subnets: {
      name: subnet.name
      properties: {
        addressPrefix: subnet.addressPrefix
        privateEndpointNetworkPolicies: 'Disabled'
      }
    }]
  }
}

output vnetId string = vnet.id
output vnetName string = vnet.name
output subnets array = vnet.properties.subnets
```

### VNet Peering Module

```bicep
// modules/peering.bicep
param spokeVnetName string
param hubVNetId string

resource spokeVnet 'Microsoft.Network/virtualNetworks@2021-05-01' existing = {
  name: spokeVnetName
}

resource spokeToHubPeering 'Microsoft.Network/virtualNetworks/virtualNetworkPeerings@2021-05-01' = {
  parent: spokeVnet
  name: 'peer-to-hub'
  properties: {
    remoteVirtualNetwork: {
      id: hubVNetId
    }
    allowVirtualNetworkAccess: true
    allowForwardedTraffic: true
    useRemoteGateways: true
  }
}
```

### Terraform Configuration

```hcl
# landing-zone/main.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "landing-zone.tfstate"
  }
}

provider "azurerm" {
  features {}
}

# Variables
variable "workload_name" {
  type        = string
  description = "Name of the workload"
}

variable "environment" {
  type        = string
  description = "Environment (dev, test, prod)"
}

variable "location" {
  type        = string
  default     = "eastus"
}

variable "spoke_address_prefix" {
  type        = string
  description = "Address prefix for spoke VNet"
}

variable "hub_vnet_id" {
  type        = string
  description = "Resource ID of hub VNet"
}

variable "firewall_private_ip" {
  type        = string
  description = "Private IP of Azure Firewall"
  default     = "10.0.0.4"
}

# Local variables
locals {
  resource_group_name = "rg-${var.workload_name}-${var.environment}"
  tags = {
    Environment = var.environment
    Workload    = var.workload_name
    ManagedBy   = "Terraform"
  }
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = local.resource_group_name
  location = var.location
  tags     = local.tags
}

# Spoke VNet
resource "azurerm_virtual_network" "spoke" {
  name                = "vnet-${var.workload_name}-${var.environment}"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = [var.spoke_address_prefix]
  tags                = local.tags
}

# Subnets
resource "azurerm_subnet" "web" {
  name                 = "snet-web"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.spoke.name
  address_prefixes     = [cidrsubnet(var.spoke_address_prefix, 8, 0)]
}

resource "azurerm_subnet" "app" {
  name                 = "snet-app"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.spoke.name
  address_prefixes     = [cidrsubnet(var.spoke_address_prefix, 8, 1)]
}

resource "azurerm_subnet" "data" {
  name                 = "snet-data"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.spoke.name
  address_prefixes     = [cidrsubnet(var.spoke_address_prefix, 8, 2)]
}

# VNet Peering - Spoke to Hub
resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                         = "peer-to-hub"
  resource_group_name          = azurerm_resource_group.main.name
  virtual_network_name         = azurerm_virtual_network.spoke.name
  remote_virtual_network_id    = var.hub_vnet_id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  use_remote_gateways          = true
}

# Route Table
resource "azurerm_route_table" "main" {
  name                          = "rt-${var.workload_name}-${var.environment}"
  location                      = azurerm_resource_group.main.location
  resource_group_name           = azurerm_resource_group.main.name
  disable_bgp_route_propagation = false
  tags                          = local.tags
}

resource "azurerm_route" "to_firewall" {
  name                   = "to-firewall"
  resource_group_name    = azurerm_resource_group.main.name
  route_table_name       = azurerm_route_table.main.name
  address_prefix         = "0.0.0.0/0"
  next_hop_type          = "VirtualAppliance"
  next_hop_in_ip_address = var.firewall_private_ip
}

# Associate route table with subnets
resource "azurerm_subnet_route_table_association" "web" {
  subnet_id      = azurerm_subnet.web.id
  route_table_id = azurerm_route_table.main.id
}

resource "azurerm_subnet_route_table_association" "app" {
  subnet_id      = azurerm_subnet.app.id
  route_table_id = azurerm_route_table.main.id
}

resource "azurerm_subnet_route_table_association" "data" {
  subnet_id      = azurerm_subnet.data.id
  route_table_id = azurerm_route_table.main.id
}

# Key Vault
resource "azurerm_key_vault" "main" {
  name                       = "kv-${var.workload_name}-${var.environment}"
  location                   = azurerm_resource_group.main.location
  resource_group_name        = azurerm_resource_group.main.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  soft_delete_retention_days = 90
  purge_protection_enabled   = true
  sku_name                   = "standard"
  tags                       = local.tags

  enable_rbac_authorization = true
}

# Storage Account
resource "azurerm_storage_account" "main" {
  name                     = "st${var.workload_name}${var.environment}"
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  min_tls_version          = "TLS1_2"
  tags                     = local.tags
}

# Data
data "azurerm_client_config" "current" {}

# Outputs
output "resource_group_name" {
  value = azurerm_resource_group.main.name
}

output "vnet_id" {
  value = azurerm_virtual_network.spoke.id
}

output "key_vault_id" {
  value = azurerm_key_vault.main.id
}
```

---

## 13.9 CI/CD for Landing Zone Deployment

### Azure DevOps Pipeline for Landing Zone

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - landing-zones/**

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: landing-zone-variables
  - name: terraformVersion
    value: '1.5.0'

stages:
  - stage: Validate
    jobs:
      - job: Validate
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: $(terraformVersion)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
              backendServiceArm: 'AzureServiceConnection'
              backendAzureRmResourceGroupName: 'tfstate-rg'
              backendAzureRmStorageAccountName: 'tfstateaccount'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'landing-zone.tfstate'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Validate'
            inputs:
              provider: 'azurerm'
              command: 'validate'
              workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
              environmentServiceNameAzureRM: 'AzureServiceConnection'
              commandOptions: '-var-file=environments/$(environment).tfvars -out=tfplan'

          - publish: $(System.DefaultWorkingDirectory)/landing-zones/tfplan
            artifact: tfplan

  - stage: DeployDev
    dependsOn: Validate
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployDev
        environment: 'landing-zone-dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: tfplan

                - task: TerraformInstaller@0
                  inputs:
                    terraformVersion: $(terraformVersion)

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
                    backendServiceArm: 'AzureServiceConnection'
                    backendAzureRmResourceGroupName: 'tfstate-rg'
                    backendAzureRmStorageAccountName: 'tfstateaccount'
                    backendAzureRmContainerName: 'tfstate'
                    backendAzureRmKey: 'landing-zone-dev.tfstate'

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
                    environmentServiceNameAzureRM: 'AzureServiceConnection'
                    commandOptions: '$(Pipeline.Workspace)/tfplan/tfplan'

  - stage: DeployProd
    dependsOn: DeployDev
    jobs:
      - deployment: DeployProd
        environment: 'landing-zone-prod'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: TerraformInstaller@0
                  inputs:
                    terraformVersion: $(terraformVersion)

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
                    backendServiceArm: 'AzureServiceConnection'
                    backendAzureRmResourceGroupName: 'tfstate-rg'
                    backendAzureRmStorageAccountName: 'tfstateaccount'
                    backendAzureRmContainerName: 'tfstate'
                    backendAzureRmKey: 'landing-zone-prod.tfstate'

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Plan'
                  inputs:
                    provider: 'azurerm'
                    command: 'plan'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
                    environmentServiceNameAzureRM: 'AzureServiceConnection'
                    commandOptions: '-var-file=environments/prod.tfvars -out=tfplan-prod'

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/landing-zones'
                    environmentServiceNameAzureRM: 'AzureServiceConnection'
                    commandOptions: 'tfplan-prod'
```

---

## 13.10 Summary and Best Practices

### Landing Zone Best Practices

| Area | Best Practice |
|------|---------------|
| **Architecture** | Use Hub & Spoke with centralized connectivity |
| **Governance** | Apply Azure Policy at management group level |
| **Security** | Force all traffic through Azure Firewall |
| **Identity** | Use Azure AD with centralized access management |
| **Networking** | Plan address spaces carefully, use NAT for spokes |
| **Monitoring** | Centralize logs in Log Analytics workspace |
| **Automation** | Deploy using Infrastructure as Code |

### Key Takeaways for DevOps Integration Engineers

1. **Understand the enterprise architecture** before implementing
2. **Use management groups** for policy inheritance
3. **Implement Hub & Spoke** for network segregation
4. **Route all traffic** through centralized firewall
5. **Apply policies** for consistent governance
6. **Automate deployments** using Bicep or Terraform
7. **Document the architecture** for team knowledge sharing

---

## Practice Questions

1. What is the purpose of a Hub VNet in an enterprise architecture?
2. How do you force spoke traffic through Azure Firewall?
3. What are the key components of an Enterprise-Scale Landing Zone?
4. How do you implement VNet peering with gateway transit?
5. What Azure Policy initiatives should be applied at the management group level?
6. How do you design address space planning for multiple subscriptions?
7. Explain the role of ExpressRoute in enterprise connectivity.

---

## Additional Resources

- [Cloud Adoption Framework](https://docs.microsoft.com/azure/cloud-adoption-framework/)
- [Enterprise-Scale Architecture](https://docs.microsoft.com/azure/cloud-adoption-framework/ready/enterprise-scale/)
- [Hub-Spoke Network Topology](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
- [Azure Landing Zone Accelerator](https://github.com/Azure/Enterprise-Scale)
