# Module 01 - Introduction to Microsoft Azure
## PowerPoint Presentation for Coaching Institute
---

# SLIDE 1: Title Slide

## Module 01
# Introduction to Microsoft Azure
### AZ-104: Microsoft Azure Administrator

**Topics Covered:**
- Cloud Computing Fundamentals
- Microsoft Azure Overview
- Azure Services & Architecture
- Azure CLI & PowerShell
- Azure Resource Manager (ARM)

---

# SLIDE 2: Learning Objectives

## What You Will Learn Today

✅ Understand cloud computing fundamentals and service models (IaaS, PaaS, SaaS)

✅ Explain what Microsoft Azure is and its core services

✅ Create and manage an Azure account

✅ Use Azure CLI and Azure PowerShell

✅ Understand Azure Resource Manager and architecture

---

# SLIDE 3: What is Cloud Computing?

## Definition

> **Cloud computing** is the delivery of computing services—including servers, storage, databases, networking, software—over the Internet to offer faster innovation, flexible resources, and economies of scale.

### Simple Analogy 🔌
Just like electricity - you don't build your own power plant, you use power from a provider and pay for what you use!

---

# SLIDE 4: Key Characteristics of Cloud

| Characteristic | What It Means |
|----------------|---------------|
| **On-demand self-service** | Get resources automatically, no waiting |
| **Broad network access** | Access from anywhere via internet |
| **Resource pooling** | Shared infrastructure, multi-tenant |
| **Rapid elasticity** | Scale up/down instantly |
| **Measured service** | Pay only for what you use |

---

# SLIDE 5: Cloud Service Models Overview

```
+--------------------------------------------------+
|                                                  |
|    On-Premises    IaaS       PaaS       SaaS     |
|                                                  |
|  +------------+ +--------+ +--------+ +--------+ |
|  |Applications| |   You  | |   You  | |Provider| |
|  |    Data    | |   You  | |   You  | |Provider| |
|  |   Runtime  | |   You  | |Provider| |Provider| |
|  |  Middleware| |   You  | |Provider| |Provider| |
|  |     O/S    | |   You  | |Provider| |Provider| |
|  |Virtualizatn| |Provider| |Provider| |Provider| |
|  |   Servers  | |Provider| |Provider| |Provider| |
|  |   Storage  | |Provider| |Provider| |Provider| |
|  | Networking | |Provider| |Provider| |Provider| |
|  +------------+ +--------+ +--------+ +--------+ |
|                                                  |
+--------------------------------------------------+
```

---

# SLIDE 6: Infrastructure as a Service (IaaS)

## IaaS - You Control the Most

**What is IaaS?**
- Virtualized computing resources over the internet
- You manage: Applications, Data, Runtime, Middleware, OS
- Provider manages: Infrastructure (servers, storage, networking)

**Azure Examples:**
- 🖥️ Azure Virtual Machines
- 💾 Azure Storage
- 🌐 Azure Virtual Networks

**Best For:** When you need complete control over infrastructure

---

# SLIDE 7: Platform as a Service (PaaS)

## PaaS - Focus on Your Code

**What is PaaS?**
- Platform for developing, running, and managing applications
- You manage: Applications and Data ONLY
- Provider manages: Everything else

**Azure Examples:**
- 🌍 Azure App Service
- 🗃️ Azure SQL Database
- ⚡ Azure Functions

**Best For:** When you want to focus only on application development

---

# SLIDE 8: Software as a Service (SaaS)

## SaaS - Ready to Use

**What is SaaS?**
- Complete software application hosted by provider
- You manage: Only your data and access
- Provider manages: Everything including the application

**Examples:**
- 📧 Microsoft 365 (Outlook, Teams, Word)
- 📊 Dynamics 365
- ☁️ Salesforce

**Best For:** Ready-to-use software without any setup

---

# SLIDE 9: Cloud Deployment Models

