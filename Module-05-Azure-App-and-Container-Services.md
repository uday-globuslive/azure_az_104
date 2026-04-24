# Module 05 - Azure App and Container Services

## Learning Objectives
By the end of this module, you will be able to:
- Deploy web applications using Azure App Service
- Create and configure App Service Web Apps for Containers
- Understand App Service Plans and pricing tiers
- Configure networking for App Services
- Work with deployment slots
- Build and manage container images
- Deploy applications using Azure Kubernetes Service (AKS)
- Use Azure Container Registry for image management

---

## 5.1 App Service Web App for Containers

### What is Azure App Service?
Azure App Service is a fully managed platform for building, deploying, and scaling web apps. It supports multiple programming languages and frameworks.

### App Service Features

| Feature | Description |
|---------|-------------|
| **Multiple Languages** | .NET, Java, Node.js, PHP, Python, Ruby |
| **DevOps Integration** | CI/CD from GitHub, Azure DevOps, Docker Hub |
| **Scaling** | Manual and autoscale |
| **Security** | SSL/TLS, authentication, network isolation |
| **Monitoring** | Built-in diagnostics and monitoring |

#### 🏢 Real-World App Service and Container Use Cases:

**Startup MVP to Scale Journey:**
```
Company: FinTech Startup (Investment App)
Journey: MVP → Product-Market Fit → Scale

Phase 1: MVP (Month 1-3)
├── App Service Plan: B1 ($55/month)
├── Stack: Node.js API + React frontend
├── Database: Azure SQL Basic ($5/month)
├── Total: ~$60/month
└── Handles: 1,000 users

Phase 2: Growth (Month 4-12)
├── App Service Plan: S1 ($70/month)
├── Added: Deployment slots (staging/prod)
├── Database: Azure SQL S1 ($30/month)
├── CDN: For static assets
└── Handles: 50,000 users

Phase 3: Scale (Year 2+)
├── App Service Plan: P2V2 auto-scale (2-10 instances)
├── AKS: For microservices architecture
├── Database: Azure SQL Hyperscale
└── Handles: 1M+ users

Key Insight: Started simple, scaled without rewriting
```

**Enterprise API Platform:**
```
Company: Insurance Corp
Challenge: Expose 50 internal APIs to partners

Architecture:
┌───────────────────────────────────────────────────────┐
│              API Management (Gateway)                  │
│  • Rate limiting: 1000 requests/minute per partner    │
│  • API keys, OAuth 2.0 authentication                 │
│  • Request/response transformation                     │
└───────────────────────┬───────────────────────────────┘
                        │
    ┌───────────────────┼───────────────────┐
    ▼                   ▼                   ▼
┌─────────┐       ┌─────────┐        ┌─────────────┐
│App Svc 1│       │App Svc 2│        │AKS Cluster  │
│Claims   │       │Quotes   │        │Policy Mgmt  │
│API      │       │API      │        │(15 services)│
└─────────┘       └─────────┘        └─────────────┘

Deployment Strategy:
- Blue/Green with deployment slots
- Canary releases: 10% traffic to new version
- Automatic rollback if error rate > 1%
```

**E-commerce on AKS (Microservices):**
```
Company: GlobalShop (Online marketplace)
Challenge: Black Friday scale (100x normal traffic)

AKS Architecture:
┌────────────────────────────────────────────────────────────┐
│                    AKS Cluster (10 nodes)                   │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────────────┐  │
│  │Catalog  │ │Cart     │ │Payment  │ │Order Processing │  │
│  │Service  │ │Service  │ │Service  │ │Service          │  │
│  │(5 pods) │ │(10 pods)│ │(3 pods) │ │(8 pods)         │  │
│  └─────────┘ └─────────┘ └─────────┘ └─────────────────┘  │
│                                                             │
│  Horizontal Pod Autoscaler:                                │
│  - Scale on CPU > 70%                                      │
│  - Scale on requests/second > 1000                         │
│                                                             │
│  Cluster Autoscaler:                                       │
│  - Min nodes: 5                                            │
│  - Max nodes: 50                                           │
│  - Scale when pods pending > 0                             │
└────────────────────────────────────────────────────────────┘

Black Friday Results:
├── Normal day: 5 nodes, ~50 pods
├── Black Friday peak: 45 nodes, ~500 pods
├── Scale-up time: 3 minutes
└── Zero downtime during 100x traffic spike
```

