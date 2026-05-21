# Module 12 - Azure Advanced Networking (Network Jobs Focus)

## Learning Objectives
By the end of this module, you will be able to:
- Design and configure Azure Virtual Networks (VNet)
- Set up VNet Peering (local and global)
- Configure VPN Gateway for site-to-site and point-to-site connectivity
- Connect on-premises networks to Azure using ExpressRoute
- Understand all hybrid connectivity options
- Implement Hub-Spoke network topology
- Configure BGP and routing
- Understand Azure networking from a Network Engineer's perspective

---

## 12.1 Azure Virtual Networks (VNet) - Deep Dive

### What is an Azure VNet?
A Virtual Network is the fundamental building block for private networking in Azure. It is logically isolated from other VNets and provides:
- IP address space isolation
- Subnet segmentation
- Traffic filtering (NSGs)
- Custom routing (UDRs)
- Connectivity to on-premises (VPN/ExpressRoute)
- Connectivity between VNets (Peering)

### VNet Address Space Design

```
Best Practice - Plan your IP space carefully!
(IP ranges cannot overlap between peered VNets)

Enterprise Example:
┌─────────────────────────────────────────────────────────────────┐
│  10.0.0.0/8  ──  Reserved for Azure                             │
│                                                                  │
│  Region: East US        Region: West US                         │
│  ┌─────────────────┐    ┌─────────────────┐                    │
│  │  10.1.0.0/16    │    │  10.2.0.0/16    │                    │
│  │                 │    │                 │                    │
│  │  Hub:10.1.0.0/24│    │  Hub:10.2.0.0/24│                    │
│  │  Spoke1:10.1.1.0│    │  Spoke1:10.2.1.0│                    │
│  │  Spoke2:10.1.2.0│    │  Spoke2:10.2.2.0│                    │
│  └─────────────────┘    └─────────────────┘                    │
│                                                                  │
│  On-Premises: 192.168.0.0/16 (NO overlap with Azure!)           │
└─────────────────────────────────────────────────────────────────┘
```

### VNet Components

| Component | Description |
|-----------|-------------|
| **Address Space** | CIDR block (e.g., 10.1.0.0/16) |
| **Subnets** | Segments of address space |
| **DNS Servers** | Custom or Azure-provided |
| **Service Endpoints** | Private access to Azure services |
| **Private Endpoints** | PaaS services with private IP |
| **DDoS Protection** | Standard or Basic tier |
| **BGP Communities** | Route filtering with ExpressRoute |

### Reserved IP Addresses in Every Subnet

In every Azure subnet, 5 IP addresses are reserved:

```
Example: Subnet 10.1.1.0/24

10.1.1.0   - Network address
10.1.1.1   - Azure Gateway (default router)
10.1.1.2   - Azure DNS mapping
10.1.1.3   - Azure DNS mapping
10.1.1.255 - Broadcast address

Usable: 10.1.1.4 to 10.1.1.254 = 251 addresses
```

### Creating VNets and Subnets

```powershell
# Create VNet with multiple subnets
$subnetConfigs = @(
    New-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "10.1.0.0/27",
    New-AzVirtualNetworkSubnetConfig -Name "AzureBastionSubnet" -AddressPrefix "10.1.0.64/26",
    New-AzVirtualNetworkSubnetConfig -Name "AzureFirewallSubnet" -AddressPrefix "10.1.1.0/26",
    New-AzVirtualNetworkSubnetConfig -Name "ManagementSubnet" -AddressPrefix "10.1.2.0/24",
    New-AzVirtualNetworkSubnetConfig -Name "WebSubnet" -AddressPrefix "10.1.3.0/24",
    New-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -AddressPrefix "10.1.4.0/24",
    New-AzVirtualNetworkSubnetConfig -Name "DataSubnet" -AddressPrefix "10.1.5.0/24"
)

New-AzVirtualNetwork `
    -Name "Hub-VNet" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -AddressPrefix "10.1.0.0/16" `
    -Subnet $subnetConfigs `
    -DnsServer "10.1.0.4"
```

```bash
# Azure CLI
az network vnet create \
    --resource-group "Network-RG" \
    --name "Hub-VNet" \
    --address-prefix "10.1.0.0/16" \
    --subnet-name "GatewaySubnet" \
    --subnet-prefix "10.1.0.0/27"

# Add additional subnets
az network vnet subnet create \
    --resource-group "Network-RG" \
    --vnet-name "Hub-VNet" \
    --name "WebSubnet" \
    --address-prefix "10.1.3.0/24"
```

### Special Subnets (Reserved Names)

| Subnet Name | Purpose |
|-------------|---------|
| **GatewaySubnet** | VPN Gateway or ExpressRoute Gateway |
| **AzureBastionSubnet** | Azure Bastion (min /26) |
| **AzureFirewallSubnet** | Azure Firewall (min /26) |
| **AzureFirewallManagementSubnet** | Azure Firewall management |

> **Important**: These subnet names are reserved. You cannot rename them.

---

## 12.2 VNet Peering

### What is VNet Peering?
VNet Peering connects two Azure VNets so that traffic between them uses Microsoft's backbone network (not public internet). Traffic is private, low-latency, and high-bandwidth.

### Types of VNet Peering

| Type | Description |
|------|-------------|
| **Local VNet Peering** | VNets in the same Azure region |
| **Global VNet Peering** | VNets in different Azure regions |

### VNet Peering - How It Works

```
Without Peering:
VM1 (VNet-A) ──► Internet ──► VM2 (VNet-B)
(Public IP, slower, less secure)