| Model | Description | Use Case |
|-------|-------------|----------|
| **☁️ Public Cloud** | Resources owned by cloud provider | Maximum scalability, cost-effectiveness |
| **🔒 Private Cloud** | Dedicated to single organization | Security, compliance requirements |
| **🔄 Hybrid Cloud** | Combination of both | Flexibility, gradual migration |

### Most organizations use **Hybrid Cloud** approach!

---

# SLIDE 10: Benefits of Cloud Computing

## Why Move to the Cloud?

| Benefit | Description |
|---------|-------------|
| 💰 **Cost Efficiency** | No upfront costs, pay-per-use |
| 📈 **Scalability** | Scale up/down based on demand |
| 🔒 **Reliability** | Built-in backup & disaster recovery |
| 🛡️ **Security** | Enterprise-grade security features |
| 🌍 **Global Reach** | Deploy worldwide in minutes |
| ⚡ **Performance** | Latest hardware & fast networks |

---

# SLIDE 11: What is Microsoft Azure?

## Microsoft Azure

> A comprehensive cloud computing platform providing 200+ services for compute, analytics, storage, and networking.

### Quick History:
- **2008** - Project "Red Dog" started
- **2010** - Windows Azure launched
- **2014** - Renamed to Microsoft Azure
- **Today** - Top 3 cloud provider globally (alongside AWS & Google Cloud)

---

# SLIDE 12: Azure Global Infrastructure

## Azure's Massive Scale

🌍 **60+ Regions** worldwide (more than any other cloud provider!)

### Key Concepts:

| Term | Description |
|------|-------------|
| **Region** | Geographical area with 1+ datacenters |
| **Availability Zone** | Physically separate location within a region |
| **Region Pair** | Two regions paired for disaster recovery |

**Examples:** East US, West Europe, Southeast Asia, Central India

---

# SLIDE 13: Availability Zones Explained

```
┌─────────────── Azure Region ───────────────┐
│                                            │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │ Zone 1  │  │ Zone 2  │  │ Zone 3  │    │
│  │         │  │         │  │         │    │
│  │Datactr A│  │Datactr B│  │Datactr C│    │
│  └─────────┘  └─────────┘  └─────────┘    │
│       ↕           ↕           ↕           │
│  ────── Low-latency Network ──────        │
└────────────────────────────────────────────┘
```

- ✅ Independent power, cooling, networking
- ✅ Minimum 3 zones in enabled regions
- ✅ **99.99% VM uptime SLA**

---

# SLIDE 14: Azure vs Competition

| Feature | Azure | AWS | Google Cloud |
|---------|-------|-----|--------------|
| **Global Regions** | 60+ | 31 | 35 |
| **Hybrid Support** | Azure Arc, Stack | Outposts | Anthos |
| **Enterprise Integration** | ⭐ Excellent | Good | Good |
| **AI/ML Services** | Azure AI | SageMaker | Vertex AI |

### Azure Advantage: Best integration with Microsoft ecosystem (Windows, Office 365, Active Directory)

---

# SLIDE 15: Azure Compute Services

## Run Your Applications

| Service | Description | When to Use |
|---------|-------------|-------------|
| 🖥️ **Virtual Machines** | IaaS virtual servers | Traditional apps, full control |
| 🌐 **App Service** | PaaS for web apps | Websites, REST APIs |
| ⚡ **Functions** | Serverless compute | Event-driven processing |
| 🐳 **AKS** | Managed Kubernetes | Container orchestration |
| 📦 **Container Instances** | Serverless containers | Quick container deployment |

---

# SLIDE 16: Azure Storage Services

## Store Your Data

| Service | Description | When to Use |
|---------|-------------|-------------|
| 📁 **Blob Storage** | Object storage | Media files, backups |
| 📂 **Azure Files** | Managed file shares | File sharing, lift-and-shift |
| 📨 **Queue Storage** | Message queuing | Decoupling applications |
| 📊 **Table Storage** | NoSQL key-value | Semi-structured data |
| 💿 **Disk Storage** | Managed disks | VM data storage |

---

# SLIDE 17: Azure Networking Services

