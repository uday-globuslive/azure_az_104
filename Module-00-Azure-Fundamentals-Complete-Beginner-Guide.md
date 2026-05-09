# Module 00 - Azure Fundamentals: Complete Beginner Guide

## 🎯 Purpose of This Guide
This guide is designed for **absolute beginners** with no prior cloud or Azure experience. By the end of this comprehensive guide, you will:
- Understand what cloud computing is and why companies use it
- Know the fundamental concepts of Microsoft Azure
- Be prepared to start your AZ-104 (Azure Administrator) certification journey
- Have a solid foundation for Azure Architect interviews

---

# PART 1: UNDERSTANDING CLOUD COMPUTING FROM SCRATCH

## 1.1 What is a Server? (The Very Basics)

### Before We Talk About Cloud, Let's Understand Traditional Computing

Imagine you want to run a website for your business. What do you need?

**Traditional Way (Before Cloud):**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  WHAT YOUR COMPANY HAD TO DO:                                                │
│                                                                              │
│  1. BUY HARDWARE:                                                            │
│     ├── Server computers: $10,000 - $100,000 each                           │
│     ├── Storage devices: $5,000 - $50,000                                   │
│     ├── Network equipment (routers, switches): $2,000 - $20,000             │
│     └── Total initial cost: $50,000 - $500,000+                             │
│                                                                              │
│  2. SET UP A DATA CENTER (Room to keep servers):                            │
│     ├── Air conditioning (servers get hot!)                                 │
│     ├── Power supply with backup generators                                  │
│     ├── Physical security (locks, cameras)                                   │
│     └── Internet connectivity                                                │
│                                                                              │
│  3. HIRE IT STAFF:                                                           │
│     ├── Server administrators                                                │
│     ├── Network engineers                                                    │
│     ├── Security specialists                                                 │
│     └── 24/7 support team                                                    │
│                                                                              │
│  4. MAINTAIN EVERYTHING:                                                     │
│     ├── Replace broken hardware                                              │
│     ├── Update software and security patches                                │
│     ├── Handle backups                                                       │
│     └── Plan for disasters (fire, flood, etc.)                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**The Problems with This Approach:**
1. **High upfront costs** - You need to buy everything before your business even starts
2. **Guessing capacity** - What if you buy 10 servers but only need 3? Or need 20?
3. **Slow to scale** - Ordering new hardware takes weeks or months
4. **Maintenance burden** - Your team spends time fixing hardware instead of building features
5. **Disaster risk** - If your server room floods, you lose everything

---

## 1.2 What is Cloud Computing? (Simple Definition)

### The Cloud = Someone Else's Computers That You Rent

**Cloud Computing means:**
> Using computers, storage, and services owned by another company (like Microsoft) through the internet, instead of buying and managing your own.

**Simple Analogy - Electricity:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  BEFORE: Generate Your Own Electricity                                       │
│  ├── Buy a generator for your house                                         │
│  ├── Buy fuel regularly                                                      │
│  ├── Maintain and repair it yourself                                        │
│  └── If it breaks, you have no power                                        │
│                                                                              │
│  AFTER: Use the Power Company (This is like Cloud!)                         │
│  ├── Plug into the wall (connect to the internet)                           │
│  ├── Use as much or as little as you need                                   │
│  ├── Pay only for what you use                                              │
│  ├── The power company handles generation, maintenance, repairs             │
│  └── They have backup systems so power rarely goes out                      │
│                                                                              │
│  CLOUD COMPUTING IS THE SAME IDEA FOR COMPUTERS!                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1.3 Why Do Companies Use Cloud Computing?

### The 7 Key Benefits Every Beginner Should Know

| Benefit | What It Means (Simple) | Real Example |
|---------|----------------------|--------------|
| **1. No Upfront Cost** | Don't buy expensive hardware. Pay monthly like Netflix | Startup launches with $0 hardware investment |
| **2. Pay-As-You-Go** | Pay only for what you actually use | Like paying for electricity by the hour |
| **3. Scale Up/Down Instantly** | Need more power? Get it in minutes. Need less? Turn it off | E-commerce site handles Black Friday traffic |
| **4. Global Reach** | Put your app in data centers worldwide in minutes | App available in US, Europe, Asia simultaneously |
| **5. High Availability** | Microsoft guarantees 99.99% uptime | Your app stays online even if some servers fail |
| **6. Security** | Microsoft invests billions in security | Bank-level security without bank-level budget |
| **7. Focus on Business** | No time wasted on hardware maintenance | Developers write code, not fix servers |

### Real Story: Netflix's Journey to Cloud
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  NETFLIX BEFORE CLOUD (2008):                                                │
│  ├── Owned data centers with thousands of servers                           │
│  ├── Server failure = millions of customers couldn't watch movies           │
│  ├── Adding new servers took weeks                                          │
│  └── Scaling for new movie releases was a nightmare                         │
│                                                                              │
│  NETFLIX AFTER MOVING TO CLOUD (AWS):                                        │
│  ├── No more hardware worries                                                │
│  ├── Auto-scales from 10 million to 200 million viewers instantly           │
│  ├── Runs in 190+ countries from cloud data centers                         │
│  └── Downtime? Almost never happens now                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1.4 The Three Types of Cloud Services (IaaS, PaaS, SaaS)

