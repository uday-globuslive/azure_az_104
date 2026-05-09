# Azure Glossary for Beginners
## Complete A-Z Reference of Azure Terms

---

# A

### AAD (Azure Active Directory)
See **Azure Active Directory**.

### ACR (Azure Container Registry)
A private registry for storing and managing Docker container images. Like Docker Hub but private and integrated with Azure services.

### Activity Log
A log of all control-plane operations in Azure (who created/modified/deleted what). Retained for 90 days. Found under Monitor > Activity Log.

### ADE (Azure Disk Encryption)
Encrypts OS and data disks on Azure VMs using BitLocker (Windows) or DM-Crypt (Linux). Keys stored in Key Vault.

### Advisor
Azure Advisor - A free service that provides personalized recommendations for cost, security, reliability, operational excellence, and performance.

### AKS (Azure Kubernetes Service)
Managed Kubernetes service. Azure handles the control plane; you manage worker nodes. Used for container orchestration at scale.

### Allocation Method
How an IP address is assigned:
- **Static**: IP remains the same (good for DNS, firewalls)
- **Dynamic**: IP may change on VM restart (cheaper for basic use)

### API Management (APIM)
A gateway that sits in front of your APIs. Provides rate limiting, authentication, caching, and developer portal.

### Application Gateway
Layer 7 (HTTP/HTTPS) load balancer with WAF capabilities. Routes traffic based on URL paths, hostnames.

### Application Insights
Part of Azure Monitor. Provides application performance monitoring (APM) - tracks requests, exceptions, dependencies, performance.

### ARM (Azure Resource Manager)
The deployment and management service for Azure. All operations go through ARM. Also refers to ARM Templates (JSON-based infrastructure as code).

### ASG (Application Security Group)
Group VMs by application (e.g., "WebServers") and use in NSG rules instead of IP addresses. Makes rules easier to manage.

### Availability Set
Logical grouping of VMs in same data center across fault domains and update domains. Provides 99.95% SLA.

### Availability Zone
Physically separate data centers within an Azure region. Each zone has independent power, cooling, networking. Provides 99.99% SLA.

### Azure AD
See **Azure Active Directory**.

### Azure Active Directory
Microsoft's cloud-based identity and access management service. Manages users, groups, and application access. Not the same as Windows Active Directory.

### Azure Arc
Extends Azure management to resources outside Azure (on-premises servers, other clouds). Manage everything from one place.

### Azure Bastion
Secure RDP/SSH access to VMs through the Azure portal. No need for public IP on VMs. Uses SSL over port 443.

### Azure Blob Storage
Object storage for unstructured data (files, images, videos, backups). Access via REST API or SDKs.

### Azure CLI
Command-line tool for managing Azure resources. Cross-platform (Windows, macOS, Linux). Commands start with `az`.

### Azure Cloud Shell
Browser-based command-line experience in Azure Portal. Includes both Bash and PowerShell. Pre-authenticated and pre-configured.

### Azure DevOps
A suite of development tools: Repos (Git), Pipelines (CI/CD), Boards (project management), Test Plans, Artifacts.

### Azure DNS
Azure's domain name hosting service. Manage DNS records for your domains. Uses Azure's global anycast network.

### Azure ExpressRoute
See **ExpressRoute**.

### Azure Files
Fully managed file shares (SMB and NFS). Can mount as network drive on Windows (Z:\) or Linux.

### Azure Firewall
Cloud-native, managed network firewall. Provides threat intelligence, FQDN filtering, network/application rules.

### Azure Functions
Serverless compute service. Run code in response to events (HTTP request, timer, queue message) without managing servers.

### Azure Monitor
Comprehensive monitoring solution. Collects metrics and logs from Azure resources, applications, and infrastructure.

### Azure Policy
Service for creating, assigning, and managing policies. Enforce rules across your Azure environment (e.g., "All VMs must be in East US").

### Azure Portal
The web-based management interface for Azure. Access at portal.azure.com.

### Azure PowerShell
PowerShell module for managing Azure. Commands use `Az` prefix (e.g., `Get-AzVM`, `New-AzResourceGroup`).

### Azure SQL Database
Managed relational database service. Fully managed - no patching, backups handled automatically.

