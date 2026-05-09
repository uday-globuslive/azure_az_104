# Azure Interview Preparation Guide
## For Azure Administrator and Azure Architect Roles

---

# PART 1: UNDERSTANDING THE ROLES

## Azure Administrator vs Azure Architect

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ROLE COMPARISON                                           │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ AZURE ADMINISTRATOR                                                    │ │
│  │ ─────────────────────                                                  │ │
│  │ Focus: Day-to-day operations and management                           │ │
│  │                                                                        │ │
│  │ Responsibilities:                                                      │ │
│  │ • Manage Azure resources (VMs, storage, networking)                   │ │
│  │ • Implement security and compliance                                   │ │
│  │ • Monitor and troubleshoot issues                                     │ │
│  │ • Configure backup and disaster recovery                              │ │
│  │ • Manage costs and optimize resources                                 │ │
│  │ • Handle access management (RBAC, Azure AD)                          │ │
│  │                                                                        │ │
│  │ Certifications: AZ-104 (Azure Administrator)                          │ │
│  │ Salary Range: $80,000 - $140,000 USD                                  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │ AZURE ARCHITECT                                                        │ │
│  │ ───────────────                                                        │ │
│  │ Focus: Design, strategy, and technical leadership                     │ │
│  │                                                                        │ │
│  │ Responsibilities:                                                      │ │
│  │ • Design cloud solutions and architectures                            │ │
│  │ • Define best practices and standards                                 │ │
│  │ • Lead migration and modernization projects                           │ │
│  │ • Evaluate and select appropriate Azure services                      │ │
│  │ • Ensure solutions meet security, performance, cost requirements      │ │
│  │ • Mentor teams and provide technical guidance                         │ │
│  │                                                                        │ │
│  │ Certifications: AZ-305 (Azure Solutions Architect Expert)             │ │
│  │ Salary Range: $120,000 - $200,000+ USD                                │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 2: FUNDAMENTAL INTERVIEW QUESTIONS

## Cloud Computing Basics

### Q1: What is Cloud Computing? Explain in simple terms.
**Answer:**
> Cloud computing is using computing resources (servers, storage, databases, networking, software) over the internet instead of owning and managing physical hardware. It's like renting vs buying - you pay for what you use, can scale up or down instantly, and the provider handles maintenance.

**Key Points to Mention:**
- On-demand self-service
- Broad network access
- Resource pooling
- Rapid elasticity
- Measured service (pay-per-use)

---

### Q2: Explain IaaS, PaaS, and SaaS with Azure examples.
**Answer:**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                              │
│  IaaS (Infrastructure as a Service)                                         │
│  ───────────────────────────────────                                        │
│  You manage: Applications, Data, Runtime, Middleware, OS                    │
│  Provider manages: Virtualization, Servers, Storage, Networking            │
│  Azure Examples: Virtual Machines, Virtual Networks, Azure Storage         │
│  Use Case: "We have a legacy app that needs specific OS configuration"     │
│                                                                              │
│  PaaS (Platform as a Service)                                               │
│  ───────────────────────────────                                            │
│  You manage: Applications, Data                                             │
│  Provider manages: Everything else                                          │
│  Azure Examples: App Service, Azure SQL Database, Azure Functions          │
│  Use Case: "We want to focus on code, not server management"               │
│                                                                              │
│  SaaS (Software as a Service)                                               │
│  ─────────────────────────────                                              │
│  You manage: Just use the software                                          │
│  Provider manages: Everything                                               │
│  Azure Examples: Microsoft 365, Dynamics 365, Power BI                      │
│  Use Case: "We need email and office tools without any IT overhead"        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q3: What is the difference between Public, Private, and Hybrid cloud?
**Answer:**

| Type | Description | Best For | Azure Example |
|------|-------------|----------|---------------|
| **Public Cloud** | Shared infrastructure, managed by provider | Cost-effective, scalable workloads | Standard Azure subscriptions |
| **Private Cloud** | Dedicated to single organization | Strict compliance, data sovereignty | Azure Government, Azure Stack |
| **Hybrid Cloud** | Combination of on-premises and public cloud | Gradual migration, compliance needs | Azure Arc, Azure Stack HCI |

**When to recommend each:**
- Public: Startups, variable workloads, no strict compliance
- Private: Government, healthcare with strict data requirements
- Hybrid: Large enterprises transitioning to cloud, legacy system integration

---

## Azure Fundamentals

### Q4: Explain Azure Regions, Availability Zones, and Availability Sets.
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       AZURE HIGH AVAILABILITY                                │
│                                                                              │
│  REGION                                                                      │
│  ──────                                                                      │
│  • Geographical area containing one or more data centers                    │
│  • 60+ regions worldwide                                                     │
│  • Choose based on: Data residency, latency, service availability          │
│  • Example: "East US", "West Europe"                                        │
│                                                                              │
│  AVAILABILITY ZONE (Within a Region)                                        │
│  ───────────────────────────────────                                        │
│  • Physically separate data centers within a region                        │
│  • Independent power, cooling, networking                                   │
│  • Minimum 3 zones per region (where available)                            │
│  • SLA: 99.99% for VMs spread across zones                                 │
│  • Protection against: Data center failures                                 │
│                                                                              │
│  AVAILABILITY SET (Within a Data Center)                                    │
│  ──────────────────────────────────────                                     │
│  • Logical grouping of VMs within single data center                       │
│  • Fault Domains: Different physical racks (max 3)                         │
│  • Update Domains: Different update groups (max 20)                        │
│  • SLA: 99.95% for 2+ VMs                                                  │
│  • Protection against: Hardware failures, maintenance updates              │
│                                                                              │
│  WHEN TO USE WHAT:                                                          │
│  ─────────────────                                                          │
│  • Need highest availability? → Availability Zones                         │
│  • Region doesn't support zones? → Availability Sets                       │
│  • Global app? → Deploy across multiple regions                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q5: What is Azure Resource Manager (ARM)?
**Answer:**
> Azure Resource Manager is the deployment and management layer for Azure. Every operation in Azure goes through ARM - whether you use Portal, CLI, PowerShell, or REST API. ARM handles authentication, authorization, and routing requests to the appropriate resource provider.

