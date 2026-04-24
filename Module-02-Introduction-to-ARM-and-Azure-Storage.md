# Module 02 - Introduction to ARM & Azure Storage

## Learning Objectives
By the end of this module, you will be able to:
- Understand Azure Resources and Subscriptions in depth
- Master Azure Resource Manager concepts and operations
- Manage Azure Resources effectively
- Apply and manage Azure Tags
- Create and configure Azure Storage Accounts
- Work with Azure Blob Storage
- Implement Azure Content Delivery Network (CDN)
- Configure Azure Files and Azure File Sync

---

## 2.1 Azure Resources & Subscriptions

### What is an Azure Resource?
An Azure resource is a manageable item that is available through Azure. Examples include:
- Virtual machines
- Storage accounts
- Web apps
- Databases
- Virtual networks
- And many more...

### Resource Identification

Each resource in Azure is uniquely identified by a **Resource ID**:

```
/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{resource-provider}/{resource-type}/{resource-name}
```

**Example:**
```
/subscriptions/12345678-1234-1234-1234-123456789abc/resourceGroups/Production-RG/providers/Microsoft.Compute/virtualMachines/WebServer-VM
```

### Subscription Deep Dive

#### Subscription Purpose
1. **Billing Boundary**: All resources in a subscription are billed together
2. **Access Control Boundary**: RBAC can be applied at subscription level
3. **Organization Unit**: Logical separation of environments/departments

#### Subscription Types

| Type | Description | Best For |
|------|-------------|----------|
| **Free** | $200 credit, limited free services | Learning, Testing |
| **Pay-As-You-Go** | Pay per usage, no commitment | Small workloads |
| **Enterprise Agreement (EA)** | Volume licensing, custom pricing | Large organizations |
| **Cloud Solution Provider (CSP)** | Through Microsoft partner | SMBs |
| **Azure in Open** | Prepaid credits | Flexibility |

##### 🏢 Real-World Subscription Strategy Use Cases:

**Enterprise Corporation - Multi-Subscription Model:**
```
Company: GlobalBank Inc. (50,000 employees)
Challenge: Separate billing, compliance, and access control

Subscription Strategy:
├── Production Subscriptions (3)
│   ├── Core Banking (PCI-DSS compliant, isolated)
│   ├── Customer Portal (internet-facing workloads)
│   └── Internal Apps (employee tools)
├── Non-Production Subscriptions (2)
│   ├── Development (developer sandbox)
│   └── Staging (pre-production testing)
└── Shared Services Subscription (1)
    ├── Hub networking (VPN, ExpressRoute)
    ├── Monitoring (Log Analytics, Sentinel)
    └── Identity (Azure AD Connect, PIM)

Benefits:
- Clear cost allocation per business unit
- Blast radius containment (security breach isolated)
- Different policies per subscription
- Simplified compliance auditing
```

**Startup - Single Subscription Evolution:**
```
Phase 1 (MVP): Single Pay-As-You-Go subscription
Phase 2 (Growth): Add Dev subscription, upgrade to reserved instances
Phase 3 (Enterprise): EA agreement, multiple subscriptions per environment
```

#### Managing Multiple Subscriptions

```
Enterprise Organization
├── Management Group: Production
│   ├── Subscription: Prod-East
│   └── Subscription: Prod-West
├── Management Group: Development
│   ├── Subscription: Dev-App1
│   └── Subscription: Dev-App2
└── Management Group: Testing
    └── Subscription: Testing-All
```

#### Subscription Quotas and Limits

| Resource | Default Limit | Maximum |
|----------|---------------|---------|
| Resource groups | 980 | 980 |
| Deployments per resource group | 800 | 800 |
| Resources per resource group | 800 | 800 |
| Virtual machines per subscription | 25,000 | 25,000 |
| Virtual networks per region | 1,000 | 1,000 |
| Storage accounts per region | 250 | 250 |

```powershell
# View subscription usage and limits
Get-AzVMUsage -Location "East US" | Format-Table

# Using Azure CLI
az vm list-usage --location "eastus" --output table
```

---

## 2.2 Azure Resource Manager

### ARM Architecture Deep Dive

