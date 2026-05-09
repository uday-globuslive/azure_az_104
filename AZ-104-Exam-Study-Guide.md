# AZ-104 Microsoft Azure Administrator - Complete Exam Guide

## 🎯 Purpose of This Guide
This is your comprehensive roadmap to passing the AZ-104 certification exam. It covers every exam topic with explanations, examples, commands, and interview preparation.

---

# EXAM OVERVIEW

## About the AZ-104 Certification

| Attribute | Details |
|-----------|---------|
| **Full Name** | Microsoft Azure Administrator Associate |
| **Exam Code** | AZ-104 |
| **Prerequisites** | None required (AZ-900 recommended) |
| **Number of Questions** | 40-60 questions |
| **Time** | 100 minutes (+ 20-30 min for NDA/survey) |
| **Passing Score** | 700/1000 (approximately 70%) |
| **Cost** | $165 USD |
| **Validity** | 1 year (requires renewal) |
| **Question Types** | Multiple choice, case studies, drag-drop, yes/no, labs |

## What This Certification Proves

Passing AZ-104 demonstrates you can:
- Manage Azure identities and governance
- Implement and manage storage
- Deploy and manage Azure compute resources
- Configure and manage virtual networking
- Monitor and backup Azure resources

---

# DOMAIN 1: MANAGE AZURE IDENTITIES AND GOVERNANCE (20-25%)

## 1.1 Manage Azure Active Directory Objects

### Azure AD User Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        AZURE AD USER TYPES                                   │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────┐              │
│  │ 1. CLOUD IDENTITY (Created directly in Azure AD)          │              │
│  │    • User: john@contoso.onmicrosoft.com                   │              │
│  │    • Password managed in Azure AD                          │              │
│  │    • Best for: Cloud-only organizations                   │              │
│  └───────────────────────────────────────────────────────────┘              │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────┐              │
│  │ 2. SYNCED IDENTITY (Synced from on-premises AD)           │              │
│  │    • User: john@contoso.com (synced via Azure AD Connect) │              │
│  │    • Password managed on-premises AD                       │              │
│  │    • Best for: Hybrid organizations                        │              │
│  └───────────────────────────────────────────────────────────┘              │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────┐              │
│  │ 3. GUEST USER (B2B - External users)                      │              │
│  │    • User: sarah@partner.com (invited)                    │              │
│  │    • Uses their own organization's credentials            │              │
│  │    • Best for: Collaborating with external partners       │              │
│  └───────────────────────────────────────────────────────────┘              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Users - PowerShell

```powershell
# Create a new user
$PasswordProfile = @{
    Password = "P@ssw0rd123!"
    ForceChangePasswordNextSignIn = $true
}

New-AzADUser `
    -DisplayName "John Smith" `
    -UserPrincipalName "john.smith@contoso.onmicrosoft.com" `
    -PasswordProfile $PasswordProfile `
    -MailNickname "johnsmith" `
    -AccountEnabled $true

# Get all users
Get-AzADUser

# Get specific user
Get-AzADUser -UserPrincipalName "john.smith@contoso.onmicrosoft.com"

# Delete a user
Remove-AzADUser -UserPrincipalName "john.smith@contoso.onmicrosoft.com"
```

### Create Users - Azure CLI

```bash
# Create a new user
az ad user create \
    --display-name "John Smith" \
    --user-principal-name "john.smith@contoso.onmicrosoft.com" \
    --password "P@ssw0rd123!" \
    --force-change-password-next-login true

# List all users
az ad user list --output table

# Get specific user
az ad user show --id "john.smith@contoso.onmicrosoft.com"

# Delete user
az ad user delete --id "john.smith@contoso.onmicrosoft.com"
```

### Azure AD Groups

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE AD GROUP TYPES                                 │
│                                                                              │
│  1. SECURITY GROUPS                                                          │
│     • Purpose: Manage access to resources                                   │
│     • Used for: RBAC assignments, application access                        │
│     • Members: Users, devices, service principals, other groups            │
│                                                                              │
│  2. MICROSOFT 365 GROUPS                                                     │
│     • Purpose: Collaboration                                                 │
│     • Used for: Shared mailbox, calendar, SharePoint, Teams                │
│     • Members: Users only                                                   │
│                                                                              │
│  MEMBERSHIP TYPES:                                                           │
│  ─────────────────                                                          │
│  • Assigned: Members added manually                                         │
│  • Dynamic User: Members added automatically based on attributes           │
│  • Dynamic Device: Devices added automatically (security groups only)      │
│                                                                              │
│  DYNAMIC MEMBERSHIP EXAMPLE:                                                │
│  Rule: (user.department -eq "Finance") and (user.jobTitle -contains "Manager")│
│  Result: All Finance Managers automatically added to the group             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Groups - Commands

```powershell
# Create a security group
New-AzADGroup -DisplayName "VM-Administrators" -MailNickname "vm-admins" -SecurityEnabled

# Add member to group
Add-AzADGroupMember -TargetGroupObjectId <GROUP-OBJECT-ID> -MemberObjectId <USER-OBJECT-ID>

# List group members
Get-AzADGroupMember -GroupObjectId <GROUP-OBJECT-ID>
```

```bash
# Create a security group
az ad group create --display-name "VM-Administrators" --mail-nickname "vm-admins"

# Add member to group
az ad group member add --group "VM-Administrators" --member-id <USER-OBJECT-ID>

# List group members
az ad group member list --group "VM-Administrators" --output table
```

---

## 1.2 Manage Azure Subscriptions and Governance

### Management Group Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AZURE MANAGEMENT HIERARCHY                                │
│                                                                              │
│                     ┌────────────────────────┐                              │
│                     │  ROOT MANAGEMENT GROUP  │                              │
│                     │  (Tenant level)         │                              │
│                     └───────────┬────────────┘                              │
│                                 │                                            │
│           ┌─────────────────────┼─────────────────────┐                     │
│           │                     │                     │                     │
│  ┌────────▼───────┐  ┌─────────▼─────────┐  ┌───────▼────────┐            │
│  │  Production MG │  │  Development MG   │  │   Sandbox MG   │            │
│  └────────┬───────┘  └─────────┬─────────┘  └───────┬────────┘            │
│           │                    │                     │                     │
│  ┌────────▼───────┐  ┌─────────▼─────────┐  ┌───────▼────────┐            │
│  │ Prod-Sub-East  │  │   Dev-Sub-1       │  │  Sandbox-Sub   │            │
│  │ Prod-Sub-West  │  │   Dev-Sub-2       │  │                │            │
│  └────────────────┘  └───────────────────┘  └────────────────┘            │
│                                                                              │
│  KEY POINTS:                                                                 │
│  • Up to 6 levels of management groups (excluding root)                     │
│  • Policies and RBAC applied at MG level inherit to all subscriptions      │
│  • 10,000 management groups per tenant                                      │
│  • Each subscription can only have ONE parent management group             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Azure Policy - Enforcing Rules

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE POLICY CONCEPTS                                │
│                                                                              │
│  WHAT IS AZURE POLICY?                                                       │
│  A service that creates, assigns, and manages rules that enforce            │
│  standards across your Azure resources.                                      │
│                                                                              │
│  POLICY EFFECTS:                                                             │
│  ───────────────                                                            │
│  • Deny: Blocks the resource creation/modification                          │
│  • Audit: Logs a warning but allows the action                             │
│  • AuditIfNotExists: Audits if a related resource doesn't exist            │
│  • DeployIfNotExists: Deploys a resource if it doesn't exist               │
│  • Disabled: Policy is not enforced                                         │
│  • Modify: Adds/removes tags or properties                                  │
│  • Append: Adds fields to resources (e.g., add required tags)              │
│                                                                              │
│  POLICY vs INITIATIVE:                                                       │
│  ─────────────────────                                                       │
│  Policy = Single rule                                                        │
│  Initiative = Collection of policies (policy set)                           │
│                                                                              │
│  EXAMPLE BUILT-IN POLICIES:                                                  │
│  • "Allowed locations" - Restrict regions where resources can be created    │
│  • "Allowed virtual machine SKUs" - Restrict VM sizes                      │
│  • "Require tag on resources" - Enforce tagging                            │
│  • "Not allowed resource types" - Block specific resource types            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Policy Assignment Commands

```powershell
# Get built-in policy definition
$Policy = Get-AzPolicyDefinition -Name "Allowed locations"

# Assign policy to resource group
New-AzPolicyAssignment `
    -Name "Restrict-Locations" `
    -PolicyDefinition $Policy `
    -Scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RG-NAME>" `
    -PolicyParameterObject @{"listOfAllowedLocations"=@("eastus","westus")}

