# Module 04 - Azure Virtual Machines

## Learning Objectives
By the end of this module, you will be able to:
- Create and configure Azure Virtual Machines
- Understand and manage Data Disks
- Configure VM networking and interfaces
- Work with ARM templates for VM deployment
- Create and use VHD templates
- Build custom VM images
- Implement Virtual Machine Scale Sets
- Configure Availability Sets for high availability

---

## 4.1 Azure Virtual Machines

### What is an Azure Virtual Machine?
Azure Virtual Machines (VMs) are on-demand, scalable computing resources offered by Azure. They provide the flexibility of virtualization without having to buy and maintain physical hardware.

### VM Components

```
┌────────────────────────────────────────────────────────────────┐
│                    Azure Virtual Machine                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Operating System                       │  │
│  │                 (Windows Server, Linux)                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐              │
│  │   OS Disk  │  │ Data Disks │  │  Temp Disk │              │
│  │  (Required)│  │ (Optional) │  │ (Ephemeral)│              │
│  └────────────┘  └────────────┘  └────────────┘              │
│                              │                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Network Interface Card (NIC)                 │  │
│  │        Private IP | Public IP (optional) | NSG           │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Virtual Network                        │  │
│  │                       (Subnet)                            │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
```

### VM Size Categories

| Category | Use Case | Examples |
|----------|----------|----------|
| **General Purpose** | Balanced CPU-to-memory, testing, dev | B, D, DS, Dv2, Dv3, Dav4 |
| **Compute Optimized** | High CPU-to-memory ratio, batch processing | F, Fs, Fsv2 |
| **Memory Optimized** | High memory-to-CPU ratio, databases | E, Es, Ev3, M, Mv2 |
| **Storage Optimized** | High disk throughput and I/O | Ls, Lsv2 |
| **GPU** | Graphics rendering, deep learning | NC, NCv2, NCv3, ND, NV |
| **High Performance Compute** | Fastest CPUs, HPC workloads | H, HB, HC |

#### 🏢 Real-World VM Size Selection Use Cases:

**E-Commerce Platform - Mixed Workloads:**
```
┌─────────────────────────────────────────────────────────────────┐
│                    Production Architecture                       │
├─────────────────────────────────────────────────────────────────┤
│ Web Tier:          D4s_v3 (4 vCPU, 16GB)    x 4 instances      │
│ ├── Why: Balanced for PHP/Node.js web servers                  │
│ └── Handles: 10,000 concurrent users                           │
│                                                                  │
│ API Tier:          F4s_v2 (4 vCPU, 8GB)     x 6 instances      │
│ ├── Why: CPU-intensive API processing                          │
│ └── Handles: REST API calls, JSON serialization                │
│                                                                  │
│ Database Tier:     E8s_v3 (8 vCPU, 64GB)    x 2 instances      │
│ ├── Why: Memory-optimized for SQL Server caching               │
│ └── Handles: 500GB database in-memory queries                  │
│                                                                  │
│ Search Tier:       L8s_v2 (8 vCPU, 64GB)    x 3 instances      │
│ ├── Why: NVMe storage for Elasticsearch                        │
│ └── Handles: 10TB search index, 1M+ products                   │
└─────────────────────────────────────────────────────────────────┘
Monthly Cost: ~$8,500 (with reserved instances: ~$5,100)
```

**AI/ML Training Pipeline:**
```
Company: AutoVision (Self-driving car startup)
Workload: Train computer vision models

VM Selection:
├── Training: NC24s_v3 (4x Tesla V100 GPUs)
│   └── Use: Deep learning model training (PyTorch/TensorFlow)
│   └── Duration: 72-hour training jobs
│   └── Cost: $8.28/hour → Use Spot VMs at $1.65/hour (80% savings)
│
├── Inference: NC6s_v3 (1x Tesla V100)
│   └── Use: Real-time object detection
│   └── Always-on for production
│
└── Data Processing: HB120rs_v2 (120 cores)
    └── Use: Parallel video frame extraction
    └── Process 1TB video data in 2 hours
```

**Financial Services - SAP Deployment:**
```
Company: MegaCorp Manufacturing
System: SAP S/4HANA with 1.5TB database

VM Selection:
├── Production: M128s (128 vCPU, 2TB RAM)
│   └── Certified for SAP HANA
│   └── Premium SSD: 4x P30 (4TB total)
│   └── Cost: $13,000/month
│
├── DR (Disaster Recovery): M64s (64 vCPU, 1TB RAM)
│   └── Azure Site Recovery enabled
│   └── RPO: 15 minutes, RTO: 2 hours
│
└── Dev/Test: E32s_v3 (scaled-down copy)
    └── Use Spot VMs for dev environments
    └── Auto-shutdown at 7 PM daily
```

### VM Naming Convention

```
VM Size: Standard_D4s_v3

Standard   = Tier
D          = Family (General Purpose)
4          = vCPUs
s          = Premium Storage capable
v3         = Version
```

### Common VM Sizes

| Size | vCPUs | Memory | Temp Storage | Max Data Disks |
|------|-------|--------|--------------|----------------|
| Standard_B2s | 2 | 4 GB | 8 GB | 4 |
| Standard_D2s_v3 | 2 | 8 GB | 16 GB | 4 |
| Standard_D4s_v3 | 4 | 16 GB | 32 GB | 8 |
| Standard_E4s_v3 | 4 | 32 GB | 64 GB | 8 |
| Standard_F4s_v2 | 4 | 8 GB | 32 GB | 8 |

### Creating a Virtual Machine

