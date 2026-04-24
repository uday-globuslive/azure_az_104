# Module 06 - Azure Networking - Part I

## Learning Objectives
By the end of this module, you will be able to:
- Design and implement Azure Virtual Networks
- Configure VNet components and address spaces
- Manage public and private IP addresses
- Create and configure subnets
- Work with Network Interface Cards (NICs)
- Implement Network Security Groups (NSGs)
- Configure Route Tables and custom routes
- Understand and use Service Tags
- Configure Azure DNS and Private DNS

---

## 6.1 Azure Virtual Networks

### What is a Virtual Network (VNet)?
An Azure Virtual Network (VNet) is the fundamental building block for your private network in Azure. It enables Azure resources to securely communicate with each other, the internet, and on-premises networks.

### VNet Key Concepts

| Concept | Description |
|---------|-------------|
| **Address Space** | IP address range in CIDR notation |
| **Subnets** | Segments within the VNet |
| **DNS** | Name resolution for resources |
| **Peering** | Connect VNets together |
| **Service Endpoints** | Direct connectivity to Azure services |

#### 🏢 Real-World VNet Design Use Cases:

**Multi-Tier Web Application (3-Tier Architecture):**
```
┌──────────────────────────────────────────────────────────────────────┐
│  Company: RetailCorp - E-commerce Platform                           │
│  Requirement: Isolate tiers, secure database, DMZ for web            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  VNET: 10.0.0.0/16 (65,536 addresses)                               │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ DMZ Subnet: 10.0.1.0/24 (Web Tier)                          │    │
│  │ ├── Application Gateway (WAF enabled)                        │    │
│  │ ├── NSG: Allow 80/443 from Internet, deny all other inbound │    │
│  │ └── 4x Web Servers (VMSS auto-scale 2-20)                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                           │                                          │
│                           ▼                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ App Subnet: 10.0.2.0/24 (Application Tier)                  │    │
│  │ ├── API servers (internal load balancer)                    │    │
│  │ ├── NSG: Allow from DMZ subnet only, deny Internet          │    │
│  │ └── Service Endpoint to Azure SQL                           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                           │                                          │
│                           ▼                                          │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │ Data Subnet: 10.0.3.0/24 (Database Tier)                    │    │
│  │ ├── Azure SQL Private Endpoint (no public access)           │    │
│  │ ├── NSG: Allow from App subnet on port 1433 only            │    │
│  │ └── Redis Cache Private Endpoint                            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  Security Result: Database has ZERO internet exposure               │
└──────────────────────────────────────────────────────────────────────┘
```

**Enterprise Hub-Spoke Model:**
```
Company: GlobalFinance (Bank with strict compliance)
Requirement: Centralized security, distributed workloads

Architecture:
                    ┌─────────────────────────┐
                    │    Hub VNet (Shared)    │
                    │    10.0.0.0/16          │
                    ├─────────────────────────┤
                    │ • Azure Firewall        │
                    │ • VPN Gateway           │
                    │ • ExpressRoute Gateway  │
                    │ • Bastion Host          │
                    │ • Log Analytics         │
                    └───────────┬─────────────┘
              ┌─────────────────┼─────────────────┐
              │                 │                 │
    ┌─────────▼─────┐  ┌───────▼───────┐  ┌─────▼─────────┐
    │ Spoke: Prod   │  │ Spoke: Dev    │  │ Spoke: DMZ    │
    │ 10.1.0.0/16   │  │ 10.2.0.0/16   │  │ 10.3.0.0/16   │
    ├───────────────┤  ├───────────────┤  ├───────────────┤
    │ Trading App   │  │ Dev/Test VMs  │  │ Public APIs   │
    │ Core Banking  │  │ CI/CD Agents  │  │ Web Apps      │
    │ (No Internet) │  │ (Limited Net) │  │ (Public LB)   │
    └───────────────┘  └───────────────┘  └───────────────┘

Benefits:
- All traffic inspected by central Firewall
- Single point for VPN/ExpressRoute
- Cost: Share expensive gateways across all spokes
- Compliance: Audit all traffic in one place
```

**Microservices on AKS with Network Isolation:**
```
Company: StreamVideo (Video streaming service)
Requirement: Kubernetes with private networking

VNet Design:
├── AKS Subnet: 10.0.0.0/16 (for pods - need many IPs!)
│   └── CNI networking: Each pod gets VNet IP
│   └── 10,000+ pods possible
│
├── AKS Services Subnet: 10.1.0.0/24
│   └── Internal load balancers
│
├── Private Endpoints Subnet: 10.2.0.0/24
│   └── Cosmos DB, Storage, Key Vault
│
└── Ingress Subnet: 10.3.0.0/24
    └── Application Gateway Ingress Controller

Result: AKS pods directly access Azure PaaS services
        via private IPs (never traverse internet)
```

### VNet Architecture

```
┌─────────────────────────── Virtual Network ───────────────────────────┐
│                        Address Space: 10.0.0.0/16                      │
│                                                                        │
│  ┌─────────────────────┐  ┌─────────────────────┐                     │
│  │   Frontend Subnet   │  │   Backend Subnet    │                     │
│  │   10.0.1.0/24       │  │   10.0.2.0/24       │                     │
│  │                     │  │                     │                     │
│  │  ┌─────┐  ┌─────┐   │  │  ┌─────┐  ┌─────┐   │                     │
│  │  │ VM1 │  │ VM2 │   │  │  │ VM3 │  │ VM4 │   │                     │
│  │  └─────┘  └─────┘   │  │  └─────┘  └─────┘   │                     │
│  └─────────────────────┘  └─────────────────────┘                     │
│                                                                        │
│  ┌─────────────────────┐  ┌─────────────────────┐                     │
│  │   Database Subnet   │  │   Gateway Subnet    │                     │
│  │   10.0.3.0/24       │  │   10.0.255.0/27     │                     │
│  │                     │  │                     │                     │
│  │  ┌─────┐            │  │  ┌───────────────┐  │                     │
│  │  │ SQL │            │  │  │ VPN Gateway   │  │                     │
│  │  └─────┘            │  │  └───────────────┘  │                     │
│  └─────────────────────┘  └─────────────────────┘                     │
└────────────────────────────────────────────────────────────────────────┘
```