**Key Benefits:**
1. **Declarative templates** - Define infrastructure as code (ARM/Bicep templates)
2. **Dependency management** - Deploy resources in correct order
3. **RBAC integration** - Unified access control
4. **Tagging** - Organize and track resources
5. **Idempotent deployments** - Run same template multiple times safely

---

### Q6: Explain the Azure hierarchy: Management Groups, Subscriptions, Resource Groups, Resources
**Answer:**

```
                    ┌─────────────────────────┐
                    │   Azure AD Tenant       │
                    │   (Your Organization)   │
                    └───────────┬─────────────┘
                                │
                    ┌───────────▼─────────────┐
                    │   Root Management Group │ ← Policies applied here affect ALL
                    └───────────┬─────────────┘
              ┌─────────────────┼─────────────────┐
              │                 │                 │
    ┌─────────▼───────┐ ┌──────▼──────┐ ┌───────▼────────┐
    │ Prod MG         │ │  Dev MG     │ │   Sandbox MG   │
    └────────┬────────┘ └──────┬──────┘ └───────┬────────┘
             │                 │                 │
    ┌────────▼────────┐ ┌──────▼──────┐ ┌───────▼────────┐
    │ Subscription:   │ │Subscription:│ │ Subscription:  │
    │ Production      │ │Development  │ │ Sandbox        │
    │ (Billing unit)  │ │             │ │                │
    └────────┬────────┘ └──────┬──────┘ └───────┬────────┘
             │                 │                 │
    ┌────────▼────────┐        │                 │
    │ Resource Groups │        │                 │
    │ ├── WebApp-RG   │        │                 │
    │ ├── Database-RG │        │                 │
    │ └── Network-RG  │        │                 │
    └────────┬────────┘        │                 │
             │                 │                 │
    ┌────────▼────────┐        │                 │
    │ Resources       │        │                 │
    │ ├── VM          │        │                 │
    │ ├── Storage     │        │                 │
    │ └── VNet        │        │                 │
    └─────────────────┘        │                 │

```

**Key Points:**
- **Management Groups**: Organize subscriptions, apply policies at scale
- **Subscriptions**: Billing boundary, access boundary
- **Resource Groups**: Logical containers, lifecycle management
- **Resources**: Individual Azure services

---

# PART 3: IDENTITY AND ACCESS MANAGEMENT

### Q7: What is Azure Active Directory? How is it different from Windows AD?
**Answer:**

| Aspect | Windows AD (On-Premises) | Azure AD (Cloud) |
|--------|-------------------------|------------------|
| **Location** | On-premises servers | Microsoft cloud |
| **Protocol** | Kerberos, NTLM, LDAP | OAuth 2.0, SAML, OpenID Connect |
| **Purpose** | Domain-joined computers | Cloud apps, web apps, mobile |
| **Structure** | Hierarchical (OU, Domains, Forest) | Flat structure |
| **Group Policy** | Yes (GPO) | No (use Conditional Access, Intune) |
| **Device Management** | Domain join | Azure AD Join, Intune |

**Key Point:** They are NOT the same thing. Azure AD is designed for cloud identity, while Windows AD is for on-premises domain services. They can work together using Azure AD Connect for hybrid identity.

---

### Q8: Explain RBAC in Azure. What are the key components?
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    RBAC (Role-Based Access Control)                          │
│                                                                              │
│  PURPOSE: Control who can do what on which resources                        │
│                                                                              │
│  THREE COMPONENTS:                                                           │
│  ─────────────────                                                          │
│                                                                              │
│  1. SECURITY PRINCIPAL (WHO)                                                │
│     ├── User (john@contoso.com)                                             │
│     ├── Group (Developers group)                                            │
│     ├── Service Principal (application identity)                           │
│     └── Managed Identity (Azure-managed for services)                      │
│                                                                              │
│  2. ROLE DEFINITION (WHAT) - Defines permissions                           │
│     Built-in:                                                                │
│     ├── Owner: Full access + can assign roles                              │
│     ├── Contributor: Full access, can't assign roles                       │
│     ├── Reader: View only                                                   │
│     └── Many specialized roles (VM Contributor, Storage Blob Data Reader)  │
│                                                                              │
│  3. SCOPE (WHERE) - Where the role applies                                  │
│     ├── Management Group                                                     │
│     ├── Subscription                                                         │
│     ├── Resource Group                                                       │
│     └── Resource                                                             │
│                                                                              │
│  ROLE ASSIGNMENT = Security Principal + Role + Scope                        │
│                                                                              │
│  INHERITANCE: Roles assigned at higher scope inherit downward              │
│  Example: Reader at Subscription → Reader for all RGs → all Resources      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Interview Follow-up:** "Give me an example of an RBAC assignment."

