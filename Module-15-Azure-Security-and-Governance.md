# Module 15 - Azure Security and Governance (Complete Guide: Beginner to Expert)

## Learning Objectives
By the end of this module, you will be able to:
- Implement Azure security best practices
- Configure network security controls
- Manage identities with Azure AD and RBAC
- Implement Azure Policy for governance
- Use Azure Security Center and Microsoft Defender
- Configure Azure Key Vault for secrets management
- Implement compliance and audit controls
- Design secure architectures for integration solutions

---

## 15.1 Azure Security Overview

### Security Layers (Defense in Depth)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Defense in Depth                                     │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                          Physical Security                             │  │
│  │                     (Azure Data Center Security)                       │  │
│  │  ┌─────────────────────────────────────────────────────────────────┐  │  │
│  │  │                      Identity & Access                           │  │  │
│  │  │               (Azure AD, RBAC, MFA, PIM)                         │  │  │
│  │  │  ┌───────────────────────────────────────────────────────────┐  │  │  │
│  │  │  │                      Perimeter                             │  │  │  │
│  │  │  │          (DDoS, Azure Firewall, WAF)                       │  │  │  │
│  │  │  │  ┌─────────────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │                     Network                          │  │  │  │  │
│  │  │  │  │           (NSG, VNet, Private Endpoints)             │  │  │  │  │
│  │  │  │  │  ┌───────────────────────────────────────────────┐  │  │  │  │  │
│  │  │  │  │  │                   Compute                      │  │  │  │  │  │
│  │  │  │  │  │        (VM Security, Container Security)       │  │  │  │  │  │
│  │  │  │  │  │  ┌─────────────────────────────────────────┐  │  │  │  │  │  │
│  │  │  │  │  │  │              Application                 │  │  │  │  │  │  │
│  │  │  │  │  │  │        (Secure Code, API Security)       │  │  │  │  │  │  │
│  │  │  │  │  │  │  ┌─────────────────────────────────┐    │  │  │  │  │  │  │
│  │  │  │  │  │  │  │             Data                 │    │  │  │  │  │  │  │
│  │  │  │  │  │  │  │  (Encryption, Classification)   │    │  │  │  │  │  │  │
│  │  │  │  │  │  │  └─────────────────────────────────┘    │  │  │  │  │  │  │
│  │  │  │  │  │  └─────────────────────────────────────────┘  │  │  │  │  │  │
│  │  │  │  │  └───────────────────────────────────────────────┘  │  │  │  │  │
│  │  │  │  └─────────────────────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Shared Responsibility Model

| Layer | IaaS | PaaS | SaaS |
|-------|------|------|------|
| **Data** | Customer | Customer | Customer |
| **Applications** | Customer | Customer | Microsoft |
| **Runtime** | Customer | Microsoft | Microsoft |
| **OS** | Customer | Microsoft | Microsoft |
| **VM/Compute** | Customer | Microsoft | Microsoft |
| **Networking** | Microsoft | Microsoft | Microsoft |
| **Storage** | Microsoft | Microsoft | Microsoft |
| **Physical** | Microsoft | Microsoft | Microsoft |

#### 🏢 Real-World Security Use Cases:

**Zero Trust Architecture Implementation:**
```
Company: FinTech Corp (Financial services, PCI-DSS compliance)
Challenge: Implement zero trust - "never trust, always verify"

Zero Trust Layers Implemented:
┌─────────────────────────────────────────────────────────────────┐
│  Layer 1: Identity Verification                                  │
│  ├── Azure AD: All users (employees, vendors, customers)        │
│  ├── MFA: Required for all (no exceptions)                      │
│  ├── Conditional Access: Device compliance + location check     │
│  ├── Identity Protection: Block high-risk sign-ins             │
│  └── PIM: Just-in-time admin access (8h max)                    │
│                                                                  │
│  Layer 2: Device Verification                                    │
│  ├── Intune: Device compliance required                         │
│  ├── Conditional Access: Only compliant devices                 │
│  └── Compliance: BitLocker enabled, antivirus active, patched   │
│                                                                  │
│  Layer 3: Network Microsegmentation                              │
│  ├── VNet: Separate for app tiers                               │
│  ├── NSG: Allow only required traffic                           │
│  ├── Private Endpoints: No public IPs for data                  │
│  └── Azure Firewall: Inspect all egress traffic                │
│                                                                  │
│  Layer 4: Application Security                                   │
│  ├── App Gateway WAF: OWASP protection                         │
│  ├── API Management: Rate limiting, auth policies               │
│  └── Managed Identity: No secrets in code                       │
│                                                                  │
│  Layer 5: Data Protection                                        │
│  ├── Key Vault: All secrets, keys, certificates                │
│  ├── Encryption: At rest (AES-256), in transit (TLS 1.3)       │
│  ├── Azure Information Protection: Document classification     │
│  └── Purview: Data governance and lineage                      │
└─────────────────────────────────────────────────────────────────┘

Verification Flow:
User Login → MFA → Device Check → Location Check → 
App Access → Network Policy → Data Access → Audit Log

Results:
├── Security incidents: 95% reduction
├── Phishing attacks: Zero successful
├── PCI-DSS audit: Passed first attempt
├── Mean time to detect threats: 15 minutes (was 24 hours)
└── Compliance posture: 98% (Defender for Cloud score)
```