With VNet Peering:
VM1 (VNet-A) ──► Microsoft Backbone ──► VM2 (VNet-B)
(Private IP, fast, secure, no internet)
```

### Peering Configurations

```
┌────────────────────────────────────────────────────────────────────┐
│                        VNet Peering Options                         │
│                                                                     │
│  VNet-A ◄────────────────────────────────────────► VNet-B          │
│                                                                     │
│  Allow VNet access:          ✓ (traffic flows between VNets)       │
│  Allow forwarded traffic:    ✓ (needed for hub-spoke routing)      │
│  Allow gateway transit:      ✓ (on Hub VNet - share VPN gateway)   │
│  Use remote gateway:         ✓ (on Spoke VNets - use hub gateway)  │
└────────────────────────────────────────────────────────────────────┘
```

### Key Peering Properties

| Property | Description | When to Enable |
|----------|-------------|----------------|
| **Allow VNet access** | Allow traffic between VNets | Always on |
| **Allow forwarded traffic** | Allow traffic from other VNets forwarded through this VNet | Hub-spoke scenarios |
| **Allow gateway transit** | Let peered VNets use this VNet's gateway | On hub VNet |
| **Use remote gateway** | Use gateway in peered VNet | On spoke VNets |

### Creating VNet Peering

```powershell
# Peer Hub → Spoke (from Hub perspective)
Add-AzVirtualNetworkPeering `
    -Name "Hub-to-Spoke1" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Spoke1-VNet" -ResourceGroupName "Network-RG").Id `
    -AllowForwardedTraffic `
    -AllowGatewayTransit

# Peer Spoke → Hub (from Spoke perspective)
Add-AzVirtualNetworkPeering `
    -Name "Spoke1-to-Hub" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Spoke1-VNet" -ResourceGroupName "Network-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-RG").Id `
    -AllowForwardedTraffic `
    -UseRemoteGateways
```

```bash
# Azure CLI
az network vnet peering create \
    --resource-group "Network-RG" \
    --name "Hub-to-Spoke1" \
    --vnet-name "Hub-VNet" \
    --remote-vnet "Spoke1-VNet" \
    --allow-vnet-access \
    --allow-forwarded-traffic \
    --allow-gateway-transit

az network vnet peering create \
    --resource-group "Network-RG" \
    --name "Spoke1-to-Hub" \
    --vnet-name "Spoke1-VNet" \
    --remote-vnet "Hub-VNet" \
    --allow-vnet-access \
    --allow-forwarded-traffic \
    --use-remote-gateways
```

### Peering Limitations

| Limitation | Detail |
|------------|--------|
| **No transitive routing** | VNet-A ↔ VNet-B ↔ VNet-C does NOT mean A ↔ C (need NVA or Azure Route Server) |
| **No overlapping address spaces** | 10.1.0.0/16 cannot peer with 10.1.0.0/24 |
| **Cannot update address space** | Must delete peering first |
| **Gateway requirement** | Only one gateway per VNet for peering |

### Transitive Routing Problem and Solution

```
Problem:
Spoke1 ◄──── Hub ────► Spoke2
Spoke1 CANNOT talk to Spoke2 directly via peering

Solutions:

Option 1: Use Azure Firewall or NVA in Hub
Spoke1 ──► Hub (Azure Firewall) ──► Spoke2
Add UDR in Spoke1 pointing 10.2.0.0/16 to Firewall private IP
Add UDR in Spoke2 pointing 10.1.0.0/16 to Firewall private IP

Option 2: Azure Route Server
Deploy Route Server in Hub → enables dynamic routing propagation
Spokes learn routes from NVA via BGP

Option 3: Virtual WAN (vWAN)
Microsoft-managed hub with built-in transitive routing
```

---

## 12.3 VPN Gateway

### What is Azure VPN Gateway?
A VPN Gateway is a specific type of Virtual Network Gateway that sends encrypted traffic over the public internet between Azure and on-premises networks.

### VPN Gateway Architecture

```
┌─────────────────┐                        ┌─────────────────┐
│   On-Premises   │                        │      Azure      │
│                 │   Encrypted Tunnel     │                 │
│  VPN Device    │◄──────────────────────►│   VPN Gateway   │
│  (Router/FW)   │   (IPSec/IKE)          │   (GatewaySubnet│
│                 │   over Internet        │)                │
│  192.168.0.0/24│                        │  10.1.0.0/16    │
└─────────────────┘                        └─────────────────┘
```

### VPN Gateway SKUs

| SKU | Throughput | Max Tunnels | Use Case |
|-----|-----------|-------------|----------|
| **Basic** | 100 Mbps | 10 | Dev/Test only |
| **VpnGw1** | 650 Mbps | 30 | Small-Medium |
| **VpnGw2** | 1 Gbps | 30 | Medium |
| **VpnGw3** | 1.25 Gbps | 30 | Large |
| **VpnGw4** | 5 Gbps | 100 | Very Large |
| **VpnGw5** | 10 Gbps | 100 | Enterprise |
| **VpnGwAZ1-5** | Same + AZ | Same | Availability Zone |

> **Note**: Basic SKU does NOT support BGP, Zone redundancy, or Active-Active.

### VPN Connection Types

| Type | Description |
|------|-------------|
| **Site-to-Site (S2S)** | Permanent tunnel from on-prem to Azure |
| **Point-to-Site (P2S)** | Individual client devices to Azure |
| **VNet-to-VNet** | Connect two Azure VNets via VPN |

### Site-to-Site VPN (S2S)

```
On-Premises Network                    Azure
┌──────────────────┐                  ┌──────────────────┐
│                  │                  │   VNet           │
│  Corporate LAN   │     IPSec/IKE   │  (10.1.0.0/16)  │
│  192.168.0.0/24  │◄───────────────►│                  │
│                  │   Port 500 UDP  │  VPN Gateway     │
│  VPN Device      │   Port 4500 UDP │  (GatewaySubnet) │
│  (Public IP)     │                  │  (Public IP)     │
└──────────────────┘                  └──────────────────┘
```

#### Creating S2S VPN

