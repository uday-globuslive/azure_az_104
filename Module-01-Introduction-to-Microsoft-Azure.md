# Module 01 - Introduction to Microsoft Azure

## Learning Objectives
By the end of this module, you will be able to:
- Understand cloud computing fundamentals and service models
- Explain what Microsoft Azure is and its core services
- Create and manage an Azure account
- Use Azure CLI and Azure PowerShell
- Understand Azure Resource Manager and architecture

---

## 1.1 Introduction to Cloud Computing

### What is Cloud Computing?
Cloud computing is the delivery of computing services—including servers, storage, databases, networking, software, analytics, and intelligence—over the Internet ("the cloud") to offer faster innovation, flexible resources, and economies of scale.

### Key Characteristics of Cloud Computing

| Characteristic | Description |
|----------------|-------------|
| **On-demand self-service** | Users can provision computing resources automatically without requiring human interaction |
| **Broad network access** | Services are available over the network and accessed through standard mechanisms |
| **Resource pooling** | Provider's computing resources are pooled to serve multiple consumers |
| **Rapid elasticity** | Capabilities can be elastically provisioned and released to scale rapidly |
| **Measured service** | Resource usage is monitored, controlled, and reported (pay-per-use) |

### Cloud Service Models

#### 1. Infrastructure as a Service (IaaS)
- **Definition**: Provides virtualized computing resources over the internet
- **What you manage**: Applications, Data, Runtime, Middleware, OS
- **Provider manages**: Virtualization, Servers, Storage, Networking
- **Examples**: Azure Virtual Machines, Azure Storage, Azure Virtual Networks

##### 🏢 Real-World Use Cases for IaaS:

| Industry | Scenario | Implementation |
|----------|----------|----------------|
| **E-commerce** | Black Friday traffic spike | Company provisions 50 additional VMs during peak shopping, scales down after |
| **Healthcare** | HIPAA-compliant imaging system | Hospital runs custom radiology software on VMs with specific compliance configurations |
| **Finance** | Legacy banking application | Bank migrates 20-year-old mainframe app to VMs without code changes (lift-and-shift) |
| **Gaming** | Game server hosting | Studio deploys dedicated game servers across regions for low-latency multiplayer |
| **Manufacturing** | SAP ERP deployment | Factory runs SAP HANA on specialized M-series VMs with 4TB RAM |

**Real Scenario - Contoso Retail:**
```
Challenge: Contoso's on-premises servers crash every Black Friday
Solution: 
- Normal days: 10 VMs running web application
- Black Friday week: Auto-scale to 100 VMs
- Post-holiday: Scale back to 10 VMs
Cost savings: 90% vs. maintaining 100 physical servers year-round
```

```
┌─────────────────────────────────┐
│  Applications       (You)       │
│  Data               (You)       │
│  Runtime            (You)       │
│  Middleware         (You)       │
│  Operating System   (You)       │
├─────────────────────────────────┤
│  Virtualization     (Provider)  │
│  Servers            (Provider)  │
│  Storage            (Provider)  │
│  Networking         (Provider)  │
└─────────────────────────────────┘
```

#### 2. Platform as a Service (PaaS)
- **Definition**: Provides a platform allowing customers to develop, run, and manage applications
- **What you manage**: Applications, Data
- **Provider manages**: Everything else (Runtime, Middleware, OS, Infrastructure)
- **Examples**: Azure App Service, Azure SQL Database, Azure Functions

##### 🏢 Real-World Use Cases for PaaS:

| Industry | Scenario | Implementation |
|----------|----------|----------------|
| **Startup** | MVP development | Startup deploys Node.js API on App Service in 2 hours, no infra management |
| **Insurance** | Claims processing API | Company builds REST API for mobile app, auto-scales based on claim submissions |
| **Media** | Video transcoding | Broadcasting company uses Azure Functions to auto-encode uploaded videos |
| **Logistics** | Real-time tracking | Shipping company uses Azure SQL for tracking millions of packages |
| **Education** | Learning management | University deploys web portal for 50,000 students with zero server management |

**Real Scenario - FastFood Chain Mobile Ordering:**
```
Challenge: Build mobile ordering system in 3 months
Solution:
- Azure App Service: Hosts ordering API (auto-scales 2-50 instances)
- Azure SQL Database: Stores orders with automatic backups
- Azure Functions: Processes payment notifications
- Cost: $2,000/month vs. $15,000/month for equivalent IaaS
- Developer focus: 100% on code, 0% on infrastructure
```