**Security Operations Center (SOC) with Sentinel:**
```
Company: GlobalBank (100 branch offices, 20,000 employees)
Challenge: Detect and respond to threats across entire estate

Sentinel SIEM/SOAR Implementation:
┌─────────────────────────────────────────────────────────────────┐
│                    Microsoft Sentinel                            │
│                                                                  │
│  Data Sources Connected:                                         │
│  ├── Azure Activity Logs (all subscriptions)                    │
│  ├── Azure AD Sign-in/Audit Logs                                │
│  ├── Microsoft 365 (email, Teams, SharePoint)                   │
│  ├── Defender for Cloud (security alerts)                       │
│  ├── Azure Firewall Logs                                        │
│  ├── NSG Flow Logs                                              │
│  ├── On-premises Syslog (via Azure Arc)                        │
│  └── Third-party: Palo Alto, CrowdStrike                       │
│                                                                  │
│  Analytics Rules:                                                │
│  ├── Impossible travel: User in NYC then Tokyo in 1 hour       │
│  ├── Brute force: 50 failed logins from same IP                │
│  ├── Data exfiltration: 10GB downloaded from SharePoint        │
│  ├── Privilege escalation: Normal user granted admin           │
│  └── Custom: Specific business logic detections                │
│                                                                  │
│  Automated Response (Playbooks):                                 │
│  ├── Trigger: High severity alert                               │
│  ├── Action 1: Block user in Azure AD                           │
│  ├── Action 2: Isolate device in Intune                         │
│  ├── Action 3: Create incident ticket in ServiceNow            │
│  ├── Action 4: Page on-call security analyst                    │
│  └── Action 5: Collect forensic data to storage                │
│                                                                  │
│  Daily Volume:                                                   │
│  ├── Events ingested: 50 million/day                           │
│  ├── Alerts generated: 500/day                                  │
│  ├── Incidents (grouped): 20/day                                │
│  └── Automated responses: 15/day                               │
└─────────────────────────────────────────────────────────────────┘

SOC Team Workflow:
┌─────────────────────────────────────────────────────────────────┐
│  Alert → Incident → Investigation → Response → Report           │
│                                                                  │
│  Investigation Tools:                                            │
│  ├── Investigation graph (visual entity relationships)         │
│  ├── Hunting queries (proactive threat hunting)                 │
│  ├── Notebooks (Jupyter for deep analysis)                      │
│  └── Entity pages (user/device timeline view)                  │
└─────────────────────────────────────────────────────────────────┘

Results:
├── MTTD (Mean Time to Detect): 15 minutes (was 24 hours)
├── MTTR (Mean Time to Respond): 30 minutes (was 4 hours)
├── Analyst efficiency: +300% (automation handles routine)
├── False positives: 20% (ML tuning ongoing)
└── Cost: $50,000/month (ROI: prevented $2M breach)
```

