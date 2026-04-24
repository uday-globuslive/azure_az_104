# Module 07 - Azure Networking - Part II

## Learning Objectives
By the end of this module, you will be able to:
- Configure Azure Application Gateway
- Implement Azure Front Door Service
- Set up Azure Traffic Manager for global load balancing
- Use Application Security Groups (ASGs)
- Deploy and configure Azure Load Balancers
- Implement Azure Firewall
- Configure Azure Bastion for secure VM access
- Use Network Watcher for diagnostics
- Understand Azure ExpressRoute

---

## 7.1 Application Gateway

### What is Application Gateway?
Azure Application Gateway is a web traffic load balancer (Layer 7) that enables you to manage traffic to your web applications.

### Key Features

| Feature | Description |
|---------|-------------|
| **URL-based routing** | Route based on URL path |
| **SSL/TLS termination** | Offload encryption |
| **Web Application Firewall (WAF)** | Protect against attacks |
| **Autoscaling** | Scale based on traffic |
| **Zone redundancy** | Deploy across availability zones |
| **Session affinity** | Cookie-based affinity |

#### 🏢 Real-World Application Gateway Use Cases:

**Multi-Application Load Balancing:**
```
Company: MediaStream (Video streaming + API platform)
Challenge: Route traffic to different backends based on URL path

Routing Configuration:
┌─────────────────────────────────────────────────────────────────┐
│                    Application Gateway WAF_v2                    │
│                                                                  │
│  Public IP: 52.168.x.x → app.mediastream.com                    │
│                                                                  │
│  Routing Rules:                                                  │
│  ├── /api/*     → API Backend Pool (AKS cluster)               │
│  ├── /video/*   → Streaming Pool (Media Services VMs)          │
│  ├── /admin/*   → Admin Pool (Private VMs, restrict IPs)       │
│  └── /*         → Frontend Pool (Static web app)               │
│                                                                  │
│  WAF Rules Applied:                                              │
│  ├── OWASP 3.2 rule set                                         │
│  ├── Rate limiting: 1000 req/min per IP                         │
│  ├── SQL injection protection                                    │
│  └── Custom rule: Block countries not in service area          │
│                                                                  │
│  Traffic Flow (Peak):                                            │
│  ├── 50,000 requests/second total                               │
│  ├── /video/*: 70% of traffic                                   │
│  ├── /api/*: 25% of traffic                                     │
│  └── Other: 5%                                                   │
└─────────────────────────────────────────────────────────────────┘

Architecture:
Internet ──► App Gateway ──┬──► AKS (API pods)
            (WAF enabled)  ├──► Media VMs (video)
                           ├──► Admin VMs (restricted)
                           └──► Static Web (CDN backed)

Results:
├── Single entry point for all services
├── WAF blocked 50,000 attacks/day
├── SSL termination reduced backend CPU 30%
├── Autoscaling handled 10x traffic spike
└── Cost: $800/month (vs $5,000 for separate LBs)
```

**Blue-Green Deployment:**
```
Company: ShopFast (E-commerce, zero downtime deployments)
Challenge: Switch traffic between blue/green environments

Application Gateway Configuration:
┌─────────────────────────────────────────────────────────────────┐
│  Backend Pools:                                                  │
│  ├── Blue Pool:  10.0.1.10, 10.0.1.11 (Current Production)     │
│  └── Green Pool: 10.0.2.10, 10.0.2.11 (New Version)            │
│                                                                  │
│  Deployment Process:                                             │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │ Step 1: Deploy v2.0 to Green Pool                            │ │
│  │ Step 2: Health probes confirm Green is healthy               │ │
│  │ Step 3: Change routing rule: Blue Pool → Green Pool          │ │
│  │ Step 4: 100% traffic now on Green                            │ │
│  │ Step 5: Monitor for 30 minutes                               │ │
│  │ Step 6: (If issues) Rollback: Green → Blue (instant)        │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Canary Option (Gradual):                                        │
│  ├── Rule 1: 90% → Blue Pool                                    │
│  ├── Rule 2: 10% → Green Pool (test subset)                     │
│  └── Adjust percentage as confidence grows                      │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Zero downtime deployments
├── Rollback time: 30 seconds
├── A/B testing enabled for new features
└── Developer confidence: Ship twice daily
```

### Application Gateway Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Application Gateway                           │
│                                                                      │
│  ┌─────────────────────┐                                            │
│  │   Frontend IP(s)    │  Public/Private IP addresses               │
│  └──────────┬──────────┘                                            │
│             │                                                        │
│  ┌──────────▼──────────┐                                            │
│  │     Listeners       │  HTTP/HTTPS, ports, SSL certs              │
│  └──────────┬──────────┘                                            │
│             │                                                        │
│  ┌──────────▼──────────┐                                            │
│  │   Routing Rules     │  Basic or Path-based                       │
│  └──────────┬──────────┘                                            │
│             │                                                        │
│  ┌──────────▼──────────┐         ┌─────────────────┐                │
│  │   Backend Pools     │────────►│  Backend Targets│                │
│  │   (HTTP Settings)   │         │  VMs, VMSS, IPs │                │
│  └─────────────────────┘         └─────────────────┘                │
└─────────────────────────────────────────────────────────────────────┘
```

### Application Gateway SKUs

| SKU | Features | Use Case |
|-----|----------|----------|
| **Standard_v2** | Autoscaling, zone redundancy | Production |
| **WAF_v2** | All Standard + WAF | Security-critical |

### Creating Application Gateway

```powershell
# Create subnet for Application Gateway
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
Add-AzVirtualNetworkSubnetConfig -Name "AppGatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.10.0/24"
$vnet | Set-AzVirtualNetwork

# Create public IP
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "Network-RG" `
  -Name "AppGW-PIP" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static"

# Create Application Gateway IP configuration
$gwIPConfig = New-AzApplicationGatewayIPConfiguration `
  -Name "appGwIPConfig" `
  -Subnet (Get-AzVirtualNetworkSubnetConfig -Name "AppGatewaySubnet" -VirtualNetwork $vnet)

# Create frontend IP configuration
$frontendIPConfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name "appGwFrontendIP" `
  -PublicIPAddress $publicIP