```powershell
# Step 1: Create VNet with GatewaySubnet
$gwSubnet = New-AzVirtualNetworkSubnetConfig `
    -Name "GatewaySubnet" `
    -AddressPrefix "10.1.0.0/27"

$vnet = New-AzVirtualNetwork `
    -Name "Hub-VNet" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -AddressPrefix "10.1.0.0/16" `
    -Subnet $gwSubnet

# Step 2: Create Public IP for VPN Gateway
$gwPIP = New-AzPublicIpAddress `
    -Name "VPN-Gateway-PIP" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -AllocationMethod Static `
    -Sku Standard

# Step 3: Create Gateway IP Configuration
$gwSubnetRef = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
$gwIPConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "GatewayIPConfig" `
    -SubnetId $gwSubnetRef.Id `
    -PublicIpAddressId $gwPIP.Id

# Step 4: Create VPN Gateway (takes 30-45 minutes)
$vpnGateway = New-AzVirtualNetworkGateway `
    -Name "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -IpConfigurations $gwIPConfig `
    -GatewayType Vpn `
    -VpnType RouteBased `
    -GatewaySku VpnGw1 `
    -EnableBgp $true `
    -Asn 65010

Write-Host "VPN Gateway Public IP: $($gwPIP.IpAddress)"
Write-Host "Share this IP with the on-premises network team!"

# Step 5: Create Local Network Gateway (represents on-premises)
$localGW = New-AzLocalNetworkGateway `
    -Name "OnPrem-LNG" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -GatewayIpAddress "203.0.113.1" `      # On-prem VPN device public IP
    -AddressPrefix "192.168.0.0/24" `       # On-prem network prefix
    -BgpPeeringAddress "192.168.0.1" `      # On-prem BGP peer IP
    -Asn 65020

# Step 6: Create VPN Connection
$vpnConnection = New-AzVirtualNetworkGatewayConnection `
    -Name "Azure-to-OnPrem" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -VirtualNetworkGateway1 $vpnGateway `
    -LocalNetworkGateway2 $localGW `
    -ConnectionType IPsec `
    -SharedKey "Your-PreSharedKey-Here" `
    -EnableBgp $true
```

### Point-to-Site VPN (P2S)

Allows individual clients (laptops, desktops) to connect to Azure VNet.

```
Individual Users                         Azure VNet
┌─────────────┐                         ┌──────────────────┐
│ Laptop/PC   │   P2S VPN Tunnel        │                  │
│ (anywhere)  │◄───────────────────────►│   VPN Gateway    │
│             │                         │                  │
└─────────────┘                         └──────────────────┘
                VPN Client Software
                (Azure VPN Client or
                 built-in Windows VPN)
```

#### P2S Authentication Methods

| Method | Description |
|--------|-------------|
| **Certificate** | Azure certificate authentication |
| **Azure AD** | Sign in with Azure AD credentials |
| **RADIUS** | Existing RADIUS server |

#### Creating P2S VPN

```powershell
# After creating VPN Gateway, configure P2S

# Step 1: Create root certificate (self-signed for testing)
$rootCert = New-SelfSignedCertificate `
    -Type Custom `
    -KeySpec Signature `
    -Subject "CN=P2SRootCert" `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyUsageProperty Sign `
    -KeyUsage CertSign

# Export root cert public key
$certBytes = $rootCert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert)
$p2sRootCertData = [System.Convert]::ToBase64String($certBytes)

# Step 2: Create client certificate
$clientCert = New-SelfSignedCertificate `
    -Type Custom `
    -DnsName "P2SClientCert" `
    -KeySpec Signature `
    -Subject "CN=P2SClientCert" `
    -KeyExportPolicy Exportable `
    -HashAlgorithm sha256 `
    -KeyLength 2048 `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -Signer $rootCert `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")

# Step 3: Configure P2S on VPN Gateway
$p2sConfig = New-AzVpnClientConfiguration `
    -ResourceGroupName "Network-RG" `
    -Name "Hub-VPN-Gateway" `
    -AuthenticationMethod "EAPTLS"

Add-AzVpnClientRootCertificate `
    -VpnClientRootCertificateName "P2SRootCert" `
    -VirtualNetworkGatewayName "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG" `
    -PublicCertData $p2sRootCertData
```

### VPN Gateway Active-Active Configuration

```
                     Active-Active VPN (High Availability)
                     
On-Prem VPN Device 1 ──────► Azure VPN GW Instance 1
                                                       } Same Connection
On-Prem VPN Device 2 ──────► Azure VPN GW Instance 2

Advantages:
- No downtime during gateway maintenance
- Both instances handle traffic simultaneously
- BGP required for Active-Active
```

### IKE/IPSec Policy Configuration

```powershell
# Create custom IKE/IPSec policy
$ikePolicy = New-AzIpsecPolicy `
    -IkeEncryption AES256 `
    -IkeIntegrity SHA384 `
    -DhGroup DHGroup24 `
    -IpsecEncryption AES256 `
    -IpsecIntegrity SHA256 `
    -PfsGroup PFS24 `
    -SALifeTimeSeconds 28800 `
    -SADataSizeKilobytes 102400000

# Apply to VPN connection
Set-AzVirtualNetworkGatewayConnection `
    -VirtualNetworkGatewayConnection (Get-AzVirtualNetworkGatewayConnection -Name "Azure-to-OnPrem" -ResourceGroupName "Network-RG") `
    -IpsecPolicies $ikePolicy
```

---

## 12.4 Connecting On-Premises to Azure - All Options

### Overview of Hybrid Connectivity Options