### Creating a Virtual Network

#### Using Azure Portal
1. Navigate to "Virtual Networks"
2. Click "Create"
3. Configure:
   - Name, Resource Group, Location
   - Address space (e.g., 10.0.0.0/16)
   - Subnets
   - Security (DDoS, Firewall)

#### Using PowerShell

```powershell
# Create resource group
New-AzResourceGroup -Name "Network-RG" -Location "East US"

# Create virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "Network-RG" `
  -Name "MyVNet" `
  -Location "East US" `
  -AddressPrefix "10.0.0.0/16"

# Add subnets
$frontendSubnet = Add-AzVirtualNetworkSubnetConfig `
  -Name "FrontendSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.1.0/24"

$backendSubnet = Add-AzVirtualNetworkSubnetConfig `
  -Name "BackendSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.2.0/24"

# Apply changes
$vnet | Set-AzVirtualNetwork
```

#### Using Azure CLI

```bash
# Create VNet with subnet
az network vnet create \
  --resource-group "Network-RG" \
  --name "MyVNet" \
  --address-prefix "10.0.0.0/16" \
  --subnet-name "FrontendSubnet" \
  --subnet-prefixes "10.0.1.0/24"

# Add additional subnet
az network vnet subnet create \
  --resource-group "Network-RG" \
  --vnet-name "MyVNet" \
  --name "BackendSubnet" \
  --address-prefixes "10.0.2.0/24"
```

### Address Space Planning

| Network Size | CIDR | Hosts | Use Case |
|--------------|------|-------|----------|
| /16 | 10.0.0.0/16 | 65,534 | Large enterprise |
| /20 | 10.0.0.0/20 | 4,094 | Medium deployment |
| /24 | 10.0.0.0/24 | 254 | Small subnet |
| /27 | 10.0.0.0/27 | 30 | Gateway subnet |

### Reserved IP Addresses

Azure reserves 5 IP addresses in each subnet:

| Address | Purpose |
|---------|---------|
| x.x.x.0 | Network address |
| x.x.x.1 | Default gateway |
| x.x.x.2, x.x.x.3 | Azure DNS mapping |
| x.x.x.255 | Broadcast address |

**Example**: For subnet 10.0.0.0/24:
- Usable IPs: 10.0.0.4 - 10.0.0.254 (251 addresses)

---

## 6.2 Azure VNet Components

### VNet Peering

Connect VNets for direct traffic flow:

```
┌──────────────────┐     VNet Peering     ┌──────────────────┐
│    VNet-A        │◄───────────────────►│    VNet-B        │
│  10.0.0.0/16     │                      │  10.1.0.0/16     │
│                  │                      │                  │
│  ┌────────────┐  │                      │  ┌────────────┐  │
│  │  Subnet A  │  │                      │  │  Subnet B  │  │
│  │  10.0.1.0  │  │                      │  │  10.1.1.0  │  │
│  └────────────┘  │                      │  └────────────┘  │
└──────────────────┘                      └──────────────────┘
```

```powershell
# Create VNet peering
Add-AzVirtualNetworkPeering `
  -Name "VNetA-to-VNetB" `
  -VirtualNetwork $vnetA `
  -RemoteVirtualNetworkId $vnetB.Id `
  -AllowForwardedTraffic `
  -AllowGatewayTransit

# Create reverse peering
Add-AzVirtualNetworkPeering `
  -Name "VNetB-to-VNetA" `
  -VirtualNetwork $vnetB `
  -RemoteVirtualNetworkId $vnetA.Id `
  -AllowForwardedTraffic `
  -UseRemoteGateways
```

### VNet-to-VNet VPN

For cross-region or encrypted connections:

```bash
# Create VPN gateway
az network vnet-gateway create \
  --resource-group "Network-RG" \
  --name "VNetAGateway" \
  --vnet "VNetA" \
  --gateway-type "Vpn" \
  --vpn-type "RouteBased" \
  --sku "VpnGw1" \
  --public-ip-address "VNetAGatewayIP"
```

### Service Endpoints

Direct, optimized connectivity to Azure services:

```powershell
# Enable service endpoint for storage
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "BackendSubnet"

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name "BackendSubnet" `
  -AddressPrefix $subnet.AddressPrefix `
  -ServiceEndpoint "Microsoft.Storage","Microsoft.Sql"

$vnet | Set-AzVirtualNetwork
```

### Private Endpoints

Private IP access to Azure services:

```
┌───────────────────────────────────────────────────────────────┐
│                        Virtual Network                         │
│                                                                │
│  ┌─────────────────────┐      ┌─────────────────────────┐    │
│  │    Application      │      │   Private Endpoint       │    │
│  │    Subnet           │─────►│   10.0.2.4               │    │
│  │                     │      │   (Storage Account)      │    │
│  └─────────────────────┘      └─────────────────────────┘    │
│                                         │                     │
└─────────────────────────────────────────│─────────────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │  Azure Storage Account │
                              │  (No public access)    │
                              └───────────────────────┘