> "To allow a developer to manage only VMs in the Development resource group, I would assign the 'Virtual Machine Contributor' role to their user account, scoped to the 'Development-RG' resource group. This follows the principle of least privilege."

---

### Q9: What is Conditional Access? Give examples of policies.
**Answer:**
> Conditional Access is Azure AD's policy engine that makes access decisions based on signals like user, device, location, and risk level.

**Common Policies:**
1. **MFA for all users** - Require multi-factor authentication for everyone
2. **Block risky sign-ins** - Block if Azure AD detects anomalous behavior
3. **Compliant device required** - Only allow Intune-managed devices
4. **Location-based** - Block access from certain countries
5. **Application-specific** - Require MFA only for sensitive apps

**Example Policy:**
> "Require MFA for all users accessing Office 365, except when on the corporate network, using a compliant device."

---

### Q10: What is Azure AD Connect? Explain sync options.
**Answer:**
> Azure AD Connect synchronizes on-premises Active Directory identities to Azure AD, enabling hybrid identity scenarios.

**Sync Options:**

| Option | Description | Best For |
|--------|-------------|----------|
| **Password Hash Sync (PHS)** | Syncs password hashes to Azure AD | Simple setup, cloud authentication |
| **Pass-through Authentication (PTA)** | Authentication happens on-premises | Organizations that can't store passwords in cloud |
| **Federation (ADFS)** | On-premises ADFS handles auth | Complex requirements, existing ADFS investment |

**Best Practice:** Microsoft recommends Password Hash Sync for most organizations - it's simple, reliable, and enables features like leaked credential detection.

---

# PART 4: NETWORKING QUESTIONS

### Q11: Design a network architecture for a 3-tier web application.
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     3-TIER APPLICATION ARCHITECTURE                          │
│                                                                              │
│  VNET: 10.0.0.0/16                                                          │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ WEB TIER (Public-facing) - Subnet: 10.0.1.0/24                      │    │
│  │ ├── Application Gateway (WAF enabled)                                │    │
│  │ │   └── Public IP, SSL termination                                  │    │
│  │ ├── VM Scale Set (Web servers)                                       │    │
│  │ │   └── Auto-scale 2-20 instances based on CPU                      │    │
│  │ └── NSG Rules:                                                       │    │
│  │     ├── Allow 443 from Internet                                      │    │
│  │     ├── Allow 80 from Internet (redirect to 443)                    │    │
│  │     └── Deny all other inbound                                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                           │                                                  │
│                           ▼ (Private communication only)                     │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ APP TIER (API/Business Logic) - Subnet: 10.0.2.0/24                 │    │
│  │ ├── Internal Load Balancer                                          │    │
│  │ ├── VM Scale Set (API servers)                                       │    │
│  │ └── NSG Rules:                                                       │    │
│  │     ├── Allow from Web Subnet (10.0.1.0/24) on port 8080            │    │
│  │     └── Deny Internet inbound                                        │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                           │                                                  │
│                           ▼                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │ DATA TIER (Database) - Subnet: 10.0.3.0/24                          │    │
│  │ ├── Azure SQL with Private Endpoint (NO public IP)                  │    │
│  │ ├── Azure Cache for Redis (Private Endpoint)                        │    │
│  │ └── NSG Rules:                                                       │    │
│  │     ├── Allow from App Subnet (10.0.2.0/24) on SQL port (1433)      │    │
│  │     └── Deny all Internet traffic                                    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                              │
│  ADDITIONAL COMPONENTS:                                                      │
│  ├── Azure Bastion Subnet (10.0.4.0/27) - Secure admin access              │
│  ├── Azure Firewall Subnet (10.0.5.0/26) - Outbound traffic inspection     │
│  └── GatewaySubnet (10.0.255.0/27) - VPN for on-premises connectivity     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Key Design Principles:**
1. **Network segmentation** - Each tier in separate subnet
2. **Defense in depth** - NSGs at subnet level, WAF for web tier
3. **Private connectivity** - Database has no public IP
4. **Scalability** - VMSS for web and app tiers
5. **Security** - Bastion for admin access, no direct RDP/SSH

---

### Q12: Explain VNet Peering vs VPN Gateway vs ExpressRoute.
**Answer:**

| Feature | VNet Peering | VPN Gateway | ExpressRoute |
|---------|-------------|-------------|--------------|
| **What it connects** | VNet to VNet | VNet to On-prem (or VNet-to-VNet) | VNet to On-prem |
| **Connection Type** | Private (Azure backbone) | Encrypted over Internet | Private dedicated line |
| **Bandwidth** | Very high (depends on VM size) | Up to 10 Gbps (VPN) | Up to 100 Gbps |
| **Latency** | Lowest (same region) | Higher (internet variability) | Low and consistent |
| **Cost** | Low (data transfer charges) | Gateway cost + data | Expensive (circuit + port) |
| **Use Case** | Connect Azure VNets | Basic hybrid connectivity | Enterprise-grade hybrid |

