# Azure Hands-On Labs Quick Start Guide
## Practical Exercises for AZ-104 Certification

---

# Introduction

This guide provides step-by-step hands-on labs to build practical Azure skills. Complete these labs in order to gain experience with core Azure services.

**Prerequisites:**
- Azure free account (https://azure.microsoft.com/free)
- Web browser (Chrome, Edge, Firefox)
- Basic understanding of networking concepts

**Cost Warning:** Most labs use free-tier resources, but always delete resources when finished to avoid charges.

---

# LAB 1: Azure Portal and Resource Groups

## Objective
Learn to navigate Azure Portal and create resource groups.

## Time
15 minutes

## Steps

### Step 1: Log into Azure Portal
1. Go to https://portal.azure.com
2. Sign in with your Microsoft account
3. You'll see the Azure Portal dashboard

### Step 2: Explore the Portal
```
Key Areas to Know:
├── Search Bar (top center) - Search for any service or resource
├── Home (left menu) - Dashboard and quick access
├── All Services (left menu) - Full list of Azure services
├── Resource Groups (left menu) - Your containers
├── Cost Management (left menu) - Track spending
└── Help + Support (top right ?) - Get help
```

### Step 3: Create Your First Resource Group
1. Click **"Resource groups"** in the left menu
2. Click **"+ Create"**
3. Configure:
   - Subscription: Your subscription
   - Resource group name: `Lab-RG-01`
   - Region: `East US` (or closest to you)
4. Click **"Review + Create"**
5. Click **"Create"**

### Step 4: Add Tags to Resource Group
1. Click on `Lab-RG-01`
2. Click **"Tags"** in the left menu
3. Add a tag:
   - Name: `Environment`
   - Value: `Learning`
4. Click **"Save"**

## Verification
✅ Resource group `Lab-RG-01` exists
✅ Tag `Environment=Learning` is applied

## Clean Up
Keep this resource group - we'll use it in future labs.

---

# LAB 2: Azure Cloud Shell and CLI Basics

## Objective
Learn to use Azure CLI through Cloud Shell.

## Time
20 minutes

## Steps

### Step 1: Open Cloud Shell
1. Click the **Cloud Shell icon** (terminal icon) in the top bar
2. Select **"Bash"** when prompted
3. If first time, click **"Create storage"** (creates storage account for shell)

### Step 2: Basic Azure CLI Commands
```bash
# Check you're logged in
az account show --query name -o tsv

# List all subscriptions
az account list --output table

# List all resource groups
az group list --output table

# Get help for any command
az group create --help
```

### Step 3: Create a Resource Group with CLI
```bash
# Create a resource group
az group create \
    --name "CLI-Lab-RG" \
    --location "eastus" \
    --tags Environment=Learning Purpose=CLI-Practice

# Verify it was created
az group list --output table
```

### Step 4: View Resource Group Details
```bash
# Show resource group details
az group show --name "CLI-Lab-RG"

# List all resources in the group (should be empty)
az resource list --resource-group "CLI-Lab-RG" --output table
```

### Step 5: Practice Output Formats
```bash
# JSON format (default)
az group show --name "CLI-Lab-RG"

# Table format (human-readable)
az group show --name "CLI-Lab-RG" --output table

# TSV format (scriptable)
az group show --name "CLI-Lab-RG" --query name --output tsv
```

## Verification
✅ Can execute Azure CLI commands
✅ Resource group `CLI-Lab-RG` created via CLI

## Clean Up
```bash
# Delete the resource group
az group delete --name "CLI-Lab-RG" --yes --no-wait
```

---

# LAB 3: Create a Virtual Machine

## Objective
Create and connect to an Azure Virtual Machine.

## Time
30 minutes

## Steps

### Step 1: Create a Linux VM using Portal
1. Search for **"Virtual machines"** in the search bar
2. Click **"+ Create" → "Azure virtual machine"**

3. **Basics tab:**
   - Subscription: Your subscription
   - Resource group: `Lab-RG-01`
   - Virtual machine name: `myFirstVM`
   - Region: `East US`
   - Availability options: No infrastructure redundancy required
   - Security type: Standard
   - Image: `Ubuntu Server 22.04 LTS`
   - Size: Click "See all sizes" → Select `Standard_B1s` (cheapest)
   - Authentication: SSH public key
   - Username: `azureuser`
   - SSH public key source: Generate new key pair
   - Key pair name: `myFirstVM_key`

4. **Disks tab:**
   - OS disk type: `Standard SSD`

5. **Networking tab:**
   - Virtual network: Create new → `myFirstVM-vnet`
   - Subnet: default (10.0.0.0/24)
   - Public IP: Create new
   - NIC NSG: Basic
   - Public inbound ports: Allow selected ports
   - Select inbound ports: SSH (22)

6. **Management tab:**
   - Auto-shutdown: Enable, set to 7:00 PM your timezone (saves money!)

7. Click **"Review + Create"** → **"Create"**

8. **Download the private key** when prompted (save it securely!)

### Step 2: Connect to VM via SSH
```bash
# Using Cloud Shell (upload your key first)
# Or from local terminal:

# Change permissions on key file
chmod 400 ~/Downloads/myFirstVM_key.pem

# Get the public IP (from Azure Portal or)
az vm list-ip-addresses --name myFirstVM --resource-group Lab-RG-01 --output table

# Connect via SSH
ssh -i ~/Downloads/myFirstVM_key.pem azureuser@<YOUR-PUBLIC-IP>
```

### Step 3: Explore the VM
```bash
# Once connected to the VM:

# Check system info
uname -a

# Check memory
free -h

# Check disk space
df -h

# Check network
ip addr

# Exit the VM
exit
```

### Step 4: Manage VM from Portal
1. Go to your VM in Azure Portal
2. Try these operations:
   - **Stop** the VM (saves money)
   - **Start** the VM
   - View **Metrics** (CPU, Network, Disk)
   - Check **Boot diagnostics** (screenshot)

### Step 5: Create VM using CLI
```bash
# Create another VM using CLI
az vm create \
    --resource-group "Lab-RG-01" \
    --name "mySecondVM" \
    --image "Ubuntu2204" \
    --admin-username "azureuser" \
    --generate-ssh-keys \
    --size "Standard_B1s" \
    --public-ip-sku Basic

# Get the public IP
az vm list-ip-addresses --name mySecondVM --resource-group Lab-RG-01 --output table
```

## Verification
✅ VM `myFirstVM` created and running
✅ Successfully connected via SSH
✅ VM `mySecondVM` created via CLI

## Clean Up
```bash
# Stop VMs to save money (don't delete yet - we'll use in next labs)
az vm stop --name myFirstVM --resource-group Lab-RG-01
az vm stop --name mySecondVM --resource-group Lab-RG-01

# Deallocate to stop billing
az vm deallocate --name myFirstVM --resource-group Lab-RG-01
az vm deallocate --name mySecondVM --resource-group Lab-RG-01
```

---

# LAB 4: Storage Account and Blob Storage

## Objective
Create a storage account and work with Blob storage.

## Time
25 minutes

## Steps

### Step 1: Create Storage Account
1. Search for **"Storage accounts"** in Portal
2. Click **"+ Create"**

3. **Basics tab:**
   - Subscription: Your subscription
   - Resource group: `Lab-RG-01`
   - Storage account name: `mystoragelab123` (must be globally unique, use random numbers)
   - Region: `East US`
   - Performance: Standard
   - Redundancy: LRS (cheapest)

4. **Advanced tab:**
   - Require secure transfer: Yes (default)
   - Enable blob public access: Disabled

5. Click **"Review"** → **"Create"**

### Step 2: Create a Blob Container
1. Go to your storage account
2. Click **"Containers"** under Data storage
3. Click **"+ Container"**
   - Name: `images`
   - Public access level: Private
4. Click **"Create"**

### Step 3: Upload a Blob
1. Click on the `images` container
2. Click **"Upload"**
3. Select any file from your computer (image, text file, etc.)
4. Click **"Upload"**

### Step 4: Access Blob Properties
1. Click on the uploaded file
2. View the URL (can't access without authentication)
3. Check properties: Size, Created, Modified

### Step 5: Generate SAS Token
1. Click on the uploaded file
2. Click **"Generate SAS"**
3. Configure:
   - Permissions: Read
   - Start: Now
   - Expiry: 1 hour from now
4. Click **"Generate SAS token and URL"**
5. Copy the **Blob SAS URL**
6. Open in new browser tab - file should download!

### Step 6: Use Azure CLI for Storage
```bash
# Get storage account key
STORAGE_KEY=$(az storage account keys list \
    --account-name mystoragelab123 \
    --resource-group Lab-RG-01 \
    --query '[0].value' \
    --output tsv)

# List containers
az storage container list \
    --account-name mystoragelab123 \
    --account-key $STORAGE_KEY \
    --output table

# List blobs in container
az storage blob list \
    --container-name images \
    --account-name mystoragelab123 \
    --account-key $STORAGE_KEY \
    --output table

# Upload a blob
echo "Hello from Azure CLI!" > hello.txt
az storage blob upload \
    --container-name images \
    --name hello.txt \
    --file hello.txt \
    --account-name mystoragelab123 \
    --account-key $STORAGE_KEY

# Download a blob
az storage blob download \
    --container-name images \
    --name hello.txt \
    --file downloaded-hello.txt \
    --account-name mystoragelab123 \
    --account-key $STORAGE_KEY

cat downloaded-hello.txt
```

### Step 7: Create a File Share
1. In storage account, click **"File shares"**
2. Click **"+ File share"**
   - Name: `myfileshare`
   - Tier: Transaction optimized
3. Click **"Create"**
4. Upload a file to the share
5. Click **"Connect"** to see mount instructions

## Verification
✅ Storage account created
✅ Blob container with uploaded file
✅ SAS URL works
✅ File share created

## Clean Up
Keep resources for now - we'll delete at end.

---

# LAB 5: Virtual Network and NSG

## Objective
Create a VNet, subnets, and configure NSG rules.

## Time
30 minutes

## Steps

### Step 1: Create a Virtual Network
1. Search for **"Virtual networks"**
2. Click **"+ Create"**

3. **Basics tab:**
   - Resource group: `Lab-RG-01`
   - Name: `Lab-VNet`
   - Region: `East US`

4. **IP Addresses tab:**
   - IPv4 address space: `10.0.0.0/16`
   - Click **"+ Add subnet"**:
     - Subnet name: `WebSubnet`
     - Starting address: `10.0.1.0`
     - Subnet size: `/24` (256 addresses)
   - Click **"Add"**
   - Click **"+ Add subnet"** again:
     - Subnet name: `DatabaseSubnet`
     - Starting address: `10.0.2.0`
     - Subnet size: `/24`
   - Click **"Add"**

5. Click **"Review + Create"** → **"Create"**

### Step 2: Create a Network Security Group
1. Search for **"Network security groups"**
2. Click **"+ Create"**
   - Resource group: `Lab-RG-01`
   - Name: `WebSubnet-NSG`
   - Region: `East US`
3. Click **"Review + Create"** → **"Create"**

### Step 3: Add NSG Rules
1. Go to `WebSubnet-NSG`
2. Click **"Inbound security rules"**
3. Click **"+ Add"**:
   - Source: Any
   - Source port ranges: *
   - Destination: Any
   - Service: HTTP
   - Action: Allow
   - Priority: 100
   - Name: `Allow-HTTP`
4. Click **"Add"**

5. Add another rule for HTTPS:
   - Service: HTTPS
   - Priority: 110
   - Name: `Allow-HTTPS`

### Step 4: Associate NSG to Subnet
1. In `WebSubnet-NSG`, click **"Subnets"**
2. Click **"+ Associate"**
3. Select:
   - Virtual network: `Lab-VNet`
   - Subnet: `WebSubnet`
4. Click **"OK"**

### Step 5: Verify NSG Rules
```bash
# List NSG rules
az network nsg rule list \
    --nsg-name WebSubnet-NSG \
    --resource-group Lab-RG-01 \
    --output table
```

### Step 6: Create using CLI
```bash
# Create a Database NSG
az network nsg create \
    --name DatabaseSubnet-NSG \
    --resource-group Lab-RG-01 \
    --location eastus

# Add rule to allow SQL only from WebSubnet
az network nsg rule create \
    --nsg-name DatabaseSubnet-NSG \
    --resource-group Lab-RG-01 \
    --name Allow-SQL-From-Web \
    --priority 100 \
    --source-address-prefixes 10.0.1.0/24 \
    --destination-port-ranges 1433 \
    --access Allow \
    --protocol Tcp \
    --direction Inbound

# Associate to DatabaseSubnet
az network vnet subnet update \
    --vnet-name Lab-VNet \
    --name DatabaseSubnet \
    --resource-group Lab-RG-01 \
    --network-security-group DatabaseSubnet-NSG
```

## Verification
✅ VNet `Lab-VNet` with two subnets created
✅ NSG `WebSubnet-NSG` with HTTP/HTTPS rules
✅ NSG associated with WebSubnet
✅ DatabaseSubnet-NSG with SQL rule

---

# LAB 6: Azure AD Users and RBAC

## Objective
Create Azure AD users and assign RBAC roles.

## Time
25 minutes

## Steps

### Step 1: Create an Azure AD User
1. Search for **"Microsoft Entra ID"** (formerly Azure AD)
2. Click **"Users"** → **"+ New user"** → **"Create new user"**
3. Configure:
   - User principal name: `labuser1@yourdomain.onmicrosoft.com`
   - Display name: `Lab User 1`
   - Auto-generate password: Yes
   - Copy the password shown!
4. Click **"Create"**

### Step 2: Create Another User
1. Create another user:
   - User principal name: `labuser2@yourdomain.onmicrosoft.com`
   - Display name: `Lab User 2`

### Step 3: Create a Security Group
1. In Microsoft Entra ID, click **"Groups"**
2. Click **"+ New group"**
   - Group type: Security
   - Group name: `Lab-VM-Admins`
   - Membership type: Assigned
3. Click **"Members"** → Add `Lab User 1`
4. Click **"Create"**

### Step 4: Assign RBAC Role to User
1. Go to your resource group `Lab-RG-01`
2. Click **"Access control (IAM)"**
3. Click **"+ Add"** → **"Add role assignment"**
4. **Role tab:** Select `Reader`
5. **Members tab:** 
   - Select: User, group, or service principal
   - Click **"+ Select members"**
   - Search and select `Lab User 2`
6. Click **"Review + assign"**

### Step 5: Assign RBAC Role to Group
1. Still in `Lab-RG-01` → Access control (IAM)
2. Click **"+ Add"** → **"Add role assignment"**
3. **Role tab:** Select `Virtual Machine Contributor`
4. **Members tab:** Select group `Lab-VM-Admins`
5. Click **"Review + assign"**

### Step 6: Verify Assignments
```bash
# List role assignments for the resource group
az role assignment list \
    --resource-group Lab-RG-01 \
    --output table

# List role assignments for a specific user
az role assignment list \
    --assignee "labuser2@yourdomain.onmicrosoft.com" \
    --output table
```

### Step 7: Test Access (Optional)
1. Open a private/incognito browser window
2. Go to portal.azure.com
3. Sign in as `labuser2@yourdomain.onmicrosoft.com`
4. Navigate to `Lab-RG-01`
5. Verify: Can view resources, but cannot create/modify

## Verification
✅ Two users created in Azure AD
✅ Security group created with member
✅ Reader role assigned to labuser2
✅ VM Contributor role assigned to group

---

# LAB 7: Azure Monitor and Alerts

## Objective
Configure Azure Monitor metrics, logs, and alerts.

## Time
25 minutes

## Steps

### Step 1: View VM Metrics
1. Go to your VM `myFirstVM`
2. Click **"Metrics"** in the left menu
3. Select metric:
   - Metric Namespace: Virtual Machine Host
   - Metric: Percentage CPU
   - Aggregation: Average
4. Click **"Add metric"** again:
   - Metric: Available Memory Bytes
5. Set time range to "Last hour"
6. Click **"Pin to dashboard"** to save this chart

### Step 2: Enable Diagnostic Settings
1. In VM, click **"Diagnostic settings"** under Monitoring
2. Click **"+ Add diagnostic setting"**
   - Name: `vm-diagnostics`
   - Logs: Select relevant categories
   - Destination: Send to Log Analytics workspace
     - Create new workspace:
       - Name: `Lab-LogAnalytics`
       - Region: East US
3. Click **"Save"**

### Step 3: Create an Alert Rule
1. In VM, click **"Alerts"**
2. Click **"+ Create"** → **"Alert rule"**

3. **Condition:**
   - Signal name: `Percentage CPU`
   - Threshold: Static
   - Operator: Greater than
   - Threshold value: 80
   - Aggregation granularity: 5 minutes

4. **Actions:**
   - Click **"+ Create action group"**
     - Action group name: `Lab-Alert-Group`
     - Short name: `LabAlert`
     - Notification type: Email/SMS/Push/Voice
     - Email: Your email address
   - Click **"Review + create"**

5. **Details:**
   - Alert rule name: `High-CPU-Alert`
   - Severity: 2 - Warning

6. Click **"Review + create"** → **"Create"**

### Step 4: Query Logs with KQL
1. Go to **"Monitor"** in Azure Portal
2. Click **"Logs"**
3. Select your Log Analytics workspace
4. Run some queries:

```kusto
// List all tables
search *
| distinct $table

// Azure Activity in last 24 hours
AzureActivity
| where TimeGenerated > ago(24h)
| project TimeGenerated, OperationName, Caller, ActivityStatus
| take 50

// Count operations by type
AzureActivity
| where TimeGenerated > ago(7d)
| summarize count() by OperationName
| top 10 by count_

// Heartbeat of VMs (if data exists)
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
```

### Step 5: Create a Dashboard
1. Click **"Dashboard"** in Portal menu
2. Click **"+ New dashboard"**
3. Name: `Lab-Monitoring-Dashboard`
4. Drag widgets:
   - Clock
   - Markdown (add notes)
5. Click **"Done customizing"**
6. Go to VM Metrics and pin charts to this dashboard

## Verification
✅ VM metrics visible
✅ Diagnostic settings configured
✅ Alert rule created
✅ Can query Log Analytics
✅ Dashboard created

---

# LAB 8: Azure Backup

## Objective
Configure backup for an Azure VM.

## Time
20 minutes

## Steps

### Step 1: Create Recovery Services Vault
1. Search for **"Recovery Services vaults"**
2. Click **"+ Create"**
   - Resource group: `Lab-RG-01`
   - Vault name: `Lab-RecoveryVault`
   - Region: `East US` (same as your VM!)
3. Click **"Review + Create"** → **"Create"**

### Step 2: Configure VM Backup
1. Go to `Lab-RecoveryVault`
2. Click **"+ Backup"**
3. **Where is your workload running?** Azure
4. **What do you want to back up?** Virtual machine
5. Click **"Backup"**
6. Select `myFirstVM`
7. Keep default policy (DefaultPolicy)
8. Click **"Enable Backup"**

### Step 3: Run Backup Now
1. In vault, go to **"Backup items"** → **"Azure Virtual Machine"**
2. Click on `myFirstVM`
3. Click **"Backup now"**
4. Select retention date (keep default)
5. Click **"OK"**
6. Monitor in **"Backup Jobs"**

### Step 4: View Backup Policy
1. In vault, click **"Backup policies"**
2. Click **"DefaultPolicy"**
3. Review:
   - Schedule: Daily at 7:30 PM
   - Retention: 30 days

### Step 5: Create Custom Policy
1. Click **"+ Add"**
2. Configure:
   - Policy name: `Lab-Weekly-Policy`
   - Backup schedule: Weekly (Sunday)
   - Instant Restore: 2 days
   - Retention: Weekly for 4 weeks
3. Click **"Create"**

### Step 6: View Recovery Points
1. Go to **"Backup items"** → **"Azure Virtual Machine"**
2. Click `myFirstVM`
3. Click **"View all Recovery Points"**
4. See available restore points

## Verification
✅ Recovery Services Vault created
✅ VM backup enabled
✅ Backup job completed
✅ Custom backup policy created

---

# LAB 9: Load Balancer

## Objective
Create a load balancer with backend VMs.

## Time
35 minutes

## Steps

### Step 1: Create Two Web Server VMs
```bash
# Create first web server
az vm create \
    --resource-group Lab-RG-01 \
    --name WebServer1 \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --vnet-name Lab-VNet \
    --subnet WebSubnet \
    --public-ip-sku Standard \
    --size Standard_B1s \
    --zone 1

# Create second web server
az vm create \
    --resource-group Lab-RG-01 \
    --name WebServer2 \
    --image Ubuntu2204 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --vnet-name Lab-VNet \
    --subnet WebSubnet \
    --public-ip-sku Standard \
    --size Standard_B1s \
    --zone 2

# Install nginx on both
az vm run-command invoke \
    --resource-group Lab-RG-01 \
    --name WebServer1 \
    --command-id RunShellScript \
    --scripts "sudo apt update && sudo apt install -y nginx && echo '<h1>Web Server 1</h1>' | sudo tee /var/www/html/index.html"

az vm run-command invoke \
    --resource-group Lab-RG-01 \
    --name WebServer2 \
    --command-id RunShellScript \
    --scripts "sudo apt update && sudo apt install -y nginx && echo '<h1>Web Server 2</h1>' | sudo tee /var/www/html/index.html"
```

### Step 2: Create Public IP for Load Balancer
```bash
az network public-ip create \
    --resource-group Lab-RG-01 \
    --name LB-PublicIP \
    --sku Standard \
    --zone 1 2 3
```

### Step 3: Create Load Balancer
1. Search for **"Load Balancers"**
2. Click **"+ Create"**
3. **Basics:**
   - Resource group: `Lab-RG-01`
   - Name: `Lab-LoadBalancer`
   - Region: `East US`
   - SKU: Standard
   - Type: Public
4. **Frontend IP:**
   - Name: `FrontEnd`
   - Public IP: `LB-PublicIP`
5. **Backend pools:**
   - Name: `WebServers-Pool`
   - Virtual network: `Lab-VNet`
   - Add VMs: `WebServer1`, `WebServer2`
6. **Inbound rules → Add load balancing rule:**
   - Name: `HTTP-Rule`
   - Frontend IP: `FrontEnd`
   - Backend pool: `WebServers-Pool`
   - Port: 80
   - Backend port: 80
   - Health probe: Create new
     - Name: `HTTP-Probe`
     - Protocol: HTTP
     - Port: 80
     - Path: /
7. Click **"Review + Create"** → **"Create"**

### Step 4: Update NSG Rules
```bash
# Allow HTTP traffic to WebSubnet
az network nsg rule create \
    --nsg-name WebSubnet-NSG \
    --resource-group Lab-RG-01 \
    --name Allow-LB-HTTP \
    --priority 90 \
    --source-address-prefixes AzureLoadBalancer \
    --destination-port-ranges 80 \
    --access Allow \
    --protocol Tcp \
    --direction Inbound
```

### Step 5: Test Load Balancer
1. Get the Load Balancer public IP
2. Open in browser: `http://<LB-PUBLIC-IP>`
3. Refresh multiple times - you should see "Web Server 1" and "Web Server 2" alternating

## Verification
✅ Two web servers created with nginx
✅ Load balancer distributing traffic
✅ Health probe monitoring backend

---

# LAB 10: Clean Up All Resources

## Objective
Delete all resources to avoid charges.

## Time
10 minutes

## Steps

### Method 1: Delete Resource Group (Easiest)
```bash
# This deletes EVERYTHING in the resource group
az group delete --name Lab-RG-01 --yes --no-wait

# Also delete CLI-Lab-RG if it exists
az group delete --name CLI-Lab-RG --yes --no-wait

# Verify deletion
az group list --output table
```

### Method 2: Using Portal
1. Go to **"Resource groups"**
2. Click `Lab-RG-01`
3. Click **"Delete resource group"**
4. Type the resource group name to confirm
5. Click **"Delete"**

### Don't Forget to Delete:
- ✅ VMs (and their disks, NICs, public IPs)
- ✅ Storage accounts
- ✅ Virtual networks
- ✅ Network security groups
- ✅ Load balancers
- ✅ Recovery Services vaults (need to stop backup first)
- ✅ Log Analytics workspaces
- ✅ Public IP addresses

### Check for Orphaned Resources
1. Go to **"All resources"** in Portal
2. Filter by subscription
3. Delete any remaining resources

### Verify Monthly Costs
1. Go to **"Cost Management + Billing"**
2. Check that no unexpected charges appear

## Final Verification
✅ All resource groups deleted
✅ No resources remaining
✅ No unexpected charges

---

# Next Steps After Labs

## Continue Learning
1. ✅ Complete AZ-104 exam study guide modules
2. ✅ Practice with more complex scenarios
3. ✅ Take Microsoft Learn free courses
4. ✅ Schedule your AZ-104 exam

## More Practice Ideas
- Deploy a multi-tier application
- Set up VPN connectivity
- Implement Azure AD B2B
- Create ARM/Bicep templates
- Set up CI/CD with Azure DevOps

## Microsoft Learn Sandboxes
- Free hands-on environments
- No Azure subscription needed
- Access at: https://docs.microsoft.com/learn

---

*Congratulations on completing the hands-on labs! Regular practice is key to mastering Azure administration.*