```

---

## 6.3 IP Address – Public & Private IPs

### Private IP Addresses

| Allocation | Description |
|------------|-------------|
| **Dynamic** | Assigned by Azure DHCP, may change on restart |
| **Static** | Fixed IP, remains constant |

```powershell
# Create NIC with static private IP
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "FrontendSubnet"

$nic = New-AzNetworkInterface `
  -ResourceGroupName "Network-RG" `
  -Name "StaticNIC" `
  -Location "East US" `
  -SubnetId $subnet.Id `
  -PrivateIpAddress "10.0.1.10"
```

### Public IP Addresses

| SKU | Features | Use Case |
|-----|----------|----------|
| **Basic** | Dynamic or static, open by default | Testing, dev |
| **Standard** | Static only, zone-redundant, secure by default | Production |

```powershell
# Create Standard public IP
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "Network-RG" `
  -Name "MyPublicIP" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static" `
  -IpAddressVersion "IPv4" `
  -Zone 1,2,3
```

```bash
# Azure CLI
az network public-ip create \
  --resource-group "Network-RG" \
  --name "MyPublicIP" \
  --sku "Standard" \
  --allocation-method "Static" \
  --zone 1 2 3
```

### IP Address Types Comparison

| Feature | Basic Public IP | Standard Public IP |
|---------|-----------------|-------------------|
| Allocation | Dynamic or Static | Static only |
| Security | Open by default | Closed by default |
| Availability Zones | Not supported | Zone-redundant |
| Load Balancer | Basic LB only | Standard LB only |
| SLA | None | 99.99% |

### Public IP Prefix

Reserve a contiguous range of public IPs:

```powershell
# Create public IP prefix
$prefix = New-AzPublicIpPrefix `
  -ResourceGroupName "Network-RG" `
  -Name "MyIPPrefix" `
  -Location "East US" `
  -PrefixLength 28 `
  -Sku "Standard"

# Create public IP from prefix
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "Network-RG" `
  -Name "IP-From-Prefix" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static" `
  -PublicIpPrefix $prefix
```

---

## 6.4 Azure VNet Subnets

### Subnet Planning

| Subnet Type | Purpose | Typical Size |
|-------------|---------|--------------|
| Web tier | Frontend servers | /24 or /25 |
| App tier | Application servers | /24 or /25 |
| Database tier | Database servers | /24 or /26 |
| Gateway | VPN/ExpressRoute | /27 |
| Bastion | Azure Bastion | /26 or larger |
| AKS | Kubernetes nodes | /21 or larger |

### Special Subnets

| Subnet Name | Purpose | Requirements |
|-------------|---------|--------------|
| GatewaySubnet | VPN/ExpressRoute Gateway | Exactly "GatewaySubnet" |
| AzureBastionSubnet | Azure Bastion | At least /26 |
| AzureFirewallSubnet | Azure Firewall | At least /26 |

### Creating Subnets

```powershell
# Add multiple subnets
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"

# Web subnet with NSG
Add-AzVirtualNetworkSubnetConfig `
  -Name "WebSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.1.0/24" `
  -NetworkSecurityGroupId $webNSG.Id

# Database subnet with service endpoint
Add-AzVirtualNetworkSubnetConfig `
  -Name "DatabaseSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.3.0/24" `
  -ServiceEndpoint "Microsoft.Sql"

# Gateway subnet
Add-AzVirtualNetworkSubnetConfig `
  -Name "GatewaySubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.255.0/27"

$vnet | Set-AzVirtualNetwork
```

### Subnet Delegation

Delegate subnets to specific Azure services:

```powershell
# Delegate subnet to App Service
$delegation = New-AzDelegation `
  -Name "AppServiceDelegation" `
  -ServiceName "Microsoft.Web/serverFarms"

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name "AppServiceSubnet" `
  -AddressPrefix "10.0.4.0/24" `
  -Delegation $delegation

$vnet | Set-AzVirtualNetwork
```

---

## 6.5 Azure Network Interface Cards (NIC)

### NIC Components

```
┌─────────────────────────────────────────────────────────────┐
│                   Network Interface                          │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                IP Configurations                     │    │
│  │  ┌─────────────────┐  ┌─────────────────┐           │    │
│  │  │ Primary IP      │  │ Secondary IP    │           │    │
│  │  │ 10.0.1.4        │  │ 10.0.1.5        │           │    │
│  │  │ + Public IP     │  │                 │           │    │
│  │  └─────────────────┘  └─────────────────┘           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Properties:                                                 │
│  - Subnet: 10.0.1.0/24                                      │
│  - NSG: WebNSG                                              │
│  - Accelerated Networking: Enabled                          │
│  - IP Forwarding: Disabled                                  │
└─────────────────────────────────────────────────────────────┘
```

### Creating a NIC

```powershell
# Create NIC with multiple IP configurations
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "WebSubnet"
$publicIP = Get-AzPublicIpAddress -ResourceGroupName "Network-RG" -Name "WebPublicIP"

# Primary IP configuration
$ipConfig1 = New-AzNetworkInterfaceIpConfig `
  -Name "ipconfig1" `
  -Primary `
  -SubnetId $subnet.Id `
  -PrivateIpAddress "10.0.1.10" `
  -PublicIpAddressId $publicIP.Id

