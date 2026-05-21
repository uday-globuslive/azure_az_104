# Module 15 - Azure Architecture Design & Interview Preparation
## (Event-Driven Architecture, Microservices, CI/CD, Security, Full Interview Answers)

> **Beginner Note**: This module ties everything together. We'll walk through a real-world architecture from scratch, explaining every decision along the way. By the end, you'll be able to confidently answer the deep technical interview question about designing Azure architecture.

---

## Learning Objectives
By the end of this module, you will be able to:
- Understand event-driven and microservices architecture
- Explain design decisions in a real-world Azure architecture
- Describe CI/CD pipelines using Azure DevOps and Terraform
- Implement security with Managed Identity and Key Vault
- Answer the deep technical screening question confidently
- Explain complex architectures to both technical and non-technical audiences

---

## 15.1 The Problem We're Solving

### Business Problem (From Interview)

```
THE SITUATION:
A company has a MONOLITHIC application built 10 years ago.

What is a Monolith?
┌─────────────────────────────────────────────────────────┐
│               ONE BIG APPLICATION                        │
│                                                          │
│   User Interface  +  Business Logic  +  Database        │
│          +  Reporting  +  Notifications  +  Payments    │
│                  ALL IN ONE PLACE                        │
│                                                          │
│   Running on: 5 on-premises servers                      │
└─────────────────────────────────────────────────────────┘

THE PROBLEMS:
1. SCALING: One part busy? You must scale the WHOLE app
2. COUPLING: Change payment logic? You might break notifications
3. DEPLOYMENT: Update one feature? Deploy EVERYTHING
4. RELIABILITY: One bug? The entire app goes down
5. TECHNOLOGY: Stuck with one programming language/framework
6. COST: Servers running 24/7 even when traffic is low
```

### The Solution - Microservices + Event-Driven

```
AFTER TRANSFORMATION:
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  User    │  │ Order    │  │ Payment  │  │Notification│
│  Service │  │ Service  │  │ Service  │  │ Service  │
│ (own DB) │  │ (own DB) │  │ (own DB) │  │ (own DB) │
└──────────┘  └──────────┘  └──────────┘  └──────────┘

Each service:
✓ Does ONE thing well
✓ Has its own database
✓ Deploys independently
✓ Scales independently
✓ Can use different technology
```

---

## 15.2 Event-Driven Architecture

### What is Event-Driven Architecture?

```
TRADITIONAL (Synchronous / Tight coupling):

OrderService ──── HTTP Call ────► PaymentService
OrderService WAITS for PaymentService to respond

Problems:
- If PaymentService is slow, OrderService is slow
- If PaymentService crashes, OrderService crashes
- High coupling between services
```

```
EVENT-DRIVEN (Loose coupling):

OrderService ──── publishes event ────► Service Bus
                  "OrderPlaced"
                                               │
                                  ┌────────────┼────────────┐
                                  ▼            ▼            ▼
                           PaymentSvc   InventorySvc  NotifySvc
                           (processes   (updates      (sends
                           payment)     stock)        email)

Benefits:
✓ OrderService doesn't know about PaymentService
✓ Services work independently
✓ Add new service = subscribe to event, no code change in OrderService
✓ Payment slow? Order still placed, payment processed later
```

### Core Event-Driven Patterns

#### Pattern 1: Message Queue (Command)

```
PRODUCER              QUEUE               CONSUMER
┌──────────┐         ┌──────────────┐    ┌──────────┐
│  Order   │─────── ►│ Service Bus  │───►│ Payment  │
│  Service │  "Process Queue         │    │ Service  │
│          │  this    (stores)       │    │          │
└──────────┘  order"  └──────────────┘    └──────────┘

Message = COMMAND ("do this thing")
One producer → One consumer
Guaranteed delivery
```

#### Pattern 2: Event Publishing (Notification)

```
PRODUCER              EVENT GRID          CONSUMERS
┌──────────┐         ┌──────────────┐    ┌──────────┐
│  Order   │─────── ►│  Event Grid  │───►│  Email   │
│  Service │  "Order  (broadcasts)  │───►│  Service │
│          │  placed"               │───►│  SMS Svc │
└──────────┘         └──────────────┘    │  Dashboard│
                                         └──────────┘

Message = EVENT ("this happened, FYI")
One producer → Many consumers
Near-real-time notification
```