```
┌─────────────────────────────────┐
│  Applications       (You)       │
│  Data               (You)       │
├─────────────────────────────────┤
│  Runtime            (Provider)  │
│  Middleware         (Provider)  │
│  Operating System   (Provider)  │
│  Virtualization     (Provider)  │
│  Servers            (Provider)  │
│  Storage            (Provider)  │
│  Networking         (Provider)  │
└─────────────────────────────────┘
```

#### 3. Software as a Service (SaaS)
- **Definition**: Software distribution model where applications are hosted by a service provider
- **What you manage**: Only your data and access
- **Provider manages**: Everything including the application
- **Examples**: Microsoft 365, Dynamics 365, Salesforce

##### 🏢 Real-World Use Cases for SaaS:

| Industry | Scenario | Implementation |
|----------|----------|----------------|
| **Legal Firm** | Email & collaboration | Law firm uses Microsoft 365 for secure email, document sharing |
| **Sales Team** | CRM system | Company uses Dynamics 365 to track 10,000 customer relationships |
| **HR Department** | Employee management | HR team uses Workday for payroll, benefits, performance reviews |
| **Marketing** | Campaign management | Marketing dept uses HubSpot for email campaigns, lead tracking |
| **Support** | Help desk tickets | Support team uses Zendesk for customer inquiries |

**Real Scenario - Global Consulting Firm:**
```
Challenge: 5,000 employees across 30 countries need collaboration tools
Solution: Microsoft 365 E5
- Email: Exchange Online (no mail servers to manage)
- Files: OneDrive & SharePoint (1TB per user)
- Communication: Teams (video, chat, channels)
- Security: Built-in compliance, DLP, eDiscovery
Deployment time: 2 weeks vs. 6 months for on-premises
Annual cost: $2.1M vs. $8M for equivalent infrastructure
```

```
┌─────────────────────────────────┐
│  Data Access        (You)       │
├─────────────────────────────────┤
│  Applications       (Provider)  │
│  Data               (Provider)  │
│  Runtime            (Provider)  │
│  Middleware         (Provider)  │
│  Operating System   (Provider)  │
│  Infrastructure     (Provider)  │
└─────────────────────────────────┘
```

### Cloud Deployment Models

| Model | Description | Best For |
|-------|-------------|----------|
| **Public Cloud** | Resources owned and operated by third-party cloud service provider | Scalability, cost-effectiveness |
| **Private Cloud** | Cloud infrastructure used exclusively by a single organization | Security, compliance requirements |
| **Hybrid Cloud** | Combination of public and private clouds | Flexibility, gradual migration |

#### 🏢 Real-World Deployment Model Use Cases:

**Public Cloud - Streaming Service:**
```
Company: VideoStream Inc.
Challenge: Handle 50M concurrent viewers during live events
Solution: 100% Azure public cloud
- CDN for content delivery globally
- Auto-scaling compute for encoding
- Cosmos DB for user preferences
Why Public: Unpredictable demand, global reach needed
```

**Private Cloud - Government Agency:**
```
Organization: Department of Defense Contractor
Challenge: Process classified data with strict compliance
Solution: Azure Government (isolated regions)
- Dedicated hardware, no multi-tenancy
- FedRAMP High, IL5 compliance
- Air-gapped networks for sensitive workloads
Why Private: Legal requirement, data sovereignty
```

**Hybrid Cloud - Hospital Network:**
```
Organization: Regional Healthcare System (15 hospitals)
Challenge: Modernize while keeping patient data on-premises
Solution: Hybrid architecture
- On-premises: Electronic Health Records (EHR), compliance
- Azure: AI/ML for diagnostics, disaster recovery
- Azure Arc: Manage all resources from single pane
- ExpressRoute: Private 10Gbps connection
Why Hybrid: Regulatory compliance + innovation
```

### Benefits of Cloud Computing

1. **Cost Efficiency**: No upfront infrastructure costs; pay only for what you use
2. **Scalability**: Scale resources up or down based on demand
3. **Reliability**: Data backup, disaster recovery, and business continuity
4. **Security**: Broad set of policies and technologies to protect data
5. **Global Reach**: Deploy applications in multiple regions worldwide
6. **Performance**: Latest hardware and fast networking infrastructure

### Teaching Tip 💡
> When explaining cloud computing to beginners, use the electricity analogy: Just like you don't build your own power plant but use electricity from a provider and pay for what you use, cloud computing lets you use computing resources without owning the infrastructure.