# Create frontend port
$frontendPort = New-AzApplicationGatewayFrontendPort `
  -Name "appGwFrontendPort" `
  -Port 80

# Create backend pool
$backendPool = New-AzApplicationGatewayBackendAddressPool `
  -Name "appGwBackendPool" `
  -BackendIPAddresses "10.0.1.4","10.0.1.5"

# Create backend HTTP settings
$backendHttpSettings = New-AzApplicationGatewayBackendHttpSetting `
  -Name "appGwBackendHttpSettings" `
  -Port 80 `
  -Protocol "Http" `
  -CookieBasedAffinity "Disabled" `
  -RequestTimeout 30

# Create HTTP listener
$listener = New-AzApplicationGatewayHttpListener `
  -Name "appGwHttpListener" `
  -Protocol "Http" `
  -FrontendIPConfiguration $frontendIPConfig `
  -FrontendPort $frontendPort

# Create routing rule
$rule = New-AzApplicationGatewayRequestRoutingRule `
  -Name "appGwRule" `
  -RuleType "Basic" `
  -HttpListener $listener `
  -BackendAddressPool $backendPool `
  -BackendHttpSettings $backendHttpSettings `
  -Priority 100

# Create Application Gateway
$sku = New-AzApplicationGatewaySku -Name "Standard_v2" -Tier "Standard_v2" -Capacity 2

New-AzApplicationGateway `
  -ResourceGroupName "Network-RG" `
  -Name "MyAppGateway" `
  -Location "East US" `
  -Sku $sku `
  -GatewayIPConfigurations $gwIPConfig `
  -FrontendIPConfigurations $frontendIPConfig `
  -FrontendPorts $frontendPort `
  -BackendAddressPools $backendPool `
  -BackendHttpSettingsCollection $backendHttpSettings `
  -HttpListeners $listener `
  -RequestRoutingRules $rule
```

### URL Path-Based Routing

```powershell
# Path map for /images/* and /video/*
$imagePathRule = New-AzApplicationGatewayPathRuleConfig `
  -Name "imagePathRule" `
  -Paths "/images/*" `
  -BackendAddressPool $imageBackendPool `
  -BackendHttpSettings $backendHttpSettings

$videoPathRule = New-AzApplicationGatewayPathRuleConfig `
  -Name "videoPathRule" `
  -Paths "/video/*" `
  -BackendAddressPool $videoBackendPool `
  -BackendHttpSettings $backendHttpSettings

$urlPathMap = New-AzApplicationGatewayUrlPathMapConfig `
  -Name "urlPathMap" `
  -PathRules $imagePathRule,$videoPathRule `
  -DefaultBackendAddressPool $defaultBackendPool `
  -DefaultBackendHttpSettings $backendHttpSettings
```

### WAF Configuration

```powershell
# Create WAF configuration
$wafConfig = New-AzApplicationGatewayWebApplicationFirewallConfiguration `
  -Enabled $true `
  -FirewallMode "Prevention" `
  -RuleSetType "OWASP" `
  -RuleSetVersion "3.2" `
  -RequestBodyCheck $true `
  -MaxRequestBodySizeInKb 128 `
  -FileUploadLimitInMb 100
```

---

## 7.2 Azure Front Door Service

### What is Azure Front Door?
Azure Front Door is a global, scalable entry-point that uses Microsoft's global edge network to create fast, secure, and highly available web applications.

### Front Door Features

| Feature | Description |
|---------|-------------|
| **Global load balancing** | Route to nearest/fastest backend |
| **SSL offload** | Terminate SSL at the edge |
| **URL-based routing** | Route based on URL path |
| **WAF** | Protection at the edge |
| **Caching** | Content caching at edge |
| **Health probes** | Monitor backend health |

### Front Door Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                      Azure Front Door                             │
│                                                                   │
│   Users worldwide                                                 │
│        │                                                          │
│        ▼                                                          │
│   ┌─────────────────────────────────────────────────┐            │
│   │              Edge Locations (POPs)               │            │
│   │   North America | Europe | Asia | Australia     │            │
│   └─────────────────────────────────────────────────┘            │
│        │                    │                    │                │
│        ▼                    ▼                    ▼                │
│   ┌──────────┐        ┌──────────┐        ┌──────────┐          │
│   │ Backend  │        │ Backend  │        │ Backend  │          │
│   │ Pool 1   │        │ Pool 2   │        │ Pool 3   │          │
│   │ (US)     │        │ (Europe) │        │ (Asia)   │          │
│   └──────────┘        └──────────┘        └──────────┘          │
└──────────────────────────────────────────────────────────────────┘
```

### Creating Azure Front Door

```bash
# Create Front Door profile
az afd profile create \
  --resource-group "FrontDoor-RG" \
  --profile-name "MyFrontDoor" \
  --sku "Premium_AzureFrontDoor"

# Create endpoint
az afd endpoint create \
  --resource-group "FrontDoor-RG" \
  --profile-name "MyFrontDoor" \
  --endpoint-name "myendpoint" \
  --enabled-state "Enabled"

# Create origin group
az afd origin-group create \
  --resource-group "FrontDoor-RG" \
  --profile-name "MyFrontDoor" \
  --origin-group-name "MyOriginGroup" \
  --probe-request-type "HEAD" \
  --probe-protocol "Https" \
  --probe-interval-in-seconds 120 \
  --sample-size 4 \
  --successful-samples-required 3

# Add origin
az afd origin create \
  --resource-group "FrontDoor-RG" \
  --profile-name "MyFrontDoor" \
  --origin-group-name "MyOriginGroup" \
  --origin-name "WebApp1" \
  --host-name "webapp1.azurewebsites.net" \
  --http-port 80 \
  --https-port 443 \
  --origin-host-header "webapp1.azurewebsites.net" \
  --priority 1 \
  --weight 1000

