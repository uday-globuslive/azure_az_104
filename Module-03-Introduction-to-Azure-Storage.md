# Module 03 - Introduction to Azure Storage (Deep Dive)

## Learning Objectives
By the end of this module, you will be able to:
- Work with Azure Table Storage for NoSQL data
- Implement Azure Queue Storage for messaging
- Use Azure Storage Explorer effectively
- Create and manage Shared Access Signatures (SAS)
- Understand Azure Data Box for large data transfers
- Configure storage replication options
- Use Azure Import/Export service

---

## 3.1 Azure Table Storage

### What is Azure Table Storage?
Azure Table Storage is a NoSQL key-value store for rapid development using massive semi-structured datasets. It offers schemaless design, allowing flexibility in data storage.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Schema-less** | Each entity can have different properties |
| **Highly scalable** | Petabytes of structured data |
| **Cost-effective** | Low cost per GB |
| **Fast access** | Indexed by PartitionKey and RowKey |

#### 🏢 Real-World Azure Storage Use Cases:

**IoT Telemetry Platform:**
```
Company: SmartFactory Inc. (Manufacturing IoT)
Challenge: Store 100M sensor readings per day

Storage Solution:
├── Table Storage for Telemetry
│   ├── PartitionKey: DeviceId + Date (e.g., "sensor001_20240115")
│   ├── RowKey: Timestamp (ticks)
│   └── Properties: Temperature, Humidity, Pressure, etc.
│
├── Data Pattern:
│   └── One partition per device per day
│   └── Query: "Get sensor001 data for Jan 15" → 1 partition scan
│
├── Cost Analysis:
│   └── 100M entities × 1KB = 100GB/day
│   └── Hot data (30 days): 3TB in Table Storage
│   └── Cold data: Lifecycle to Archive tier
│   └── Monthly cost: ~$150 vs. $3,000 for equivalent SQL

Benefits:
- Sub-second queries for single device
- No schema changes needed for new sensor types
- 10x cheaper than relational database
```

**User Session Storage (Gaming):**
```
Company: MobileGames Ltd.
Challenge: Store 50M player sessions across 100+ game servers

Table Storage Design:
PartitionKey: PlayerId (distributed across servers)
RowKey: SessionId

Example Entity:
{
  "PartitionKey": "player_abc123",
  "RowKey": "session_2024011512345",
  "GameLevel": 45,
  "Score": 125000,
  "Inventory": "[sword,shield,potion]",
  "LastCheckpoint": "level5_boss",
  "Timestamp": "2024-01-15T10:30:00Z"
}

Why Table Storage:
- Schema-less: Each game has different attributes
- Fast: 20ms latency for player lookup
- Cheap: $0.045/GB vs. $0.20/GB for SQL
```

**Blob Storage - Media Platform:**
```
Company: NewsMedia Corp
Challenge: Store and serve 10PB of video content

Architecture:
├── Hot Tier (Frequently accessed - last 7 days)
│   └── 500TB of recent videos
│   └── CDN integration for streaming
│
├── Cool Tier (Archival - 7-90 days)
│   └── 2PB of older content
│   └── Still accessible in milliseconds
│
├── Archive Tier (Rarely accessed - 90+ days)
│   └── 7.5PB historical footage
│   └── Rehydration: 1-15 hours
│   └── Cost: $0.00099/GB/month

Lifecycle Policy:
blob.mp4 → [Day 0: Hot] → [Day 7: Cool] → [Day 90: Archive]

Monthly Savings: $150,000 vs. keeping all data Hot
```

**Queue Storage - Order Processing:**
```
Company: FastDelivery (Food delivery app)
Challenge: Handle 50,000 orders/hour peak

Queue Architecture:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ Mobile App  │───►│ Order Queue │───►│ Processors  │
│ (Producer)  │    │ (Buffer)    │    │ (Consumers) │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                   Message Example:
                   {
                     "orderId": "ORD-123",
                     "restaurant": "PizzaPlace",
                     "items": ["pizza", "soda"],
                     "deliveryAddress": "123 Main St"
                   }

Benefits:
- Peak handling: Queue absorbs 10x spike
- Retry logic: Failed orders re-queued
- Scaling: Add processors during lunch rush
- Decoupling: App doesn't wait for processing
```

### Table Storage Structure

```
┌─────────────────────────────────────────────────────────────┐
│                     Storage Account                          │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                      Table                           │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │                   Entity (Row)                 │  │    │
│  │  │  PartitionKey | RowKey | Properties...        │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  │  ┌───────────────────────────────────────────────┐  │    │
│  │  │                   Entity (Row)                 │  │    │
│  │  │  PartitionKey | RowKey | Properties...        │  │    │
│  │  └───────────────────────────────────────────────┘  │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Entity Properties

| Property | Description | Required |
|----------|-------------|----------|
| **PartitionKey** | Groups entities for scaling/querying | Yes |
| **RowKey** | Unique identifier within partition | Yes |
| **Timestamp** | Last modification time (auto-managed) | Yes (auto) |
| **Custom Properties** | Up to 252 custom properties | No |

### Data Types Supported

| Type | Description |
|------|-------------|
| String | Up to 64 KB |
| Binary | Up to 64 KB |
| Boolean | True/False |
| DateTime | UTC time since January 1, 1601 |
| Double | 64-bit floating point |
| GUID | 128-bit identifier |
| Int32 | 32-bit integer |
| Int64 | 64-bit integer |

### Working with Table Storage

#### Create Table

```powershell
# Get storage context
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123").Context