```
┌────────────────────────────────────────────────────────────────┐
│                      User Request                               │
│    (Portal / CLI / PowerShell / SDK / REST API)                │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                    Azure Active Directory                       │
│              (Authentication & Authorization)                   │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                  Azure Resource Manager                         │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐    │
│  │   Request    │ │  Template    │ │    Dependency        │    │
│  │  Validation  │ │  Processing  │ │    Resolution        │    │
│  └──────────────┘ └──────────────┘ └──────────────────────┘    │
└────────────────────────────┬───────────────────────────────────┘
                             │
                             ▼
┌────────────────────────────────────────────────────────────────┐
│                    Resource Providers                           │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │Microsoft.Compute │  │Microsoft.Storage │  │Microsoft.Web │  │
│  │  (VMs, Disks)    │  │(Storage Accounts)│  │ (App Service)│  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

### Resource Providers

Resource providers are services that supply Azure resources.

#### Common Resource Providers

| Provider | Resources | Description |
|----------|-----------|-------------|
| Microsoft.Compute | virtualMachines, disks | Compute resources |
| Microsoft.Storage | storageAccounts | Storage services |
| Microsoft.Network | virtualNetworks, publicIPAddresses | Networking |
| Microsoft.Web | sites, serverFarms | Web applications |
| Microsoft.Sql | servers, databases | SQL databases |
| Microsoft.KeyVault | vaults | Secret management |

#### Managing Resource Providers

```powershell
# List all registered providers
Get-AzResourceProvider | Select-Object ProviderNamespace, RegistrationState

# Register a resource provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.Batch"

