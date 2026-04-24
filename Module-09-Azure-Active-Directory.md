# Module 09 - Microsoft Azure Active Directory

## Learning Objectives
By the end of this module, you will be able to:
- Understand Azure Active Directory concepts
- Compare Windows AD with Azure AD
- Manage Azure AD Users
- Create and manage Azure AD Groups
- Configure Azure AD Domains
- Work with Azure AD Tenants
- Implement authentication options
- Configure Azure AD Connect
- Set up Self-Service Password Reset (SSPR)
- Implement Multi-Factor Authentication (MFA)
- Apply Resource Locks

---

## 9.1 Azure Active Directory (Azure AD)

### What is Azure Active Directory?
Azure Active Directory (Azure AD) is Microsoft's cloud-based identity and access management service. It helps employees sign in and access resources.

### Azure AD Features

| Feature | Description |
|---------|-------------|
| **Single Sign-On (SSO)** | One set of credentials for all apps |
| **Multi-Factor Authentication** | Additional verification methods |
| **Self-Service Password Reset** | Users can reset their passwords |
| **Conditional Access** | Policy-based access control |
| **Application Management** | Integrate SaaS applications |
| **B2B Collaboration** | External user access |
| **B2C** | Customer identity management |
| **Device Management** | Register and manage devices |

#### 🏢 Real-World Azure AD Use Cases:

**Enterprise Single Sign-On (SSO):**
```
Company: GlobalCorp (15,000 employees, 200+ applications)
Challenge: Users had 20+ passwords, IT spent 40% time on password resets

Before Azure AD:
┌─────────────────────────────────────────────────────────────────┐
│  User Experience:                                                │
│  ├── Login to laptop (AD password)                              │
│  ├── Login to Salesforce (different password)                   │
│  ├── Login to ServiceNow (another password)                     │
│  ├── Login to Workday (yet another password)                    │
│  └── ... 15 more applications                                   │
│                                                                  │
│  IT Burden:                                                      │
│  ├── Password reset tickets: 500/week                           │
│  ├── Average reset time: 15 minutes                             │
│  └── Annual cost: $400,000 in IT time                          │
└─────────────────────────────────────────────────────────────────┘

After Azure AD SSO:
┌─────────────────────────────────────────────────────────────────┐
│                         Azure AD                                 │
│                            │                                     │
│  ┌─────────────────────────┼────────────────────────────────┐   │
│  │                         │                                 │   │
│  ▼           ▼           ▼           ▼           ▼          │   │
│  Salesforce  ServiceNow  Workday   SAP         Custom Apps   │   │
│  (SAML)      (OIDC)      (SAML)    (SAML)      (OIDC)       │   │
│                                                              │   │
│  User Experience:                                            │   │
│  ├── Login once to Azure AD (with MFA)                       │   │
│  ├── Click app in My Apps portal                             │   │
│  └── Automatically signed in (no password)                   │   │
└─────────────────────────────────────────────────────────────────┘

Results:
├── User passwords managed: 1 (down from 20+)
├── Password reset tickets: 50/week (90% reduction)
├── User productivity: +30 minutes/day saved
├── Security incidents: 60% reduction (fewer weak passwords)
└── Annual savings: $350,000
```

**Conditional Access for Zero Trust:**
```
Company: FinanceSecure (Bank, strict compliance requirements)
Challenge: Grant access based on user, device, location, and risk

Conditional Access Policies:
┌─────────────────────────────────────────────────────────────────┐
│  Policy 1: MFA Required for All Users                           │
│  ├── Who: All users                                              │
│  ├── What: All cloud apps                                        │
│  ├── Conditions: Any location                                    │
│  └── Control: Require MFA                                        │
│                                                                  │
│  Policy 2: Block High-Risk Countries                             │
│  ├── Who: All users                                              │
│  ├── What: All cloud apps                                        │
│  ├── Conditions: Location = High-risk countries                 │
│  └── Control: Block access                                       │
│                                                                  │
│  Policy 3: Compliant Device Required for Finance Apps           │
│  ├── Who: Finance department                                     │
│  ├── What: SAP, Banking portal, Trading platform                │
│  ├── Conditions: Any device                                      │
│  └── Control: Require compliant device (Intune managed)         │
│                                                                  │
│  Policy 4: Session Timeout for Sensitive Data                   │
│  ├── Who: Traders                                                │
│  ├── What: Trading platform                                      │
│  ├── Conditions: After 1 hour inactive                          │
│  └── Control: Sign-in frequency = 1 hour                        │
│                                                                  │
│  Policy 5: Risky Sign-In Protection                             │
│  ├── Who: All users                                              │
│  ├── Conditions: Sign-in risk = High (Identity Protection)      │
│  └── Control: Block access + notify security                    │
└─────────────────────────────────────────────────────────────────┘

Decision Flow:
User Sign-in → Azure AD → Check all policies → Grant/Deny/MFA

Results:
├── Unauthorized access attempts blocked: 10,000/month
├── Compliance audit: Passed (100% MFA enforcement)
├── Phishing success rate: Near zero (compromised password alone won't work)
└── User experience: Seamless for compliant users
```

