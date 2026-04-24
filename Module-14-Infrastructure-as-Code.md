# Module 14 - Infrastructure as Code (Complete Guide: Beginner to Expert)

## Learning Objectives
By the end of this module, you will be able to:
- Understand Infrastructure as Code principles and benefits
- Master ARM Templates for Azure deployments
- Create and deploy Azure resources using Bicep
- Implement Terraform for multi-cloud infrastructure
- Write reusable and modular IaC code
- Implement IaC testing and validation
- Integrate IaC with CI/CD pipelines
- Apply best practices for production IaC

---

## 14.1 Introduction to Infrastructure as Code

### What is Infrastructure as Code?
Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable configuration files rather than through manual processes.

### Benefits of Infrastructure as Code

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Benefits of Infrastructure as Code                        │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │
│  │   Consistency   │  │  Repeatability  │  │    Version      │            │
│  │                 │  │                 │  │    Control      │            │
│  │ Same config     │  │ Deploy same     │  │ Track changes,  │            │
│  │ every time      │  │ infra anywhere  │  │ rollback        │            │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘            │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │
│  │   Automation    │  │  Documentation  │  │    Testing      │            │
│  │                 │  │                 │  │                 │            │
│  │ CI/CD           │  │ Code IS the     │  │ Validate before │            │
│  │ integration     │  │ documentation   │  │ deployment      │            │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘            │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐            │
│  │  Cost Control   │  │   Compliance    │  │     Speed       │            │
│  │                 │  │                 │  │                 │            │
│  │ Review changes  │  │ Policy-as-Code  │  │ Deploy in       │            │
│  │ before deploy   │  │ enforcement     │  │ minutes         │            │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘            │
└─────────────────────────────────────────────────────────────────────────────┘
```

### IaC Approaches

| Approach | Description | Tools |
|----------|-------------|-------|
| **Declarative** | Define desired state, tool figures out how | Terraform, ARM, Bicep |
| **Imperative** | Define exact steps to achieve state | Scripts, PowerShell, CLI |

#### 🏢 Real-World IaC Use Cases:

**Multi-Environment Deployment:**
```
Company: SaaS Platform (Dev, Staging, Prod environments)
Challenge: Consistent environments, different scales

Bicep Modular Structure:
├── modules/
│   ├── networking.bicep          # VNet, subnets, NSGs
│   ├── compute.bicep             # App Service, Functions
│   ├── database.bicep            # SQL, Cosmos DB
│   └── monitoring.bicep          # Log Analytics, alerts
│
├── environments/
│   ├── dev.parameters.json       # Small VMs, basic SKUs
│   ├── staging.parameters.json   # Medium VMs, staging slots
│   └── prod.parameters.json      # Large VMs, HA config
│
└── main.bicep                    # Orchestrates modules

Parameter Differences:
┌─────────────────────────────────────────────────────────────────┐
│  Environment  │  App Service  │  SQL Database  │  Cost/Month   │
├───────────────┼───────────────┼────────────────┼───────────────┤
│  Development  │  B1 (Basic)   │  Basic 5DTU    │  $150         │
│  Staging      │  S1 (Standard)│  Standard S0   │  $400         │
│  Production   │  P2V2 (x3)    │  Premium P1    │  $2,500       │
└─────────────────────────────────────────────────────────────────┘

CI/CD Pipeline:
┌─────────────────────────────────────────────────────────────────┐
│  Commit to main branch                                          │
│      │                                                           │
│      ▼                                                           │
│  what-if (preview changes)                                       │
│      │                                                           │
│      ▼                                                           │
│  Deploy to Dev (auto-approve)                                    │
│      │                                                           │
│      ▼                                                           │
│  Deploy to Staging (auto-approve if tests pass)                  │
│      │                                                           │
│      ▼                                                           │
│  Deploy to Prod (manual approval required)                       │
│      │                                                           │
│      ▼                                                           │
│  Smoke tests → Rollback if failed                               │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Deployment time: 15 minutes (was 4 hours manual)
├── Configuration drift: Zero (IaC is source of truth)
├── Environment parity: 100% (same code, different params)
└── Rollback time: 10 minutes (redeploy previous version)
```

**Disaster Recovery Automation:**
```
Company: CriticalApp (Finance, RPO: 15 min, RTO: 1 hour)
Challenge: Automated DR site deployment