# Secondary IP configuration
$ipConfig2 = New-AzNetworkInterfaceIpConfig `
  -Name "ipconfig2" `
  -SubnetId $subnet.Id `
  -PrivateIpAddress "10.0.1.11"

# Create NIC
$nic = New-AzNetworkInterface `
  -ResourceGroupName "Network-RG" `
  -Name "WebNIC" `
  -Location "East US" `
  -IpConfiguration $ipConfig1,$ipConfig2 `
  -EnableAcceleratedNetworking
```

```bash
# Azure CLI
az network nic create \
  --resource-group "Network-RG" \
  --name "WebNIC" \
  --vnet-name "MyVNet" \
  --subnet "WebSubnet" \
  --private-ip-address "10.0.1.10" \
  --public-ip-address "WebPublicIP" \
  --accelerated-networking true
```

### Attaching NIC to VM

```powershell
# Add NIC to existing VM (VM must be stopped)
$vm = Get-AzVM -ResourceGroupName "Network-RG" -Name "MyVM"
$nic = Get-AzNetworkInterface -ResourceGroupName "Network-RG" -Name "SecondaryNIC"

Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
Update-AzVM -ResourceGroupName "Network-RG" -VM $vm
```

### NIC Configuration Options

| Option | Description |
|--------|-------------|
| **IP Forwarding** | Forward traffic to other IPs (for NVAs) |
| **Accelerated Networking** | SR-IOV for better performance |
| **DNS Servers** | Custom DNS settings |

---

## 6.6 Network Security Group (NSG)

### What is an NSG?
A Network Security Group contains security rules that allow or deny inbound and outbound network traffic.

### NSG Rule Properties

| Property | Description |
|----------|-------------|
| **Name** | Unique rule name |
| **Priority** | 100-4096 (lower = higher priority) |
| **Source/Destination** | IP, CIDR, Service Tag, ASG |
| **Protocol** | TCP, UDP, ICMP, Any |
| **Port Range** | Single port, range, or * |
| **Direction** | Inbound or Outbound |
| **Action** | Allow or Deny |

### Default Rules

| Priority | Name | Direction | Source | Destination | Action |
|----------|------|-----------|--------|-------------|--------|
| 65000 | AllowVnetInBound | Inbound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowAzureLoadBalancerInBound | Inbound | AzureLoadBalancer | * | Allow |
| 65500 | DenyAllInBound | Inbound | * | * | Deny |
| 65000 | AllowVnetOutBound | Outbound | VirtualNetwork | VirtualNetwork | Allow |
| 65001 | AllowInternetOutBound | Outbound | * | Internet | Allow |
| 65500 | DenyAllOutBound | Outbound | * | * | Deny |

### Creating NSG

```powershell
# Create NSG
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "Network-RG" `
  -Name "WebNSG" `
  -Location "East US"

# Add rule to allow HTTP
$nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "Allow-HTTP" `
  -Description "Allow HTTP traffic" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80

# Add rule to allow HTTPS
$nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "Allow-HTTPS" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 110 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 443

# Add rule to deny all other inbound
$nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "Deny-All-Inbound" `
  -Access Deny `
  -Protocol * `
  -Direction Inbound `
  -Priority 4000 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange *

# Apply changes
$nsg | Set-AzNetworkSecurityGroup
```

```bash
# Azure CLI
az network nsg create \
  --resource-group "Network-RG" \
  --name "WebNSG"

az network nsg rule create \
  --resource-group "Network-RG" \
  --nsg-name "WebNSG" \
  --name "Allow-HTTP" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --destination-port-ranges 80
```

### Associating NSG

```powershell
# Associate with subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "WebSubnet"

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name "WebSubnet" `
  -AddressPrefix $subnet.AddressPrefix `
  -NetworkSecurityGroupId $nsg.Id

$vnet | Set-AzVirtualNetwork

# Associate with NIC
$nic = Get-AzNetworkInterface -ResourceGroupName "Network-RG" -Name "WebNIC"
$nic.NetworkSecurityGroup = $nsg
$nic | Set-AzNetworkInterface
```

### NSG Flow Evaluation

```
Inbound Traffic Flow:
┌─────────┐     ┌─────────────┐     ┌────────────┐     ┌────────┐
│ Internet│────►│ Subnet NSG  │────►│  NIC NSG   │────►│  VM    │
└─────────┘     │ (if exists) │     │ (if exists)│     └────────┘
                └─────────────┘     └────────────┘
                
Outbound Traffic Flow:
┌────────┐     ┌────────────┐     ┌─────────────┐     ┌──────────┐
│  VM    │────►│  NIC NSG   │────►│ Subnet NSG  │────►│ Internet │
└────────┘     │ (if exists)│     │ (if exists) │     └──────────┘
               └────────────┘     └─────────────┘
```

---

## 6.7 Route Tables

### What are Route Tables?
Route Tables contain routes that direct network traffic. Azure automatically creates system routes, but you can create custom routes.

### System Routes

| Destination | Next Hop |
|-------------|----------|
| VNet address space | Virtual network |
| 0.0.0.0/0 | Internet |
| 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 | None (dropped) |

### Custom Route Types

| Next Hop Type | Description |
|---------------|-------------|
| **Virtual network** | Routes within VNet |
| **Internet** | Routes to internet |
| **Virtual network gateway** | Routes through VPN/ExpressRoute |
| **Virtual appliance** | Routes through NVA (firewall) |
| **None** | Drop traffic |

### Creating Route Table

```powershell
# Create route table
$routeTable = New-AzRouteTable `
  -ResourceGroupName "Network-RG" `
  -Name "BackendRouteTable" `
  -Location "East US"

