# Module 08 - Authentication and Authorization in Azure using RBAC

## Learning Objectives
By the end of this module, you will be able to:
- Understand Identity and Access Management in Azure
- Implement Role-Based Access Control (RBAC)
- Create and manage role definitions
- Assign roles to users, groups, and service principals
- Manage Azure Users and Groups
- Implement RBAC policies

---

## 8.1 Identity and Access Management in Azure

### What is IAM?
Identity and Access Management (IAM) is the practice of ensuring the right people have appropriate access to technology resources. In Azure, IAM is implemented through Azure Active Directory and Role-Based Access Control.

#### 🏢 Real-World IAM and RBAC Use Cases:

**Enterprise Access Control Strategy:**
```
Company: FinanceBank (5,000 employees)
Challenge: Implement least-privilege access across Azure

Role Assignment Strategy:
┌─────────────────────────────────────────────────────────────────┐
│                    Management Group: FinanceBank                │
│                    Role: Security Reader (Security Team)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Subscription: Production                                 │   │
│  │ Role: Reader (All IT Staff) - view but can't modify     │   │
│  │                                                          │   │
│  │   ┌───────────────────────────────────────────────────┐ │   │
│  │   │ Resource Group: Prod-WebApp                       │ │   │
│  │   │ Role: Contributor (WebApp Team)                   │ │   │
│  │   │ Role: Website Contributor (Developers)            │ │   │
│  │   └───────────────────────────────────────────────────┘ │   │
│  │                                                          │   │
│  │   ┌───────────────────────────────────────────────────┐ │   │
│  │   │ Resource Group: Prod-Database                     │ │   │
│  │   │ Role: SQL DB Contributor (DBA Team only)          │ │   │
│  │   │ NO access for developers (sensitive data)         │ │   │
│  │   └───────────────────────────────────────────────────┘ │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Subscription: Development                                │   │
│  │ Role: Contributor (All Developers) - full dev access    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Security Audit Results:
- Before: 500 users had Owner access everywhere
- After: 50 users have scoped Contributor, rest have Reader
- Risk reduction: 90% fewer over-privileged accounts
```

**DevOps Pipeline Service Principal:**
```
Scenario: CI/CD pipeline needs to deploy to Azure

Security Challenge:
- Developers shouldn't have production access
- Pipeline needs automated deployment capability
- Follow principle of least privilege

Solution:
┌─────────────────────────────────────────────────────────────┐
│  Service Principal: "GitHub-Actions-Deploy"                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Role Assignments:                                           │
│  ├── Resource Group: Prod-WebApp                            │
│  │   └── Role: Website Contributor (deploy apps only)       │
│  │                                                           │
│  ├── Azure Container Registry                                │
│  │   └── Role: AcrPush (push images only)                   │
│  │                                                           │
│  └── Key Vault                                               │
│      └── Role: Key Vault Secrets User (read secrets)        │
│                                                              │
│  What SP CAN do:                                             │
│  ✓ Deploy web apps                                           │
│  ✓ Push container images                                     │
│  ✓ Read deployment secrets                                   │
│                                                              │
│  What SP CANNOT do:                                          │
│  ✗ Delete resources                                          │
│  ✗ Access databases                                          │
│  ✗ Modify networking                                         │
│  ✗ Change RBAC assignments                                   │
└─────────────────────────────────────────────────────────────┘
```

**Custom Role for Help Desk:**
```
Scenario: Help desk needs to restart VMs but not delete them

Custom Role Definition:
{
  "Name": "VM Operator",
  "Description": "Can start, stop, restart VMs but not delete",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/delete",
    "Microsoft.Compute/virtualMachines/write"
  ],
  "AssignableScopes": [
    "/subscriptions/xxx/resourceGroups/Production-VMs"
  ]
}

Result:
- Help desk can resolve "my VM is slow" tickets
- Cannot accidentally delete production VMs
- Audit log shows who restarted what and when
```