#### Pattern 3: Saga Pattern (Distributed Transactions)

```
PROBLEM: How to keep data consistent across multiple services?

SAGA PATTERN (sequence of local transactions):

Step 1: OrderService creates order (status: PENDING)
         │
         ▼ publishes "OrderCreated" event
Step 2: PaymentService charges card (status: CHARGED)
         │
         ▼ publishes "PaymentSuccess" event
Step 3: InventoryService reserves stock
         │
         ▼ publishes "StockReserved" event
Step 4: OrderService updates order (status: CONFIRMED)

IF Step 3 FAILS:
Step 3: InventoryService fails → publishes "StockFailed" event
Step 2b: PaymentService refunds card (COMPENSATING TRANSACTION)
Step 1b: OrderService cancels order (status: CANCELLED)
```

---

## 15.3 The Full Architecture Design

### Architecture Overview (ATT Solutions Example)

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         Complete Azure Architecture                           │
│                                                                               │
│  EXTERNAL CLIENTS             APIM                  SERVICES                 │
│  ┌──────────────┐            ┌──────┐              ┌──────────────────────┐  │
│  │ Mobile App   │────────── ►│      │─────────────►│   AKS Cluster        │  │
│  │ Web App      │            │ API  │              │  ┌────────────────┐  │  │
│  │ Partner APIs │────────── ►│  MGT │              │  │ OrderService   │  │  │
│  └──────────────┘            │      │              │  │ UserService    │  │  │
│                              └──┬───┘              │  │ ProductService │  │  │
│                                 │                  │  │ PaymentService │  │  │
│                                 │                  │  └────────────────┘  │  │
│                                 │                  └──────────────────────┘  │
│                                 │                           │                │
│                                 ▼                           ▼                │
│                          ┌──────────────┐         ┌─────────────────────┐  │
│                          │ Azure        │         │ Azure Service Bus    │  │
│                          │ Functions    │         │ ┌─────────────────┐  │  │
│                          │ (processing) │         │ │ order-queue     │  │  │
│                          └──────┬───────┘         │ │ payment-queue   │  │  │
│                                 │                  │ │ order-events    │  │  │
│                                 ▼                  │ │  (topic)        │  │  │
│                          ┌──────────────┐          │ └─────────────────┘  │  │
│                          │ Event Grid   │◄────────►└─────────────────────┘  │
│                          └──────┬───────┘                                   │
│                                 │                                            │
│   ┌─────────────────────────────▼──────────────────────────────────────┐   │
│   │                     DATA LAYER                                      │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │   │
│   │  │Azure SQL │  │Cosmos DB │  │  Redis   │  │  Blob Storage    │   │   │
│   │  │(orders)  │  │(catalog) │  │  Cache   │  │  (files/media)   │   │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────────────────┘   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                               │
│   NETWORKING                        SECURITY                                │
│   ┌──────────────────────────┐      ┌──────────────────────────────────┐   │
│   │ Hub-Spoke VNet           │      │ Key Vault (secrets)               │   │
│   │ Private Endpoints (PaaS) │      │ Managed Identity (no passwords)   │   │
│   │ NSGs + Azure Firewall    │      │ Azure AD + Conditional Access     │   │
│   │ VPN to On-Premises       │      │ Defender for Cloud               │   │
│   └──────────────────────────┘      └──────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 15.4 Service-by-Service Deep Explanation

### Why AKS (Azure Kubernetes Service)?

```
WHAT IS KUBERNETES?
Kubernetes = Container orchestration platform
It runs your microservices in containers and manages:
- Starting/stopping containers
- Scaling (more containers when busy)
- Self-healing (restarts failed containers)
- Load balancing between containers
- Rolling deployments (zero downtime updates)

WHY AKS (AZURE VERSION)?
- Microsoft manages the Kubernetes control plane
- Integrated with Azure AD for auth
- Built-in monitoring with Azure Monitor
- Automatic node scaling
- Integration with ACR (container registry)

80 SERVICES on AKS means:
- Each microservice runs in its own pod (container group)
- Services communicate via internal Kubernetes service DNS
- External traffic comes via APIM → Ingress Controller → AKS
```

### Why Azure Container Registry (ACR)?