# Add route to send internet traffic through firewall
Add-AzRouteConfig `
  -RouteTable $routeTable `
  -Name "ToFirewall" `
  -AddressPrefix "0.0.0.0/0" `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress "10.0.100.4"

# Add route to specific subnet
Add-AzRouteConfig `
  -RouteTable $routeTable `
  -Name "ToDatabase" `
  -AddressPrefix "10.0.3.0/24" `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress "10.0.100.4"

# Apply changes
Set-AzRouteTable -RouteTable $routeTable
```

```bash
# Azure CLI
az network route-table create \
  --resource-group "Network-RG" \
  --name "BackendRouteTable"

az network route-table route create \
  --resource-group "Network-RG" \
  --route-table-name "BackendRouteTable" \
  --name "ToFirewall" \
  --address-prefix "0.0.0.0/0" \
  --next-hop-type "VirtualAppliance" \
  --next-hop-ip-address "10.0.100.4"
```

### Associating Route Table with Subnet

```powershell
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$routeTable = Get-AzRouteTable -ResourceGroupName "Network-RG" -Name "BackendRouteTable"

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name "BackendSubnet" `
  -AddressPrefix "10.0.2.0/24" `
  -RouteTableId $routeTable.Id

$vnet | Set-AzVirtualNetwork
```

### Hub-Spoke Network with Route Tables

```
                    ┌───────────────────────────────────────┐
                    │           Hub VNet                     │
                    │  ┌─────────────────────────────────┐  │
                    │  │      Azure Firewall              │  │
                    │  │      10.0.100.4                  │  │
                    │  └─────────────────────────────────┘  │
                    │          ▲         ▲                   │
                    └──────────│─────────│───────────────────┘
                               │         │
          ┌────────────────────┘         └────────────────────┐
          │                                                    │
┌─────────┴─────────┐                          ┌─────────────┴───────┐
│   Spoke VNet 1    │                          │   Spoke VNet 2      │
│                   │                          │                     │
│  UDR: 0.0.0.0/0   │                          │  UDR: 0.0.0.0/0     │
│  → 10.0.100.4     │                          │  → 10.0.100.4       │
└───────────────────┘                          └─────────────────────┘
```

---

## 6.8 Service Tags

### What are Service Tags?
Service Tags represent groups of IP address prefixes from Azure services. They simplify NSG rule creation.

### Common Service Tags

| Tag | Description |
|-----|-------------|
| **Internet** | All public Internet IPs |
| **VirtualNetwork** | VNet address space + peered VNets |
| **AzureLoadBalancer** | Azure infrastructure LB |
| **Storage** | Azure Storage IPs |
| **Sql** | Azure SQL Database IPs |
| **AzureCloud** | All public Azure IPs |
| **AzureMonitor** | Azure monitoring services |
| **AzureActiveDirectory** | Azure AD IPs |
| **GatewayManager** | Azure Gateway Manager |

### Regional Service Tags

```
Storage.EastUS       - Storage in East US only
Sql.WestEurope       - SQL in West Europe only
AzureCloud.CentralUS - Azure cloud in Central US
```

### Using Service Tags in NSG

```powershell
# Allow outbound to Azure Storage
$nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "Allow-Storage-Outbound" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Outbound `
  -Priority 100 `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange * `
  -DestinationAddressPrefix Storage `
  -DestinationPortRange 443

# Allow outbound to Azure SQL
$nsg | Add-AzNetworkSecurityRuleConfig `
  -Name "Allow-SQL-Outbound" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Outbound `
  -Priority 110 `
  -SourceAddressPrefix VirtualNetwork `
  -SourcePortRange * `
  -DestinationAddressPrefix Sql `
  -DestinationPortRange 1433

$nsg | Set-AzNetworkSecurityGroup
```

```bash
# Azure CLI with service tags
az network nsg rule create \
  --resource-group "Network-RG" \
  --nsg-name "AppNSG" \
  --name "Allow-Storage" \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --destination-address-prefixes Storage \
  --destination-port-ranges 443
```

---

## 6.9 Azure DNS

### What is Azure DNS?
Azure DNS provides hosting for DNS domains, allowing you to manage DNS records using Azure infrastructure.

### DNS Zone Types

| Type | Description |
|------|-------------|
| **Public DNS Zone** | Resolves names on the Internet |
| **Private DNS Zone** | Resolves names within VNets |

### DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | IPv4 address | `www → 203.0.113.1` |
| **AAAA** | IPv6 address | `www → 2001:db8::1` |
| **CNAME** | Canonical name (alias) | `www → myapp.azurewebsites.net` |
| **MX** | Mail exchange | `→ mail.contoso.com (priority 10)` |
| **TXT** | Text records | Verification, SPF |
| **NS** | Name servers | Azure NS records |
| **SOA** | Start of authority | Zone metadata |

### Creating Public DNS Zone

```powershell
# Create DNS zone
New-AzDnsZone `
  -ResourceGroupName "DNS-RG" `
  -Name "contoso.com"

# Add A record
New-AzDnsRecordSet `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "contoso.com" `
  -Name "www" `
  -RecordType A `
  -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -IPv4Address "203.0.113.1")

# Add CNAME record
New-AzDnsRecordSet `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "contoso.com" `
  -Name "api" `
  -RecordType CNAME `
  -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -Cname "myapi.azurewebsites.net")

# Add MX record
New-AzDnsRecordSet `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "contoso.com" `
  -Name "@" `
  -RecordType MX `
  -Ttl 3600 `
  -DnsRecords (New-AzDnsRecordConfig -Exchange "mail.contoso.com" -Preference 10)
```