# Create route
az afd route create \
  --resource-group "FrontDoor-RG" \
  --profile-name "MyFrontDoor" \
  --endpoint-name "myendpoint" \
  --route-name "MyRoute" \
  --origin-group "MyOriginGroup" \
  --supported-protocols "Http" "Https" \
  --patterns-to-match "/*" \
  --forwarding-protocol "HttpsOnly"
```

### Front Door vs Application Gateway

| Feature | Front Door | Application Gateway |
|---------|------------|---------------------|
| Scope | Global | Regional |
| Protocol | HTTP/HTTPS | HTTP/HTTPS, WebSocket |
| WAF | Yes | Yes |
| SSL offload | Yes | Yes |
| Caching | Yes | No |
| Backend types | Public endpoints | VMs, IPs in VNet |

---

## 7.3 Azure Traffic Manager

### What is Traffic Manager?
Azure Traffic Manager is a DNS-based traffic load balancer that enables you to distribute traffic across global Azure regions.

### Routing Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **Priority** | Route to primary; failover to secondary | Active-passive DR |
| **Weighted** | Distribute based on weights | Gradual deployment |
| **Performance** | Route to fastest endpoint | Best latency |
| **Geographic** | Route based on user location | Data sovereignty |
| **Multivalue** | Return multiple healthy endpoints | Client-side LB |
| **Subnet** | Route based on source IP | Enterprise routing |

### Traffic Manager Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Traffic Manager Profile                   │
│              myapp.trafficmanager.net                        │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Routing Method: Performance               │  │
│  └───────────────────────────────────────────────────────┘  │
│                          │                                   │
│         ┌────────────────┼────────────────┐                 │
│         ▼                ▼                ▼                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │ Endpoint 1  │  │ Endpoint 2  │  │ Endpoint 3  │        │
│  │ East US     │  │ West Europe │  │ Southeast   │        │
│  │ webapp1     │  │ webapp2     │  │ Asia        │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### Creating Traffic Manager

```powershell
# Create Traffic Manager profile
$profile = New-AzTrafficManagerProfile `
  -ResourceGroupName "TM-RG" `
  -Name "MyTrafficManager" `
  -TrafficRoutingMethod "Performance" `
  -RelativeDnsName "myapp" `
  -Ttl 30 `
  -MonitorProtocol "HTTPS" `
  -MonitorPort 443 `
  -MonitorPath "/"

# Add Azure endpoint
New-AzTrafficManagerEndpoint `
  -ResourceGroupName "TM-RG" `
  -ProfileName "MyTrafficManager" `
  -Name "EastUS-Endpoint" `
  -Type "AzureEndpoints" `
  -TargetResourceId "/subscriptions/.../sites/webapp-eastus" `
  -EndpointStatus "Enabled"

# Add external endpoint
New-AzTrafficManagerEndpoint `
  -ResourceGroupName "TM-RG" `
  -ProfileName "MyTrafficManager" `
  -Name "OnPrem-Endpoint" `
  -Type "ExternalEndpoints" `
  -Target "onprem.contoso.com" `
  -EndpointLocation "West US" `
  -EndpointStatus "Enabled"
```

```bash
# Azure CLI
az network traffic-manager profile create \
  --resource-group "TM-RG" \
  --name "MyTrafficManager" \
  --routing-method "Performance" \
  --unique-dns-name "myapp" \
  --ttl 30 \
  --protocol "HTTPS" \
  --port 443 \
  --path "/"

az network traffic-manager endpoint create \
  --resource-group "TM-RG" \
  --profile-name "MyTrafficManager" \
  --name "EastUS-Endpoint" \
  --type "azureEndpoints" \
  --target-resource-id "/subscriptions/.../sites/webapp-eastus" \
  --endpoint-status "enabled"
```

### Nested Traffic Manager Profiles

```
                 Parent Profile
              (Geographic routing)
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
    Child Profile  Child Profile  Child Profile
    (US users)     (EU users)     (Asia users)
    Performance    Performance    Performance
         │            │            │
    ┌────┴────┐  ┌────┴────┐  ┌────┴────┐
    │  East   │  │  West   │  │ South-  │
    │  US     │  │  Europe │  │ east    │
    │  West   │  │  North  │  │ Asia    │
    │  US     │  │  Europe │  │         │
    └─────────┘  └─────────┘  └─────────┘
```

---

## 7.4 Application Security Groups

### What are Application Security Groups?
Application Security Groups (ASGs) allow you to group VMs and define network security policies based on application structure.

### ASG Benefits

| Benefit | Description |
|---------|-------------|
| **Simplified rules** | Use app names instead of IPs |
| **Scalable** | Add/remove VMs without rule changes |
| **Micro-segmentation** | Fine-grained security |
| **Maintainable** | Self-documenting rules |

### ASG Architecture

```
Without ASG:                      With ASG:
NSG Rule:                         NSG Rule:
Allow 10.0.1.4 → 10.0.2.4:1433   Allow WebServers → DBServers:1433
Allow 10.0.1.5 → 10.0.2.4:1433
Allow 10.0.1.6 → 10.0.2.4:1433

┌─────────────────┐              ┌─────────────────┐
│   Web VMs       │              │   ASG:          │
│   10.0.1.4      │              │   WebServers    │
│   10.0.1.5      │              │   (Contains all │
│   10.0.1.6      │              │   web VMs)      │
└─────────────────┘              └─────────────────┘
        │                                │
        ▼                                ▼
┌─────────────────┐              ┌─────────────────┐
│   DB VMs        │              │   ASG:          │
│   10.0.2.4      │              │   DBServers     │
│   10.0.2.5      │              │   (Contains all │
└─────────────────┘              │   DB VMs)       │
                                 └─────────────────┘
```

### Creating and Using ASGs

```powershell
# Create ASGs
$webASG = New-AzApplicationSecurityGroup `
  -ResourceGroupName "Network-RG" `
  -Name "WebServers" `
  -Location "East US"