### Azure Stack
Microsoft's solution for running Azure services on-premises. Three flavors: Azure Stack Hub, Azure Stack HCI, Azure Stack Edge.

---

# B

### Backup Vault (Recovery Services Vault)
Container for backup data. Stores backups of VMs, databases, files. Configure backup policies here.

### Bicep
A domain-specific language (DSL) for deploying Azure resources. Cleaner syntax than ARM JSON templates. Compiles to ARM.

### Blob
Binary Large Object. Any file stored in Azure Blob Storage. Three types: Block blob, Append blob, Page blob.

### Blob Tier
Storage tier affecting cost and retrieval time:
- **Hot**: Frequently accessed
- **Cool**: Infrequently accessed (30+ days)
- **Cold**: Rarely accessed (90+ days)  
- **Archive**: Rarely accessed, hours to retrieve

### BYOK (Bring Your Own Key)
Use your own encryption keys in Azure services instead of Microsoft-managed keys. Keys stored in Key Vault.

---

# C

### CDN (Content Delivery Network)
Distributes content to edge locations worldwide. Reduces latency by serving content from servers close to users.

### CIDR (Classless Inter-Domain Routing)
Notation for IP address ranges. Example: 10.0.0.0/16 means first 16 bits are fixed, giving 65,536 addresses.

### CMK (Customer-Managed Key)
Encryption keys that you control and manage in Key Vault. Gives you control over key lifecycle.

### Conditional Access
Azure AD feature for policy-based access control. "If user is on untrusted network, require MFA."

### Contributor
Built-in RBAC role. Full access to resources but cannot grant access to others.

### Cosmos DB
Globally distributed, multi-model database. Supports SQL API, MongoDB API, Cassandra API, and more.

---

# D

### Data Disk
Additional storage attached to a VM (not the OS disk). Used for application data, databases.

### Deployment Slot
App Service feature. Separate instance of your app (e.g., staging) that you can swap with production.

### Diagnostic Settings
Configure where to send platform logs and metrics (Log Analytics, Storage Account, Event Hub).

### DDoS Protection
Protection against Distributed Denial of Service attacks:
- **Basic**: Free, automatic
- **Standard**: Paid, enhanced protection with telemetry

---

# E

### Elastic Pool
Azure SQL feature. Share resources (DTUs or vCores) among multiple databases. Cost-effective for variable workloads.

### Encryption at Rest
Data encrypted when stored. Azure Storage uses 256-bit AES encryption by default.

### Encryption in Transit
Data encrypted while moving over network. Use HTTPS, TLS, encrypted VPN.

### ExpressRoute
Private connection from your network to Azure. Does not go over public internet. Used for high bandwidth, low latency needs.

---

# F

### Fault Domain
Physical grouping in a data center sharing power and network switch. If one fails, others keep running.

### Frontend IP
The IP address where traffic arrives at a load balancer or application gateway.

### Function App
The hosting container for Azure Functions. Can contain multiple functions.

---

# G

### Gateway Subnet
Special subnet for VPN Gateway or ExpressRoute Gateway. Must be named "GatewaySubnet".

### Geo-Redundant Storage (GRS)
Storage replicated to a secondary region. 6 copies total (3 primary + 3 secondary).

### Global Peering
VNet Peering between VNets in different Azure regions. Traffic stays on Microsoft backbone.

---

# H

### Hybrid Cloud
Combination of public cloud (Azure) and private infrastructure (on-premises).

### Hybrid Benefit (Azure Hybrid Benefit)
Use existing Windows Server or SQL Server licenses in Azure. Significant cost savings.

---

# I

### IaaS (Infrastructure as a Service)
Cloud service model where you rent infrastructure (VMs, storage, networks). You manage OS and up.

### Image
Template for creating VMs. Contains OS and optionally applications. Can be Microsoft-provided or custom.

### Inbound Rule
NSG rule controlling traffic coming INTO Azure resources.

---

# J

### JIT (Just-In-Time) Access
Security feature that locks down VM management ports (RDP, SSH) until explicitly requested. Reduces attack surface.

---

# K

### Key Vault
Secure storage for secrets (passwords, connection strings), keys (encryption), and certificates.

### KQL (Kusto Query Language)
Query language used in Log Analytics and Application Insights. Used to query and analyze logs.