Terraform Multi-Region Setup:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Primary Region (East US)        Secondary Region (West US)     │
│  ┌─────────────────────┐         ┌─────────────────────┐       │
│  │  Production         │         │  DR Site            │       │
│  │  ├── App Service    │◄───────►│  ├── App Service    │       │
│  │  ├── SQL Primary    │   Geo-  │  ├── SQL Secondary  │       │
│  │  ├── Storage GRS    │   Rep   │  ├── Storage        │       │
│  │  └── Traffic Mgr    │         │  └── (Hot Standby)  │       │
│  └─────────────────────┘         └─────────────────────┘       │
│                                                                  │
│  terraform/                                                      │
│  ├── modules/                                                    │
│  │   ├── primary-region/                                        │
│  │   └── dr-region/                                             │
│  ├── main.tf           # Deploys both regions                  │
│  └── failover.tf       # Runbook for DR activation             │
│                                                                  │
│  DR Activation (Automated):                                      │
│  1. Health check fails for 5 minutes                            │
│  2. Azure Automation runbook triggered                          │
│  3. Terraform apply -var="active_region=westus"                │
│  4. Traffic Manager updated                                      │
│  5. Notifications sent                                           │
│  6. Total time: 45 minutes (meets 1-hour RTO)                   │
└─────────────────────────────────────────────────────────────────┘

DR Test (Quarterly):
├── Simulate primary failure
├── Terraform auto-deploys DR
├── Verify application works
├── Failback to primary
└── Document recovery time (actual: 38 minutes)
```

**Self-Service Infrastructure (Platform Engineering):**
```
Company: TechCorp (500 developers, 50 teams)
Challenge: Enable teams to deploy infrastructure safely

Internal Developer Platform:
┌─────────────────────────────────────────────────────────────────┐
│                    Platform Team (IaC Modules)                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  terraform-azure-modules (Internal Registry)                    │
│  ├── web-app-standard/      # Pre-approved web app config      │
│  │   └── Includes: App Service, Storage, Key Vault             │
│  │   └── Security: Private endpoint, managed identity          │
│  │   └── Compliance: Logging, retention, encryption             │
│  │                                                               │
│  ├── api-backend/           # Pre-approved API infrastructure  │
│  │   └── Includes: AKS namespace, API Management               │
│  │                                                               │
│  └── data-pipeline/         # Pre-approved analytics stack     │
│      └── Includes: Data Factory, Synapse, ADLS                 │
│                                                                  │
│  Usage (Product Team):                                           │
│  module "my_web_app" {                                          │
│    source  = "terraform-azure-modules/web-app-standard/azure"  │
│    version = "1.2.0"                                            │
│                                                                  │
│    app_name    = "product-catalog"                              │
│    environment = "prod"                                          │
│    team        = "ecommerce"                                     │
│    # All security/compliance included automatically             │
│  }                                                               │
└─────────────────────────────────────────────────────────────────┘

Benefits:
├── Developer self-service: Yes (no tickets to Infra team)
├── Compliance: 100% (built into modules)
├── Time to infrastructure: 30 minutes (was 2 weeks)
├── Module updates: Automatic security patches
└── Audit: Full history in Git
```

**Immutable Infrastructure Pattern:**
```
Concept: Never modify, always replace

Traditional (Mutable):
VM created → Updates applied → Config drift → Inconsistent

Immutable (IaC):
VM v1.0 created ─┐
                 ├─► Delete v1.0 ─► VM v1.1 created
VM v1.1 defined ─┘   (no modify)

Implementation with Packer + Terraform:
┌─────────────────────────────────────────────────────────────────┐
│  Step 1: Packer builds golden image                             │
│  ├── Base: Ubuntu 22.04                                         │
│  ├── Install: Docker, monitoring agents                         │
│  ├── Harden: CIS benchmark compliance                           │
│  └── Output: Azure Managed Image v1.2.3                        │
│                                                                  │
│  Step 2: Terraform deploys VMSS                                 │
│  ├── Uses: Managed Image v1.2.3                                 │
│  └── Update policy: Rolling (zero downtime)                     │
│                                                                  │
│  Update Process:                                                 │
│  ├── Build new image v1.2.4                                     │
│  ├── Terraform updates VMSS image reference                     │
│  ├── VMSS rolling update replaces VMs                           │
│  └── Old VMs deleted (never modified)                           │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Configuration drift: Impossible
├── Rollback: Point to previous image
├── Security: Every deployment is fresh
└── Debugging: Exact same image as prod
```

### IaC Tools for Azure

| Tool | Type | Pros | Cons |
|------|------|------|------|
| **ARM Templates** | Declarative, JSON | Native Azure, full feature support | Verbose, complex syntax |
| **Bicep** | Declarative, DSL | Cleaner syntax, transpiles to ARM | Azure-only |
| **Terraform** | Declarative, HCL | Multi-cloud, large ecosystem | Learning curve, state management |
| **Pulumi** | Declarative, code | Real programming languages | Complexity |

---

## 14.2 ARM Templates

### ARM Template Structure

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        // Input values
    },
    "variables": {
        // Calculated values
    },
    "resources": [
        // Resources to deploy
    ],
    "outputs": {
        // Return values
    }
}
```