```bash
# Azure CLI
az network dns zone create \
  --resource-group "DNS-RG" \
  --name "contoso.com"

az network dns record-set a add-record \
  --resource-group "DNS-RG" \
  --zone-name "contoso.com" \
  --record-set-name "www" \
  --ipv4-address "203.0.113.1"
```

### Delegating Domain to Azure DNS

1. Get Azure DNS name servers:
```powershell
(Get-AzDnsZone -ResourceGroupName "DNS-RG" -Name "contoso.com").NameServers
```

2. Update NS records at your domain registrar to point to Azure DNS servers:
   - ns1-01.azure-dns.com
   - ns2-01.azure-dns.net
   - ns3-01.azure-dns.org
   - ns4-01.azure-dns.info

---

## 6.10 Private DNS

### What is Private DNS?
Azure Private DNS provides DNS resolution within virtual networks without needing a custom DNS solution.

### Private DNS Features

| Feature | Description |
|---------|-------------|
| **Automatic VM registration** | VMs auto-register DNS names |
| **Split-horizon DNS** | Same name, different resolutions |
| **VNet links** | Connect zones to multiple VNets |

### Creating Private DNS Zone

```powershell
# Create private DNS zone
$privateDnsZone = New-AzPrivateDnsZone `
  -ResourceGroupName "DNS-RG" `
  -Name "internal.contoso.com"

# Link to VNet with auto-registration
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"

New-AzPrivateDnsVirtualNetworkLink `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "internal.contoso.com" `
  -Name "MyVNetLink" `
  -VirtualNetworkId $vnet.Id `
  -EnableRegistration
```

```bash
# Azure CLI
az network private-dns zone create \
  --resource-group "DNS-RG" \
  --name "internal.contoso.com"

az network private-dns link vnet create \
  --resource-group "DNS-RG" \
  --zone-name "internal.contoso.com" \
  --name "MyVNetLink" \
  --virtual-network "MyVNet" \
  --registration-enabled true
```

### Private DNS Records

```powershell
# Add A record
New-AzPrivateDnsRecordSet `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "internal.contoso.com" `
  -Name "db" `
  -RecordType A `
  -Ttl 3600 `
  -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address "10.0.3.4")

# Add CNAME record
New-AzPrivateDnsRecordSet `
  -ResourceGroupName "DNS-RG" `
  -ZoneName "internal.contoso.com" `
  -Name "database" `
  -RecordType CNAME `
  -Ttl 3600 `
  -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -Cname "db.internal.contoso.com")
```

### Private DNS Zone for Azure Services

Link private DNS zones for Azure private endpoints:

| Service | Zone Name |
|---------|-----------|
| Azure SQL | privatelink.database.windows.net |
| Azure Storage (blob) | privatelink.blob.core.windows.net |
| Azure Key Vault | privatelink.vaultcore.azure.net |
| Azure Web Apps | privatelink.azurewebsites.net |

---

## Hands-on Exercises

### Exercise 1: VNet Creation

```powershell
# Create complete VNet with multiple subnets
$resourceGroup = "Lab-Network-RG"
$location = "East US"

New-AzResourceGroup -Name $resourceGroup -Location $location

$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $resourceGroup `
  -Name "LabVNet" `
  -Location $location `
  -AddressPrefix "10.10.0.0/16"

# Add subnets
@{
    "WebSubnet" = "10.10.1.0/24"
    "AppSubnet" = "10.10.2.0/24"
    "DBSubnet" = "10.10.3.0/24"
    "GatewaySubnet" = "10.10.255.0/27"
}.GetEnumerator() | ForEach-Object {
    Add-AzVirtualNetworkSubnetConfig `
      -Name $_.Key `
      -VirtualNetwork $vnet `
      -AddressPrefix $_.Value
}

$vnet | Set-AzVirtualNetwork
```

### Exercise 2: Create and Configure VNet Peering

```powershell
# Create two VNets
$vnet1 = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Name "VNet1" -Location $location -AddressPrefix "10.1.0.0/16"
$vnet2 = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Name "VNet2" -Location $location -AddressPrefix "10.2.0.0/16"

# Create peering
Add-AzVirtualNetworkPeering -Name "VNet1-to-VNet2" -VirtualNetwork $vnet1 -RemoteVirtualNetworkId $vnet2.Id
Add-AzVirtualNetworkPeering -Name "VNet2-to-VNet1" -VirtualNetwork $vnet2 -RemoteVirtualNetworkId $vnet1.Id

# Verify peering
Get-AzVirtualNetworkPeering -ResourceGroupName $resourceGroup -VirtualNetworkName "VNet1"
```

### Exercise 3-14: Complete Networking Lab

```powershell
# Exercise 3: Verify VNet connectivity
Test-NetConnection -ComputerName "10.2.0.4" -Port 22

# Exercise 4: Assign static IP
$nic = Get-AzNetworkInterface -Name "MyNIC" -ResourceGroupName $resourceGroup
$nic.IpConfigurations[0].PrivateIpAllocationMethod = "Static"
$nic.IpConfigurations[0].PrivateIpAddress = "10.10.1.100"
$nic | Set-AzNetworkInterface

# Exercise 5: Create route table
$routeTable = New-AzRouteTable -ResourceGroupName $resourceGroup -Name "CustomRouteTable" -Location $location
Add-AzRouteConfig -RouteTable $routeTable -Name "ToInternet" -AddressPrefix "0.0.0.0/0" -NextHopType "VirtualAppliance" -NextHopIpAddress "10.10.100.4"
Set-AzRouteTable -RouteTable $routeTable