# Check provider registration status
(Get-AzResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState
```

```bash
# Azure CLI
az provider list --output table
az provider register --namespace Microsoft.Batch
az provider show --namespace Microsoft.Batch --query "registrationState"
```

### ARM Template Deployment Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| **Incremental** (default) | Leaves existing resources unchanged | Adding new resources |
| **Complete** | Deletes resources not in template | Full environment deployment |

```powershell
# Incremental deployment (default)
New-AzResourceGroupDeployment `
  -ResourceGroupName "MyRG" `
  -TemplateFile "template.json" `
  -Mode Incremental

# Complete deployment (destructive!)
New-AzResourceGroupDeployment `
  -ResourceGroupName "MyRG" `
  -TemplateFile "template.json" `
  -Mode Complete
```

### What-If Deployment

Preview changes before actual deployment:

```powershell
# Preview deployment changes
New-AzResourceGroupDeployment `
  -ResourceGroupName "MyRG" `
  -TemplateFile "template.json" `
  -WhatIf
```

---

## 2.3 Managing Azure Resources

### Resource Group Operations

#### Creating Resource Groups

```powershell
# PowerShell
New-AzResourceGroup -Name "Production-RG" -Location "East US" -Tag @{Environment="Production"; Owner="IT"}
```

```bash
# Azure CLI
az group create --name "Production-RG" --location "eastus" --tags Environment=Production Owner=IT
```

#### Listing Resources

```powershell
# List all resources in a resource group
Get-AzResource -ResourceGroupName "Production-RG" | Format-Table Name, ResourceType, Location

# Filter by resource type
Get-AzResource -ResourceGroupName "Production-RG" -ResourceType "Microsoft.Compute/virtualMachines"
```

```bash
# Azure CLI
az resource list --resource-group "Production-RG" --output table
az resource list --resource-group "Production-RG" --resource-type "Microsoft.Compute/virtualMachines"
```

### Moving Resources Between Resource Groups

#### Rules for Moving Resources
1. Both source and destination must be locked during move
2. Resource provider must be registered in destination subscription
3. Not all resources support move operations
4. Source and destination can be different subscriptions

#### Resources That CAN Be Moved
- Virtual Machines (with all dependencies)
- Storage Accounts
- Virtual Networks
- SQL Databases
- App Service plans and apps

#### Resources That CANNOT Be Moved
- Azure Active Directory Domain Services
- Azure Backup vaults
- Azure DevOps organization resources
- Classic deployment resources

```powershell
# Move resources to another resource group
$resource = Get-AzResource -Name "MyVM" -ResourceGroupName "Source-RG"
Move-AzResource -DestinationResourceGroupName "Destination-RG" -ResourceId $resource.ResourceId
```

```bash
# Azure CLI
az resource move --destination-group "Destination-RG" --ids "/subscriptions/{sub-id}/resourceGroups/Source-RG/providers/Microsoft.Compute/virtualMachines/MyVM"
```

### Deleting Resources

```powershell
# Delete a single resource
Remove-AzResource -ResourceId "/subscriptions/{sub-id}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM"

# Delete entire resource group
Remove-AzResourceGroup -Name "MyRG" -Force
```

### Resource Locks

Prevent accidental deletion or modification:

| Lock Type | Effect |
|-----------|--------|
| **CanNotDelete** | Resources can be read and modified, but not deleted |
| **ReadOnly** | Resources can only be read, no modifications allowed |

```powershell
# Create a delete lock
New-AzResourceLock `
  -LockName "PreventDelete" `
  -LockLevel CanNotDelete `
  -ResourceGroupName "Production-RG" `
  -LockNotes "Prevent accidental deletion of production resources"

# Create a read-only lock
New-AzResourceLock `
  -LockName "ReadOnlyLock" `
  -LockLevel ReadOnly `
  -ResourceGroupName "Production-RG"

# View locks
Get-AzResourceLock -ResourceGroupName "Production-RG"

# Remove a lock
Remove-AzResourceLock -LockName "PreventDelete" -ResourceGroupName "Production-RG"
```

---

## 2.4 Azure Tags

### What Are Tags?
Tags are metadata elements (key-value pairs) that you apply to Azure resources for organization and tracking.

### Tag Use Cases

| Use Case | Example Tag |
|----------|-------------|
| Cost allocation | `CostCenter: Marketing` |
| Environment identification | `Environment: Production` |
| Ownership | `Owner: John.Doe@company.com` |
| Application grouping | `Application: ERP-System` |
| Automation | `AutoShutdown: True` |

### Tag Limits

| Limit | Value |
|-------|-------|
| Tags per resource | 50 |
| Tag name max length | 512 characters |
| Tag value max length | 256 characters |
| Tag name (storage accounts) | 128 characters |
| Tag value (storage accounts) | 256 characters |

### Managing Tags

#### Apply Tags During Resource Creation

```powershell
# PowerShell - Create resource with tags
New-AzResourceGroup -Name "Tagged-RG" -Location "East US" `
  -Tag @{
    Environment = "Production"
    CostCenter = "IT-001"
    Owner = "admin@company.com"
    Application = "CRM"
  }
```

```bash
# Azure CLI
az group create --name "Tagged-RG" --location "eastus" \
  --tags Environment=Production CostCenter=IT-001 Owner=admin@company.com Application=CRM
```

#### Update Tags on Existing Resources

```powershell
# Get existing resource
$resource = Get-AzResource -Name "MyVM" -ResourceGroupName "MyRG"

# Add/Update tags (merge with existing)
$tags = $resource.Tags
$tags["Status"] = "Active"
$tags["LastUpdated"] = (Get-Date).ToString()
Set-AzResource -ResourceId $resource.ResourceId -Tag $tags -Force

# Replace all tags
$newTags = @{Environment = "Dev"; Project = "NewProject"}
Set-AzResource -ResourceId $resource.ResourceId -Tag $newTags -Force
```

```bash
# Azure CLI - Merge tags
az resource tag --tags Status=Active --ids "/subscriptions/.../resources/MyVM"

# Replace all tags
az resource tag --tags Environment=Dev Project=NewProject --ids "/subscriptions/.../resources/MyVM"
```

#### Remove Tags

```powershell
# Remove specific tag
$resource = Get-AzResource -Name "MyVM" -ResourceGroupName "MyRG"
$tags = $resource.Tags
$tags.Remove("Status")
Set-AzResource -ResourceId $resource.ResourceId -Tag $tags -Force

# Remove all tags
Set-AzResource -ResourceId $resource.ResourceId -Tag @{} -Force
```

#### Query Resources by Tags

```powershell
# Find all resources with specific tag
Get-AzResource -Tag @{Environment = "Production"}

# Find all resources with any value for a tag key
Get-AzResource -TagName "CostCenter"
```

```bash
# Azure CLI
az resource list --tag Environment=Production --output table
```

### Tag Inheritance
- Tags are NOT inherited from resource groups to resources
- Use Azure Policy to enforce tag inheritance

### Azure Policy for Tags

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "field": "[concat('tags[', parameters('tagName'), ']')]",
      "exists": "false"
    },
    "then": {
      "effect": "deny"
    }
  },
  "parameters": {
    "tagName": {
      "type": "String",
      "metadata": {
        "displayName": "Tag Name",
        "description": "Name of the tag required on resources"
      }
    }
  }
}
```

---

## 2.5 Azure Storage Account & Its Types

### What is a Storage Account?
An Azure Storage Account provides a unique namespace in Azure for your data. Every object you store has an address that includes your unique account name.

### Storage Account Naming Rules
- Must be between 3 and 24 characters
- Can contain only lowercase letters and numbers
- Must be globally unique across all of Azure

### Storage Account Types

| Type | Supported Services | Performance | Use Case |
|------|-------------------|-------------|----------|
| **General Purpose v2 (GPv2)** | Blob, File, Queue, Table, Data Lake | Standard/Premium | Most scenarios |
| **General Purpose v1 (GPv1)** | Blob, File, Queue, Table | Standard/Premium | Legacy |
| **BlockBlobStorage** | Block blobs, Append blobs | Premium | High transaction rates |
| **FileStorage** | File shares only | Premium | Enterprise file shares |
| **BlobStorage** | Block blobs, Append blobs | Standard | Legacy blob storage |

### Performance Tiers

| Tier | Storage Type | Use Case |
|------|-------------|----------|
| **Standard** | HDD-based | Cost-effective, backup, archival |
| **Premium** | SSD-based | Low latency, high IOPS |

### Access Tiers (for Blob Storage)

| Tier | Description | Storage Cost | Access Cost |
|------|-------------|--------------|-------------|
| **Hot** | Frequently accessed data | Highest | Lowest |
| **Cool** | Infrequently accessed (30+ days) | Lower | Higher |
| **Cold** | Rarely accessed (90+ days) | Even lower | Even higher |
| **Archive** | Rarely accessed, hours to retrieve | Lowest | Highest |

### Creating a Storage Account

```powershell
# Create a General Purpose v2 Storage Account
New-AzStorageAccount `
  -ResourceGroupName "Storage-RG" `
  -Name "mystorageaccount123" `
  -Location "East US" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot" `
  -EnableHttpsTrafficOnly $true `
  -MinimumTlsVersion "TLS1_2"
```