#### Using Azure Portal
1. Navigate to "Virtual Machines"
2. Click "Create" → "Azure Virtual Machine"
3. Configure:
   - Basics: Subscription, Resource Group, VM name, Region, Image, Size
   - Disks: OS disk type, Data disks
   - Networking: VNet, Subnet, Public IP, NSG
   - Management: Monitoring, Auto-shutdown
   - Advanced: Extensions, Cloud-init

#### Using PowerShell

```powershell
# Variables
$resourceGroup = "VM-RG"
$location = "East US"
$vmName = "WebServer-VM"
$adminUser = "azureuser"
$adminPassword = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force

# Create resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create VM
New-AzVM `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -VirtualNetworkName "MyVNet" `
  -SubnetName "default" `
  -SecurityGroupName "MyNSG" `
  -PublicIpAddressName "MyPublicIP" `
  -OpenPorts 80,3389 `
  -Size "Standard_D2s_v3" `
  -Credential (New-Object PSCredential($adminUser, $adminPassword)) `
  -Image "Win2019Datacenter"
```

#### Using Azure CLI

```bash
# Create resource group
az group create --name "VM-RG" --location "eastus"

# Create VM
az vm create \
  --resource-group "VM-RG" \
  --name "WebServer-VM" \
  --image "Win2019Datacenter" \
  --admin-username "azureuser" \
  --admin-password "P@ssw0rd123!" \
  --size "Standard_D2s_v3" \
  --public-ip-sku "Standard"
```

### VM Operations

```powershell
# Start VM
Start-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM"

# Stop VM (still billed for compute, just not running)
Stop-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM"

# Deallocate VM (stop billing)
Stop-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM" -Force

# Restart VM
Restart-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM"

# Delete VM
Remove-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM" -Force

# Resize VM
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM"
$vm.HardwareProfile.VmSize = "Standard_D4s_v3"
Update-AzVM -VM $vm -ResourceGroupName "VM-RG"
```

### VM States

| State | Description | Billed |
|-------|-------------|--------|
| **Starting** | VM is starting | Yes |
| **Running** | VM is running | Yes |
| **Stopping** | VM is stopping | Yes |
| **Stopped** | OS stopped, still allocated | Yes |
| **Deallocated** | Resources released | No (except storage) |

---

## 4.2 Data Disks in Azure

### Types of Disks

| Disk Type | Description | Managed |
|-----------|-------------|---------|
| **OS Disk** | Contains operating system | Yes |
| **Data Disk** | Additional storage for applications/data | Yes |
| **Temporary Disk** | Non-persistent, for temp data | No |

### Disk Storage Types

| Type | Use Case | IOPS | Throughput | Latency |
|------|----------|------|------------|---------|
| **Ultra Disk** | Mission-critical, SAP HANA | Up to 160,000 | Up to 2,000 MB/s | Sub-ms |
| **Premium SSD v2** | Production workloads | Up to 80,000 | Up to 1,200 MB/s | Sub-ms |
| **Premium SSD** | Production, latency-sensitive | Up to 20,000 | Up to 900 MB/s | Single-digit ms |
| **Standard SSD** | Dev/test, web servers | Up to 6,000 | Up to 750 MB/s | Single-digit ms |
| **Standard HDD** | Backup, infrequent access | Up to 2,000 | Up to 500 MB/s | Variable |

### Disk Size Tiers (Premium SSD)

| Size | Capacity | IOPS | Throughput |
|------|----------|------|------------|
| P4 | 32 GB | 120 | 25 MB/s |
| P10 | 128 GB | 500 | 100 MB/s |
| P20 | 512 GB | 2,300 | 150 MB/s |
| P30 | 1 TB | 5,000 | 200 MB/s |
| P40 | 2 TB | 7,500 | 250 MB/s |
| P50 | 4 TB | 7,500 | 250 MB/s |

### Managing Data Disks

#### Create and Attach Data Disk

```powershell
# Create managed disk
$diskConfig = New-AzDiskConfig `
  -Location "East US" `
  -CreateOption Empty `
  -DiskSizeGB 128 `
  -SkuName "Premium_LRS"

$dataDisk = New-AzDisk `
  -ResourceGroupName "VM-RG" `
  -DiskName "DataDisk-01" `
  -Disk $diskConfig

# Attach to VM
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM"
$vm = Add-AzVMDataDisk `
  -VM $vm `
  -Name "DataDisk-01" `
  -CreateOption Attach `
  -ManagedDiskId $dataDisk.Id `
  -Lun 0

Update-AzVM -VM $vm -ResourceGroupName "VM-RG"
```

```bash
# Azure CLI
az disk create \
  --resource-group "VM-RG" \
  --name "DataDisk-01" \
  --size-gb 128 \
  --sku "Premium_LRS"

az vm disk attach \
  --resource-group "VM-RG" \
  --vm-name "WebServer-VM" \
  --name "DataDisk-01"
```

#### Initialize Disk in Windows

```powershell
# Connect to VM and run in PowerShell
Get-Disk | Where-Object PartitionStyle -eq 'RAW' | 
  Initialize-Disk -PartitionStyle GPT -PassThru | 
  New-Partition -AssignDriveLetter -UseMaximumSize | 
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
```

#### Initialize Disk in Linux

```bash
# Find new disk
lsblk

# Create partition
sudo fdisk /dev/sdc
# Press: n (new), p (primary), Enter (defaults), w (write)

# Create filesystem
sudo mkfs.ext4 /dev/sdc1

# Mount disk
sudo mkdir /datadrive
sudo mount /dev/sdc1 /datadrive

# Add to fstab for persistence
echo "/dev/sdc1 /datadrive ext4 defaults 0 0" | sudo tee -a /etc/fstab
```