### Complete ARM Template Example

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "type": "string",
            "minLength": 3,
            "maxLength": 24,
            "metadata": {
                "description": "Name of the storage account"
            }
        },
        "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
                "description": "Location for all resources"
            }
        },
        "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "allowedValues": [
                "Standard_LRS",
                "Standard_GRS",
                "Standard_ZRS",
                "Premium_LRS"
            ]
        },
        "environment": {
            "type": "string",
            "allowedValues": ["dev", "test", "prod"]
        }
    },
    "variables": {
        "storageAccountFullName": "[concat(parameters('storageAccountName'), uniqueString(resourceGroup().id))]",
        "defaultTags": {
            "Environment": "[parameters('environment')]",
            "ManagedBy": "ARM"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2023-01-01",
            "name": "[variables('storageAccountFullName')]",
            "location": "[parameters('location')]",
            "tags": "[variables('defaultTags')]",
            "sku": {
                "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {
                "minimumTlsVersion": "TLS1_2",
                "supportsHttpsTrafficOnly": true,
                "allowBlobPublicAccess": false,
                "networkAcls": {
                    "defaultAction": "Deny",
                    "bypass": "AzureServices"
                }
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts/blobServices/containers",
            "apiVersion": "2023-01-01",
            "name": "[concat(variables('storageAccountFullName'), '/default/data')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountFullName'))]"
            ],
            "properties": {
                "publicAccess": "None"
            }
        }
    ],
    "outputs": {
        "storageAccountId": {
            "type": "string",
            "value": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountFullName'))]"
        },
        "storageAccountName": {
            "type": "string",
            "value": "[variables('storageAccountFullName')]"
        },
        "primaryEndpoint": {
            "type": "string",
            "value": "[reference(variables('storageAccountFullName')).primaryEndpoints.blob]"
        }
    }
}
```

### ARM Template Functions

| Category | Functions |
|----------|-----------|
| **Array** | array, concat, contains, createArray, empty, first, last, length |
| **Comparison** | equals, greater, less, greaterOrEquals, lessOrEquals |
| **Date** | dateTimeAdd, utcNow |
| **Deployment** | deployment, environment, parameters, variables |
| **Logical** | and, bool, false, if, not, or, true |
| **Numeric** | add, div, mod, mul, sub, int, float, copyIndex |
| **Resource** | extensionResourceId, list*, reference, resourceGroup, resourceId, subscription |
| **String** | base64, concat, contains, dataUri, empty, endsWith, first, format, guid, indexOf, join, last, length, newGuid, padLeft, replace, skip, split, startsWith, string, substring, take, toLower, toUpper, trim, uniqueString, uri, uriComponent |

### Deploying ARM Templates

```powershell
# Deploy to resource group
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "template.json" `
    -TemplateParameterFile "parameters.json" `
    -Name "DeploymentName"

# Deploy to subscription scope
New-AzSubscriptionDeployment `
    -Location "East US" `
    -TemplateFile "subscription-template.json" `
    -Name "SubscriptionDeployment"

# What-if deployment
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "template.json" `
    -WhatIf
```

```bash
# Azure CLI
az deployment group create \
    --resource-group MyRG \
    --template-file template.json \
    --parameters @parameters.json

# What-if
az deployment group what-if \
    --resource-group MyRG \
    --template-file template.json
```

### Linked Templates

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageTemplateUri": {
            "type": "string"
        }
    },
    "resources": [
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2021-04-01",
            "name": "linkedStorageDeployment",
            "properties": {
                "mode": "Incremental",
                "templateLink": {
                    "uri": "[parameters('storageTemplateUri')]",
                    "contentVersion": "1.0.0.0"
                },
                "parameters": {
                    "storageAccountName": {
                        "value": "mystorage"
                    }
                }
            }
        }
    ]
}
```

---

## 14.3 Azure Bicep

### What is Bicep?
Bicep is a domain-specific language (DSL) that uses declarative syntax to deploy Azure resources. It's a transparent abstraction over ARM templates.

### Bicep vs ARM Template

```
ARM Template (JSON)                      Bicep
─────────────────────────────────────────────────────────────────────
{                                        resource storageAccount
  "type": "Microsoft.Storage/             'Microsoft.Storage/
    storageAccounts",                       storageAccounts@2023-01-01' = {
  "apiVersion": "2023-01-01",              name: 'mystorageaccount'
  "name": "mystorageaccount",              location: resourceGroup().location
  "location": "[resourceGroup()            sku: {
    .location]",                              name: 'Standard_LRS'
  "sku": {                                 }
    "name": "Standard_LRS"                 kind: 'StorageV2'
  },                                     }
  "kind": "StorageV2"
}
```

### Bicep File Structure