---

## 1.2 What is Microsoft Azure?

### Definition
Microsoft Azure is a comprehensive cloud computing platform and infrastructure created by Microsoft. It provides a wide range of cloud services, including compute, analytics, storage, and networking.

### History of Azure
- **2008**: Project "Red Dog" started
- **2010**: Windows Azure launched
- **2014**: Renamed to Microsoft Azure
- **Present**: One of the top 3 cloud providers globally

### Azure's Global Infrastructure

#### Regions
- Azure has **60+ regions** worldwide (more than any other cloud provider)
- A region is a geographical area containing one or more datacenters
- Examples: East US, West Europe, Southeast Asia, Central India

#### Availability Zones
- Physically separate locations within an Azure region
- Each zone has independent power, cooling, and networking
- Minimum of **3 separate zones** in enabled regions
- Provides **99.99% VM uptime SLA**

```
┌─────────────────── Azure Region ───────────────────┐
│                                                     │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │
│  │   Zone 1    │  │   Zone 2    │  │   Zone 3    │ │
│  │             │  │             │  │             │ │
│  │ Datacenter  │  │ Datacenter  │  │ Datacenter  │ │
│  │    A, B     │  │    C, D     │  │    E, F     │ │
│  └─────────────┘  └─────────────┘  └─────────────┘ │
│        ↕               ↕               ↕           │
│  ─────────── Low-latency Network ──────────────    │
└─────────────────────────────────────────────────────┘
```

#### Region Pairs
- Each region is paired with another region within the same geography
- Provides data residency and disaster recovery
- Examples: East US ↔ West US, North Europe ↔ West Europe

### Azure vs Other Cloud Providers

| Feature | Azure | AWS | Google Cloud |
|---------|-------|-----|--------------|
| Global Regions | 60+ | 31 | 35 |
| Hybrid Support | Azure Arc, Azure Stack | AWS Outposts | Anthos |
| Enterprise Integration | Excellent (Microsoft ecosystem) | Good | Good |
| AI/ML Services | Azure AI, Cognitive Services | SageMaker | Vertex AI |

---

## 1.3 Microsoft Azure Services

### Azure Service Categories

#### 1. Compute Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure Virtual Machines** | IaaS virtual servers | Running traditional applications |
| **Azure App Service** | PaaS for web applications | Web apps, REST APIs |
| **Azure Functions** | Serverless compute | Event-driven processing |
| **Azure Kubernetes Service (AKS)** | Managed Kubernetes | Container orchestration |
| **Azure Container Instances** | Serverless containers | Quick container deployment |

#### 2. Storage Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure Blob Storage** | Object storage | Unstructured data, media files |
| **Azure Files** | Managed file shares | Lift-and-shift applications |
| **Azure Queue Storage** | Message queuing | Decoupling applications |
| **Azure Table Storage** | NoSQL key-value store | Semi-structured data |
| **Azure Disk Storage** | Managed disks for VMs | VM data storage |

#### 3. Networking Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure Virtual Network (VNet)** | Private network in Azure | Isolating resources |
| **Azure Load Balancer** | Layer 4 load balancing | Distributing traffic |
| **Azure Application Gateway** | Layer 7 load balancing | Web application traffic |
| **Azure VPN Gateway** | VPN connectivity | Hybrid connectivity |
| **Azure ExpressRoute** | Private connection to Azure | High-speed, dedicated connection |

#### 4. Database Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure SQL Database** | Managed SQL database | Relational data |
| **Azure Cosmos DB** | Globally distributed NoSQL | Multi-region applications |
| **Azure Database for MySQL** | Managed MySQL | Open-source MySQL apps |
| **Azure Database for PostgreSQL** | Managed PostgreSQL | Open-source PostgreSQL apps |

#### 5. Identity Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure Active Directory** | Identity management | Authentication, SSO |
| **Azure AD B2C** | Consumer identity | Customer-facing apps |
| **Azure AD Domain Services** | Managed domain services | Legacy app support |

#### 6. Security Services
| Service | Description | Use Case |
|---------|-------------|----------|
| **Azure Security Center** | Security management | Threat protection |
| **Azure Key Vault** | Secrets management | Storing keys, certificates |
| **Azure DDoS Protection** | DDoS protection | Protecting against attacks |