### Think of It Like Pizza 🍕

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        THE PIZZA ANALOGY                                     │
│                                                                              │
│  ┌───────────────┬───────────────┬───────────────┬───────────────┐          │
│  │  TRADITIONAL  │     IaaS      │     PaaS      │     SaaS      │          │
│  │  (On-Premises)│               │               │               │          │
│  ├───────────────┼───────────────┼───────────────┼───────────────┤          │
│  │  Make pizza   │ Buy dough,    │ Order pizza   │ Go to pizza   │          │
│  │  at home      │ toppings.     │ kit. Add your │ restaurant    │          │
│  │  from scratch │ Use someone's │ toppings,     │ and eat.      │          │
│  │               │ oven.         │ bake at home. │               │          │
│  ├───────────────┼───────────────┼───────────────┼───────────────┤          │
│  │  YOU manage:  │ YOU manage:   │ YOU manage:   │ YOU manage:   │          │
│  │  Everything   │ Data &        │ Data &        │ Nothing       │          │
│  │               │ Applications  │ Applications  │ (just use it) │          │
│  └───────────────┴───────────────┴───────────────┴───────────────┘          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### IaaS (Infrastructure as a Service)
**What it is:** You rent the hardware (servers, storage, networks) but manage everything else

**Azure Examples:**
- **Azure Virtual Machines** - Rent a computer in the cloud
- **Azure Storage** - Rent hard drive space
- **Azure Virtual Network** - Rent network infrastructure

**When to use:** 
- Migrating existing applications that need specific server configurations
- Running custom software that needs specific OS settings
- Full control is required

**Real Example:**
> "Our company has a legacy banking application that only runs on Windows Server 2016 with specific software. We moved it to an Azure VM without changing any code."

### PaaS (Platform as a Service)
**What it is:** You just write code. Microsoft handles everything else (servers, OS, updates, scaling)

**Azure Examples:**
- **Azure App Service** - Just upload your website/API code
- **Azure SQL Database** - Just use the database, don't worry about server management
- **Azure Functions** - Just write small pieces of code that run when triggered

**When to use:**
- Building new applications
- When you want to focus on code, not infrastructure
- When you need auto-scaling without managing servers

**Real Example:**
> "We built our mobile app backend using Azure App Service. It auto-scales from 2 to 50 instances during peak hours. We've never touched a server."

### SaaS (Software as a Service)
**What it is:** Just use the software through your browser. Everything is managed for you.

**Examples:**
- **Microsoft 365** - Email, Word, Excel in the cloud
- **Salesforce** - Customer management system
- **Dropbox** - File storage

**When to use:**
- Standard business applications (email, documents, CRM)
- When you don't need customization

**Real Example:**
> "Our company uses Microsoft 365 for all email and documents. 500 employees, zero servers to manage."

---

## 1.5 Public, Private, and Hybrid Cloud

### The Three Deployment Models

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  PUBLIC CLOUD                                                                │
│  ────────────                                                                │
│  Resources shared among multiple customers (but securely isolated)          │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │          MICROSOFT'S DATA CENTER                                  │       │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐              │       │
│  │  │ Your    │  │ Company │  │ Company │  │ Company │              │       │
│  │  │ Company │  │ B's     │  │ C's     │  │ D's     │              │       │
│  │  │ Data    │  │ Data    │  │ Data    │  │ Data    │              │       │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘              │       │
│  │  (All isolated and secure, but sharing the same building)        │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  Best for: Most companies, cost-effective, scalable                         │
│  Examples: Azure, AWS, Google Cloud                                         │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PRIVATE CLOUD                                                               │
│  ─────────────                                                               │
│  Dedicated infrastructure for one organization only                          │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │          YOUR COMPANY'S PRIVATE DATA CENTER                       │       │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐              │       │
│  │  │ Your    │  │ Your    │  │ Your    │  │ Your    │              │       │
│  │  │ HR      │  │ Finance │  │ Product │  │ Secret  │              │       │
│  │  │ Data    │  │ Data    │  │ Data    │  │ Projects│              │       │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘              │       │
│  │  (Only your company has access to this infrastructure)           │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  Best for: Banks, government, healthcare with strict regulations            │
│  Examples: Azure Stack, VMware on-premises                                  │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  HYBRID CLOUD                                                                │
│  ────────────                                                                │
│  Combination of public and private cloud                                     │
│                                                                              │
│  ┌───────────────────────────┐    ┌─────────────────────────────────┐       │
│  │  YOUR PRIVATE DATA CENTER │◄──►│ MICROSOFT AZURE (PUBLIC CLOUD)  │       │
│  │  ┌─────────┐ ┌─────────┐  │    │ ┌─────────┐ ┌─────────┐        │       │
│  │  │Sensitive│ │Patient  │  │    │ │ Web     │ │ AI/ML   │        │       │
│  │  │Financial│ │Records  │  │    │ │ Apps    │ │Workloads│        │       │
│  │  │ Data    │ │ (HIPAA) │  │    │ │         │ │         │        │       │
│  │  └─────────┘ └─────────┘  │    │ └─────────┘ └─────────┘        │       │
│  └───────────────────────────┘    └─────────────────────────────────┘       │
│                                                                              │
│  Best for: Large enterprises transitioning to cloud, compliance needs       │
│  Real Example: Hospital keeps patient records on-premises but uses          │
│                Azure for AI-powered diagnostics                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 2: INTRODUCTION TO MICROSOFT AZURE

## 2.1 What is Microsoft Azure?

### Simple Definition
> **Microsoft Azure** is Microsoft's cloud computing platform that provides hundreds of services to help you build, run, and manage applications.