```bash
# Azure CLI
az storage account create \
  --name "mystorageaccount123" \
  --resource-group "Storage-RG" \
  --location "eastus" \
  --sku "Standard_LRS" \
  --kind "StorageV2" \
  --access-tier "Hot" \
  --https-only true \
  --min-tls-version "TLS1_2"
```

### Storage Account Endpoints

| Service | Endpoint Format |
|---------|-----------------|
| Blob | `https://<account>.blob.core.windows.net` |
| Data Lake | `https://<account>.dfs.core.windows.net` |
| File | `https://<account>.file.core.windows.net` |
| Queue | `https://<account>.queue.core.windows.net` |
| Table | `https://<account>.table.core.windows.net` |

---

## 2.6 Azure Blob Storage

### What is Blob Storage?
Azure Blob Storage is Microsoft's object storage solution for the cloud. Optimized for storing massive amounts of unstructured data.

### Blob Storage Structure

```
Storage Account
└── Container (similar to a folder)
    ├── Blob (file)
    ├── Blob
    └── Virtual Directory
        └── Blob
```

### Types of Blobs

| Type | Description | Use Case | Max Size |
|------|-------------|----------|----------|
| **Block Blobs** | Store text and binary data | Documents, images, videos | ~190.7 TB |
| **Append Blobs** | Optimized for append operations | Log files, audit trails | ~195 GB |
| **Page Blobs** | Random access files | VHD files for VMs | 8 TB |

### Container Access Levels

| Level | Access |
|-------|--------|
| **Private** | No anonymous access (default) |
| **Blob** | Anonymous read access for blobs only |
| **Container** | Anonymous read access for container and blobs |

### Working with Blob Storage

#### Create Container

```powershell
# Get storage account context
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123").Context

# Create container
New-AzStorageContainer -Name "documents" -Context $context -Permission Off
```

```bash
# Azure CLI
az storage container create \
  --name "documents" \
  --account-name "mystorageaccount123" \
  --public-access off
```

#### Upload Blobs

```powershell
# Upload a single file
Set-AzStorageBlobContent `
  -File "C:\Files\report.pdf" `
  -Container "documents" `
  -Blob "reports/2024/report.pdf" `
  -Context $context

# Upload all files from a directory
Get-ChildItem -Path "C:\Files\*.txt" | ForEach-Object {
    Set-AzStorageBlobContent -File $_.FullName -Container "documents" -Context $context
}
```

```bash
# Azure CLI
az storage blob upload \
  --file "report.pdf" \
  --container-name "documents" \
  --name "reports/2024/report.pdf" \
  --account-name "mystorageaccount123"

# Upload entire directory
az storage blob upload-batch \
  --destination "documents" \
  --source "./local-folder" \
  --account-name "mystorageaccount123"
```

#### Download Blobs

```powershell
# Download a blob
Get-AzStorageBlobContent `
  -Container "documents" `
  -Blob "reports/2024/report.pdf" `
  -Destination "C:\Downloads\" `
  -Context $context
```

#### List Blobs

```powershell
# List all blobs in container
Get-AzStorageBlob -Container "documents" -Context $context
```

### Blob Lifecycle Management

Automate transitioning blobs between access tiers:

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["documents/"]
        }
      }
    }
  ]
}
```

---

## 2.7 Azure Content Delivery Network (CDN)

### What is Azure CDN?
Azure CDN is a distributed network of servers that can efficiently deliver web content to users by caching content at edge locations around the world.

### How CDN Works

```
User Request                    
    │                           
    ▼                           
┌─────────┐     Cache Miss    ┌──────────────┐
│   CDN   │ ─────────────────→│    Origin    │
│  Edge   │                   │   Server     │
│  Node   │ ←─────────────────│  (Storage/   │
└─────────┘    Return Data    │   Web App)   │
    │                         └──────────────┘
    │ Cache Hit
    ▼
┌─────────┐
│  User   │
└─────────┘
```

### CDN Providers in Azure

| Provider | Features |
|----------|----------|
| **Microsoft CDN** | Integrated, simple, cost-effective |
| **Akamai** | Enterprise-grade, extensive features |
| **Verizon** | High performance, advanced optimizations |

### CDN Concepts