**When to use what:**
- **VNet Peering**: Connecting workloads across VNets in Azure
- **VPN Gateway**: Small/medium hybrid scenarios, budget-conscious
- **ExpressRoute**: Large enterprise, compliance requirements, consistent low latency

---

### Q13: What is a Network Security Group (NSG)? How do rules work?
**Answer:**
> An NSG is a firewall that filters traffic to/from Azure resources. It contains security rules that allow or deny traffic based on source, destination, port, and protocol.

**Rule Processing:**
1. Rules processed by priority (100 = highest, 4096 = lowest)
2. Once a match is found, processing stops
3. Default rules cannot be deleted but can be overridden

```
Example NSG Rules (Web Server):

Priority | Name             | Direction | Source      | Port | Action
---------|------------------|-----------|-------------|------|-------
100      | Allow-HTTP       | Inbound   | Internet    | 80   | Allow
110      | Allow-HTTPS      | Inbound   | Internet    | 443  | Allow
120      | Allow-SSH-Admin  | Inbound   | 10.0.0.5/32 | 22   | Allow
65500    | DenyAllInbound   | Inbound   | *           | *    | Deny (default)
```

**Best Practices:**
- Use Service Tags instead of IPs where possible
- Apply NSGs at subnet level for consistency
- Log NSG flow logs for troubleshooting
- Review and audit rules regularly

---

### Q14: How does Azure Load Balancer differ from Application Gateway?
**Answer:**

| Feature | Azure Load Balancer | Application Gateway |
|---------|-------------------|---------------------|
| **Layer** | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| **Routing** | IP + Port only | URL path, hostname, headers |
| **SSL Termination** | No | Yes |
| **WAF** | No | Yes (optional) |
| **Session Affinity** | Source IP | Cookie-based |
| **Health Probes** | TCP, HTTP | HTTP, HTTPS |
| **Use Case** | Non-HTTP workloads, internal LB | Web applications, API routing |
| **Cost** | Lower | Higher |

**Decision Guide:**
- HTTP/HTTPS traffic? → Application Gateway
- Need WAF protection? → Application Gateway
- Non-HTTP (database, custom protocol)? → Load Balancer
- Internal load balancing? → Either (LB is simpler/cheaper)

---

# PART 5: COMPUTE QUESTIONS

### Q15: When would you use VMs vs App Service vs Containers vs Functions?
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     COMPUTE DECISION GUIDE                                   │
│                                                                              │
│  VIRTUAL MACHINES (IaaS)                                                     │
│  ────────────────────────                                                   │
│  Choose when:                                                                │
│  ├── Need full control over OS                                              │
│  ├── Running legacy applications                                             │
│  ├── Specific software or driver requirements                               │
│  ├── Lift-and-shift migration                                               │
│  └── Custom network configurations                                          │
│                                                                              │
│  APP SERVICE (PaaS)                                                          │
│  ─────────────────                                                          │
│  Choose when:                                                                │
│  ├── Web apps and APIs (.NET, Node.js, Python, PHP, Java)                  │
│  ├── Want managed platform (no OS management)                              │
│  ├── Need deployment slots for staging                                      │
│  ├── Built-in CI/CD integration needed                                      │
│  └── Auto-scaling without managing infrastructure                          │
│                                                                              │
│  CONTAINERS (AKS/ACI)                                                        │
│  ────────────────────                                                       │
│  Choose when:                                                                │
│  ├── Microservices architecture                                             │
│  ├── Need portability across environments                                   │
│  ├── High-density workloads (many apps on less infrastructure)            │
│  ├── Complex orchestration requirements (use AKS)                          │
│  └── Rapid scaling with consistent environments                            │
│                                                                              │
│  AZURE FUNCTIONS (Serverless)                                               │
│  ───────────────────────────                                                │
│  Choose when:                                                                │
│  ├── Event-driven, short-running code                                       │
│  ├── Pay only when code runs (consumption plan)                            │
│  ├── Triggers: HTTP, timer, queue, blob, etc.                              │
│  ├── Glue code between services                                            │
│  └── Background processing jobs                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q16: Explain VM Scale Sets and when to use them.
**Answer:**
> VM Scale Sets allow you to deploy and manage a group of identical VMs that can automatically increase or decrease based on demand or schedule.

**Key Features:**
- **Auto-scale**: Based on metrics (CPU, memory, custom) or schedule
- **High availability**: Automatically spread across Fault Domains and Availability Zones
- **Load balancer integration**: Works with Azure LB or Application Gateway
- **Rolling upgrades**: Update VMs without downtime

**Use Cases:**
1. **Web servers** - Scale based on traffic
2. **Batch processing** - Scale up for jobs, scale down after
3. **Big data** - Spark/Hadoop clusters

**Sample Autoscale Rule:**
```
Trigger: Average CPU > 70% for 5 minutes
Action: Add 2 VM instances
Cooldown: 5 minutes

Trigger: Average CPU < 30% for 10 minutes
Action: Remove 1 VM instance
```

---