# Create table
New-AzStorageTable -Name "Customers" -Context $context
```

```bash
# Azure CLI
az storage table create --name "Customers" --account-name "mystorageaccount123"
```

#### Insert Entities

```powershell
# Using Azure.Data.Tables module
Install-Module -Name Az.Storage

$context = New-AzStorageContext -StorageAccountName "mystorageaccount123" -StorageAccountKey "<key>"
$table = Get-AzStorageTable -Name "Customers" -Context $context

# Create entity
$entity = @{
    PartitionKey = "USA"
    RowKey = "C001"
    Name = "John Doe"
    Email = "john@example.com"
    Age = 30
}

Add-AzTableRow -Table $table.CloudTable -PartitionKey "USA" -RowKey "C001" -property @{Name="John Doe"; Email="john@example.com"; Age=30}
```

#### Query Entities

```powershell
# Get all entities in a partition
$result = Get-AzTableRow -Table $table.CloudTable -PartitionKey "USA"

# Get specific entity
$entity = Get-AzTableRow -Table $table.CloudTable -PartitionKey "USA" -RowKey "C001"

# Query with filter
$result = Get-AzTableRow -Table $table.CloudTable -ColumnName "Age" -Value 30 -Operator Equal
```

### Partition Key Best Practices

| Strategy | Use Case | Example |
|----------|----------|---------|
| **By date** | Time-series data | "2024-01" |
| **By region** | Geographic distribution | "US-East" |
| **By user** | User-specific data | "user123" |
| **By category** | Product catalog | "Electronics" |

### Table Storage vs. Cosmos DB Table API

| Feature | Table Storage | Cosmos DB Table API |
|---------|---------------|---------------------|
| Global distribution | No | Yes |
| Latency SLA | No | <10ms |
| Throughput | Variable | Guaranteed |
| Indexing | PartitionKey + RowKey only | Automatic on all properties |
| Cost | Lower | Higher |

---

## 3.2 Azure Queue Storage

### What is Azure Queue Storage?
Azure Queue Storage provides cloud messaging between application components. It enables asynchronous message queueing for communication between components.

### Key Characteristics

| Feature | Value |
|---------|-------|
| Max message size | 64 KB |
| Max queue size | 500 TB |
| Max message TTL | 7 days (or unlimited) |
| Typical latency | Milliseconds |

### Queue Storage Architecture

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│   Producer  │────►│    Queue     │────►│  Consumer   │
│ Application │     │   Storage    │     │ Application │
└─────────────┘     └──────────────┘     └─────────────┘
                          │
                          ▼
                    ┌──────────┐
                    │ Messages │
                    │ [M1][M2] │
                    │ [M3][M4] │
                    └──────────┘
```

### Use Cases

1. **Decoupling applications**: Separate producer and consumer workloads
2. **Load leveling**: Handle burst traffic gracefully
3. **Background processing**: Offload time-consuming tasks
4. **Workflow orchestration**: Multi-step processing pipelines

### Working with Queue Storage

#### Create Queue

```powershell
# Get storage context
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123").Context

# Create queue
New-AzStorageQueue -Name "taskqueue" -Context $context
```

```bash
# Azure CLI
az storage queue create --name "taskqueue" --account-name "mystorageaccount123"
```

#### Send Messages

```powershell
# Get queue reference
$queue = Get-AzStorageQueue -Name "taskqueue" -Context $context

# Add message
$message = [Microsoft.Azure.Storage.Queue.CloudQueueMessage]::new("Process order #12345")
$queue.CloudQueue.AddMessageAsync($message)

# Add message with options
$queue.QueueClient.SendMessage(
    "Urgent: Process refund",
    [TimeSpan]::FromSeconds(0),      # Visibility delay
    [TimeSpan]::FromDays(7)          # Time to live
)
```

```bash
# Azure CLI
az storage message put --queue-name "taskqueue" --content "Process order #12345" --account-name "mystorageaccount123"
```

#### Receive Messages

```powershell
# Peek at message (doesn't remove it)
$peekedMessage = $queue.CloudQueue.PeekMessageAsync().Result

# Get message (makes it invisible)
$message = $queue.CloudQueue.GetMessageAsync().Result

# Process the message
Write-Host "Processing: $($message.AsString)"

# Delete after processing
$queue.CloudQueue.DeleteMessageAsync($message)
```

```bash
# Azure CLI - Get message
az storage message get --queue-name "taskqueue" --account-name "mystorageaccount123"

# Delete message
az storage message delete --queue-name "taskqueue" --id "<message-id>" --pop-receipt "<pop-receipt>" --account-name "mystorageaccount123"
```

### Message Visibility

When a message is retrieved, it becomes invisible for a specified duration:

```
Timeline:
[Message in queue] → [Message retrieved - starts invisibility timer]
                    → [30 seconds default visibility timeout]
                    → [If not deleted, message reappears in queue]
```

### Queue Storage Patterns