```bicep
// targetScope - where to deploy
targetScope = 'resourceGroup'  // Default

// Parameters - inputs
@description('The name of the environment')
@allowed([
  'dev'
  'test'
  'prod'
])
param environment string

@description('The location for resources')
param location string = resourceGroup().location

@minLength(3)
@maxLength(24)
param storageAccountName string

@secure()
param adminPassword string

// Variables - calculated values
var resourcePrefix = '${environment}-'
var uniqueSuffix = uniqueString(resourceGroup().id)

// Resources - what to deploy
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: '${storageAccountName}${uniqueSuffix}'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
  }
}

// Outputs - return values
output storageAccountId string = storageAccount.id
output primaryEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

### Complete Bicep Example - Web App with SQL

```bicep
// main.bicep
targetScope = 'resourceGroup'

@description('Environment name')
@allowed(['dev', 'test', 'prod'])
param environment string

@description('Location for all resources')
param location string = resourceGroup().location

@description('The name of the web application')
param webAppName string

@description('The administrator username for SQL Server')
param sqlAdminUsername string

@secure()
@description('The administrator password for SQL Server')
param sqlAdminPassword string

// Variables
var appServicePlanName = 'asp-${webAppName}-${environment}'
var webAppFullName = 'app-${webAppName}-${environment}'
var sqlServerName = 'sql-${webAppName}-${environment}'
var sqlDatabaseName = 'sqldb-${webAppName}'
var appInsightsName = 'appi-${webAppName}-${environment}'

var tags = {
  Environment: environment
  Application: webAppName
  ManagedBy: 'Bicep'
}

// App Service Plan
resource appServicePlan 'Microsoft.Web/serverfarms@2022-09-01' = {
  name: appServicePlanName
  location: location
  tags: tags
  sku: {
    name: environment == 'prod' ? 'P1v3' : 'B1'
    tier: environment == 'prod' ? 'PremiumV3' : 'Basic'
  }
  kind: 'linux'
  properties: {
    reserved: true
  }
}

// Web App
resource webApp 'Microsoft.Web/sites@2022-09-01' = {
  name: webAppFullName
  location: location
  tags: tags
  properties: {
    serverFarmId: appServicePlan.id
    siteConfig: {
      linuxFxVersion: 'DOTNETCORE|8.0'
      alwaysOn: environment == 'prod'
      minTlsVersion: '1.2'
      http20Enabled: true
      appSettings: [
        {
          name: 'APPINSIGHTS_INSTRUMENTATIONKEY'
          value: appInsights.properties.InstrumentationKey
        }
        {
          name: 'APPLICATIONINSIGHTS_CONNECTION_STRING'
          value: appInsights.properties.ConnectionString
        }
        {
          name: 'ApplicationInsightsAgent_EXTENSION_VERSION'
          value: '~3'
        }
      ]
      connectionStrings: [
        {
          name: 'DefaultConnection'
          connectionString: 'Server=tcp:${sqlServer.properties.fullyQualifiedDomainName},1433;Database=${sqlDatabaseName};User ID=${sqlAdminUsername};Password=${sqlAdminPassword};Encrypt=true;'
          type: 'SQLAzure'
        }
      ]
    }
    httpsOnly: true
  }
  identity: {
    type: 'SystemAssigned'
  }
}

// SQL Server
resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
  name: sqlServerName
  location: location
  tags: tags
  properties: {
    administratorLogin: sqlAdminUsername
    administratorLoginPassword: sqlAdminPassword
    version: '12.0'
    minimalTlsVersion: '1.2'
    publicNetworkAccess: 'Enabled'
  }
}

// SQL Database
resource sqlDatabase 'Microsoft.Sql/servers/databases@2022-05-01-preview' = {
  parent: sqlServer
  name: sqlDatabaseName
  location: location
  tags: tags
  sku: {
    name: environment == 'prod' ? 'S1' : 'Basic'
    tier: environment == 'prod' ? 'Standard' : 'Basic'
  }
  properties: {
    collation: 'SQL_Latin1_General_CP1_CI_AS'
    maxSizeBytes: 2147483648
  }
}

// SQL Firewall rule for Azure services
resource sqlFirewallRule 'Microsoft.Sql/servers/firewallRules@2022-05-01-preview' = {
  parent: sqlServer
  name: 'AllowAzureServices'
  properties: {
    startIpAddress: '0.0.0.0'
    endIpAddress: '0.0.0.0'
  }
}

// Application Insights
resource appInsights 'Microsoft.Insights/components@2020-02-02' = {
  name: appInsightsName
  location: location
  tags: tags
  kind: 'web'
  properties: {
    Application_Type: 'web'
    Request_Source: 'rest'
  }
}

// Outputs
output webAppUrl string = 'https://${webApp.properties.defaultHostName}'
output webAppPrincipalId string = webApp.identity.principalId
output sqlServerFqdn string = sqlServer.properties.fullyQualifiedDomainName
output appInsightsKey string = appInsights.properties.InstrumentationKey
```

### Bicep Modules

```bicep
// modules/storage.bicep
@description('Storage account name')
param storageAccountName string