---

# L

### Load Balancer
Distributes traffic across multiple backend instances:
- **Basic**: Free, limited features
- **Standard**: Paid, zone-redundant, secure by default

### Log Analytics
Service for collecting, analyzing, and acting on telemetry data. Uses KQL for queries.

### LRS (Locally Redundant Storage)
Three copies of data in ONE data center. Lowest cost, lowest redundancy.

---

# M

### Managed Disk
Azure-managed virtual hard disks. No storage account management needed. Types: Standard HDD, Standard SSD, Premium SSD, Ultra Disk.

### Managed Identity
Azure-managed identity for resources. No credentials to manage. Two types:
- **System-assigned**: Tied to resource lifecycle
- **User-assigned**: Independent, can be shared

### Management Group
Container for organizing subscriptions. Apply policies and RBAC at scale.

### Marketplace
Azure's store for finding and deploying third-party solutions and Microsoft services.

### MFA (Multi-Factor Authentication)
Require additional verification beyond password. "Something you know + something you have."

---

# N

### NIC (Network Interface Card)
Virtual network interface attached to a VM. Contains private IP, optionally public IP, NSG association.

### NSG (Network Security Group)
Firewall for Azure resources. Contains inbound and outbound rules based on source, destination, port, protocol.

---

# O

### OS Disk
The disk containing the VM's operating system. Every VM has one.

### Outbound Rule
NSG rule controlling traffic going OUT FROM Azure resources.

### Owner
Built-in RBAC role. Full access to resources INCLUDING ability to grant access to others.

---

# P

### PaaS (Platform as a Service)
Cloud service model where you only manage applications and data. Provider manages infrastructure.

### Peering
Connection between two VNets. Traffic uses Microsoft backbone, never touches internet.

### PIM (Privileged Identity Management)
Azure AD feature for time-limited, approval-based access to privileged roles. "Just-in-time" admin access.

### Policy
See **Azure Policy**.

### Premium Storage
SSD-backed high-performance storage for VMs. Required for some VM sizes.

### Private Endpoint
Private IP address for an Azure PaaS service in your VNet. Traffic never goes to internet.

### Private Link
Service enabling Private Endpoints. Brings Azure services into your VNet.

### Public IP
IP address accessible from internet. Can be static or dynamic.

---

# Q

### Queue Storage
Azure Storage service for storing messages. Used for async communication between applications.

---

# R

### RA-GRS (Read-Access Geo-Redundant Storage)
GRS plus read access to data in secondary region before failover.

### RBAC (Role-Based Access Control)
Permissions system. Assign roles to users/groups at specific scopes.

### Reader
Built-in RBAC role. View-only access to resources.

### Recovery Point
A snapshot of backup data at a specific point in time.

### Recovery Services Vault
Container for backup and site recovery data.

### Region
Geographical area containing Azure data centers. Example: "East US", "West Europe".

### Region Pair
Two regions paired for disaster recovery. Data is replicated between them.

### Reserved Instance
Commit to 1 or 3 years of VM usage for 40-72% discount.

### Resource
Any manageable item in Azure (VM, storage account, database, etc.).

### Resource Group
Logical container for Azure resources. Used for organization and lifecycle management.

### Resource Provider
Service that supplies Azure resources. Example: Microsoft.Compute (VMs), Microsoft.Storage (storage).

### RTO (Recovery Time Objective)
Maximum acceptable time to restore service after disaster.

### RPO (Recovery Point Objective)
Maximum acceptable amount of data loss measured in time.

---

# S

### SaaS (Software as a Service)
Cloud service model where provider manages everything. You just use the software. Example: Microsoft 365.

### SAS (Shared Access Signature)
Token providing limited access to storage resources. Includes permissions and expiration time.

### Scale Out
Add more instances (horizontal scaling). Example: Go from 2 VMs to 10 VMs.

### Scale Up
Move to larger instance (vertical scaling). Example: Go from 2 CPU to 8 CPU VM.

### Service Endpoint
Optimized route from VNet to Azure PaaS services. Service still has public IP but traffic stays on Azure backbone.

### Service Principal
Identity for applications, services, and automation tools. Like a "user" for non-humans.