**B2B Collaboration:**
```
Company: ConsultingFirm (Works with 500 client organizations)
Challenge: Share documents securely with external clients

B2B Guest User Setup:
┌─────────────────────────────────────────────────────────────────┐
│  ConsultingFirm Tenant (Azure AD)                                │
│                                                                  │
│  Internal Users:           Guest Users (B2B):                   │
│  ├── john@consulting.com   ├── sarah@clienta.com (invited)     │
│  ├── mary@consulting.com   ├── mike@clientb.com (invited)      │
│  └── ...500 employees      └── ...10,000 external guests       │
│                                                                  │
│  Guest User Capabilities:                                        │
│  ├── Access specific Teams channels                             │
│  ├── View shared SharePoint sites                               │
│  ├── Collaborate on Power BI reports                            │
│  └── Use designated apps (client portal)                        │
│                                                                  │
│  Security Controls:                                              │
│  ├── Guests must use their corporate account (federated)        │
│  ├── MFA required (guest's organization OR our tenant)          │
│  ├── Access review every 90 days (auto-remove stale guests)     │
│  └── No access to internal channels/finance apps                │
│                                                                  │
│  Guest Invitation Flow:                                          │
│  Employee clicks "Share" → Enter guest email → Azure AD sends   │
│  invite → Guest accepts → Guest uses their company credentials  │
│  → Accesses only authorized resources                            │
└─────────────────────────────────────────────────────────────────┘

Results:
├── External collaboration: Seamless, no shared passwords
├── Security: Each guest authenticated by their own organization
├── Cleanup: Auto-remove guests who haven't signed in for 90 days
└── Audit: Full log of all guest access activities
```

**B2C Customer Identity:**
```
Company: RetailApp (E-commerce, 10 million customers)
Challenge: Customers need easy, secure sign-up/sign-in

Azure AD B2C Configuration:
┌─────────────────────────────────────────────────────────────────┐
│                      Azure AD B2C Tenant                         │
│                                                                  │
│  Sign-Up/Sign-In Options:                                        │
│  ├── Email + Password (local account)                           │
│  ├── Social: Google, Facebook, Apple, Microsoft                 │
│  └── Passwordless: Email OTP, Phone                             │
│                                                                  │
│  User Flow (Sign-Up):                                            │
│  ├── Step 1: Choose sign-up method (email or social)            │
│  ├── Step 2: Collect attributes (name, email, preferences)      │
│  ├── Step 3: Email verification (prevent fake accounts)         │
│  ├── Step 4: MFA setup (optional but encouraged)                │
│  └── Step 5: Create account → Redirect to app                  │
│                                                                  │
│  User Flow (Sign-In):                                            │
│  ├── User enters email/password OR clicks social login          │
│  ├── Token issued (customizable claims)                         │
│  └── User redirected to app authenticated                       │
│                                                                  │
│  Security Features:                                              │
│  ├── CAPTCHA: Block bots during sign-up                        │
│  ├── Identity Protection: Block risky sign-ins                 │
│  ├── Password complexity: Enforced policies                     │
│  └── Brute force protection: Account lockout                   │
│                                                                  │
│  Custom UI:                                                       │
│  └── Branded sign-in pages (company logo, colors)              │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Sign-up conversion: +25% (social login option)
├── Support tickets (password reset): -80% (self-service)
├── Fraudulent accounts: -90% (Identity Protection)
├── Scale: 10M users, 100K sign-ins/day
└── Cost: $0.0025/MAU ($25,000/month for 10M users)
```