```
┌────────────────────────────────────────────────────────────────────────┐
│              On-Premises to Azure Connectivity Options                  │
│                                                                         │
│  Option 1: VPN Gateway (Site-to-Site)                                  │
│  On-prem ──────── IPSec over Internet ──────────► Azure VNet           │
│  Bandwidth: up to 10 Gbps | Cost: Low | Latency: Variable              │
│                                                                         │
│  Option 2: ExpressRoute                                                 │
│  On-prem ──── Private Dedicated Circuit ────────► Azure                │
│  Bandwidth: 50 Mbps to 100 Gbps | Cost: High | Latency: Predictable    │
│                                                                         │
│  Option 3: ExpressRoute + VPN Failover                                 │
│  ExpressRoute primary + VPN backup                                      │
│  High availability for critical workloads                               │
│                                                                         │
│  Option 4: Virtual WAN                                                  │
│  Managed hub connecting branches, VNets, ExpressRoute, VPN              │
│  Ideal for: SD-WAN, large scale branch connectivity                     │
│                                                                         │
│  Option 5: Azure Bastion (Management access only)                      │
│  Browser-based RDP/SSH to VMs without public IPs                       │
└────────────────────────────────────────────────────────────────────────┘
```

### Option Comparison

| Feature | VPN Gateway | ExpressRoute | Virtual WAN |
|---------|-------------|--------------|-------------|
| **Medium** | Public Internet | Private network | Both |
| **Bandwidth** | Up to 10 Gbps | Up to 100 Gbps | Varies |
| **Latency** | Variable | Predictable, low | Varies |
| **SLA** | 99.9-99.95% | 99.95% | 99.95% |
| **Encryption** | Yes (IPSec) | No (optional MACsec) | Yes |
| **Setup time** | Hours | Weeks/months | Hours-days |
| **Cost** | Low | High | Medium |
| **BGP** | Optional | Required | Yes |

---

## 12.5 Azure ExpressRoute

### What is ExpressRoute?
ExpressRoute creates private connections between Azure datacenters and on-premises infrastructure through connectivity providers.

### ExpressRoute Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                       ExpressRoute Architecture                       │
│                                                                       │
│  Customer Site           Connectivity Provider         Microsoft      │
│  ┌──────────┐           ┌─────────────────┐          ┌──────────┐    │
│  │          │           │                 │          │          │    │
│  │ Customer │           │   MPLS/Carrier  │          │  Azure   │    │
│  │  Edge    │◄─────────►│   Network       │◄────────►│  Edge    │    │
│  │ (CE)     │           │                 │          │  (MSEE)  │    │
│  │          │  Private  │                 │  Private │          │    │
│  └──────────┘  Circuit  └─────────────────┘ Peering └──────────┘    │
│                                                              │        │
│                                                              ▼        │
│                                                     ┌──────────────┐ │
│                                                     │  Azure VNet  │ │
│                                                     │  (via VNet   │ │
│                                                     │   Gateway)   │ │
│                                                     └──────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### ExpressRoute Connection Models

| Model | Description |
|-------|-------------|
| **Cloud Exchange Co-location** | Hosted in colocation facility with provider |
| **Point-to-Point Ethernet** | Dedicated Ethernet between your site and Microsoft |
| **Any-to-Any (IPVPN)** | Integrate Azure into your WAN (MPLS) |
| **ExpressRoute Direct** | Direct connection to Microsoft 100 Gbps |

### ExpressRoute Peering Types

| Peering | Traffic |
|---------|---------|
| **Private Peering** | Azure VNets (VM-level connectivity) |
| **Microsoft Peering** | Microsoft 365, Azure public services |

```
Private Peering:
On-prem ──────► ExpressRoute Circuit ──────► Azure VNets

Microsoft Peering:
On-prem ──────► ExpressRoute Circuit ──────► Microsoft 365, Azure Storage (public endpoints)
```

### ExpressRoute Bandwidth Options

| Speed | Use Case |
|-------|----------|
| 50 Mbps | Small office |
| 100 Mbps | Small-medium |
| 200 Mbps | Medium |
| 500 Mbps | Medium-large |
| 1 Gbps | Large enterprise |
| 2 Gbps | Very large |
| 5 Gbps | High-bandwidth |
| 10 Gbps | Datacenter migration |
| 100 Gbps | ExpressRoute Direct |

### ExpressRoute Configuration

```powershell
# Create ExpressRoute circuit
New-AzExpressRouteCircuit `
    -Name "MyExpressRouteCircuit" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -SkuTier "Standard" `
    -SkuFamily "MeteredData" `
    -ServiceProviderName "Equinix" `
    -PeeringLocation "Silicon Valley" `
    -BandwidthInMbps 1000

# After provider provisions the circuit:
# Get the service key and provide to provider
$circuit = Get-AzExpressRouteCircuit -Name "MyExpressRouteCircuit" -ResourceGroupName "Network-RG"
Write-Host "Service Key: $($circuit.ServiceKey)"

# Configure private peering
Add-AzExpressRouteCircuitPeeringConfig `
    -Name "AzurePrivatePeering" `
    -ExpressRouteCircuit $circuit `
    -PeeringType AzurePrivatePeering `
    -PeerASN 65020 `
    -PrimaryPeerAddressPrefix "192.168.100.0/30" `
    -SecondaryPeerAddressPrefix "192.168.100.4/30" `
    -VlanId 100

Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

# Connect VNet to ExpressRoute
$circuit = Get-AzExpressRouteCircuit -Name "MyExpressRouteCircuit" -ResourceGroupName "Network-RG"
$gw = Get-AzVirtualNetworkGateway -Name "Hub-ER-Gateway" -ResourceGroupName "Network-RG"

New-AzVirtualNetworkGatewayConnection `
    -Name "VNet-to-ExpressRoute" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -VirtualNetworkGateway1 $gw `
    -PeerId $circuit.Id `
    -ConnectionType ExpressRoute
```

### ExpressRoute vs VPN Decision Guide

```
Choose ExpressRoute when:
✓ You need > 1 Gbps bandwidth
✓ Predictable latency required (financial trading, real-time)
✓ Compliance requires private network
✓ Large data migration (petabytes)
✓ Consistent performance for business-critical apps
✓ Hybrid Active Directory, SAP HANA on Azure