```
WHAT IS A CONTAINER REGISTRY?
Like Docker Hub but private and in your Azure tenant.

WORKFLOW:
Developer writes code
    │
    ▼
CI/CD pipeline (Azure DevOps)
    │
    ▼ builds Docker image
Azure Container Registry (ACR)  ← stores the image
    │
    ▼ deploys image
AKS pulls image from ACR and runs it

WHY ACR?
✓ Private (not public like Docker Hub)
✓ Integrated with Azure AD and AKS
✓ Geo-replication (images in multiple regions)
✓ Vulnerability scanning
✓ Content trust (signed images)
```

### Why Service Bus vs Event Grid (In This Architecture)?

```
Service Bus used for:
- Order processing queue (guaranteed, ordered, one worker)
- Payment processing queue (financial transactions need guarantees)
- Retry logic built in (dead-letter queue)
- Sessions for grouping related messages (all messages for order#123)

Event Grid used for:
- "Order confirmed" → notify Email AND SMS AND Dashboard simultaneously
- "Payment failed" → trigger multiple notification services
- "New product added" → notify search indexer + CDN + mobile apps
- Reacting to Azure events (blob uploaded → Function triggered)

KEY DECISION RULE:
"Process this task" → Service Bus (command, one consumer)
"This thing happened" → Event Grid (event, many notified)
```

### Why APIM (API Management)?

```
80+ microservices each with their own URL is chaos.
Clients would need to know 80 URLs.

APIM solves this:
1. ONE URL for all: https://api.mycompany.com
2. Routes internally to right microservice
3. Auth: Validates JWT token once (don't repeat in each service)
4. Rate limiting: "Only 1000 calls/minute per client app"
5. Caching: Common responses served from cache (faster)
6. Versioning: /v1/orders, /v2/orders run simultaneously
7. Developer portal: All APIs documented in one place
```

### Why Logic Apps?

```
ORCHESTRATION PROBLEM:
Some business processes involve multiple steps and services
Example: Order approval workflow
1. Order placed
2. Manager approval required (if > £500)
3. If approved: process payment
4. If rejected: notify customer, refund

This is ORCHESTRATION — coordinating multiple steps.

WHY LOGIC APPS (not code)?
- Visual workflow designer
- Non-developers can understand and modify
- Built-in connectors (Outlook for email approval)
- Built-in retry logic
- Run history for troubleshooting
- Easy to change workflow without code deployment
```

---

## 15.5 Environment Setup (Landing Zone)

### Management Group Structure Used

```
Root MG
├── Platform
│   ├── Connectivity-Sub (hub network, VPN, DNS, Firewall)
│   ├── Management-Sub (Log Analytics, Backup, Update Mgmt)
│   └── Identity-Sub (Azure AD DS, Key Vault platform)
│
└── Application Landing Zones
    ├── Production MG
    │   ├── ATT-Prod-Sub (main app workloads)
    │   └── ATT-Data-Sub (databases, storage)
    └── NonProduction MG
        └── ATT-Dev-Sub (dev + test + UAT)
```

### Why This Structure?

```
CONNECTIVITY SUBSCRIPTION — Why separate?
- Hub VNet lives here
- Managed by platform/network team only
- Other teams cannot accidentally change hub network
- One VPN gateway serves ALL workloads (cost efficient)

MANAGEMENT SUBSCRIPTION — Why separate?
- Central Log Analytics Workspace
- All subscriptions send logs here
- Security team can query ALL logs from one place
- Backup vaults, update management here

PRODUCTION vs NON-PROD separate subscriptions:
- Prod: tighter policies (no public IPs, encryption mandatory)
- Dev: relaxed policies (can create public IPs for testing)
- Separate billing (know exactly what prod costs)
- Blast radius: issue in dev doesn't affect prod
```

### Policies Applied