#### Competing Consumers Pattern

```
                        ┌──────────────┐
                   ┌───►│  Consumer 1  │
┌──────────┐       │    └──────────────┘
│  Queue   │───────┤    
│ [Tasks]  │       │    ┌──────────────┐
└──────────┘       ├───►│  Consumer 2  │
                   │    └──────────────┘
                   │    
                   │    ┌──────────────┐
                   └───►│  Consumer 3  │
                        └──────────────┘
```

#### Poison Message Handling

```powershell
# Track dequeue count to identify poison messages
$message = $queue.CloudQueue.GetMessageAsync().Result

if ($message.DequeueCount > 5) {
    # Move to dead-letter queue
    $deadLetterQueue.CloudQueue.AddMessageAsync($message)
    $queue.CloudQueue.DeleteMessageAsync($message)
}
```

---

## 3.3 Azure Storage Explorer

### What is Azure Storage Explorer?
Azure Storage Explorer is a standalone application for managing Azure Storage resources. It provides a graphical interface for working with blobs, files, queues, and tables.

### Key Features

| Feature | Description |
|---------|-------------|
| Cross-platform | Windows, macOS, Linux |
| Multiple accounts | Manage multiple storage accounts |
| Local development | Connect to Azure Storage Emulator |
| Upload/Download | Drag-and-drop support |
| Access management | SAS, keys, Azure AD |

### Installation

1. Download from: https://azure.microsoft.com/features/storage-explorer/
2. Install the application
3. Sign in to Azure or connect with connection string/SAS

### Connection Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| Azure Account | Sign in with Azure AD | Full management access |
| Connection String | Storage account connection string | Quick access |
| SAS URI | Shared Access Signature | Limited, time-bound access |
| Account Name + Key | Storage account key | Programmatic access |
| Local Emulator | Azurite or Storage Emulator | Development |

### Common Operations

#### Blob Operations
```
Storage Explorer
├── Blob Containers
│   ├── Create Container
│   ├── Upload Files/Folders
│   ├── Download Files
│   ├── Copy/Move Blobs
│   ├── Set Access Level
│   ├── Generate SAS
│   └── Manage Snapshots
```

#### File Share Operations
```
Storage Explorer
├── File Shares
│   ├── Create Share
│   ├── Upload Files/Folders
│   ├── Create Directories
│   ├── Download Files
│   ├── Set Quota
│   └── Create Snapshots
```

### Using Storage Explorer with Emulator

For local development, use Azurite (replacement for Azure Storage Emulator):

```bash
# Install Azurite
npm install -g azurite

# Run Azurite
azurite --silent --location c:\azurite --debug c:\azurite\debug.log
```

Connect in Storage Explorer:
1. Click "Connect" icon
2. Select "Attach to a local emulator"
3. Use default ports (10000, 10001, 10002)

---

## 3.4 Azure Shared Access Signature (SAS)

### What is SAS?
A Shared Access Signature (SAS) provides secure delegated access to resources in your storage account without exposing your account keys.

### Types of SAS

| Type | Description | Scope |
|------|-------------|-------|
| **User Delegation SAS** | Secured with Azure AD credentials | Blob storage only |
| **Service SAS** | Delegates access to specific service | Blob, Queue, Table, or File |
| **Account SAS** | Delegates access to multiple services | Account level |

### SAS Components

```
https://mystorageaccount.blob.core.windows.net/container/blob.txt
?sv=2020-08-04        # Signed version
&ss=b                 # Signed services (b=blob)
&srt=o                # Signed resource types (o=object)
&sp=r                 # Signed permissions (r=read)
&se=2024-01-01        # Signed expiry
&st=2023-01-01        # Signed start
&spr=https            # Signed protocol
&sig=<signature>      # Signature
```

### SAS Parameters

| Parameter | Description | Values |
|-----------|-------------|--------|
| sv | Service version | API version date |
| ss | Signed services | b(blob), f(file), q(queue), t(table) |
| srt | Signed resource types | s(service), c(container), o(object) |
| sp | Signed permissions | r(read), w(write), d(delete), l(list), a(add), c(create), u(update), p(process) |
| se | Signed expiry | ISO 8601 date |
| st | Signed start | ISO 8601 date |
| sip | Signed IP | IP or range |
| spr | Signed protocol | https, http |
| sig | Signature | HMAC-SHA256 |

### Creating SAS Tokens

#### Account SAS

```powershell
# Create Account SAS
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123").Context

$sasToken = New-AzStorageAccountSASToken `
  -Context $context `
  -Service Blob,File `
  -ResourceType Service,Container,Object `
  -Permission "rwdlacup" `
  -ExpiryTime (Get-Date).AddDays(7) `
  -Protocol HttpsOnly

Write-Host "SAS Token: $sasToken"
```

#### Service SAS (for Blob)

```powershell
# Create Blob SAS
$sasToken = New-AzStorageBlobSASToken `
  -Container "documents" `
  -Blob "report.pdf" `
  -Permission "r" `
  -ExpiryTime (Get-Date).AddHours(1) `
  -Context $context