| Term | Description |
|------|-------------|
| **CDN Profile** | Container for CDN endpoints |
| **CDN Endpoint** | The URL where your content is served |
| **Origin** | Source of your content (storage, web app) |
| **POP** | Point of Presence - edge server location |
| **Rules Engine** | Customize CDN behavior |

### Creating CDN

#### Step 1: Create CDN Profile

```powershell
# Create CDN Profile
New-AzCdnProfile `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -Sku "Standard_Microsoft" `
  -Location "Global"
```

#### Step 2: Create CDN Endpoint

```powershell
# Create endpoint pointing to blob storage
New-AzCdnEndpoint `
  -EndpointName "mycontentendpoint" `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -Location "Global" `
  -OriginHostName "mystorageaccount.blob.core.windows.net" `
  -OriginName "StorageOrigin"
```

### CDN Caching Behavior

| Caching Rule | Behavior |
|--------------|----------|
| **Ignore query string** | Cache serves same content regardless of query string |
| **Bypass cache for query string** | No caching when query string present |
| **Cache every unique URL** | Each query string variation is cached separately |

### Cache Purging

```powershell
# Purge specific content
Unpublish-AzCdnEndpointContent `
  -EndpointName "mycontentendpoint" `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -ContentPath @("/images/*", "/css/style.css")

# Purge all content
Unpublish-AzCdnEndpointContent `
  -EndpointName "mycontentendpoint" `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -ContentPath @("/*")
```

---

## 2.8 Azure Files Storage

### What is Azure Files?
Azure Files offers fully managed file shares in the cloud that are accessible via SMB (Server Message Block) and NFS (Network File System) protocols.

### Key Features
- SMB 3.0 and SMB 2.1 support
- NFS 4.1 support (Linux)
- Can be mounted on Windows, Linux, macOS
- Supports Azure AD authentication
- Snapshot capability

### Azure Files vs. Blob Storage

| Feature | Azure Files | Blob Storage |
|---------|-------------|--------------|
| Protocol | SMB, NFS | REST API |
| Use case | Lift-and-shift, shared files | Unstructured data |
| Mounting | Yes (as network drive) | No |
| Structure | Hierarchical (files/folders) | Flat (with virtual folders) |

### Share Types

| Type | Performance | Best For |
|------|-------------|----------|
| **Standard** | HDD, up to 5,000 IOPS | General file sharing |
| **Premium** | SSD, up to 100,000 IOPS | High-performance workloads |

### Creating File Share

```powershell
# Create file share
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123").Context

New-AzStorageShare `
  -Name "documents" `
  -Context $context `
  -QuotaGiB 100
```

```bash
# Azure CLI
az storage share create \
  --name "documents" \
  --account-name "mystorageaccount123" \
  --quota 100
```

### Mounting Azure File Share

#### Windows

```powershell
# Mount as network drive
$connectTestResult = Test-NetConnection -ComputerName mystorageaccount123.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded) {
    cmd.exe /C "cmdkey /add:`"mystorageaccount123.file.core.windows.net`" /user:`"Azure\mystorageaccount123`" /pass:`"<storage-account-key>`""
    New-PSDrive -Name Z -PSProvider FileSystem -Root "\\mystorageaccount123.file.core.windows.net\documents" -Persist
}
```

#### Linux

```bash
# Install CIFS utilities
sudo apt-get install cifs-utils

# Create mount point
sudo mkdir /mnt/azurefiles

# Mount the share
sudo mount -t cifs //mystorageaccount123.file.core.windows.net/documents /mnt/azurefiles -o vers=3.0,username=mystorageaccount123,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

### File Share Snapshots

```powershell
# Create snapshot
$share = Get-AzStorageShare -Name "documents" -Context $context
$snapshot = $share.CloudFileShare.Snapshot()

# List snapshots
Get-AzStorageShare -Name "documents" -Context $context -SnapshotTime $snapshot.SnapshotTime
```

---

## 2.9 Azure File Sync

### What is Azure File Sync?
Azure File Sync enables centralizing file shares in Azure Files while maintaining compatibility with Windows File Server. It transforms Windows Server into a quick cache of your Azure file share.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure File Sync                          │
│                                                             │
│  ┌───────────────┐        ┌───────────────┐                │
│  │ On-premises   │        │   Azure       │                │
│  │ Windows       │◄──────►│   File        │                │
│  │ Server        │  Sync  │   Share       │                │
│  └───────────────┘        └───────────────┘                │
│         │                                                   │
│         ▼                                                   │
│  ┌───────────────┐                                         │
│  │ Azure File    │                                         │
│  │ Sync Agent    │                                         │
│  └───────────────┘                                         │
└─────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Storage Sync Service** | Azure resource that manages sync relationships |
| **Sync Group** | Defines sync topology (which endpoints sync together) |
| **Registered Server** | On-premises server with File Sync agent |
| **Server Endpoint** | Local folder to sync |
| **Cloud Endpoint** | Azure file share to sync to |

### Cloud Tiering