### Azure in Numbers (Why It Matters)
```
┌─────────────────────────────────────────────────────────────────────────────┐
│  MICROSOFT AZURE AT A GLANCE:                                                │
│                                                                              │
│  ├── 60+ Regions worldwide (data centers on every continent)                │
│  ├── 200+ Products and services                                             │
│  ├── 95% of Fortune 500 companies use Azure                                 │
│  ├── $100+ billion annual revenue (growing 25%+ yearly)                     │
│  ├── 10,000+ employees in security alone                                    │
│  └── #2 cloud provider globally (after AWS, ahead of Google)               │
│                                                                              │
│  WHY THIS MATTERS FOR YOUR CAREER:                                          │
│  ├── Azure skills are in HIGH DEMAND                                        │
│  ├── Azure admin salaries: $80,000 - $150,000+ (varies by location)        │
│  ├── Azure architect salaries: $120,000 - $200,000+                        │
│  └── Job growth: Cloud roles growing 3x faster than IT overall             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.2 Azure Global Infrastructure (Where Your Data Lives)

### Understanding Azure Regions and Availability Zones

**What is an Azure Region?**
> A region is a geographical area containing one or more data centers. Example: "East US" is a region in Virginia, USA.

**What is an Availability Zone?**
> Within each region, there are usually 3 physically separate data centers called Availability Zones. If one zone fails, the others keep running.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE GLOBAL INFRASTRUCTURE                          │
│                                                                              │
│  Example: "East US 2" Region (Located in Virginia, USA)                     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │                        EAST US 2 REGION                             │     │
│  │                                                                     │     │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │     │
│  │  │ Availability │  │ Availability │  │ Availability │              │     │
│  │  │   Zone 1     │  │   Zone 2     │  │   Zone 3     │              │     │
│  │  │              │  │              │  │              │              │     │
│  │  │  Data Center │  │  Data Center │  │  Data Center │              │     │
│  │  │  (Building 1)│  │  (Building 2)│  │  (Building 3)│              │     │
│  │  │              │  │              │  │              │              │     │
│  │  │  • Separate  │  │  • Separate  │  │  • Separate  │              │     │
│  │  │    power     │  │    power     │  │    power     │              │     │
│  │  │  • Separate  │  │  • Separate  │  │  • Separate  │              │     │
│  │  │    cooling   │  │    cooling   │  │    cooling   │              │     │
│  │  │  • Separate  │  │  • Separate  │  │  • Separate  │              │     │
│  │  │    network   │  │    network   │  │    network   │              │     │
│  │  └──────────────┘  └──────────────┘  └──────────────┘              │     │
│  │         │                  │                  │                     │     │
│  │         └──────────────────┼──────────────────┘                     │     │
│  │                            │                                        │     │
│  │              High-speed fiber optic connections                    │     │
│  │              (Less than 2ms latency between zones)                 │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  WHY ZONES MATTER:                                                          │
│  ├── Zone 1 has fire? Your app keeps running in Zones 2 and 3              │
│  ├── Zone 2 loses power? Zones 1 and 3 handle the load                     │
│  └── Result: 99.99% uptime (less than 1 hour downtime per year)            │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Region Pairs
Microsoft pairs regions for disaster recovery. If one entire region fails, the paired region is ready.

| Region | Paired Region | Location |
|--------|--------------|----------|
| East US | West US | Virginia ↔ California |
| North Europe | West Europe | Ireland ↔ Netherlands |
| Southeast Asia | East Asia | Singapore ↔ Hong Kong |

---

## 2.3 Azure Account Hierarchy (How Azure is Organized)

### The 4-Level Structure You MUST Understand

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     AZURE ACCOUNT HIERARCHY                                  │
│                                                                              │
│  Level 1: AZURE ACCOUNT (Your Login)                                        │
│  ───────────────────────────────────                                        │
│  This is YOU. Your email/password that logs into Azure.                     │
│  Example: john.smith@company.com                                            │
│                                                                              │
│                          │                                                   │
│                          ▼                                                   │
│  Level 2: AZURE ACTIVE DIRECTORY TENANT (Your Organization)                │
│  ───────────────────────────────────────────────────────────                │
│  This is your COMPANY's Azure identity system.                              │
│  Contains: All users, groups, applications, permissions                     │
│  Example: "Contoso Corporation Tenant"                                      │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │  Azure AD Tenant: contoso.onmicrosoft.com                        │       │
│  │  ├── Users: 500 employees                                         │       │
│  │  ├── Groups: IT Team, Finance, HR, etc.                          │       │
│  │  └── Enterprise Apps: Salesforce, SAP, custom apps              │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                          │                                                   │
│                          ▼                                                   │
│  Level 3: MANAGEMENT GROUPS (Optional Organization Layer)                   │
│  ────────────────────────────────────────────────────────                   │
│  Used by large companies to organize subscriptions                          │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │  Root Management Group                                            │       │
│  │  ├── Production MG                                                │       │
│  │  │   ├── Subscription: Prod-Web                                   │       │
│  │  │   └── Subscription: Prod-Database                              │       │
│  │  └── Development MG                                               │       │
│  │      ├── Subscription: Dev-Team1                                  │       │
│  │      └── Subscription: Dev-Team2                                  │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                          │                                                   │
│                          ▼                                                   │
│  Level 4: SUBSCRIPTIONS (Billing & Access Boundary)                         │
│  ──────────────────────────────────────────────────                         │
│  This is your BILLING ACCOUNT. All resources here are billed together.     │
│  Most important level for day-to-day administration.                        │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │  Subscription: "Production-Subscription"                         │       │
│  │  ID: 12345678-1234-1234-1234-123456789abc                        │       │
│  │  Monthly Bill: $5,000                                             │       │
│  │  Owner: IT Department                                             │       │
│  │                                                                   │       │
│  │  Contains Resource Groups (containers for resources):           │       │
│  │  ├── ResourceGroup: "WebApp-RG"                                  │       │
│  │  │   ├── App Service                                             │       │
│  │  │   └── SQL Database                                            │       │
│  │  └── ResourceGroup: "Networking-RG"                              │       │
│  │      ├── Virtual Network                                         │       │
│  │      └── Load Balancer                                           │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### What is a Resource Group? (Very Important!)

**Simple Definition:**
> A Resource Group is a **container** that holds related Azure resources. Think of it like a folder on your computer.

**Rules About Resource Groups:**
1. Every resource MUST be in exactly one resource group
2. Resource groups can span regions
3. When you delete a resource group, ALL resources inside are deleted
4. Resources can be moved between resource groups
5. Resource groups are FREE (no charge)

**Best Practice Naming:**
```
Company-Environment-Purpose-ResourceType