@description('Location')
param location string

@description('SKU name')
@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_ZRS'
])
param skuName string = 'Standard_LRS'

@description('Tags')
param tags object = {}

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  tags: tags
  sku: {
    name: skuName
  }
  kind: 'StorageV2'
  properties: {
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
  }
}

output id string = storageAccount.id
output name string = storageAccount.name
output primaryEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

```bicep
// main.bicep - Using modules
module storage 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'mystorageaccount'
    location: location
    skuName: 'Standard_LRS'
    tags: {
      Environment: 'dev'
    }
  }
}

// Use outputs from module
output storageId string = storage.outputs.id
```

### Deploying Bicep

```bash
# Compile Bicep to ARM
az bicep build --file main.bicep

# Deploy directly
az deployment group create \
    --resource-group MyRG \
    --template-file main.bicep \
    --parameters environment=dev

# What-if
az deployment group what-if \
    --resource-group MyRG \
    --template-file main.bicep \
    --parameters environment=dev
```

```powershell
# PowerShell deployment
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "main.bicep" `
    -environment "dev"
```

---

## 14.4 Terraform for Azure

### Terraform Basics

```hcl
# Configure the Azure provider
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  required_version = ">= 1.0"
}

provider "azurerm" {
  features {}
}

# Resource definition
resource "azurerm_resource_group" "example" {
  name     = "example-rg"
  location = "East US"
}
```

### Terraform Workflow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Terraform Workflow                                    │
│                                                                              │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│   │  Write   │ →  │   Init   │ →  │   Plan   │ →  │  Apply   │            │
│   │   Code   │    │          │    │          │    │          │            │
│   └──────────┘    └──────────┘    └──────────┘    └──────────┘            │
│        │              │               │               │                     │
│        │              │               │               │                     │
│   Create .tf      Download        Generate        Execute                   │
│   files           providers       execution       changes                   │
│                   & modules       plan                                      │
│                                                                              │
│                                       │                                      │
│                                       ▼                                      │
│                              ┌────────────────┐                             │
│                              │  State File    │                             │
│                              │ (terraform.    │                             │
│                              │   tfstate)     │                             │
│                              └────────────────┘                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Complete Terraform Configuration

```hcl
# main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }

  # Backend configuration for state storage
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}

provider "azurerm" {
  features {
    key_vault {
      purge_soft_delete_on_destroy = true
    }
    resource_group {
      prevent_deletion_if_contains_resources = false
    }
  }
}

# Variables
variable "environment" {
  type        = string
  description = "Environment name"
  validation {
    condition     = contains(["dev", "test", "prod"], var.environment)
    error_message = "Environment must be dev, test, or prod."
  }
}

variable "location" {
  type        = string
  default     = "eastus"
  description = "Azure region for resources"
}

variable "app_name" {
  type        = string
  description = "Application name"
}

variable "sql_admin_password" {
  type        = string
  sensitive   = true
  description = "SQL Server administrator password"
}