# Full URL with SAS
$blobUrl = "https://mystorageaccount123.blob.core.windows.net/documents/report.pdf" + $sasToken
Write-Host "SAS URL: $blobUrl"
```

```bash
# Azure CLI
az storage blob generate-sas \
  --account-name "mystorageaccount123" \
  --container-name "documents" \
  --name "report.pdf" \
  --permissions r \
  --expiry "2024-12-31T23:59:59Z" \
  --output tsv
```

#### Container SAS

```powershell
$sasToken = New-AzStorageContainerSASToken `
  -Container "documents" `
  -Permission "rwdl" `
  -ExpiryTime (Get-Date).AddDays(30) `
  -Context $context
```

### Stored Access Policy

A stored access policy provides additional control over service-level SAS:

```powershell
# Create stored access policy
$policy = New-AzStorageContainerStoredAccessPolicy `
  -Container "documents" `
  -Policy "ReadPolicy" `
  -Permission "r" `
  -ExpiryTime (Get-Date).AddYears(1) `
  -Context $context

# Create SAS using policy
$sasToken = New-AzStorageContainerSASToken `
  -Container "documents" `
  -Policy "ReadPolicy" `
  -Context $context

# Revoke access by deleting policy
Remove-AzStorageContainerStoredAccessPolicy `
  -Container "documents" `
  -Policy "ReadPolicy" `
  -Context $context
```

### SAS Best Practices

| Practice | Description |
|----------|-------------|
| Use HTTPS only | Always require HTTPS protocol |
| Short expiry | Use shortest practical expiry time |
| Minimum permissions | Grant only required permissions |
| Use stored access policies | Allows revocation without regenerating keys |
| User delegation SAS | Preferred for blob storage (Azure AD secured) |

---

## 3.5 Azure Data Box

### What is Azure Data Box?
Azure Data Box is a family of products designed to transfer large amounts of data to Azure when network transfer is impractical.

### Data Box Products

| Product | Capacity | Use Case |
|---------|----------|----------|
| **Data Box Disk** | Up to 35 TB | Small transfers |
| **Data Box** | 100 TB | Medium transfers |
| **Data Box Heavy** | 1 PB | Large-scale transfers |

### Data Box Specifications

#### Data Box (Standard)

| Specification | Details |
|---------------|---------|
| Usable capacity | 80 TB |
| Weight | ~50 lbs (23 kg) |
| Dimensions | 309 x 430 x 502 mm |
| Interface | 1 Gbps RJ45, 10 Gbps SFP+ |
| Encryption | AES 256-bit |
| Transfer speed | Up to 80 TB in 1 business day |

### Data Box Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     Data Box Workflow                        │
│                                                              │
│  1. Order     2. Receive    3. Copy       4. Return         │
│  ┌─────┐     ┌───────┐     ┌──────┐      ┌────────┐        │
│  │Azure│────►│ Data  │────►│ Your │─────►│ Ship   │        │
│  │Portal│    │ Box   │     │ Data │      │ Back   │        │
│  └─────┘     └───────┘     └──────┘      └────────┘        │
│                                                │             │
│                                                ▼             │
│  5. Upload    ◄───────────────────────────────┘             │
│  ┌─────────┐                                                │
│  │ Azure   │                                                │
│  │ Storage │                                                │
│  └─────────┘                                                │
└─────────────────────────────────────────────────────────────┘
```

### Ordering Data Box

1. **Create order** in Azure Portal
2. **Receive device** via shipping
3. **Connect and unlock** using portal credentials
4. **Copy data** using SMB or NFS
5. **Prepare to ship** - device erases itself
6. **Ship back** to Microsoft datacenter
7. **Upload** - Data uploaded to your storage account

### Data Copy Commands

```powershell
# Connect to Data Box (appears as network share)
net use X: \\<DataBoxIP>\<ShareName>

# Copy using Robocopy
robocopy D:\SourceData X:\DestinationFolder /E /Z /MT:32

# Copy using AzCopy
azcopy copy "D:\SourceData" "\\<DataBoxIP>\<ShareName>" --recursive
```

### Data Box vs. Network Transfer

| Factor | Data Box | Network |
|--------|----------|---------|
| 1 TB transfer | Same day | Hours-Days |
| 10 TB transfer | Same day | Days-Weeks |
| 100 TB transfer | 1-2 days | Weeks-Months |
| 1 PB transfer | 1-2 weeks | Months-Years |

---

## 3.6 Azure Storage Replication

### What is Storage Replication?
Azure Storage always stores multiple copies of your data to protect against planned and unplanned events. Replication options provide different levels of durability and availability.

### Replication Options

| Option | Copies | Availability | Durability |
|--------|--------|--------------|------------|
| **LRS** | 3 copies in single datacenter | 99.999999999% (11 9s) | Single DC protected |
| **ZRS** | 3 copies across 3 zones | 99.9999999999% (12 9s) | Zone failure protected |
| **GRS** | 6 copies (3 primary + 3 secondary region) | 99.99999999999999% (16 9s) | Region failure protected |
| **GZRS** | 6 copies (3 zones + 3 secondary region) | 99.99999999999999% (16 9s) | Zone + region protected |

### Locally Redundant Storage (LRS)

```
┌──────────────────────────────────┐
│           Datacenter              │
│  ┌──────┐  ┌──────┐  ┌──────┐   │
│  │Copy 1│  │Copy 2│  │Copy 3│   │
│  └──────┘  └──────┘  └──────┘   │
└──────────────────────────────────┘