Examples:
├── Contoso-Prod-WebApp-RG
├── Contoso-Dev-Database-RG
├── Contoso-Test-API-RG
└── Contoso-Shared-Networking-RG
```

---

## 2.4 Azure Services Overview (The Main Categories)

### The Core Service Categories

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      AZURE SERVICES - MAIN CATEGORIES                       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 💻 COMPUTE (Run your code)                                          │    │
│  │    Virtual Machines - Rent servers                                  │    │
│  │    App Service - Host web apps (PaaS)                               │    │
│  │    Azure Functions - Serverless code execution                      │    │
│  │    Azure Kubernetes Service - Container orchestration               │    │
│  │    Virtual Machine Scale Sets - Auto-scaling VMs                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 🗄️ STORAGE (Store your data)                                        │    │
│  │    Blob Storage - Object storage (files, images, videos)            │    │
│  │    File Storage - Shared network drives                             │    │
│  │    Queue Storage - Message queuing                                  │    │
│  │    Table Storage - NoSQL key-value store                            │    │
│  │    Disk Storage - Hard drives for VMs                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 🌐 NETWORKING (Connect everything)                                  │    │
│  │    Virtual Network - Private network in Azure                       │    │
│  │    Load Balancer - Distribute traffic                               │    │
│  │    VPN Gateway - Connect to on-premises                             │    │
│  │    Application Gateway - Web traffic load balancer                  │    │
│  │    Azure DNS - Domain name management                               │    │
│  │    ExpressRoute - Private connection to Azure                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 🗃️ DATABASES (Store structured data)                                │    │
│  │    Azure SQL Database - Managed SQL Server                          │    │
│  │    Cosmos DB - Globally distributed NoSQL                           │    │
│  │    Azure Database for MySQL/PostgreSQL                              │    │
│  │    Azure Cache for Redis - In-memory cache                          │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 🔐 IDENTITY & SECURITY (Protect everything)                         │    │
│  │    Azure Active Directory - Identity management                     │    │
│  │    Azure Key Vault - Store secrets and keys                         │    │
│  │    Azure Security Center - Security monitoring                      │    │
│  │    Azure Sentinel - SIEM (Security Information & Event Management) │    │
│  │    Azure DDoS Protection - Protect from attacks                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 📊 MONITORING & MANAGEMENT (Watch and control)                      │    │
│  │    Azure Monitor - Collect and analyze telemetry                    │    │
│  │    Log Analytics - Query and analyze logs                           │    │
│  │    Azure Policy - Enforce rules                                     │    │
│  │    Azure Cost Management - Track spending                           │    │
│  │    Azure Automation - Automate tasks                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2.5 How to Access Azure (Four Ways)

### 1. Azure Portal (Web Interface)
- **URL:** https://portal.azure.com
- **Best for:** Visual management, beginners, occasional tasks
- **Pros:** Easy to use, no installation required
- **Cons:** Slow for repetitive tasks, can't automate

### 2. Azure CLI (Command Line - Any OS)
```bash
# Install on Windows (run in PowerShell as Admin)
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'

# Login to Azure
az login

# List all your subscriptions
az account list --output table

# Create a resource group
az group create --name MyResourceGroup --location eastus

# List all resource groups
az group list --output table
```

### 3. Azure PowerShell
```powershell
# Install Azure PowerShell module
Install-Module -Name Az -AllowClobber -Scope CurrentUser

# Login to Azure
Connect-AzAccount

# List all subscriptions
Get-AzSubscription

# Create a resource group
New-AzResourceGroup -Name "MyResourceGroup" -Location "EastUS"