### Disk Encryption

| Type | Description | Key Management |
|------|-------------|----------------|
| **Azure Disk Encryption (ADE)** | BitLocker (Windows) / DM-Crypt (Linux) | Azure Key Vault |
| **Server-Side Encryption (SSE)** | Encryption at rest by default | Platform-managed or customer-managed |
| **Host-based Encryption** | Encryption at the host level | Temp disks + caches encrypted |

```powershell
# Enable Azure Disk Encryption
$keyVault = Get-AzKeyVault -VaultName "MyKeyVault" -ResourceGroupName "KeyVault-RG"

Set-AzVMDiskEncryptionExtension `
  -ResourceGroupName "VM-RG" `
  -VMName "WebServer-VM" `
  -DiskEncryptionKeyVaultUrl $keyVault.VaultUri `
  -DiskEncryptionKeyVaultId $keyVault.ResourceId
```

### Disk Snapshots

```powershell
# Create snapshot
$disk = Get-AzDisk -ResourceGroupName "VM-RG" -DiskName "OsDisk-01"

$snapshotConfig = New-AzSnapshotConfig `
  -SourceUri $disk.Id `
  -Location "East US" `
  -CreateOption Copy

$snapshot = New-AzSnapshot `
  -ResourceGroupName "VM-RG" `
  -SnapshotName "OsDisk-Snapshot-$(Get-Date -Format 'yyyyMMdd')" `
  -Snapshot $snapshotConfig
```

---

## 4.3 Azure VMs & Interfaces

### Network Interface Card (NIC)

Every VM requires at least one NIC. The NIC provides:
- Private IP address (required)
- Public IP address (optional)
- Network Security Group association
- DNS settings

### NIC Configuration

```
┌─────────────────────────────────────────────────────────────┐
│                    Network Interface                         │
│                                                              │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ IP Configuration│  │ IP Configuration│  (Multiple IPs)  │
│  │  Primary        │  │  Secondary      │                   │
│  │  10.0.0.4       │  │  10.0.0.5       │                   │
│  └─────────────────┘  └─────────────────┘                   │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────────────────┐                    │
│  │         Network Security Group      │                    │
│  │  (Inbound/Outbound Rules)           │                    │
│  └─────────────────────────────────────┘                    │
│           │                                                  │
│           ▼                                                  │
│  ┌─────────────────────────────────────┐                    │
│  │     Virtual Network / Subnet        │                    │
│  └─────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

### Creating a NIC

```powershell
# Create NIC
$subnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet" | 
          Get-AzVirtualNetworkSubnetConfig -Name "default"

$nic = New-AzNetworkInterface `
  -Name "MyNIC" `
  -ResourceGroupName "VM-RG" `
  -Location "East US" `
  -SubnetId $subnet.Id

# Create NIC with public IP
$publicIP = New-AzPublicIpAddress `
  -Name "MyPublicIP" `
  -ResourceGroupName "VM-RG" `
  -Location "East US" `
  -AllocationMethod Static `
  -Sku Standard

$nic = New-AzNetworkInterface `
  -Name "MyNIC-Public" `
  -ResourceGroupName "VM-RG" `
  -Location "East US" `
  -SubnetId $subnet.Id `
  -PublicIpAddressId $publicIP.Id
```

### Multiple NICs

VMs can have multiple NICs (number depends on VM size):

| VM Size | Max NICs |
|---------|----------|
| Standard_B2s | 2 |
| Standard_D4s_v3 | 2 |
| Standard_D8s_v3 | 4 |
| Standard_D16s_v3 | 8 |

```powershell
# Add secondary NIC to existing VM (VM must be deallocated)
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "MultiNIC-VM"

$subnet2 = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet" | 
           Get-AzVirtualNetworkSubnetConfig -Name "BackendSubnet"

$nic2 = New-AzNetworkInterface `
  -Name "SecondaryNIC" `
  -ResourceGroupName "VM-RG" `
  -Location "East US" `
  -SubnetId $subnet2.Id

Add-AzVMNetworkInterface -VM $vm -Id $nic2.Id
Update-AzVM -VM $vm -ResourceGroupName "VM-RG"
```

### Accelerated Networking

Accelerated Networking enables single root I/O virtualization (SR-IOV) for improved network performance.

**Benefits:**
- Lower latency
- Higher packets per second
- Consistent latency

```powershell
# Enable on existing NIC
$nic = Get-AzNetworkInterface -ResourceGroupName "VM-RG" -Name "MyNIC"
$nic.EnableAcceleratedNetworking = $true
$nic | Set-AzNetworkInterface
```

---

## 4.4 ARM Templates

### ARM Template for VM Deployment

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "Name of the virtual machine"
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "allowedValues": [
        "Standard_B2s",
        "Standard_D2s_v3",
        "Standard_D4s_v3"
      ]
    }
  },
  "variables": {
    "virtualNetworkName": "[concat(parameters('vmName'), '-vnet')]",
    "subnetName": "default",
    "networkInterfaceName": "[concat(parameters('vmName'), '-nic')]",
    "publicIPAddressName": "[concat(parameters('vmName'), '-pip')]",
    "networkSecurityGroupName": "[concat(parameters('vmName'), '-nsg')]"
  },
  "resources": [
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-02-01",
      "name": "[variables('networkSecurityGroupName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": [
          {
            "name": "AllowRDP",
            "properties": {
              "priority": 1000,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "3389"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "[variables('virtualNetworkName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "10.0.0.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-02-01",
      "name": "[variables('publicIPAddressName')]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-02-01",
      "name": "[variables('networkInterfaceName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))]"
              }
            }
          }
        ],
        "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('networkSecurityGroupName'))]"
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-07-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('networkInterfaceName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2019-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            }
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('networkInterfaceName'))]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "publicIPAddress": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPAddressName'))).ipAddress]"
    }
  }
}
```