$dbASG = New-AzApplicationSecurityGroup `
  -ResourceGroupName "Network-RG" `
  -Name "DatabaseServers" `
  -Location "East US"

# Create NSG rule using ASGs
$rule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-Web-to-DB" `
  -Description "Allow web servers to access database" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceApplicationSecurityGroupId $webASG.Id `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $dbASG.Id `
  -DestinationPortRange 1433

$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName "Network-RG" `
  -Name "AppNSG" `
  -Location "East US" `
  -SecurityRules $rule

# Associate NIC with ASG
$nic = Get-AzNetworkInterface -ResourceGroupName "Network-RG" -Name "WebVM-NIC"
$nic.IpConfigurations[0].ApplicationSecurityGroups = @($webASG)
$nic | Set-AzNetworkInterface
```

```bash
# Azure CLI
az network asg create \
  --resource-group "Network-RG" \
  --name "WebServers"

az network asg create \
  --resource-group "Network-RG" \
  --name "DatabaseServers"

az network nsg rule create \
  --resource-group "Network-RG" \
  --nsg-name "AppNSG" \
  --name "Allow-Web-to-DB" \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs "WebServers" \
  --destination-asgs "DatabaseServers" \
  --destination-port-ranges 1433
```

---

## 7.5 Azure Load Balancers

### What is Azure Load Balancer?
Azure Load Balancer is a Layer 4 (TCP/UDP) load balancer that distributes incoming traffic among healthy VMs.

### Load Balancer Types

| Type | SKU | Features |
|------|-----|----------|
| **Public** | Basic/Standard | Internet to VMs |
| **Internal** | Basic/Standard | VNet to VMs |

### SKU Comparison

| Feature | Basic | Standard |
|---------|-------|----------|
| Backend pool size | Up to 300 | Up to 1000 |
| Health probes | TCP, HTTP | TCP, HTTP, HTTPS |
| Availability Zones | Not supported | Zone-redundant |
| SLA | None | 99.99% |
| Outbound rules | Not supported | Supported |
| Security | Open by default | Secure by default |

### Load Balancer Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Azure Load Balancer                          │
│                                                                  │
│  ┌─────────────────────┐     ┌─────────────────────────────┐   │
│  │   Frontend IP       │     │      Backend Pool           │   │
│  │   (Public/Private)  │────►│   ┌────┐ ┌────┐ ┌────┐      │   │
│  └─────────────────────┘     │   │VM1 │ │VM2 │ │VM3 │      │   │
│            │                 │   └────┘ └────┘ └────┘      │   │
│            │                 └─────────────────────────────┘   │
│  ┌─────────▼───────────┐                   ▲                   │
│  │    Load Balancing   │                   │                   │
│  │       Rules         │───────────────────┘                   │
│  │  (Port, Protocol)   │                                       │
│  └─────────────────────┘     ┌─────────────────────────────┐   │
│            │                 │      Health Probes           │   │
│  ┌─────────▼───────────┐     │   TCP/HTTP/HTTPS checks      │   │
│  │   Inbound NAT Rules │     └─────────────────────────────┘   │
│  │   (Individual VMs)  │                                       │
│  └─────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Public Load Balancer

```powershell
# Create public IP
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName "LB-RG" `
  -Name "LB-PublicIP" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static"

# Create frontend IP configuration
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "LB-Frontend" `
  -PublicIpAddress $publicIP

# Create backend pool
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig `
  -Name "LB-BackendPool"

# Create health probe
$healthProbe = New-AzLoadBalancerProbeConfig `
  -Name "HealthProbe-HTTP" `
  -Protocol "Http" `
  -Port 80 `
  -RequestPath "/" `
  -IntervalInSeconds 15 `
  -ProbeCount 2

# Create load balancing rule
$lbRule = New-AzLoadBalancerRuleConfig `
  -Name "HTTP-Rule" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Probe $healthProbe `
  -Protocol "Tcp" `
  -FrontendPort 80 `
  -BackendPort 80 `
  -EnableFloatingIP $false

# Create inbound NAT rule for RDP
$natRule1 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name "RDP-VM1" `
  -FrontendIpConfiguration $frontendIP `
  -Protocol "Tcp" `
  -FrontendPort 3389 `
  -BackendPort 3389

# Create Load Balancer
$lb = New-AzLoadBalancer `
  -ResourceGroupName "LB-RG" `
  -Name "MyLoadBalancer" `
  -Location "East US" `
  -Sku "Standard" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Probe $healthProbe `
  -LoadBalancingRule $lbRule `
  -InboundNatRule $natRule1
```

### Creating Internal Load Balancer

```powershell
# Get subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "BackendSubnet"

# Create frontend IP (private)
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "ILB-Frontend" `
  -PrivateIpAddress "10.0.2.100" `
  -SubnetId $subnet.Id

# Create internal load balancer
$ilb = New-AzLoadBalancer `
  -ResourceGroupName "LB-RG" `
  -Name "InternalLB" `
  -Location "East US" `
  -Sku "Standard" `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -Probe $healthProbe `
  -LoadBalancingRule $lbRule
```

### Adding VMs to Backend Pool

```powershell
# Get load balancer
$lb = Get-AzLoadBalancer -ResourceGroupName "LB-RG" -Name "MyLoadBalancer"
$backendPool = $lb.BackendAddressPools[0]

# Update NIC to use backend pool
$nic = Get-AzNetworkInterface -ResourceGroupName "LB-RG" -Name "VM1-NIC"
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools = $backendPool
$nic | Set-AzNetworkInterface
```

---

## 7.6 Azure Firewall

### What is Azure Firewall?
Azure Firewall is a managed, cloud-based network security service that protects your Azure Virtual Network resources.

### Azure Firewall Features

| Feature | Description |
|---------|-------------|
| **Stateful firewall** | Full state inspection |
| **Built-in HA** | No additional configuration |
| **Threat intelligence** | Block known malicious IPs |
| **FQDN filtering** | Filter by domain names |
| **Network rules** | L4 filtering |
| **Application rules** | L7 filtering (HTTP/HTTPS) |
| **DNAT** | Destination NAT for inbound |