# View policy assignments
Get-AzPolicyAssignment

# Check compliance
Get-AzPolicyState -SubscriptionId <SUBSCRIPTION-ID> | Where-Object {$_.ComplianceState -eq "NonCompliant"}
```

```bash
# Get policy definition
az policy definition list --query "[?contains(displayName, 'Allowed locations')]"

# Assign policy
az policy assignment create \
    --name "Restrict-Locations" \
    --policy "e56962a6-4747-49cd-b67b-bf8b01975c4c" \
    --scope "/subscriptions/<SUBSCRIPTION-ID>/resourceGroups/<RG-NAME>" \
    --params '{"listOfAllowedLocations":{"value":["eastus","westus"]}}'

# Check compliance
az policy state list --filter "complianceState eq 'NonCompliant'"
```

---

## 1.3 Manage Role-Based Access Control (RBAC)

### RBAC Deep Dive

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           RBAC COMPONENTS                                    │
│                                                                              │
│  1. SECURITY PRINCIPAL (WHO)                                                │
│     ├── User: john@company.com                                              │
│     ├── Group: "Developers" Azure AD group                                  │
│     ├── Service Principal: Application identity                            │
│     └── Managed Identity: Azure-managed identity for services              │
│                                                                              │
│  2. ROLE DEFINITION (WHAT)                                                  │
│     Built-in roles:                                                          │
│     ├── Owner: Full access + can assign roles                              │
│     ├── Contributor: Full access, cannot assign roles                      │
│     ├── Reader: View only                                                   │
│     ├── User Access Administrator: Manage access only                      │
│     └── Many more specialized roles                                         │
│                                                                              │
│     Custom roles:                                                            │
│     └── Define specific Actions and NotActions                             │
│                                                                              │
│  3. SCOPE (WHERE)                                                           │
│     ├── Management Group: /providers/Microsoft.Management/managementGroups/│
│     ├── Subscription: /subscriptions/{subscription-id}                     │
│     ├── Resource Group: .../resourceGroups/{resource-group}                │
│     └── Resource: .../providers/Microsoft.Compute/virtualMachines/{vm}     │
│                                                                              │
│  4. ROLE ASSIGNMENT = Principal + Role + Scope                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Built-in Roles Reference Table

| Role | Can Do | Common Use |
|------|--------|------------|
| **Owner** | Everything + manage access | Subscription owners |
| **Contributor** | Everything except manage access | Developers, Admins |
| **Reader** | View only | Auditors, Read-only users |
| **User Access Administrator** | Manage access only | Security team |
| **Virtual Machine Contributor** | Manage VMs | VM administrators |
| **Storage Account Contributor** | Manage storage accounts | Storage admins |
| **Storage Blob Data Contributor** | Read/write blob data | Applications |
| **Network Contributor** | Manage networks | Network team |
| **Security Admin** | Manage security policies | Security team |
| **Key Vault Administrator** | Manage Key Vault | Security team |

### RBAC Commands

```powershell
# List all role assignments in a subscription
Get-AzRoleAssignment

# List role assignments for a specific user
Get-AzRoleAssignment -SignInName "john@contoso.com"

# Assign a role
New-AzRoleAssignment `
    -SignInName "john@contoso.com" `
    -RoleDefinitionName "Virtual Machine Contributor" `
    -ResourceGroupName "VMs-RG"

# Remove a role assignment
Remove-AzRoleAssignment `
    -SignInName "john@contoso.com" `
    -RoleDefinitionName "Virtual Machine Contributor" `
    -ResourceGroupName "VMs-RG"

# List available role definitions
Get-AzRoleDefinition | Select-Object Name, Description | Sort-Object Name
```

```bash
# List all role assignments
az role assignment list --output table

# List role assignments for user
az role assignment list --assignee "john@contoso.com" --output table

# Assign a role
az role assignment create \
    --assignee "john@contoso.com" \
    --role "Virtual Machine Contributor" \
    --resource-group "VMs-RG"

# Remove role assignment
az role assignment delete \
    --assignee "john@contoso.com" \
    --role "Virtual Machine Contributor" \
    --resource-group "VMs-RG"

# List role definitions
az role definition list --output table
```

### Custom Role Example

```json
{
  "Name": "VM Operator Custom",
  "Description": "Can start and restart VMs but not create or delete them",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscription-id}"
  ]
}
```

```bash
# Create custom role from JSON
az role definition create --role-definition @vm-operator-custom.json
```

---

## 1.4 Resource Locks

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         RESOURCE LOCKS                                       │
│                                                                              │
│  PURPOSE: Prevent accidental deletion or modification of critical resources │
│                                                                              │
│  LOCK TYPES:                                                                 │
│  ────────────                                                               │
│  ┌────────────────┬──────────────────────────────────────────────────┐      │
│  │ Delete Lock    │ Prevent deletion. Can still modify resources.    │      │
│  │ (CanNotDelete) │ Example: Prevent accidental VM deletion         │      │
│  ├────────────────┼──────────────────────────────────────────────────┤      │
│  │ ReadOnly Lock  │ Prevent any modifications (like Reader role).   │      │
│  │                │ Example: Lock production database configuration │      │
│  └────────────────┴──────────────────────────────────────────────────┘      │
│                                                                              │
│  INHERITANCE:                                                                │
│  • Locks applied at higher scope inherit to child resources                 │
│  • Subscription lock → All RGs → All Resources                             │
│  • Resource Group lock → All Resources in that RG                          │
│                                                                              │
│  WHO CAN MANAGE LOCKS?                                                       │
│  • Owner and User Access Administrator roles                                │
│  • Custom roles with Microsoft.Authorization/locks/* action                 │
│                                                                              │
│  NOTE: Even Owner role CANNOT bypass a lock without removing it first       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Lock Commands

```powershell
# Create a delete lock on a resource group
New-AzResourceLock `
    -LockName "DoNotDelete" `
    -LockLevel CanNotDelete `
    -ResourceGroupName "Production-RG" `
    -LockNotes "Critical production resources"

# Create a read-only lock
New-AzResourceLock `
    -LockName "ReadOnlyLock" `
    -LockLevel ReadOnly `
    -ResourceGroupName "Production-RG"

# List all locks in a resource group
Get-AzResourceLock -ResourceGroupName "Production-RG"

# Remove a lock
Remove-AzResourceLock -LockName "DoNotDelete" -ResourceGroupName "Production-RG"
```

```bash
# Create a delete lock
az lock create \
    --name "DoNotDelete" \
    --lock-type CanNotDelete \
    --resource-group "Production-RG" \
    --notes "Critical production resources"

# List locks
az lock list --resource-group "Production-RG" --output table

# Delete a lock
az lock delete --name "DoNotDelete" --resource-group "Production-RG"
```

---

# DOMAIN 2: IMPLEMENT AND MANAGE STORAGE (15-20%)

## 2.1 Configure Storage Accounts