### Deploying ARM Template

```powershell
# Deploy using PowerShell
New-AzResourceGroupDeployment `
  -ResourceGroupName "VM-RG" `
  -TemplateFile ".\vm-template.json" `
  -vmName "ProductionVM" `
  -adminUsername "azureadmin" `
  -vmSize "Standard_D4s_v3"

# Prompt for password
```

```bash
# Deploy using Azure CLI
az deployment group create \
  --resource-group "VM-RG" \
  --template-file "vm-template.json" \
  --parameters vmName="ProductionVM" adminUsername="azureadmin"
```

### Exporting ARM Template from Existing VM

```powershell
# Export template of existing resource group
Export-AzResourceGroup `
  -ResourceGroupName "VM-RG" `
  -Path ".\exported-template.json"
```

---

## 4.5 VHD Templates

### What is a VHD?
Virtual Hard Disk (VHD) is the disk format used by Azure VMs. It contains the OS and data.

### VHD Types

| Type | Description |
|------|-------------|
| **Fixed VHD** | Disk size is allocated upfront |
| **Dynamic VHD** | Grows as data is written (not supported for Azure VMs) |

### Uploading VHD to Azure

```powershell
# Prepare VHD (must be fixed size, generation 1)
# Convert if needed using Hyper-V Manager or PowerShell

# Upload VHD to blob storage
$context = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount").Context

Add-AzVhd `
  -ResourceGroupName "VM-RG" `
  -Destination "https://mystorageaccount.blob.core.windows.net/vhds/myimage.vhd" `
  -LocalFilePath "C:\VMs\myimage.vhd"
```

### Creating VM from VHD

```powershell
# Create managed disk from VHD
$diskConfig = New-AzDiskConfig `
  -Location "East US" `
  -CreateOption Import `
  -SourceUri "https://mystorageaccount.blob.core.windows.net/vhds/myimage.vhd" `
  -StorageAccountId "/subscriptions/.../storageAccounts/mystorageaccount"

$osDisk = New-AzDisk `
  -ResourceGroupName "VM-RG" `
  -DiskName "ImportedOSDisk" `
  -Disk $diskConfig

# Create VM from disk
$vm = New-AzVMConfig -VMName "ImportedVM" -VMSize "Standard_D2s_v3"
$vm = Set-AzVMOSDisk -VM $vm -ManagedDiskId $osDisk.Id -CreateOption Attach -Windows

# Add NIC and create VM
$vm = Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
New-AzVM -ResourceGroupName "VM-RG" -Location "East US" -VM $vm
```

---

## 4.6 Custom Images of Azure VM

### What is a Custom Image?
A custom image is a snapshot of a VM that can be used to create new VMs with the same configuration.

### Generalization Process

#### Windows (Sysprep)
```powershell
# Inside the VM, run:
C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown
```

#### Linux
```bash
# Inside the VM, run:
sudo waagent -deprovision+user -force
```

### Creating Custom Image

```powershell
# Deallocate VM
Stop-AzVM -ResourceGroupName "VM-RG" -Name "TemplateVM" -Force

# Generalize VM (marks it as template)
Set-AzVm -ResourceGroupName "VM-RG" -Name "TemplateVM" -Generalized

# Create image
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "TemplateVM"

$imageConfig = New-AzImageConfig -Location "East US" -SourceVirtualMachineId $vm.Id

$image = New-AzImage `
  -ResourceGroupName "Images-RG" `
  -ImageName "WebServer-Image-v1" `
  -Image $imageConfig
```

### Azure Compute Gallery (Shared Image Gallery)

For enterprise image management:

```powershell
# Create gallery
$gallery = New-AzGallery `
  -ResourceGroupName "Gallery-RG" `
  -Name "EnterpriseGallery" `
  -Location "East US" `
  -Description "Shared images for organization"

# Create image definition
$imageDefinition = New-AzGalleryImageDefinition `
  -ResourceGroupName "Gallery-RG" `
  -GalleryName "EnterpriseGallery" `
  -Name "WebServer" `
  -Location "East US" `
  -Publisher "Contoso" `
  -Offer "WebServers" `
  -Sku "Standard" `
  -OsState "Generalized" `
  -OsType "Windows"

# Create image version
$imageVersion = New-AzGalleryImageVersion `
  -ResourceGroupName "Gallery-RG" `
  -GalleryName "EnterpriseGallery" `
  -GalleryImageDefinitionName "WebServer" `
  -Name "1.0.0" `
  -Location "East US" `
  -SourceImageId $image.Id
```

### Creating VM from Custom Image

```powershell
# From managed image
New-AzVM `
  -ResourceGroupName "VM-RG" `
  -Name "NewWebServer" `
  -Location "East US" `
  -ImageName "WebServer-Image-v1" `
  -Size "Standard_D2s_v3" `
  -Credential $cred

# From gallery image
$galleryImage = Get-AzGalleryImageVersion `
  -ResourceGroupName "Gallery-RG" `
  -GalleryName "EnterpriseGallery" `
  -GalleryImageDefinitionName "WebServer" `
  -Name "1.0.0"