### Azure Firewall Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Hub VNet                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Azure Firewall Subnet                       │   │
│  │                10.0.100.0/26                             │   │
│  │  ┌─────────────────────────────────────────────────┐    │   │
│  │  │            Azure Firewall                        │    │   │
│  │  │         (Public IP + Private IP)                 │    │   │
│  │  │                                                  │    │   │
│  │  │   Rules:                                         │    │   │
│  │  │   - Network Rules (IP/Port)                      │    │   │
│  │  │   - Application Rules (FQDN)                     │    │   │
│  │  │   - NAT Rules (DNAT)                            │    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                  │
│                        (UDR Routes)                             │
│                              │                                  │
│         ┌────────────────────┼─────────────────────┐           │
│         ▼                    ▼                     ▼           │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐     │
│  │ Spoke VNet 1 │    │ Spoke VNet 2 │    │ On-premises  │     │
│  └──────────────┘    └──────────────┘    └──────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

### Creating Azure Firewall

```powershell
# Create firewall subnet (must be named AzureFirewallSubnet)
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Hub-RG" -Name "HubVNet"
Add-AzVirtualNetworkSubnetConfig `
  -Name "AzureFirewallSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.100.0/26"
$vnet | Set-AzVirtualNetwork

# Create public IP
$fwPublicIP = New-AzPublicIpAddress `
  -ResourceGroupName "Hub-RG" `
  -Name "AzFW-PIP" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static"

# Create firewall
$fw = New-AzFirewall `
  -ResourceGroupName "Hub-RG" `
  -Name "AzureFirewall" `
  -Location "East US" `
  -VirtualNetwork $vnet `
  -PublicIpAddress $fwPublicIP
```

### Firewall Rules

#### Network Rules

```powershell
# Create network rule collection
$netRule1 = New-AzFirewallNetworkRule `
  -Name "Allow-DNS" `
  -Protocol UDP `
  -SourceAddress "10.0.0.0/8" `
  -DestinationAddress "168.63.129.16" `
  -DestinationPort 53

$netRuleCollection = New-AzFirewallNetworkRuleCollection `
  -Name "Net-Coll-01" `
  -Priority 200 `
  -Rule $netRule1 `
  -ActionType "Allow"

$fw.NetworkRuleCollections.Add($netRuleCollection)
Set-AzFirewall -AzureFirewall $fw
```

#### Application Rules

```powershell
# Create application rule collection
$appRule1 = New-AzFirewallApplicationRule `
  -Name "Allow-Microsoft" `
  -SourceAddress "10.0.0.0/8" `
  -Protocol "https:443" `
  -TargetFqdn "*.microsoft.com","*.azure.com"

$appRuleCollection = New-AzFirewallApplicationRuleCollection `
  -Name "App-Coll-01" `
  -Priority 300 `
  -Rule $appRule1 `
  -ActionType "Allow"

$fw.ApplicationRuleCollections.Add($appRuleCollection)
Set-AzFirewall -AzureFirewall $fw
```

#### DNAT Rules

```powershell
# Create NAT rule for RDP
$natRule = New-AzFirewallNatRule `
  -Name "RDP-to-VM1" `
  -Protocol TCP `
  -SourceAddress "*" `
  -DestinationAddress $fwPublicIP.IpAddress `
  -DestinationPort 3389 `
  -TranslatedAddress "10.0.1.4" `
  -TranslatedPort 3389

$natRuleCollection = New-AzFirewallNatRuleCollection `
  -Name "NAT-Coll-01" `
  -Priority 100 `
  -Rule $natRule

$fw.NatRuleCollections.Add($natRuleCollection)
Set-AzFirewall -AzureFirewall $fw
```

---

## 7.7 Azure Bastion

### What is Azure Bastion?
Azure Bastion provides secure and seamless RDP/SSH connectivity to your VMs directly through the Azure Portal over SSL.

### Bastion Benefits

| Benefit | Description |
|---------|-------------|
| **No public IP needed** | VMs don't need public IPs |
| **No NSG rules** | No need to open RDP/SSH ports |
| **Protection against port scanning** | VMs not exposed to Internet |
| **Browser-based** | No client software needed |
| **Hardened and updated** | Microsoft manages the service |

### Bastion Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Azure Portal                            │
│                   (Browser-based access)                     │
└─────────────────────────────┬───────────────────────────────┘
                              │ HTTPS (443)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Virtual Network                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           AzureBastionSubnet (≥/26)                 │   │
│  │  ┌─────────────────────────────────────────────┐   │   │
│  │  │            Azure Bastion                     │   │   │
│  │  │         (Public IP required)                 │   │   │
│  │  └─────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │ RDP/SSH                          │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   VM Subnet                          │   │
│  │      ┌────────┐  ┌────────┐  ┌────────┐             │   │
│  │      │  VM1   │  │  VM2   │  │  VM3   │             │   │
│  │      │(No PIP)│  │(No PIP)│  │(No PIP)│             │   │
│  │      └────────┘  └────────┘  └────────┘             │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Creating Azure Bastion

```powershell
# Create Bastion subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
Add-AzVirtualNetworkSubnetConfig `
  -Name "AzureBastionSubnet" `
  -VirtualNetwork $vnet `
  -AddressPrefix "10.0.200.0/26"
$vnet | Set-AzVirtualNetwork

# Create public IP for Bastion
$bastionIP = New-AzPublicIpAddress `
  -ResourceGroupName "Network-RG" `
  -Name "Bastion-PIP" `
  -Location "East US" `
  -Sku "Standard" `
  -AllocationMethod "Static"

# Create Bastion
New-AzBastion `
  -ResourceGroupName "Network-RG" `
  -Name "MyBastion" `
  -PublicIpAddress $bastionIP `
  -VirtualNetwork $vnet
```

```bash
# Azure CLI
az network bastion create \
  --resource-group "Network-RG" \
  --name "MyBastion" \
  --vnet-name "MyVNet" \
  --public-ip-address "Bastion-PIP" \
  --location "eastus"