### Teaching Tip 💡
> Organize services by the problem they solve, not by technical categories. For example: "Need to host a website? Use App Service. Need to run Windows Server? Use Virtual Machines."

---

## 1.4 Creating a Microsoft Azure Account

### Prerequisites
- Valid email address
- Phone number for verification
- Credit/Debit card (for verification, not charged for free tier)
- Microsoft account (or create one during signup)

### Step-by-Step Account Creation

#### Step 1: Navigate to Azure Portal
```
URL: https://azure.microsoft.com/free
```

#### Step 2: Click "Start Free"
- Choose between:
  - **Free Account**: $200 credit for 30 days + 12 months of free services
  - **Pay-As-You-Go**: No commitment, pay only for what you use

#### Step 3: Sign In or Create Microsoft Account
- Use existing Microsoft account (Outlook, Hotmail, etc.)
- Or create a new account

#### Step 4: Identity Verification
- Enter phone number
- Receive and enter verification code

#### Step 5: Payment Information
- Enter credit card details
- **Important**: You won't be charged unless you explicitly upgrade

#### Step 6: Agreement and Create
- Accept terms and conditions
- Click "Sign up"

### Azure Free Account Benefits

| Benefit | Details |
|---------|---------|
| **Free Credit** | $200 for first 30 days |
| **12-Month Free Services** | Popular services free for 12 months |
| **Always Free Services** | 55+ services always free |

### Always Free Services Include:
- Azure App Service (10 web apps)
- Azure Functions (1 million requests/month)
- Azure Cosmos DB (25 GB storage)
- Azure DevOps (5 users)
- And many more...

### Azure Portal Overview

```
┌─────────────────── Azure Portal ───────────────────────────┐
│  ┌────────┐  Search bar                     [?] [⚙] [👤]  │
│  │ ☰ Menu │  ─────────────────────────────                 │
├──┴────────┴────────────────────────────────────────────────┤
│                                                             │
│  Dashboard / Home                                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │  Resource   │  │  Service    │  │   Cost      │         │
│  │  Groups     │  │   Health    │  │  Management │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│  Recent Resources                                           │
│  ┌─────────────────────────────────────────────────┐       │
│  │  VM1 | Storage1 | VNet1 | SQL-DB1              │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 1.5 Azure CLI and Azure PowerShell

### Overview
Azure provides multiple ways to manage resources:
1. **Azure Portal** - Web-based graphical interface
2. **Azure CLI** - Cross-platform command-line tool
3. **Azure PowerShell** - PowerShell module for Azure
4. **Azure Cloud Shell** - Browser-based shell
5. **Azure SDKs** - Programming libraries
6. **Azure REST API** - Direct API access

### Azure CLI

#### What is Azure CLI?
- Cross-platform command-line interface
- Available on Windows, macOS, Linux
- Commands start with `az`
- Uses bash-like syntax

#### Installing Azure CLI

**Windows (PowerShell):**
```powershell
# Using MSI installer (recommended)
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'

# Using winget
winget install -e --id Microsoft.AzureCLI
```

**macOS:**
```bash
brew update && brew install azure-cli
```

**Linux (Ubuntu/Debian):**
```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### Azure CLI Basic Commands

```bash
# Login to Azure
az login

# Show current account
az account show

# List all subscriptions
az account list --output table

# Set active subscription
az account set --subscription "Subscription-Name"

# List resource groups
az group list --output table

# Create a resource group
az group create --name MyResourceGroup --location eastus

# List all VMs
az vm list --output table

# Get help for any command
az vm create --help
```

#### Output Formats
```bash
# JSON (default)
az group list

# Table format (human-readable)
az group list --output table

# TSV (Tab-separated values)
az group list --output tsv

# YAML format
az group list --output yaml
```

### Azure PowerShell

#### What is Azure PowerShell?
- PowerShell module for managing Azure resources
- Uses PowerShell verb-noun syntax (Get-, Set-, New-, Remove-)
- Ideal for Windows administrators
- Cross-platform with PowerShell Core

#### Installing Azure PowerShell

```powershell
# Install the Az module
Install-Module -Name Az -Repository PSGallery -Force

# Update Az module
Update-Module -Name Az

# Import the module
Import-Module Az
```

#### Azure PowerShell Basic Commands

```powershell
# Login to Azure
Connect-AzAccount

# Get current context
Get-AzContext

# List all subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -SubscriptionName "Subscription-Name"

# List resource groups
Get-AzResourceGroup

# Create a resource group
New-AzResourceGroup -Name "MyResourceGroup" -Location "East US"

# List all VMs
Get-AzVM

# Get help for any command
Get-Help New-AzVM -Detailed
```