# List all resource groups
Get-AzResourceGroup | Format-Table
```

### 4. Azure Cloud Shell (Browser-based)
- Access from portal by clicking the terminal icon
- Pre-installed CLI and PowerShell
- No local installation needed
- Free 5GB storage included

---

## 2.6 Azure Resource Manager (ARM) - The Brain of Azure

### What is ARM?
> Azure Resource Manager (ARM) is the deployment and management service for Azure. Every action you take in Azure goes through ARM.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    HOW ARM WORKS (SIMPLIFIED)                                │
│                                                                              │
│   YOU (through Portal, CLI, PowerShell, or REST API)                        │
│                           │                                                  │
│                           ▼                                                  │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                  AZURE RESOURCE MANAGER                              │   │
│   │                                                                      │   │
│   │  1. Authenticates you (checks your identity)                        │   │
│   │  2. Authorizes you (checks your permissions)                        │   │
│   │  3. Validates your request                                          │   │
│   │  4. Sends request to the right service                             │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                           │                                                  │
│           ┌───────────────┼───────────────┐                                 │
│           ▼               ▼               ▼                                 │
│   ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                          │
│   │ Compute     │ │ Storage     │ │ Networking  │                          │
│   │ Service     │ │ Service     │ │ Service     │                          │
│   │ (creates VM)│ │(creates SA) │ │(creates VNet│                          │
│   └─────────────┘ └─────────────┘ └─────────────┘                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### ARM Templates (Infrastructure as Code)

ARM Templates let you define your infrastructure in JSON files. This means:
- **Repeatable:** Deploy the same environment 100 times identically
- **Version controlled:** Store in Git, track changes
- **Documented:** The template IS the documentation

**Simple ARM Template Example:**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-02-01",
      "name": "mystorageaccount12345",
      "location": "eastus",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

---

# PART 3: CORE CONCEPTS FOR AZ-104 CERTIFICATION

## 3.1 Understanding Azure Identity (Azure AD)

### Why Identity Matters
> "Identity is the new security perimeter." 
> 
> In the cloud, you can't rely on physical security. Identity (who you are) determines what you can access.

### Azure AD vs Traditional Active Directory

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 WINDOWS AD vs AZURE AD COMPARISON                            │
│                                                                              │
│  ┌────────────────────────────┐  ┌────────────────────────────┐             │
│  │  WINDOWS ACTIVE DIRECTORY  │  │    AZURE ACTIVE DIRECTORY  │             │
│  │  (Traditional/On-Premises) │  │    (Cloud-Based)           │             │
│  ├────────────────────────────┤  ├────────────────────────────┤             │
│  │                            │  │                            │             │
│  │  • Runs on Windows Server  │  │  • Cloud service (SaaS)    │             │
│  │  • In your data center     │  │  • In Microsoft's cloud    │             │
│  │  • Uses Kerberos protocol  │  │  • Uses OAuth/SAML/OIDC    │             │
│  │  • For on-premises apps    │  │  • For cloud apps + on-prem│             │
│  │  • Domain Controllers      │  │  • No servers to manage    │             │
│  │  • Group Policy (GPO)      │  │  • Conditional Access      │             │
│  │  • Organizational Units    │  │  • Administrative Units    │             │
│  │                            │  │                            │             │
│  └────────────────────────────┘  └────────────────────────────┘             │
│                                                                              │
│  KEY INSIGHT: They're NOT the same thing! Azure AD is not "Windows AD       │
│  in the cloud." It's a completely different identity platform designed      │
│  for cloud scenarios.                                                        │
│                                                                              │
│  THEY CAN WORK TOGETHER: Azure AD Connect syncs users from Windows AD       │
│  to Azure AD, giving you hybrid identity.                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Azure AD Objects

| Object | What It Is | Example |
|--------|-----------|---------|
| **User** | A person's identity | john@company.com |
| **Group** | Collection of users | "Marketing Team" group |
| **Service Principal** | Identity for applications/services | "WebApp-Backend" |
| **Managed Identity** | Azure-managed identity for resources | VM's identity to access Key Vault |
| **App Registration** | Defining an application in Azure AD | "Contoso Mobile App" |

---

## 3.2 Understanding Azure RBAC (Role-Based Access Control)

### The Golden Rule of RBAC
> "Who can do what on which resources?"

### RBAC Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        RBAC ASSIGNMENT FORMULA                               │
│                                                                              │
│          WHO          +        WHAT         +        WHERE                   │
│     (Security         +       (Role         +       (Scope)                  │
│      Principal)              Definition)                                     │
│                                                                              │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐            │
│  │     User        │ + │    Reader       │ + │  Subscription   │            │
│  │ john@company.com│   │ (can view only) │   │   "Prod-Sub"    │            │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘            │
│                                                                              │
│  RESULT: John can view all resources in Prod-Sub but cannot modify anything │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ANOTHER EXAMPLE:                                                            │
│                                                                              │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐            │
│  │     Group       │ + │    Virtual      │ + │ Resource Group  │            │
│  │  "VM-Admins"    │   │    Machine      │   │   "VMs-RG"      │            │
│  │                 │   │   Contributor   │   │                 │            │
│  └─────────────────┘   └─────────────────┘   └─────────────────┘            │
│                                                                              │
│  RESULT: All members of VM-Admins group can manage VMs in VMs-RG            │
│          but NOT in other resource groups                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Built-in Roles (Most Important for AZ-104)

| Role | What It Can Do | Scope |
|------|---------------|-------|
| **Owner** | Full access + can delegate access to others | Any |
| **Contributor** | Full access EXCEPT managing access | Any |
| **Reader** | View only, cannot change anything | Any |
| **User Access Administrator** | Manage user access only | Any |
| **Virtual Machine Contributor** | Manage VMs only | Usually RG |
| **Storage Blob Data Contributor** | Read/write blob data | Storage |
| **Network Contributor** | Manage networks | Usually RG |

### Scope Hierarchy (Permissions Inherit Downward)

```
Management Group (Company-wide policies)
       │
       └── Subscription (Billing boundary)
              │
              └── Resource Group (Logical container)
                     │
                     └── Resource (Individual service)

IF you give "Reader" at Subscription level:
   → User can read ALL Resource Groups in that subscription
   → User can read ALL Resources in those groups