```

### Connecting via Bastion

1. Navigate to VM in Azure Portal
2. Click "Connect" → "Bastion"
3. Enter credentials
4. Click "Connect"

Session opens in browser window with full RDP/SSH capability.

---

## 7.8 Network Watcher

### What is Network Watcher?
Network Watcher provides tools to monitor, diagnose, and gain insights into your Azure network.

### Network Watcher Tools

| Tool | Purpose |
|------|---------|
| **IP Flow Verify** | Check if traffic is allowed/denied |
| **Next Hop** | Determine next hop for traffic |
| **NSG Flow Logs** | Log NSG traffic information |
| **VPN Troubleshoot** | Diagnose VPN gateway issues |
| **Packet Capture** | Capture packets on VM |
| **Connection Troubleshoot** | Test connectivity between resources |
| **Topology** | Visualize network topology |

### Enabling Network Watcher

```powershell
# Enable Network Watcher in region
New-AzNetworkWatcher `
  -Name "NetworkWatcher_eastus" `
  -ResourceGroupName "NetworkWatcherRG" `
  -Location "East US"
```

### IP Flow Verify

```powershell
# Test if traffic is allowed
$result = Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId "/subscriptions/.../virtualMachines/MyVM" `
  -Direction "Inbound" `
  -Protocol "TCP" `
  -LocalIPAddress "10.0.1.4" `
  -LocalPort 80 `
  -RemoteIPAddress "203.0.113.1" `
  -RemotePort 12345

$result.Access  # "Allow" or "Deny"
$result.RuleName  # NSG rule that matched
```

### Next Hop

```powershell
# Determine routing
$result = Get-AzNetworkWatcherNextHop `
  -NetworkWatcher $networkWatcher `
  -TargetVirtualMachineId "/subscriptions/.../virtualMachines/MyVM" `
  -SourceIPAddress "10.0.1.4" `
  -DestinationIPAddress "10.0.2.4"

$result.NextHopType  # VirtualNetwork, Internet, VirtualAppliance, etc.
$result.NextHopIpAddress
```

### NSG Flow Logs

```powershell
# Enable NSG flow logs
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "Network-RG" -Name "MyNSG"
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "flowlogsstorage"

Set-AzNetworkWatcherConfigFlowLog `
  -NetworkWatcher $networkWatcher `
  -TargetResourceId $nsg.Id `
  -StorageAccountId $storageAccount.Id `
  -EnableFlowLog $true `
  -FormatType Json `
  -FormatVersion 2
```

### Connection Troubleshoot

```powershell
# Test connectivity
$result = Test-AzNetworkWatcherConnectivity `
  -NetworkWatcher $networkWatcher `
  -SourceId "/subscriptions/.../virtualMachines/VM1" `
  -DestinationId "/subscriptions/.../virtualMachines/VM2" `
  -DestinationPort 3389

$result.ConnectionStatus  # Reachable, Unreachable
$result.AvgLatencyInMs
$result.Hops
```

---

## 7.9-7.11 Azure ExpressRoute

### What is ExpressRoute?
Azure ExpressRoute provides dedicated, private connections between your on-premises networks and Microsoft cloud services.

### ExpressRoute Benefits

| Benefit | Description |
|---------|-------------|
| **Private connectivity** | Not over public Internet |
| **Higher bandwidth** | Up to 100 Gbps |
| **Lower latency** | Predictable performance |
| **High reliability** | 99.95% SLA |
| **Global reach** | Connect any location |

### ExpressRoute Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                        On-premises Network                         │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │                   Customer Edge Router                      │   │
│  └────────────────────────────────────────────────────────────┘   │
│                              │                                     │
│                    (Physical connection)                           │
│                              │                                     │
│  ┌────────────────────────────────────────────────────────────┐   │
│  │              Connectivity Provider / Direct Port            │   │
│  │              (AT&T, Equinix, Megaport, etc.)               │   │
│  └────────────────────────────────────────────────────────────┘   │
└──────────────────────────────│────────────────────────────────────┘
                               │
┌──────────────────────────────▼────────────────────────────────────┐
│                        Microsoft Edge                              │
│                  (Primary + Secondary routers)                     │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                  ExpressRoute Circuit                        │  │
│  │  ┌──────────────────┐    ┌──────────────────┐               │  │
│  │  │ Private Peering  │    │ Microsoft Peering│               │  │
│  │  │ (Azure VNets)    │    │ (M365, Dynamics) │               │  │
│  │  └──────────────────┘    └──────────────────┘               │  │
│  └─────────────────────────────────────────────────────────────┘  │
│                              │                                     │
│                      ┌───────┴───────┐                            │
│                      ▼               ▼                            │
│              ┌──────────────┐  ┌──────────────┐                   │
│              │  Azure VNets │  │ Microsoft 365│                   │
│              └──────────────┘  └──────────────┘                   │
└───────────────────────────────────────────────────────────────────┘
```

### ExpressRoute Peering Types

| Peering | Purpose |
|---------|---------|
| **Azure Private Peering** | Connect to Azure VNets |
| **Microsoft Peering** | Connect to Microsoft 365, Azure PaaS |

### ExpressRoute SKUs

| SKU | Features |
|-----|----------|
| **Local** | Access to 1-2 regions, unlimited data |
| **Standard** | Access to region, metered data |
| **Premium** | Global access, more routes |

### Creating ExpressRoute Circuit

```powershell
# Create ExpressRoute circuit
$circuit = New-AzExpressRouteCircuit `
  -ResourceGroupName "ExpressRoute-RG" `
  -Name "MyExpressRoute" `
  -Location "East US" `
  -ServiceProviderName "Equinix" `
  -PeeringLocation "Washington DC" `
  -BandwidthInMbps 1000 `
  -SkuFamily "MeteredData" `
  -SkuTier "Standard"

# Get service key for provider
$circuit.ServiceKey
```

### Connecting VNet to ExpressRoute

```powershell
# Create ExpressRoute Gateway
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
$gwIP = New-AzPublicIpAddress -ResourceGroupName "ER-RG" -Name "ER-GW-PIP" -Location "East US" -AllocationMethod "Dynamic"
$gwIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name "gwIPConfig" -SubnetId $gwSubnet.Id -PublicIpAddressId $gwIP.Id