# Locals
locals {
  resource_prefix = "${var.app_name}-${var.environment}"
  common_tags = {
    Environment = var.environment
    Application = var.app_name
    ManagedBy   = "Terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }
}

# Random suffix for unique names
resource "random_string" "suffix" {
  length  = 6
  special = false
  upper   = false
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "rg-${local.resource_prefix}"
  location = var.location
  tags     = local.common_tags
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "vnet-${local.resource_prefix}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  address_space       = ["10.0.0.0/16"]
  tags                = local.common_tags
}

# Subnets
resource "azurerm_subnet" "web" {
  name                 = "snet-web"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
  service_endpoints    = ["Microsoft.Sql", "Microsoft.KeyVault"]

  delegation {
    name = "webapp-delegation"
    service_delegation {
      name = "Microsoft.Web/serverFarms"
      actions = [
        "Microsoft.Network/virtualNetworks/subnets/action"
      ]
    }
  }
}

resource "azurerm_subnet" "app" {
  name                 = "snet-app"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
  service_endpoints    = ["Microsoft.Sql"]
}

# App Service Plan
resource "azurerm_service_plan" "main" {
  name                = "asp-${local.resource_prefix}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  os_type             = "Linux"
  sku_name            = var.environment == "prod" ? "P1v3" : "B1"
  tags                = local.common_tags
}

# Application Insights
resource "azurerm_application_insights" "main" {
  name                = "appi-${local.resource_prefix}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  application_type    = "web"
  tags                = local.common_tags
}

# Web App
resource "azurerm_linux_web_app" "main" {
  name                = "app-${local.resource_prefix}-${random_string.suffix.result}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  service_plan_id     = azurerm_service_plan.main.id
  https_only          = true
  tags                = local.common_tags

  site_config {
    always_on        = var.environment == "prod"
    minimum_tls_version = "1.2"
    http2_enabled    = true

    application_stack {
      dotnet_version = "8.0"
    }
  }

  app_settings = {
    "APPINSIGHTS_INSTRUMENTATIONKEY"        = azurerm_application_insights.main.instrumentation_key
    "APPLICATIONINSIGHTS_CONNECTION_STRING" = azurerm_application_insights.main.connection_string
    "ApplicationInsightsAgent_EXTENSION_VERSION" = "~3"
  }

  connection_string {
    name  = "DefaultConnection"
    type  = "SQLAzure"
    value = "Server=tcp:${azurerm_mssql_server.main.fully_qualified_domain_name},1433;Database=${azurerm_mssql_database.main.name};Authentication=Active Directory Default;"
  }

  identity {
    type = "SystemAssigned"
  }

  virtual_network_subnet_id = azurerm_subnet.web.id
}

# SQL Server
resource "azurerm_mssql_server" "main" {
  name                         = "sql-${local.resource_prefix}-${random_string.suffix.result}"
  resource_group_name          = azurerm_resource_group.main.name
  location                     = azurerm_resource_group.main.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = var.sql_admin_password
  minimum_tls_version          = "1.2"
  tags                         = local.common_tags

  azuread_administrator {
    login_username = "AzureAD Admin"
    object_id      = data.azurerm_client_config.current.object_id
  }
}

# SQL Database
resource "azurerm_mssql_database" "main" {
  name                        = "sqldb-${local.resource_prefix}"
  server_id                   = azurerm_mssql_server.main.id
  collation                   = "SQL_Latin1_General_CP1_CI_AS"
  max_size_gb                 = var.environment == "prod" ? 50 : 2
  sku_name                    = var.environment == "prod" ? "S1" : "Basic"
  zone_redundant              = var.environment == "prod"
  tags                        = local.common_tags
}

# SQL Firewall - Allow Azure Services
resource "azurerm_mssql_firewall_rule" "azure_services" {
  name             = "AllowAzureServices"
  server_id        = azurerm_mssql_server.main.id
  start_ip_address = "0.0.0.0"
  end_ip_address   = "0.0.0.0"
}

# SQL VNet Rule
resource "azurerm_mssql_virtual_network_rule" "web" {
  name      = "web-subnet-rule"
  server_id = azurerm_mssql_server.main.id
  subnet_id = azurerm_subnet.web.id
}

# Key Vault
resource "azurerm_key_vault" "main" {
  name                       = "kv-${local.resource_prefix}-${random_string.suffix.result}"
  resource_group_name        = azurerm_resource_group.main.name
  location                   = azurerm_resource_group.main.location
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 90
  purge_protection_enabled   = var.environment == "prod"
  enable_rbac_authorization  = true
  tags                       = local.common_tags

  network_acls {
    default_action             = "Deny"
    bypass                     = "AzureServices"
    virtual_network_subnet_ids = [azurerm_subnet.web.id]
  }
}

# Data sources
data "azurerm_client_config" "current" {}

# Outputs
output "web_app_url" {
  value       = "https://${azurerm_linux_web_app.main.default_hostname}"
  description = "Web App URL"
}

output "sql_server_fqdn" {
  value       = azurerm_mssql_server.main.fully_qualified_domain_name
  description = "SQL Server FQDN"
}

output "key_vault_uri" {
  value       = azurerm_key_vault.main.vault_uri
  description = "Key Vault URI"
}

output "application_insights_key" {
  value       = azurerm_application_insights.main.instrumentation_key
  sensitive   = true
  description = "Application Insights Key"
}
```

### Terraform Variables File

```hcl
# terraform.tfvars
environment = "dev"
location    = "eastus"
app_name    = "mywebapp"

# environments/dev.tfvars
environment = "dev"
location    = "eastus"
app_name    = "mywebapp"

# environments/prod.tfvars
environment = "prod"
location    = "eastus"
app_name    = "mywebapp"
```

### Terraform Modules

```hcl
# modules/webapp/main.tf
variable "name" {
  type = string
}

variable "resource_group_name" {
  type = string
}

variable "location" {
  type = string
}

variable "service_plan_id" {
  type = string
}

variable "tags" {
  type    = map(string)
  default = {}
}

resource "azurerm_linux_web_app" "main" {
  name                = var.name
  resource_group_name = var.resource_group_name
  location            = var.location
  service_plan_id     = var.service_plan_id
  https_only          = true
  tags                = var.tags

  site_config {
    always_on = true
    application_stack {
      dotnet_version = "8.0"
    }
  }

  identity {
    type = "SystemAssigned"
  }
}

output "id" {
  value = azurerm_linux_web_app.main.id
}

output "default_hostname" {
  value = azurerm_linux_web_app.main.default_hostname
}

output "principal_id" {
  value = azurerm_linux_web_app.main.identity[0].principal_id
}
```

```hcl
# Using the module
module "webapp" {
  source = "./modules/webapp"

  name                = "app-${local.resource_prefix}"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  service_plan_id     = azurerm_service_plan.main.id
  tags                = local.common_tags
}
```

### Terraform Commands

```bash
# Initialize
terraform init

# Format code
terraform fmt -recursive

# Validate configuration
terraform validate

# Plan deployment
terraform plan -var-file="environments/dev.tfvars" -out=tfplan

# Apply
terraform apply tfplan

# Apply with auto-approve (CI/CD)
terraform apply -var-file="environments/dev.tfvars" -auto-approve

# Destroy
terraform destroy -var-file="environments/dev.tfvars"

# Import existing resource
terraform import azurerm_resource_group.example /subscriptions/{sub-id}/resourceGroups/example-rg

# Show state
terraform state list
terraform state show azurerm_resource_group.main

# Move state
terraform state mv azurerm_resource_group.old azurerm_resource_group.new

# Remove from state (without destroying)
terraform state rm azurerm_resource_group.example
```

---

## 14.5 State Management

### Remote State with Azure Storage

```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "myapp/terraform.tfstate"
    use_azuread_auth     = true
  }
}
```

### Setting Up State Storage

```bash
# Create storage for state
az group create --name tfstate-rg --location eastus