```

---

## 3.3 Azure Virtual Networks (VNet) Basics

### What is a Virtual Network?
> A VNet is your private network in Azure. Resources inside the same VNet can talk to each other automatically.

### VNet Key Concepts for Beginners

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        VIRTUAL NETWORK CONCEPTS                              │
│                                                                              │
│  1. ADDRESS SPACE (CIDR Notation)                                           │
│  ─────────────────────────────────                                          │
│  The range of IP addresses your network can use.                            │
│                                                                              │
│  Example: 10.0.0.0/16 means:                                                │
│  ├── First 16 bits are fixed (10.0)                                         │
│  ├── Last 16 bits can vary (0.0 to 255.255)                                │
│  └── Total: 65,536 IP addresses available                                   │
│                                                                              │
│  Common CIDR blocks:                                                         │
│  ├── /8  = 16 million IPs (10.0.0.0/8)                                      │
│  ├── /16 = 65,536 IPs (10.0.0.0/16)                                         │
│  ├── /24 = 256 IPs (10.0.1.0/24)                                            │
│  └── /27 = 32 IPs (10.0.1.0/27)                                             │
│                                                                              │
│  2. SUBNETS (Dividing Your Network)                                         │
│  ───────────────────────────────────                                         │
│  Split your VNet into smaller networks for organization and security.       │
│                                                                              │
│  Example:                                                                    │
│  VNet: 10.0.0.0/16                                                          │
│  ├── Subnet: WebServers    10.0.1.0/24  (256 IPs)                          │
│  ├── Subnet: AppServers    10.0.2.0/24  (256 IPs)                          │
│  ├── Subnet: Databases     10.0.3.0/24  (256 IPs)                          │
│  └── Subnet: GatewaySubnet 10.0.255.0/27 (32 IPs)                          │
│                                                                              │
│  3. NETWORK SECURITY GROUP (NSG)                                            │
│  ────────────────────────────────                                            │
│  A firewall for your subnet or NIC. Controls what traffic is allowed.       │
│                                                                              │
│  Example NSG Rules:                                                          │
│  ├── Allow: TCP 443 from Internet (HTTPS)                                   │
│  ├── Allow: TCP 22 from My IP (SSH for admin)                               │
│  └── Deny: Everything else                                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3.4 Azure Storage Basics

### The Four Types of Azure Storage

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AZURE STORAGE TYPES                                   │
│                                                                              │
│  All stored in a STORAGE ACCOUNT (the container for storage services)       │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 1. BLOB STORAGE (Binary Large OBjects)                              │    │
│  │    ───────────────────────────────────                              │    │
│  │    • For unstructured data: files, images, videos, backups         │    │
│  │    • Like Dropbox or Google Drive, but for applications            │    │
│  │    • Three tiers: Hot (frequent), Cool (infrequent), Archive (rare)│    │
│  │                                                                     │    │
│  │    Real example: "Store all user uploaded profile pictures"        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 2. FILE STORAGE (Azure Files)                                       │    │
│  │    ───────────────────────────                                      │    │
│  │    • Network file shares (SMB and NFS protocols)                   │    │
│  │    • Mount as a drive letter in Windows: Z:\SharedFiles            │    │
│  │    • Replace on-premises file servers                              │    │
│  │                                                                     │    │
│  │    Real example: "Share documents across 100 VMs"                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 3. QUEUE STORAGE                                                    │    │
│  │    ─────────────────                                                │    │
│  │    • Store messages that apps read later                           │    │
│  │    • For decoupling application components                         │    │
│  │    • Messages up to 64KB each                                      │    │
│  │                                                                     │    │
│  │    Real example: "Order service puts order, inventory reads later" │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ 4. TABLE STORAGE                                                    │    │
│  │    ────────────────                                                 │    │
│  │    • NoSQL key-value store                                         │    │
│  │    • Simple structured data without complex relationships          │    │
│  │    • Very cheap, very scalable                                     │    │
│  │                                                                     │    │
│  │    Real example: "Store IoT sensor readings from 1 million devices"│    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Storage Account Redundancy Options

| Option | Description | Use Case | Durability |
|--------|-------------|----------|------------|
| **LRS** | 3 copies in ONE data center | Dev/Test, non-critical | 99.999999999% |
| **ZRS** | 3 copies across 3 zones | Production, single region | 99.9999999999% |
| **GRS** | LRS + copy to paired region | Disaster recovery | 99.99999999999999% |
| **GZRS** | ZRS + copy to paired region | Mission critical | Highest |

---

## 3.5 Azure Virtual Machines Basics

### What You Need to Create a VM

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VM CREATION CHECKLIST                                   │
│                                                                              │
│  1. BASICS                                                                   │
│     ├── Subscription (which bill to charge)                                 │
│     ├── Resource Group (where to put it)                                    │
│     ├── VM Name (e.g., "WebServer01")                                       │
│     ├── Region (e.g., "East US")                                            │
│     └── Availability option (Availability Set, Zone, or none)              │
│                                                                              │
│  2. IMAGE (Operating System)                                                 │
│     ├── Windows Server 2022                                                  │
│     ├── Ubuntu Server 22.04                                                  │
│     ├── Red Hat Enterprise Linux                                             │
│     └── Custom image (your pre-configured OS)                               │
│                                                                              │
│  3. SIZE (How powerful?)                                                     │
│     ├── B1s: 1 vCPU, 1GB RAM ($7/month) - tiny, testing only               │
│     ├── D2s_v3: 2 vCPU, 8GB RAM ($70/month) - small production             │
│     ├── D4s_v3: 4 vCPU, 16GB RAM ($140/month) - medium production          │
│     └── Choose based on workload needs                                       │
│                                                                              │
│  4. ADMINISTRATOR ACCOUNT                                                    │
│     ├── Username (NOT "admin" or "administrator")                           │
│     └── Password or SSH key (SSH recommended for Linux)                     │
│                                                                              │
│  5. DISKS                                                                    │
│     ├── OS Disk: Where the OS is installed (required)                       │
│     └── Data Disks: Additional storage (optional)                           │
│                                                                              │
│  6. NETWORKING                                                               │
│     ├── Virtual Network: Which VNet to put it in                            │
│     ├── Subnet: Which subnet                                                 │
│     ├── Public IP: If you need internet access to VM                        │
│     └── NSG: Firewall rules                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 4: HANDS-ON GETTING STARTED

## 4.1 Creating Your First Azure Account (Free!)

### Step-by-Step Guide
1. Go to https://azure.microsoft.com/free
2. Click "Start free"
3. Sign in with Microsoft account (or create one)
4. Provide phone verification
5. Provide credit card (for verification only - won't charge for free tier)
6. Accept terms

### What You Get FREE:
- **$200 credit** for first 30 days
- **12 months** of popular free services
- **Always free** services (25+ services)

### Always Free Services Include:
| Service | Free Amount |
|---------|------------|
| Azure Functions | 1 million executions/month |
| Azure Cosmos DB | 25 GB storage |
| Azure DevOps | 5 users |
| Azure App Service | 10 web apps |
| Azure Active Directory | 50,000 objects |

---

## 4.2 Your First Resource: Create a Resource Group

### Using Azure Portal
1. Go to https://portal.azure.com
2. Click "Resource groups" in the left menu (or search for it)
3. Click "+ Create"
4. Select your subscription
5. Name it: "MyFirst-ResourceGroup"
6. Select region: "East US"
7. Click "Review + Create" then "Create"

### Using Azure CLI
```bash
# Login first
az login