$erGateway = New-AzVirtualNetworkGateway `
  -ResourceGroupName "ER-RG" `
  -Name "ERGateway" `
  -Location "East US" `
  -IpConfigurations $gwIPConfig `
  -GatewayType "ExpressRoute" `
  -GatewaySku "Standard"

# Create connection to circuit
New-AzVirtualNetworkGatewayConnection `
  -ResourceGroupName "ER-RG" `
  -Name "VNet-to-ER" `
  -Location "East US" `
  -VirtualNetworkGateway1 $erGateway `
  -ConnectionType "ExpressRoute" `
  -PeerId $circuit.Id
```

---

## Hands-on Exercises

### Exercise 1: Create Internal Load Balancer

```powershell
$resourceGroup = "LB-Lab-RG"
$location = "East US"

# Create VNet
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Name "LB-VNet" -Location $location -AddressPrefix "10.0.0.0/16"
Add-AzVirtualNetworkSubnetConfig -Name "BackendSubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.1.0/24"
$vnet | Set-AzVirtualNetwork
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork (Get-AzVirtualNetwork -Name "LB-VNet" -ResourceGroupName $resourceGroup) -Name "BackendSubnet"

# Create Internal LB
$frontendIP = New-AzLoadBalancerFrontendIpConfig -Name "ILB-Frontend" -PrivateIpAddress "10.0.1.100" -SubnetId $subnet.Id
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "ILB-Backend"
$probe = New-AzLoadBalancerProbeConfig -Name "Health-TCP" -Protocol Tcp -Port 80 -IntervalInSeconds 15 -ProbeCount 2
$rule = New-AzLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $backendPool -Probe $probe -Protocol Tcp -FrontendPort 80 -BackendPort 80

New-AzLoadBalancer -ResourceGroupName $resourceGroup -Name "InternalLB" -Location $location -Sku Standard -FrontendIpConfiguration $frontendIP -BackendAddressPool $backendPool -Probe $probe -LoadBalancingRule $rule
```

### Exercise 2: Create Public Load Balancer

```powershell
# Create public IP
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Name "PLB-PIP" -Location $location -Sku Standard -AllocationMethod Static

# Create public LB
$frontendIP = New-AzLoadBalancerFrontendIpConfig -Name "PLB-Frontend" -PublicIpAddress $pip
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "PLB-Backend"
$probe = New-AzLoadBalancerProbeConfig -Name "Health-HTTP" -Protocol Http -Port 80 -RequestPath "/" -IntervalInSeconds 15 -ProbeCount 2
$rule = New-AzLoadBalancerRuleConfig -Name "HTTP-Rule" -FrontendIpConfiguration $frontendIP -BackendAddressPool $backendPool -Probe $probe -Protocol Tcp -FrontendPort 80 -BackendPort 80

New-AzLoadBalancer -ResourceGroupName $resourceGroup -Name "PublicLB" -Location $location -Sku Standard -FrontendIpConfiguration $frontendIP -BackendAddressPool $backendPool -Probe $probe -LoadBalancingRule $rule
```

### Exercise 3-6: Application Gateway, Front Door, Traffic Manager, Bastion

```powershell
# See detailed creation scripts in respective sections above

# Exercise 6: Deploy Azure Bastion
$vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroup -Name "LB-VNet"
Add-AzVirtualNetworkSubnetConfig -Name "AzureBastionSubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.200.0/26"
$vnet | Set-AzVirtualNetwork

$bastionPIP = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Name "Bastion-PIP" -Location $location -Sku Standard -AllocationMethod Static

New-AzBastion -ResourceGroupName $resourceGroup -Name "LabBastion" -PublicIpAddress $bastionPIP -VirtualNetwork $vnet
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Application Gateway | Layer 7 LB, WAF, URL routing, SSL offload |
| Front Door | Global LB, CDN, edge security |
| Traffic Manager | DNS-based routing, multiple methods |
| ASGs | Group-based NSG rules, simpler management |
| Load Balancer | Layer 4 LB, Basic/Standard SKUs |
| Azure Firewall | Managed firewall, FQDN filtering, threat intelligence |
| Bastion | Secure RDP/SSH without public IPs |
| Network Watcher | Diagnostics, flow logs, troubleshooting |
| ExpressRoute | Private connectivity, high bandwidth |

---

## Review Questions

1. Compare Application Gateway vs Azure Load Balancer.
2. What routing methods does Traffic Manager support?
3. How do Application Security Groups simplify NSG rules?
4. What is the minimum subnet size for Azure Bastion?
5. What are the three rule types in Azure Firewall?
6. What are ExpressRoute peering types?

---

## Additional Resources