az storage account create \
    --name tfstateaccount \
    --resource-group tfstate-rg \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2 \
    --min-tls-version TLS1_2 \
    --allow-blob-public-access false

az storage container create \
    --name tfstate \
    --account-name tfstateaccount

# Enable versioning for state protection
az storage blob service-properties update \
    --account-name tfstateaccount \
    --enable-versioning true
```

### State Locking

```hcl
# Azure Storage provides automatic locking
# When deploying, Terraform creates a lease on the state blob
# This prevents concurrent modifications
```

---

## 14.6 IaC Testing and Validation

### Terraform Testing

```hcl
# tests/main.tftest.hcl
run "verify_resource_group" {
  command = plan

  variables {
    environment = "dev"
    app_name    = "testapp"
    location    = "eastus"
  }

  assert {
    condition     = azurerm_resource_group.main.location == "eastus"
    error_message = "Resource group must be in eastus"
  }
}

run "verify_webapp_https" {
  command = plan

  variables {
    environment = "dev"
    app_name    = "testapp"
    location    = "eastus"
  }

  assert {
    condition     = azurerm_linux_web_app.main.https_only == true
    error_message = "Web app must enforce HTTPS"
  }
}
```

### Bicep What-If

```bash
# Preview changes before deployment
az deployment group what-if \
    --resource-group MyRG \
    --template-file main.bicep \
    --parameters environment=dev
```

### Linting and Validation

```bash
# Terraform
terraform fmt -check -recursive
terraform validate

# Bicep
az bicep build --file main.bicep  # Validates syntax
az bicep lint --file main.bicep   # Linting

# ARM Template
az deployment group validate \
    --resource-group MyRG \
    --template-file template.json
```

---

## 14.7 IaC CI/CD Integration

### Azure DevOps Pipeline for Terraform

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - infrastructure/**

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: terraform-variables
  - name: TF_VERSION
    value: '1.5.0'
  - name: WORKING_DIR
    value: '$(System.DefaultWorkingDirectory)/infrastructure'

stages:
  - stage: Validate
    displayName: 'Validate'
    jobs:
      - job: Validate
        steps:
          - task: TerraformInstaller@0
            displayName: 'Install Terraform'
            inputs:
              terraformVersion: $(TF_VERSION)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: $(WORKING_DIR)
              backendServiceArm: 'AzureServiceConnection'
              backendAzureRmResourceGroupName: '$(TF_STATE_RG)'
              backendAzureRmStorageAccountName: '$(TF_STATE_STORAGE)'
              backendAzureRmContainerName: '$(TF_STATE_CONTAINER)'
              backendAzureRmKey: '$(TF_STATE_KEY)'

          - script: terraform fmt -check -recursive
            displayName: 'Terraform Format Check'
            workingDirectory: $(WORKING_DIR)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Validate'
            inputs:
              provider: 'azurerm'
              command: 'validate'
              workingDirectory: $(WORKING_DIR)

  - stage: Plan
    displayName: 'Plan'
    dependsOn: Validate
    jobs:
      - job: Plan
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: $(TF_VERSION)

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: $(WORKING_DIR)
              backendServiceArm: 'AzureServiceConnection'
              backendAzureRmResourceGroupName: '$(TF_STATE_RG)'
              backendAzureRmStorageAccountName: '$(TF_STATE_STORAGE)'
              backendAzureRmContainerName: '$(TF_STATE_CONTAINER)'
              backendAzureRmKey: '$(TF_STATE_KEY)'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: $(WORKING_DIR)
              environmentServiceNameAzureRM: 'AzureServiceConnection'
              commandOptions: '-var-file=environments/$(Environment).tfvars -out=tfplan'

          - publish: $(WORKING_DIR)/tfplan
            artifact: tfplan
            displayName: 'Publish Plan'

  - stage: Apply
    displayName: 'Apply'
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Apply
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: tfplan

                - task: TerraformInstaller@0
                  inputs:
                    terraformVersion: $(TF_VERSION)

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Init'
                  inputs:
                    provider: 'azurerm'
                    command: 'init'
                    workingDirectory: $(WORKING_DIR)
                    backendServiceArm: 'AzureServiceConnection'
                    backendAzureRmResourceGroupName: '$(TF_STATE_RG)'
                    backendAzureRmStorageAccountName: '$(TF_STATE_STORAGE)'
                    backendAzureRmContainerName: '$(TF_STATE_CONTAINER)'
                    backendAzureRmKey: '$(TF_STATE_KEY)'

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: $(WORKING_DIR)
                    environmentServiceNameAzureRM: 'AzureServiceConnection'
                    commandOptions: '$(Pipeline.Workspace)/tfplan/tfplan'
```