Choose VPN when:
✓ Budget is a concern
✓ Quick setup needed
✓ Bandwidth < 1 Gbps is sufficient
✓ Dev/Test environments
✓ Branch offices
✓ Backup connectivity for ExpressRoute
```

---

## 12.6 Azure Virtual WAN (vWAN)

### What is Virtual WAN?
Azure Virtual WAN is a networking service that provides optimized and automated branch-to-branch connectivity through Azure.

### Virtual WAN Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Azure Virtual WAN                          │
│                                                                  │
│   ┌────────────────────────────────────────────────────────┐   │
│   │                   vWAN Hub (East US)                    │   │
│   │                                                         │   │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────┐   │   │
│   │  │  VPN GW  │  │  ER GW   │  │  Azure   │  │ User │   │   │
│   │  │ (branch) │  │(datacenter│  │ Firewall │  │ VPN  │   │   │
│   │  └──────────┘  └──────────┘  └──────────┘  └──────┘   │   │
│   └───────────────────────────────────┬────────────────────┘   │
│                                       │                          │
│       ┌───────────────────────────────┼──────────────────┐      │
│       ▼                               ▼                  ▼      │
│  ┌─────────┐                    ┌─────────┐         ┌─────────┐ │
│  │ Branch  │                    │Spoke VNet│         │ Branch  │ │
│  │ Office1 │                    │(Workloads│         │ Office2 │ │
│  │(VPN S2S)│                    │)         │         │(VPN S2S)│ │
│  └─────────┘                    └─────────┘         └─────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Virtual WAN Tiers

| Feature | Basic | Standard |
|---------|-------|----------|
| Branch-to-Azure VPN | Yes | Yes |
| VNet connections | Yes | Yes |
| Branch-to-branch via Azure | No | Yes |
| ExpressRoute | No | Yes |
| User VPN (P2S) | No | Yes |
| Azure Firewall | No | Yes |
| NVA in hub | No | Yes |

---

## 12.7 Hub-Spoke Network Topology

### What is Hub-Spoke?
The most common enterprise Azure network topology where a central Hub VNet connects to multiple Spoke VNets.

### Hub-Spoke Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                      Hub-Spoke Topology                             │
│                                                                     │
│                       ┌─────────────┐                              │
│                       │   HUB VNet  │                              │
│              ┌────────┤             ├────────┐                     │
│              │        │ • VPN/ER GW │        │                     │
│              │        │ • Firewall  │        │                     │
│              │        │ • Bastion   │        │                     │
│              │        │ • DNS       │        │                     │
│              │        └──────┬──────┘        │                     │
│              │               │               │                     │
│    ┌─────────▼──┐   ┌────────▼───┐   ┌──────▼──────┐             │
│    │  Spoke1    │   │  Spoke2    │   │   Spoke3    │             │
│    │  VNet      │   │  VNet      │   │   VNet      │             │
│    │            │   │            │   │             │             │
│    │  Dev/Test  │   │ Production │   │   DMZ       │             │
│    └────────────┘   └────────────┘   └─────────────┘             │
│                                                                     │
│              │                                                      │
│      ┌───────▼──────────────────┐                                  │
│      │   On-Premises Network    │                                  │
│      │   (via VPN or ER)        │                                  │
│      └──────────────────────────┘                                  │
└────────────────────────────────────────────────────────────────────┘
```

### Hub Services

| Service | Purpose |
|---------|---------|
| **Azure Firewall** | Traffic inspection and filtering |
| **VPN/ExpressRoute Gateway** | Hybrid connectivity |
| **Azure Bastion** | Secure VM access |
| **DNS Resolver** | Private DNS resolution |
| **NVA** | Third-party network appliance |

### Creating Hub-Spoke with UDRs

```powershell
# Create route table for spoke subnets
# Forces all traffic through Hub Firewall

$routes = @(
    New-AzRouteConfig -Name "to-onprem" -AddressPrefix "192.168.0.0/16" -NextHopType VirtualAppliance -NextHopIpAddress "10.1.1.4",
    New-AzRouteConfig -Name "to-spoke2" -AddressPrefix "10.2.0.0/16" -NextHopType VirtualAppliance -NextHopIpAddress "10.1.1.4",
    New-AzRouteConfig -Name "to-internet" -AddressPrefix "0.0.0.0/0" -NextHopType VirtualAppliance -NextHopIpAddress "10.1.1.4"
)

$routeTable = New-AzRouteTable `
    -Name "Spoke1-RouteTable" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -Route $routes

# Associate with Spoke1 subnet
$vnet = Get-AzVirtualNetwork -Name "Spoke1-VNet" -ResourceGroupName "Network-RG"
Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name "AppSubnet" `
    -AddressPrefix "10.2.1.0/24" `
    -RouteTable $routeTable
$vnet | Set-AzVirtualNetwork
```

---

## 12.8 BGP (Border Gateway Protocol) in Azure

### What is BGP?
BGP is the routing protocol of the internet. In Azure, BGP enables dynamic route exchange between Azure VPN Gateway/ExpressRoute and on-premises routers.

### Why BGP in Azure?

| Without BGP | With BGP |
|-------------|----------|
| Static routes only | Dynamic route learning |
| Manual updates when network changes | Automatic route updates |
| Single connection | Supports active-active, failover |
| Can't handle route changes | Graceful failover |

### BGP Key Concepts

| Term | Description |
|------|-------------|
| **ASN** | Autonomous System Number (identifies network) |
| **BGP Peer** | Router exchanging routes |
| **Route Advertisement** | Sending routes to peer |
| **AS Path** | List of ASNs a route traversed |

### Azure Reserved ASNs

| ASN | Reserved For |
|-----|-------------|
| 65515 | Azure VPN Gateway |
| 65517 | Azure Route Server |
| 12076 | Microsoft (ExpressRoute) |
| 65000-65535 | Private ASN range (for customers) |

### Configuring BGP with VPN Gateway

```powershell
# Create VPN Gateway with BGP
New-AzVirtualNetworkGateway `
    -Name "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -IpConfigurations $gwIPConfig `
    -GatewayType Vpn `
    -VpnType RouteBased `
    -GatewaySku VpnGw1 `
    -EnableBgp $true `
    -Asn 65010               # Azure ASN