New-AzVM `
  -ResourceGroupName "VM-RG" `
  -Name "NewWebServer2" `
  -Location "East US" `
  -Image $galleryImage.Id `
  -Size "Standard_D2s_v3" `
  -Credential $cred
```

---

## 4.7 Virtual Machine Scale Sets

### What is a VM Scale Set?
Virtual Machine Scale Sets (VMSS) allow you to create and manage a group of identical, load-balanced VMs. The number of VM instances can automatically increase or decrease in response to demand.

### VMSS Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    VM Scale Set                                  │
│                                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                   Load Balancer                        │     │
│   └───────────────────────────────────────────────────────┘     │
│              │            │            │                         │
│              ▼            ▼            ▼                         │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│   │   VM 0       │ │   VM 1       │ │   VM 2       │            │
│   │              │ │              │ │              │            │
│   │  ┌────────┐  │ │  ┌────────┐  │ │  ┌────────┐  │            │
│   │  │ OS Disk│  │ │  │ OS Disk│  │ │  │ OS Disk│  │            │
│   │  └────────┘  │ │  └────────┘  │ │  └────────┘  │            │
│   └──────────────┘ └──────────────┘ └──────────────┘            │
│                                                                  │
│   ┌───────────────────────────────────────────────────────┐     │
│   │               Autoscale Rules                          │     │
│   │   Scale Out: CPU > 70% → Add instance                 │     │
│   │   Scale In:  CPU < 30% → Remove instance              │     │
│   └───────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

### Creating VM Scale Set

```powershell
# Create VMSS
$vmssConfig = New-AzVmssConfig `
  -Location "East US" `
  -SkuCapacity 3 `
  -SkuName "Standard_D2s_v3" `
  -UpgradePolicyMode "Automatic"

# Set OS profile
Set-AzVmssOsProfile `
  -VirtualMachineScaleSet $vmssConfig `
  -ComputerNamePrefix "webvm" `
  -AdminUsername "azureuser" `
  -AdminPassword "P@ssw0rd123!"

# Set image reference
Set-AzVmssStorageProfile `
  -VirtualMachineScaleSet $vmssConfig `
  -ImageReferencePublisher "MicrosoftWindowsServer" `
  -ImageReferenceOffer "WindowsServer" `
  -ImageReferenceSku "2019-Datacenter" `
  -ImageReferenceVersion "latest" `
  -OsDiskCreateOption "FromImage"

# Set network profile
$ipConfig = New-AzVmssIPConfig `
  -Name "vmssIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -SubnetId $subnet.Id

Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "vmssNIC" `
  -Primary $true `
  -IPConfiguration $ipConfig

# Create VMSS
New-AzVmss `
  -ResourceGroupName "VMSS-RG" `
  -Name "WebVMSS" `
  -VirtualMachineScaleSet $vmssConfig
```

```bash
# Azure CLI
az vmss create \
  --resource-group "VMSS-RG" \
  --name "WebVMSS" \
  --image "Win2019Datacenter" \
  --instance-count 3 \
  --vm-sku "Standard_D2s_v3" \
  --admin-username "azureuser" \
  --admin-password "P@ssw0rd123!" \
  --upgrade-policy-mode "Automatic" \
  --load-balancer "WebLB"
```

### Autoscale Configuration

```powershell
# Add autoscale settings
$rule1 = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId "/subscriptions/.../virtualMachineScaleSets/WebVMSS" `
  -TimeGrain 00:01:00 `
  -MetricStatistic "Average" `
  -TimeWindow 00:05:00 `
  -Operator "GreaterThan" `
  -Threshold 70 `
  -ScaleActionDirection "Increase" `
  -ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

$rule2 = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId "/subscriptions/.../virtualMachineScaleSets/WebVMSS" `
  -TimeGrain 00:01:00 `
  -MetricStatistic "Average" `
  -TimeWindow 00:05:00 `
  -Operator "LessThan" `
  -Threshold 30 `
  -ScaleActionDirection "Decrease" `
  -ScaleActionScaleType "ChangeCount" `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

$profile = New-AzAutoscaleProfile `
  -DefaultCapacity 3 `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $rule1, $rule2 `
  -Name "ScaleProfile"

Add-AzAutoscaleSetting `
  -ResourceGroupName "VMSS-RG" `
  -Name "WebVMSSAutoscale" `
  -Location "East US" `
  -TargetResourceId "/subscriptions/.../virtualMachineScaleSets/WebVMSS" `
  -AutoscaleProfile $profile
```

### VMSS Upgrade Modes

| Mode | Behavior |
|------|----------|
| **Manual** | Instances not upgraded automatically |
| **Automatic** | All instances upgraded immediately |
| **Rolling** | Instances upgraded in batches |

---

## 4.8 Virtual Machine Availability Sets

### What is an Availability Set?
Availability Sets ensure that VMs are distributed across multiple fault domains and update domains within a datacenter to protect against hardware failures and planned maintenance.

### Key Concepts

| Concept | Description | Max |
|---------|-------------|-----|
| **Fault Domain (FD)** | VMs that share common power source and network switch | 3 |
| **Update Domain (UD)** | VMs that can be rebooted together during maintenance | 20 |

### Availability Set Architecture