```powershell
# Policy 1: Required Tags (applied at Root MG)
# Every resource must have Environment and Owner tags

# Policy 2: Allowed Regions (applied at Production MG)
# Only UK South and UK West allowed in production

# Policy 3: No Public IPs (applied at Production MG)
# All resources must use Private Endpoints

# Policy 4: Auto-deploy Monitoring Agent (applied at Landing Zones MG)
# Every VM automatically gets Log Analytics agent

# Policy 5: Require HTTPS (applied at Landing Zones MG)
# All App Services must use HTTPS only

# Policy 6: Require Encryption (applied at Production MG)
# All storage accounts must have encryption at rest enabled

# Creating and assigning these policies:
$allowedLocationsPolicy = Get-AzPolicyDefinition | 
    Where-Object { $_.Properties.DisplayName -eq "Allowed locations" }

New-AzPolicyAssignment `
    -Name "prod-allowed-locations" `
    -DisplayName "Production - UK Regions Only" `
    -Scope "/providers/Microsoft.Management/managementGroups/Production" `
    -PolicyDefinition $allowedLocationsPolicy `
    -PolicyParameterObject @{
        listOfAllowedLocations = @{ value = @("uksouth", "ukwest") }
    }
```

---

## 15.6 Networking Design

### Hub-Spoke for ATT Architecture

```
CONNECTIVITY SUBSCRIPTION:
Hub VNet (10.0.0.0/16)
├── GatewaySubnet (10.0.0.0/27)          → VPN Gateway to on-prem
├── AzureFirewallSubnet (10.0.1.0/26)    → Azure Firewall Premium
├── AzureBastionSubnet (10.0.2.0/26)     → Secure VM access
└── SharedServicesSubnet (10.0.3.0/24)   → DNS Resolver, Jumpbox

PRODUCTION SUBSCRIPTION:
Spoke VNet - App Tier (10.1.0.0/16)
├── snet-aks (10.1.1.0/24)               → AKS nodes
├── snet-apim (10.1.2.0/24)              → API Management
├── snet-functions (10.1.3.0/24)         → Function App VNet integration
└── snet-logic (10.1.4.0/24)             → Logic Apps VNet integration

PRODUCTION SUBSCRIPTION:
Spoke VNet - Data Tier (10.2.0.0/16)
├── snet-sql (10.2.1.0/24)               → Azure SQL Private Endpoints
├── snet-cosmos (10.2.2.0/24)            → Cosmos DB Private Endpoints
├── snet-servicebus (10.2.3.0/24)        → Service Bus Private Endpoints
└── snet-storage (10.2.4.0/24)           → Storage Account Private Endpoints
```

### Private Endpoints (Critical for Production)

```
WHAT IS A PRIVATE ENDPOINT?

Without Private Endpoint:
App (10.1.1.5) ──HTTP──► Internet ──► Azure SQL (public endpoint, public IP)
                                       DANGEROUS: SQL accessible from internet

With Private Endpoint:
App (10.1.1.5) ──HTTP──► Private Endpoint (10.2.1.10) ──► Azure SQL
                                                           SAFE: SQL only accessible from inside VNet
                                                           No public internet exposure
```

```powershell
# Create Private Endpoint for Azure SQL
$sqlServer = Get-AzSqlServer -ResourceGroupName "rg-data" -ServerName "sql-myapp-prod"

New-AzPrivateEndpoint `
    -ResourceGroupName "rg-data" `
    -Name "pep-sql-myapp-prod" `
    -Location "uksouth" `
    -Subnet (Get-AzVirtualNetworkSubnetConfig `
        -Name "snet-sql" `
        -VirtualNetwork (Get-AzVirtualNetwork -Name "vnet-spoke-data" -ResourceGroupName "rg-connectivity")) `
    -PrivateLinkServiceConnection (
        New-AzPrivateLinkServiceConnection `
            -Name "sql-connection" `
            -PrivateLinkServiceId $sqlServer.ResourceId `
            -GroupId "sqlServer"
    )

# Register in Private DNS Zone
New-AzPrivateDnsZoneGroup `
    -ResourceGroupName "rg-data" `
    -PrivateEndpointName "pep-sql-myapp-prod" `
    -Name "sql-dns-group" `
    -PrivateDnsZoneConfig (
        New-AzPrivateDnsZoneConfig `
            -Name "privatelink.database.windows.net" `
            -PrivateDnsZoneId (Get-AzPrivateDnsZone -ResourceGroupName "rg-dns" -Name "privatelink.database.windows.net").ResourceId
    )
```

---

## 15.7 Security Implementation

### Managed Identity + Key Vault Pattern

```
THE SECURE PATTERN:
                        ┌──────────────────────┐
                        │  Azure Key Vault      │
                        │  ┌──────────────────┐ │
                        │  │ db-password       │ │
                        │  │ service-bus-conn  │ │
                        │  │ third-party-key   │ │
                        │  └──────────────────┘ │
                        └──────────┬────────────┘
                                   │ read secrets
                                   │ (via Managed Identity)
                        ┌──────────▼────────────┐
                        │   AKS Pod             │
                        │   (has MI identity)   │
                        │   OrderService        │
                        └───────────────────────┘