# Create the resource group
az group create --name "MyFirst-ResourceGroup" --location "eastus"
```

### Using PowerShell
```powershell
# Login first
Connect-AzAccount

# Create the resource group
New-AzResourceGroup -Name "MyFirst-ResourceGroup" -Location "EastUS"
```

---

## 4.3 Your First Virtual Machine

### Step-by-Step Portal Guide

1. **Go to Virtual Machines** → Click "+ Create" → "Azure virtual machine"

2. **Basics tab:**
   - Subscription: Your subscription
   - Resource group: MyFirst-ResourceGroup
   - VM name: MyFirstVM
   - Region: East US
   - Image: Ubuntu Server 22.04 LTS
   - Size: B1s (free tier eligible)
   - Username: azureadmin
   - SSH public key: Generate new

3. **Disks tab:** Keep defaults

4. **Networking tab:** 
   - VNet: Will auto-create
   - Public IP: Will auto-create
   - NIC NSG: Basic
   - Public inbound ports: Allow SSH (22)

5. **Review + Create** → Click **Create**

6. **Download the private key** when prompted

### Connect to Your VM
```bash
# Change permissions on key file
chmod 400 ~/Downloads/MyFirstVM_key.pem

# Connect via SSH
ssh -i ~/Downloads/MyFirstVM_key.pem azureadmin@<YOUR-PUBLIC-IP>
```

---

# PART 5: WHAT'S NEXT - AZ-104 EXAM ROADMAP

## 5.1 AZ-104 Exam Overview

### Exam Details
| Attribute | Details |
|-----------|---------|
| Full Name | Microsoft Azure Administrator |
| Exam Code | AZ-104 |
| Questions | 40-60 questions |
| Duration | 100 minutes |
| Passing Score | 700/1000 |
| Cost | $165 USD |
| Format | Multiple choice, case studies, drag-drop |

### Exam Domains (What to Study)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AZ-104 EXAM DOMAINS & WEIGHTS                             │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────┐                │
│  │ 1. Manage Azure Identities & Governance (20-25%)        │ ████████░░░    │
│  │    • Azure AD users, groups, authentication             │                │
│  │    • RBAC roles and assignments                         │                │
│  │    • Azure subscriptions and governance                 │                │
│  │    • Azure Policy                                        │                │
│  └─────────────────────────────────────────────────────────┘                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────┐                │
│  │ 2. Implement & Manage Storage (15-20%)                  │ ██████░░░░░    │
│  │    • Storage accounts                                    │                │
│  │    • Blob storage, files, queues, tables                │                │
│  │    • Storage security (SAS, access keys)                │                │
│  │    • Azure File Sync                                     │                │
│  └─────────────────────────────────────────────────────────┘                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────┐                │
│  │ 3. Deploy & Manage Azure Compute (20-25%)               │ ████████░░░    │
│  │    • Virtual Machines (create, configure, scale)        │                │
│  │    • VM availability (scale sets, availability sets)   │                │
│  │    • Azure App Service                                   │                │
│  │    • Azure Container Instances                          │                │
│  └─────────────────────────────────────────────────────────┘                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────┐                │
│  │ 4. Implement & Manage Virtual Networking (20-25%)       │ ████████░░░    │
│  │    • Virtual Networks and subnets                       │                │
│  │    • Network Security Groups                            │                │
│  │    • Azure Load Balancer                                │                │
│  │    • VNet peering, VPN Gateway                          │                │
│  │    • Azure DNS                                           │                │
│  └─────────────────────────────────────────────────────────┘                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────┐                │
│  │ 5. Monitor & Maintain Azure Resources (10-15%)          │ █████░░░░░░    │
│  │    • Azure Monitor                                       │                │
│  │    • Log Analytics                                       │                │
│  │    • Alerts and action groups                           │                │
│  │    • Azure Backup                                        │                │
│  └─────────────────────────────────────────────────────────┘                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5.2 Study Order Recommendation

### Week 1-2: Foundation
- ✅ Complete this Module 00
- ✅ Create free Azure account
- ✅ Practice in Azure Portal
- ✅ Learn Azure CLI basics

### Week 3-4: Identity & Governance
- Study Module 08: Authentication & Authorization
- Study Module 09: Azure Active Directory
- Hands-on: Create users, groups, assign RBAC roles

### Week 5-6: Storage
- Study Module 02: ARM & Azure Storage
- Study Module 03: Azure Storage (advanced)
- Hands-on: Create storage accounts, upload blobs, configure SAS

### Week 7-8: Compute
- Study Module 04: Azure Virtual Machines
- Study Module 05: App & Container Services
- Hands-on: Create VMs, scale sets, App Service

### Week 9-10: Networking
- Study Module 06: Azure Networking Part I
- Study Module 07: Azure Networking Part II
- Hands-on: Create VNets, NSGs, peering, load balancers

### Week 11-12: Monitoring & Review
- Study Module 10: Azure Monitoring
- Take practice exams
- Review weak areas
- Schedule exam

---

## 5.3 Essential Terms Glossary

| Term | Simple Definition |
|------|------------------|
| **ARM** | Azure Resource Manager - the control plane for all Azure operations |
| **Availability Set** | Place VMs in different fault/update domains within a data center |
| **Availability Zone** | Physically separate data centers within a region |
| **Blob** | Binary Large Object - any file stored in Azure Storage |
| **CIDR** | Notation for IP address ranges (e.g., 10.0.0.0/16) |
| **NSG** | Network Security Group - a firewall for Azure resources |
| **RBAC** | Role-Based Access Control - who can do what where |
| **SAS** | Shared Access Signature - temporary access token for storage |
| **SKU** | Stock Keeping Unit - defines pricing tier and capabilities |
| **Subnet** | A subdivision of a Virtual Network |
| **Tenant** | An Azure AD instance representing your organization |
| **VNet** | Virtual Network - your private network in Azure |
| **VNet Peering** | Connect two VNets so resources can communicate |
| **VPN Gateway** | Connect Azure VNet to on-premises network securely |

---

# PART 6: COMMON INTERVIEW QUESTIONS (BEGINNER LEVEL)

## 6.1 Basic Conceptual Questions

**Q1: What is the difference between IaaS, PaaS, and SaaS?**
> **Answer:** IaaS gives you infrastructure (VMs, storage) - you manage OS and up. PaaS gives you a platform (App Service) - you only manage code. SaaS gives you software (Microsoft 365) - you just use it. Use IaaS for lift-and-shift migrations, PaaS for new development, SaaS for standard business apps.

**Q2: What is an Azure Region?**
> **Answer:** A region is a geographical area containing one or more data centers. Azure has 60+ regions globally. Each region has low-latency network connectivity between its data centers. You choose regions based on data residency requirements, proximity to users, and service availability.

**Q3: What is a Resource Group?**
> **Answer:** A resource group is a logical container for Azure resources. All resources must belong to exactly one resource group. Resource groups help organize resources by lifecycle, project, or application. Deleting a resource group deletes all resources inside it.

**Q4: What is the difference between Windows AD and Azure AD?**
> **Answer:** Windows AD is an on-premises directory service using Kerberos, primarily for domain-joined computers. Azure AD is a cloud-based identity service using modern protocols (OAuth, SAML, OIDC), designed for web applications and cloud resources. They can work together using Azure AD Connect for hybrid identity.

**Q5: What is RBAC and why is it important?**
> **Answer:** RBAC (Role-Based Access Control) answers "Who can do what on which resources." It's important because it implements least privilege - users only get the minimum permissions needed. RBAC has three components: security principal (who), role definition (what), and scope (where).

---

## 6.2 Scenario-Based Questions

**Q: Your company wants to host a website that needs to handle traffic spikes during holidays. What Azure service would you recommend?**
> **Answer:** Azure App Service with autoscaling, or a VM Scale Set if we need more control. App Service is PaaS so we focus on code while Azure handles scaling automatically. We can configure autoscale rules based on CPU, memory, or request count.

**Q: A developer needs to be able to create and manage VMs in the development resource group only. What do you do?**
> **Answer:** Assign the "Virtual Machine Contributor" role to the developer, scoped to the development resource group. This follows least privilege - they can manage VMs but nothing else, and only in that specific resource group.

**Q: How would you ensure a VM in Azure stays running even if a hardware failure occurs?**
> **Answer:** Deploy the VM in an Availability Zone for protection against entire data center failures, or use an Availability Set for protection against hardware/update failures within a data center. For even higher availability, deploy multiple VMs behind a Load Balancer across multiple zones.

---

# Summary: Key Takeaways for Beginners

1. **Cloud = Rented infrastructure** - Pay for what you use, scale instantly
2. **Azure = Microsoft's cloud** - 200+ services, global presence
3. **Everything goes through ARM** - Azure Resource Manager is the control plane
4. **Resources live in Resource Groups** - Organize by lifecycle or application
5. **Identity is key** - Azure AD manages who you are and what you can do
6. **RBAC controls access** - Who + What role + Where = Access
7. **Networking is foundational** - VNets, subnets, NSGs protect your resources
8. **Storage has many options** - Blob for files, Files for shares, Table for NoSQL

**Next Steps:**
1. Create your free Azure account
2. Complete hands-on labs in this module
3. Move to Module 01 for deeper cloud computing concepts
4. Follow the study roadmap for AZ-104 certification

---

*This guide was created to give complete beginners a solid foundation for Microsoft Azure. For detailed coverage of each topic, continue with Modules 01-26 in this training series.*