### Storage Account Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STORAGE ACCOUNT TYPES                                     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ StorageV2 (General-purpose v2) - RECOMMENDED                       │     │
│  │ • Supports all storage services (Blob, File, Queue, Table)        │     │
│  │ • Supports all access tiers (Hot, Cool, Archive)                  │     │
│  │ • Lowest per-GB prices                                             │     │
│  │ • USE FOR: Almost everything                                       │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ BlobStorage (Legacy)                                               │     │
│  │ • Only blob storage                                                │     │
│  │ • USE FOR: Don't use for new deployments                          │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ FileStorage (Premium)                                              │     │
│  │ • Only file shares                                                 │     │
│  │ • SSD-based, high performance                                     │     │
│  │ • USE FOR: Enterprise file shares, high IOPS requirements         │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ BlockBlobStorage (Premium)                                         │     │
│  │ • Only block/append blobs                                          │     │
│  │ • SSD-based, high performance                                     │     │
│  │ • USE FOR: High-transaction workloads, low latency                │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Storage Redundancy Options (CRITICAL FOR EXAM!)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  STORAGE REDUNDANCY COMPARISON                               │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │   LRS    │  Locally Redundant Storage                                  │ │
│  │          │  • 3 copies in ONE data center                              │ │
│  │          │  • Protects: Hardware failures                              │ │
│  │          │  • Does NOT protect: Data center disasters                  │ │
│  │          │  • Durability: 99.999999999% (11 9's)                       │ │
│  │          │  • Cost: Lowest                                              │ │
│  │          │  • Use: Dev/test, non-critical, easily re-creatable data    │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │   ZRS    │  Zone Redundant Storage                                     │ │
│  │          │  • 3 copies across 3 Availability Zones                     │ │
│  │          │  • Protects: Data center disasters within region            │ │
│  │          │  • Durability: 99.9999999999% (12 9's)                      │ │
│  │          │  • Use: High availability, single-region production         │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │   GRS    │  Geo-Redundant Storage                                      │ │
│  │          │  • LRS in primary + LRS in secondary region (paired)        │ │
│  │          │  • 6 copies total (3 + 3)                                   │ │
│  │          │  • Protects: Regional disasters                             │ │
│  │          │  • Read access to secondary: Only during failover           │ │
│  │          │  • Durability: 99.99999999999999% (16 9's)                  │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │  RA-GRS  │  Read-Access Geo-Redundant Storage                          │ │
│  │          │  • Same as GRS + READ access to secondary region            │ │
│  │          │  • Can read from secondary anytime (without failover)       │ │
│  │          │  • Use: Read-heavy workloads needing DR                     │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │   GZRS   │  Geo-Zone-Redundant Storage                                 │ │
│  │          │  • ZRS in primary + LRS in secondary                        │ │
│  │          │  • Best of both: Zone + Region protection                   │ │
│  │          │  • Use: Mission-critical data                               │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌──────────┬─────────────────────────────────────────────────────────────┐ │
│  │ RA-GZRS  │  Read-Access Geo-Zone-Redundant Storage                     │ │
│  │          │  • GZRS + Read access to secondary                          │ │
│  │          │  • Highest durability and availability                      │ │
│  │          │  • Use: Most critical, globally distributed apps            │ │
│  └──────────┴─────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Storage Account Commands

```powershell
# Create a storage account (Standard, LRS)
New-AzStorageAccount `
    -ResourceGroupName "Storage-RG" `
    -Name "mystorageaccount12345" `
    -Location "eastus" `
    -SkuName "Standard_LRS" `
    -Kind "StorageV2" `
    -AccessTier "Hot"

# Create with GRS redundancy
New-AzStorageAccount `
    -ResourceGroupName "Storage-RG" `
    -Name "mystorageaccountgrs" `
    -Location "eastus" `
    -SkuName "Standard_GRS" `
    -Kind "StorageV2"

# Get storage account keys
Get-AzStorageAccountKey -ResourceGroupName "Storage-RG" -Name "mystorageaccount12345"
```

```bash
# Create storage account
az storage account create \
    --name "mystorageaccount12345" \
    --resource-group "Storage-RG" \
    --location "eastus" \
    --sku "Standard_LRS" \
    --kind "StorageV2" \
    --access-tier "Hot"

# Get storage account keys
az storage account keys list \
    --account-name "mystorageaccount12345" \
    --resource-group "Storage-RG" \
    --output table
```

---

## 2.2 Configure Azure Blob Storage

### Blob Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         BLOB TYPES                                           │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ BLOCK BLOB (Most common)                                           │     │
│  │ • Optimized for uploads and downloads                             │     │
│  │ • Made up of blocks (up to 190.7 TB per blob)                     │     │
│  │ • Each block: up to 4000 MB                                        │     │
│  │ • Use for: Files, documents, images, videos                       │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ APPEND BLOB                                                        │     │
│  │ • Optimized for append operations                                  │     │
│  │ • Cannot modify existing blocks                                    │     │
│  │ • Use for: Logging, audit trails, streaming data                  │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ PAGE BLOB                                                          │     │
│  │ • Optimized for random read/write operations                      │     │
│  │ • Used for Azure VM disks (VHD files)                             │     │
│  │ • Up to 8 TB per blob                                              │     │
│  │ • Use for: Virtual machine disks                                   │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Access Tiers (Cost Optimization)

| Tier | Storage Cost | Access Cost | Min Retention | Use Case |
|------|-------------|-------------|---------------|----------|
| **Hot** | Highest | Lowest | None | Frequently accessed data |
| **Cool** | Lower | Higher | 30 days | Infrequently accessed, 30+ days |
| **Cold** | Even Lower | Even Higher | 90 days | Rarely accessed, 90+ days |
| **Archive** | Lowest | Highest | 180 days | Rarely accessed, hours to rehydrate |

### Blob Container Access Levels

| Level | Description |
|-------|-------------|
| **Private** (default) | No anonymous access. Requires authentication. |
| **Blob** | Anonymous read access to blobs only |
| **Container** | Anonymous read + list access to entire container |

### Blob Commands

```powershell
# Get storage context
$storageAccount = Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount12345"
$ctx = $storageAccount.Context

# Create a container
New-AzStorageContainer -Name "mycontainer" -Context $ctx -Permission Off

# Upload a blob
Set-AzStorageBlobContent `
    -Container "mycontainer" `
    -File "C:\localfile.txt" `
    -Blob "remotefile.txt" `
    -Context $ctx

# Download a blob
Get-AzStorageBlobContent `
    -Container "mycontainer" `
    -Blob "remotefile.txt" `
    -Destination "C:\downloads\" `
    -Context $ctx

# List blobs
Get-AzStorageBlob -Container "mycontainer" -Context $ctx

# Set blob tier to Cool
Set-AzStorageBlobTier -Container "mycontainer" -Blob "remotefile.txt" -Tier Cool -Context $ctx
```

```bash
# Create container
az storage container create \
    --name "mycontainer" \
    --account-name "mystorageaccount12345" \
    --public-access off

# Upload blob
az storage blob upload \
    --account-name "mystorageaccount12345" \
    --container-name "mycontainer" \
    --name "remotefile.txt" \
    --file "localfile.txt"

# Download blob
az storage blob download \
    --account-name "mystorageaccount12345" \
    --container-name "mycontainer" \
    --name "remotefile.txt" \
    --file "downloaded.txt"

# List blobs
az storage blob list \
    --account-name "mystorageaccount12345" \
    --container-name "mycontainer" \
    --output table

# Set blob tier
az storage blob set-tier \
    --account-name "mystorageaccount12345" \
    --container-name "mycontainer" \
    --name "remotefile.txt" \
    --tier "Cool"
```

---

## 2.3 Configure Storage Security

### Shared Access Signatures (SAS)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SHARED ACCESS SIGNATURE (SAS)                             │
│                                                                              │
│  WHAT IS SAS?                                                                │
│  A URI that grants limited access to storage resources without exposing     │
│  account keys. Includes permissions and validity period.                    │
│                                                                              │
│  SAS TYPES:                                                                  │
│  ───────────                                                                │
│  1. User Delegation SAS (Most secure)                                       │
│     • Signed with Azure AD credentials                                      │
│     • Only for Blob and Queue storage                                       │
│     • Best practice for production                                          │
│                                                                              │
│  2. Service SAS                                                              │
│     • Signed with storage account key                                       │
│     • Access to specific service (Blob, Queue, Table, File)                │
│     • Grants access to one or more resources in a single service           │
│                                                                              │
│  3. Account SAS                                                              │
│     • Signed with storage account key                                       │
│     • Access to multiple services                                           │
│     • Can delegate access to service-level operations                      │
│                                                                              │
│  SAS URL EXAMPLE:                                                            │
│  https://myaccount.blob.core.windows.net/photos/puppy.jpg                   │
│  ?sv=2021-06-08        ← API version                                        │
│  &ss=b                 ← Services (b=blob, f=file, q=queue, t=table)        │
│  &srt=o                ← Resource types (s=service, c=container, o=object) │
│  &sp=r                 ← Permissions (r=read, w=write, d=delete, etc.)     │
│  &se=2024-01-01T00:00Z ← Expiry time                                        │
│  &sig=xyz123...        ← Signature                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Generate SAS Token

```powershell
# Generate Account SAS
$ctx = (Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount").Context

$sasToken = New-AzStorageAccountSASToken `
    -Context $ctx `
    -Service Blob,File `
    -ResourceType Service,Container,Object `
    -Permission "rl" `
    -ExpiryTime (Get-Date).AddDays(7)

# Generate Service SAS for a container
$containerSAS = New-AzStorageContainerSASToken `
    -Name "mycontainer" `
    -Context $ctx `
    -Permission "rl" `
    -ExpiryTime (Get-Date).AddHours(2)
```

```bash
# Generate Account SAS
az storage account generate-sas \
    --account-name "mystorageaccount" \
    --services "bf" \
    --resource-types "sco" \
    --permissions "rl" \
    --expiry "2024-12-31T23:59:59Z"

# Generate Container SAS
az storage container generate-sas \
    --account-name "mystorageaccount" \
    --name "mycontainer" \
    --permissions "rl" \
    --expiry "2024-12-31T23:59:59Z"
```

### Stored Access Policies

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STORED ACCESS POLICIES                                    │
│                                                                              │
│  PROBLEM: If you share a SAS token and need to revoke it, you can't!       │
│  (Unless you regenerate the account key, which breaks ALL SAS tokens)      │
│                                                                              │
│  SOLUTION: Stored Access Policies                                            │
│  • Define policy on the container/queue/table/share                         │
│  • SAS tokens reference the policy                                          │
│  • Modify or delete policy = immediate effect on all linked SAS tokens     │
│                                                                              │
│  MAX 5 POLICIES per container/queue/table/share                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

```powershell
# Create stored access policy
New-AzStorageContainerStoredAccessPolicy `
    -Container "mycontainer" `
    -Policy "mypolicy" `
    -Permission "rl" `
    -ExpiryTime (Get-Date).AddDays(30) `
    -Context $ctx

# Generate SAS using stored policy
New-AzStorageContainerSASToken `
    -Name "mycontainer" `
    -Policy "mypolicy" `
    -Context $ctx
```

---

## 2.4 Azure Files

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE FILES                                          │
│                                                                              │
│  WHAT IS IT?                                                                 │
│  Fully managed file shares in the cloud accessible via SMB and NFS.        │
│                                                                              │
│  PROTOCOLS:                                                                  │
│  • SMB 3.x: Windows, macOS, Linux (port 445)                               │
│  • NFS 4.1: Linux (premium tier only)                                       │
│  • REST API: Any platform                                                    │
│                                                                              │
│  KEY FEATURES:                                                               │
│  • Mount on Windows: net use Z: \\storageaccount.file.core.windows.net\share│
│  • Mount on Linux: mount -t cifs                                            │
│  • Snapshots: Point-in-time copies                                          │
│  • Soft delete: Recover accidentally deleted shares                         │
│  • Azure File Sync: Cache on-premises, sync to cloud                        │
│                                                                              │
│  SHARE SIZE LIMITS:                                                          │
│  • Standard: 5 TiB (default), 100 TiB (with large file shares enabled)     │
│  • Premium: 100 TiB                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Azure Files Commands

```powershell
# Create file share
New-AzStorageShare -Name "myshare" -Context $ctx -QuotaGiB 100

# Get connection string for mounting
Get-AzStorageAccount -ResourceGroupName "Storage-RG" -Name "mystorageaccount" | 
    Get-AzStorageAccountKey | 
    Select-Object -First 1

# Upload file to share
Set-AzStorageFileContent `
    -ShareName "myshare" `
    -Source "C:\localfile.txt" `
    -Path "remotefile.txt" `
    -Context $ctx
```

```bash
# Create file share
az storage share create \
    --name "myshare" \
    --account-name "mystorageaccount" \
    --quota 100

# List shares
az storage share list --account-name "mystorageaccount" --output table
```

### Mount Azure File Share on Windows

```powershell
# Mount as Z: drive
$connectTestResult = Test-NetConnection -ComputerName mystorageaccount.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded) {
    cmd.exe /C "net use Z: \\mystorageaccount.file.core.windows.net\myshare /user:Azure\mystorageaccount <STORAGE-ACCOUNT-KEY>"
} else {
    Write-Error "Port 445 is blocked"
}
```

---

# DOMAIN 3: DEPLOY AND MANAGE COMPUTE RESOURCES (20-25%)

## 3.1 Automate Deployment Using ARM Templates

### ARM Template Structure

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
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    }
  },
  
  "variables": {
    "nicName": "[concat(parameters('vmName'), '-nic')]",
    "vnetName": "[concat(parameters('vmName'), '-vnet')]"
  },
  
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "[variables('vnetName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "default",
            "properties": {
              "addressPrefix": "10.0.0.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-07-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_D2s_v3"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ],
  
  "outputs": {
    "vmResourceId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

### Deploy ARM Templates

```powershell
# Deploy ARM template
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "azuredeploy.json" `
    -TemplateParameterFile "azuredeploy.parameters.json"

# Deploy with inline parameters
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "azuredeploy.json" `
    -vmName "myVM" `
    -adminUsername "azureuser"

# What-If deployment (preview changes)
New-AzResourceGroupDeployment `
    -ResourceGroupName "MyRG" `
    -TemplateFile "azuredeploy.json" `
    -WhatIf
```

```bash
# Deploy ARM template
az deployment group create \
    --resource-group "MyRG" \
    --template-file "azuredeploy.json" \
    --parameters @azuredeploy.parameters.json

# What-If deployment
az deployment group what-if \
    --resource-group "MyRG" \
    --template-file "azuredeploy.json"
```

---

## 3.2 Configure Virtual Machines

### VM Availability Options

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   VM AVAILABILITY OPTIONS                                    │
│                                                                              │
│  1. AVAILABILITY SETS                                                        │
│  ────────────────────                                                       │
│  • Logical grouping within a single data center                             │
│  • Fault Domains: Separate physical racks (power, network)                  │
│  • Update Domains: Separate update groups (not rebooted together)          │
│  • Max 3 Fault Domains, 20 Update Domains                                   │
│  • SLA: 99.95% (2 or more VMs)                                              │
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────┐       │
│  │ Data Center                                                       │       │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐                  │       │
│  │  │   FD 0     │  │   FD 1     │  │   FD 2     │                  │       │
│  │  │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │                  │       │
│  │  │ │  VM1   │ │  │ │  VM2   │ │  │ │  VM3   │ │                  │       │
│  │  │ └────────┘ │  │ └────────┘ │  │ └────────┘ │                  │       │
│  │  └────────────┘  └────────────┘  └────────────┘                  │       │
│  └──────────────────────────────────────────────────────────────────┘       │
│                                                                              │
│  2. AVAILABILITY ZONES                                                       │
│  ──────────────────────                                                      │
│  • Physically separate data centers within a region                         │
│  • Each zone has independent power, cooling, networking                     │
│  • Min 3 zones per enabled region                                           │
│  • SLA: 99.99% (2 or more VMs across zones)                                 │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ Region (e.g., East US 2)                                           │     │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐                   │     │
│  │  │   Zone 1   │  │   Zone 2   │  │   Zone 3   │                   │     │
│  │  │   (DC A)   │  │   (DC B)   │  │   (DC C)   │                   │     │
│  │  │ ┌────────┐ │  │ ┌────────┐ │  │ ┌────────┐ │                   │     │
│  │  │ │  VM1   │ │  │ │  VM2   │ │  │ │  VM3   │ │                   │     │
│  │  │ └────────┘ │  │ └────────┘ │  │ └────────┘ │                   │     │
│  │  └────────────┘  └────────────┘  └────────────┘                   │     │
│  └────────────────────────────────────────────────────────────────────┘     │
│                                                                              │
│  3. VIRTUAL MACHINE SCALE SETS (VMSS)                                        │
│  ────────────────────────────────────                                        │
│  • Group of identical VMs                                                    │
│  • Auto-scale based on demand (CPU, memory, schedule)                       │
│  • Spread across Fault Domains and Availability Zones                       │
│  • Up to 1000 VMs per scale set (100 for custom images)                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create VMs with Availability

```powershell
# Create Availability Set
New-AzAvailabilitySet `
    -ResourceGroupName "MyRG" `
    -Name "WebServers-AS" `
    -Location "eastus" `
    -PlatformFaultDomainCount 3 `
    -PlatformUpdateDomainCount 5 `
    -Sku "Aligned"

# Create VM in Availability Set
New-AzVM `
    -ResourceGroupName "MyRG" `
    -Name "WebVM1" `
    -Location "eastus" `
    -AvailabilitySetName "WebServers-AS" `
    -Image "Win2019Datacenter" `
    -Size "Standard_D2s_v3"

# Create VM in Availability Zone
New-AzVM `
    -ResourceGroupName "MyRG" `
    -Name "WebVM1" `
    -Location "eastus" `
    -Zone 1 `
    -Image "Win2019Datacenter"
```

### VM Scale Set Commands

```powershell
# Create VMSS
New-AzVmss `
    -ResourceGroupName "MyRG" `
    -VMScaleSetName "WebVMSS" `
    -Location "eastus" `
    -VirtualNetworkName "MyVNet" `
    -SubnetName "default" `
    -PublicIpAddressName "WebVMSS-PIP" `
    -LoadBalancerName "WebVMSS-LB" `
    -UpgradePolicyMode "Automatic" `
    -InstanceCount 2

# Scale manually
Update-AzVmss `
    -ResourceGroupName "MyRG" `
    -VMScaleSetName "WebVMSS" `
    -InstanceCount 5

# Get VMSS instances
Get-AzVmssVM -ResourceGroupName "MyRG" -VMScaleSetName "WebVMSS"
```

```bash
# Create VMSS
az vmss create \
    --resource-group "MyRG" \
    --name "WebVMSS" \
    --image "UbuntuLTS" \
    --upgrade-policy-mode automatic \
    --admin-username azureuser \
    --generate-ssh-keys \
    --instance-count 2

# Scale VMSS
az vmss scale \
    --resource-group "MyRG" \
    --name "WebVMSS" \
    --new-capacity 5
```

---

## 3.3 Configure Azure App Service

### App Service Plans

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    APP SERVICE PLAN TIERS                                    │
│                                                                              │
│  ┌───────────┬────────────────────────────────────────────────────────┐     │
│  │   FREE    │ F1: Shared infrastructure, 60 min/day compute, no SLA  │     │
│  ├───────────┼────────────────────────────────────────────────────────┤     │
│  │  SHARED   │ D1: Shared infrastructure, custom domains, no SLA      │     │
│  ├───────────┼────────────────────────────────────────────────────────┤     │
│  │   BASIC   │ B1/B2/B3: Dedicated VMs, custom domains, SSL           │     │
│  │           │ Manual scale up to 3 instances                         │     │
│  ├───────────┼────────────────────────────────────────────────────────┤     │
│  │ STANDARD  │ S1/S2/S3: Auto-scale (up to 10 instances)              │     │
│  │           │ Staging slots, daily backups, Traffic Manager          │     │
│  │           │ 99.95% SLA                                              │     │
│  ├───────────┼────────────────────────────────────────────────────────┤     │
│  │  PREMIUM  │ P1v2/P2v2/P3v2, P1v3/P2v3/P3v3                         │     │
│  │           │ Auto-scale (up to 30 instances), more memory/CPU       │     │
│  │           │ VNet integration, Private endpoints                    │     │
│  ├───────────┼────────────────────────────────────────────────────────┤     │
│  │ ISOLATED  │ I1/I2/I3, I1v2/I2v2/I3v2                               │     │
│  │           │ App Service Environment (ASE)                          │     │
│  │           │ Dedicated VNet, 100 instances, highest scale           │     │
│  │           │ 99.95% SLA                                              │     │
│  └───────────┴────────────────────────────────────────────────────────┘     │
│                                                                              │
│  KEY CONCEPTS:                                                               │
│  • App Service Plan = The compute resources (size + scale)                  │
│  • Multiple web apps can share the same plan                                │
│  • Scale UP = Change tier (more CPU/RAM)                                    │
│  • Scale OUT = Add more instances (horizontal)                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Deployment Slots

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     DEPLOYMENT SLOTS                                         │
│                                                                              │
│  WHAT ARE THEY?                                                              │
│  Live app instances with their own hostnames, used for staging/testing.    │
│                                                                              │
│  WORKFLOW:                                                                   │
│  1. Deploy new code to staging slot                                         │
│  2. Test and validate                                                        │
│  3. Swap staging with production (zero downtime)                            │
│  4. If issues occur, swap back immediately                                  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────┐         │
│  │ Production Slot                  Staging Slot                  │         │
│  │ myapp.azurewebsites.net          myapp-staging.azurewebsites.net│         │
│  │                                                                │         │
│  │   Version 1.0              ←SWAP→    Version 1.1              │         │
│  │   (current users)                    (testing)                │         │
│  └────────────────────────────────────────────────────────────────┘         │
│                                                                              │
│  SLOT-SPECIFIC SETTINGS:                                                     │
│  Settings that stay with the slot (not swapped):                            │
│  • Connection strings (marked as slot setting)                              │
│  • App settings (marked as slot setting)                                    │
│  • Custom domain bindings                                                    │
│  • SSL certificates                                                          │
│                                                                              │
│  AVAILABILITY:                                                               │
│  • Standard tier: 5 slots                                                    │
│  • Premium/Isolated: 20 slots                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### App Service Commands

```powershell
# Create App Service Plan
New-AzAppServicePlan `
    -Name "MyAppServicePlan" `
    -Location "eastus" `
    -ResourceGroupName "MyRG" `
    -Tier "Standard" `
    -WorkerSize "Small"

# Create Web App
New-AzWebApp `
    -Name "myuniquewebapp12345" `
    -Location "eastus" `
    -ResourceGroupName "MyRG" `
    -AppServicePlan "MyAppServicePlan"

# Create deployment slot
New-AzWebAppSlot `
    -Name "myuniquewebapp12345" `
    -ResourceGroupName "MyRG" `
    -Slot "staging"

# Swap slots
Switch-AzWebAppSlot `
    -Name "myuniquewebapp12345" `
    -ResourceGroupName "MyRG" `
    -SourceSlotName "staging" `
    -DestinationSlotName "production"
```

```bash
# Create App Service Plan
az appservice plan create \
    --name "MyAppServicePlan" \
    --resource-group "MyRG" \
    --sku "S1" \
    --location "eastus"

# Create Web App
az webapp create \
    --name "myuniquewebapp12345" \
    --resource-group "MyRG" \
    --plan "MyAppServicePlan"

# Create deployment slot
az webapp deployment slot create \
    --name "myuniquewebapp12345" \
    --resource-group "MyRG" \
    --slot "staging"

# Swap slots
az webapp deployment slot swap \
    --name "myuniquewebapp12345" \
    --resource-group "MyRG" \
    --slot "staging" \
    --target-slot "production"
```

---

# DOMAIN 4: CONFIGURE AND MANAGE VIRTUAL NETWORKING (20-25%)

## 4.1 Configure Virtual Networks

### VNet Architecture Review

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      VNET COMPONENTS                                         │
│                                                                              │
│  1. ADDRESS SPACE                                                            │
│     • Private IP range (RFC 1918): 10.x.x.x, 172.16.x.x - 172.31.x.x,       │
│       192.168.x.x                                                            │
│     • Cannot overlap with on-premises if connecting via VPN/ExpressRoute    │
│     • CIDR notation: 10.0.0.0/16 = 65,536 IPs                               │
│                                                                              │
│  2. SUBNETS                                                                  │
│     • Subdivide address space                                                │
│     • Azure reserves 5 IPs per subnet (first 4 + last 1)                    │
│     • /24 subnet = 251 usable IPs (not 256)                                 │
│                                                                              │
│  3. SPECIAL SUBNETS                                                          │
│     • GatewaySubnet: Required for VPN/ExpressRoute Gateway                  │
│     • AzureFirewallSubnet: Required for Azure Firewall (/26 minimum)        │
│     • AzureBastionSubnet: Required for Azure Bastion (/27 minimum)          │
│                                                                              │
│  4. DNS                                                                      │
│     • Default: Azure-provided DNS (168.63.129.16)                           │
│     • Custom: Specify your own DNS servers                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### VNet Peering

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         VNET PEERING                                         │
│                                                                              │
│  WHAT IS IT?                                                                 │
│  Connect two VNets so resources can communicate using private IPs.         │
│  Traffic stays on Microsoft backbone (never goes to internet).             │
│                                                                              │
│  TYPES:                                                                      │
│  ─────────                                                                  │
│  • Regional VNet Peering: VNets in the same region                          │
│  • Global VNet Peering: VNets in different regions                          │
│                                                                              │
│  KEY PROPERTIES:                                                             │
│  ────────────────                                                           │
│  • Non-transitive: If A↔B and B↔C, A cannot talk to C (unless A↔C too)     │
│  • Requires peering in BOTH directions (bidirectional setup)               │
│  • Address spaces cannot overlap                                            │
│  • Low latency, high bandwidth                                              │
│                                                                              │
│  CONFIGURATION OPTIONS:                                                      │
│  • Allow forwarded traffic: Accept traffic forwarded by NVA                 │
│  • Allow gateway transit: Let peered VNet use your VPN gateway             │
│  • Use remote gateway: Use the peered VNet's VPN gateway                   │
│                                                                              │
│  ┌─────────────┐                          ┌─────────────┐                   │
│  │   VNet A    │◄──── Peering Link ────►│   VNet B    │                   │
│  │ 10.0.0.0/16 │                          │ 10.1.0.0/16 │                   │
│  │             │                          │             │                   │
│  │ ┌─────────┐ │                          │ ┌─────────┐ │                   │
│  │ │   VM1   │─┼──────────────────────────┼─│   VM2   │ │                   │
│  │ │10.0.1.4 │ │    Private IP routing    │ │10.1.1.4 │ │                   │
│  │ └─────────┘ │                          │ └─────────┘ │                   │
│  └─────────────┘                          └─────────────┘                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create VNet Peering

```powershell
# Get VNet references
$vnetA = Get-AzVirtualNetwork -Name "VNetA" -ResourceGroupName "RG-A"
$vnetB = Get-AzVirtualNetwork -Name "VNetB" -ResourceGroupName "RG-B"

# Create peering from A to B
Add-AzVirtualNetworkPeering `
    -Name "AtoB" `
    -VirtualNetwork $vnetA `
    -RemoteVirtualNetworkId $vnetB.Id `
    -AllowForwardedTraffic

# Create peering from B to A
Add-AzVirtualNetworkPeering `
    -Name "BtoA" `
    -VirtualNetwork $vnetB `
    -RemoteVirtualNetworkId $vnetA.Id `
    -AllowForwardedTraffic
```

```bash
# Create peering from A to B
az network vnet peering create \
    --name "AtoB" \
    --resource-group "RG-A" \
    --vnet-name "VNetA" \
    --remote-vnet "/subscriptions/{sub-id}/resourceGroups/RG-B/providers/Microsoft.Network/virtualNetworks/VNetB" \
    --allow-forwarded-traffic \
    --allow-vnet-access

# Create peering from B to A
az network vnet peering create \
    --name "BtoA" \
    --resource-group "RG-B" \
    --vnet-name "VNetB" \
    --remote-vnet "/subscriptions/{sub-id}/resourceGroups/RG-A/providers/Microsoft.Network/virtualNetworks/VNetA" \
    --allow-forwarded-traffic \
    --allow-vnet-access
```

---

## 4.2 Configure Network Security Groups (NSGs)

### NSG Rules

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           NSG RULES                                          │
│                                                                              │
│  RULE PROPERTIES:                                                            │
│  ─────────────────                                                          │
│  • Name: Descriptive name                                                    │
│  • Priority: 100-4096 (lower number = higher priority)                      │
│  • Source: IP, CIDR, Service Tag, ASG                                       │
│  • Source Port: * or specific port                                          │
│  • Destination: IP, CIDR, Service Tag, ASG                                  │
│  • Destination Port: * or specific port(s)                                  │
│  • Protocol: TCP, UDP, ICMP, *                                              │
│  • Action: Allow or Deny                                                     │
│  • Direction: Inbound or Outbound                                            │
│                                                                              │
│  DEFAULT RULES (Cannot delete, can override with lower priority):           │
│  ─────────────────────────────────────────────────────────────              │
│  INBOUND:                                                                    │
│  65000: AllowVnetInBound (Allow all traffic within VNet)                   │
│  65001: AllowAzureLoadBalancerInBound (Allow health probes)                │
│  65500: DenyAllInBound (Deny everything else)                               │
│                                                                              │
│  OUTBOUND:                                                                   │
│  65000: AllowVnetOutBound (Allow all traffic within VNet)                  │
│  65001: AllowInternetOutBound (Allow outbound internet)                    │
│  65500: DenyAllOutBound (Deny everything else)                              │
│                                                                              │
│  ASSOCIATION:                                                                │
│  • To Subnet: All resources in subnet get this NSG                         │
│  • To NIC: Only that specific NIC gets this NSG                            │
│  • Both: Traffic must pass BOTH NSG rules                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Service Tags (Common)

| Service Tag | Description |
|------------|-------------|
| **VirtualNetwork** | VNet address space + peered VNets + VPN-connected |
| **AzureLoadBalancer** | Azure Load Balancer health probes |
| **Internet** | Everything outside Azure VNet |
| **AzureCloud** | All Azure public IP addresses |
| **Storage** | Azure Storage service |
| **Sql** | Azure SQL Database |
| **AzureActiveDirectory** | Azure AD |
| **AzureMonitor** | Azure monitoring services |

### NSG Commands

```powershell
# Create NSG
$nsg = New-AzNetworkSecurityGroup `
    -Name "WebServer-NSG" `
    -ResourceGroupName "MyRG" `
    -Location "eastus"

# Add inbound rule (allow HTTP)
$nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "Allow-HTTP" `
    -Description "Allow HTTP" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority 100 `
    -SourceAddressPrefix "Internet" `
    -SourcePortRange "*" `
    -DestinationAddressPrefix "*" `
    -DestinationPortRange 80

# Add another rule (allow HTTPS)
$nsg | Add-AzNetworkSecurityRuleConfig `
    -Name "Allow-HTTPS" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority 110 `
    -SourceAddressPrefix "Internet" `
    -SourcePortRange "*" `
    -DestinationAddressPrefix "*" `
    -DestinationPortRange 443

# Update NSG
$nsg | Set-AzNetworkSecurityGroup

# Associate NSG to subnet
$vnet = Get-AzVirtualNetwork -Name "MyVNet" -ResourceGroupName "MyRG"
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "WebSubnet"
$subnet.NetworkSecurityGroup = $nsg
$vnet | Set-AzVirtualNetwork
```

```bash
# Create NSG
az network nsg create \
    --name "WebServer-NSG" \
    --resource-group "MyRG" \
    --location "eastus"

# Add rule
az network nsg rule create \
    --nsg-name "WebServer-NSG" \
    --resource-group "MyRG" \
    --name "Allow-HTTP" \
    --priority 100 \
    --source-address-prefixes Internet \
    --destination-port-ranges 80 \
    --access Allow \
    --protocol Tcp \
    --direction Inbound

# Associate to subnet
az network vnet subnet update \
    --vnet-name "MyVNet" \
    --name "WebSubnet" \
    --resource-group "MyRG" \
    --network-security-group "WebServer-NSG"
```

---

## 4.3 Configure Azure Load Balancer

### Load Balancer Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LOAD BALANCER COMPARISON                                  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ BASIC LOAD BALANCER                                                   │  │
│  │ • Free (no cost for LB itself)                                        │  │
│  │ • Up to 300 instances                                                 │  │
│  │ • No Availability Zones support                                       │  │
│  │ • No SLA                                                              │  │
│  │ • Open by default (NSG optional)                                      │  │
│  │ • Being retired - use Standard for new deployments                   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │ STANDARD LOAD BALANCER                                                │  │
│  │ • Pay per hour + data processed                                       │  │
│  │ • Up to 1000 instances                                                │  │
│  │ • Availability Zones support (zone-redundant)                        │  │
│  │ • 99.99% SLA                                                          │  │
│  │ • Secure by default (NSG required to allow traffic)                  │  │
│  │ • HA Ports (all ports load balanced)                                 │  │
│  │ • Outbound rules                                                      │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  TYPES BY TRAFFIC:                                                           │
│  • Public LB: Internet-facing, distributes traffic to backend VMs          │
│  • Internal LB: Private, distributes traffic within VNet                   │
│                                                                              │
│  LOAD BALANCER COMPONENTS:                                                   │
│  ──────────────────────────                                                 │
│  1. Frontend IP: Where traffic arrives (public or private IP)              │
│  2. Backend Pool: VMs that receive traffic                                  │
│  3. Health Probe: Checks if backend VMs are healthy                        │
│  4. Load Balancing Rule: How to distribute traffic                         │
│  5. Inbound NAT Rule: Direct traffic to specific VM                        │
│  6. Outbound Rule: How VMs access internet (Standard LB only)              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Load Balancer

```powershell
# Create Public IP
$pip = New-AzPublicIpAddress `
    -Name "LB-PublicIP" `
    -ResourceGroupName "MyRG" `
    -Location "eastus" `
    -Sku "Standard" `
    -AllocationMethod "Static"

# Create Load Balancer
$lb = New-AzLoadBalancer `
    -Name "MyLoadBalancer" `
    -ResourceGroupName "MyRG" `
    -Location "eastus" `
    -Sku "Standard" `
    -FrontendIpConfiguration (New-AzLoadBalancerFrontendIpConfig -Name "FrontEnd" -PublicIpAddress $pip) `
    -BackendAddressPool (New-AzLoadBalancerBackendAddressPoolConfig -Name "BackendPool") `
    -Probe (New-AzLoadBalancerProbeConfig -Name "HealthProbe" -Protocol "Http" -Port 80 -RequestPath "/" -IntervalInSeconds 15 -ProbeCount 2) `
    -LoadBalancingRule (New-AzLoadBalancerRuleConfig -Name "HTTPRule" -Protocol "Tcp" -FrontendPort 80 -BackendPort 80 -FrontendIpConfiguration (New-AzLoadBalancerFrontendIpConfig -Name "FrontEnd" -PublicIpAddress $pip) -BackendAddressPool (New-AzLoadBalancerBackendAddressPoolConfig -Name "BackendPool") -Probe (New-AzLoadBalancerProbeConfig -Name "HealthProbe"))
```

```bash
# Create Public IP
az network public-ip create \
    --name "LB-PublicIP" \
    --resource-group "MyRG" \
    --sku "Standard" \
    --allocation-method "Static"

# Create Load Balancer
az network lb create \
    --name "MyLoadBalancer" \
    --resource-group "MyRG" \
    --sku "Standard" \
    --frontend-ip-name "FrontEnd" \
    --public-ip-address "LB-PublicIP" \
    --backend-pool-name "BackendPool"

# Create Health Probe
az network lb probe create \
    --lb-name "MyLoadBalancer" \
    --resource-group "MyRG" \
    --name "HealthProbe" \
    --protocol "Http" \
    --port 80 \
    --path "/"

# Create Load Balancing Rule
az network lb rule create \
    --lb-name "MyLoadBalancer" \
    --resource-group "MyRG" \
    --name "HTTPRule" \
    --protocol "Tcp" \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name "FrontEnd" \
    --backend-pool-name "BackendPool" \
    --probe-name "HealthProbe"
```

---

## 4.4 Configure Azure DNS

### Azure DNS Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AZURE DNS                                            │
│                                                                              │
│  PUBLIC DNS:                                                                 │
│  • Host your domain's DNS records                                           │
│  • Example: mycompany.com points to your Azure resources                    │
│  • Anycast network for high availability                                    │
│  • Alias records for apex domains                                           │
│                                                                              │
│  PRIVATE DNS:                                                                │
│  • Name resolution within VNets                                             │
│  • No custom DNS server needed                                              │
│  • Automatic VM registration                                                 │
│  • Works across peered VNets                                                │
│                                                                              │
│  COMMON RECORD TYPES:                                                        │
│  ─────────────────────                                                      │
│  • A: IPv4 address (webapp.contoso.com → 20.185.100.1)                     │
│  • AAAA: IPv6 address                                                        │
│  • CNAME: Canonical name (www.contoso.com → contoso.com)                   │
│  • MX: Mail exchange (contoso.com → mail.contoso.com)                      │
│  • NS: Name server                                                           │
│  • TXT: Text (SPF, DKIM verification)                                       │
│  • SRV: Service locator                                                      │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### DNS Commands

```powershell
# Create DNS Zone (Public)
New-AzDnsZone -Name "contoso.com" -ResourceGroupName "DNS-RG"

# Create A Record
New-AzDnsRecordSet `
    -Name "www" `
    -ZoneName "contoso.com" `
    -ResourceGroupName "DNS-RG" `
    -RecordType "A" `
    -Ttl 3600 `
    -DnsRecords (New-AzDnsRecordConfig -IPv4Address "20.185.100.1")

# Create CNAME Record
New-AzDnsRecordSet `
    -Name "blog" `
    -ZoneName "contoso.com" `
    -ResourceGroupName "DNS-RG" `
    -RecordType "CNAME" `
    -Ttl 3600 `
    -DnsRecords (New-AzDnsRecordConfig -Cname "contoso.azurewebsites.net")

# Create Private DNS Zone
New-AzPrivateDnsZone -Name "contoso.internal" -ResourceGroupName "DNS-RG"

# Link Private DNS to VNet
New-AzPrivateDnsVirtualNetworkLink `
    -Name "VNetLink" `
    -ZoneName "contoso.internal" `
    -ResourceGroupName "DNS-RG" `
    -VirtualNetworkId $vnet.Id `
    -EnableRegistration
```

```bash
# Create DNS Zone
az network dns zone create \
    --name "contoso.com" \
    --resource-group "DNS-RG"

# Create A Record
az network dns record-set a add-record \
    --zone-name "contoso.com" \
    --resource-group "DNS-RG" \
    --record-set-name "www" \
    --ipv4-address "20.185.100.1"

# Create Private DNS Zone
az network private-dns zone create \
    --name "contoso.internal" \
    --resource-group "DNS-RG"

# Link to VNet
az network private-dns link vnet create \
    --zone-name "contoso.internal" \
    --resource-group "DNS-RG" \
    --name "VNetLink" \
    --virtual-network "MyVNet" \
    --registration-enabled true
```

---

# DOMAIN 5: MONITOR AND MAINTAIN AZURE RESOURCES (10-15%)

## 5.1 Configure Azure Monitor

### Azure Monitor Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      AZURE MONITOR ECOSYSTEM                                 │
│                                                                              │
│  DATA SOURCES                      DATA STORES              ANALYZE/ACT     │
│  ────────────                      ───────────              ────────────    │
│                                                                              │
│  ┌─────────────────┐               ┌───────────────┐        ┌───────────┐   │
│  │ Application     │──Telemetry──►│   Metrics     │──────►│ Dashboards│   │
│  │ (App Insights)  │               │ (Time-series) │        │ Visualize │   │
│  └─────────────────┘               └───────────────┘        └───────────┘   │
│                                                                              │
│  ┌─────────────────┐               ┌───────────────┐        ┌───────────┐   │
│  │ Guest OS        │───Logs────►│    Logs       │──────►│  Alerts   │   │
│  │ (Perf counters) │               │(Log Analytics)│        │  Notify   │   │
│  └─────────────────┘               └───────────────┘        └───────────┘   │
│                                                                              │
│  ┌─────────────────┐                                        ┌───────────┐   │
│  │ Azure Resources │                                        │ Autoscale │   │
│  │ (Activity logs) │                                        │   Act     │   │
│  └─────────────────┘                                        └───────────┘   │
│                                                                              │
│  ┌─────────────────┐                                        ┌───────────┐   │
│  │ Azure Tenant    │                                        │  Workbooks│   │
│  │ (Sign-in logs)  │                                        │  Reports  │   │
│  └─────────────────┘                                        └───────────┘   │
│                                                                              │
│  KEY CONCEPTS:                                                               │
│  ─────────────                                                              │
│  • Metrics: Numerical data at regular intervals (CPU %, memory %)          │
│  • Logs: Text/structured data (events, traces, errors)                     │
│  • Log Analytics Workspace: Central repository for logs                    │
│  • Alerts: Notify when conditions are met                                  │
│  • Action Groups: Define who/what gets notified                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Log Analytics Workspace

```powershell
# Create Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
    -ResourceGroupName "Monitoring-RG" `
    -Name "MyLogAnalytics" `
    -Location "eastus" `
    -Sku "PerGB2018"

# Enable diagnostic settings for a resource
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "Monitoring-RG" -Name "MyLogAnalytics"
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"

Set-AzDiagnosticSetting `
    -ResourceId $vm.Id `
    -WorkspaceId $workspace.ResourceId `
    -Name "VM-Diagnostics" `
    -Enabled $true `
    -Category "Administrative", "Security", "ServiceHealth"
```

```bash
# Create Log Analytics Workspace
az monitor log-analytics workspace create \
    --resource-group "Monitoring-RG" \
    --workspace-name "MyLogAnalytics" \
    --location "eastus"

# Enable diagnostic settings
az monitor diagnostic-settings create \
    --name "VM-Diagnostics" \
    --resource "/subscriptions/{sub}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" \
    --workspace "/subscriptions/{sub}/resourceGroups/Monitoring-RG/providers/Microsoft.OperationalInsights/workspaces/MyLogAnalytics" \
    --logs '[{"category": "Administrative", "enabled": true}]'
```

### Basic KQL Queries (Log Analytics)

```kusto
// Find all errors in the last 24 hours
AzureActivity
| where TimeGenerated > ago(24h)
| where Level == "Error"
| project TimeGenerated, OperationName, Caller, ActivityStatus

// VM performance - CPU usage over 80%
Perf
| where ObjectName == "Processor"
| where CounterName == "% Processor Time"
| where CounterValue > 80
| summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 5m), Computer
| render timechart

// Count of VMs by resource group
AzureActivity
| where ResourceProvider == "Microsoft.Compute"
| summarize count() by ResourceGroup
| render piechart

// Failed sign-ins
SigninLogs
| where ResultType != 0
| project TimeGenerated, UserPrincipalName, AppDisplayName, ResultDescription, IPAddress
```

---

## 5.2 Configure Alerts

### Alert Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         ALERT TYPES                                          │
│                                                                              │
│  1. METRIC ALERTS                                                            │
│     • Based on metric values (CPU, memory, etc.)                            │
│     • Near real-time (1 minute minimum)                                     │
│     • Example: Alert when CPU > 80% for 5 minutes                           │
│                                                                              │
│  2. LOG ALERTS                                                               │
│     • Based on Log Analytics queries                                        │
│     • Run at scheduled intervals                                            │
│     • Example: Alert when more than 5 failed logins                         │
│                                                                              │
│  3. ACTIVITY LOG ALERTS                                                      │
│     • Based on Azure Activity Log events                                    │
│     • Trigger on resource creation, deletion, etc.                          │
│     • Example: Alert when VM is stopped                                      │
│                                                                              │
│  ALERT RULE COMPONENTS:                                                      │
│  ─────────────────────────                                                  │
│  • Target Resource: What to monitor                                         │
│  • Condition: When to trigger (signal + logic)                              │
│  • Action Group: What to do when triggered                                  │
│  • Alert Details: Name, description, severity                               │
│                                                                              │
│  SEVERITY LEVELS:                                                            │
│  • 0 (Sev0): Critical                                                        │
│  • 1 (Sev1): Error                                                           │
│  • 2 (Sev2): Warning                                                         │
│  • 3 (Sev3): Informational                                                   │
│  • 4 (Sev4): Verbose                                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Create Action Groups and Alerts

```powershell
# Create Action Group
$emailReceiver = New-AzActionGroupEmailReceiverObject `
    -Name "EmailAdmin" `
    -EmailAddress "admin@contoso.com"

$smsReceiver = New-AzActionGroupSmsReceiverObject `
    -Name "SMSAdmin" `
    -PhoneNumber "+1234567890" `
    -CountryCode "1"

New-AzActionGroup `
    -Name "CriticalAlerts-AG" `
    -ResourceGroupName "Monitoring-RG" `
    -ShortName "CritAlert" `
    -Location "Global" `
    -EmailReceiver $emailReceiver `
    -SmsReceiver $smsReceiver

# Create Metric Alert (CPU > 80%)
$condition = New-AzMetricAlertRuleV2Criteria `
    -MetricName "Percentage CPU" `
    -Operator "GreaterThan" `
    -Threshold 80 `
    -TimeAggregation "Average"

Add-AzMetricAlertRuleV2 `
    -Name "High-CPU-Alert" `
    -ResourceGroupName "Monitoring-RG" `
    -WindowSize 00:05:00 `
    -Frequency 00:01:00 `
    -TargetResourceId "/subscriptions/{sub}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" `
    -Condition $condition `
    -ActionGroupId "/subscriptions/{sub}/resourceGroups/Monitoring-RG/providers/Microsoft.Insights/actionGroups/CriticalAlerts-AG" `
    -Severity 2
```

```bash
# Create Action Group
az monitor action-group create \
    --name "CriticalAlerts-AG" \
    --resource-group "Monitoring-RG" \
    --short-name "CritAlert" \
    --action email admin admin@contoso.com \
    --action sms smsadmin 1 1234567890

# Create Metric Alert
az monitor metrics alert create \
    --name "High-CPU-Alert" \
    --resource-group "Monitoring-RG" \
    --scopes "/subscriptions/{sub}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" \
    --condition "avg Percentage CPU > 80" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --action "/subscriptions/{sub}/resourceGroups/Monitoring-RG/providers/Microsoft.Insights/actionGroups/CriticalAlerts-AG" \
    --severity 2
```

---

## 5.3 Configure Azure Backup

### Backup Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      AZURE BACKUP                                            │
│                                                                              │
│  WHAT CAN BE BACKED UP:                                                      │
│  ───────────────────────                                                    │
│  • Azure VMs (full VM backup)                                               │
│  • Azure Files (file shares)                                                 │
│  • SQL Server in Azure VMs                                                   │
│  • SAP HANA in Azure VMs                                                     │
│  • Azure Blob Storage                                                        │
│  • Azure Disks                                                               │
│  • Azure Database for PostgreSQL                                             │
│                                                                              │
│  KEY COMPONENTS:                                                             │
│  ────────────────                                                           │
│  1. Recovery Services Vault                                                  │
│     • Container for backup data                                             │
│     • Houses backup policies                                                 │
│     • Must be in same region as resources (or paired for cross-region)     │
│                                                                              │
│  2. Backup Policy                                                            │
│     • Defines backup schedule (daily, weekly, monthly, yearly)             │
│     • Defines retention period                                               │
│     • Example: Daily at 2 AM, keep for 30 days                             │
│                                                                              │
│  3. Recovery Point                                                           │
│     • A snapshot of data at a point in time                                 │
│     • Types: Crash-consistent, Application-consistent, File-consistent     │
│                                                                              │
│  RESTORE OPTIONS:                                                            │
│  ────────────────                                                           │
│  • Create new VM from backup                                                 │
│  • Replace existing VM disks                                                 │
│  • Restore specific files/folders                                           │
│  • Cross-region restore (if enabled)                                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Configure VM Backup

```powershell
# Create Recovery Services Vault
New-AzRecoveryServicesVault `
    -Name "MyRecoveryVault" `
    -ResourceGroupName "Backup-RG" `
    -Location "eastus"

# Set vault context
$vault = Get-AzRecoveryServicesVault -Name "MyRecoveryVault" -ResourceGroupName "Backup-RG"
Set-AzRecoveryServicesVaultContext -Vault $vault

# Set storage redundancy
Set-AzRecoveryServicesBackupProperty `
    -Vault $vault `
    -BackupStorageRedundancy GeoRedundant

# Get default VM backup policy
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -Name "DefaultPolicy"

# Enable backup for a VM
$vm = Get-AzVM -ResourceGroupName "MyRG" -Name "MyVM"
Enable-AzRecoveryServicesBackupProtection `
    -Policy $policy `
    -Name $vm.Name `
    -ResourceGroupName $vm.ResourceGroupName

# Trigger immediate backup
$container = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVM -VaultId $vault.ID
$item = Get-AzRecoveryServicesBackupItem -Container $container -WorkloadType AzureVM -VaultId $vault.ID
Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $vault.ID
```

```bash
# Create Recovery Services Vault
az backup vault create \
    --name "MyRecoveryVault" \
    --resource-group "Backup-RG" \
    --location "eastus"

# Enable backup for VM with default policy
az backup protection enable-for-vm \
    --resource-group "Backup-RG" \
    --vault-name "MyRecoveryVault" \
    --vm "/subscriptions/{sub}/resourceGroups/MyRG/providers/Microsoft.Compute/virtualMachines/MyVM" \
    --policy-name "DefaultPolicy"

# Trigger backup now
az backup protection backup-now \
    --resource-group "Backup-RG" \
    --vault-name "MyRecoveryVault" \
    --container-name "iaasvmcontainer;myvm" \
    --item-name "vm;myvm"

# List recovery points
az backup recoverypoint list \
    --resource-group "Backup-RG" \
    --vault-name "MyRecoveryVault" \
    --container-name "iaasvmcontainer;myvm" \
    --item-name "vm;myvm"
```

---

# STUDY TIPS FOR AZ-104

## 1. Practice Lab Recommendations

| Topic | Lab Exercise |
|-------|-------------|
| Identity | Create users, groups, assign RBAC roles |
| Storage | Create storage account, upload blobs, configure SAS |
| Compute | Create VM, scale set, configure availability |
| Networking | Create VNet, NSG rules, peering, load balancer |
| Monitoring | Configure alerts, Log Analytics queries |
| Backup | Configure VM backup, perform restore |

## 2. Key Exam Tips

1. **Know the CLI/PowerShell commands** - Many questions ask for the right command
2. **Understand pricing tiers** - Basic vs Standard, SKUs matter
3. **Remember limits and quotas** - Max VMs, NSG rules, etc.
4. **Redundancy options are heavily tested** - LRS, ZRS, GRS, RA-GRS
5. **Network Security is critical** - NSG rules, service tags, priority
6. **RBAC scope inheritance** - Understand how permissions flow down
7. **Time to study**: 60-80 hours for someone with some Azure experience

## 3. Practice Exam Resources

- Microsoft Learn (free)
- Microsoft Practice Assessments (free)
- Whizlabs
- MeasureUp (official practice tests)

---

*Good luck with your AZ-104 certification! Continue studying with the other modules in this training series for detailed coverage of each topic.*