No password in code, no password in config, no password anywhere!
```

### Implementing Managed Identity for AKS

```powershell
# AKS uses a system-assigned managed identity by default
# Get the AKS kubelet identity (used by pods)
$aks = Get-AzAksCluster -ResourceGroupName "rg-production" -Name "aks-myapp-prod"
$kubeletIdentityId = $aks.IdentityProfile.KubeletIdentity.ObjectId

# Grant Key Vault access to AKS
Set-AzKeyVaultAccessPolicy `
    -VaultName "kv-myapp-prod" `
    -ObjectId $kubeletIdentityId `
    -PermissionsToSecrets Get, List

# Grant ACR access (pull images)
New-AzRoleAssignment `
    -ObjectId $kubeletIdentityId `
    -RoleDefinitionName "AcrPull" `
    -Scope (Get-AzContainerRegistry -ResourceGroupName "rg-platform" -Name "acrmyappprod").Id
```

### Using Secrets in Application Code

```csharp
// Accessing Key Vault from application (using Managed Identity)
// NO connection strings or passwords in code!

using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var client = new SecretClient(
    new Uri("https://kv-myapp-prod.vault.azure.net/"),
    new DefaultAzureCredential()  // Automatically uses Managed Identity
);

// Get database connection string from Key Vault
KeyVaultSecret secret = await client.GetSecretAsync("db-connection-string");
string connectionString = secret.Value;