### Azure Cloud Shell

#### Features
- Browser-based shell accessible from Azure Portal
- Pre-installed with Azure CLI and PowerShell
- Persistent storage (5 GB per user)
- Access from anywhere with a browser
- Choose between Bash or PowerShell

#### Accessing Cloud Shell
1. Click the Cloud Shell icon (>_) in Azure Portal
2. Or go to https://shell.azure.com

### Comparison: CLI vs PowerShell

| Feature | Azure CLI | Azure PowerShell |
|---------|-----------|------------------|
| Syntax | `az <command>` | `Verb-AzNoun` |
| Platform | Cross-platform | Cross-platform |
| Scripting | Bash scripting | PowerShell scripting |
| Output | JSON by default | Objects |
| Learning curve | Easier for Linux users | Easier for Windows admins |

### Teaching Tip 💡
> Choose the tool that fits your team's expertise. Linux backgrounds → Azure CLI. Windows/Microsoft backgrounds → Azure PowerShell. Both are equally capable.

---

## 1.6 Managing Azure Resources & Subscriptions

### Azure Resource Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│                    Management Groups                         │
│    (Organize subscriptions for governance and compliance)    │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                      Subscriptions                           │
│    (Billing boundary and access control boundary)            │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                    Resource Groups                           │
│    (Logical containers for resources)                        │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                       Resources                              │
│    (VMs, Storage accounts, Databases, etc.)                 │
└─────────────────────────────────────────────────────────────┘
```

### Management Groups
- Container for organizing subscriptions
- Apply governance conditions (policies, RBAC)
- Up to 6 levels of depth
- One root management group per directory

```
Root Management Group
├── IT Department
│   ├── Development Subscription
│   └── Testing Subscription
├── Finance Department
│   └── Finance Subscription
└── Production
    ├── Prod-East Subscription
    └── Prod-West Subscription
```

### Subscriptions

#### What is a Subscription?
- A billing container for Azure resources
- Access control boundary
- Links to an Azure AD tenant

#### Types of Subscriptions
| Type | Description |
|------|-------------|
| **Free** | $200 credit, 12 months free services |
| **Pay-As-You-Go** | Pay only for what you use |
| **Enterprise Agreement** | For large organizations |
| **CSP** | Through Microsoft partners |
| **Visual Studio** | For MSDN subscribers |

#### Subscription Limits (Selected)
| Resource | Default Limit |
|----------|---------------|
| Resource groups per subscription | 980 |
| VMs per subscription | 25,000 |
| Storage accounts per region | 250 |
| Virtual networks per subscription | 1,000 |

### Resource Groups

#### What is a Resource Group?
- Logical container for Azure resources
- All resources must belong to one resource group
- Resources can only be in one resource group
- Resource groups cannot be nested

#### Best Practices for Resource Groups
1. **Group by lifecycle**: Resources deployed, updated, deleted together
2. **Group by application**: All resources for one application
3. **Group by environment**: Dev, Test, Prod
4. **Group by department**: IT, Finance, HR

```powershell
# Create resource group
New-AzResourceGroup -Name "Production-RG" -Location "East US"

# List all resources in a group
Get-AzResource -ResourceGroupName "Production-RG"

# Delete resource group (deletes all resources inside)
Remove-AzResourceGroup -Name "Production-RG"
```

### Resources

#### Resource Components
Every Azure resource has:
- **Name**: Unique identifier (naming conventions vary by resource type)
- **Type**: Microsoft.Compute/virtualMachines, Microsoft.Storage/storageAccounts
- **Location**: Azure region where resource is deployed
- **Resource Group**: Container for the resource
- **Tags**: Key-value pairs for organization

---

## 1.7 Azure Resource Manager (ARM)

### What is Azure Resource Manager?
Azure Resource Manager (ARM) is the deployment and management service for Azure. It provides a consistent management layer for all Azure operations.

### ARM Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Clients                              │
│   Portal | CLI | PowerShell | SDKs | REST API               │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Azure Resource Manager (ARM)                    │
│   • Authentication & Authorization                          │
│   • Request Processing                                      │
│   • Template Deployment                                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                   Resource Providers                         │
│   Microsoft.Compute | Microsoft.Storage | Microsoft.Network │
│   Microsoft.Web | Microsoft.Sql | Microsoft.KeyVault        │
└─────────────────────────────────────────────────────────────┘
```