**Managed Identity for Applications:**
```
Scenario: Web app needs to access Key Vault and Storage

Traditional (Bad) Approach:
- Store connection strings in app config
- Rotate secrets manually
- Risk: Secrets in source control

Modern (Good) Approach - Managed Identity:
┌──────────────────────────────────────────────────────────────┐
│  Web App (System-Assigned Managed Identity)                  │
│  Identity ID: auto-generated, no credentials to manage       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────────────┐      ┌─────────────────────────┐    │
│  │    Key Vault       │      │    Storage Account      │    │
│  │                    │      │                         │    │
│  │  Role: Key Vault   │      │  Role: Storage Blob     │    │
│  │  Secrets User      │      │  Data Contributor       │    │
│  └────────────────────┘      └─────────────────────────┘    │
│                                                               │
│  Code (C#):                                                   │
│  var credential = new DefaultAzureCredential();               │
│  // No secrets needed! Uses managed identity automatically    │
│  var client = new SecretClient(vaultUri, credential);         │
│  var secret = await client.GetSecretAsync("db-password");     │
└──────────────────────────────────────────────────────────────┘

Benefits:
- Zero secrets in code or config
- Automatic credential rotation
- Works across Azure services
- Audit trail of access
```

### IAM Components in Azure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Azure Identity & Access Management               │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Azure Active Directory                      │  │
│  │                                                                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐   │  │
│  │  │   Users     │  │   Groups    │  │ Service Principals  │   │  │
│  │  │             │  │             │  │ (Applications)      │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘   │  │
│  │                           │                                   │  │
│  │                           ▼                                   │  │
│  │  ┌─────────────────────────────────────────────────────────┐ │  │
│  │  │              Authentication (Who are you?)               │ │  │
│  │  │   Passwords | MFA | Certificates | Tokens               │ │  │
│  │  └─────────────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                │                                     │
│                                ▼                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │         Role-Based Access Control (RBAC)                       │  │
│  │                  Authorization (What can you do?)              │  │
│  │                                                                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────────────────┐ │  │
│  │  │   Roles     │  │   Scopes    │  │  Role Assignments     │ │  │
│  │  │ (Permissions│  │ (Resources) │  │ (Who + Role + Scope)  │ │  │
│  │  └─────────────┘  └─────────────┘  └───────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Security Principals

A security principal is an object representing an entity requesting access to resources:

| Principal Type | Description |
|----------------|-------------|
| **User** | Individual with Azure AD account |
| **Group** | Collection of users |
| **Service Principal** | Identity for applications |
| **Managed Identity** | Auto-managed service principal |

### Scopes in Azure

RBAC can be assigned at different scope levels:

```
Management Group
      │
      └── Subscription
              │
              └── Resource Group
                      │
                      └── Resource
```

| Scope | Example |
|-------|---------|
| **Management Group** | /providers/Microsoft.Management/managementGroups/{mgId} |
| **Subscription** | /subscriptions/{subscriptionId} |
| **Resource Group** | /subscriptions/{subId}/resourceGroups/{rgName} |
| **Resource** | /subscriptions/{subId}/resourceGroups/{rgName}/providers/{provider}/{resourceType}/{resourceName} |

### Inheritance

Permissions are inherited from parent to child scopes:

```
Management Group (Owner role)
      │
      └── Subscription (Inherits Owner)
              │
              └── Resource Group (Inherits Owner)
                      │
                      └── VM (Inherits Owner)
```

---

## 8.2 Role-Based Access Management (RBAC)

### What is RBAC?
Role-Based Access Control (RBAC) provides fine-grained access management for Azure resources. It allows you to grant users, groups, and applications specific permissions.

### RBAC Components

| Component | Description |
|-----------|-------------|
| **Role Definition** | Collection of permissions |
| **Security Principal** | User, group, or service principal |
| **Scope** | Level where access applies |
| **Role Assignment** | Binding of role to principal at scope |

### How RBAC Works

```
Role Assignment = Security Principal + Role Definition + Scope

Example:
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│   John Doe      │ + │   Contributor   │ + │  Resource Group │
│   (User)        │   │   (Role)        │   │  (Scope)        │
└─────────────────┘   └─────────────────┘   └─────────────────┘
                              │
                              ▼
John Doe can manage all resources in the Resource Group
(but cannot manage access to it)
```

### Built-in Roles

| Role | Description | Actions |
|------|-------------|---------|
| **Owner** | Full access + can delegate | * |
| **Contributor** | Full access, no delegation | * (except authorization) |
| **Reader** | View only | */read |
| **User Access Administrator** | Manage user access | */Authorization/* |

### Common Built-in Roles

| Category | Role | Description |
|----------|------|-------------|
| **General** | Owner | Full access |
| | Contributor | Manage, no access control |
| | Reader | View only |
| **Compute** | Virtual Machine Contributor | Manage VMs |
| | Virtual Machine Administrator Login | Login as admin |
| **Network** | Network Contributor | Manage networking |
| **Storage** | Storage Account Contributor | Manage storage |
| | Storage Blob Data Contributor | Read/write blob data |
| **Database** | SQL DB Contributor | Manage SQL databases |
| **Security** | Security Admin | Manage security policies |
| | Security Reader | View security settings |

### Checking Role Assignments

```powershell
# List all role assignments for a resource group
Get-AzRoleAssignment -ResourceGroupName "MyRG"

# List role assignments for a specific user
Get-AzRoleAssignment -SignInName "user@company.com"

# List role assignments for a subscription
Get-AzRoleAssignment -Scope "/subscriptions/{subscriptionId}"
```

```bash
# Azure CLI
az role assignment list --resource-group "MyRG"
az role assignment list --assignee "user@company.com"
```

---

## 8.3 Role Definitions

### Role Definition Structure

```json
{
  "Name": "Custom Role Name",
  "Id": "unique-guid",
  "IsCustom": true,
  "Description": "Description of what this role does",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/write",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": [
    "Microsoft.Compute/virtualMachines/delete"
  ],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId}"
  ]
}
```

### Permission Types

| Type | Description | Example |
|------|-------------|---------|
| **Actions** | Management operations | Microsoft.Compute/virtualMachines/* |
| **NotActions** | Excluded from Actions | Microsoft.Compute/virtualMachines/delete |
| **DataActions** | Data operations | Microsoft.Storage/.../blobs/read |
| **NotDataActions** | Excluded data operations | Microsoft.Storage/.../blobs/delete |

### Action Format

```
{Provider}/{ResourceType}/{Operation}

Examples:
Microsoft.Compute/virtualMachines/read          - Read VMs
Microsoft.Compute/virtualMachines/write         - Create/Update VMs
Microsoft.Compute/virtualMachines/delete        - Delete VMs
Microsoft.Compute/virtualMachines/start/action  - Start VMs
Microsoft.Compute/virtualMachines/*             - All VM operations
*                                               - All operations
```

### Viewing Role Definitions

```powershell
# List all role definitions
Get-AzRoleDefinition | Select-Object Name, IsCustom

# Get specific role details
Get-AzRoleDefinition -Name "Contributor"

# List actions for a role
(Get-AzRoleDefinition -Name "Virtual Machine Contributor").Actions
```

```bash
# Azure CLI
az role definition list --output table
az role definition list --name "Contributor"
```

### Creating Custom Roles

```powershell
# Create custom role from JSON
$roleDefinition = @"
{
  "Name": "VM Operator",
  "Description": "Can start, stop, and restart VMs",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Network/networkInterfaces/read",
    "Microsoft.Network/publicIPAddresses/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/12345678-1234-1234-1234-123456789abc"
  ]
}
"@

$roleDefinition | Out-File -FilePath "VMOperator.json"
New-AzRoleDefinition -InputFile "VMOperator.json"
```

```bash
# Azure CLI
az role definition create --role-definition @VMOperator.json
```

### Custom Role Example: Storage Blob Reader

```json
{
  "Name": "Storage Blob Reader Custom",
  "Description": "Read access to blob storage data",
  "Actions": [
    "Microsoft.Storage/storageAccounts/read",
    "Microsoft.Storage/storageAccounts/blobServices/containers/read"
  ],
  "DataActions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

### Updating Custom Roles

```powershell
# Get existing role
$role = Get-AzRoleDefinition -Name "VM Operator"

# Modify permissions
$role.Actions.Add("Microsoft.Compute/virtualMachines/vmSizes/read")

# Update the role
Set-AzRoleDefinition -Role $role
```

### Deleting Custom Roles

```powershell
# Remove custom role
Remove-AzRoleDefinition -Name "VM Operator"
```

---

## 8.4 Role Assignment in Azure Resources

### Creating Role Assignments

```powershell
# Assign role to user
New-AzRoleAssignment `
  -SignInName "user@company.com" `
  -RoleDefinitionName "Virtual Machine Contributor" `
  -ResourceGroupName "Production-RG"

# Assign role to group
New-AzRoleAssignment `
  -ObjectId "group-object-id" `
  -RoleDefinitionName "Reader" `
  -Scope "/subscriptions/{subscription-id}"

# Assign role to service principal
New-AzRoleAssignment `
  -ApplicationId "app-id" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "App-RG"

# Assign role at management group level
New-AzRoleAssignment `
  -ObjectId "user-object-id" `
  -RoleDefinitionName "Reader" `
  -Scope "/providers/Microsoft.Management/managementGroups/mg-id"
```

```bash
# Azure CLI
az role assignment create \
  --assignee "user@company.com" \
  --role "Virtual Machine Contributor" \
  --resource-group "Production-RG"

az role assignment create \
  --assignee-object-id "group-id" \
  --role "Reader" \
  --scope "/subscriptions/{subscription-id}"
```

### Role Assignment at Different Scopes

```powershell
# Subscription scope
New-AzRoleAssignment `
  -SignInName "admin@company.com" `
  -RoleDefinitionName "Owner" `
  -Scope "/subscriptions/{sub-id}"

# Resource group scope
New-AzRoleAssignment `
  -SignInName "dev@company.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "Dev-RG"

# Resource scope
New-AzRoleAssignment `
  -SignInName "operator@company.com" `
  -RoleDefinitionName "Virtual Machine Contributor" `
  -Scope "/subscriptions/{sub-id}/resourceGroups/Prod-RG/providers/Microsoft.Compute/virtualMachines/WebServer"
```

### Viewing Role Assignments

```powershell
# All assignments in a resource group
Get-AzRoleAssignment -ResourceGroupName "Production-RG"

# Assignments for a specific user
Get-AzRoleAssignment -SignInName "user@company.com" -ExpandPrincipalGroups

# Assignments for current user
Get-AzRoleAssignment

# Filter by role
Get-AzRoleAssignment -RoleDefinitionName "Owner"
```

### Removing Role Assignments

```powershell
# Remove specific assignment
Remove-AzRoleAssignment `
  -SignInName "user@company.com" `
  -RoleDefinitionName "Contributor" `
  -ResourceGroupName "Production-RG"
```

---

## 8.5 Azure Users & Groups

### User Types in Azure AD

| Type | Description |
|------|-------------|
| **Member** | Users in your organization |
| **Guest** | External users (B2B) |
| **Synced** | Users synced from on-premises AD |

### Creating Users

```powershell
# Create Azure AD user
$passwordProfile = @{
    Password = "P@ssw0rd123!"
    ForceChangePasswordNextSignIn = $true
}

New-AzADUser `
  -DisplayName "John Doe" `
  -UserPrincipalName "john.doe@company.onmicrosoft.com" `
  -MailNickname "john.doe" `
  -PasswordProfile $passwordProfile `
  -AccountEnabled $true
```

```bash
# Azure CLI
az ad user create \
  --display-name "John Doe" \
  --user-principal-name "john.doe@company.onmicrosoft.com" \
  --password "P@ssw0rd123!" \
  --force-change-password-next-login
```

### Creating Groups

| Group Type | Description |
|------------|-------------|
| **Security** | Used for RBAC and resource access |
| **Microsoft 365** | Includes mailbox, SharePoint |

```powershell
# Create security group
New-AzADGroup `
  -DisplayName "IT Admins" `
  -MailNickname "ITAdmins" `
  -Description "IT Administrators group"
```

```bash
# Azure CLI
az ad group create \
  --display-name "IT Admins" \
  --mail-nickname "ITAdmins"
```

### Managing Group Membership

```powershell
# Add user to group
Add-AzADGroupMember `
  -TargetGroupObjectId "group-object-id" `
  -MemberObjectId "user-object-id"

# Remove user from group
Remove-AzADGroupMember `
  -GroupObjectId "group-object-id" `
  -MemberObjectId "user-object-id"

# List group members
Get-AzADGroupMember -GroupObjectId "group-object-id"
```

### Dynamic Groups

Groups with automatic membership based on rules:

```
Rule example: (user.department -eq "IT") -and (user.jobTitle -contains "Administrator")
```

---

## 8.6 RBAC Policies

### Azure Policy vs RBAC

| Feature | Azure Policy | RBAC |
|---------|-------------|------|
| Purpose | What resources can do | What users can do |
| Scope | Resource compliance | Access control |
| Example | "VMs must have encryption" | "User can create VMs" |

### Combining RBAC with Azure Policy

```
┌─────────────────────────────────────────────────────────────────┐
│                      Azure Governance                            │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                     RBAC                                 │   │
│  │        Who can perform what actions?                     │   │
│  │        - User A can create VMs                          │   │
│  │        - User B can only read resources                 │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              +                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   Azure Policy                           │   │
│  │        How can resources be configured?                  │   │
│  │        - VMs must be in East US                         │   │
│  │        - All resources must be tagged                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              =                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              Complete Governance                         │   │
│  │   User A can create VMs only in East US with tags       │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Deny Assignments

Deny assignments block users from performing specific actions even if role assignments grant access:

```
Priority: Deny > Allow

Example:
- User has Contributor role
- Deny assignment blocks VM deletion
- Result: User cannot delete VMs
```

```powershell
# View deny assignments (typically created by Azure Blueprints)
Get-AzDenyAssignment -ResourceGroupName "Protected-RG"
```

### Privileged Identity Management (PIM)

PIM provides just-in-time privileged access:

| Feature | Description |
|---------|-------------|
| **Eligible assignments** | Activate when needed |
| **Time-bound access** | Auto-expire after duration |
| **Approval workflow** | Require approval for activation |
| **Audit trail** | Track all privileged access |

### Best Practices for RBAC

| Practice | Description |
|----------|-------------|
| **Least privilege** | Grant minimum required permissions |
| **Use groups** | Assign roles to groups, not users |
| **Review regularly** | Audit role assignments periodically |
| **Use built-in roles** | Before creating custom roles |
| **Scope appropriately** | Apply at narrowest scope needed |
| **Document custom roles** | Maintain clear documentation |

---

## Hands-on Exercises

### Exercise 1: Create a Custom Role for Azure Resources

```powershell
# Create custom role: VM Reader with Start/Stop
$customRole = @"
{
  "Name": "VM Reader with Start Stop",
  "Description": "Can view VMs and start/stop them",
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/powerOff/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/instanceView/read",
    "Microsoft.Network/networkInterfaces/read",
    "Microsoft.Network/publicIPAddresses/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/$($(Get-AzContext).Subscription.Id)"
  ]
}
"@

$customRole | Out-File -FilePath "VMReaderStartStop.json" -Encoding UTF8
New-AzRoleDefinition -InputFile "VMReaderStartStop.json"

# Verify creation
Get-AzRoleDefinition -Name "VM Reader with Start Stop"
```

### Exercise 2: Assign a Role to Configure Access to Azure Resources

```powershell
# Create test user
$password = ConvertTo-SecureString "TempP@ssw0rd123!" -AsPlainText -Force
$user = New-AzADUser `
  -DisplayName "Test Operator" `
  -UserPrincipalName "testoperator@$($(Get-AzContext).Tenant.Id).onmicrosoft.com" `
  -Password $password `
  -AccountEnabled $true

# Create resource group for testing
New-AzResourceGroup -Name "RBAC-Test-RG" -Location "East US"

# Assign custom role to user
New-AzRoleAssignment `
  -ObjectId $user.Id `
  -RoleDefinitionName "VM Reader with Start Stop" `
  -ResourceGroupName "RBAC-Test-RG"

# Verify assignment
Get-AzRoleAssignment -ResourceGroupName "RBAC-Test-RG" | Select-Object DisplayName, RoleDefinitionName, Scope

# Create a group and assign role
$group = New-AzADGroup -DisplayName "VM Operators" -MailNickname "VMOperators"

New-AzRoleAssignment `
  -ObjectId $group.Id `
  -RoleDefinitionName "VM Reader with Start Stop" `
  -ResourceGroupName "RBAC-Test-RG"

# Add user to group
Add-AzADGroupMember -TargetGroupObjectId $group.Id -MemberObjectId $user.Id

# Test effective permissions
Get-AzRoleAssignment -SignInName "testoperator@company.onmicrosoft.com" -ExpandPrincipalGroups
```

### Complete RBAC Lab

```powershell
# 1. Create users
$users = @(
    @{Name="Alice Admin"; UPN="alice.admin"; Role="Owner"},
    @{Name="Bob Developer"; UPN="bob.dev"; Role="Contributor"},
    @{Name="Charlie Reader"; UPN="charlie.reader"; Role="Reader"}
)

foreach ($u in $users) {
    $pwd = ConvertTo-SecureString "P@ss$(Get-Random)!" -AsPlainText -Force
    $domain = (Get-AzContext).Tenant.Id
    $newUser = New-AzADUser `
        -DisplayName $u.Name `
        -UserPrincipalName "$($u.UPN)@$domain.onmicrosoft.com" `
        -Password $pwd `
        -AccountEnabled $true
    
    Write-Host "Created $($u.Name)"
}

# 2. Create groups
$groups = @("Admins", "Developers", "ReadOnlyUsers")
foreach ($g in $groups) {
    New-AzADGroup -DisplayName $g -MailNickname $g
    Write-Host "Created group: $g"
}

# 3. Assign roles to groups at subscription level
$sub = (Get-AzContext).Subscription.Id

$adminsGroup = Get-AzADGroup -DisplayName "Admins"
New-AzRoleAssignment -ObjectId $adminsGroup.Id -RoleDefinitionName "Owner" -Scope "/subscriptions/$sub"

$devsGroup = Get-AzADGroup -DisplayName "Developers"
New-AzRoleAssignment -ObjectId $devsGroup.Id -RoleDefinitionName "Contributor" -Scope "/subscriptions/$sub"

$readersGroup = Get-AzADGroup -DisplayName "ReadOnlyUsers"
New-AzRoleAssignment -ObjectId $readersGroup.Id -RoleDefinitionName "Reader" -Scope "/subscriptions/$sub"

# 4. Verify all assignments
Get-AzRoleAssignment | Where-Object {$_.Scope -eq "/subscriptions/$sub"} | 
    Select-Object DisplayName, RoleDefinitionName, Scope | Format-Table
```

---

## Summary

| Topic | Key Points |
|-------|------------|
| IAM | Authentication (who) + Authorization (what) |
| RBAC | Role assignments = Principal + Role + Scope |
| Role Definitions | Actions, NotActions, DataActions |
| Built-in Roles | Owner, Contributor, Reader + service-specific |
| Custom Roles | Create when built-in roles don't fit |
| Role Assignment | Can be at management group, subscription, RG, or resource |
| Users & Groups | Assign roles to groups for easier management |
| Best Practices | Least privilege, use groups, regular reviews |

---

## Review Questions

1. What are the three components of a role assignment?
2. What is the difference between Actions and DataActions?
3. When would you create a custom role instead of using built-in roles?
4. How does role inheritance work across scopes?
5. What is the difference between RBAC and Azure Policy?
6. Why should you assign roles to groups instead of users?

---

## Additional Resources

- [Azure RBAC Documentation](https://docs.microsoft.com/azure/role-based-access-control/)
- [Built-in Roles Reference](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles)
- [Custom Role Tutorial](https://docs.microsoft.com/azure/role-based-access-control/custom-roles-powershell)
- [Azure AD Documentation](https://docs.microsoft.com/azure/active-directory/)

---

## 8.6 Advanced RBAC and Identity Governance (Expert Level)

### Custom Role Creation

```powershell
# Create custom role for DevOps team
$role = @{
    Name = "DevOps Deployment Operator"
    Description = "Can deploy and manage applications but cannot modify infrastructure"
    Actions = @(
        "Microsoft.Web/sites/*",
        "Microsoft.Compute/virtualMachines/read",
        "Microsoft.Compute/virtualMachines/start/action",
        "Microsoft.Compute/virtualMachines/restart/action",
        "Microsoft.ContainerService/managedClusters/read",
        "Microsoft.ContainerService/managedClusters/listClusterUserCredential/action",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Insights/metrics/read",
        "Microsoft.Insights/logs/read"
    )
    NotActions = @(
        "Microsoft.Web/sites/config/write",
        "Microsoft.Web/sites/delete"
    )
    DataActions = @(
        "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
    )
    AssignableScopes = @(
        "/subscriptions/$subscriptionId/resourceGroups/Production-RG",
        "/subscriptions/$subscriptionId/resourceGroups/Staging-RG"
    )
}

$customRole = New-AzRoleDefinition -Role $role

# Assign custom role to Azure AD group
$group = Get-AzADGroup -DisplayName "DevOps-Team"
New-AzRoleAssignment `
    -ObjectId $group.Id `
    -RoleDefinitionId $customRole.Id `
    -Scope "/subscriptions/$subscriptionId/resourceGroups/Production-RG"
```

### Privileged Identity Management (PIM)

```powershell
# Activate PIM role (using Azure AD PowerShell)
Connect-AzureAD

# Get eligible role assignment
$eligibleRoles = Get-AzureADMSPrivilegedRoleAssignment `
    -ProviderId "aadRoles" `
    -ResourceId (Get-AzureADTenantDetail).ObjectId `
    -Filter "subjectId eq '$userId'"

# Activate role
$schedule = New-Object Microsoft.Open.MSGraph.Model.AzureADMSPrivilegedSchedule
$schedule.Type = "Once"
$schedule.StartDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ss.fffZ")
$schedule.Duration = "PT4H"  # 4 hours

Open-AzureADMSPrivilegedRoleAssignmentRequest `
    -ProviderId "aadRoles" `
    -ResourceId (Get-AzureADTenantDetail).ObjectId `
    -RoleDefinitionId $eligibleRoles[0].RoleDefinitionId `
    -SubjectId $userId `
    -Type "UserAdd" `
    -AssignmentState "Active" `
    -Schedule $schedule `
    -Reason "Emergency production deployment"
```

### Conditional Access Policies

```powershell
# Create Conditional Access policy for MFA on Azure Management
$conditions = @{
    Applications = @{
        IncludeApplications = @("797f4846-ba00-4fd7-ba43-dac1f8f63013")  # Azure Management
    }
    Users = @{
        IncludeUsers = @("All")
        ExcludeGroups = @($emergencyAccessGroup.Id)
    }
    Locations = @{
        IncludeLocations = @("All")
        ExcludeLocations = @($trustedLocationId)
    }
    SignInRiskLevels = @("medium", "high")
}

$grantControls = @{
    BuiltInControls = @("mfa")
    Operator = "OR"
}

$sessionControls = @{
    SignInFrequency = @{
        Value = 4
        Type = "hours"
        IsEnabled = $true
    }
}

New-AzureADMSConditionalAccessPolicy `
    -DisplayName "Require MFA for Azure Management" `
    -State "enabled" `
    -Conditions $conditions `
    -GrantControls $grantControls `
    -SessionControls $sessionControls
```

### Managed Identities

```powershell
# Enable system-assigned managed identity on VM
$vm = Get-AzVM -ResourceGroupName "VM-RG" -Name "AppServer"
$vm.Identity = @{ Type = "SystemAssigned" }
$vm | Update-AzVM

# Create user-assigned managed identity
$userIdentity = New-AzUserAssignedIdentity `
    -ResourceGroupName "Identity-RG" `
    -Name "AppServiceIdentity" `
    -Location "eastus"

# Assign roles to managed identity
New-AzRoleAssignment `
    -ObjectId $vm.Identity.PrincipalId `
    -RoleDefinitionName "Storage Blob Data Contributor" `
    -Scope $storageAccount.Id

New-AzRoleAssignment `
    -ObjectId $vm.Identity.PrincipalId `
    -RoleDefinitionName "Key Vault Secrets User" `
    -Scope $keyVault.ResourceId

# Use managed identity in code (no credentials needed!)
$token = Invoke-RestMethod `
    -Uri "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/" `
    -Headers @{ Metadata = "true" }

$authHeader = @{ Authorization = "Bearer $($token.access_token)" }
```

### Azure AD Access Reviews

```powershell
# Create access review for privileged roles
$params = @{
    displayName = "Quarterly Admin Access Review"
    scope = @{
        query = "/roleManagement/directory/roleAssignments?`$filter=roleDefinitionId eq '$globalAdminRoleId'"
        queryType = "MicrosoftGraph"
    }
    reviewers = @(
        @{
            query = "/users/$reviewerUserId"
            queryType = "MicrosoftGraph"
        }
    )
    settings = @{
        mailNotificationsEnabled = $true
        reminderNotificationsEnabled = $true
        justificationRequiredOnApproval = $true
        defaultDecisionEnabled = $true
        defaultDecision = "Deny"
        instanceDurationInDays = 14
        autoApplyDecisionsEnabled = $true
        recommendationsEnabled = $true
        recurrence = @{
            pattern = @{
                type = "absoluteMonthly"
                interval = 3
            }
            range = @{
                type = "noEnd"
                startDate = (Get-Date).ToString("yyyy-MM-dd")
            }
        }
    }
}

New-MgAccessReviewScheduleDefinition -BodyParameter $params
```

### Deny Assignments

```powershell
# View deny assignments (typically created by Azure Blueprints)
Get-AzDenyAssignment -Scope "/subscriptions/$subscriptionId"

# Deny assignments are used to block actions even if RBAC allows them
# Common use: Protecting Blueprint-deployed resources
```

### RBAC Best Practices Checklist

| Practice | Implementation |
|----------|---------------|
| Use groups | Assign roles to Azure AD groups, not users |
| Least privilege | Start minimal, add permissions as needed |
| Use built-in roles | Custom roles only when necessary |
| Regular reviews | Quarterly access reviews with PIM |
| Emergency access | Break-glass accounts with monitoring |
| Managed identities | For service-to-service auth |
| Conditional access | MFA for privileged operations |