# Local Network Gateway with BGP
New-AzLocalNetworkGateway `
    -Name "OnPrem-LNG" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -GatewayIpAddress "203.0.113.1" `
    -AddressPrefix "192.168.0.0/24" `
    -BgpPeeringAddress "192.168.0.1" `   # On-prem BGP peer IP
    -Asn 65020                             # On-prem ASN

# Get BGP status
Get-AzVirtualNetworkGatewayBgpPeerStatus `
    -VirtualNetworkGatewayName "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG"

# Get advertised routes
Get-AzVirtualNetworkGatewayAdvertisedRoute `
    -VirtualNetworkGatewayName "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG" `
    -Peer "192.168.0.1"

# Get learned routes
Get-AzVirtualNetworkGatewayLearnedRoute `
    -VirtualNetworkGatewayName "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG"
```

---

## 12.9 Azure Route Server

### What is Azure Route Server?
Azure Route Server enables dynamic routing between your NVA (Network Virtual Appliance) and Azure virtual network, eliminating the need to manually configure route tables.

### Route Server Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                         VNet                                      │
│                                                                   │
│   ┌──────────────────┐      BGP       ┌──────────────────┐       │
│   │   NVA / VPN      │◄──────────────►│  Azure Route     │       │
│   │   Appliance      │   Route Exchange│  Server          │       │
│   └──────────────────┘                └──────────────────┘       │
│                                                  │                │
│                                    Programmed    │                │
│                                    into VMs      │                │
│                                                  ▼                │
│                                       All VNet Resources          │
└──────────────────────────────────────────────────────────────────┘
```

### Creating Azure Route Server

```powershell
# Create Route Server subnet in VNet
$subnet = New-AzVirtualNetworkSubnetConfig `
    -Name "RouteServerSubnet" `
    -AddressPrefix "10.1.10.0/27"

# Create Route Server Public IP
$rsPIP = New-AzPublicIpAddress `
    -Name "RouteServer-PIP" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -AllocationMethod Static `
    -Sku Standard

# Create Route Server
New-AzRouteServer `
    -RouteServerName "MyRouteServer" `
    -ResourceGroupName "Network-RG" `
    -Location "East US" `
    -HostedSubnet $subnet.Id `
    -PublicIpAddress $rsPIP `
    -AllowBranchToBranchTraffic    # Enable transitive routing between spokes

# Add BGP peer (NVA)
Add-AzRouteServerPeer `
    -RouteServerName "MyRouteServer" `
    -ResourceGroupName "Network-RG" `
    -PeerName "MyNVA" `
    -PeerIp "10.1.1.5" `
    -PeerAsn 65100
```

---

## 12.10 Azure Networking from a Network Engineer's Perspective

### Network Engineer's Day-to-Day Tasks

```
┌────────────────────────────────────────────────────────────────────┐
│              Azure Network Engineer Responsibilities                 │
│                                                                     │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐ │
│  │   Design &       │  │   Operations     │  │   Security       │ │
│  │   Architecture   │  │                  │  │                  │ │
│  │                  │  │  • Monitor VPN   │  │  • NSG rules     │ │
│  │  • VNet design   │  │    tunnels       │  │  • Azure FW      │ │
│  │  • IP planning   │  │  • Troubleshoot  │  │  • DDoS          │ │
│  │  • Peering       │  │    connectivity  │  │  • Private EP    │ │
│  │  • Hub-spoke     │  │  • BGP routes    │  │  • WAF           │ │
│  │  • DR planning   │  │  • DNS issues    │  │  • Zero trust    │ │
│  └──────────────────┘  └──────────────────┘  └──────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

### Key Networking Skills for Azure Jobs

| Skill Category | Technologies |
|----------------|-------------|
| **Cloud Networking** | VNet, Subnet, UDR, NSG, ASG |
| **Hybrid Connectivity** | VPN Gateway, ExpressRoute, Virtual WAN |
| **DNS** | Azure DNS, Private DNS, DNS Resolver |
| **Load Balancing** | Azure LB, Application Gateway, Front Door, Traffic Manager |
| **Security** | Azure Firewall, WAF, DDoS Protection |
| **Monitoring** | Network Watcher, Connection Monitor, NSG Flow Logs |
| **Routing Protocols** | BGP, static routes, UDR |
| **Protocols** | IPSec/IKE, BGP, DNS, HTTP/S, TCP/IP |

### Common Network Interview Questions

**Q: What is the difference between NSG and Azure Firewall?**
> NSG is a basic layer-3/4 filter applied to subnets or NICs using rules. Azure Firewall is a fully stateful managed firewall service with FQDN filtering, threat intelligence, TLS inspection, and IDPS. NSG = cheap, simple. Azure Firewall = enterprise, advanced, expensive.

**Q: How does VNet peering handle transitive routing?**
> VNet peering is NOT transitive by default. If A peers with B and B peers with C, A cannot talk to C through B. You need Azure Firewall/NVA in the hub with UDRs or Azure Route Server to enable transitivity.

**Q: What is the GatewaySubnet and why is it special?**
> GatewaySubnet is a reserved subnet where VPN Gateway or ExpressRoute Gateway is deployed. It must be named exactly "GatewaySubnet." No other resources should be placed in it. Minimum size /27 (Microsoft recommends /27 or larger).