**Compliance Automation (Healthcare HIPAA):**
```
Company: HealthCare System (50 hospitals, strict HIPAA)
Challenge: Continuous compliance monitoring and enforcement

Azure Policy for HIPAA:
┌─────────────────────────────────────────────────────────────────┐
│  Policy Initiative: HIPAA/HITRUST (built-in + custom)           │
│                                                                  │
│  Encryption Policies:                                            │
│  ├── Storage: Require encryption at rest                        │
│  ├── SQL: Require TDE enabled                                   │
│  ├── VM: Require disk encryption                                │
│  └── Network: Require TLS 1.2 minimum                          │
│                                                                  │
│  Access Control Policies:                                        │
│  ├── No public IPs on VMs                                       │
│  ├── Require private endpoints for PaaS                        │
│  ├── MFA required for all users                                 │
│  └── No RBAC Owner role (must use custom)                      │
│                                                                  │
│  Audit Logging Policies:                                         │
│  ├── Require diagnostic logs on all resources                  │
│  ├── Minimum retention: 90 days                                 │
│  └── Central Log Analytics workspace required                   │
│                                                                  │
│  Effect Configuration:                                           │
│  ├── Production: Deny (block non-compliant)                     │
│  ├── Development: Audit (report but allow)                      │
│  └── Legacy: DeployIfNotExists (auto-remediate)                │
│                                                                  │
│  Compliance Dashboard:                                           │
│  ├── Overall: 94% compliant (improving)                         │
│  ├── By resource type: Storage 100%, VM 89%                     │
│  └── Non-compliant: 47 resources (remediation in progress)     │
└─────────────────────────────────────────────────────────────────┘

Audit Preparation:
┌─────────────────────────────────────────────────────────────────┐
│  Auditor Question: "Show me all encrypted data stores"          │
│                                                                  │
│  Response:                                                       │
│  ├── Export: Policy compliance report (CSV)                     │
│  ├── Dashboard: Azure Security Center compliance view           │
│  └── Evidence: 30 seconds to produce (was 2 weeks manually)    │
└─────────────────────────────────────────────────────────────────┘

Results:
├── HIPAA audit: Passed with zero findings
├── Compliance drift: Detected within 15 minutes
├── Evidence generation: Minutes instead of weeks
├── Auto-remediation: 200+ resources fixed automatically
└── Compliance score: 94% → 99% over 6 months
```

**Key Vault Secrets Management:**
```
Company: Multi-team Development (20 teams, 50 applications)
Challenge: Centralized secrets without developers having access

Key Vault Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                        Key Vault Strategy                        │
│                                                                  │
│  Vault per Environment:                                          │
│  ├── kv-shared-nonprod: Development + Staging secrets          │
│  ├── kv-app1-prod: App1 production secrets                      │
│  ├── kv-app2-prod: App2 production secrets                      │
│  └── ...                                                         │
│                                                                  │
│  Access Model (Managed Identity):                                │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │  App Service/AKS Pod                                          │ │
│  │       │                                                        │ │
│  │       │ Managed Identity (no password!)                       │ │
│  │       │                                                        │ │
│  │       ▼                                                        │ │
│  │   Key Vault ───► "Get" permission on specific secrets       │ │
│  │       │                                                        │ │
│  │       ▼                                                        │ │
│  │   App reads DB connection string at runtime                  │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RBAC Roles:                                                     │
│  ├── Key Vault Administrator: Security team only (2 people)    │
│  ├── Key Vault Secrets User: App managed identities            │
│  ├── Key Vault Secrets Officer: DevOps pipelines (set secrets) │
│  └── Developers: Read secret names, not values (audit only)   │
│                                                                  │
│  Secret Rotation:                                                │
│  ├── Azure-managed secrets: Automatic (Storage, Cosmos)        │
│  ├── SQL password: Azure Function rotates every 30 days        │
│  ├── API keys: Event Grid trigger on expiry notification       │
│  └── Certificates: Auto-renew via Key Vault + App Gateway      │
└─────────────────────────────────────────────────────────────────┘

Before vs After:
├── Before: Secrets in config files, Git repos (exposed)
├── After: Zero secrets in code or config
├── Breach risk: Dramatically reduced
└── Audit: Full log of every secret access
```

---

## 15.2 Network Security

### Network Security Groups (NSGs)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Network Security Group                                │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     Inbound Security Rules                             │  │
│  │                                                                         │  │
│  │  Priority │ Name              │ Source      │ Dest Port │ Action      │  │
│  │  ─────────┼───────────────────┼─────────────┼───────────┼─────────────│  │
│  │  100      │ Allow-HTTPS       │ Internet    │ 443       │ Allow       │  │
│  │  110      │ Allow-SSH         │ 10.0.0.0/8  │ 22        │ Allow       │  │
│  │  120      │ Allow-RDP         │ 10.0.0.0/8  │ 3389      │ Allow       │  │
│  │  65000    │ AllowVnetInBound  │ VirtualNet  │ Any       │ Allow       │  │
│  │  65001    │ AllowAzureLB      │ AzureLB     │ Any       │ Allow       │  │
│  │  65500    │ DenyAllInBound    │ Any         │ Any       │ Deny        │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                     Outbound Security Rules                            │  │
│  │                                                                         │  │
│  │  Priority │ Name               │ Dest        │ Dest Port │ Action     │  │
│  │  ─────────┼────────────────────┼─────────────┼───────────┼────────────│  │
│  │  100      │ Allow-Storage      │ Storage     │ 443       │ Allow      │  │
│  │  110      │ Allow-SQL          │ Sql         │ 1433      │ Allow      │  │
│  │  65000    │ AllowVnetOutBound  │ VirtualNet  │ Any       │ Allow      │  │
│  │  65001    │ AllowInternetOut   │ Internet    │ Any       │ Allow      │  │
│  │  65500    │ DenyAllOutBound    │ Any         │ Any       │ Deny       │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Creating NSG Rules