```
┌────────────────────────── Availability Set ──────────────────────────┐
│                                                                       │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │   Fault Domain 0 │  │   Fault Domain 1 │  │   Fault Domain 2 │   │
│  │                  │  │                  │  │                  │   │
│  │  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │   │
│  │  │UD 0: VM1   │  │  │  │UD 0: VM2   │  │  │  │UD 1: VM3   │  │   │
│  │  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │   │
│  │                  │  │                  │  │                  │   │
│  │  ┌────────────┐  │  │  ┌────────────┐  │  │  ┌────────────┐  │   │
│  │  │UD 1: VM4   │  │  │  │UD 2: VM5   │  │  │  │UD 2: VM6   │  │   │
│  │  └────────────┘  │  │  └────────────┘  │  │  └────────────┘  │   │
│  │                  │  │                  │  │                  │   │
│  │  Power Supply A  │  │  Power Supply B  │  │  Power Supply C  │   │
│  │  Network Switch A│  │  Network Switch B│  │  Network Switch C│   │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘   │
└───────────────────────────────────────────────────────────────────────┘
```

### Creating Availability Set

```powershell
# Create availability set
$availabilitySet = New-AzAvailabilitySet `
  -ResourceGroupName "HA-RG" `
  -Name "WebAvailabilitySet" `
  -Location "East US" `
  -PlatformFaultDomainCount 3 `
  -PlatformUpdateDomainCount 5 `
  -Sku "Aligned"

# Create VM in availability set
$vm = New-AzVMConfig `
  -VMName "WebVM1" `
  -VMSize "Standard_D2s_v3" `
  -AvailabilitySetId $availabilitySet.Id

# Continue VM configuration and create...
```

```bash
# Azure CLI
az vm availability-set create \
  --resource-group "HA-RG" \
  --name "WebAvailabilitySet" \
  --platform-fault-domain-count 3 \
  --platform-update-domain-count 5

# Create VM in availability set
az vm create \
  --resource-group "HA-RG" \
  --name "WebVM1" \
  --availability-set "WebAvailabilitySet" \
  --image "Win2019Datacenter" \
  --size "Standard_D2s_v3" \
  --admin-username "azureuser" \
  --admin-password "P@ssw0rd123!"
```

### Availability Sets vs Availability Zones

| Feature | Availability Sets | Availability Zones |
|---------|-------------------|-------------------|
| Scope | Single datacenter | Multiple datacenters |
| SLA | 99.95% | 99.99% |
| Fault Domains | Up to 3 | 3 zones |
| Network latency | Lowest | Low |
| Cost | No additional | No additional |

### When to Use What

| Scenario | Solution |
|----------|----------|
| Protect against hardware failure | Availability Set |
| Protect against datacenter failure | Availability Zone |
| Global disaster recovery | Multiple regions |
| Auto-scaling | VM Scale Set |

---

## Hands-on Exercises

### Exercise 1: Creating and Configuring an Azure VM

```powershell
# Complete VM creation script
$resourceGroup = "Lab-VM-RG"
$location = "East US"
$vmName = "LabWebServer"

# Create resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create VM with all components
$cred = Get-Credential -Message "Enter admin credentials"

New-AzVM `
  -ResourceGroupName $resourceGroup `
  -Name $vmName `
  -Location $location `
  -Image "Win2019Datacenter" `
  -Size "Standard_D2s_v3" `
  -Credential $cred `
  -VirtualNetworkName "$vmName-VNet" `
  -SubnetName "default" `
  -SecurityGroupName "$vmName-NSG" `
  -PublicIpAddressName "$vmName-PIP" `
  -OpenPorts 80,443,3389

# Add data disk
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$vm = Add-AzVMDataDisk -VM $vm -Name "$vmName-DataDisk" -DiskSizeInGB 128 -Lun 0 -CreateOption Empty
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup

# Install IIS using Custom Script Extension
Set-AzVMExtension `
  -ResourceGroupName $resourceGroup `
  -VMName $vmName `
  -Name "IIS" `
  -Publisher "Microsoft.Compute" `
  -ExtensionType "CustomScriptExtension" `
  -TypeHandlerVersion "1.10" `
  -Settings @{commandToExecute = "powershell Install-WindowsFeature -name Web-Server -IncludeManagementTools"}
```

### Exercise 2: Deploying a Custom Image of Azure VM

```powershell
# Step 1: Create base VM and configure
# ... (create VM with your required software)

# Step 2: Generalize the VM
# Inside VM PowerShell:
# C:\Windows\System32\Sysprep\sysprep.exe /oobe /generalize /shutdown

# Step 3: Create image from generalized VM
Stop-AzVM -ResourceGroupName "Lab-VM-RG" -Name "TemplateVM" -Force
Set-AzVm -ResourceGroupName "Lab-VM-RG" -Name "TemplateVM" -Generalized

$vm = Get-AzVM -ResourceGroupName "Lab-VM-RG" -Name "TemplateVM"
$image = New-AzImage `
  -ResourceGroupName "Images-RG" `
  -ImageName "CustomWebServer-Image" `
  -Image (New-AzImageConfig -Location "East US" -SourceVirtualMachineId $vm.Id)

# Step 4: Deploy VM from custom image
New-AzVM `
  -ResourceGroupName "Prod-RG" `
  -Name "ProdWebServer1" `
  -Location "East US" `
  -Image $image.Id `
  -Size "Standard_D2s_v3" `
  -Credential (Get-Credential) `
  -VirtualNetworkName "Prod-VNet" `
  -SubnetName "WebSubnet"
```

### Exercise 3: Virtual Machine Scale Sets