**Q: What is the difference between Route-Based and Policy-Based VPN?**
> Route-Based (RouteBased): Uses routing to determine which traffic goes through the tunnel. Supports dynamic routing, multiple tunnels, P2S, and active-active. Policy-Based (PolicyBased): Uses static policies/ACLs to define traffic. Only one tunnel, no P2S, no active-active, legacy. Always prefer Route-Based.

**Q: Explain ExpressRoute private peering.**
> Private peering connects your on-premises network to Azure VNets using private IP space. Traffic never traverses the public internet. Requires a /30 subnet for each BGP session (primary and secondary). Requires an ExpressRoute Gateway in your VNet.

**Q: What are the BGP ASN requirements for Azure VPN?**
> Azure VPN Gateway uses ASN 65515 by default. You can customize it. On-premises should use a different ASN (private range 64512-65534). ASNs 65515, 65517, 65518, and 12076 are reserved by Microsoft.

**Q: How would you implement a highly available VPN connection?**
> Use Active-Active VPN Gateway (both instances active). Configure two tunnels to two on-premises VPN devices. Use BGP for dynamic failover. This gives you resilience at both Azure side and on-premises side.

### Network Troubleshooting Commands

```powershell
# Check VPN connection status
Get-AzVirtualNetworkGatewayConnection `
    -Name "Azure-to-OnPrem" `
    -ResourceGroupName "Network-RG" |
    Select-Object Name, ConnectionStatus, EgressBytesTransferred, IngressBytesTransferred

# Get BGP peer status
Get-AzVirtualNetworkGatewayBgpPeerStatus `
    -VirtualNetworkGatewayName "Hub-VPN-Gateway" `
    -ResourceGroupName "Network-RG" |
    Format-Table

# Check effective routes on a NIC
Get-AzEffectiveRouteTable `
    -ResourceGroupName "VM-RG" `
    -NetworkInterfaceName "MyVM-nic" |
    Format-Table

# Check effective NSG rules
Get-AzEffectiveNetworkSecurityGroup `
    -ResourceGroupName "VM-RG" `
    -NetworkInterfaceName "MyVM-nic"

# Test connectivity using Network Watcher
$result = Test-AzNetworkWatcherConnectivity `
    -NetworkWatcher (Get-AzNetworkWatcher -Location "East US") `
    -SourceId "/subscriptions/{sub-id}/resourceGroups/VM-RG/providers/Microsoft.Compute/virtualMachines/VM1" `
    -DestinationAddress "10.2.1.4" `
    -DestinationPort 443

Write-Host "Connection Status: $($result.ConnectionStatus)"
Write-Host "Avg Latency: $($result.AvgLatencyInMs) ms"

# IP flow verify (test NSG rules)
Test-AzNetworkWatcherIPFlow `
    -NetworkWatcher (Get-AzNetworkWatcher -Location "East US") `
    -TargetVirtualMachineId "/subscriptions/{sub-id}/resourceGroups/VM-RG/providers/Microsoft.Compute/virtualMachines/VM1" `
    -Direction Inbound `
    -Protocol TCP `
    -RemoteIPAddress "192.168.1.100" `
    -LocalIPAddress "10.1.1.4" `
    -LocalPort 80 `
    -RemotePort 12345
```

### Azure DNS Deep Dive

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Azure DNS Architecture                          │
│                                                                      │
│  ┌─────────────────────┐    ┌─────────────────────┐                │
│  │  Azure Public DNS   │    │  Azure Private DNS  │                │
│  │                     │    │                     │                │
│  │  contoso.com → A    │    │  internal.corp.com  │                │
│  │  www → 1.2.3.4      │    │  web01 → 10.1.1.4  │                │
│  │  (Internet facing)  │    │  sql01 → 10.1.2.4  │                │
│  │                     │    │  (Private/internal) │                │
│  └─────────────────────┘    └─────────────────────┘                │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              Azure DNS Private Resolver                      │   │
│  │                                                              │   │
│  │  On-premises DNS ──► Forwarding Rules ──► Azure Private DNS │   │
│  │  Azure VMs DNS   ──► Conditional Forward ──► On-prem DNS    │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

```powershell
# Create Private DNS Zone
New-AzPrivateDnsZone `
    -ResourceGroupName "Network-RG" `
    -Name "internal.contoso.com"

# Link to VNet (auto-registration)
New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName "Network-RG" `
    -ZoneName "internal.contoso.com" `
    -Name "Hub-VNet-Link" `
    -VirtualNetworkId (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-RG").Id `
    -EnableRegistration $true    # Auto-register VMs in this zone

# Create DNS record
New-AzPrivateDnsRecordSet `
    -ResourceGroupName "Network-RG" `
    -ZoneName "internal.contoso.com" `
    -Name "webserver01" `
    -RecordType A `
    -Ttl 300 `
    -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address "10.1.3.10")
```

---

## Hands-on Exercises

### Exercise 1: Build Hub-Spoke Network

```powershell
# Create Hub VNet
$hubVNet = New-AzVirtualNetwork `
    -Name "Hub-VNet" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -AddressPrefix "10.0.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $hubVNet -AddressPrefix "10.0.0.0/27"
Add-AzVirtualNetworkSubnetConfig -Name "ManagementSubnet" -VirtualNetwork $hubVNet -AddressPrefix "10.0.1.0/24"
$hubVNet | Set-AzVirtualNetwork

# Create Spoke1 VNet
$spoke1VNet = New-AzVirtualNetwork `
    -Name "Spoke1-VNet" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -AddressPrefix "10.1.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "WebSubnet" -VirtualNetwork $spoke1VNet -AddressPrefix "10.1.1.0/24"
$spoke1VNet | Set-AzVirtualNetwork