✓ Lowest cost
✓ Protects against drive failures
✗ Not protected against DC failure
```

### Zone-Redundant Storage (ZRS)

```
┌─────────────── Azure Region ───────────────┐
│                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐    │
│  │  Zone 1 │  │  Zone 2 │  │  Zone 3 │    │
│  │ ┌─────┐ │  │ ┌─────┐ │  │ ┌─────┐ │    │
│  │ │Copy1│ │  │ │Copy2│ │  │ │Copy3│ │    │
│  │ └─────┘ │  │ └─────┘ │  │ └─────┘ │    │
│  └─────────┘  └─────────┘  └─────────┘    │
│                                             │
└─────────────────────────────────────────────┘

✓ Protects against zone failures
✓ High availability within region
✗ Higher cost than LRS
✗ Not protected against regional failure
```

### Geo-Redundant Storage (GRS)

```
┌─────────── Primary Region ───────────┐    ┌─────────── Secondary Region ──────────┐
│           Datacenter                  │    │           Datacenter                   │
│  ┌──────┐  ┌──────┐  ┌──────┐       │    │  ┌──────┐  ┌──────┐  ┌──────┐        │
│  │Copy 1│  │Copy 2│  │Copy 3│       │════│  │Copy 4│  │Copy 5│  │Copy 6│        │
│  └──────┘  └──────┘  └──────┘       │    │  └──────┘  └──────┘  └──────┘        │
└───────────────────────────────────────┘    └────────────────────────────────────────┘
        Read/Write Access                           Failover Only (GRS)
                                                   OR Read Access (RA-GRS)

✓ Protects against regional disasters
✓ 16 9s durability
✗ Higher cost
✗ Secondary is async (potential data loss)
```

### Read-Access Options

| Option | Secondary Region Access |
|--------|------------------------|
| GRS | Failover only |
| RA-GRS | Always readable |
| GZRS | Failover only |
| RA-GZRS | Always readable |

### Changing Replication Type

```powershell
# Change replication
Set-AzStorageAccount `
  -ResourceGroupName "Storage-RG" `
  -Name "mystorageaccount123" `
  -SkuName "Standard_GRS"
```

```bash
# Azure CLI
az storage account update \
  --name "mystorageaccount123" \
  --resource-group "Storage-RG" \
  --sku "Standard_GRS"
```

### Replication Selection Guide

| Scenario | Recommended |
|----------|-------------|
| Cost-sensitive, non-critical data | LRS |
| High availability within region | ZRS |
| Disaster recovery required | GRS or GZRS |
| Read access to secondary needed | RA-GRS or RA-GZRS |
| Mission-critical workloads | RA-GZRS |

---

## 3.7 Data Replication Options

### Understanding RPO and RTO

| Term | Definition | Storage Impact |
|------|------------|----------------|
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss | GRS: ~15 minutes |
| **RTO** (Recovery Time Objective) | Maximum acceptable downtime | Failover time |

### Synchronous vs Asynchronous Replication

#### Synchronous (LRS, ZRS)
```
Write Request → Primary → Copy 1, Copy 2, Copy 3 → All Acknowledged → Success
                         (simultaneous writes)
```

#### Asynchronous (GRS, GZRS)
```
Write Request → Primary Region → Success
                    │
                    └──► (background replication) → Secondary Region
```

### Failover Process

```powershell
# Initiate storage account failover (GRS/GZRS only)
Invoke-AzStorageAccountFailover `
  -ResourceGroupName "Storage-RG" `
  -Name "mystorageaccount123"
```

**Important**: After failover:
- Secondary becomes primary
- Replication type changes to LRS
- Must reconfigure replication after failover

### Object Replication (Blob Storage)

Asynchronously copy blobs between storage accounts:

```powershell
# Create replication policy
$rule = New-AzStorageObjectReplicationPolicyRule `
  -SourceContainer "source-container" `
  -DestinationContainer "dest-container"

$policy = Set-AzStorageObjectReplicationPolicy `
  -ResourceGroupName "Storage-RG" `
  -AccountName "sourceaccount" `
  -DestinationAccountName "destaccount" `
  -Rule $rule
```

---

## 3.8 Azure Import/Export Service

### What is Azure Import/Export?
Azure Import/Export service enables transferring large amounts of data to Azure Blob storage and Azure Files by shipping disk drives to an Azure datacenter.

### Import vs Export

| Operation | Direction | Supported Services |
|-----------|-----------|-------------------|
| **Import** | On-premises → Azure | Blob storage, Azure Files |
| **Export** | Azure → On-premises | Blob storage only |