### Benefits of ARM

| Benefit | Description |
|---------|-------------|
| **Declarative Templates** | Define what you want, ARM handles how |
| **Dependency Management** | ARM deploys resources in correct order |
| **Consistent Management** | Same API for all resources |
| **Access Control** | RBAC for fine-grained access |
| **Tagging** | Organize and track resources |
| **Billing** | View costs by tags or groups |

### ARM Templates

#### What is an ARM Template?
- JSON file defining infrastructure as code
- Declarative syntax
- Idempotent deployments
- Version controlled

#### ARM Template Structure
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the storage account"
      }
    }
  },
  "variables": {
    "storageAccountSku": "Standard_LRS"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "[variables('storageAccountSku')]"
      },
      "kind": "StorageV2"
    }
  ],
  "outputs": {
    "storageEndpoint": {
      "type": "string",
      "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
    }
  }
}
```

#### Template Sections Explained

| Section | Purpose |
|---------|---------|
| **$schema** | Location of the JSON schema file |
| **contentVersion** | Version of the template |
| **parameters** | Values provided during deployment |
| **variables** | Values constructed within template |
| **resources** | Azure resources to deploy |
| **outputs** | Values returned after deployment |

### Deploying ARM Templates

```powershell
# Deploy using PowerShell
New-AzResourceGroupDeployment `
  -ResourceGroupName "MyResourceGroup" `
  -TemplateFile "template.json" `
  -TemplateParameterFile "parameters.json"
```

```bash
# Deploy using Azure CLI
az deployment group create \
  --resource-group MyResourceGroup \
  --template-file template.json \
  --parameters @parameters.json
```

### Bicep - Modern Alternative

Bicep is a domain-specific language (DSL) that compiles to ARM templates:

```bicep
param storageAccountName string

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-02-01' = {
  name: storageAccountName
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

---

## 1.8 Microsoft Azure Architecture

### Global Infrastructure Components

#### Geographies
- Discrete markets preserving data residency and compliance
- Contains two or more regions
- Examples: United States, Europe, Asia Pacific

#### Regions
- Physical location with one or more datacenters
- Low-latency network connection between datacenters
- Some services are region-specific

#### Availability Zones
- Unique physical locations within a region
- Made up of one or more datacenters
- Independent power, cooling, networking
- Connected via high-speed, private fiber

#### Datacenters
- Physical buildings with computing infrastructure
- Multiple datacenters form Availability Zones

### Availability Concepts

#### Availability Sets
- Logical grouping of VMs within a datacenter
- Protects against hardware failures
- **Fault Domains**: Separate power and network (max 3)
- **Update Domains**: Separate maintenance windows (max 20)

```
Availability Set
├── Fault Domain 0
│   ├── Update Domain 0 [VM1]
│   ├── Update Domain 1 [VM3]
│   └── Update Domain 2 [VM5]
├── Fault Domain 1
│   ├── Update Domain 0 [VM2]
│   ├── Update Domain 1 [VM4]
│   └── Update Domain 2 [VM6]
└── Fault Domain 2
    ├── Update Domain 0 [VM7]
    └── Update Domain 1 [VM8]
```

#### Availability Zones
- Higher level of availability than Availability Sets
- Protect against entire datacenter failures
- SLA: 99.99% uptime

| Protection Level | Availability Set | Availability Zone |
|------------------|------------------|-------------------|
| Protects against | Hardware failure | Datacenter failure |
| SLA | 99.95% | 99.99% |
| Scope | Single datacenter | Multiple datacenters |

### Service Level Agreements (SLAs)

| Service | SLA |
|---------|-----|
| Single VM (Premium SSD) | 99.9% |
| VMs in Availability Set | 99.95% |
| VMs in Availability Zones | 99.99% |
| Azure App Service | 99.95% |
| Azure SQL Database | 99.99% |

### Calculating Composite SLA
For services working together, multiply individual SLAs:
```
Web App (99.95%) × SQL Database (99.99%) = 99.94%
```

---

## Hands-on Exercises

### Exercise 1: Creating a Microsoft Azure Account
1. Go to https://azure.microsoft.com/free
2. Click "Start free"
3. Sign in with Microsoft account
4. Complete verification
5. Accept terms and create account
6. Explore the Azure Portal dashboard

### Exercise 2: Configuring Azure PowerShell
```powershell
# Step 1: Install Azure PowerShell module
Install-Module -Name Az -AllowClobber -Scope CurrentUser