## Connect Your Resources

| Service | Description | When to Use |
|---------|-------------|-------------|
| 🌐 **Virtual Network** | Private network | Isolating resources |
| ⚖️ **Load Balancer** | Layer 4 balancing | Distributing traffic |
| 🚪 **Application Gateway** | Layer 7 balancing | Web app traffic |
| 🔗 **VPN Gateway** | VPN connectivity | Hybrid connectivity |
| 🚄 **ExpressRoute** | Private connection | High-speed dedicated link |

---

# SLIDE 18: Azure Database Services

## Manage Your Databases

| Service | Description | When to Use |
|---------|-------------|-------------|
| 🗃️ **Azure SQL Database** | Managed SQL | Relational data |
| 🌍 **Cosmos DB** | Global NoSQL | Multi-region apps |
| 🐬 **Database for MySQL** | Managed MySQL | MySQL workloads |
| 🐘 **Database for PostgreSQL** | Managed PostgreSQL | PostgreSQL workloads |

---

# SLIDE 19: Creating Azure Account

## Get Started - It's FREE!

### Azure Free Account Benefits:

| Benefit | Details |
|---------|---------|
| 💵 **Free Credit** | $200 for first 30 days |
| 📅 **12-Month Free** | Popular services free for 12 months |
| ♾️ **Always Free** | 55+ services always free |