### Import Job Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     Import Job Workflow                      │
│                                                              │
│  1. Prepare           2. Create Job       3. Ship           │
│  ┌──────────────┐    ┌───────────────┐   ┌─────────────┐   │
│  │ Copy data to │───►│ Create import │──►│ Ship drives │   │
│  │ disk drives  │    │ job in portal │   │ to Azure DC │   │
│  └──────────────┘    └───────────────┘   └─────────────┘   │
│                                                 │            │
│  5. Complete         4. Process              ◄──┘            │
│  ┌──────────────┐    ┌───────────────┐                      │
│  │ Data in      │◄───│ Azure copies  │                      │
│  │ storage acct │    │ data to blob  │                      │
│  └──────────────┘    └───────────────┘                      │
└─────────────────────────────────────────────────────────────┘
```

### Supported Drives

| Type | Supported |
|------|-----------|
| 2.5-inch SSD | Yes |
| 2.5-inch HDD | Yes |
| 3.5-inch HDD | Yes (SATA II or SATA III) |

### Preparing Drives with WAImportExport Tool

```powershell
# Download WAImportExport tool from Microsoft

# Prepare drive for import
WAImportExport.exe PrepImport `
  /j:MyJob.jrn `
  /id:session1 `
  /sk:<storageaccountkey> `
  /t:F `
  /bk:<BitLockerKey> `
  /srcdir:C:\DataToUpload `
  /dstdir:importcontainer/
```

### Creating Import Job

```powershell
# Create import job
$importJob = New-AzImportExport `
  -ResourceGroupName "Import-RG" `
  -Name "MyImportJob" `
  -Location "East US" `
  -StorageAccountId "/subscriptions/.../storageAccounts/mystorageaccount123" `
  -JobType "Import" `
  -ReturnAddressRecipientName "John Doe" `
  -ReturnAddressStreetAddress1 "123 Main St" `
  -ReturnAddressCity "Seattle" `
  -ReturnAddressStateOrProvince "WA" `
  -ReturnAddressPostalCode "98101" `
  -ReturnAddressCountryOrRegion "USA" `
  -ReturnAddressPhone "555-1234" `
  -DiagnosticsPath "importcontainer" `
  -DriveList @(@{DriveId="9WM1234"; BitLockerKey="<key>"; ManifestFile="DriveManifest.xml"})
```

### Export Job

```powershell
# Create export job
$exportJob = New-AzImportExport `
  -ResourceGroupName "Export-RG" `
  -Name "MyExportJob" `
  -Location "East US" `
  -StorageAccountId "/subscriptions/.../storageAccounts/mystorageaccount123" `
  -JobType "Export" `
  -BlobListBlobPath "exportcontainer/bloblist.txt"
```

### Import/Export vs Data Box

| Factor | Import/Export | Data Box |
|--------|---------------|----------|
| You provide | Your own drives | Microsoft provides device |
| Capacity | Multiple drives (each up to 10 TB) | Up to 1 PB |
| Complexity | Higher (prepare, encrypt) | Lower (plug and copy) |
| Cost | Pay per drive | Pay per device rental |

---

## Hands-on Exercises

### Exercise 1: Attach & Detach External Storage Account

```powershell
# Using Azure Storage Explorer:
# 1. Open Storage Explorer
# 2. Click "Connect to Azure Storage"
# 3. Select "Use a storage account name and key"

# Get connection string
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount123"
$key = (Get-AzStorageAccountKey -ResourceGroupName "Storage-RG" -Name "mystorageaccount123")[0].Value

# Connection string format:
# DefaultEndpointsProtocol=https;AccountName=mystorageaccount123;AccountKey=<key>;EndpointSuffix=core.windows.net

# To detach: Right-click storage account in Storage Explorer > Detach
```

### Exercise 2: Storage Explorer - Blob, File, Queues, Tables

```powershell
# Blob operations in Storage Explorer
# 1. Navigate to Blob Containers
# 2. Create new container "exercise-container"
# 3. Upload files via drag-and-drop
# 4. Set access level (Private/Blob/Container)

# File operations
# 1. Navigate to File Shares
# 2. Create "exercise-fileshare"
# 3. Create directories and upload files

# Queue operations
# 1. Navigate to Queues
# 2. Create "exercise-queue"
# 3. Add messages

# Table operations
# 1. Navigate to Tables
# 2. Create "exercise-table"
# 3. Add entities
```

### Exercise 3: Backup - Archive

```powershell
# Set blob access tier to Archive
Set-AzStorageBlobTier `
  -Container "backup" `
  -Blob "old-backup.zip" `
  -Tier Archive `
  -Context $context

# List archived blobs
Get-AzStorageBlob -Container "backup" -Context $context | Where-Object {$_.AccessTier -eq "Archive"}

# Rehydrate archived blob to Hot (takes hours)
Set-AzStorageBlobTier `
  -Container "backup" `
  -Blob "old-backup.zip" `
  -Tier Hot `
  -Context $context
```

### Exercise 4: Backup - Snapshots

```powershell
# Create blob snapshot
$blob = Get-AzStorageBlob -Container "documents" -Blob "important.docx" -Context $context
$snapshot = $blob.ICloudBlob.CreateSnapshot()

Write-Host "Snapshot time: $($snapshot.SnapshotTime)"

# List snapshots
Get-AzStorageBlob -Container "documents" -Blob "important.docx" -Context $context -IncludeSnapshot

# Restore from snapshot (copy snapshot to new blob)
Start-AzStorageBlobCopy `
  -SrcBlob "important.docx" `
  -SrcContainer "documents" `
  -SrcContext $context `
  -DestBlob "important-restored.docx" `
  -DestContainer "documents" `
  -DestContext $context `
  -SrcSnapshot $snapshot.SnapshotTime