```powershell
# Create NSG
$nsg = New-AzNetworkSecurityGroup `
    -ResourceGroupName "Security-RG" `
    -Location "East US" `
    -Name "Web-NSG"

# Add inbound rule - Allow HTTPS
$nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "Allow-HTTPS" `
    -Description "Allow HTTPS traffic" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 100 `
    -SourceAddressPrefix Internet `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 443

# Add outbound rule - Deny all except Azure services
$nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "Deny-Internet" `
    -Description "Deny internet except Azure services" `
    -Access Deny `
    -Protocol * `
    -Direction Outbound `
    -Priority 4000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix Internet `
    -DestinationPortRange *

# Save NSG
$nsg | Set-AzNetworkSecurityGroup
```

### Application Security Groups (ASGs)

```powershell
# Create ASGs
$webAsg = New-AzApplicationSecurityGroup `
    -ResourceGroupName "Security-RG" `
    -Name "Web-ASG" `
    -Location "East US"

$dbAsg = New-AzApplicationSecurityGroup `
    -ResourceGroupName "Security-RG" `
    -Name "DB-ASG" `
    -Location "East US"

# NSG rule using ASGs
$nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "Allow-Web-to-DB" `
    -Access Allow `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 200 `
    -SourceApplicationSecurityGroup $webAsg `
    -SourcePortRange * `
    -DestinationApplicationSecurityGroup $dbAsg `
    -DestinationPortRange 1433
```

### Azure Firewall

```powershell
# Create Azure Firewall
$firewallPip = New-AzPublicIpAddress `
    -Name "AzFW-PIP" `
    -ResourceGroupName "Security-RG" `
    -Location "East US" `
    -Sku "Standard" `
    -AllocationMethod "Static"

# Create Firewall Policy
$policy = New-AzFirewallPolicy `
    -Name "EnterprisePolicy" `
    -ResourceGroupName "Security-RG" `
    -Location "East US" `
    -ThreatIntelMode "Alert"

# Network rule collection
$networkRule = New-AzFirewallPolicyNetworkRule `
    -Name "AllowDNS" `
    -Protocol UDP `
    -SourceAddress "10.0.0.0/8" `
    -DestinationAddress "168.63.129.16" `
    -DestinationPort "53"

$networkRuleCollection = New-AzFirewallPolicyNetworkRuleCollection `
    -Name "NetworkRules" `
    -Priority 100 `
    -ActionType Allow `
    -Rule $networkRule

# Application rule collection
$appRule = New-AzFirewallPolicyApplicationRule `
    -Name "AllowMicrosoftUpdates" `
    -SourceAddress "10.0.0.0/8" `
    -Protocol @("https:443") `
    -TargetFqdn @("*.microsoft.com", "*.windowsupdate.com")

$appRuleCollection = New-AzFirewallPolicyApplicationRuleCollection `
    -Name "AppRules" `
    -Priority 200 `
    -ActionType Allow `
    -Rule $appRule
```

### Web Application Firewall (WAF)

```powershell
# Create WAF Policy
$wafPolicy = New-AzApplicationGatewayWebApplicationFirewallPolicy `
    -Name "WAF-Policy" `
    -ResourceGroupName "Security-RG" `
    -Location "East US"

# Configure managed rules
$managedRuleSet = New-AzApplicationGatewayFirewallPolicyManagedRuleSet `
    -RuleSetType "OWASP" `
    -RuleSetVersion "3.2"

$managedRules = New-AzApplicationGatewayFirewallPolicyManagedRule `
    -ManagedRuleSet $managedRuleSet

# Configure WAF settings
$wafPolicy.PolicySettings.Mode = "Prevention"
$wafPolicy.PolicySettings.State = "Enabled"
$wafPolicy.PolicySettings.RequestBodyCheck = $true
$wafPolicy.ManagedRules = $managedRules

Set-AzApplicationGatewayWebApplicationFirewallPolicy `
    -Name "WAF-Policy" `
    -ResourceGroupName "Security-RG" `
    -PolicySettings $wafPolicy.PolicySettings `
    -ManagedRules $wafPolicy.ManagedRules
```

### Private Endpoints