# Exercise 6: Add routes
Add-AzRouteConfig -RouteTable $routeTable -Name "ToOnPrem" -AddressPrefix "192.168.0.0/16" -NextHopType "VirtualNetworkGateway"
Set-AzRouteTable -RouteTable $routeTable

# Exercise 7: Create NIC
$subnet = Get-AzVirtualNetwork -Name "LabVNet" -ResourceGroupName $resourceGroup | Get-AzVirtualNetworkSubnetConfig -Name "WebSubnet"
$nic = New-AzNetworkInterface -Name "WebNIC" -ResourceGroupName $resourceGroup -Location $location -SubnetId $subnet.Id

# Exercise 8: Attach NIC to VM
$vm = Get-AzVM -Name "WebServer" -ResourceGroupName $resourceGroup
Add-AzVMNetworkInterface -VM $vm -Id $nic.Id
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup

# Exercise 9-10: Create DNS zone and RecordSet
$zone = New-AzDnsZone -ResourceGroupName $resourceGroup -Name "lab.contoso.com"
New-AzDnsRecordSet -ZoneName "lab.contoso.com" -ResourceGroupName $resourceGroup -Name "www" -RecordType A -Ttl 300 -DnsRecords (New-AzDnsRecordConfig -IPv4Address "10.10.1.100")

# Exercise 11-14: Create and configure NSG
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Name "WebNSG" -Location $location

$nsg | Add-AzNetworkSecurityRuleConfig -Name "Allow-HTTP" -Protocol Tcp -Direction Inbound -Priority 100 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 80 -Access Allow
$nsg | Add-AzNetworkSecurityRuleConfig -Name "Allow-HTTPS" -Protocol Tcp -Direction Inbound -Priority 110 -SourceAddressPrefix Internet -SourcePortRange * -DestinationAddressPrefix * -DestinationPortRange 443 -Access Allow
$nsg | Set-AzNetworkSecurityGroup

# Attach NSG to subnet
$vnet = Get-AzVirtualNetwork -Name "LabVNet" -ResourceGroupName $resourceGroup
Set-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "WebSubnet" -AddressPrefix "10.10.1.0/24" -NetworkSecurityGroupId $nsg.Id
$vnet | Set-AzVirtualNetwork

# Verify NSG
Get-AzEffectiveNetworkSecurityGroup -NetworkInterfaceName "WebNIC" -ResourceGroupName $resourceGroup
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Virtual Networks | Address spaces, regions, isolation |
| VNet Components | Peering, service endpoints, private endpoints |
| IP Addresses | Static/Dynamic, Basic/Standard SKU |
| Subnets | Address planning, reserved IPs, delegation |
| NICs | IP configurations, accelerated networking |
| NSGs | Security rules, priorities, flow evaluation |
| Route Tables | Custom routes, UDRs, hub-spoke routing |
| Service Tags | Azure service IP groups for NSGs |
| Azure DNS | Public/Private zones, record types |

---

## Review Questions

1. How many IP addresses does Azure reserve in each subnet?
2. What is the difference between NSG association at subnet vs NIC level?
3. Explain the purpose of Service Tags in NSG rules.
4. What is the difference between public and private DNS zones?
5. When would you use User-Defined Routes (UDRs)?
6. What are the requirements for the GatewaySubnet?

---

## Additional Resources