```powershell
# Create complete VMSS with load balancer and autoscale

# Create resource group
New-AzResourceGroup -Name "VMSS-Lab-RG" -Location "East US"

# Create VMSS using simplified command
New-AzVmss `
  -ResourceGroupName "VMSS-Lab-RG" `
  -VMScaleSetName "WebVMSS" `
  -Location "East US" `
  -VirtualNetworkName "VMSS-VNet" `
  -SubnetName "default" `
  -PublicIpAddressName "VMSS-PIP" `
  -LoadBalancerName "VMSS-LB" `
  -UpgradePolicyMode "Automatic" `
  -InstanceCount 3 `
  -VMSize "Standard_D2s_v3" `
  -Credential (Get-Credential) `
  -ImageName "Win2019Datacenter"

# Configure autoscale
$vmss = Get-AzVmss -ResourceGroupName "VMSS-Lab-RG" -VMScaleSetName "WebVMSS"

# Scale out rule
$scaleOutRule = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $vmss.Id `
  -TimeGrain 00:01:00 `
  -MetricStatistic Average `
  -TimeWindow 00:05:00 `
  -Operator GreaterThan `
  -Threshold 70 `
  -ScaleActionDirection Increase `
  -ScaleActionScaleType ChangeCount `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

# Scale in rule
$scaleInRule = New-AzAutoscaleRule `
  -MetricName "Percentage CPU" `
  -MetricResourceId $vmss.Id `
  -TimeGrain 00:01:00 `
  -MetricStatistic Average `
  -TimeWindow 00:05:00 `
  -Operator LessThan `
  -Threshold 30 `
  -ScaleActionDirection Decrease `
  -ScaleActionScaleType ChangeCount `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

$profile = New-AzAutoscaleProfile `
  -DefaultCapacity 3 `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $scaleOutRule,$scaleInRule `
  -Name "WebProfile"

Add-AzAutoscaleSetting `
  -ResourceGroupName "VMSS-Lab-RG" `
  -Name "WebVMSSAutoscale" `
  -Location "East US" `
  -TargetResourceId $vmss.Id `
  -AutoscaleProfile $profile

# Test: Install CPU stress tool and verify scaling
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Azure VMs | IaaS compute, multiple sizes/families, various OS options |
| Data Disks | OS/Data/Temp disks, Premium SSD for production |
| NIC & Interfaces | Private/Public IPs, NSG, multiple NICs, accelerated networking |
| ARM Templates | Infrastructure as code, repeatable deployments |
| VHD Templates | Import on-premises VMs, fixed VHD format |
| Custom Images | Sysprep/deprovision, Azure Compute Gallery for sharing |
| Scale Sets | Auto-scaling VMs, load balanced, update policies |
| Availability Sets | Fault/Update domains, 99.95% SLA |

---

## Review Questions

1. What is the difference between Stopped and Deallocated VM states?
2. Explain the difference between Premium SSD and Standard SSD.
3. What is the purpose of Sysprep before creating a custom image?
4. How do Fault Domains and Update Domains work in Availability Sets?
5. When would you use VM Scale Sets vs. individual VMs?
6. What is the benefit of using Azure Compute Gallery?

---

## Additional Resources

- [Azure VM Documentation](https://docs.microsoft.com/azure/virtual-machines/)
- [VM Sizes](https://docs.microsoft.com/azure/virtual-machines/sizes)
- [Managed Disks](https://docs.microsoft.com/azure/virtual-machines/managed-disks-overview)
- [VM Scale Sets](https://docs.microsoft.com/azure/virtual-machine-scale-sets/)
- [Availability Options](https://docs.microsoft.com/azure/virtual-machines/availability)

---

## 4.8 Advanced VM Management (Expert Level)

### VM Extensions for Automation

```powershell
# Install Custom Script Extension
Set-AzVMExtension `
    -ResourceGroupName "VM-RG" `
    -VMName "WebServer-VM" `
    -Name "CustomScriptExtension" `
    -Publisher "Microsoft.Compute" `
    -ExtensionType "CustomScriptExtension" `
    -TypeHandlerVersion "1.10" `
    -Settings @{
        "fileUris" = @("https://mystorageaccount.blob.core.windows.net/scripts/setup.ps1")
        "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File setup.ps1"
    }

# Install Azure Monitor Agent
Set-AzVMExtension `
    -ResourceGroupName "VM-RG" `
    -VMName "WebServer-VM" `
    -Name "AzureMonitorWindowsAgent" `
    -Publisher "Microsoft.Azure.Monitor" `
    -ExtensionType "AzureMonitorWindowsAgent" `
    -TypeHandlerVersion "1.0" `
    -EnableAutomaticUpgrade $true

# Install Dependency Agent for VM Insights
Set-AzVMExtension `
    -ResourceGroupName "VM-RG" `
    -VMName "WebServer-VM" `
    -Name "DependencyAgentWindows" `
    -Publisher "Microsoft.Azure.Monitoring.DependencyAgent" `
    -ExtensionType "DependencyAgentWindows" `
    -TypeHandlerVersion "9.10"
```

### Update Management

```powershell
# Enable Update Management on VM
$automationAccount = Get-AzAutomationAccount -ResourceGroupName "Automation-RG" -Name "MyAutomation"
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "Monitoring-RG" -Name "MyWorkspace"

# Create software update configuration
$schedule = New-AzAutomationSchedule `
    -ResourceGroupName "Automation-RG" `
    -AutomationAccountName "MyAutomation" `
    -Name "WeeklyUpdates" `
    -StartTime (Get-Date).AddDays(1) `
    -DayOfWeek Sunday `
    -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration `
    -ResourceGroupName "Automation-RG" `
    -AutomationAccountName "MyAutomation" `
    -Schedule $schedule `
    -Windows `
    -IncludedUpdateClassification Critical, Security `
    -Duration (New-TimeSpan -Hours 3) `
    -AzureVMResourceId @($vm.Id)
```

### Disk Encryption at Scale