### Azure AD Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Azure Active Directory                        │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                      Tenant                              │   │
│  │        (Organization's dedicated instance)               │   │
│  │                                                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│  │  │   Users     │  │   Groups    │  │   Devices   │      │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │   │
│  │                                                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│  │  │   Apps      │  │   Domains   │  │   Roles     │      │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐             │
│         ▼                    ▼                    ▼             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │ Azure       │    │ Microsoft   │    │ Third-Party │         │
│  │ Services    │    │ 365         │    │ Apps        │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Azure AD Editions

| Edition | Features |
|---------|----------|
| **Free** | User/group management, SSO (10 apps), basic reports |
| **Microsoft 365 Apps** | Free + company branding, SLA |
| **Premium P1** | Hybrid identity, conditional access, dynamic groups |
| **Premium P2** | P1 + Identity Protection, PIM, access reviews |

### Azure AD Terminology

| Term | Description |
|------|-------------|
| **Tenant** | Dedicated Azure AD instance for organization |
| **Directory** | Container for users, groups, apps |
| **Domain** | DNS name for sign-in (contoso.onmicrosoft.com) |
| **Subscription** | Billing/access container (linked to tenant) |

---

## 9.2 Windows AD vs Azure AD

### Key Differences

| Feature | Windows AD | Azure AD |
|---------|------------|----------|
| **Protocol** | LDAP, Kerberos | REST APIs, OAuth 2.0, SAML |
| **Structure** | Hierarchical (OUs, trees, forests) | Flat |
| **Query** | LDAP queries | Microsoft Graph |
| **Authentication** | Kerberos tickets | Tokens (JWT) |
| **Devices** | Domain join | Azure AD join, registration |
| **Group Policy** | GPOs | Intune, Conditional Access |
| **Location** | On-premises servers | Cloud service |

### Authentication Comparison

```
Windows AD Authentication:
┌─────────┐    Kerberos    ┌────────────┐    Access    ┌──────────┐
│  User   │──────────────►│   Domain   │────────────►│ Resource │
│         │    Ticket      │ Controller │              │          │
└─────────┘               └────────────┘              └──────────┘

Azure AD Authentication:
┌─────────┐    OAuth 2.0   ┌────────────┐    Token    ┌──────────┐
│  User   │──────────────►│  Azure AD  │────────────►│  Cloud   │
│         │    (HTTPS)     │            │    (JWT)   │   App    │
└─────────┘               └────────────┘              └──────────┘
```

### Use Cases

| Scenario | Solution |
|----------|----------|
| Cloud-only organization | Azure AD only |
| On-premises only | Windows AD |
| Hybrid identity | Azure AD + Windows AD + Azure AD Connect |
| Migration to cloud | Start with hybrid, transition to cloud |

---

## 9.3 Azure AD Users

### User Types

| Type | Description |
|------|-------------|
| **Cloud identity** | Created and managed in Azure AD |
| **Synced identity** | Synced from on-premises AD |
| **Guest user** | External user (B2B) |

### User Properties

| Property | Description |
|----------|-------------|
| **User Principal Name** | Sign-in name (user@domain.com) |
| **Display Name** | Friendly name |
| **Job Title** | Position |
| **Department** | Team/department |
| **Usage Location** | For licensing (required for M365) |
| **Manager** | Reporting structure |

### Creating Users

```powershell
# Create user with Azure AD module
Install-Module AzureAD
Connect-AzureAD

$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$passwordProfile.Password = "P@ssw0rd123!"
$passwordProfile.ForceChangePasswordNextLogin = $true

New-AzureADUser `
  -DisplayName "John Doe" `
  -UserPrincipalName "john.doe@contoso.onmicrosoft.com" `
  -PasswordProfile $passwordProfile `
  -AccountEnabled $true `
  -MailNickname "john.doe" `
  -Department "IT" `
  -JobTitle "Administrator"
```

```powershell
# Using Az module
New-AzADUser `
  -DisplayName "Jane Smith" `
  -UserPrincipalName "jane.smith@contoso.onmicrosoft.com" `
  -Password (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -AccountEnabled $true
```

```bash
# Azure CLI
az ad user create \
  --display-name "John Doe" \
  --user-principal-name "john.doe@contoso.onmicrosoft.com" \
  --password "P@ssw0rd123!" \
  --force-change-password-next-login
```

### Managing Users

```powershell
# Get all users
Get-AzureADUser

# Get specific user
Get-AzureADUser -ObjectId "john.doe@contoso.onmicrosoft.com"

# Update user
Set-AzureADUser `
  -ObjectId "john.doe@contoso.onmicrosoft.com" `
  -Department "Engineering" `
  -JobTitle "Senior Engineer"

# Delete user
Remove-AzureADUser -ObjectId "john.doe@contoso.onmicrosoft.com"

# Restore deleted user (within 30 days)
Restore-AzureADDeletedApplication -ObjectId "deleted-user-object-id"
```

### Bulk User Operations

```powershell
# Bulk create users from CSV
$users = Import-Csv "users.csv"

foreach ($user in $users) {
    $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
    $passwordProfile.Password = "TempP@ss123!"
    $passwordProfile.ForceChangePasswordNextLogin = $true
    
    New-AzureADUser `
      -DisplayName $user.DisplayName `
      -UserPrincipalName $user.UPN `
      -PasswordProfile $passwordProfile `
      -AccountEnabled $true `
      -MailNickname $user.MailNickname `
      -Department $user.Department
}
```

---

## 9.4 Azure AD Groups

### Group Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Security** | Access control to resources | RBAC, app access |
| **Microsoft 365** | Collaboration + mail/calendar | Teams, SharePoint |

### Membership Types

| Type | Description |
|------|-------------|
| **Assigned** | Manually add/remove members |
| **Dynamic User** | Automatic based on user attributes |
| **Dynamic Device** | Automatic based on device attributes |

### Creating Groups

```powershell
# Create security group
New-AzureADGroup `
  -DisplayName "IT Administrators" `
  -Description "IT Admin team" `
  -SecurityEnabled $true `
  -MailEnabled $false `
  -MailNickname "ITAdmins"

# Create Microsoft 365 group
New-AzureADMSGroup `
  -DisplayName "Project Team" `
  -Description "Project team collaboration" `
  -GroupTypes "Unified" `
  -SecurityEnabled $false `
  -MailEnabled $true `
  -MailNickname "ProjectTeam"
```

### Dynamic Group Rules

```powershell
# Create dynamic group
New-AzureADMSGroup `
  -DisplayName "All IT Staff" `
  -Description "All users in IT department" `
  -GroupTypes "DynamicMembership" `
  -SecurityEnabled $true `
  -MailEnabled $false `
  -MailNickname "AllITStaff" `
  -MembershipRule '(user.department -eq "IT")' `
  -MembershipRuleProcessingState "On"
```

### Common Dynamic Group Rules

| Rule | Description |
|------|-------------|
| `user.department -eq "Sales"` | Users in Sales department |
| `user.jobTitle -contains "Manager"` | Job title contains Manager |
| `user.accountEnabled -eq true` | All enabled accounts |
| `user.userType -eq "Guest"` | All guest users |
| `user.country -eq "US"` | Users in United States |

### Managing Group Membership

```powershell
# Add member to group
Add-AzureADGroupMember `
  -ObjectId "group-object-id" `
  -RefObjectId "user-object-id"

# Remove member
Remove-AzureADGroupMember `
  -ObjectId "group-object-id" `
  -MemberId "user-object-id"

# List members
Get-AzureADGroupMember -ObjectId "group-object-id"

# Add group owner
Add-AzureADGroupOwner `
  -ObjectId "group-object-id" `
  -RefObjectId "owner-object-id"
```

---

## 9.5 Azure AD Domains

### Default Domain
Every Azure AD tenant gets a default domain: `tenantname.onmicrosoft.com`

### Custom Domains
Add your organization's domain (e.g., contoso.com) for professional UPN addresses.

### Adding Custom Domain

1. **Add domain in Azure AD**
```powershell
New-AzureADDomain -Name "contoso.com"
```

2. **Get verification records**
```powershell
Get-AzureADDomainVerificationDnsRecord -Name "contoso.com"
```

3. **Add DNS records at registrar**
   - TXT record OR
   - MX record

4. **Verify domain**
```powershell
Confirm-AzureADDomain -Name "contoso.com"
```

### Domain Types

| Type | Description |
|------|-------------|
| **Managed** | DNS managed by Microsoft (for M365) |
| **Federated** | Authentication via on-premises ADFS |

```powershell
# List all domains
Get-AzureADDomain

# Set primary domain
Set-AzureADDomain -Name "contoso.com" -IsDefault $true
```

---

## 9.6 Azure AD Tenants

### What is a Tenant?
A tenant is a dedicated instance of Azure AD that represents an organization. It's created automatically when an organization signs up for a Microsoft cloud service.

### Tenant Characteristics

| Property | Description |
|----------|-------------|
| **Single instance** | One tenant per organization |
| **Globally unique** | Unique tenant ID |
| **Trust boundary** | Security boundary |
| **Resource container** | Contains all directory objects |

### Managing Tenants

```powershell
# Get tenant details
Get-AzureADTenantDetail

# Get tenant ID
(Get-AzureADTenantDetail).ObjectId

# List all subscriptions in tenant
Get-AzSubscription
```

### Multi-Tenant Scenarios

| Scenario | Description |
|----------|-------------|
| **Single tenant** | One organization |
| **Multi-tenant apps** | App serves multiple tenants |
| **B2B** | Guest access from other tenants |
| **B2C** | Customer identity (separate tenant) |

### Switching Tenants

```powershell
# Connect to specific tenant
Connect-AzureAD -TenantId "tenant-id"

# Switch Azure context to different tenant
Connect-AzAccount -Tenant "tenant-id"
```

---

## 9.7 Authentication Options

### Authentication Methods

| Method | Description |
|--------|-------------|
| **Password** | Traditional username/password |
| **Passwordless** | FIDO2, Windows Hello, Authenticator app |
| **MFA** | Multiple verification factors |
| **Certificate** | X.509 certificate-based |
| **Federation** | ADFS, third-party IdP |

### Password Hash Synchronization

```
┌─────────────────┐     Hash     ┌─────────────────┐
│  On-premises    │────────────►│    Azure AD     │
│  AD (Password)  │   Sync      │  (Password Hash)│
└─────────────────┘             └─────────────────┘
         ▲                              │
         │                              │
         └────── User Authenticates ────┘
```

### Pass-through Authentication

```
┌─────────────────┐    Validate   ┌─────────────────┐
│    Azure AD     │◄────────────►│  On-premises    │
│  (Auth Request) │    Password  │  AD (via Agent) │
└─────────────────┘              └─────────────────┘
```

### Federation with ADFS

```
┌─────────────────┐   Redirect   ┌─────────────────┐   Validate   ┌─────────────────┐
│    Azure AD     │────────────►│     ADFS        │◄────────────►│  On-premises AD │
└─────────────────┘             └─────────────────┘              └─────────────────┘
                                        │
                                   SAML Token
                                        │
                                        ▼
                               ┌─────────────────┐
                               │      User       │
                               └─────────────────┘
```

### Comparison

| Feature | Password Hash Sync | Pass-through Auth | Federation |
|---------|-------------------|-------------------|------------|
| Cloud resilience | High | Medium | Low |
| On-prem dependency | Low | High | High |
| Complexity | Low | Medium | High |
| Smart Lockout | Yes | Yes | Depends |
| MFA integration | Native | Native | Varies |

---

## 9.8 Azure AD Connect

### What is Azure AD Connect?
Azure AD Connect synchronizes on-premises Active Directory with Azure AD, enabling hybrid identity.

### Azure AD Connect Components

```
On-premises Environment                     Azure Cloud
┌─────────────────────────┐               ┌─────────────────────────┐
│                         │               │                         │
│   Active Directory      │               │      Azure AD           │
│   ┌─────────────────┐   │               │   ┌─────────────────┐   │
│   │   Users         │   │               │   │   Users         │   │
│   │   Groups        │   │   ◄──────────►│   │   Groups        │   │
│   │   Computers     │   │   Azure AD    │   │   Devices       │   │
│   └─────────────────┘   │   Connect     │   └─────────────────┘   │
│                         │   Sync        │                         │
│   ┌─────────────────┐   │               │                         │
│   │  Azure AD       │   │               │                         │
│   │  Connect Server │   │               │                         │
│   └─────────────────┘   │               │                         │
│                         │               │                         │
└─────────────────────────┘               └─────────────────────────┘
```

### Installation Prerequisites

| Requirement | Details |
|-------------|---------|
| **Server OS** | Windows Server 2016+ |
| **Memory** | 4 GB+ |
| **Domain** | Must be domain-joined |
| **Account** | Enterprise Admin + Global Admin |
| **Connectivity** | Internet access to Azure |

### Synchronization Options

| Option | Description |
|--------|-------------|
| **Full sync** | Sync all objects |
| **OU filtering** | Sync specific OUs only |
| **Attribute filtering** | Sync specific attributes |
| **Group filtering** | Sync based on group membership |

### Configuring Azure AD Connect

```powershell
# Install Azure AD Connect (interactive wizard)
# Download from https://www.microsoft.com/download/details.aspx?id=47594

# After installation, manage sync:
# Start sync cycle
Start-ADSyncSyncCycle -PolicyType Delta

# Run full sync
Start-ADSyncSyncCycle -PolicyType Initial

# Check sync status
Get-ADSyncScheduler

# View sync errors
Get-ADSyncConnectorRunStatus
```

### Synchronization Schedule

| Cycle Type | Frequency |
|------------|-----------|
| **Delta sync** | Every 30 minutes |
| **Full sync** | Initial or on-demand |

---

## 9.9 Self-Service Password Reset (SSPR)

### What is SSPR?
Self-Service Password Reset allows users to reset their own passwords without contacting IT help desk.

### SSPR Configuration Options

| Option | Description |
|--------|-------------|
| **Disabled** | No SSPR |
| **Selected** | SSPR for specific groups |
| **All** | SSPR for all users |

### Authentication Methods for SSPR

| Method | Description |
|--------|-------------|
| **Email** | Send code to alternate email |
| **Mobile phone** | SMS or call |
| **Office phone** | Call to office number |
| **Security questions** | Answer predefined questions |
| **Mobile app notification** | Authenticator app approval |
| **Mobile app code** | Authenticator app TOTP code |

### Configuring SSPR

1. **Navigate to Azure AD** → Password reset
2. **Enable SSPR** for selected groups or all users
3. **Configure authentication methods** (require 1 or 2 methods)
4. **Set registration requirements**
5. **Configure notifications**

```powershell
# Using Microsoft Graph API
# Requires Microsoft.Graph module

# Enable SSPR for a group
$params = @{
    enablementType = "Selected"
    enabledGroupId = "group-object-id"
}

# Note: SSPR is typically configured via Azure Portal
```

### SSPR Registration

Users can register at: https://aka.ms/ssprsetup

### Password Writeback

Write back password changes to on-premises AD:

1. Enable in Azure AD Connect
2. Requires Azure AD Premium P1+
3. Enables cloud password reset to update on-prem AD

---

## 9.10 Multi-Factor Authentication (MFA)

### What is MFA?
Multi-Factor Authentication requires multiple verification methods, combining:
- Something you **know** (password)
- Something you **have** (phone, token)
- Something you **are** (biometrics)

### MFA Methods

| Method | Type |
|--------|------|
| Microsoft Authenticator | Push notification or TOTP |
| SMS | Text message code |
| Voice call | Automated phone call |
| FIDO2 security key | Hardware token |
| Windows Hello | Biometric/PIN |
| OATH tokens | Hardware/software TOTP |

### Enabling MFA

#### Per-user MFA (Legacy)

```
Azure Portal → Azure Active Directory → Users → Per-user MFA
```

#### Service Settings

Configure default methods and options:
- Allow/block verification methods
- Remember MFA on trusted devices
- Configure fraud alerts

### Conditional Access (Recommended)

```powershell
# Create Conditional Access policy requiring MFA
# Using Azure Portal: Azure AD → Security → Conditional Access

# Key components:
# - Users and groups: Who the policy applies to
# - Cloud apps: Which apps require MFA
# - Conditions: When to apply (location, device, risk)
# - Grant: Require MFA
```

### Conditional Access Policy Example

```
Policy: Require MFA for admin roles
┌─────────────────────────────────────────────────────────────┐
│ Assignments                                                  │
│   Users: Directory roles (Global Admin, User Admin, etc.)   │
│   Cloud apps: All cloud apps                                │
│   Conditions: Any location                                  │
├─────────────────────────────────────────────────────────────┤
│ Access Controls                                              │
│   Grant: Require multi-factor authentication                │
└─────────────────────────────────────────────────────────────┘
```

### Security Defaults

Enable basic security for all organizations (free):

1. Require MFA for admins
2. Require MFA for all users (when needed)
3. Block legacy authentication
4. Protect privileged actions

```powershell
# Check security defaults status via Azure Portal:
# Azure AD → Properties → Manage Security defaults
```

---

## 9.11 Resource Locks

### What are Resource Locks?
Resource locks prevent accidental deletion or modification of Azure resources.

### Lock Types

| Lock Type | Effect |
|-----------|--------|
| **ReadOnly** | Resources can be read but not modified/deleted |
| **CanNotDelete** | Resources can be modified but not deleted |

### Lock Inheritance

```
Subscription (CanNotDelete)
      │
      └── Resource Group (Inherits lock)
              │
              └── Resources (Inherits lock)
```

### Creating Resource Locks

```powershell
# Lock at subscription level
New-AzResourceLock `
  -LockName "SubscriptionLock" `
  -LockLevel CanNotDelete `
  -Scope "/subscriptions/{subscription-id}" `
  -LockNotes "Prevent accidental subscription deletion"

# Lock at resource group level
New-AzResourceLock `
  -LockName "RG-DeleteLock" `
  -LockLevel CanNotDelete `
  -ResourceGroupName "Production-RG" `
  -LockNotes "Protect production resources"

# Lock at resource level
New-AzResourceLock `
  -LockName "VM-ReadOnly" `
  -LockLevel ReadOnly `
  -ResourceGroupName "Production-RG" `
  -ResourceName "CriticalVM" `
  -ResourceType "Microsoft.Compute/virtualMachines"
```

```bash
# Azure CLI
az lock create \
  --name "RG-DeleteLock" \
  --lock-type CanNotDelete \
  --resource-group "Production-RG" \
  --notes "Protect production resources"
```

### Managing Locks

```powershell
# List all locks
Get-AzResourceLock

# List locks for resource group
Get-AzResourceLock -ResourceGroupName "Production-RG"

# Remove lock (required before deletion)
Remove-AzResourceLock `
  -LockName "RG-DeleteLock" `
  -ResourceGroupName "Production-RG"
```

### Lock Considerations

| Consideration | Details |
|---------------|---------|
| **Owner required** | Must be Owner or User Access Administrator to manage locks |
| **Before deletion** | Must remove lock before deleting resource |
| **Applies to all** | Locks apply to all users, including admins |
| **Inherited** | Child resources inherit parent locks |

---

## Hands-on Exercises

### Exercise 1: Add or Delete Users Using Azure Active Directory

```powershell
# Connect to Azure AD
Connect-AzureAD

# Create multiple users
$users = @(
    @{DisplayName="Alice Johnson"; UPN="alice.johnson"; Dept="IT"; Title="Admin"},
    @{DisplayName="Bob Wilson"; UPN="bob.wilson"; Dept="Sales"; Title="Manager"},
    @{DisplayName="Carol Brown"; UPN="carol.brown"; Dept="HR"; Title="Specialist"}
)

$domain = (Get-AzureADDomain | Where-Object {$_.IsDefault}).Name

foreach ($u in $users) {
    $pwd = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
    $pwd.Password = "Welcome123!"
    $pwd.ForceChangePasswordNextLogin = $true
    
    New-AzureADUser `
        -DisplayName $u.DisplayName `
        -UserPrincipalName "$($u.UPN)@$domain" `
        -PasswordProfile $pwd `
        -AccountEnabled $true `
        -MailNickname $u.UPN `
        -Department $u.Dept `
        -JobTitle $u.Title
    
    Write-Host "Created user: $($u.DisplayName)"
}

# Verify users
Get-AzureADUser | Select-Object DisplayName, UserPrincipalName, Department

# Delete a user
$userToDelete = Get-AzureADUser -SearchString "Carol Brown"
Remove-AzureADUser -ObjectId $userToDelete.ObjectId
Write-Host "Deleted user: Carol Brown"

# Restore deleted user (within 30 days)
Get-AzureADDeletedUser | Where-Object {$_.DisplayName -eq "Carol Brown"}
```

### Exercise 2: Add or Delete Tenants Using Azure Active Directory

```powershell
# Note: Creating new tenants requires specific permissions
# Navigate to Azure Portal → Azure Active Directory → Create a tenant

# List current tenant info
Get-AzureADTenantDetail | Select-Object DisplayName, ObjectId

# Switch to different tenant (if you have access)
Disconnect-AzureAD
Connect-AzureAD -TenantId "other-tenant-id"
```

### Exercise 3: Create a Basic Group and Add Members

```powershell
# Create security group
$itGroup = New-AzureADGroup `
    -DisplayName "IT Department" `
    -Description "All IT staff members" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickname "ITDepartment"

Write-Host "Created group: IT Department with ID: $($itGroup.ObjectId)"

# Create M365 group
$projectGroup = New-AzureADMSGroup `
    -DisplayName "Project Alpha Team" `
    -Description "Team for Project Alpha" `
    -GroupTypes "Unified" `
    -SecurityEnabled $false `
    -MailEnabled $true `
    -MailNickname "ProjectAlpha"

# Add members to group
$users = Get-AzureADUser | Where-Object {$_.Department -eq "IT"}
foreach ($user in $users) {
    Add-AzureADGroupMember -ObjectId $itGroup.ObjectId -RefObjectId $user.ObjectId
    Write-Host "Added $($user.DisplayName) to IT Department group"
}

# Create dynamic group
$dynamicGroup = New-AzureADMSGroup `
    -DisplayName "All Sales Staff" `
    -Description "Dynamic group for all sales department" `
    -GroupTypes "DynamicMembership" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickname "AllSales" `
    -MembershipRule '(user.department -eq "Sales")' `
    -MembershipRuleProcessingState "On"

Write-Host "Created dynamic group: All Sales Staff"

# List group members
Get-AzureADGroupMember -ObjectId $itGroup.ObjectId | Select-Object DisplayName, UserPrincipalName
```

### Exercise 4: Applying Resource Locks

```powershell
# Create resource group for testing
New-AzResourceGroup -Name "Lock-Test-RG" -Location "East US"

# Create a storage account in the group
$storageAccount = New-AzStorageAccount `
    -ResourceGroupName "Lock-Test-RG" `
    -Name "lockteststorage$(Get-Random -Maximum 9999)" `
    -Location "East US" `
    -SkuName "Standard_LRS"

# Apply CanNotDelete lock to resource group
$rgLock = New-AzResourceLock `
    -LockName "Prevent-RG-Delete" `
    -LockLevel CanNotDelete `
    -ResourceGroupName "Lock-Test-RG" `
    -LockNotes "Prevent accidental deletion of test resources"

Write-Host "Applied CanNotDelete lock to resource group"

# Apply ReadOnly lock to storage account
$saLock = New-AzResourceLock `
    -LockName "ReadOnly-Storage" `
    -LockLevel ReadOnly `
    -ResourceGroupName "Lock-Test-RG" `
    -ResourceName $storageAccount.StorageAccountName `
    -ResourceType "Microsoft.Storage/storageAccounts" `
    -LockNotes "Prevent modifications to storage account"

Write-Host "Applied ReadOnly lock to storage account"

# Try to modify storage (will fail due to ReadOnly lock)
try {
    Set-AzStorageAccount `
        -ResourceGroupName "Lock-Test-RG" `
        -Name $storageAccount.StorageAccountName `
        -EnableHttpsTrafficOnly $true
} catch {
    Write-Host "Cannot modify storage account - locked as ReadOnly" -ForegroundColor Yellow
}

# Try to delete resource group (will fail due to CanNotDelete lock)
try {
    Remove-AzResourceGroup -Name "Lock-Test-RG" -Force
} catch {
    Write-Host "Cannot delete resource group - locked with CanNotDelete" -ForegroundColor Yellow
}

# View all locks
Get-AzResourceLock -ResourceGroupName "Lock-Test-RG" | Format-Table Name, ResourceType, LockLevel

# Remove locks to clean up
Remove-AzResourceLock -LockName "ReadOnly-Storage" -ResourceGroupName "Lock-Test-RG" `
    -ResourceName $storageAccount.StorageAccountName -ResourceType "Microsoft.Storage/storageAccounts"
Remove-AzResourceLock -LockName "Prevent-RG-Delete" -ResourceGroupName "Lock-Test-RG"

Write-Host "Locks removed successfully"
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| Azure AD | Cloud identity service, SSO, application management |
| Windows AD vs Azure AD | Different protocols, flat vs hierarchical structure |
| Users | Cloud, synced, guest identities |
| Groups | Security and M365, assigned and dynamic |
| Domains | Default + custom domains, verification required |
| Tenants | Organization's dedicated Azure AD instance |
| Authentication | Password hash sync, pass-through, federation |
| Azure AD Connect | Sync on-prem AD to Azure AD |
| SSPR | Self-service password reset for users |
| MFA | Multiple verification factors |
| Resource Locks | Prevent accidental deletion/modification |

---

## Review Questions

1. What is the difference between Windows AD and Azure AD authentication protocols?
2. Explain the three user types in Azure AD.
3. What is the purpose of dynamic groups?
4. Compare password hash sync, pass-through authentication, and federation.
5. What authentication methods can be used for SSPR?
6. How do resource locks inherit from parent to child resources?

---

## Additional Resources

- [Azure Active Directory Documentation](https://docs.microsoft.com/azure/active-directory/)
- [Azure AD Connect Documentation](https://docs.microsoft.com/azure/active-directory/hybrid/)
- [MFA Documentation](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks)
- [SSPR Documentation](https://docs.microsoft.com/azure/active-directory/authentication/concept-sspr-howitworks)
- [Conditional Access](https://docs.microsoft.com/azure/active-directory/conditional-access/)

---

## 9.8 Advanced Azure AD Scenarios (Expert Level)

### Hybrid Identity with Azure AD Connect

```powershell
# Azure AD Connect configuration (run on-premises)
# Password Hash Sync with Seamless SSO

# Verify sync status
Get-ADSyncScheduler

# Force delta sync
Start-ADSyncSyncCycle -PolicyType Delta

# Full sync (use carefully)
Start-ADSyncSyncCycle -PolicyType Initial

# Check connector status
Get-ADSyncConnectorRunStatus

# Export Azure AD Connect configuration
Get-ADSyncServerConfiguration -Path "C:\AADConnect\Config"
```

### Azure AD B2B (Business-to-Business)

```powershell
# Invite external users
$invitation = New-AzureADMSInvitation `
    -InvitedUserEmailAddress "partner@external.com" `
    -InvitedUserDisplayName "External Partner" `
    -InviteRedirectUrl "https://myapp.azurewebsites.net" `
    -SendInvitationMessage $true `
    -InvitedUserMessageInfo @{
        CustomizedMessageBody = "You have been invited to collaborate on Project X"
    }

# Bulk invite from CSV
$guests = Import-Csv "guests.csv"
foreach ($guest in $guests) {
    New-AzureADMSInvitation `
        -InvitedUserEmailAddress $guest.Email `
        -InvitedUserDisplayName $guest.Name `
        -InviteRedirectUrl "https://myportal.azurewebsites.net"
}

# Configure cross-tenant access settings
$crossTenantSettings = @{
    b2bCollaborationInbound = @{
        applications = @{
            accessType = "allowed"
            targets = @(@{ target = "Office365" })
        }
        usersAndGroups = @{
            accessType = "allowed"
            targets = @(@{ target = "AllUsers" })
        }
    }
}
```

### Azure AD B2C (Business-to-Consumer)

```xml
<!-- Custom policy for social login + MFA -->
<TrustFrameworkPolicy>
  <UserJourneys>
    <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp">
          <ClaimsProviderSelections>
            <ClaimsProviderSelection TargetClaimsExchangeId="GoogleExchange" />
            <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange" />
            <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmail" />
          </ClaimsProviderSelections>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
          <ClaimsExchanges>
            <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAuth2" />
            <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAuth2" />
          </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="3" Type="ClaimsExchange">
          <ClaimsExchange Id="PhoneFactor" TechnicalProfileReferenceId="PhoneFactor-InputOrVerify" />
        </OrchestrationStep>
        <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
      </OrchestrationSteps>
    </UserJourney>
  </UserJourneys>
</TrustFrameworkPolicy>
```

### Workload Identity Federation

```powershell
# Create app registration for GitHub Actions
$app = New-AzADApplication -DisplayName "GitHub-Actions-Deploy"

# Add federated credential for GitHub
$credential = @{
    name = "github-main-branch"
    issuer = "https://token.actions.githubusercontent.com"
    subject = "repo:myorg/myrepo:ref:refs/heads/main"
    audiences = @("api://AzureADTokenExchange")
}

New-AzADAppFederatedCredential `
    -ApplicationObjectId $app.Id `
    -Name $credential.name `
    -Issuer $credential.issuer `
    -Subject $credential.subject `
    -Audience $credential.audiences

# Create service principal and assign roles
$sp = New-AzADServicePrincipal -ApplicationId $app.AppId
New-AzRoleAssignment `
    -ObjectId $sp.Id `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/$subscriptionId/resourceGroups/Production-RG"
```

```yaml
# GitHub Actions workflow using workload identity
name: Deploy to Azure
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      
      - name: Deploy Bicep
        uses: azure/arm-deploy@v1
        with:
          resourceGroupName: Production-RG
          template: ./main.bicep
```

### Identity Governance

```powershell
# Create entitlement management access package
$catalog = Get-MgEntitlementManagementAccessPackageCatalog -Filter "displayName eq 'IT Resources'"

$accessPackage = New-MgEntitlementManagementAccessPackage `
    -DisplayName "Developer Resources" `
    -Description "Access to development tools and environments" `
    -CatalogId $catalog.Id `
    -IsHidden $false

# Add resource roles to access package
$resourceRole = @{
    originId = $devGroupId
    originSystem = "AadGroup"
    accessPackageResource = @{
        id = $resourceId
        resourceType = "AadGroup"
        originId = $devGroupId
        originSystem = "AadGroup"
    }
}

New-MgEntitlementManagementAccessPackageResourceRoleScope `
    -AccessPackageId $accessPackage.Id `
    -AccessPackageResourceRole $resourceRole

# Create approval policy
$policy = @{
    displayName = "Manager Approval Required"
    description = "Requires manager approval for access"
    requestorSettings = @{
        scopeType = "AllExistingDirectoryMemberUsers"
        acceptRequests = $true
    }
    requestApprovalSettings = @{
        isApprovalRequired = $true
        isApprovalRequiredForExtension = $false
        approvalStages = @(
            @{
                approvalStageTimeOutInDays = 14
                isApproverJustificationRequired = $true
                primaryApprovers = @(
                    @{
                        odatatype = "#microsoft.graph.singleUser"
                        userId = $managerId
                    }
                )
            }
        )
    }
    accessReviewSettings = @{
        isEnabled = $true
        recurrenceType = "quarterly"
        reviewerType = "Manager"
    }
}

New-MgEntitlementManagementAccessPackageAssignmentPolicy `
    -AccessPackageId $accessPackage.Id `
    -BodyParameter $policy
```