```powershell
# Create Private Endpoint for Storage
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Data-RG" -Name "mystorageaccount"

$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
    -Name "StoragePEC" `
    -PrivateLinkServiceId $storageAccount.Id `
    -GroupId "blob"

$vnet = Get-AzVirtualNetwork -Name "Hub-VNet" -ResourceGroupName "Network-RG"
$subnet = $vnet.Subnets | Where-Object { $_.Name -eq "PrivateEndpoints" }

New-AzPrivateEndpoint `
    -ResourceGroupName "Data-RG" `
    -Name "Storage-PE" `
    -Location "East US" `
    -Subnet $subnet `
    -PrivateLinkServiceConnection $privateEndpointConnection
```

---

## 15.3 Identity and Access Management

### Azure AD Security Features

| Feature | Description |
|---------|-------------|
| **MFA** | Multi-factor authentication |
| **Conditional Access** | Context-aware access policies |
| **PIM** | Privileged Identity Management |
| **Identity Protection** | Risk-based policies |
| **Access Reviews** | Periodic access verification |

### Conditional Access Policies

```powershell
# Using Microsoft Graph PowerShell
# Require MFA for Azure Management
$conditions = @{
    applications = @{
        includeApplications = @("797f4846-ba00-4fd7-ba43-dac1f8f63013")  # Azure Management
    }
    users = @{
        includeGroups = @("all-users-group-id")
        excludeGroups = @("emergency-access-group-id")
    }
    locations = @{
        includeLocations = @("All")
        excludeLocations = @("trusted-location-id")
    }
}

$grantControls = @{
    operator = "OR"
    builtInControls = @("mfa")
}

$policy = @{
    displayName = "Require MFA for Azure Management"
    state = "enabled"
    conditions = $conditions
    grantControls = $grantControls
}

# Apply via Graph API
Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" -Body ($policy | ConvertTo-Json -Depth 10)
```

### Managed Identities

```powershell
# System-assigned managed identity
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "MyVM"
Update-AzVM -ResourceGroupName "VM-RG" -VM $vm -IdentityType SystemAssigned

# User-assigned managed identity
$identity = New-AzUserAssignedIdentity `
    -ResourceGroupName "Identity-RG" `
    -Name "MyAppIdentity" `
    -Location "East US"

# Assign to web app
$webapp = Get-AzWebApp -ResourceGroupName "App-RG" -Name "MyWebApp"
$webapp.Identity = @{
    Type = "UserAssigned"
    UserAssignedIdentities = @{
        $identity.Id = @{}
    }
}
Set-AzWebApp -WebApp $webapp
```

### RBAC Custom Roles

```json
{
    "Name": "Integration Developer",
    "Id": "custom-role-guid",
    "IsCustom": true,
    "Description": "Can manage integration resources",
    "Actions": [
        "Microsoft.Logic/*",
        "Microsoft.ServiceBus/*",
        "Microsoft.EventGrid/*",
        "Microsoft.ApiManagement/service/read",
        "Microsoft.Web/sites/*",
        "Microsoft.Insights/components/*"
    ],
    "NotActions": [
        "*/delete",
        "Microsoft.Authorization/*/Write"
    ],
    "DataActions": [],
    "NotDataActions": [],
    "AssignableScopes": [
        "/subscriptions/{subscription-id}/resourceGroups/Integration-RG"
    ]
}
```

```powershell
# Create custom role
$role = Get-Content "custom-role.json" | ConvertFrom-Json
New-AzRoleDefinition -InputFile "custom-role.json"

# Assign role
New-AzRoleAssignment `
    -ObjectId "user-or-group-object-id" `
    -RoleDefinitionName "Integration Developer" `
    -ResourceGroupName "Integration-RG"
```

---

## 15.4 Azure Key Vault

### Key Vault Configuration

```powershell
# Create Key Vault
$keyVault = New-AzKeyVault `
    -Name "kv-myapp-prod" `
    -ResourceGroupName "Security-RG" `
    -Location "East US" `
    -EnabledForDeployment `
    -EnabledForTemplateDeployment `
    -EnabledForDiskEncryption `
    -EnablePurgeProtection `
    -SoftDeleteRetentionInDays 90 `
    -EnableRbacAuthorization

# Set network rules
Update-AzKeyVaultNetworkRuleSet `
    -VaultName "kv-myapp-prod" `
    -DefaultAction Deny `
    -Bypass AzureServices `
    -VirtualNetworkResourceId "/subscriptions/.../subnets/app-subnet"
```

### Managing Secrets