# Create Spoke2 VNet
$spoke2VNet = New-AzVirtualNetwork `
    -Name "Spoke2-VNet" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -AddressPrefix "10.2.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "AppSubnet" -VirtualNetwork $spoke2VNet -AddressPrefix "10.2.1.0/24"
$spoke2VNet | Set-AzVirtualNetwork

# Peer Hub ↔ Spoke1
Add-AzVirtualNetworkPeering `
    -Name "Hub-to-Spoke1" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-Lab-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Spoke1-VNet" -ResourceGroupName "Network-Lab-RG").Id `
    -AllowForwardedTraffic

Add-AzVirtualNetworkPeering `
    -Name "Spoke1-to-Hub" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Spoke1-VNet" -ResourceGroupName "Network-Lab-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-Lab-RG").Id `
    -AllowForwardedTraffic

# Peer Hub ↔ Spoke2
Add-AzVirtualNetworkPeering `
    -Name "Hub-to-Spoke2" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-Lab-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Spoke2-VNet" -ResourceGroupName "Network-Lab-RG").Id `
    -AllowForwardedTraffic

Add-AzVirtualNetworkPeering `
    -Name "Spoke2-to-Hub" `
    -VirtualNetwork (Get-AzVirtualNetwork -Name "Spoke2-VNet" -ResourceGroupName "Network-Lab-RG") `
    -RemoteVirtualNetworkId (Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-Lab-RG").Id `
    -AllowForwardedTraffic

# Verify peering status
Get-AzVirtualNetworkPeering `
    -VirtualNetworkName "Hub-VNet" `
    -ResourceGroupName "Network-Lab-RG" |
    Select-Object Name, PeeringState | Format-Table

Write-Host "Hub-Spoke topology created successfully!"
```

### Exercise 2: Configure Site-to-Site VPN

```powershell
# Create GatewaySubnet (already in hub above)
$vnet = Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-Lab-RG"

# Create VPN Gateway Public IP
$gwPIP = New-AzPublicIpAddress `
    -Name "VPN-GW-PIP" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -AllocationMethod Static -Sku Standard

# Create gateway config
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
$gwIPConfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "GWIPConfig" `
    -SubnetId $gwSubnet.Id `
    -PublicIpAddressId $gwPIP.Id

# Create VPN Gateway (takes ~30 minutes)
Write-Host "Creating VPN Gateway - this takes 30-45 minutes..."
$vpnGW = New-AzVirtualNetworkGateway `
    -Name "Hub-VPN-GW" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -IpConfigurations $gwIPConfig `
    -GatewayType Vpn `
    -VpnType RouteBased `
    -GatewaySku VpnGw1 `
    -EnableBgp $true `
    -Asn 65010

Write-Host "VPN Gateway created. Public IP: $($gwPIP.IpAddress)"
Write-Host "Provide this IP to the on-premises team to configure their VPN device"

# Create Local Network Gateway
$localGW = New-AzLocalNetworkGateway `
    -Name "OnPrem-LNG" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -GatewayIpAddress "1.2.3.4" `        # Replace with real on-prem IP
    -AddressPrefix @("192.168.1.0/24", "192.168.2.0/24")

# Create connection
New-AzVirtualNetworkGatewayConnection `
    -Name "Azure-OnPrem-Conn" `
    -ResourceGroupName "Network-Lab-RG" `
    -Location "East US" `
    -VirtualNetworkGateway1 $vpnGW `
    -LocalNetworkGateway2 $localGW `
    -ConnectionType IPsec `
    -SharedKey "SuperSecretPreSharedKey123!" `
    -EnableBgp $false

# Check connection status
Get-AzVirtualNetworkGatewayConnection `
    -Name "Azure-OnPrem-Conn" `
    -ResourceGroupName "Network-Lab-RG" |
    Select-Object Name, ConnectionStatus
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| VNet | Isolated network, address spaces, subnets, 5 reserved IPs per subnet |
| VNet Peering | Microsoft backbone, not transitive, both sides needed, gateway transit |
| VPN Gateway | IPSec over internet, S2S/P2S, RouteBased preferred, BGP support |
| ExpressRoute | Private dedicated circuit, predictable latency, up to 100 Gbps |
| Virtual WAN | Managed hub, branch connectivity, scales to 1000s of branches |
| Hub-Spoke | Centralize shared services (FW, GW, Bastion) in hub |
| BGP | Dynamic routing, required for active-active, failover |
| Route Server | Dynamic NVA route injection into VNet |

---

## Networking Job Roles in Azure

| Role | Focus Areas |
|------|-------------|
| **Azure Network Engineer** | VNet, peering, VPN, ExpressRoute, routing |
| **Cloud Architect (Network)** | Hub-spoke design, WAN strategy, IP planning |
| **Network Security Engineer** | Azure Firewall, WAF, NSG, DDoS, Zero Trust |
| **SD-WAN/WAN Engineer** | Virtual WAN, BGP, branch connectivity |
| **DevNetOps Engineer** | IaC for networking (Terraform/Bicep), CI/CD |

---

## Review Questions

1. What are the 5 reserved IP addresses in every Azure subnet?
2. What is the difference between local and global VNet peering?
3. Why is VNet peering NOT transitive and how do you solve it?
4. When would you choose ExpressRoute over VPN Gateway?
5. What is the purpose of the "Allow Gateway Transit" peering property?
6. What is BGP and why is it important in Azure hybrid connectivity?
7. What is the minimum subnet size for GatewaySubnet, AzureBastionSubnet, and AzureFirewallSubnet?

---

## Additional Resources

- [VNet Documentation](https://docs.microsoft.com/azure/virtual-network/)
- [VPN Gateway Documentation](https://docs.microsoft.com/azure/vpn-gateway/)
- [ExpressRoute Documentation](https://docs.microsoft.com/azure/expressroute/)
- [Azure Virtual WAN](https://docs.microsoft.com/azure/virtual-wan/)
- [Hub-Spoke Architecture](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
- [Azure BGP Overview](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-bgp-overview)
- [Network Watcher](https://docs.microsoft.com/azure/network-watcher/)