**Serverless Event Processing:**
```
Company: SmartHome Inc.
Challenge: Process 10M IoT events daily

Container Apps + Functions Architecture:
                                         
     IoT Devices ──► Event Hub ──► Container Apps ──► Cosmos DB
         │              │              │                   │
    (10M/day)     (Ingestion)    (Processing)         (Storage)
                        │              │
                        ▼              ▼
                   Auto-scale      Scale to zero
                   0-100 replicas  when no events

Container Apps Config:
- Min replicas: 0 (scale to zero = cost savings)
- Max replicas: 100
- Scale trigger: Event Hub messages > 100

Monthly Cost:
- Traditional VMs (always-on): $2,000
- Container Apps (scale-to-zero): $400
- Savings: 80%
```

### Web App for Containers

Web App for Containers allows you to run custom Docker containers on App Service.

```
┌─────────────────────────────────────────────────────────────┐
│                   App Service Plan                           │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           Web App for Containers                     │    │
│  │                                                      │    │
│  │  ┌────────────────────────────────────────────┐     │    │
│  │  │          Custom Docker Container           │     │    │
│  │  │                                            │     │    │
│  │  │   Your Application Code + Dependencies    │     │    │
│  │  │                                            │     │    │
│  │  └────────────────────────────────────────────┘     │    │
│  │                      │                               │    │
│  │           Pulls from Container Registry              │    │
│  │    (Docker Hub / Azure Container Registry)           │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Creating Web App for Containers

#### Using Azure Portal
1. Create a new "Web App"
2. Select "Docker Container" as publish option
3. Choose Linux as operating system
4. Configure container settings:
   - Image source (Azure Container Registry, Docker Hub, Private Registry)
   - Image and tag
   - Startup command (optional)

#### Using Azure CLI

```bash
# Create resource group
az group create --name "WebApp-RG" --location "eastus"

# Create App Service Plan (Linux)
az appservice plan create \
  --name "MyAppServicePlan" \
  --resource-group "WebApp-RG" \
  --is-linux \
  --sku "P1V2"

# Create Web App for Containers
az webapp create \
  --name "my-container-app" \
  --resource-group "WebApp-RG" \
  --plan "MyAppServicePlan" \
  --deployment-container-image-name "nginx:latest"

# Configure with Azure Container Registry
az webapp config container set \
  --name "my-container-app" \
  --resource-group "WebApp-RG" \
  --docker-custom-image-name "myacr.azurecr.io/myapp:v1" \
  --docker-registry-server-url "https://myacr.azurecr.io" \
  --docker-registry-server-user "myacr" \
  --docker-registry-server-password "<password>"
```

#### Using PowerShell

```powershell
# Create App Service Plan
New-AzAppServicePlan `
  -ResourceGroupName "WebApp-RG" `
  -Name "MyAppServicePlan" `
  -Location "East US" `
  -Linux `
  -Tier "PremiumV2" `
  -WorkerSize "Small"

# Create Web App
New-AzWebApp `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-container-app" `
  -AppServicePlan "MyAppServicePlan" `
  -ContainerImageName "nginx:latest"
```

### Multi-Container Apps

Deploy multiple containers using Docker Compose:

```yaml
# docker-compose.yml
version: '3'
services:
  web:
    image: myacr.azurecr.io/frontend:latest
    ports:
      - "80:80"
  api:
    image: myacr.azurecr.io/backend:latest
    ports:
      - "8080:8080"
  redis:
    image: redis:alpine
```

```bash
# Deploy multi-container
az webapp config container set \
  --name "my-multi-container-app" \
  --resource-group "WebApp-RG" \
  --multicontainer-config-type "compose" \
  --multicontainer-config-file "docker-compose.yml"
```

---

## 5.2 App Service Plan