```powershell
# Store secret
$secretValue = ConvertTo-SecureString "MySecretValue" -AsPlainText -Force
Set-AzKeyVaultSecret `
    -VaultName "kv-myapp-prod" `
    -Name "DatabasePassword" `
    -SecretValue $secretValue `
    -Tags @{
        "Application" = "MyApp"
        "Environment" = "Production"
    }

# Retrieve secret
$secret = Get-AzKeyVaultSecret -VaultName "kv-myapp-prod" -Name "DatabasePassword"
$password = $secret.SecretValue | ConvertFrom-SecureString -AsPlainText

# Secret with expiration
Set-AzKeyVaultSecret `
    -VaultName "kv-myapp-prod" `
    -Name "ApiKey" `
    -SecretValue $secretValue `
    -Expires (Get-Date).AddDays(90) `
    -NotBefore (Get-Date)
```

### Key Vault in Applications

```csharp
// .NET - Using Azure.Identity and Azure.Security.KeyVault.Secrets
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Using managed identity
var client = new SecretClient(
    new Uri("https://kv-myapp-prod.vault.azure.net/"),
    new DefaultAzureCredential());

// Get secret
KeyVaultSecret secret = await client.GetSecretAsync("DatabasePassword");
string connectionString = secret.Value;

// Using in ASP.NET Core configuration
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var builtConfig = config.Build();
            config.AddAzureKeyVault(
                new Uri($"https://{builtConfig["KeyVaultName"]}.vault.azure.net/"),
                new DefaultAzureCredential());
        });
```

### Key Vault Access Policies vs RBAC

| Approach | Pros | Cons |
|----------|------|------|
| **Access Policies** | Simple, vault-level control | Limited granularity |
| **RBAC** | Granular, consistent with Azure | More complex setup |

```powershell
# RBAC approach (recommended)
New-AzRoleAssignment `
    -ObjectId "app-service-principal-id" `
    -RoleDefinitionName "Key Vault Secrets User" `
    -Scope "/subscriptions/.../resourceGroups/Security-RG/providers/Microsoft.KeyVault/vaults/kv-myapp-prod"
```

---

## 15.5 Azure Policy

### Policy Definition

```json
{
    "mode": "Indexed",
    "policyRule": {
        "if": {
            "allOf": [
                {
                    "field": "type",
                    "equals": "Microsoft.Storage/storageAccounts"
                },
                {
                    "field": "Microsoft.Storage/storageAccounts/supportsHttpsTrafficOnly",
                    "notEquals": true
                }
            ]
        },
        "then": {
            "effect": "deny"
        }
    }
}
```

### Common Built-in Policies

| Policy | Description |
|--------|-------------|
| **Allowed locations** | Restrict resource deployment regions |
| **Allowed VM SKUs** | Control VM sizes |
| **Require tag** | Enforce tagging standards |
| **Audit HTTPS** | Audit resources missing HTTPS |
| **Deploy diagnostics** | Auto-deploy diagnostic settings |
| **Deny public IP** | Prevent public IP creation |

### Policy Initiative for Security

```json
{
    "properties": {
        "displayName": "Security Baseline Initiative",
        "description": "Collection of security policies",
        "policyDefinitions": [
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/404c3081-a854-4457-ae30-26a93ef643f9",
                "policyDefinitionReferenceId": "AuditSecureTransfer"
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/4f4f78b8-e367-4b10-a341-d9a4ad5cf1c7",
                "policyDefinitionReferenceId": "AuditPublicNetwork"
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1a4e592a-6a6e-44a5-9814-e36264ca96e7",
                "policyDefinitionReferenceId": "RequireMinTLS"
            }
        ]
    }
}
```

### Policy Assignment

```powershell
# Assign policy to management group
$policyDefinition = Get-AzPolicyDefinition -Name "Require HTTPS for storage"

New-AzPolicyAssignment `
    -Name "require-https-storage" `
    -Scope "/providers/Microsoft.Management/managementGroups/root-mg" `
    -PolicyDefinition $policyDefinition `
    -EnforcementMode Default

# With remediation
$policyAssignment = New-AzPolicyAssignment `
    -Name "deploy-diagnostics" `
    -Scope "/subscriptions/xxx" `
    -PolicyDefinition $deployDiagnosticsPolicy `
    -Location "East US" `
    -IdentityType SystemAssigned

# Create remediation task
Start-AzPolicyRemediation `
    -Name "remediate-diagnostics" `
    -PolicyAssignmentId $policyAssignment.PolicyAssignmentId `
    -ResourceDiscoveryMode ReEvaluateCompliance
```

---

## 15.6 Microsoft Defender for Cloud

### Security Posture Management