// Use the connection string
using var connection = new SqlConnection(connectionString);
```

### Network Security Groups (NSGs)

```powershell
# NSG for AKS subnet - only allow specific traffic
$nsgRules = @(
    # Allow APIM to call AKS
    New-AzNetworkSecurityRuleConfig `
        -Name "Allow-APIM-inbound" `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 100 `
        -SourceAddressPrefix "10.1.2.0/24" `    # APIM subnet
        -SourcePortRange "*" `
        -DestinationAddressPrefix "10.1.1.0/24" ` # AKS subnet
        -DestinationPortRange "443" `
        -Access Allow,

    # Allow Azure Load Balancer
    New-AzNetworkSecurityRuleConfig `
        -Name "Allow-AzureLB" `
        -Protocol "*" `
        -Direction Inbound `
        -Priority 200 `
        -SourceAddressPrefix "AzureLoadBalancer" `
        -SourcePortRange "*" `
        -DestinationAddressPrefix "*" `
        -DestinationPortRange "*" `
        -Access Allow,

    # Deny all other inbound
    New-AzNetworkSecurityRuleConfig `
        -Name "DenyAll-inbound" `
        -Protocol "*" `
        -Direction Inbound `
        -Priority 4096 `
        -SourceAddressPrefix "*" `
        -SourcePortRange "*" `
        -DestinationAddressPrefix "*" `
        -DestinationPortRange "*" `
        -Access Deny
)

New-AzNetworkSecurityGroup `
    -Name "nsg-aks-subnet" `
    -ResourceGroupName "rg-production-network" `
    -Location "uksouth" `
    -SecurityRules $nsgRules
```

---

## 15.8 CI/CD with Azure DevOps and Terraform

### What is Infrastructure as Code (IaC)?

```
WITHOUT IaC (manual):
New environment needed?
→ Click through Azure Portal
→ Takes hours, error-prone, not repeatable
→ "It worked in dev but not in prod" (different configs)

WITH IaC (Terraform/Bicep):
New environment needed?
→ Run terraform apply
→ Takes minutes, always same result, repeatable
→ Dev and Prod built from SAME code = consistent
```

### Terraform Overview

```hcl
# main.tf - Define Azure resources as code

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  # Store state in Azure Blob Storage (shared state)
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "satfstatemyapp"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

# Create Resource Group
resource "azurerm_resource_group" "production" {
  name     = "rg-production-myapp"
  location = "uksouth"
  tags = {
    Environment = "Production"
    Owner       = "Platform Team"
    CostCenter  = "IT-001"
  }
}

# Create VNet
resource "azurerm_virtual_network" "spoke_prod" {
  name                = "vnet-spoke-production"
  resource_group_name = azurerm_resource_group.production.name
  location            = azurerm_resource_group.production.location
  address_space       = ["10.1.0.0/16"]
}

# Create AKS Cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-myapp-prod"
  location            = azurerm_resource_group.production.location
  resource_group_name = azurerm_resource_group.production.name
  dns_prefix          = "myapp-prod"

  default_node_pool {
    name       = "system"
    node_count = 3
    vm_size    = "Standard_DS2_v2"
    vnet_subnet_id = azurerm_subnet.aks.id
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    load_balancer_sku = "standard"
  }
}
```

### Azure DevOps Pipeline

```yaml
# azure-pipelines.yml - CI/CD Pipeline

trigger:
  branches:
    include:
      - main
      - develop

stages:
  #============================================
  # STAGE 1: BUILD
  #============================================
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: BuildJob
        pool:
          vmImage: 'ubuntu-latest'
        steps:
          # Restore NuGet packages
          - task: DotNetCoreCLI@2
            displayName: 'Restore packages'
            inputs:
              command: restore
              projects: '**/*.csproj'

          # Build the application
          - task: DotNetCoreCLI@2
            displayName: 'Build application'
            inputs:
              command: build
              projects: '**/*.csproj'
              arguments: '--configuration Release'

          # Run unit tests
          - task: DotNetCoreCLI@2
            displayName: 'Run tests'
            inputs:
              command: test
              projects: '**/*Tests.csproj'
              arguments: '--collect:"XPlat Code Coverage"'

          # Build Docker image
          - task: Docker@2
            displayName: 'Build Docker image'
            inputs:
              containerRegistry: 'acr-service-connection'
              repository: 'orderservice'
              command: 'build'
              Dockerfile: '**/Dockerfile'
              tags: '$(Build.BuildId)'

          # Push to ACR
          - task: Docker@2
            displayName: 'Push to ACR'
            inputs:
              containerRegistry: 'acr-service-connection'
              repository: 'orderservice'
              command: 'push'
              tags: '$(Build.BuildId)'

  #============================================
  # STAGE 2: DEPLOY TO DEV
  #============================================
  - stage: DeployDev
    displayName: 'Deploy to Dev'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployToAKSDev
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                # Apply Terraform for infrastructure
                - task: TerraformTaskV3@3
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/terraform/dev'

                - task: TerraformTaskV3@3
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(System.DefaultWorkingDirectory)/terraform/dev'
                    environmentServiceNameAzureRM: 'azure-service-connection'

                # Deploy to AKS
                - task: KubernetesManifest@0
                  displayName: 'Deploy to AKS Dev'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'aks-dev-service-connection'
                    manifests: '$(System.DefaultWorkingDirectory)/k8s/dev/*.yaml'
                    containers: 'acrmyapp.azurecr.io/orderservice:$(Build.BuildId)'

  #============================================
  # STAGE 3: DEPLOY TO PRODUCTION (with approval)
  #============================================
  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployDev
    condition: succeeded()
    jobs:
      - deployment: DeployToAKSProd
        environment: 'production'  # Has approval gate configured
        strategy:
          runOnce:
            deploy:
              steps:
                - task: KubernetesManifest@0
                  displayName: 'Deploy to AKS Prod'
                  inputs:
                    action: 'deploy'
                    kubernetesServiceConnection: 'aks-prod-service-connection'
                    manifests: '$(System.DefaultWorkingDirectory)/k8s/prod/*.yaml'
                    containers: 'acrmyapp.azurecr.io/orderservice:$(Build.BuildId)'
```

### Why Azure DevOps?

| Feature | Value |
|---------|-------|
| **Boards** | Sprint planning, work items, backlog |
| **Repos** | Git repositories (alternative to GitHub) |
| **Pipelines** | CI/CD automation |
| **Artifacts** | Package management (NuGet, npm) |
| **Test Plans** | Manual + automated test tracking |

```
CI (Continuous Integration):
Developer pushes code → Pipeline triggers automatically → Build + Test
→ Catch bugs immediately, not weeks later

CD (Continuous Deployment):
After passing tests → Automatically deploy to Dev
After dev passes → Require approval → Deploy to Production
→ Reliable, repeatable deployments
→ Audit trail of every change
```

---

## 15.9 Full Interview Answer Framework

### How to Answer the Deep Technical Question

```
THE QUESTION:
"Walk through a specific project where you designed and implemented
Azure architecture from scratch involving Service Bus, Event Grid,
Landing Zones, and Hub & Spoke network."

THE STRUCTURE (STAR method):
S - Situation (business problem)
T - Task (your responsibility)
A - Action (what you did - the technical detail)
R - Result (outcome/benefit)
```

### Model Answer - Word for Word

---

**SITUATION:**

> "At ATT Solutions, we had a legacy monolithic application running on on-premises servers. The application handled 80+ business processes — orders, payments, inventory, notifications — all tightly coupled in a single codebase. The problems were significant: the system was impossible to scale individual components, any deployment risked breaking unrelated features, and the infrastructure couldn't handle peak load without major investment."

---

**TASK:**

> "I was brought in as the Azure architect to design and implement a complete migration from this monolith to a cloud-native, event-driven microservices architecture on Azure. This was a greenfield architecture — no existing Azure environment, so I had to build everything from scratch."

---

**ACTION - LANDING ZONE SETUP:**

> "The first thing I did was establish an Azure Landing Zone before any application workloads were touched. I created a Management Group hierarchy — a Root group with Platform and Application Landing Zone groups underneath. Under Platform, I created three subscriptions: Connectivity (for shared networking), Management (for centralized logging and monitoring), and Identity (for Azure AD and Key Vault). Under Application Landing Zones, I created separate Production and Non-Production management groups, each with their own subscriptions.
>
> I then applied Azure Policies at the management group level. At the root, I enforced required tags (Environment, Owner, CostCenter) and enabled Microsoft Defender for Cloud. At the Production management group, I restricted resources to UK South and UK West only, denied public IP addresses, and enforced encryption on all storage. These policies meant every team working in production automatically inherited consistent governance."

---

**ACTION - NETWORKING:**

> "For networking, I implemented a Hub-Spoke topology. In the Connectivity subscription, I created a Hub VNet containing an Azure Firewall Premium for traffic inspection and filtering, a VPN Gateway for connecting our on-premises datacenter via IPSec tunnel, Azure Bastion for secure VM access without public IPs, and a Private DNS Resolver. Then I created Spoke VNets in the production subscription — one for the application tier housing AKS, APIM, and Functions, and one for the data tier housing private endpoints for SQL, Cosmos DB, and Service Bus. I peered the spokes to the hub with 'Use Remote Gateways' enabled on spokes and 'Allow Gateway Transit' on the hub, then applied User Defined Routes to force all spoke traffic through the Azure Firewall. Every PaaS service was configured with Private Endpoints, completely removing public internet exposure."

---

**ACTION - INTEGRATION PLATFORM:**

> "For the integration layer, I used Azure API Management as the single entry point for all 80+ microservices. APIM handled authentication using JWT validation, applied rate limiting per client application, performed request routing to the right microservice in AKS, and provided a developer portal for documentation.
>
> For async communication between services, I chose Azure Service Bus for commands — messages that need guaranteed delivery and processing. For example, when an order is placed, the OrderService drops a message on the order-processing queue and continues. The PaymentService picks it up independently. This decoupling meant a slow payment service wouldn't block order placement. I used Service Bus Topics for scenarios where multiple services need the same message — the order-events topic had subscriptions from InventoryService, NotificationService, and ReportingService.
>
> For event notifications — something happened, notify whoever cares — I used Azure Event Grid. When payment is confirmed, Event Grid broadcasts this to multiple handlers simultaneously: one Function sends a confirmation email, another updates the order status, another logs to the analytics system.
>
> Azure Functions handled stateless processing tasks — transforming data formats, processing queue messages, scheduled jobs. Azure Logic Apps handled multi-step approval workflows, particularly order approval processes where we needed manager email approval integrated with Outlook, because Logic Apps gave us a visual workflow that the business team could understand and modify."

---

**ACTION - SECURITY:**

> "Security was built in from the start. All application identities used Managed Identity — no service accounts, no passwords stored in configuration files. Applications retrieved secrets from Azure Key Vault at runtime using their Managed Identity. This meant even if someone got access to the code or configuration, there were no credentials to steal.
>
> For human access, all operations team members accessed Azure through PIM — Privileged Identity Management — meaning nobody had standing Owner or Contributor access. They requested elevated access when needed, got it for a limited time window, and it expired automatically."

---

**ACTION - AUTOMATION:**

> "Every piece of infrastructure was defined in Terraform and stored in Azure Repos. Environments were deployed and updated via Azure DevOps pipelines. A developer pushing code triggered a CI pipeline that built the Docker image, ran unit tests, pushed the image to Azure Container Registry, and deployed to the dev AKS cluster automatically. Promotion to production required manual approval in Azure DevOps, creating an audit trail of who approved every production deployment."

---

**ACTION - MONITORING:**

> "All subscriptions sent logs to a central Log Analytics Workspace in the Management subscription, enforced via an Azure Policy with DeployIfNotExists effect that automatically installed the monitoring agent on every VM. Azure Monitor alerts notified the operations team of critical issues. Application Insights was integrated into every microservice for end-to-end distributed tracing, so when a request failed we could trace it across all 80 services to find exactly where it broke."

---

**RESULT:**

> "The result was a scalable, event-driven platform that handled peak load by auto-scaling individual microservices in AKS without any manual intervention. Deployment frequency went from monthly releases to multiple times per day, with deployment time dropping from 4 hours to 15 minutes. The security posture was significantly improved — Microsoft Defender for Cloud showed a Secure Score above 80% from day one due to the Landing Zone policies. The team had complete visibility through centralized monitoring, and the business had audit trails for every change through Azure DevOps and Activity Logs."

---

## 15.10 Quick Reference - Key Concepts

### Architecture Decision Summary

| Decision | Choice | Reason |
|----------|--------|--------|
| Container orchestration | AKS | Managed Kubernetes, Azure integration |
| API Gateway | APIM | Security, rate limiting, single entry |
| Async messaging | Service Bus | Guaranteed delivery, transactions |
| Event notifications | Event Grid | Fan-out, near-real-time |
| Serverless processing | Azure Functions | Event-driven, cost per execution |
| Orchestration | Logic Apps | Visual, business approvals |
| Secrets | Key Vault + Managed Identity | No passwords in code |
| Network topology | Hub-Spoke | Centralized control, cost |
| IaC | Terraform | Multi-cloud, mature tooling |
| CI/CD | Azure DevOps | Microsoft ecosystem, enterprise features |

### Terminology Cheat Sheet

| Term | Simple Explanation |
|------|-------------------|
| **Microservices** | Small, independent services that each do one thing |
| **Monolith** | One big application that does everything |
| **Event-driven** | Services communicate by publishing/subscribing to events |
| **Loose coupling** | Services don't directly depend on each other |
| **Container** | Packaged application with all its dependencies |
| **Kubernetes/AKS** | Manages running many containers at scale |
| **CI/CD** | Automate build, test, and deploy when code changes |
| **IaC** | Define infrastructure in code files (Terraform) |
| **Managed Identity** | Azure app identity with no password |
| **Private Endpoint** | Private IP for a PaaS service inside your VNet |
| **Landing Zone** | Pre-built secure Azure foundation |
| **Hub-Spoke** | Central hub for shared services, spokes for workloads |

---

## Summary

| Topic | Key Points |
|-------|------------|
| Event-Driven Architecture | Services communicate via events, loose coupling |
| Service Bus | Reliable messaging, commands, queues + topics |
| Event Grid | Event notifications, broadcast to many |
| APIM | Single gateway, security, rate limiting |
| AKS | Container orchestration for microservices |
| Functions | Serverless event processing |
| Logic Apps | Visual workflows, approvals |
| Landing Zone | Foundation: MG, Subscriptions, Policy, RBAC |
| Hub-Spoke | Centralized network with private endpoints |
| Key Vault + MI | Zero-password security |
| Azure DevOps + Terraform | Automated, repeatable infrastructure and deployments |

---

## Additional Resources

- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Microservices on Azure](https://docs.microsoft.com/azure/architecture/microservices/)
- [Event-Driven Architecture](https://docs.microsoft.com/azure/architecture/guide/architecture-styles/event-driven)
- [AKS Documentation](https://docs.microsoft.com/azure/aks/)
- [Terraform AzureRM Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure DevOps Documentation](https://docs.microsoft.com/azure/devops/)
- [Well-Architected Framework](https://docs.microsoft.com/azure/architecture/framework/)