### What is an App Service Plan?
An App Service Plan defines the compute resources for your web apps. All apps in the same plan share these resources.

### App Service Plan Tiers

| Tier | Features | Use Case |
|------|----------|----------|
| **Free (F1)** | Shared compute, 60 min/day | Testing, learning |
| **Shared (D1)** | Shared compute, custom domain | Development |
| **Basic (B1-B3)** | Dedicated compute, SSL | Dev/test |
| **Standard (S1-S3)** | Auto-scale, staging slots, backups | Production |
| **Premium (P1-P3)** | Enhanced scaling, more slots | High-scale production |
| **PremiumV2/V3** | Dv2/Dv3-series VMs, faster | Performance-critical |
| **Isolated (I1-I3)** | Dedicated VNet, highest scale | Enterprise isolation |

### Plan Comparison

| Feature | Free | Shared | Basic | Standard | Premium | Isolated |
|---------|------|--------|-------|----------|---------|----------|
| Custom domain | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| SSL | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| Manual scale | 1 | 1 | Up to 3 | Up to 10 | Up to 30 | Up to 100 |
| Auto-scale | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Staging slots | 0 | 0 | 0 | 5 | 20 | 20 |
| Daily backups | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| VNet Integration | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |

### Creating App Service Plan

```powershell
# Create Basic tier plan
New-AzAppServicePlan `
  -ResourceGroupName "WebApp-RG" `
  -Name "BasicPlan" `
  -Location "East US" `
  -Tier "Basic" `
  -WorkerSize "Small" `
  -NumberOfWorkers 1

# Create Premium plan with autoscale
New-AzAppServicePlan `
  -ResourceGroupName "WebApp-RG" `
  -Name "PremiumPlan" `
  -Location "East US" `
  -Tier "PremiumV2" `
  -WorkerSize "Small" `
  -NumberOfWorkers 2
```

### Scaling App Service Plans

#### Scale Up (vertical)
Change to a higher tier with more resources:

```powershell
Set-AzAppServicePlan `
  -ResourceGroupName "WebApp-RG" `
  -Name "MyPlan" `
  -Tier "PremiumV2" `
  -WorkerSize "Medium"
```

#### Scale Out (horizontal)
Add more instances:

```powershell
# Manual scale
Set-AzAppServicePlan `
  -ResourceGroupName "WebApp-RG" `
  -Name "MyPlan" `
  -NumberOfWorkers 5
```

#### Autoscale Configuration

```powershell
# Create autoscale rule based on CPU
$rule = New-AzAutoscaleRule `
  -MetricName "CpuPercentage" `
  -MetricResourceId "/subscriptions/.../serverFarms/MyPlan" `
  -TimeGrain 00:01:00 `
  -MetricStatistic Average `
  -TimeWindow 00:10:00 `
  -Operator GreaterThan `
  -Threshold 70 `
  -ScaleActionDirection Increase `
  -ScaleActionScaleType ChangeCount `
  -ScaleActionValue 1 `
  -ScaleActionCooldown 00:05:00

$profile = New-AzAutoscaleProfile `
  -DefaultCapacity 2 `
  -MaximumCapacity 10 `
  -MinimumCapacity 2 `
  -Rule $rule `
  -Name "CPU-Scale"

Add-AzAutoscaleSetting `
  -ResourceGroupName "WebApp-RG" `
  -Name "WebAppAutoscale" `
  -Location "East US" `
  -TargetResourceId "/subscriptions/.../serverFarms/MyPlan" `
  -AutoscaleProfile $profile
```

---

## 5.3 Networking for an App Service

### Network Features Overview

| Feature | Description | Tier Required |
|---------|-------------|---------------|
| **App-assigned address** | Outbound dedicated IP | All |
| **Access restrictions** | IP-based firewall | All |
| **Service endpoints** | Direct access to Azure services | Standard+ |
| **VNet Integration** | Outbound access to VNet | Standard+ |
| **Private Endpoints** | Inbound private access | Premium+ |
| **Hybrid Connections** | Connect to on-premises | Standard+ |

### VNet Integration