```powershell
# Create Key Vault for disk encryption
$keyVault = New-AzKeyVault `
    -ResourceGroupName "Security-RG" `
    -VaultName "DiskEncryptKV" `
    -Location "eastus" `
    -EnabledForDiskEncryption `
    -EnableSoftDelete `
    -EnablePurgeProtection

# Create encryption key
$key = Add-AzKeyVaultKey `
    -VaultName "DiskEncryptKV" `
    -Name "DiskEncryptionKey" `
    -Destination Software

# Enable encryption on VM
Set-AzVMDiskEncryptionExtension `
    -ResourceGroupName "VM-RG" `
    -VMName "WebServer-VM" `
    -DiskEncryptionKeyVaultUrl $keyVault.VaultUri `
    -DiskEncryptionKeyVaultId $keyVault.ResourceId `
    -KeyEncryptionKeyUrl $key.Id `
    -KeyEncryptionKeyVaultId $keyVault.ResourceId `
    -VolumeType All `
    -SkipVmBackup `
    -Force

# Verify encryption status
Get-AzVmDiskEncryptionStatus -ResourceGroupName "VM-RG" -VMName "WebServer-VM"
```

### VM Image Builder

```powershell
# Create image template (JSON definition)
$imageTemplate = @"
{
    "type": "Microsoft.VirtualMachineImages/imageTemplates",
    "location": "eastus",
    "properties": {
        "source": {
            "type": "PlatformImage",
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2022-datacenter-azure-edition",
            "version": "latest"
        },
        "customize": [
            {
                "type": "PowerShell",
                "name": "InstallIIS",
                "runElevated": true,
                "inline": [
                    "Install-WindowsFeature -Name Web-Server -IncludeManagementTools",
                    "Install-WindowsFeature -Name NET-Framework-45-ASPNET"
                ]
            }
        ],
        "distribute": [
            {
                "type": "SharedImage",
                "galleryImageId": "/subscriptions/{subId}/resourceGroups/Gallery-RG/providers/Microsoft.Compute/galleries/MyGallery/images/WebServerImage/versions/1.0.0",
                "runOutputName": "WebServerImageOutput",
                "replicationRegions": ["eastus", "westus2"]
            }
        ]
    }
}
"@
```

### VMSS Flexible Orchestration

```powershell
# Create VMSS with Flexible orchestration (modern approach)
$vmssConfig = New-AzVmssConfig `
    -Location "eastus" `
    -PlatformFaultDomainCount 3 `
    -OrchestrationMode "Flexible"

# Use existing VNet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -VirtualNetwork $vnet

# Configure network
$vmssConfig = Add-AzVmssNetworkInterfaceConfiguration `
    -VirtualMachineScaleSet $vmssConfig `
    -Name "vmssnic" `
    -Primary $true `
    -IpConfiguration @(
        @{
            Name = "ipconfig1"
            SubnetId = $subnet.Id
            LoadBalancerBackendAddressPoolsId = @($lb.BackendAddressPools[0].Id)
        }
    )

# Configure autoscale
$rule1 = New-AzAutoscaleRule `
    -MetricName "Percentage CPU" `
    -MetricResourceId $vmss.Id `
    -TimeGrain 00:01:00 `
    -MetricStatistic "Average" `
    -TimeWindow 00:05:00 `
    -Operator "GreaterThan" `
    -Threshold 75 `
    -ScaleActionDirection "Increase" `
    -ScaleActionScaleType "ChangeCount" `
    -ScaleActionValue 2 `
    -ScaleActionCooldown 00:05:00

$rule2 = New-AzAutoscaleRule `
    -MetricName "Percentage CPU" `
    -MetricResourceId $vmss.Id `
    -TimeGrain 00:01:00 `
    -MetricStatistic "Average" `
    -TimeWindow 00:05:00 `
    -Operator "LessThan" `
    -Threshold 25 `
    -ScaleActionDirection "Decrease" `
    -ScaleActionScaleType "ChangeCount" `
    -ScaleActionValue 1 `
    -ScaleActionCooldown 00:10:00

$profile = New-AzAutoscaleProfile `
    -DefaultCapacity 2 `
    -MaximumCapacity 10 `
    -MinimumCapacity 2 `
    -Rule $rule1, $rule2 `
    -Name "AutoscaleProfile"

Add-AzAutoscaleSetting `
    -ResourceGroupName "VMSS-RG" `
    -Name "AutoscaleSetting" `
    -Location "eastus" `
    -TargetResourceId $vmss.Id `
    -AutoscaleProfile $profile `
    -Notification @(
        @{
            Email = @{ SendToSubscriptionAdministrator = $true }
        }
    )
```

### VM Troubleshooting Commands

```powershell
# Serial console access
Invoke-AzVMRunCommand `
    -ResourceGroupName "VM-RG" `
    -VMName "WebServer-VM" `
    -CommandId "RunPowerShellScript" `
    -ScriptString "Get-Service | Where-Object {`$_.Status -eq 'Stopped'}"

# Boot diagnostics
Get-AzVmBootDiagnosticsData `
    -ResourceGroupName "VM-RG" `
    -Name "WebServer-VM" `
    -Windows `
    -LocalPath ".\bootdiag"

# Network watcher - VM connectivity check
Test-AzNetworkWatcherConnectivity `
    -NetworkWatcher $networkWatcher `
    -SourceId $vm.Id `
    -DestinationAddress "www.microsoft.com" `
    -DestinationPort 443

# Redeploy VM to new host (if experiencing issues)
Set-AzVM -ResourceGroupName "VM-RG" -Name "WebServer-VM" -Redeploy
```