```powershell
# Enable Microsoft Defender for Cloud
Set-AzSecurityPricing `
    -Name "VirtualMachines" `
    -PricingTier "Standard"

Set-AzSecurityPricing `
    -Name "SqlServers" `
    -PricingTier "Standard"

Set-AzSecurityPricing `
    -Name "AppServices" `
    -PricingTier "Standard"

Set-AzSecurityPricing `
    -Name "StorageAccounts" `
    -PricingTier "Standard"

Set-AzSecurityPricing `
    -Name "KeyVaults" `
    -PricingTier "Standard"
```

### Security Recommendations

```powershell
# Get security recommendations
Get-AzSecurityTask | Select-Object Name, RecommendationType, State

# Get secure score
Get-AzSecuritySecureScore

# Get security assessments
Get-AzSecurityAssessment | Where-Object { $_.Status.Code -ne "Healthy" }
```

### Just-in-Time VM Access

```powershell
# Configure JIT policy
$jitPolicy = @{
    kind = "Basic"
    properties = @{
        virtualMachines = @(
            @{
                id = "/subscriptions/.../resourceGroups/VM-RG/providers/Microsoft.Compute/virtualMachines/MyVM"
                ports = @(
                    @{
                        number = 22
                        protocol = "TCP"
                        allowedSourceAddressPrefix = "10.0.0.0/8"
                        maxRequestAccessDuration = "PT3H"
                    }
                    @{
                        number = 3389
                        protocol = "TCP"
                        allowedSourceAddressPrefix = "10.0.0.0/8"
                        maxRequestAccessDuration = "PT3H"
                    }
                )
            }
        )
    }
}

# Apply JIT policy via REST API
Invoke-AzRestMethod `
    -Method PUT `
    -Path "/subscriptions/{sub}/resourceGroups/VM-RG/providers/Microsoft.Security/locations/eastus/jitNetworkAccessPolicies/default?api-version=2020-01-01" `
    -Payload ($jitPolicy | ConvertTo-Json -Depth 10)
```

---

## 15.7 Microsoft Sentinel (SIEM)

### Sentinel Setup

```powershell
# Create Log Analytics workspace
$workspace = New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "Security-RG" `
    -Name "law-sentinel" `
    -Location "East US" `
    -Sku "PerGB2018"

# Enable Sentinel
New-AzSentinelOnboardingState `
    -ResourceGroupName "Security-RG" `
    -WorkspaceName "law-sentinel"
```

### Data Connectors

```powershell
# Enable Azure AD connector
New-AzSentinelDataConnector `
    -ResourceGroupName "Security-RG" `
    -WorkspaceName "law-sentinel" `
    -Kind "AzureActiveDirectory" `
    -TenantId (Get-AzContext).Tenant.Id `
    -DataTypes @{
        alerts = @{ state = "enabled" }
    }

# Enable Azure Activity connector
New-AzSentinelDataConnector `
    -ResourceGroupName "Security-RG" `
    -WorkspaceName "law-sentinel" `
    -Kind "AzureActivity" `
    -DataTypes @{
        subscription = @{ state = "enabled" }
    }
```

### Analytics Rules (KQL)

```kusto
// Detect multiple failed logins
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType != 0  // Failed login
| summarize FailedAttempts = count() by UserPrincipalName, IPAddress
| where FailedAttempts > 5
| project UserPrincipalName, IPAddress, FailedAttempts

// Detect privilege escalation
AzureActivity
| where TimeGenerated > ago(1d)
| where OperationNameValue has_any ("Microsoft.Authorization/roleAssignments/write")
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup

// Detect unusual resource deployment
AzureActivity
| where TimeGenerated > ago(7d)
| where OperationNameValue endswith "/write"
| where ActivityStatusValue == "Success"
| summarize ResourceTypes = make_set(ResourceProviderValue), Count = count() by Caller
| where Count > 50
| order by Count desc
```

---

## 15.8 Compliance and Audit

### Azure Audit Logs

```powershell
# Get activity logs
Get-AzActivityLog `
    -StartTime (Get-Date).AddDays(-7) `
    -EndTime (Get-Date) `
    -ResourceGroupName "Integration-RG"

# Export to storage
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Audit-RG" -Name "auditlogs"

Set-AzDiagnosticSetting `
    -Name "ExportActivityLogs" `
    -ResourceId "/subscriptions/{sub-id}" `
    -StorageAccountId $storageAccount.Id `
    -Enabled $true `
    -Category @("Administrative", "Security", "ServiceHealth", "Alert")
```

### Compliance Dashboard

```powershell
# Get compliance state
Get-AzPolicyState `
    -SubscriptionId (Get-AzContext).Subscription.Id `
    -Filter "ComplianceState eq 'NonCompliant'" |
    Group-Object PolicyDefinitionName |
    Select-Object Name, Count