```
┌─────────────────────────────────────────────────────────────┐
│                      Virtual Network                         │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                 Integration Subnet                   │   │
│   │              (Delegated to App Service)              │   │
│   └─────────────────────────────────────────────────────┘   │
│                            │                                 │
│                   (Outbound traffic)                         │
│                            │                                 │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  Other Subnets                       │   │
│   │     (VMs, Databases, Private Endpoints)              │   │
│   └─────────────────────────────────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               │
┌──────────────────────────────┴──────────────────────────────┐
│                        App Service                           │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                     Web App                          │   │
│   │              (Can access VNet resources)             │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

#### Configure VNet Integration

```powershell
# Add VNet integration
$vnet = Get-AzVirtualNetwork -ResourceGroupName "Network-RG" -Name "MyVNet"
$subnet = $vnet.Subnets | Where-Object {$_.Name -eq "IntegrationSubnet"}

# Delegate subnet to App Service
$subnet.Delegations = @()
$delegation = New-AzDelegation -Name "appServiceDelegation" -ServiceName "Microsoft.Web/serverFarms"
$subnet.Delegations.Add($delegation)
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Add VNet integration to web app
Set-AzWebApp `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -VirtualNetworkId $vnet.Id `
  -SwiftSupported
```

```bash
# Azure CLI
az webapp vnet-integration add \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --vnet "MyVNet" \
  --subnet "IntegrationSubnet"
```

### Private Endpoints

Allow inbound traffic only through private IP:

```powershell
# Create private endpoint
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
  -Name "webappConnection" `
  -PrivateLinkServiceId "/subscriptions/.../sites/my-webapp" `
  -GroupId "sites"

$privateEndpoint = New-AzPrivateEndpoint `
  -ResourceGroupName "Network-RG" `
  -Name "webapp-private-endpoint" `
  -Location "East US" `
  -Subnet $subnet `
  -PrivateLinkServiceConnection $privateEndpointConnection
```

### Access Restrictions

```bash
# Add IP restriction
az webapp config access-restriction add \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --rule-name "AllowOffice" \
  --action "Allow" \
  --priority 100 \
  --ip-address "203.0.113.0/24"

# Add service endpoint restriction
az webapp config access-restriction add \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --rule-name "AllowVNet" \
  --action "Allow" \
  --priority 200 \
  --vnet-name "MyVNet" \
  --subnet "WebSubnet"
```

---

## 5.4 Deployment Slots

### What are Deployment Slots?
Deployment slots are live apps with their own hostnames. They allow you to deploy new versions and swap them with production without downtime.

### Slot Benefits

| Benefit | Description |
|---------|-------------|
| **Zero-downtime deployment** | Swap slots instantly |
| **Validation** | Test in staging before production |
| **Rollback** | Swap back if issues occur |
| **Warm-up** | Pre-warm app before swap |
| **A/B Testing** | Route traffic percentage to slots |

### Slot Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     my-webapp.azurewebsites.net              │
│                        (Production Slot)                     │
└─────────────────────────────────────────────────────────────┘
                              ↕ SWAP
┌─────────────────────────────────────────────────────────────┐
│               my-webapp-staging.azurewebsites.net            │
│                        (Staging Slot)                        │
└─────────────────────────────────────────────────────────────┘
                              ↕ SWAP
┌─────────────────────────────────────────────────────────────┐
│                 my-webapp-dev.azurewebsites.net              │
│                        (Dev Slot)                            │
└─────────────────────────────────────────────────────────────┘
```

### Creating Deployment Slots

```powershell
# Create staging slot
New-AzWebAppSlot `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -Slot "staging" `
  -AppServicePlan "MyAppServicePlan"

# Create dev slot
New-AzWebAppSlot `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -Slot "dev"
```

```bash
# Azure CLI
az webapp deployment slot create \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --slot "staging"
```

### Slot Settings

Settings can be "sticky" (slot-specific) or swap with the app:

| Setting Type | Swaps | Use Case |
|-------------|-------|----------|
| **Slot settings** | No | Environment-specific config |
| **App settings** | Yes (unless marked sticky) | Application config |
| **Connection strings** | Yes (unless marked sticky) | Database connections |