### GitHub Actions for Bicep

```yaml
# .github/workflows/deploy-bicep.yml
name: Deploy Bicep

on:
  push:
    branches: [main]
    paths:
      - 'infrastructure/**'
  pull_request:
    branches: [main]
    paths:
      - 'infrastructure/**'

permissions:
  id-token: write
  contents: read

env:
  AZURE_RESOURCE_GROUP: 'myapp-rg'
  AZURE_LOCATION: 'eastus'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Build Bicep
        run: az bicep build --file infrastructure/main.bicep

      - name: Validate Deployment
        run: |
          az deployment group validate \
            --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
            --template-file infrastructure/main.bicep \
            --parameters environment=dev

  what-if:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: What-If
        run: |
          az deployment group what-if \
            --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
            --template-file infrastructure/main.bicep \
            --parameters environment=dev

  deploy:
    needs: what-if
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy
        run: |
          az deployment group create \
            --resource-group ${{ env.AZURE_RESOURCE_GROUP }} \
            --template-file infrastructure/main.bicep \
            --parameters environment=prod
```

---

## 14.8 Best Practices

### IaC Best Practices

| Category | Best Practice |
|----------|---------------|
| **Version Control** | Store all IaC in Git, use branching strategy |
| **Modularity** | Create reusable modules for common patterns |
| **State Management** | Use remote state with locking |
| **Security** | Never store secrets in code, use Key Vault |
| **Testing** | Validate, lint, and test before applying |
| **Documentation** | Document modules, variables, and outputs |
| **Naming** | Use consistent naming conventions |
| **Tagging** | Apply mandatory tags for cost and management |

### Security Best Practices

```hcl
# Bad - Secret in code
resource "azurerm_mssql_server" "bad" {
  administrator_login_password = "P@ssw0rd123!"  # NEVER DO THIS
}

# Good - Variable with sensitive flag
variable "sql_password" {
  type      = string
  sensitive = true
}

# Better - Reference from Key Vault
data "azurerm_key_vault_secret" "sql_password" {
  name         = "sql-admin-password"
  key_vault_id = data.azurerm_key_vault.main.id
}

resource "azurerm_mssql_server" "good" {
  administrator_login_password = data.azurerm_key_vault_secret.sql_password.value
}
```

---

## 14.9 Summary

### Key Takeaways

1. **Choose the right tool** - ARM for Azure-native, Bicep for cleaner syntax, Terraform for multi-cloud
2. **Use modules** for reusability and consistency
3. **Manage state properly** with remote backends and locking
4. **Test and validate** before applying changes
5. **Integrate with CI/CD** for automated deployments
6. **Follow security practices** - never store secrets in code

---

## Practice Questions

1. What is the difference between ARM templates and Bicep?
2. How do you manage Terraform state for team collaboration?
3. What is the purpose of the What-If operation?
4. How do you create reusable modules in Terraform?
5. What are the deployment modes in ARM templates?
6. How do you reference Key Vault secrets in IaC?
7. Explain the Terraform workflow (init, plan, apply).

---

## Additional Resources

- [ARM Template Documentation](https://docs.microsoft.com/azure/azure-resource-manager/templates/)
- [Bicep Documentation](https://docs.microsoft.com/azure/azure-resource-manager/bicep/)
- [Terraform Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs)
- [Azure DevOps Terraform Extension](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.custom-terraform-tasks)