- [Azure Virtual Network Documentation](https://docs.microsoft.com/azure/virtual-network/)
- [Network Security Groups](https://docs.microsoft.com/azure/virtual-network/network-security-groups-overview)
- [Azure DNS Documentation](https://docs.microsoft.com/azure/dns/)
- [Virtual Network Peering](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)

---

## 6.6 Advanced Networking Patterns (Expert Level)

### Hub and Spoke Network Topology

```
                    ┌─────────────────────────────────┐
                    │          Hub VNet               │
                    │        10.0.0.0/16              │
                    │                                 │
                    │  ┌─────────────┐ ┌───────────┐ │
                    │  │   Firewall  │ │VPN Gateway│ │
                    │  │  10.0.0.0/26│ │10.0.1.0/27│ │
                    │  └─────────────┘ └───────────┘ │
                    │           │VNet Peering        │
          ┌─────────┴───────────┼────────────────────┴───────────┐
          │                     │                                │
          ▼                     ▼                                ▼
┌─────────────────┐   ┌─────────────────┐           ┌─────────────────┐
│  Spoke VNet 1   │   │  Spoke VNet 2   │           │  Spoke VNet 3   │
│  10.1.0.0/16    │   │  10.2.0.0/16    │           │  10.3.0.0/16    │
│                 │   │                 │           │                 │
│┌───────────────┐│   │┌───────────────┐│           │┌───────────────┐│
││Production     ││   ││Development    ││           ││Shared Services││
││Workloads      ││   ││Workloads      ││           ││               ││
│└───────────────┘│   │└───────────────┘│           │└───────────────┘│
└─────────────────┘   └─────────────────┘           └─────────────────┘
```

```powershell
# Create Hub VNet
$hubVnet = New-AzVirtualNetwork `
    -ResourceGroupName "Network-RG" `
    -Name "Hub-VNet" `
    -Location "eastus" `
    -AddressPrefix "10.0.0.0/16"

# Add subnets to Hub
$firewallSubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "AzureFirewallSubnet" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.0.0/26"

$gatewaySubnet = Add-AzVirtualNetworkSubnetConfig `
    -Name "GatewaySubnet" `
    -VirtualNetwork $hubVnet `
    -AddressPrefix "10.0.1.0/27"

$hubVnet | Set-AzVirtualNetwork

# Create Spoke VNets
$spoke1Vnet = New-AzVirtualNetwork `
    -ResourceGroupName "Network-RG" `
    -Name "Spoke1-VNet" `
    -Location "eastus" `
    -AddressPrefix "10.1.0.0/16"

# Create peering from Hub to Spoke (with gateway transit)
Add-AzVirtualNetworkPeering `
    -Name "Hub-to-Spoke1" `
    -VirtualNetwork $hubVnet `
    -RemoteVirtualNetworkId $spoke1Vnet.Id `
    -AllowGatewayTransit `
    -AllowForwardedTraffic

# Create peering from Spoke to Hub (use remote gateway)
Add-AzVirtualNetworkPeering `
    -Name "Spoke1-to-Hub" `
    -VirtualNetwork $spoke1Vnet `
    -RemoteVirtualNetworkId $hubVnet.Id `
    -UseRemoteGateways `
    -AllowForwardedTraffic
```

### Private Link and Private Endpoints

```powershell
# Create Private Endpoint for Storage Account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount"

$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
    -Name "storage-pe-connection" `
    -PrivateLinkServiceId $storageAccount.Id `
    -GroupId "blob"

$subnet = Get-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-RG") `
    -Name "PrivateEndpoints"

$privateEndpoint = New-AzPrivateEndpoint `
    -ResourceGroupName "Network-RG" `
    -Name "storage-pe" `
    -Location "eastus" `
    -Subnet $subnet `
    -PrivateLinkServiceConnection $privateEndpointConnection

# Create Private DNS Zone for blob storage
$privateDnsZone = New-AzPrivateDnsZone `
    -ResourceGroupName "Network-RG" `
    -Name "privatelink.blob.core.windows.net"

# Link DNS zone to VNet
New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName "Network-RG" `
    -ZoneName "privatelink.blob.core.windows.net" `
    -Name "hub-vnet-link" `
    -VirtualNetworkId $hubVnet.Id `
    -EnableRegistration $false

# Create DNS record
$dnsConfig = New-AzPrivateDnsZoneConfig `
    -Name "blob" `
    -PrivateDnsZoneId $privateDnsZone.ResourceId

New-AzPrivateDnsZoneGroup `
    -ResourceGroupName "Network-RG" `
    -PrivateEndpointName "storage-pe" `
    -Name "blob-zone-group" `
    -PrivateDnsZoneConfig $dnsConfig
```

### User Defined Routes (UDR) for Forced Tunneling

```powershell
# Create route table for forcing traffic through firewall
$routeTable = New-AzRouteTable `
    -ResourceGroupName "Network-RG" `
    -Name "Spoke1-RouteTable" `
    -Location "eastus" `
    -DisableBgpRoutePropagation

# Add route to send all traffic to firewall
Add-AzRouteConfig `
    -RouteTable $routeTable `
    -Name "ToFirewall" `
    -AddressPrefix "0.0.0.0/0" `
    -NextHopType "VirtualAppliance" `
    -NextHopIpAddress "10.0.0.4"  # Firewall private IP

Set-AzRouteTable -RouteTable $routeTable

# Associate route table with subnet
$spokeSubnet = Get-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $spoke1Vnet `
    -Name "Workload"

Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $spoke1Vnet `
    -Name "Workload" `
    -AddressPrefix $spokeSubnet.AddressPrefix `
    -RouteTable $routeTable

$spoke1Vnet | Set-AzVirtualNetwork
```

### NSG Flow Logs Analysis

```powershell
# Enable NSG Flow Logs v2
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "Network-RG" -Name "WebTier-NSG"
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "flowlogsstorage"
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "Monitoring-RG" -Name "LogAnalytics"

Set-AzNetworkWatcherFlowLog `
    -NetworkWatcher $networkWatcher `
    -Name "WebTier-FlowLog" `
    -TargetResourceId $nsg.Id `
    -StorageId $storageAccount.Id `
    -EnableRetention $true `
    -RetentionPolicyDays 30 `
    -FormatVersion 2 `
    -EnableTrafficAnalytics `
    -TrafficAnalyticsWorkspaceId $workspace.CustomerId `
    -TrafficAnalyticsInterval 10

# KQL query for traffic analysis
$query = @"
AzureNetworkAnalytics_CL
| where TimeGenerated > ago(24h)
| where FlowType_s == 'ExternalPublic'
| summarize TotalBytes = sum(InboundBytes_d + OutboundBytes_d) by DestinationIP_s
| top 10 by TotalBytes desc
"@
```

### Azure Virtual WAN

```powershell
# Create Virtual WAN
$virtualWan = New-AzVirtualWan `
    -ResourceGroupName "WAN-RG" `
    -Name "Enterprise-vWAN" `
    -Location "eastus" `
    -VirtualWANType Standard `
    -AllowVnetToVnetTraffic $true `
    -AllowBranchToBranchTraffic $true

# Create Virtual Hub
$virtualHub = New-AzVirtualHub `
    -ResourceGroupName "WAN-RG" `
    -Name "EastUS-Hub" `
    -VirtualWan $virtualWan `
    -Location "eastus" `
    -AddressPrefix "10.100.0.0/24" `
    -Sku Standard

# Connect VNet to Virtual Hub
$vnetConnection = New-AzVirtualHubVnetConnection `
    -ResourceGroupName "WAN-RG" `
    -VirtualHubName "EastUS-Hub" `
    -Name "Spoke1-Connection" `
    -RemoteVirtualNetworkId $spoke1Vnet.Id `
    -EnableInternetSecurity $true
```