```powershell
# Set slot-specific app setting
Set-AzWebAppSlotConfigName `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -AppSettingNames "Environment","DatabaseConnection"

# Set slot setting value
Set-AzWebAppSlot `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -Slot "staging" `
  -AppSettings @{Environment="Staging"; FeatureFlag="True"}
```

### Swapping Slots

```powershell
# Swap staging to production
Switch-AzWebAppSlot `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -SourceSlotName "staging" `
  -DestinationSlotName "production"

# Swap with preview (multi-phase)
Switch-AzWebAppSlot `
  -ResourceGroupName "WebApp-RG" `
  -Name "my-webapp" `
  -SourceSlotName "staging" `
  -DestinationSlotName "production" `
  -PreserveVnet
```

```bash
# Azure CLI
az webapp deployment slot swap \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --slot "staging" \
  --target-slot "production"
```

### Traffic Routing

Route a percentage of traffic to a slot:

```bash
# Route 10% of traffic to staging
az webapp traffic-routing set \
  --name "my-webapp" \
  --resource-group "WebApp-RG" \
  --distribution staging=10
```

---

## 5.5 Container Image

### Building Container Images

#### Dockerfile Basics

```dockerfile
# Basic .NET application Dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore "MyApp.csproj"
COPY . .
RUN dotnet build "MyApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

```dockerfile
# Node.js application Dockerfile
FROM node:16-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

### Building with Docker

```bash
# Build image
docker build -t myapp:v1 .

# Run locally
docker run -d -p 8080:80 myapp:v1

# Tag for Azure Container Registry
docker tag myapp:v1 myacr.azurecr.io/myapp:v1

# Push to ACR
docker push myacr.azurecr.io/myapp:v1
```

### Building with Azure Container Registry Tasks

```bash
# Quick build using ACR Tasks
az acr build \
  --registry "myacr" \
  --image "myapp:v1" \
  --file "Dockerfile" \
  .

# Build with multi-stage Dockerfile
az acr build \
  --registry "myacr" \
  --image "myapp:{{.Run.ID}}" \
  --file "Dockerfile" \
  https://github.com/myorg/myrepo.git
```

### Container Best Practices

| Practice | Description |
|----------|-------------|
| Use small base images | Alpine-based images reduce size |
| Multi-stage builds | Separate build and runtime |
| Non-root user | Don't run as root |
| .dockerignore | Exclude unnecessary files |
| Health checks | Add HEALTHCHECK instruction |
| Layer caching | Order commands to maximize cache |

---

## 5.6 Azure Kubernetes Service (AKS)

### What is AKS?
Azure Kubernetes Service is a managed Kubernetes service that simplifies deploying, managing, and scaling containerized applications.

### AKS Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     AKS Cluster                                  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              Control Plane (Managed by Azure)             │   │
│  │   API Server | etcd | Scheduler | Controller Manager     │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     Node Pool                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │   │
│  │  │   Node 1    │  │   Node 2    │  │   Node 3    │       │   │
│  │  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │       │   │
│  │  │  │Pod    │  │  │  │Pod    │  │  │  │Pod    │  │       │   │
│  │  │  │Pod    │  │  │  │Pod    │  │  │  │Pod    │  │       │   │
│  │  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │       │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Creating AKS Cluster

```bash
# Create resource group
az group create --name "AKS-RG" --location "eastus"

# Create AKS cluster
az aks create \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster" \
  --node-count 3 \
  --node-vm-size "Standard_D2s_v3" \
  --generate-ssh-keys \
  --enable-managed-identity \
  --network-plugin "azure" \
  --enable-addons monitoring

# Get credentials
az aks get-credentials \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster"

# Verify connection
kubectl get nodes
```

```powershell
# PowerShell
New-AzAksCluster `
  -ResourceGroupName "AKS-RG" `
  -Name "MyAKSCluster" `
  -NodeCount 3 `
  -NodeVmSize "Standard_D2s_v3" `
  -NetworkPlugin "azure" `
  -GenerateSshKey
```

### Deploying to AKS