# Export compliance report
$complianceReport = Get-AzPolicyState -All |
    Select-Object ResourceId, PolicyDefinitionName, ComplianceState, Timestamp |
    Export-Csv -Path "compliance-report.csv"
```

---

## 15.9 Security for Integration Services

### Securing API Management

```xml
<!-- Validate JWT token -->
<policies>
    <inbound>
        <validate-jwt header-name="Authorization" require-scheme="Bearer">
            <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
            <required-claims>
                <claim name="aud">
                    <value>api://myapi</value>
                </claim>
                <claim name="roles" match="any">
                    <value>api.read</value>
                    <value>api.write</value>
                </claim>
            </required-claims>
        </validate-jwt>
        
        <!-- Rate limiting -->
        <rate-limit calls="100" renewal-period="60" />
        
        <!-- IP filtering -->
        <ip-filter action="allow">
            <address-range from="10.0.0.0" to="10.255.255.255" />
        </ip-filter>
        
        <!-- CORS -->
        <cors allow-credentials="true">
            <allowed-origins>
                <origin>https://myapp.com</origin>
            </allowed-origins>
            <allowed-methods>
                <method>GET</method>
                <method>POST</method>
            </allowed-methods>
        </cors>
    </inbound>
</policies>
```

### Securing Service Bus

```powershell
# Service Bus with private endpoint
$serviceBusNamespace = New-AzServiceBusNamespace `
    -ResourceGroupName "Integration-RG" `
    -Name "sb-secure" `
    -Location "East US" `
    -SkuName "Premium" `
    -DisableLocalAuth $true  # Force Azure AD auth

# Create private endpoint
$privateEndpoint = New-AzPrivateEndpoint `
    -ResourceGroupName "Integration-RG" `
    -Name "sb-pe" `
    -Location "East US" `
    -Subnet $subnet `
    -PrivateLinkServiceConnection @{
        Name = "sb-connection"
        PrivateLinkServiceId = $serviceBusNamespace.Id
        GroupIds = @("namespace")
    }
```

### Securing Azure Functions

```json
// host.json - Security settings
{
    "extensions": {
        "http": {
            "routePrefix": "api",
            "maxConcurrentRequests": 100,
            "maxOutstandingRequests": 200
        }
    },
    "functionTimeout": "00:05:00"
}
```

```csharp
// Function with Azure AD authentication
[Function("SecureFunction")]
public async Task<HttpResponseData> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestData req,
    FunctionContext executionContext)
{
    // Validate token
    var authHeader = req.Headers.GetValues("Authorization").FirstOrDefault();
    if (string.IsNullOrEmpty(authHeader) || !authHeader.StartsWith("Bearer "))
    {
        var unauthorizedResponse = req.CreateResponse(HttpStatusCode.Unauthorized);
        return unauthorizedResponse;
    }

    var token = authHeader.Substring("Bearer ".Length);
    var tokenHandler = new JwtSecurityTokenHandler();
    
    // Validate and process...
}
```

---

## 15.10 Summary and Best Practices

### Security Best Practices

| Area | Best Practice |
|------|---------------|
| **Network** | Use NSGs, private endpoints, firewall |
| **Identity** | Enable MFA, use managed identities |
| **Secrets** | Store in Key Vault, rotate regularly |
| **Monitoring** | Enable Defender, configure alerts |
| **Compliance** | Apply Azure Policy, audit regularly |
| **Access** | Implement least privilege, use RBAC |

### Integration Security Checklist

- [ ] Enable HTTPS/TLS 1.2+ for all endpoints
- [ ] Use managed identities for service authentication
- [ ] Store secrets in Key Vault
- [ ] Implement rate limiting on APIs
- [ ] Configure WAF for web applications
- [ ] Use private endpoints for PaaS services
- [ ] Enable diagnostic logging
- [ ] Configure alerts for security events
- [ ] Review access regularly
- [ ] Apply Azure Policy for compliance

---

## Practice Questions

1. What is the difference between NSGs and Azure Firewall?
2. How do you implement zero-trust security in Azure?
3. What are the benefits of using managed identities?
4. How do you secure secrets in an integration solution?
5. What policies should be applied at the management group level?
6. How do you configure JIT VM access?
7. Explain the role of Microsoft Sentinel in security operations.

---

## Additional Resources

- [Azure Security Documentation](https://docs.microsoft.com/azure/security/)
- [Microsoft Cloud Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/)
- [Azure Policy Samples](https://github.com/Azure/azure-policy)
- [Microsoft Defender for Cloud](https://docs.microsoft.com/azure/defender-for-cloud/)