- [Application Gateway Documentation](https://docs.microsoft.com/azure/application-gateway/)
- [Azure Front Door Documentation](https://docs.microsoft.com/azure/frontdoor/)
- [Traffic Manager Documentation](https://docs.microsoft.com/azure/traffic-manager/)
- [Azure Firewall Documentation](https://docs.microsoft.com/azure/firewall/)
- [Azure Bastion Documentation](https://docs.microsoft.com/azure/bastion/)
- [ExpressRoute Documentation](https://docs.microsoft.com/azure/expressroute/)

---

## 7.6 Advanced Network Security Patterns (Expert Level)

### Azure Firewall Policy with Rule Collection Groups

```powershell
# Create Firewall Policy
$fwPolicy = New-AzFirewallPolicy `
    -ResourceGroupName "Network-RG" `
    -Name "Enterprise-FW-Policy" `
    -Location "eastus" `
    -ThreatIntelMode Alert `
    -IntrusionDetection @{
        Mode = "Alert"
        Configuration = @{
            SignatureOverrides = @()
            BypassTrafficSettings = @()
        }
    }

# Create Rule Collection Group for Network Rules
$networkRule1 = New-AzFirewallPolicyNetworkRule `
    -Name "AllowDNS" `
    -Protocol UDP `
    -SourceAddress "10.0.0.0/8" `
    -DestinationAddress "168.63.129.16" `
    -DestinationPort 53

$networkRule2 = New-AzFirewallPolicyNetworkRule `
    -Name "AllowNTP" `
    -Protocol UDP `
    -SourceAddress "10.0.0.0/8" `
    -DestinationAddress "pool.ntp.org" `
    -DestinationPort 123

$networkRuleCollection = New-AzFirewallPolicyFilterRuleCollection `
    -Name "InfrastructureRules" `
    -Priority 100 `
    -ActionType Allow `
    -Rule $networkRule1, $networkRule2

$networkRCG = New-AzFirewallPolicyRuleCollectionGroup `
    -ResourceGroupName "Network-RG" `
    -FirewallPolicyName "Enterprise-FW-Policy" `
    -Name "NetworkRuleCollectionGroup" `
    -Priority 200 `
    -RuleCollection $networkRuleCollection

# Create Application Rule Collection
$appRule1 = New-AzFirewallPolicyApplicationRule `
    -Name "AllowMicrosoftUpdates" `
    -SourceAddress "10.0.0.0/8" `
    -Protocol @{"Type"="Https";"Port"=443} `
    -TargetFqdn @("*.windowsupdate.com", "*.microsoft.com", "*.msftconnecttest.com")

$appRule2 = New-AzFirewallPolicyApplicationRule `
    -Name "AllowAzureServices" `
    -SourceAddress "10.0.0.0/8" `
    -Protocol @{"Type"="Https";"Port"=443} `
    -FqdnTag @("AzureKubernetesService", "WindowsVirtualDesktop")

$appRuleCollection = New-AzFirewallPolicyFilterRuleCollection `
    -Name "ApplicationRules" `
    -Priority 100 `
    -ActionType Allow `
    -Rule $appRule1, $appRule2
```

### DDoS Protection Plan

```powershell
# Create DDoS Protection Plan
$ddosPlan = New-AzDdosProtectionPlan `
    -ResourceGroupName "Security-RG" `
    -Name "Enterprise-DDoS-Plan" `
    -Location "eastus"

# Associate with VNet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "Hub-VNet"
$vnet.DdosProtectionPlan = @{ Id = $ddosPlan.Id }
$vnet.EnableDdosProtection = $true
$vnet | Set-AzVirtualNetwork

# Configure DDoS diagnostic settings
Set-AzDiagnosticSetting `
    -ResourceId $ddosPlan.Id `
    -Name "DDoS-Diagnostics" `
    -WorkspaceId $workspace.ResourceId `
    -Enabled $true `
    -Category "DDoSProtectionNotifications", "DDoSMitigationFlowLogs", "DDoSMitigationReports"
```

### ExpressRoute with Private Peering

```powershell
# Create ExpressRoute circuit
$circuit = New-AzExpressRouteCircuit `
    -ResourceGroupName "Connectivity-RG" `
    -Name "Enterprise-ER-Circuit" `
    -Location "eastus" `
    -ServiceProviderName "Equinix" `
    -PeeringLocation "Washington DC" `
    -BandwidthInMbps 1000 `
    -Sku_Family MeteredData `
    -Sku_Tier Premium

# Configure Private Peering
Add-AzExpressRouteCircuitPeeringConfig `
    -ExpressRouteCircuit $circuit `
    -Name "AzurePrivatePeering" `
    -PeeringType AzurePrivatePeering `
    -PeerASN 65020 `
    -PrimaryPeerAddressPrefix "172.16.0.0/30" `
    -SecondaryPeerAddressPrefix "172.16.0.4/30" `
    -VlanId 100

Set-AzExpressRouteCircuit -ExpressRouteCircuit $circuit

# Create ExpressRoute Gateway
$gwsubnet = Get-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $hubVnet `
    -Name "GatewaySubnet"

$gwpip = New-AzPublicIpAddress `
    -ResourceGroupName "Connectivity-RG" `
    -Name "ER-Gateway-PIP" `
    -Location "eastus" `
    -AllocationMethod Static `
    -Sku Standard `
    -Zone 1, 2, 3

$gwipconfig = New-AzVirtualNetworkGatewayIpConfig `
    -Name "gwipconfig" `
    -SubnetId $gwsubnet.Id `
    -PublicIpAddressId $gwpip.Id

$ergateway = New-AzVirtualNetworkGateway `
    -ResourceGroupName "Connectivity-RG" `
    -Name "Enterprise-ER-Gateway" `
    -Location "eastus" `
    -IpConfigurations $gwipconfig `
    -GatewayType ExpressRoute `
    -GatewaySku ErGw2AZ

# Connect circuit to gateway
New-AzVirtualNetworkGatewayConnection `
    -ResourceGroupName "Connectivity-RG" `
    -Name "ER-Connection" `
    -Location "eastus" `
    -VirtualNetworkGateway1 $ergateway `
    -PeerId $circuit.Id `
    -ConnectionType ExpressRoute
```

### Network Troubleshooting

```powershell
# IP Flow Verify
Test-AzNetworkWatcherIPFlow `
    -NetworkWatcher $networkWatcher `
    -TargetVirtualMachineId $vm.Id `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 443 `
    -RemotePort 50000 `
    -LocalIPAddress "10.0.1.4" `
    -RemoteIPAddress "203.0.113.1"

# Next Hop
Get-AzNetworkWatcherNextHop `
    -NetworkWatcher $networkWatcher `
    -TargetVirtualMachineId $vm.Id `
    -SourceIPAddress "10.0.1.4" `
    -DestinationIPAddress "10.2.0.4"

# Effective Security Rules
Get-AzEffectiveNetworkSecurityGroup `
    -NetworkInterfaceId $nic.Id

# Connection Monitor
New-AzNetworkWatcherConnectionMonitor `
    -NetworkWatcher $networkWatcher `
    -Name "WebServer-Monitor" `
    -SourceResourceId $vm.Id `
    -DestinationAddress "www.microsoft.com" `
    -DestinationPort 443 `
    -TestFrequencySec 60
```