#### Deployment YAML

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myacr.azurecr.io/myapp:v1
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
```

```bash
# Deploy
kubectl apply -f deployment.yaml

# Check status
kubectl get pods
kubectl get services
kubectl get deployments
```

### Node Pools

```bash
# Add new node pool
az aks nodepool add \
  --resource-group "AKS-RG" \
  --cluster-name "MyAKSCluster" \
  --name "gpupool" \
  --node-count 2 \
  --node-vm-size "Standard_NC6" \
  --labels "hardware=gpu"

# Scale node pool
az aks nodepool scale \
  --resource-group "AKS-RG" \
  --cluster-name "MyAKSCluster" \
  --name "nodepool1" \
  --node-count 5
```

### AKS Scaling

#### Cluster Autoscaler

```bash
# Enable cluster autoscaler
az aks update \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster" \
  --enable-cluster-autoscaler \
  --min-count 1 \
  --max-count 10
```

#### Horizontal Pod Autoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### AKS Networking

| Network Plugin | Description |
|---------------|-------------|
| **kubenet** | Basic networking, NAT for pod traffic |
| **Azure CNI** | Pods get VNet IPs, direct connectivity |
| **Azure CNI Overlay** | Large-scale clusters, overlay network |

```bash
# Create AKS with Azure CNI
az aks create \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster" \
  --network-plugin "azure" \
  --vnet-subnet-id "/subscriptions/.../subnets/aks-subnet" \
  --service-cidr "10.2.0.0/16" \
  --dns-service-ip "10.2.0.10"
```

---

## 5.7 Azure Container Registry

### What is Azure Container Registry?
Azure Container Registry (ACR) is a managed Docker registry service for storing and managing container images.

### ACR Tiers

| Tier | Storage | Features |
|------|---------|----------|
| **Basic** | 10 GB | Development |
| **Standard** | 100 GB | Production, geo-replication |
| **Premium** | 500 GB | High-scale, private link, content trust |

### Creating Container Registry

```bash
# Create ACR
az acr create \
  --resource-group "ACR-RG" \
  --name "mycontainerregistry" \
  --sku "Standard" \
  --admin-enabled true

# Get login credentials
az acr credential show \
  --name "mycontainerregistry"
```

```powershell
# PowerShell
New-AzContainerRegistry `
  -ResourceGroupName "ACR-RG" `
  -Name "mycontainerregistry" `
  -Sku "Standard" `
  -EnableAdminUser
```

### Working with ACR

#### Authentication

```bash
# Login with Azure CLI
az acr login --name "mycontainerregistry"

# Login with Docker
docker login mycontainerregistry.azurecr.io

# Service Principal (for CI/CD)
az ad sp create-for-rbac \
  --name "acr-service-principal" \
  --scopes "/subscriptions/.../containerRegistries/mycontainerregistry" \
  --role "acrpush"
```

#### Push and Pull Images

```bash
# Tag image
docker tag myapp:v1 mycontainerregistry.azurecr.io/myapp:v1

# Push image
docker push mycontainerregistry.azurecr.io/myapp:v1

# Pull image
docker pull mycontainerregistry.azurecr.io/myapp:v1

# List repositories
az acr repository list --name "mycontainerregistry"

# List tags
az acr repository show-tags \
  --name "mycontainerregistry" \
  --repository "myapp"
```

### ACR Integration with AKS

```bash
# Attach ACR to AKS cluster
az aks update \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster" \
  --attach-acr "mycontainerregistry"

# Create with ACR attached
az aks create \
  --resource-group "AKS-RG" \
  --name "MyAKSCluster" \
  --attach-acr "mycontainerregistry"
```

### ACR Tasks

Automate image builds:

```bash
# Quick build
az acr build \
  --registry "mycontainerregistry" \
  --image "myapp:v1" \
  .

# Create task for automatic builds
az acr task create \
  --registry "mycontainerregistry" \
  --name "build-on-push" \
  --image "myapp:{{.Run.ID}}" \
  --context https://github.com/myorg/myrepo.git \
  --file "Dockerfile" \
  --git-access-token "<PAT>"