# Step 2: Import the module
Import-Module Az

# Step 3: Connect to Azure
Connect-AzAccount

# Step 4: Verify connection
Get-AzContext

# Step 5: List subscriptions
Get-AzSubscription

# Step 6: Create a test resource group
New-AzResourceGroup -Name "Test-RG" -Location "East US"

# Step 7: Verify creation
Get-AzResourceGroup -Name "Test-RG"

# Step 8: Clean up
Remove-AzResourceGroup -Name "Test-RG" -Force
```

### Exercise 3: Configuring Azure CLI
```bash
# Step 1: Install Azure CLI (varies by OS)
# Windows: Download from https://aka.ms/installazurecliwindows

# Step 2: Verify installation
az --version

# Step 3: Login to Azure
az login

# Step 4: Verify connection
az account show

# Step 5: List subscriptions
az account list --output table

# Step 6: Create a test resource group
az group create --name "CLI-Test-RG" --location "westus"

# Step 7: Verify creation
az group show --name "CLI-Test-RG"

# Step 8: Clean up
az group delete --name "CLI-Test-RG" --yes
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Cloud Computing | IaaS, PaaS, SaaS models; On-demand, scalable resources |
| Microsoft Azure | 60+ regions, comprehensive cloud services |
| Azure Services | Compute, Storage, Networking, Database, Identity |
| Azure Account | Free tier available, $200 credit for 30 days |
| Azure CLI | Cross-platform, `az` commands, bash-like |
| Azure PowerShell | PowerShell module, `Verb-AzNoun` syntax |
| Resource Hierarchy | Management Groups → Subscriptions → Resource Groups → Resources |
| ARM | Deployment service, JSON templates, declarative |
| Architecture | Regions, Availability Zones, High availability |

---

## Review Questions

1. What are the three cloud service models? Explain each.
2. What is the difference between a region and an availability zone?
3. How do Azure CLI and Azure PowerShell differ?
4. What is Azure Resource Manager and what are its benefits?
5. Explain the hierarchy of Azure resources.
6. What is the purpose of availability sets vs availability zones?

---

## Additional Resources