```

### Exercise 5: Backup - AzCopy

```powershell
# Download AzCopy from https://aka.ms/downloadazcopy

# Login to AzCopy
azcopy login

# Copy blob to local
azcopy copy "https://mystorageaccount123.blob.core.windows.net/documents/report.pdf" "C:\Backups\"

# Copy entire container
azcopy copy "https://mystorageaccount123.blob.core.windows.net/documents/*" "C:\Backups\documents\" --recursive

# Sync local folder to blob (only copies changes)
azcopy sync "C:\Data" "https://mystorageaccount123.blob.core.windows.net/backup" --recursive

# Copy between storage accounts
azcopy copy "https://source.blob.core.windows.net/container/*" "https://dest.blob.core.windows.net/container/" --recursive
```

### Exercise 6: Azure Shared Access Signature (SAS)

```powershell
# Create various SAS tokens

# 1. Container SAS with read-only access
$containerSAS = New-AzStorageContainerSASToken `
  -Container "public" `
  -Permission "r" `
  -ExpiryTime (Get-Date).AddDays(7) `
  -Context $context

Write-Host "Container SAS: $containerSAS"

# 2. Blob SAS with write access
$blobSAS = New-AzStorageBlobSASToken `
  -Container "uploads" `
  -Blob "data.csv" `
  -Permission "rw" `
  -ExpiryTime (Get-Date).AddHours(2) `
  -Context $context

# 3. Account SAS with multiple permissions
$accountSAS = New-AzStorageAccountSASToken `
  -Context $context `
  -Service Blob,File `
  -ResourceType Container,Object `
  -Permission "rwdlac" `
  -ExpiryTime (Get-Date).AddMonths(1) `
  -Protocol HttpsOnly

# 4. Create stored access policy for revocation capability
New-AzStorageContainerStoredAccessPolicy `
  -Container "documents" `
  -Policy "DownloadPolicy" `
  -Permission "r" `
  -ExpiryTime (Get-Date).AddYears(1) `
  -Context $context

$policySAS = New-AzStorageContainerSASToken `
  -Container "documents" `
  -Policy "DownloadPolicy" `
  -Context $context
```

### Exercise 7: Use Azure Data Factory Copy Data Tool

```powershell
# Using Azure Portal:
# 1. Create Azure Data Factory
# 2. Open Data Factory Studio
# 3. Click "Ingest" (Copy Data tool)
# 4. Choose task type: "Built-in copy task"
# 5. Configure source (e.g., File System)
# 6. Configure destination (Azure Blob Storage)
# 7. Set copy settings and schedule
# 8. Review and complete

# Using PowerShell to create Data Factory
New-AzDataFactoryV2 `
  -ResourceGroupName "DataFactory-RG" `
  -Name "MyDataFactory$(Get-Random -Maximum 9999)" `
  -Location "East US"

# Create linked service for storage
$storageLinkedService = @{
    name = "AzureStorageLinkedService"
    properties = @{
        type = "AzureBlobStorage"
        typeProperties = @{
            connectionString = "your-connection-string"
        }
    }
}
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Table Storage | NoSQL key-value store, PartitionKey + RowKey, 252 properties |
| Queue Storage | Asynchronous messaging, 64 KB max message, 7 days TTL |
| Storage Explorer | Cross-platform GUI, multiple connection methods |
| SAS | Delegated access, time-limited, stored access policies |
| Data Box | Large data transfers, 80 TB - 1 PB capacity |
| Replication | LRS, ZRS, GRS, GZRS options for durability |
| Import/Export | Ship drives to Azure, WAImportExport tool |

---

## Review Questions

1. What is the difference between PartitionKey and RowKey in Table Storage?
2. Explain the visibility timeout concept in Queue Storage.
3. What are the three types of Shared Access Signatures?
4. When would you use Data Box vs. network transfer?
5. Compare LRS, ZRS, and GRS replication options.
6. What tool is used to prepare drives for the Import/Export service?

---

## Additional Resources