```

### Geo-Replication (Premium)

```bash
# Add replication to West Europe
az acr replication create \
  --registry "mycontainerregistry" \
  --location "westeurope"

# List replications
az acr replication list \
  --registry "mycontainerregistry"
```

---

## Hands-on Exercises

### Exercise 1: Create an App Service Web App for Containers

```bash
# Step 1: Create resource group and App Service Plan
az group create --name "Container-WebApp-RG" --location "eastus"

az appservice plan create \
  --name "ContainerPlan" \
  --resource-group "Container-WebApp-RG" \
  --is-linux \
  --sku "P1V2"

# Step 2: Create Web App with nginx container
az webapp create \
  --name "my-container-webapp-$(date +%s)" \
  --resource-group "Container-WebApp-RG" \
  --plan "ContainerPlan" \
  --deployment-container-image-name "nginx:alpine"

# Step 3: Configure continuous deployment
az webapp deployment container config \
  --name "my-container-webapp" \
  --resource-group "Container-WebApp-RG" \
  --enable-cd true

# Step 4: View logs
az webapp log tail \
  --name "my-container-webapp" \
  --resource-group "Container-WebApp-RG"
```

### Exercise 2: Create a Container Image

```dockerfile
# Create Dockerfile
# Dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

```bash
# Build image
docker build -t mynode-app:v1 .

# Test locally
docker run -d -p 3000:3000 mynode-app:v1
curl http://localhost:3000

# Push to ACR
docker tag mynode-app:v1 myacr.azurecr.io/mynode-app:v1
az acr login --name myacr
docker push myacr.azurecr.io/mynode-app:v1
```

### Exercise 3: Configure Azure Kubernetes Service

```bash
# Create AKS cluster
az aks create \
  --resource-group "AKS-Lab-RG" \
  --name "LabAKSCluster" \
  --node-count 2 \
  --node-vm-size "Standard_B2s" \
  --generate-ssh-keys \
  --attach-acr "myacr"

# Get credentials
az aks get-credentials --resource-group "AKS-Lab-RG" --name "LabAKSCluster"

# Deploy sample application
kubectl create deployment nginx --image=nginx:alpine --replicas=3
kubectl expose deployment nginx --port=80 --type=LoadBalancer

# Check deployment
kubectl get pods
kubectl get services

# Scale deployment
kubectl scale deployment nginx --replicas=5
```

### Exercise 4: Publish and Automate Image Deployment

```bash
# Create ACR task for automatic builds
az acr task create \
  --registry "myacr" \
  --name "auto-build" \
  --image "myapp:{{.Run.ID}}" \
  --context "https://github.com/myuser/myrepo.git" \
  --file "Dockerfile" \
  --git-access-token "$GIT_PAT" \
  --commit-trigger-enabled true

# Run task manually
az acr task run --registry "myacr" --name "auto-build"

# View task runs
az acr task list-runs --registry "myacr" --output table

# Configure AKS to use latest image
# Update deployment to use ACR webhook or GitOps
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| App Service Containers | Run custom Docker containers on managed platform |
| App Service Plans | Compute resources, tiers from Free to Isolated |
| Networking | VNet integration, private endpoints, access restrictions |
| Deployment Slots | Zero-downtime deployments, staging environments |
| Container Images | Dockerfile, multi-stage builds, best practices |
| AKS | Managed Kubernetes, node pools, autoscaling |
| ACR | Managed container registry, tasks, geo-replication |

---

## Review Questions

1. What is the difference between App Service Plan tiers?
2. How do deployment slots enable zero-downtime deployments?
3. What is VNet Integration and when would you use it?
4. Explain the benefits of using multi-stage Docker builds.
5. How does AKS differ from running Kubernetes yourself?
6. What are ACR Tasks and how do they automate CI/CD?

---

## Additional Resources

- [Azure App Service Documentation](https://docs.microsoft.com/azure/app-service/)
- [Azure Kubernetes Service Documentation](https://docs.microsoft.com/azure/aks/)
- [Azure Container Registry Documentation](https://docs.microsoft.com/azure/container-registry/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)

---

## 5.6 Advanced AKS and Container Patterns (Expert Level)

### AKS Cluster Configuration

```powershell
# Create production-grade AKS cluster
az aks create `
    --resource-group "AKS-RG" `
    --name "prod-aks-cluster" `
    --kubernetes-version 1.28.3 `
    --node-count 3 `
    --node-vm-size "Standard_D4s_v3" `
    --enable-cluster-autoscaler `
    --min-count 3 `
    --max-count 10 `
    --network-plugin azure `
    --network-policy azure `
    --vnet-subnet-id $subnetId `
    --docker-bridge-address 172.17.0.1/16 `
    --dns-service-ip 10.2.0.10 `
    --service-cidr 10.2.0.0/24 `
    --enable-managed-identity `
    --enable-aad `
    --enable-azure-rbac `
    --enable-defender `
    --enable-workload-identity `
    --zones 1 2 3