- [Microsoft Learn - Azure Fundamentals](https://docs.microsoft.com/learn/paths/azure-fundamentals/)
- [Azure Documentation](https://docs.microsoft.com/azure/)
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Azure CLI Documentation](https://docs.microsoft.com/cli/azure/)
- [Azure PowerShell Documentation](https://docs.microsoft.com/powershell/azure/)

---

## 1.6 Advanced Azure Architecture Concepts (Expert Level)

### Azure Resource Manager (ARM) Deep Dive

ARM is the deployment and management service for Azure. Every request to Azure services goes through ARM.

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          ARM Request Flow                                 │
│                                                                           │
│  User Request ──► Azure AD Auth ──► ARM Endpoint ──► Resource Provider  │
│                                                                           │
│  Resource Provider Types:                                                 │
│  ├── Microsoft.Compute (VMs, Disks, VMSS)                                │
│  ├── Microsoft.Storage (Storage Accounts, Blobs)                         │
│  ├── Microsoft.Network (VNets, Load Balancers, DNS)                     │
│  ├── Microsoft.Web (App Service, Functions)                              │
│  └── Microsoft.Sql (SQL Database, Managed Instance)                     │
└──────────────────────────────────────────────────────────────────────────┘
```

### ARM API and Control Plane

```powershell
# List all resource providers in your subscription
Get-AzResourceProvider | Where-Object { $_.RegistrationState -eq "Registered" } | 
    Select-Object ProviderNamespace, RegistrationState | 
    Sort-Object ProviderNamespace

# Register a provider (required before using certain services)
Register-AzResourceProvider -ProviderNamespace "Microsoft.Kubernetes"

# Get available API versions for a resource type
(Get-AzResourceProvider -ProviderNamespace Microsoft.Compute).ResourceTypes |
    Where-Object { $_.ResourceTypeName -eq "virtualMachines" } |
    Select-Object -ExpandProperty ApiVersions | Select-Object -First 5
```

### Azure Pricing Models and Cost Optimization

| Pricing Model | Description | Savings | Use Case |
|---------------|-------------|---------|----------|
| **Pay-As-You-Go** | Pay per usage, no commitment | 0% | Variable/unpredictable workloads |
| **Reserved Instances** | 1 or 3-year commitment | Up to 72% | Steady-state workloads |
| **Spot VMs** | Unused capacity, interruptible | Up to 90% | Batch processing, fault-tolerant |
| **Hybrid Benefit** | Use existing licenses | Up to 40% | Windows Server/SQL migration |
| **Dev/Test** | Discounted pricing | ~50% | Non-production environments |

#### Cost Management Best Practices

```powershell
# Tag all resources for cost allocation
$costTags = @{
    CostCenter   = "IT-12345"
    Environment  = "Production"
    Application  = "ERP"
    Owner        = "finance@company.com"
    Budget       = "150000"
}

# Apply tags to resource group
Set-AzResourceGroup -Name "MyRG" -Tag $costTags

# Create budget alert using Azure CLI
az consumption budget create `
    --budget-name "Monthly-IT-Budget" `
    --amount 10000 `
    --time-grain Monthly `
    --category Cost `
    --resource-group "MyRG" `
    --notifications 80=Actual,90=Actual,100=Forecasted
```

### Azure Service Level Agreements (SLAs)

| Service | SLA | Conditions |
|---------|-----|------------|
| Virtual Machines (Single) | 99.9% | Premium SSD |
| Virtual Machines (Availability Set) | 99.95% | 2+ VMs in different fault domains |
| Virtual Machines (Availability Zone) | 99.99% | 2+ VMs across zones |
| Azure SQL Database | 99.99% | Standard/Premium tier |
| Azure App Service | 99.95% | Standard tier and above |
| Azure Storage (GRS) | 99.99% read, 99.9% write | GRS/RA-GRS enabled |

#### Composite SLA Calculation

```
Web App (99.95%) → SQL Database (99.99%) → Storage (99.9%)

Composite SLA = 0.9995 × 0.9999 × 0.999 = 99.84%

With redundancy (2 web apps):
Web Tier = 1 - (1 - 0.9995)² = 99.999975%
Composite = 0.99999975 × 0.9999 × 0.999 = 99.89%
```

### Azure Compliance and Governance

```powershell
# View compliance offerings
$complianceOfferings = @(
    "ISO 27001", "SOC 1/2/3", "HIPAA", "PCI DSS", 
    "FedRAMP High", "GDPR", "HITRUST", "CSA STAR"
)

# Enable Azure Policy for compliance
$policyDefinition = Get-AzPolicyDefinition | 
    Where-Object { $_.Properties.DisplayName -like "*require tag*" }

New-AzPolicyAssignment `
    -Name "RequireCostCenterTag" `
    -PolicyDefinition $policyDefinition `
    -Scope "/subscriptions/$subscriptionId" `
    -PolicyParameterObject @{ tagName = "CostCenter" }
```

### Azure Well-Architected Framework

The five pillars for building reliable cloud applications:

| Pillar | Description | Key Services |
|--------|-------------|--------------|
| **Reliability** | Recover from failures | Availability Zones, Backup, Site Recovery |
| **Security** | Protect against threats | Defender, Key Vault, DDoS Protection |
| **Cost Optimization** | Manage costs | Cost Management, Advisor, Reserved Instances |
| **Operational Excellence** | Deploy and monitor | Monitor, DevOps, Resource Graph |
| **Performance Efficiency** | Meet workload demands | CDN, Load Balancer, Autoscale |

### Enterprise Subscription Organization

```
Root Management Group
├── Platform Management Group
│   ├── Identity Subscription
│   │   └── Azure AD, Domain Services
│   ├── Connectivity Subscription
│   │   └── Hub VNet, VPN Gateway, Firewall
│   └── Management Subscription
│       └── Log Analytics, Automation
├── Landing Zones Management Group  
│   ├── Production Subscription
│   │   └── Production workloads
│   └── Non-Production Subscription
│       └── Dev/Test workloads
└── Sandbox Management Group
    └── Innovation Subscription
        └── Experimentation
```

```powershell
# Create management group hierarchy
New-AzManagementGroup -GroupName "Platform" -DisplayName "Platform"
New-AzManagementGroup -GroupName "LandingZones" -DisplayName "Landing Zones" 
New-AzManagementGroup -GroupName "Sandbox" -DisplayName "Sandbox"

# Move subscription to management group
New-AzManagementGroupSubscription `
    -GroupName "LandingZones" `
    -SubscriptionId $subscriptionId
```