- [Azure Table Storage Documentation](https://docs.microsoft.com/azure/storage/tables/)
- [Azure Queue Storage Documentation](https://docs.microsoft.com/azure/storage/queues/)
- [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/)
- [SAS Overview](https://docs.microsoft.com/azure/storage/common/storage-sas-overview)
- [Azure Data Box Documentation](https://docs.microsoft.com/azure/databox/)
- [Storage Redundancy](https://docs.microsoft.com/azure/storage/common/storage-redundancy)

---

## 3.6 Advanced Storage Patterns and Optimization (Expert Level)

### Blob Storage Lifecycle Management

```json
{
  "rules": [
    {
      "name": "MoveToArchive",
      "enabled": true,
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
          },
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/", "backups/"]
        }
      }
    }
  ]
}
```

```powershell
# Apply lifecycle policy
$rule = @{
    Name = "ArchiveOldData"
    Enabled = $true
    Definition = @{
        Actions = @{
            BaseBlob = @{
                TierToCool = @{ DaysAfterModificationGreaterThan = 30 }
                TierToArchive = @{ DaysAfterModificationGreaterThan = 90 }
            }
        }
        Filters = @{
            BlobTypes = @("blockBlob")
            PrefixMatch = @("data/archive/")
        }
    }
}

Set-AzStorageAccountManagementPolicy `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "mystorageaccount" `
    -Rule @($rule)
```

### Object Replication for Disaster Recovery

```powershell
# Enable object replication between storage accounts
$destContext = (Get-AzStorageAccount -ResourceGroupName "DR-RG" -Name "drstorage").Context
$srcContext = (Get-AzStorageAccount -ResourceGroupName "Prod-RG" -Name "prodstorage").Context

# Create replication policy
$ruleParams = @{
    SourceContainer = "production-data"
    DestinationContainer = "replicated-data"
    SourceStorageAccount = "prodstorage"
    DestinationStorageAccount = "drstorage"
}

# Note: Requires enabling versioning and change feed on both accounts
Enable-AzStorageBlobChangeFeed -ResourceGroupName "Prod-RG" -StorageAccountName "prodstorage"
Update-AzStorageBlobServiceProperty -ResourceGroupName "Prod-RG" -StorageAccountName "prodstorage" -IsVersioningEnabled $true
```

### High-Performance Blob Operations

```powershell
# Parallel upload with AzCopy
$env:AZCOPY_CONCURRENCY_VALUE = "32"
azcopy copy ".\localdata\*" "https://mystorageaccount.blob.core.windows.net/data" `
    --recursive `
    --put-md5 `
    --check-length `
    --block-size-mb 100

# Batch operations using Azure CLI
az storage blob upload-batch `
    --account-name "mystorageaccount" `
    --destination "container-name" `
    --source ".\localfolder" `
    --pattern "*.csv" `
    --max-connections 32

# Download with multi-threading
azcopy copy "https://mystorageaccount.blob.core.windows.net/data/*" `
    ".\localdata" `
    --recursive `
    --check-md5 FailIfDifferent
```

### Storage Account Failover

```powershell
# Check last sync time
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Prod-RG" -Name "mystorageaccount"
$storageAccount.FailoverInProgress
$storageAccount.LastGeoFailoverTime

# Initiate account failover (for RA-GRS/RA-GZRS accounts)
Invoke-AzStorageAccountFailover `
    -ResourceGroupName "Prod-RG" `
    -Name "mystorageaccount" `
    -Force

# Monitor failover progress
Get-AzStorageAccount -ResourceGroupName "Prod-RG" -Name "mystorageaccount" |
    Select-Object StorageAccountName, PrimaryLocation, StatusOfPrimary, FailoverInProgress
```

### Immutable Storage for Compliance

```powershell
# Enable immutability policy (WORM - Write Once Read Many)
$container = Get-AzStorageContainer -Context $context -Name "legal-hold"

# Set time-based retention
Set-AzRmStorageContainerImmutabilityPolicy `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "compliancestorage" `
    -ContainerName "legal-hold" `
    -ImmutabilityPeriod 365 `
    -AllowProtectedAppendWrites $true

# Lock policy (irreversible!)
Lock-AzRmStorageContainerImmutabilityPolicy `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "compliancestorage" `
    -ContainerName "legal-hold" `
    -Etag $policy.Etag

# Add legal hold tags
Add-AzRmStorageContainerLegalHold `
    -ResourceGroupName "Storage-RG" `
    -StorageAccountName "compliancestorage" `
    -ContainerName "legal-hold" `
    -Tag "LegalCase-2024-001", "Audit-Q4"
```

### Storage Analytics and Monitoring

```powershell
# Enable storage analytics
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount").Context

Set-AzStorageServiceLoggingProperty `
    -Context $context `
    -ServiceType Blob `
    -LoggingOperations All `
    -RetentionDays 30 `
    -Version 2.0

Set-AzStorageServiceMetricsProperty `
    -Context $context `
    -ServiceType Blob `
    -MetricsType Hour `
    -MetricsLevel ServiceAndApi `
    -RetentionDays 30

# Query storage analytics logs
$logs = Get-AzStorageBlob -Context $context -Container "`$logs" | 
    Where-Object { $_.LastModified -gt (Get-Date).AddDays(-1) }

foreach ($log in $logs) {
    $content = $log | Get-AzStorageBlobContent -Destination ".\logs\" -Force
}
```

### Cost Optimization Strategies

| Strategy | Savings | Implementation |
|----------|---------|----------------|
| **Access Tier Optimization** | 50-80% | Lifecycle policies for Cool/Archive |
| **Reserved Capacity** | Up to 38% | 1-3 year capacity reservations |
| **Delete Unused Blobs** | Variable | Scheduled cleanup scripts |
| **Compression** | 50-90% | Client-side compression before upload |
| **Deduplication** | 30-60% | Azure Backup or client-side dedup |

```powershell
# Analyze blob inventory for cost optimization
$blobs = Get-AzStorageBlob -Context $context -Container "data"
$analysis = $blobs | Group-Object -Property AccessTier | 
    Select-Object Name, Count, @{N='TotalGB';E={($_.Group | Measure-Object Length -Sum).Sum / 1GB}}

# Find blobs that haven't been accessed (candidates for cool/archive)
$unusedBlobs = $blobs | Where-Object { 
    $_.LastAccessTime -lt (Get-Date).AddDays(-30) -and 
    $_.AccessTier -eq "Hot" 
}
```