```

### Kubernetes Deployment Best Practices

```yaml
# production-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
  labels:
    app: web-api
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web-api
  template:
    metadata:
      labels:
        app: web-api
    spec:
      serviceAccountName: web-api-sa
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: web-api
      containers:
        - name: web-api
          image: myacr.azurecr.io/web-api:v1.2.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          env:
            - name: DB_CONNECTION
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: connection-string
          volumeMounts:
            - name: config
              mountPath: /app/config
      volumes:
        - name: config
          configMap:
            name: web-api-config
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-api-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-api
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### Ingress Controller with Azure Application Gateway

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-api-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/ssl-redirect: "true"
    appgw.ingress.kubernetes.io/connection-draining: "true"
    appgw.ingress.kubernetes.io/connection-draining-timeout: "30"
    appgw.ingress.kubernetes.io/waf-policy-for-path: "/subscriptions/.../wafs/mywaf"
spec:
  tls:
    - hosts:
        - api.example.com
      secretName: tls-secret
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /api/v1
            pathType: Prefix
            backend:
              service:
                name: web-api
                port:
                  number: 80
```

### Azure Container Apps (Serverless Containers)

```powershell
# Create Container Apps environment
az containerapp env create `
    --name "prod-env" `
    --resource-group "ContainerApps-RG" `
    --location "eastus" `
    --logs-workspace-id $workspaceId `
    --logs-workspace-key $workspaceKey

# Deploy container app with scaling
az containerapp create `
    --name "web-api" `
    --resource-group "ContainerApps-RG" `
    --environment "prod-env" `
    --image "myacr.azurecr.io/web-api:latest" `
    --target-port 8080 `
    --ingress external `
    --min-replicas 1 `
    --max-replicas 10 `
    --cpu 0.5 `
    --memory 1.0Gi `
    --scale-rule-name "http-rule" `
    --scale-rule-type "http" `
    --scale-rule-http-concurrency 100 `
    --registry-server myacr.azurecr.io `
    --registry-identity system `
    --secrets "db-conn=secretvalue" `
    --env-vars "DB_CONNECTION=secretref:db-conn"

# Add custom scaling rule (KEDA)
az containerapp update `
    --name "web-api" `
    --resource-group "ContainerApps-RG" `
    --scale-rule-name "queue-rule" `
    --scale-rule-type "azure-servicebus" `
    --scale-rule-metadata `
        "queueName=orders" `
        "namespace=mysbns" `
        "messageCount=10" `
    --scale-rule-auth "connection=service-bus-connection"
```

### GitOps with Flux

```powershell
# Enable GitOps extension on AKS
az k8s-extension create `
    --resource-group "AKS-RG" `
    --cluster-name "prod-aks-cluster" `
    --cluster-type managedClusters `
    --name flux `
    --extension-type microsoft.flux

# Create Flux configuration
az k8s-configuration flux create `
    --resource-group "AKS-RG" `
    --cluster-name "prod-aks-cluster" `
    --cluster-type managedClusters `
    --name gitops-config `
    --namespace flux-system `
    --scope cluster `
    --url https://github.com/myorg/k8s-config `
    --branch main `
    --kustomization name=apps path=./apps prune=true
```