### SKU (Stock Keeping Unit)
Defines the tier, size, or version of an Azure resource. Example: Standard_D2s_v3.

### Slot
See **Deployment Slot**.

### Snapshot
Point-in-time copy of a managed disk.

### Spot VM
VMs using spare Azure capacity at up to 90% discount. Can be evicted when Azure needs capacity back.

### SSD
Solid State Drive. Faster than HDD. Premium SSD required for some VM sizes.

### Storage Account
Container for Azure Storage services (Blob, File, Queue, Table).

### Subnet
Subdivision of a VNet address space. Resources in a subnet can have NSGs applied.

### Subscription
Billing and access boundary. All resources exist within a subscription.

---

# T

### Table Storage
NoSQL key-value store for structured data. Simple, cheap, scalable.

### Tag
Name-value pair for organizing resources. Example: Environment=Production, CostCenter=IT.

### Temp Disk
Temporary storage on a VM. Data lost on restart. Used for swap files, caches.

### Tenant
An organization's dedicated instance of Azure AD. Identified by a unique domain.

### Throughput Unit
Measure of Event Hub capacity.

### Tier
Pricing/feature level of a service. Example: Basic, Standard, Premium.

### Traffic Manager
DNS-based traffic load balancer. Routes users to closest or healthiest endpoint.

---

# U

### UDR (User-Defined Route)
Custom routes in a route table. Override Azure's default routing.

### Ultra Disk
Highest performance Azure disk. Sub-millisecond latency. For data-intensive workloads.

### Update Domain
Grouping of VMs that can be updated together during maintenance. Max 20.

---

# V

### VHD (Virtual Hard Disk)
File format for virtual machine disks. Azure managed disks abstract this away.

### Virtual Network
See **VNet**.

### VM (Virtual Machine)
A compute resource that acts as a physical server. You choose OS, size, and storage.

### VMSS (Virtual Machine Scale Sets)
Group of identical VMs that can automatically scale based on demand.

### VNet (Virtual Network)
Your private network in Azure. Contains subnets, IP addresses, routes, security.

### VNet Peering
See **Peering**.

### VPN Gateway
Connects your VNet to other networks (on-premises, other VNets) over encrypted VPN.

---

# W

### WAF (Web Application Firewall)
Protects web applications from common attacks (SQL injection, XSS). Part of Application Gateway or Front Door.

### Well-Architected Framework
Microsoft's guidance for building quality workloads. Five pillars: Reliability, Security, Cost, Operations, Performance.

---

# Z

### Zone
See **Availability Zone**.

### ZRS (Zone-Redundant Storage)
Three copies of data across Availability Zones in one region. Better protection than LRS.

---

# Common Abbreviations Quick Reference

| Abbreviation | Full Term |
|-------------|-----------|
| AAD | Azure Active Directory |
| ACR | Azure Container Registry |
| AKS | Azure Kubernetes Service |
| APIM | API Management |
| ARM | Azure Resource Manager |
| ASG | Application Security Group |
| CDN | Content Delivery Network |
| CIDR | Classless Inter-Domain Routing |
| CMK | Customer-Managed Key |
| DDoS | Distributed Denial of Service |
| DNS | Domain Name System |
| DTU | Database Transaction Unit |
| GRS | Geo-Redundant Storage |
| IaaS | Infrastructure as a Service |
| JIT | Just-In-Time |
| KQL | Kusto Query Language |
| LRS | Locally Redundant Storage |
| MFA | Multi-Factor Authentication |
| NIC | Network Interface Card |
| NSG | Network Security Group |
| PaaS | Platform as a Service |
| PIM | Privileged Identity Management |
| RBAC | Role-Based Access Control |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SaaS | Software as a Service |
| SAS | Shared Access Signature |
| SKU | Stock Keeping Unit |
| SLA | Service Level Agreement |
| UDR | User-Defined Route |
| VHD | Virtual Hard Disk |
| VM | Virtual Machine |
| VMSS | Virtual Machine Scale Sets |
| VNet | Virtual Network |
| VPN | Virtual Private Network |
| WAF | Web Application Firewall |
| ZRS | Zone-Redundant Storage |

---

*Use this glossary as a quick reference while studying. If you encounter a term not listed here, check Microsoft's official documentation at docs.microsoft.com.*