### Steps:
1. Go to **https://azure.microsoft.com/free**
2. Click "Start Free"
3. Sign in with Microsoft account
4. Verify phone number
5. Enter payment info (won't be charged!)
6. Start using Azure!

---

# SLIDE 20: Azure Portal Overview

```
┌─────────────── Azure Portal ───────────────────┐
│  ☰ Menu    🔍 Search                [?][⚙][👤] │
├────────────────────────────────────────────────┤
│                                                │
│  📊 Dashboard / Home                           │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ │
│  │  Resource  │ │  Service   │ │   Cost     │ │
│  │   Groups   │ │   Health   │ │ Management │ │
│  └────────────┘ └────────────┘ └────────────┘ │
│                                                │
│  📋 Recent Resources                           │
│  VM1 | Storage1 | VNet1 | SQL-DB1             │
│                                                │
└────────────────────────────────────────────────┘
```

**Access at:** https://portal.azure.com

---

# SLIDE 21: Ways to Manage Azure

## Multiple Management Options

| Tool | Best For |
|------|----------|
| 🖥️ **Azure Portal** | Visual management, beginners |
| 💻 **Azure CLI** | Linux admins, cross-platform |
| ⚡ **Azure PowerShell** | Windows admins, scripting |
| ☁️ **Cloud Shell** | Quick access from browser |
| 🔧 **SDKs** | Application integration |
| 🔌 **REST API** | Custom automation |

---

# SLIDE 22: Azure CLI

## Command Line Interface

### Key Features:
- Cross-platform (Windows, macOS, Linux)
- Commands start with `az`
- Bash-like syntax

### Essential Commands:
```bash
# Login to Azure
az login

# List subscriptions
az account list --output table

# Create resource group
az group create --name MyRG --location eastus

# List all VMs
az vm list --output table
```

---

# SLIDE 23: Azure PowerShell

## PowerShell Module for Azure

### Key Features:
- Uses PowerShell verb-noun syntax
- Ideal for Windows administrators
- Cross-platform with PowerShell Core

### Essential Commands:
```powershell
# Login to Azure
Connect-AzAccount

# List subscriptions
Get-AzSubscription

# Create resource group
New-AzResourceGroup -Name "MyRG" -Location "East US"

# List all VMs
Get-AzVM
```

---

# SLIDE 24: Azure Cloud Shell

## Browser-Based Shell

### Features:
- ✅ Accessible from Azure Portal
- ✅ Pre-installed with CLI & PowerShell
- ✅ 5 GB persistent storage per user
- ✅ Choose Bash or PowerShell
- ✅ Available at https://shell.azure.com

### No Installation Required!

---

# SLIDE 25: CLI vs PowerShell Comparison

| Feature | Azure CLI | Azure PowerShell |
|---------|-----------|------------------|
| **Syntax** | `az <command>` | `Verb-AzNoun` |
| **Platform** | Cross-platform | Cross-platform |
| **Output** | JSON by default | PowerShell objects |
| **Scripting** | Bash scripts | PowerShell scripts |
| **Best For** | Linux users | Windows admins |

### Both are equally powerful - choose based on your team's expertise!

---

# SLIDE 26: Azure Resource Hierarchy

## Organization Structure

```
┌─────────────────────────────────────┐
│        Management Groups            │ ← Organize subscriptions
├─────────────────────────────────────┤
│          Subscriptions              │ ← Billing & access boundary
├─────────────────────────────────────┤
│        Resource Groups              │ ← Logical containers
├─────────────────────────────────────┤
│           Resources                 │ ← VMs, Storage, DBs, etc.
└─────────────────────────────────────┘
```

---

# SLIDE 27: Subscriptions Explained

## Azure Subscriptions

### What is a Subscription?
- 💳 Billing container for Azure resources
- 🔐 Access control boundary
- 🔗 Links to an Azure AD tenant

### Types:
| Type | Description |
|------|-------------|
| **Free** | $200 credit, 12 months free |
| **Pay-As-You-Go** | Pay only for usage |
| **Enterprise Agreement** | Large organizations |
| **Visual Studio** | MSDN subscribers |

---

# SLIDE 28: Resource Groups

## Logical Containers for Resources

### What is a Resource Group?
- Container that holds related resources
- All resources MUST belong to one resource group
- Resources can only be in ONE resource group
- Deleting group deletes ALL resources inside!

### Best Practices:
- ✅ Group by lifecycle (deploy/delete together)
- ✅ Group by application
- ✅ Group by environment (Dev, Test, Prod)
- ✅ Group by department

---

# SLIDE 29: Resource Components

## Every Azure Resource Has:

| Component | Description |
|-----------|-------------|
| **Name** | Unique identifier |
| **Type** | Microsoft.Compute/virtualMachines |
| **Location** | Azure region (eastus, westeurope) |
| **Resource Group** | Parent container |
| **Tags** | Key-value pairs for organization |

### Example:
```
Name: mywebserver
Type: Microsoft.Compute/virtualMachines
Location: East US
Resource Group: Production-RG
Tags: Environment=Prod, Owner=IT
```

---

# SLIDE 30: Azure Resource Manager (ARM)

## The Control Plane

> ARM is the deployment and management service for Azure that provides a consistent management layer.

```
┌────────────────────────────────────┐
│  Portal | CLI | PowerShell | API   │
└───────────────┬────────────────────┘
                ↓
┌───────────────────────────────────────┐
│     Azure Resource Manager (ARM)      │
│  • Authentication & Authorization     │
│  • Request Processing                 │
│  • Template Deployment                │
└───────────────┬───────────────────────┘
                ↓
┌───────────────────────────────────────┐
│         Resource Providers            │
│  Compute | Storage | Network | SQL    │
└───────────────────────────────────────┘
```

---

# SLIDE 31: Benefits of ARM

## Why ARM is Powerful

| Benefit | Description |
|---------|-------------|
| 📝 **Declarative Templates** | Define what, ARM handles how |
| 🔄 **Dependency Management** | Deploys in correct order |
| 🔧 **Consistent Management** | Same API for all resources |
| 🔐 **Access Control** | RBAC for fine-grained access |
| 🏷️ **Tagging** | Organize and track resources |
| 💰 **Billing** | View costs by tags or groups |

---

# SLIDE 32: ARM Templates

## Infrastructure as Code

### What is an ARM Template?
- JSON file defining infrastructure
- Declarative syntax
- Repeatable deployments
- Version controlled in Git

### Template Structure:
```json
{
  "$schema": "...",
  "contentVersion": "1.0.0.0",
  "parameters": { },
  "variables": { },
  "resources": [ ],
  "outputs": { }
}
```

---

# SLIDE 33: ARM Template Sections

| Section | Purpose |
|---------|---------|
| **$schema** | JSON schema location |
| **contentVersion** | Template version |
| **parameters** | Values provided at deployment |
| **variables** | Computed values |
| **resources** | Azure resources to deploy |
| **outputs** | Values returned after deployment |

---

# SLIDE 34: Deploying ARM Templates

## Two Ways to Deploy

### Using PowerShell:
```powershell
New-AzResourceGroupDeployment `
  -ResourceGroupName "MyResourceGroup" `
  -TemplateFile "template.json"
```

### Using Azure CLI:
```bash
az deployment group create \
  --resource-group MyResourceGroup \
  --template-file template.json
```

---

# SLIDE 35: Bicep - Modern Alternative

## Simpler Syntax for ARM

### ARM Template (JSON):
```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2021-02-01",
  "name": "[parameters('name')]",
  ...
}
```

### Same in Bicep:
```bicep
resource storage 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  location: resourceGroup().location
  kind: 'StorageV2'
}
```

**Bicep compiles to ARM templates!**

---

# SLIDE 36: Availability Concepts

## High Availability in Azure

### Availability Sets
- Logical grouping within a datacenter
- **Fault Domains**: Separate power/network (max 3)
- **Update Domains**: Separate maintenance (max 20)

### Availability Zones
- Physically separate locations in a region
- Independent power, cooling, networking
- **99.99% SLA**

---

# SLIDE 37: Module Summary

## Key Takeaways

✅ **Cloud Computing**: IaaS, PaaS, SaaS - know the differences

✅ **Azure**: 60+ regions, 200+ services, enterprise-ready

✅ **Management Tools**: Portal, CLI, PowerShell, Cloud Shell

✅ **Resource Hierarchy**: Management Groups → Subscriptions → Resource Groups → Resources

✅ **ARM**: The deployment and management layer for all Azure

✅ **Templates**: Infrastructure as Code for repeatable deployments

---

# SLIDE 38: Demo Time! 🎯

## Live Demonstration

1. **Azure Portal Navigation**
   - Dashboard overview
   - Resource creation wizard

2. **Azure Cloud Shell**
   - CLI commands demo
   - PowerShell commands demo

3. **Create a Resource Group**
   - Using Portal
   - Using CLI/PowerShell

4. **Explore Azure Services**
   - Browse marketplace
   - View pricing calculator

---

# SLIDE 39: Practice Exercises

## Try These Yourself!

1. ✏️ Create a free Azure account
2. ✏️ Navigate the Azure Portal
3. ✏️ Open Cloud Shell and run basic commands
4. ✏️ Create a Resource Group using CLI
5. ✏️ Create a Resource Group using PowerShell
6. ✏️ Explore the Azure Marketplace
7. ✏️ Use the Azure Pricing Calculator

---

# SLIDE 40: Questions & Next Module

## Any Questions?

### 📚 Next Module:
**Module 02: Introduction to ARM and Azure Storage**
- Deep dive into ARM Templates
- Azure Storage Account types
- Blob, Files, Queues, Tables

### 📖 Resources:
- Microsoft Learn: https://learn.microsoft.com/azure
- Azure Documentation: https://docs.microsoft.com/azure
- Azure Free Account: https://azure.microsoft.com/free

---

# END OF PRESENTATION

## Conversion Tips for PowerPoint:

1. **Each "# SLIDE" section** = One PowerPoint slide
2. **Copy content** between slide markers
3. **Use tables** as provided for professional look
4. **Code blocks** → Use Consolas/Courier font
5. **Diagrams** → Recreate using SmartArt or shapes

### Suggested Theme:
- Use Microsoft Azure blue color (#0078D4)
- Clean, minimal design
- Add Azure icons from Microsoft's icon library

### Download Azure Icons:
https://docs.microsoft.com/en-us/azure/architecture/icons/