### Q17: How do you secure a Virtual Machine in Azure?
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VM SECURITY BEST PRACTICES                              │
│                                                                              │
│  1. NETWORK SECURITY                                                         │
│     ├── Put VMs in private subnets (no public IPs where possible)           │
│     ├── Use NSGs with least-privilege rules                                 │
│     ├── Use Azure Bastion for admin access (not public RDP/SSH)            │
│     └── Enable Just-In-Time VM access (locks down management ports)        │
│                                                                              │
│  2. IDENTITY & ACCESS                                                        │
│     ├── Use Azure AD authentication for Linux VMs (not just passwords)     │
│     ├── Use strong passwords or SSH keys                                    │
│     ├── Implement RBAC (don't give everyone Owner access)                  │
│     └── Use Managed Identities for VM-to-Azure service access              │
│                                                                              │
│  3. UPDATES & PATCHES                                                        │
│     ├── Enable Azure Update Manager                                         │
│     ├── Schedule maintenance windows                                         │
│     └── Use custom images with pre-installed updates                        │
│                                                                              │
│  4. DISK ENCRYPTION                                                          │
│     ├── Enable Azure Disk Encryption (ADE)                                  │
│     ├── Use Customer-Managed Keys (CMK) for sensitive data                 │
│     └── Store keys in Azure Key Vault                                       │
│                                                                              │
│  5. MONITORING & DETECTION                                                   │
│     ├── Enable Microsoft Defender for Cloud                                 │
│     ├── Enable boot diagnostics                                              │
│     ├── Configure Log Analytics agent                                        │
│     └── Set up alerts for suspicious activity                               │
│                                                                              │
│  6. BACKUP & RECOVERY                                                        │
│     ├── Enable Azure Backup                                                  │
│     ├── Test restores regularly                                             │
│     └── Consider cross-region backup for DR                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 6: STORAGE QUESTIONS

### Q18: Explain Azure Storage redundancy options. When would you use each?
**Answer:**

| Option | Copies | Protection Level | Best For |
|--------|--------|-----------------|----------|
| **LRS** | 3 in one data center | Hardware failure | Dev/test, non-critical |
| **ZRS** | 3 across zones | Zone failure | Production, single-region |
| **GRS** | 6 (3+3 across regions) | Regional disaster | DR requirements |
| **RA-GRS** | GRS + read access to secondary | Regional disaster + read availability | Read-heavy DR scenarios |
| **GZRS** | ZRS + GRS combined | Zone + Regional | Mission-critical |
| **RA-GZRS** | Highest protection | Zone + Regional + Read | Most critical data |

**Quick Decision:**
- Development: LRS (cheapest)
- Production single-region: ZRS
- Production with DR: GRS or GZRS
- Need to read during outage: RA-GRS or RA-GZRS

---

### Q19: What is the difference between Blob Storage access tiers?
**Answer:**

| Tier | Storage Cost | Access Cost | Retrieval Time | Minimum Retention |
|------|-------------|-------------|----------------|-------------------|
| **Hot** | Highest | Lowest | Immediate | None |
| **Cool** | ~50% less | Higher | Immediate | 30 days |
| **Cold** | ~70% less | Even higher | Immediate | 90 days |
| **Archive** | ~90% less | Highest | Hours (rehydration) | 180 days |

**Best Practices:**
- **Hot**: Frequently accessed data (active documents, images)
- **Cool**: Infrequently accessed, store 30+ days (monthly backups)
- **Cold**: Rarely accessed, store 90+ days (quarterly reports)
- **Archive**: Rarely accessed, acceptable retrieval delay (compliance archives, old logs)

**Lifecycle Management**: Automatically transition blobs between tiers based on age:
```
Day 0-30: Hot
Day 31-90: Cool
Day 91-365: Cold
After 365 days: Archive
```

---

### Q20: What is a Shared Access Signature (SAS)? When should you use it?
**Answer:**
> A SAS is a URI that provides limited access to Azure Storage resources without exposing the account key. It includes permissions, validity period, and a signature.

**Types of SAS:**
1. **User Delegation SAS** - Signed with Azure AD credentials (most secure)
2. **Service SAS** - Access to specific service (Blob, File, Queue, Table)
3. **Account SAS** - Access to multiple services

**When to use:**
- Third-party application needs temporary access to storage
- External partner needs to upload files
- Generating download links for users
- Any scenario where you don't want to share account keys

**Security Best Practices:**
- Use shortest validity period possible
- Use HTTPS only
- Use Stored Access Policies (can revoke access)
- Use User Delegation SAS when possible
- Don't include SAS in logs

---

# PART 7: ARCHITECT-LEVEL QUESTIONS

### Q21: Design a disaster recovery strategy for a critical application.
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│               DISASTER RECOVERY ARCHITECTURE                                 │
│                                                                              │
│  PRIMARY REGION (East US)              SECONDARY REGION (West US)           │
│  ─────────────────────────             ──────────────────────────           │
│                                                                              │
│  ┌─────────────────────┐               ┌─────────────────────┐              │
│  │ Traffic Manager     │←── DNS ────►│ Traffic Manager     │              │
│  │ (Priority routing)  │               │ (Failover target)   │              │
│  └─────────┬───────────┘               └─────────┬───────────┘              │
│            │                                     │                          │
│  ┌─────────▼───────────┐               ┌─────────▼───────────┐              │
│  │ App Gateway + WAF   │               │ App Gateway + WAF   │              │
│  │ (Active)            │               │ (Standby/Warm)      │              │
│  └─────────┬───────────┘               └─────────┬───────────┘              │
│            │                                     │                          │
│  ┌─────────▼───────────┐               ┌─────────▼───────────┐              │
│  │ VM Scale Set        │               │ VM Scale Set        │              │
│  │ (Capacity: 10)      │     ←ASR→    │ (Capacity: 2)       │              │
│  └─────────┬───────────┘   Replication └─────────┬───────────┘              │
│            │                                     │                          │
│  ┌─────────▼───────────┐               ┌─────────▼───────────┐              │
│  │ Azure SQL (Primary) │←─ Active ──►│ Azure SQL (Secondary)│              │
│  │                     │   Geo-Rep    │ (Read replica)       │              │
│  └─────────────────────┘               └─────────────────────┘              │
│                                                                              │
│  ┌─────────────────────┐               ┌─────────────────────┐              │
│  │ Storage (GRS)       │←─ Async ──►│ Storage (Secondary)  │              │
│  │ mystorageea.blob... │   Replication│ mystorageea-secondary│              │
│  └─────────────────────┘               └─────────────────────┘              │
│                                                                              │
│  KEY METRICS:                                                                │
│  ├── RTO (Recovery Time Objective): 1 hour                                  │
│  ├── RPO (Recovery Point Objective): 15 minutes                            │
│  └── Annual DR test scheduled: Quarterly                                    │
│                                                                              │
│  COMPONENTS:                                                                 │
│  ├── Azure Site Recovery (ASR): VM replication                             │
│  ├── SQL Active Geo-Replication: Database sync                             │
│  ├── GRS Storage: Blob replication                                          │
│  └── Traffic Manager: DNS-based failover                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**DR Strategies by Cost/Complexity:**

| Strategy | RTO | Cost | Description |
|----------|-----|------|-------------|
| **Backup/Restore** | Hours-Days | Lowest | Restore from backups |
| **Pilot Light** | Minutes-Hours | Low | Minimal standby, scale up on failover |
| **Warm Standby** | Minutes | Medium | Reduced-scale running in secondary |
| **Hot Standby** | Seconds-Minutes | Highest | Full-scale running, active-active |

---

### Q22: How would you optimize Azure costs for a company spending too much?
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    COST OPTIMIZATION FRAMEWORK                               │
│                                                                              │
│  1. VISIBILITY (Know where money goes)                                      │
│  ─────────────────────────────────────                                      │
│  ├── Enable Azure Cost Management + Billing                                 │
│  ├── Set up cost alerts and budgets                                        │
│  ├── Implement tagging strategy (Owner, CostCenter, Environment)           │
│  └── Review Advisor recommendations weekly                                  │
│                                                                              │
│  2. RIGHT-SIZE RESOURCES                                                     │
│  ───────────────────────                                                    │
│  ├── Analyze VM utilization (Azure Advisor)                                │
│  │   └── 20% CPU average? Downsize to smaller SKU                          │
│  ├── Scale down non-production environments                                │
│  └── Remove unused resources (orphaned disks, IPs)                         │
│                                                                              │
│  3. COMMITMENT DISCOUNTS                                                     │
│  ───────────────────────                                                    │
│  ├── Reserved Instances (1 or 3 year): 40-72% savings                      │
│  │   └── For predictable, always-on workloads                              │
│  ├── Azure Savings Plans: Flexible compute commitment                      │
│  └── Azure Hybrid Benefit: Use existing Windows/SQL licenses              │
│                                                                              │
│  4. AUTO-SCALING & SCHEDULING                                                │
│  ───────────────────────────                                                │
│  ├── Implement auto-scaling for variable workloads                         │
│  ├── Auto-shutdown dev/test VMs (7PM-7AM, weekends)                        │
│  └── Scale down/pause during off-hours                                      │
│                                                                              │
│  5. ARCHITECTURE CHANGES                                                     │
│  ───────────────────────                                                    │
│  ├── Move to PaaS where possible (less overhead, often cheaper)            │
│  ├── Use Spot VMs for fault-tolerant batch workloads (90% savings)        │
│  ├── Implement storage tiering (Hot → Cool → Archive)                      │
│  └── Use serverless for event-driven workloads                             │
│                                                                              │
│  6. GOVERNANCE                                                               │
│  ────────────                                                               │
│  ├── Azure Policy to enforce cost controls                                  │
│  │   └── "Block expensive VM SKUs in dev"                                  │
│  ├── Require tags on all resources                                         │
│  └── Regular monthly cost reviews                                           │
│                                                                              │
│  QUICK WINS:                                                                 │
│  ├── Delete unused public IPs ($4/month each)                              │
│  ├── Delete unattached managed disks                                       │
│  ├── Delete empty resource groups (no cost, but clutter)                   │
│  └── Resize over-provisioned VMs                                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q23: Explain Azure Well-Architected Framework pillars.
**Answer:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│               AZURE WELL-ARCHITECTED FRAMEWORK                               │
│                                                                              │
│  1. RELIABILITY (Does it keep working?)                                      │
│  ──────────────────────────────────────                                     │
│  ├── Design for failure: What if any component fails?                       │
│  ├── Implement redundancy (zones, regions)                                  │
│  ├── Auto-scaling to handle demand                                          │
│  ├── Regular disaster recovery testing                                      │
│  └── Key question: "What's our RTO and RPO?"                               │
│                                                                              │
│  2. SECURITY (Is it protected?)                                              │
│  ──────────────────────────────                                             │
│  ├── Defense in depth (multiple security layers)                           │
│  ├── Identity as primary security perimeter                                │
│  ├── Encrypt data at rest and in transit                                   │
│  ├── Principle of least privilege                                          │
│  └── Key question: "How do we detect and respond to threats?"              │
│                                                                              │
│  3. COST OPTIMIZATION (Is it efficient?)                                     │
│  ──────────────────────────────────────                                     │
│  ├── Right-size resources                                                   │
│  ├── Use reservations for predictable workloads                            │
│  ├── Implement auto-scaling                                                 │
│  ├── Monitor and optimize continuously                                      │
│  └── Key question: "Are we getting value for our spend?"                   │
│                                                                              │
│  4. OPERATIONAL EXCELLENCE (Can we run it well?)                            │
│  ───────────────────────────────────────────────                            │
│  ├── Infrastructure as Code (ARM, Terraform, Bicep)                        │
│  ├── CI/CD pipelines for deployments                                       │
│  ├── Monitoring and alerting                                                │
│  ├── Documented runbooks and procedures                                    │
│  └── Key question: "Can we deploy and operate confidently?"                │
│                                                                              │
│  5. PERFORMANCE EFFICIENCY (Is it fast enough?)                             │
│  ──────────────────────────────────────────────                             │
│  ├── Right compute tier for workload                                        │
│  ├── Caching strategies (Redis, CDN)                                       │
│  ├── Database optimization                                                  │
│  ├── Load testing and benchmarking                                         │
│  └── Key question: "Does it meet our performance SLAs?"                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q24: How would you migrate an on-premises application to Azure?
**Answer:**

**Migration Strategies (The 5 Rs):**

| Strategy | Description | When to Use |
|----------|-------------|-------------|
| **Rehost** (Lift-and-shift) | Move as-is to VMs | Quick migration, minimal changes |
| **Refactor** | Minor changes for PaaS | Modernize incrementally |
| **Rearchitect** | Significant code changes | Cloud-native benefits needed |
| **Rebuild** | Rewrite from scratch | Legacy, tech debt too high |
| **Replace** | Use SaaS instead | Standard functionality available |

**Migration Process:**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     MIGRATION PHASES                                         │
│                                                                              │
│  PHASE 1: ASSESS                                                             │
│  ─────────────────                                                          │
│  ├── Inventory all applications, dependencies                              │
│  ├── Azure Migrate: Discover and assess VMs                                │
│  ├── Identify compatibility issues                                          │
│  ├── Estimate Azure costs                                                   │
│  └── Create migration roadmap (prioritize apps)                            │
│                                                                              │
│  PHASE 2: PLAN                                                               │
│  ─────────────                                                              │
│  ├── Design target Azure architecture                                       │
│  ├── Plan network connectivity (VPN/ExpressRoute)                          │
│  ├── Define governance (subscriptions, RGs, policies)                      │
│  ├── Plan identity (Azure AD Connect)                                      │
│  └── Create detailed migration plan per application                        │
│                                                                              │
│  PHASE 3: MIGRATE                                                            │
│  ────────────────                                                           │
│  ├── Set up target Azure environment                                        │
│  ├── Migrate workloads (Azure Migrate, ASR, DMS)                           │
│  ├── Test thoroughly in Azure                                              │
│  ├── Plan cutover windows                                                  │
│  └── Execute cutover with rollback plan                                    │
│                                                                              │
│  PHASE 4: OPTIMIZE                                                           │
│  ─────────────────                                                          │
│  ├── Right-size resources based on actual usage                            │
│  ├── Implement monitoring and alerts                                       │
│  ├── Enable backups and DR                                                  │
│  ├── Apply security best practices                                         │
│  └── Consider further modernization (PaaS, containers)                     │
│                                                                              │
│  TOOLS:                                                                      │
│  ├── Azure Migrate: Discovery, assessment, migration                        │
│  ├── Azure Site Recovery: VM replication                                    │
│  ├── Azure Database Migration Service: Database migration                  │
│  └── Azure App Service Migration Assistant: Web app migration              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 8: SCENARIO-BASED QUESTIONS

### Q25: Your application is slow. How do you troubleshoot?
**Answer:**

```
TROUBLESHOOTING FLOWCHART:

1. IDENTIFY THE SYMPTOM
   └── Is it specific pages, all pages, or API calls?

2. CHECK AZURE METRICS (Azure Monitor)
   ├── VM CPU/Memory high? → Right-size or scale out
   ├── Database DTU/CPU maxed? → Upgrade tier, optimize queries
   └── Network latency high? → Check region, consider CDN

3. CHECK APPLICATION INSIGHTS
   ├── Slow dependencies identified? → Database, external API
   ├── Exception rate high? → Bug causing retry loops
   └── Which operations are slow? → Focus optimization

4. CHECK NETWORK
   ├── NSG blocking? → Review flow logs
   └── Load balancer health probes failing? → Backend unhealthy

5. COMMON SOLUTIONS
   ├── Add caching (Redis) for frequently accessed data
   ├── Implement CDN for static content
   ├── Optimize database queries (indexes, query tuning)
   ├── Scale out (add more instances)
   └── Scale up (bigger VMs)
```

---

### Q26: How do you handle secrets and sensitive configuration in Azure?
**Answer:**

**Best Practice: Azure Key Vault**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SECRET MANAGEMENT ARCHITECTURE                            │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                      Azure Key Vault                                 │    │
│  │  ├── Secrets: Connection strings, API keys, passwords              │    │
│  │  ├── Keys: Encryption keys (HSM-backed optional)                   │    │
│  │  └── Certificates: SSL certificates                                │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                            ▲                                                 │
│                            │ Managed Identity                                │
│                            │ (no credentials in code)                        │
│  ┌─────────────────────────┴────────────────────────────────────────────┐   │
│  │ App Service / Function / VM                                          │   │
│  │                                                                       │   │
│  │ // Code retrieves secret at runtime                                  │   │
│  │ var secretClient = new SecretClient(vaultUri, new DefaultAzureCredential());│
│  │ var secret = await secretClient.GetSecretAsync("DatabasePassword");  │   │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  NEVER DO:                                                                   │
│  ├── Store secrets in code/config files                                     │
│  ├── Store secrets in environment variables (OK temporarily)               │
│  ├── Log secrets                                                            │
│  └── Share Key Vault access widely                                          │
│                                                                              │
│  ALWAYS DO:                                                                  │
│  ├── Use Managed Identity for Azure-to-Azure auth                          │
│  ├── Enable soft-delete and purge protection                               │
│  ├── Rotate secrets regularly                                               │
│  └── Audit access with Azure Monitor                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Q27: A production VM is not responding. Walk me through your troubleshooting.
**Answer:**

```
PRODUCTION VM TROUBLESHOOTING RUNBOOK:

1. VERIFY THE ISSUE
   ├── Can you ping the VM? (Check NSG rules first)
   ├── Can you connect via Azure Bastion / Serial Console?
   └── Check Azure Service Health for outages

2. CHECK AZURE PORTAL
   ├── VM Status: Is it "Running"?
   ├── Boot Diagnostics: Screenshot shows login screen?
   ├── Metrics: CPU/Memory/Disk IOPS normal?
   └── Activity Log: Any recent changes?

3. USE SERIAL CONSOLE (if RDP/SSH fails)
   ├── Built into Azure Portal
   ├── Works even if networking is broken
   └── Can access Windows SAC or Linux shell

4. CHECK COMMON CAUSES
   ├── Disk Full? → Clear space, resize disk
   ├── Memory exhausted? → Restart, resize VM
   ├── Windows Update reboot pending? → Complete update
   ├── Firewall blocking RDP/SSH? → Use Run Command
   └── NIC disconnected? → Check NIC status

5. MITIGATION OPTIONS
   ├── Restart VM (through portal)
   ├── Redeploy VM (moves to new host)
   ├── Reset RDP/SSH configuration
   └── Restore from backup (last resort)

6. IF VM WON'T START
   ├── Check Activity Log for errors
   ├── Try Redeploy (new hardware)
   ├── Attach OS disk to recovery VM
   └── Contact Azure Support
```

---

# PART 9: BEHAVIORAL QUESTIONS

### Q28: Tell me about a challenging Azure project you worked on.
**Framework for Answer (STAR Method):**

**Example Answer:**
> **Situation**: "Our company needed to migrate a legacy on-premises ERP system to Azure with zero downtime. The system had 50+ servers and a 10TB database."
>
> **Task**: "I was the lead architect responsible for designing the migration strategy and ensuring business continuity."
>
> **Action**: "I proposed a phased migration approach:
> 1. Set up hybrid connectivity using ExpressRoute
> 2. Used Azure Site Recovery to replicate VMs
> 3. Implemented Azure SQL Managed Instance with transactional replication
> 4. Planned cutover during a low-usage weekend with detailed rollback procedures"
>
> **Result**: "We completed the migration with only 2 hours of planned downtime. Performance improved 40% due to right-sizing, and monthly infrastructure costs decreased by 30% using Reserved Instances."

---

### Q29: How do you stay current with Azure updates?
**Good Answer Points:**
- Microsoft Learn and documentation
- Azure updates blog (azure.microsoft.com/updates)
- Azure Friday videos
- Microsoft Ignite and Build conferences
- Hands-on practice in sandbox subscriptions
- Community: Reddit, Twitter, user groups
- Certifications and renewal

---

# PART 10: QUESTIONS TO ASK THE INTERVIEWER

1. "What does your current Azure architecture look like?"
2. "What are the biggest challenges your team faces with Azure?"
3. "How does your team handle change management and deployments?"
4. "What certifications does the team have, and do you support certification?"
5. "What's the ratio of on-premises to cloud workloads?"
6. "How do you approach cloud cost management?"
7. "What does a typical day/week look like in this role?"
8. "What Azure services are you planning to adopt in the next year?"

---

# QUICK REFERENCE: KEY NUMBERS TO KNOW

| Resource | Limit |
|----------|-------|
| Subscriptions per tenant | Unlimited |
| Resource groups per subscription | 980 |
| Resources per resource group | 800 |
| Tags per resource | 50 |
| RBAC assignments per subscription | 2,000 |
| VMs per subscription per region | 25,000 |
| VNets per subscription per region | 1,000 |
| Subnets per VNet | 3,000 |
| NSG rules per NSG | 1,000 |
| Storage accounts per subscription per region | 250 |
| App Service Plans per subscription | 100 |

---

*Good luck with your interview! Remember: It's okay to say "I don't know, but here's how I would find out..." - this shows intellectual honesty and problem-solving skills.*