Cloud tiering is an optional feature that moves frequently accessed files to local storage while keeping rarely accessed files in the cloud.

**Benefits:**
- Reduce on-premises storage requirements
- Maintain full namespace locally
- Files appear local but stored in Azure

**Tiering Policies:**
- **Volume free space policy**: Keep X% of volume free
- **Date policy**: Tier files not accessed in X days

### Setting Up Azure File Sync

#### Step 1: Create Storage Sync Service

```powershell
# Create Storage Sync Service
New-AzStorageSyncService `
  -ResourceGroupName "FileSync-RG" `
  -Name "MyStorageSyncService" `
  -Location "East US"
```

#### Step 2: Create Sync Group

```powershell
# Create Sync Group
New-AzStorageSyncGroup `
  -ResourceGroupName "FileSync-RG" `
  -StorageSyncServiceName "MyStorageSyncService" `
  -Name "MySyncGroup"
```

#### Step 3: Add Cloud Endpoint (Azure File Share)

```powershell
# Add cloud endpoint
New-AzStorageSyncCloudEndpoint `
  -ResourceGroupName "FileSync-RG" `
  -StorageSyncServiceName "MyStorageSyncService" `
  -SyncGroupName "MySyncGroup" `
  -StorageAccountResourceId "/subscriptions/{sub}/resourceGroups/Storage-RG/providers/Microsoft.Storage/storageAccounts/mystorageaccount123" `
  -AzureFileShareName "documents"
```

#### Step 4: Install Agent on Windows Server
1. Download Azure File Sync agent from Microsoft
2. Install on Windows Server
3. Register server with Storage Sync Service

#### Step 5: Add Server Endpoint

```powershell
# Add server endpoint
New-AzStorageSyncServerEndpoint `
  -ResourceGroupName "FileSync-RG" `
  -StorageSyncServiceName "MyStorageSyncService" `
  -SyncGroupName "MySyncGroup" `
  -ServerResourceId "/subscriptions/{sub}/resourceGroups/FileSync-RG/providers/Microsoft.StorageSync/storageSyncServices/MyStorageSyncService/registeredServers/{server-id}" `
  -Path "D:\FileShare" `
  -CloudTieringEnabled
```

---

## Hands-on Exercises

### Exercise 1: Manage Resource Groups in Azure

```powershell
# Create multiple resource groups
$locations = @("East US", "West US", "North Europe")
$environments = @("Dev", "Test", "Prod")

foreach ($env in $environments) {
    New-AzResourceGroup `
      -Name "$env-RG" `
      -Location $locations[0] `
      -Tag @{Environment = $env; CreatedBy = "Admin"}
}

# Verify creation
Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -match "RG$"} | Format-Table
```

### Exercise 2: Move Resource Between Resource Groups

```powershell
# Create source resource group with a storage account
New-AzResourceGroup -Name "Source-RG" -Location "East US"
New-AzStorageAccount -ResourceGroupName "Source-RG" -Name "sourcestorageacc123" -Location "East US" -SkuName "Standard_LRS"

# Create destination resource group
New-AzResourceGroup -Name "Destination-RG" -Location "East US"

# Get resource ID
$resource = Get-AzResource -Name "sourcestorageacc123" -ResourceGroupName "Source-RG"

# Move the resource
Move-AzResource -DestinationResourceGroupName "Destination-RG" -ResourceId $resource.ResourceId

# Verify
Get-AzResource -ResourceGroupName "Destination-RG"
```

### Exercise 3: Apply Tags

```powershell
# Apply tags to a resource group
$tags = @{
    Environment = "Production"
    CostCenter = "IT-001"
    Owner = "admin@company.com"
    Project = "ERP"
    AutoShutdown = "False"
}

Set-AzResourceGroup -Name "Production-RG" -Tag $tags

# Apply tags to all resources in the group
$rg = Get-AzResourceGroup -Name "Production-RG"
Get-AzResource -ResourceGroupName $rg.ResourceGroupName | ForEach-Object {
    Set-AzResource -ResourceId $_.ResourceId -Tag $rg.Tags -Force
}
```

### Exercise 4: Create Storage Account

```powershell
# Create a secure storage account
New-AzStorageAccount `
  -ResourceGroupName "Storage-RG" `
  -Name "securestorage$(Get-Random -Maximum 9999)" `
  -Location "East US" `
  -SkuName "Standard_GRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot" `
  -EnableHttpsTrafficOnly $true `
  -MinimumTlsVersion "TLS1_2" `
  -AllowBlobPublicAccess $false `
  -Tag @{Environment = "Production"}
```

### Exercise 5: Access Storage Account

```powershell
# Get storage context using account key
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123"
$context = $storageAccount.Context

# OR get context with key
$key = (Get-AzStorageAccountKey -ResourceGroupName "Storage-RG" -Name "mystorageaccount123")[0].Value
$context = New-AzStorageContext -StorageAccountName "mystorageaccount123" -StorageAccountKey $key
```

### Exercise 6: Create Blob Storage

```powershell
# Create container
New-AzStorageContainer -Name "mycontainer" -Context $context -Permission Off

# Upload a file
Set-AzStorageBlobContent -File "C:\temp\test.txt" -Container "mycontainer" -Blob "test.txt" -Context $context
```

### Exercise 7: Upload to Blob Storage

```powershell
# Upload multiple files
Get-ChildItem "C:\Documents\*.pdf" | ForEach-Object {
    Set-AzStorageBlobContent `
      -File $_.FullName `
      -Container "documents" `
      -Blob "pdfs/$($_.Name)" `
      -Context $context
}

# Verify uploads
Get-AzStorageBlob -Container "documents" -Context $context | Format-Table Name, Length, LastModified
```

### Exercise 8: Create a File Share

```powershell
# Create file share
New-AzStorageShare -Name "companyfiles" -Context $context -QuotaGiB 50

# Create directories
New-AzStorageDirectory -ShareName "companyfiles" -Path "HR" -Context $context
New-AzStorageDirectory -ShareName "companyfiles" -Path "Finance" -Context $context

# Upload files
Set-AzStorageFileContent -ShareName "companyfiles" -Source "C:\files\policy.pdf" -Path "HR/policy.pdf" -Context $context
```

### Exercise 9: Creating and Using CDN Endpoint

```powershell
# Create CDN Profile
New-AzCdnProfile `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -Sku "Standard_Microsoft" `
  -Location "Global"

# Create CDN Endpoint
New-AzCdnEndpoint `
  -EndpointName "mywebcontent" `
  -ProfileName "MyCDNProfile" `
  -ResourceGroupName "CDN-RG" `
  -Location "Global" `
  -OriginHostName "mystorageaccount123.blob.core.windows.net" `
  -OriginPath "/public" `
  -OriginName "BlobOrigin"

# Access content via CDN
# URL: https://mywebcontent.azureedge.net/images/logo.png
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Resources & Subscriptions | Unique Resource IDs, subscription limits, billing boundaries |
| Azure Resource Manager | Central management layer, resource providers, deployment modes |
| Managing Resources | CRUD operations, moving resources, resource locks |
| Azure Tags | Key-value metadata, 50 tags per resource, cost tracking |
| Storage Account | GPv2 recommended, Standard/Premium tiers, Hot/Cool/Archive access |
| Blob Storage | Block/Append/Page blobs, containers, lifecycle management |
| Azure CDN | Edge caching, multiple providers, cache purging |
| Azure Files | SMB/NFS file shares, Windows/Linux mounting |
| Azure File Sync | Hybrid sync, cloud tiering, centralized file management |

---

## Review Questions

1. What is the maximum number of tags you can apply to a resource?
2. What are the two deployment modes in ARM? When would you use each?
3. Explain the difference between Block blobs and Page blobs.
4. What are the three access tiers for blob storage?
5. How does Azure CDN improve application performance?
6. What is the purpose of cloud tiering in Azure File Sync?

---

## Additional Resources

- [Azure Storage Documentation](https://docs.microsoft.com/azure/storage/)
- [ARM Template Reference](https://docs.microsoft.com/azure/azure-resource-manager/templates/)
- [Azure CDN Documentation](https://docs.microsoft.com/azure/cdn/)
- [Azure File Sync Planning Guide](https://docs.microsoft.com/azure/storage/file-sync/file-sync-planning)

---

## 2.6 Advanced ARM Templates and Deployment Strategies (Expert Level)

### ARM Template Best Practices

#### Modular Template Design

```json
// main.bicep (using Bicep for cleaner syntax)
targetScope = 'subscription'

param environment string
param location string = 'eastus'

// Resource Group
resource rg 'Microsoft.Resources/resourceGroups@2021-04-01' = {
  name: 'rg-${environment}'
  location: location
  tags: {
    Environment: environment
    ManagedBy: 'Bicep'
  }
}

// Deploy networking module
module networking 'modules/networking.bicep' = {
  name: 'networking-deployment'
  scope: rg
  params: {
    environment: environment
    location: location
  }
}

// Deploy compute module
module compute 'modules/compute.bicep' = {
  name: 'compute-deployment'
  scope: rg
  params: {
    environment: environment
    location: location
    subnetId: networking.outputs.subnetId
  }
  dependsOn: [
    networking
  ]
}

output vnetId string = networking.outputs.vnetId
```

#### Template Specs for Enterprise

```powershell
# Create a Template Spec for reusable deployments
$templateSpec = New-AzTemplateSpec `
    -ResourceGroupName "TemplateSpecs-RG" `
    -Name "WebAppTemplate" `
    -Version "1.0.0" `
    -Location "eastus" `
    -TemplateFile "webapp.bicep" `
    -Description "Standard web application deployment"

# Deploy from Template Spec
New-AzResourceGroupDeployment `
    -ResourceGroupName "Production-RG" `
    -TemplateSpecId $templateSpec.Versions[0].Id `
    -TemplateParameterObject @{
        appName = "mywebapp"
        environment = "prod"
    }
```

### Deployment Strategies

#### Blue-Green Deployments

```powershell
# Deploy to staging slot
az webapp deployment slot create `
    --name "myapp" `
    --resource-group "WebApp-RG" `
    --slot "staging"

# Deploy new version to staging
az webapp deployment source config-zip `
    --name "myapp" `
    --resource-group "WebApp-RG" `
    --slot "staging" `
    --src "newversion.zip"

# Warm up staging slot
$stagingUrl = "https://myapp-staging.azurewebsites.net/health"
Invoke-WebRequest -Uri $stagingUrl -UseBasicParsing

# Swap slots (zero-downtime deployment)
az webapp deployment slot swap `
    --name "myapp" `
    --resource-group "WebApp-RG" `
    --slot "staging"
```

#### Canary Deployments with Traffic Manager

```powershell
# Create Traffic Manager profile with weighted routing
New-AzTrafficManagerProfile `
    -Name "myapp-tm" `
    -ResourceGroupName "WebApp-RG" `
    -TrafficRoutingMethod Weighted `
    -RelativeDnsName "myapp" `
    -Ttl 30 `
    -MonitorProtocol HTTPS `
    -MonitorPort 443 `
    -MonitorPath "/health"

# Add endpoints with weights (90% to v1, 10% to v2)
New-AzTrafficManagerEndpoint `
    -Name "v1-endpoint" `
    -ProfileName "myapp-tm" `
    -ResourceGroupName "WebApp-RG" `
    -Type AzureEndpoints `
    -TargetResourceId $webAppV1.Id `
    -Weight 90

New-AzTrafficManagerEndpoint `
    -Name "v2-endpoint" `
    -ProfileName "myapp-tm" `
    -ResourceGroupName "WebApp-RG" `
    -Type AzureEndpoints `
    -TargetResourceId $webAppV2.Id `
    -Weight 10
```

### Azure Resource Graph for Advanced Queries

```powershell
# Install Resource Graph module
Install-Module -Name Az.ResourceGraph

# Find all resources without required tags
$query = @"
resources
| where tags !contains 'CostCenter' or tags !contains 'Owner'
| project name, type, resourceGroup, subscriptionId
| order by type asc
"@
Search-AzGraph -Query $query

# Find VMs not using managed disks
$query = @"
resources
| where type == 'microsoft.compute/virtualmachines'
| where properties.storageProfile.osDisk.managedDisk == ''
| project name, resourceGroup, location
"@
Search-AzGraph -Query $query

# Aggregate resources by type across subscriptions
$query = @"
resources
| summarize count() by type
| order by count_ desc
| take 20
"@
Search-AzGraph -Query $query -Subscription $allSubscriptions
```

### Storage Account Security Hardening

```powershell
# Create secure storage account with all security features
$storageParams = @{
    ResourceGroupName      = "Storage-RG"
    Name                   = "mysecurestorage"
    Location               = "eastus"
    SkuName                = "Standard_GRS"
    Kind                   = "StorageV2"
    AccessTier             = "Hot"
    MinimumTlsVersion      = "TLS1_2"
    EnableHttpsTrafficOnly = $true
    AllowBlobPublicAccess  = $false
    EnableNfsV3            = $false
}
$storage = New-AzStorageAccount @storageParams

# Enable infrastructure encryption (double encryption)
Set-AzStorageAccount `
    -ResourceGroupName "Storage-RG" `
    -Name "mysecurestorage" `
    -RequireInfrastructureEncryption

# Configure firewall rules
Update-AzStorageAccountNetworkRuleSet `
    -ResourceGroupName "Storage-RG" `
    -Name "mysecurestorage" `
    -DefaultAction Deny `
    -Bypass AzureServices

# Add allowed IP ranges
Add-AzStorageAccountNetworkRule `
    -ResourceGroupName "Storage-RG" `
    -Name "mysecurestorage" `
    -IPAddressOrRange "203.0.113.0/24"

# Enable Private Endpoint
$privateEndpoint = New-AzPrivateEndpoint `
    -ResourceGroupName "Storage-RG" `
    -Name "pe-mysecurestorage" `
    -Location "eastus" `
    -Subnet $subnet `
    -PrivateLinkServiceConnection $privateConnections

# Enable soft delete for blobs (ransomware protection)
Enable-AzStorageBlobDeleteRetentionPolicy `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "mysecurestorage" `
    -RetentionDays 30

# Enable versioning
Update-AzStorageBlobServiceProperty `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "mysecurestorage" `
    -IsVersioningEnabled $true
```
