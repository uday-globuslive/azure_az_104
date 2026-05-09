# Module 20: Active Directory and Group Policy (On-Premises and Cloud)

## Table of Contents
1. [Active Directory Architecture](#ad-architecture)
2. [On-Premises AD Management](#on-prem-ad)
3. [Group Policy Objects (GPO)](#gpo)
4. [Azure Active Directory (Entra ID)](#azure-ad)
5. [Hybrid Identity](#hybrid-identity)
6. [AD Automation with PowerShell and Ansible](#ad-automation)
7. [Migration Scenarios](#migration-scenarios)
8. [Interview Scenarios and Use Cases](#interview-scenarios)

---

## 1. Active Directory Architecture <a name="ad-architecture"></a>

### AD Domain Services Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                      Forest: corp.local                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐    │
│  │              Root Domain: corp.local                     │    │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │    │
│  │  │    DC01     │  │    DC02     │  │    RODC01   │      │    │
│  │  │ (FSMO)      │  │ (Replica)   │  │ (Read-Only) │      │    │
│  │  │ Schema      │  │             │  │  Branch     │      │    │
│  │  │ Naming      │  │             │  │             │      │    │
│  │  │ PDC         │  │             │  │             │      │    │
│  │  │ RID         │  │             │  │             │      │    │
│  │  │ Infra       │  │             │  │             │      │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                      │
│           ┌───────────────┴───────────────┐                     │
│           ▼                               ▼                      │
│  ┌─────────────────────┐     ┌─────────────────────┐           │
│  │ Child Domain:       │     │ Child Domain:       │           │
│  │ us.corp.local       │     │ eu.corp.local       │           │
│  │  ┌───┐  ┌───┐       │     │  ┌───┐  ┌───┐       │           │
│  │  │DC │  │DC │       │     │  │DC │  │DC │       │           │
│  │  └───┘  └───┘       │     │  └───┘  └───┘       │           │
│  └─────────────────────┘     └─────────────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

### AD Components Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                    Active Directory Components                   │
├─────────────────────────────────────────────────────────────────┤
│  Logical Components          │  Physical Components             │
│  ─────────────────           │  ────────────────────            │
│  • Forests                   │  • Domain Controllers             │
│  • Trees                     │  • Sites                         │
│  • Domains                   │  • Subnets                       │
│  • Organizational Units      │  • Site Links                    │
│  • Objects (Users, Groups,   │  • Global Catalog Servers        │
│    Computers, GPOs)          │  • Read-Only Domain Controllers  │
├─────────────────────────────────────────────────────────────────┤
│  Services                                                        │
│  ────────                                                        │
│  • AD DS (Domain Services)   • AD LDS (Lightweight Dir Services)│
│  • AD CS (Certificate Svcs)  • AD FS (Federation Services)      │
│  • AD RMS (Rights Mgmt)                                         │
└─────────────────────────────────────────────────────────────────┘
```

### FSMO Roles
```
Forest-Wide Roles (One per Forest):
├── Schema Master
│   └── Controls schema modifications
└── Domain Naming Master
    └── Controls addition/removal of domains

Domain-Wide Roles (One per Domain):
├── PDC Emulator
│   ├── Time synchronization
│   ├── Password changes
│   └── Account lockouts
├── RID Master
│   └── Allocates RID pools to DCs
└── Infrastructure Master
    └── Updates cross-domain references
```

---

## 2. On-Premises AD Management <a name="on-prem-ad"></a>

### Domain Controller Deployment
```powershell
# Install AD DS Role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller (New Forest)
Install-ADDSForest `
    -DomainName "corp.local" `
    -DomainNetBIOSName "CORP" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -DatabasePath "D:\NTDS" `
    -LogPath "D:\NTDS" `
    -SysvolPath "D:\SYSVOL" `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
    -Force

# Add Additional Domain Controller
Install-ADDSDomainController `
    -DomainName "corp.local" `
    -InstallDns:$true `
    -Credential (Get-Credential) `
    -DatabasePath "D:\NTDS" `
    -LogPath "D:\NTDS" `
    -SysvolPath "D:\SYSVOL" `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force) `
    -Force
```

### Organizational Unit Structure
```powershell
# Create OU Structure
$OUStructure = @(
    "OU=Corp,DC=corp,DC=local",
    "OU=Users,OU=Corp,DC=corp,DC=local",
    "OU=Groups,OU=Corp,DC=corp,DC=local",
    "OU=Computers,OU=Corp,DC=corp,DC=local",
    "OU=Servers,OU=Corp,DC=corp,DC=local",
    "OU=Service Accounts,OU=Corp,DC=corp,DC=local",
    "OU=Workstations,OU=Computers,OU=Corp,DC=corp,DC=local",
    "OU=Web Servers,OU=Servers,OU=Corp,DC=corp,DC=local",
    "OU=Database Servers,OU=Servers,OU=Corp,DC=corp,DC=local",
    "OU=Application Servers,OU=Servers,OU=Corp,DC=corp,DC=local"
)

foreach ($OU in $OUStructure) {
    $Name = ($OU -split ",")[0] -replace "OU=", ""
    $Path = ($OU -split ",", 2)[1]
    
    if (-not (Get-ADOrganizationalUnit -Filter "DistinguishedName -eq '$OU'" -ErrorAction SilentlyContinue)) {
        New-ADOrganizationalUnit -Name $Name -Path $Path -ProtectedFromAccidentalDeletion $true
        Write-Host "Created OU: $Name"
    }
}
```

### User and Group Management
```powershell
# Bulk User Creation
$Users = Import-Csv "users.csv"

foreach ($User in $Users) {
    $Password = ConvertTo-SecureString $User.Password -AsPlainText -Force
    
    $UserParams = @{
        SamAccountName    = $User.SamAccountName
        UserPrincipalName = "$($User.SamAccountName)@corp.local"
        Name              = "$($User.FirstName) $($User.LastName)"
        GivenName         = $User.FirstName
        Surname           = $User.LastName
        DisplayName       = "$($User.FirstName) $($User.LastName)"
        EmailAddress      = $User.Email
        Department        = $User.Department
        Title             = $User.Title
        Path              = "OU=Users,OU=Corp,DC=corp,DC=local"
        AccountPassword   = $Password
        Enabled           = $true
        ChangePasswordAtLogon = $true
    }
    
    New-ADUser @UserParams
    
    # Add to groups
    if ($User.Groups) {
        $Groups = $User.Groups -split ";"
        foreach ($Group in $Groups) {
            Add-ADGroupMember -Identity $Group -Members $User.SamAccountName
        }
    }
}

# Create Security Groups
$Groups = @(
    @{Name="IT-Admins"; Description="IT Administrators"; Scope="Global"; Category="Security"},
    @{Name="HR-Users"; Description="HR Department Users"; Scope="Global"; Category="Security"},
    @{Name="Finance-Users"; Description="Finance Department"; Scope="Global"; Category="Security"},
    @{Name="VPN-Users"; Description="VPN Access Group"; Scope="Global"; Category="Security"},
    @{Name="Server-Admins"; Description="Server Administrators"; Scope="Global"; Category="Security"}
)

foreach ($Group in $Groups) {
    New-ADGroup `
        -Name $Group.Name `
        -Description $Group.Description `
        -GroupScope $Group.Scope `
        -GroupCategory $Group.Category `
        -Path "OU=Groups,OU=Corp,DC=corp,DC=local"
}
```

### Service Account Management
```powershell
# Create Managed Service Account (gMSA)
# First, create KDS root key (if not exists)
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

# Create Group Managed Service Account
New-ADServiceAccount `
    -Name "svc_webapp" `
    -DNSHostName "svc_webapp.corp.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "Web-Servers" `
    -Description "Web Application Service Account" `
    -ServicePrincipalNames @("HTTP/webapp.corp.local", "HTTP/webapp")

# Install gMSA on target server
Install-ADServiceAccount -Identity "svc_webapp"

# Test gMSA
Test-ADServiceAccount -Identity "svc_webapp"

# Configure service to use gMSA
$Service = Get-WmiObject Win32_Service -Filter "Name='WebAppService'"
$Service.Change($null, $null, $null, $null, $null, $null, "CORP\svc_webapp$", $null, $null, $null, $null)
```

---

## 3. Group Policy Objects (GPO) <a name="gpo"></a>

### GPO Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    Group Policy Processing                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Local Policy → Site → Domain → OU → Child OU                   │
│       ↓           ↓       ↓        ↓        ↓                   │
│  [Lowest]  ←────────────────────────────── [Highest]            │
│  Priority                                  Priority              │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│                    GPO Components                                │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Group Policy Container (GPC) - Active Directory         │    │
│  │  - Stored in AD database                                 │    │
│  │  - Contains GPO properties, links, permissions           │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Group Policy Template (GPT) - SYSVOL                    │    │
│  │  - Stored in \\domain\SYSVOL\domain\Policies             │    │
│  │  - Contains actual policy settings                       │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

### Security Baseline GPOs
```powershell
# Create Security Baseline GPO
$GPO = New-GPO -Name "Security-Baseline-Servers" -Comment "CIS Benchmark Level 1 for Servers"

# Link to OU
New-GPLink -Guid $GPO.Id -Target "OU=Servers,OU=Corp,DC=corp,DC=local" -LinkEnabled Yes

# Configure Password Policy (Computer Configuration)
Set-GPRegistryValue -Name "Security-Baseline-Servers" `
    -Key "HKLM\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters" `
    -ValueName "MaximumPasswordAge" `
    -Type DWord `
    -Value 60

# Configure Account Lockout
Set-GPRegistryValue -Name "Security-Baseline-Servers" `
    -Key "HKLM\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters" `
    -ValueName "LockoutBadCount" `
    -Type DWord `
    -Value 5

# Configure Audit Policy
$AuditSettings = @"
[Unicode]
Unicode=yes
[Version]
signature="`$CHICAGO`$"
Revision=1
[System Access]
MinimumPasswordAge = 1
MaximumPasswordAge = 60
MinimumPasswordLength = 14
PasswordComplexity = 1
PasswordHistorySize = 24
LockoutBadCount = 5
ResetLockoutCount = 30
LockoutDuration = 30
[Event Audit]
AuditSystemEvents = 3
AuditLogonEvents = 3
AuditObjectAccess = 3
AuditPrivilegeUse = 3
AuditPolicyChange = 3
AuditAccountManage = 3
AuditProcessTracking = 0
AuditDSAccess = 3
AuditAccountLogon = 3
"@

$AuditSettings | Out-File "C:\Temp\SecEdit.inf" -Encoding Unicode
secedit /configure /db secedit.sdb /cfg "C:\Temp\SecEdit.inf" /areas SECURITYPOLICY
```

### GPO for Windows Firewall
```powershell
# Create Firewall GPO
$FirewallGPO = New-GPO -Name "Firewall-Policy-Servers"

# Enable Windows Firewall for all profiles
Set-GPRegistryValue -Name "Firewall-Policy-Servers" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" `
    -ValueName "EnableFirewall" `
    -Type DWord `
    -Value 1

Set-GPRegistryValue -Name "Firewall-Policy-Servers" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PrivateProfile" `
    -ValueName "EnableFirewall" `
    -Type DWord `
    -Value 1

Set-GPRegistryValue -Name "Firewall-Policy-Servers" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\PublicProfile" `
    -ValueName "EnableFirewall" `
    -Type DWord `
    -Value 1

# Block inbound by default
Set-GPRegistryValue -Name "Firewall-Policy-Servers" `
    -Key "HKLM\SOFTWARE\Policies\Microsoft\WindowsFirewall\DomainProfile" `
    -ValueName "DefaultInboundAction" `
    -Type DWord `
    -Value 1
```

### GPO Preferences for Drive Mappings
```xml
<!-- Drive Mapping GPO Preference -->
<?xml version="1.0" encoding="utf-8"?>
<Drives clsid="{8FDDCC1A-0C3C-43cd-A6B4-71A6DF20DA8C}">
  <Drive clsid="{935D1B74-9CB8-4e3c-9914-7DD559B7A417}" 
         name="S:" 
         status="S:" 
         image="2" 
         changed="2024-01-15 10:00:00" 
         uid="{12345678-1234-1234-1234-123456789ABC}" 
         bypassErrors="1">
    <Properties action="U" 
                thisDrive="NOCHANGE" 
                allDrives="NOCHANGE" 
                userName="" 
                path="\\fileserver\shared" 
                label="Shared Drive" 
                persistent="1" 
                useLetter="1" 
                letter="S"/>
    <Filters>
      <FilterGroup bool="AND" not="0" 
                   name="CORP\Finance-Users" 
                   sid="S-1-5-21-..." 
                   userContext="1" 
                   primaryGroup="0" 
                   localGroup="0"/>
    </Filters>
  </Drive>
</Drives>
```

### GPO Backup and Restore
```powershell
# Backup all GPOs
$BackupPath = "\\fileserver\GPOBackups\$(Get-Date -Format 'yyyy-MM-dd')"
New-Item -ItemType Directory -Path $BackupPath -Force

Get-GPO -All | ForEach-Object {
    Backup-GPO -Guid $_.Id -Path $BackupPath
    Write-Host "Backed up: $($_.DisplayName)"
}

# Export GPO Report
Get-GPOReport -All -ReportType HTML -Path "$BackupPath\GPO-Report.html"

# Restore GPO
Restore-GPO -Name "Security-Baseline-Servers" -Path $BackupPath

# Import GPO to different domain
Import-GPO -BackupGpoName "Security-Baseline-Servers" `
    -TargetName "Security-Baseline-Servers" `
    -Path $BackupPath `
    -MigrationTable "C:\Migration\migration.migtable" `
    -CreateIfNeeded
```

### GPO Troubleshooting Commands
```powershell
# Force Group Policy Update
gpupdate /force

# Get Resultant Set of Policy
gpresult /r
gpresult /h "C:\Temp\GPResult.html"

# Get GPO applied to specific computer
Get-GPResultantSetOfPolicy -Computer "SERVER01" -ReportType HTML -Path "C:\Temp\SERVER01-RSoP.html"

# Check GPO replication status
Get-ADReplicationPartnerMetadata -Target "corp.local" -Scope Domain | 
    Select-Object Server, LastReplicationSuccess, LastReplicationResult

# Check SYSVOL replication
dcdiag /test:sysvolcheck /e
repadmin /syncall /APed

# Compare GPO versions across DCs
Get-ADDomainController -Filter * | ForEach-Object {
    $DC = $_.HostName
    $GPOPath = "\\$DC\SYSVOL\corp.local\Policies"
    Get-ChildItem $GPOPath -Directory | ForEach-Object {
        $GPT = Get-Content "$($_.FullName)\GPT.INI" | Where-Object { $_ -match "Version" }
        [PSCustomObject]@{
            DC = $DC
            GPO = $_.Name
            Version = $GPT
        }
    }
}
```

---

## 4. Azure Active Directory (Entra ID) <a name="azure-ad"></a>

### Azure AD Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    Microsoft Entra ID (Azure AD)                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                      Tenant                              │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │    │
│  │  │   Users     │ │   Groups    │ │Applications │        │    │
│  │  │ (Cloud/     │ │ (Security/  │ │ (Enterprise │        │    │
│  │  │  Synced)    │ │  M365)      │ │  Apps)      │        │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘        │    │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │    │
│  │  │  Devices    │ │Conditional  │ │   Roles     │        │    │
│  │  │ (Registered/│ │  Access     │ │  (RBAC)     │        │    │
│  │  │  Joined)    │ │  Policies   │ │             │        │    │
│  │  └─────────────┘ └─────────────┘ └─────────────┘        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Integration Points:                                             │
│  • Azure Resources (Subscriptions, Resource Groups)              │
│  • Microsoft 365 Services                                        │
│  • Third-party SaaS Applications                                │
│  • On-premises AD (via Azure AD Connect)                        │
└─────────────────────────────────────────────────────────────────┘
```

### Azure AD Management with PowerShell
```powershell
# Connect to Azure AD
Connect-AzureAD
# Or using Microsoft Graph
Connect-MgGraph -Scopes "User.ReadWrite.All", "Group.ReadWrite.All"

# Create Azure AD Users
$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$PasswordProfile.Password = "P@ssw0rd123!"
$PasswordProfile.ForceChangePasswordNextLogin = $true

New-AzureADUser `
    -DisplayName "John Doe" `
    -UserPrincipalName "john.doe@contoso.onmicrosoft.com" `
    -AccountEnabled $true `
    -PasswordProfile $PasswordProfile `
    -MailNickName "johndoe" `
    -Department "IT" `
    -JobTitle "System Administrator"

# Using Microsoft Graph
$NewUser = @{
    accountEnabled = $true
    displayName = "Jane Smith"
    mailNickname = "janesmith"
    userPrincipalName = "jane.smith@contoso.onmicrosoft.com"
    passwordProfile = @{
        forceChangePasswordNextSignIn = $true
        password = "P@ssw0rd123!"
    }
}
New-MgUser -BodyParameter $NewUser

# Create Security Group
New-AzureADGroup `
    -DisplayName "Cloud Admins" `
    -Description "Cloud Administration Team" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "cloudadmins"

# Create Dynamic Group
New-AzureADMSGroup `
    -DisplayName "All IT Users" `
    -Description "Dynamic group for IT department" `
    -SecurityEnabled $true `
    -MailEnabled $false `
    -MailNickName "allit" `
    -GroupTypes "DynamicMembership" `
    -MembershipRule "(user.department -eq 'IT')" `
    -MembershipRuleProcessingState "On"
```

### Conditional Access Policies
```powershell
# Create Conditional Access Policy using Microsoft Graph
$CAPolicy = @{
    displayName = "Require MFA for Admin Roles"
    state = "enabled"
    conditions = @{
        applications = @{
            includeApplications = @("All")
        }
        users = @{
            includeRoles = @(
                "62e90394-69f5-4237-9190-012177145e10"  # Global Admin
                "f28a1f50-f6e7-4571-818b-6a12f2af6b6c"  # SharePoint Admin
            )
        }
        signInRiskLevels = @("high", "medium")
        clientAppTypes = @("all")
    }
    grantControls = @{
        operator = "OR"
        builtInControls = @("mfa")
    }
    sessionControls = @{
        signInFrequency = @{
            value = 1
            type = "hours"
            isEnabled = $true
        }
    }
}

# Apply using Graph API
Invoke-MgGraphRequest -Method POST -Uri "https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies" -Body $CAPolicy

# Block Legacy Authentication
$BlockLegacyAuth = @{
    displayName = "Block Legacy Authentication"
    state = "enabled"
    conditions = @{
        applications = @{
            includeApplications = @("All")
        }
        users = @{
            includeUsers = @("All")
            excludeUsers = @()
        }
        clientAppTypes = @(
            "exchangeActiveSync"
            "other"
        )
    }
    grantControls = @{
        operator = "OR"
        builtInControls = @("block")
    }
}
```

### Azure AD Application Registration
```powershell
# Register Application
$App = New-AzureADApplication `
    -DisplayName "My API Application" `
    -IdentifierUris "https://api.contoso.com" `
    -ReplyUrls @("https://api.contoso.com/callback") `
    -RequiredResourceAccess @(
        @{
            ResourceAppId = "00000003-0000-0000-c000-000000000000"  # Microsoft Graph
            ResourceAccess = @(
                @{
                    Id = "e1fe6dd8-ba31-4d61-89e7-88639da4683d"  # User.Read
                    Type = "Scope"
                }
            )
        }
    )

# Create Service Principal
New-AzureADServicePrincipal -AppId $App.AppId

# Create Client Secret
$Secret = New-AzureADApplicationPasswordCredential `
    -ObjectId $App.ObjectId `
    -CustomKeyIdentifier "MyAPISecret" `
    -EndDate (Get-Date).AddYears(1)

Write-Host "Application ID: $($App.AppId)"
Write-Host "Client Secret: $($Secret.Value)"

# Assign API Permissions (Admin Consent)
$GraphSP = Get-AzureADServicePrincipal -Filter "AppId eq '00000003-0000-0000-c000-000000000000'"
$AppSP = Get-AzureADServicePrincipal -Filter "AppId eq '$($App.AppId)'"

New-AzureADServiceAppRoleAssignment `
    -ObjectId $AppSP.ObjectId `
    -PrincipalId $AppSP.ObjectId `
    -ResourceId $GraphSP.ObjectId `
    -Id "df021288-bdef-4463-88db-98f22de89214"  # User.Read.All
```

---

## 5. Hybrid Identity <a name="hybrid-identity"></a>

### Hybrid Identity Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    Hybrid Identity Architecture                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  On-Premises                          Cloud                      │
│  ────────────                         ─────                      │
│  ┌─────────────┐                     ┌─────────────┐            │
│  │ Active      │     Sync            │ Azure AD    │            │
│  │ Directory   │ ◄──────────────────►│ (Entra ID)  │            │
│  │ Domain      │    Azure AD         │             │            │
│  │ Services    │    Connect          │             │            │
│  └─────────────┘                     └─────────────┘            │
│         │                                   │                    │
│         │                                   │                    │
│         ▼                                   ▼                    │
│  ┌─────────────┐                     ┌─────────────┐            │
│  │ AD FS      │     Federation       │ Azure       │            │
│  │ (Optional) │ ◄──────────────────►│ Applications│            │
│  └─────────────┘                     └─────────────┘            │
│                                                                  │
├─────────────────────────────────────────────────────────────────┤
│  Authentication Methods:                                         │
│  • Password Hash Sync (PHS)                                     │
│  • Pass-through Authentication (PTA)                            │
│  • Federation with AD FS                                        │
└─────────────────────────────────────────────────────────────────┘
```

### Azure AD Connect Configuration
```powershell
# Azure AD Connect Installation Prerequisites
# Run on dedicated server

# Install Azure AD Connect (Silent Installation)
$AADConnectPath = "C:\Install\AzureADConnect.msi"
$LogPath = "C:\Install\AADConnect_Install.log"

Start-Process msiexec.exe -ArgumentList "/i `"$AADConnectPath`" /qn /l*v `"$LogPath`"" -Wait

# Post-installation: Configure via GUI or scripted

# Check Sync Status
Get-ADSyncConnector
Get-ADSyncConnectorStatistics

# Force Full Sync
Start-ADSyncSyncCycle -PolicyType Initial

# Force Delta Sync
Start-ADSyncSyncCycle -PolicyType Delta

# Check Sync Errors
Get-ADSyncScheduler
Get-ADSyncRunStepResult | Where-Object { $_.StepResult -ne "success" }

# Export Configuration for Backup
Get-ADSyncServerConfiguration -Path "C:\Backup\AADConnect"
```

### Password Hash Sync Configuration
```powershell
# Enable Password Hash Sync
Set-ADSyncAADPasswordSyncConfiguration `
    -ConnectorName "corp.local - AAD" `
    -Enable $true

# Verify PHS Status
Get-ADSyncAADPasswordSyncConfiguration -ConnectorName "corp.local - AAD"

# Monitor Password Sync
$Events = Get-WinEvent -LogName "Application" -FilterXPath "*[System[Provider[@Name='Directory Synchronization']]]" | 
    Where-Object { $_.TimeCreated -gt (Get-Date).AddHours(-24) }

$Events | Format-Table TimeCreated, Message -AutoSize
```

### Pass-Through Authentication
```powershell
# Install PTA Agent on additional servers
# Download from Azure Portal: Azure AD > Azure AD Connect > Pass-through Authentication

# Check PTA Agent Status
Get-WinEvent -LogName "Application" -FilterXPath "*[System[Provider[@Name='Microsoft.Azure.ActiveDirectory.Agent.PassthroughAuthentication']]]" | 
    Select-Object -First 10

# PTA Troubleshooting
Test-NetConnection -ComputerName "login.microsoftonline.com" -Port 443
Test-NetConnection -ComputerName "autologon.microsoftazuread-sso.com" -Port 443
```

### Seamless SSO Configuration
```powershell
# Enable Seamless SSO during Azure AD Connect setup
# Or configure manually:

# Create computer account in AD
$AzureADSSOComputerAccount = "AZUREADSSOACC"
$Password = ConvertTo-SecureString "ComplexPassword123!" -AsPlainText -Force

New-ADComputer -Name $AzureADSSOComputerAccount `
    -Path "CN=Computers,DC=corp,DC=local" `
    -ServiceAccount "CORP\svc_aadconnect" `
    -AccountPassword $Password `
    -Enabled $true

# Configure SPN
setspn -a HOST/$AzureADSSOComputerAccount.corp.local CORP\$AzureADSSOComputerAccount
setspn -a RestrictedKrbHost/$AzureADSSOComputerAccount.corp.local CORP\$AzureADSSOComputerAccount

# Configure Group Policy for SSO
# Create GPO to add Azure AD URLs to Intranet Zone
Set-GPRegistryValue -Name "Seamless-SSO-Policy" `
    -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\microsoftazuread-sso.com\autologon" `
    -ValueName "https" `
    -Type DWord `
    -Value 1
```

---

## 6. AD Automation with PowerShell and Ansible <a name="ad-automation"></a>

### PowerShell AD Automation Module
```powershell
# ADAutomation.psm1

function New-StandardUser {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$FirstName,
        
        [Parameter(Mandatory=$true)]
        [string]$LastName,
        
        [Parameter(Mandatory=$true)]
        [string]$Department,
        
        [Parameter(Mandatory=$true)]
        [string]$Title,
        
        [Parameter(Mandatory=$false)]
        [string]$Manager,
        
        [Parameter(Mandatory=$false)]
        [string[]]$Groups
    )
    
    $SamAccountName = "$($FirstName.Substring(0,1))$LastName".ToLower()
    $UPN = "$SamAccountName@corp.local"
    $Password = New-RandomPassword -Length 16
    
    # Map department to OU
    $OUMapping = @{
        "IT"       = "OU=IT,OU=Users,OU=Corp,DC=corp,DC=local"
        "HR"       = "OU=HR,OU=Users,OU=Corp,DC=corp,DC=local"
        "Finance"  = "OU=Finance,OU=Users,OU=Corp,DC=corp,DC=local"
        "Sales"    = "OU=Sales,OU=Users,OU=Corp,DC=corp,DC=local"
        "Default"  = "OU=Users,OU=Corp,DC=corp,DC=local"
    }
    
    $TargetOU = if ($OUMapping.ContainsKey($Department)) { 
        $OUMapping[$Department] 
    } else { 
        $OUMapping["Default"] 
    }
    
    try {
        $UserParams = @{
            SamAccountName        = $SamAccountName
            UserPrincipalName     = $UPN
            Name                  = "$FirstName $LastName"
            GivenName             = $FirstName
            Surname               = $LastName
            DisplayName           = "$FirstName $LastName"
            EmailAddress          = "$SamAccountName@corp.com"
            Department            = $Department
            Title                 = $Title
            Path                  = $TargetOU
            AccountPassword       = (ConvertTo-SecureString $Password -AsPlainText -Force)
            Enabled               = $true
            ChangePasswordAtLogon = $true
        }
        
        if ($Manager) {
            $ManagerDN = (Get-ADUser -Identity $Manager).DistinguishedName
            $UserParams.Add("Manager", $ManagerDN)
        }
        
        New-ADUser @UserParams
        
        # Add to groups
        if ($Groups) {
            foreach ($Group in $Groups) {
                Add-ADGroupMember -Identity $Group -Members $SamAccountName
            }
        }
        
        # Add to default department group
        $DeptGroup = "$Department-Users"
        if (Get-ADGroup -Filter "Name -eq '$DeptGroup'" -ErrorAction SilentlyContinue) {
            Add-ADGroupMember -Identity $DeptGroup -Members $SamAccountName
        }
        
        # Create home folder
        $HomePath = "\\fileserver\homes\$SamAccountName"
        New-Item -ItemType Directory -Path $HomePath -Force
        
        # Set NTFS permissions
        $ACL = Get-Acl $HomePath
        $User = "CORP\$SamAccountName"
        $Permission = $User, "FullControl", "ContainerInherit,ObjectInherit", "None", "Allow"
        $AccessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $Permission
        $ACL.SetAccessRule($AccessRule)
        Set-Acl -Path $HomePath -AclObject $ACL
        
        return @{
            Success      = $true
            Username     = $SamAccountName
            Password     = $Password
            UPN          = $UPN
            OU           = $TargetOU
            HomePath     = $HomePath
        }
    }
    catch {
        return @{
            Success = $false
            Error   = $_.Exception.Message
        }
    }
}

function New-RandomPassword {
    param([int]$Length = 16)
    
    $CharSet = @{
        Uppercase = (65..90) | ForEach-Object { [char]$_ }
        Lowercase = (97..122) | ForEach-Object { [char]$_ }
        Numbers   = (48..57) | ForEach-Object { [char]$_ }
        Special   = '!@#$%^&*()_+-=[]{}|;:,.<>?'.ToCharArray()
    }
    
    $Password = @()
    $Password += $CharSet.Uppercase | Get-Random -Count 2
    $Password += $CharSet.Lowercase | Get-Random -Count 2
    $Password += $CharSet.Numbers | Get-Random -Count 2
    $Password += $CharSet.Special | Get-Random -Count 2
    
    $AllChars = $CharSet.Uppercase + $CharSet.Lowercase + $CharSet.Numbers + $CharSet.Special
    $Password += $AllChars | Get-Random -Count ($Length - 8)
    
    return ($Password | Get-Random -Count $Password.Count) -join ''
}

function Disable-TerminatedUser {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$SamAccountName,
        
        [Parameter(Mandatory=$false)]
        [string]$Reason = "Terminated"
    )
    
    try {
        $User = Get-ADUser -Identity $SamAccountName -Properties *
        
        # Disable account
        Disable-ADAccount -Identity $SamAccountName
        
        # Remove from all groups except Domain Users
        $Groups = Get-ADPrincipalGroupMembership -Identity $SamAccountName | 
            Where-Object { $_.Name -ne "Domain Users" }
        
        foreach ($Group in $Groups) {
            Remove-ADGroupMember -Identity $Group -Members $SamAccountName -Confirm:$false
        }
        
        # Move to Disabled Users OU
        $DisabledOU = "OU=Disabled Users,OU=Corp,DC=corp,DC=local"
        Move-ADObject -Identity $User.DistinguishedName -TargetPath $DisabledOU
        
        # Update description
        $NewDescription = "DISABLED: $(Get-Date -Format 'yyyy-MM-dd') - $Reason - Original: $($User.Description)"
        Set-ADUser -Identity $SamAccountName -Description $NewDescription
        
        # Hide from GAL (if Exchange attribute exists)
        Set-ADUser -Identity $SamAccountName -Replace @{msExchHideFromAddressLists=$true}
        
        # Log the action
        $LogEntry = @{
            Timestamp   = Get-Date
            Action      = "UserDisabled"
            Username    = $SamAccountName
            Reason      = $Reason
            PerformedBy = $env:USERNAME
        }
        
        $LogEntry | Export-Csv "C:\Logs\UserTerminations.csv" -Append -NoTypeInformation
        
        return @{
            Success     = $true
            Username    = $SamAccountName
            NewLocation = "$DisabledOU"
        }
    }
    catch {
        return @{
            Success = $false
            Error   = $_.Exception.Message
        }
    }
}

Export-ModuleMember -Function *
```

### Ansible AD Automation
```yaml
# playbooks/ad-management.yml
---
- name: Active Directory Management
  hosts: domain_controllers
  gather_facts: yes
  
  vars:
    domain_name: "corp.local"
    domain_admin: "CORP\\Administrator"
    
  tasks:
    - name: Create Organizational Units
      win_domain_ou:
        name: "{{ item.name }}"
        path: "{{ item.path }}"
        state: present
        protected: yes
      loop:
        - { name: "Servers", path: "OU=Corp,DC=corp,DC=local" }
        - { name: "Workstations", path: "OU=Corp,DC=corp,DC=local" }
        - { name: "Users", path: "OU=Corp,DC=corp,DC=local" }
        - { name: "Groups", path: "OU=Corp,DC=corp,DC=local" }
        - { name: "Service Accounts", path: "OU=Corp,DC=corp,DC=local" }
      
    - name: Create Security Groups
      win_domain_group:
        name: "{{ item.name }}"
        description: "{{ item.description }}"
        scope: global
        category: security
        organizational_unit: "OU=Groups,OU=Corp,DC=corp,DC=local"
        state: present
      loop:
        - { name: "IT-Admins", description: "IT Administrators" }
        - { name: "Server-Admins", description: "Server Administrators" }
        - { name: "Help-Desk", description: "Help Desk Staff" }
        - { name: "VPN-Users", description: "VPN Access Group" }
        
    - name: Create Users from CSV
      win_domain_user:
        name: "{{ item.sam_account_name }}"
        upn: "{{ item.sam_account_name }}@corp.local"
        firstname: "{{ item.first_name }}"
        surname: "{{ item.last_name }}"
        email: "{{ item.email }}"
        password: "{{ item.password }}"
        state: present
        password_never_expires: no
        user_cannot_change_password: no
        enabled: yes
        path: "OU=Users,OU=Corp,DC=corp,DC=local"
        groups:
          - "{{ item.department }}-Users"
      loop: "{{ lookup('file', 'users.json') | from_json }}"
      no_log: true
      
    - name: Configure Group Policy Link
      win_gpo_link:
        gpo_name: "Security-Baseline-Servers"
        target_ou: "OU=Servers,OU=Corp,DC=corp,DC=local"
        state: present
        enforced: yes
        enabled: yes

---
# playbooks/ad-health-check.yml
- name: Active Directory Health Check
  hosts: domain_controllers
  gather_facts: yes
  
  tasks:
    - name: Check AD Services
      win_service:
        name: "{{ item }}"
      register: ad_services
      loop:
        - NTDS
        - DNS
        - Netlogon
        - W32Time
        - DFSR
      
    - name: Verify SYSVOL Replication
      win_shell: |
        repadmin /replsummary
      register: repl_status
      changed_when: false
      
    - name: Check FSMO Roles
      win_shell: |
        netdom query fsmo
      register: fsmo_roles
      changed_when: false
      
    - name: Test DNS Resolution
      win_shell: |
        nslookup corp.local
        nslookup _ldap._tcp.dc._msdcs.corp.local
      register: dns_test
      changed_when: false
      
    - name: Generate Health Report
      template:
        src: ad-health-report.html.j2
        dest: "C:\\Reports\\AD-Health-{{ ansible_date_time.date }}.html"
```

---

## 7. Migration Scenarios <a name="migration-scenarios"></a>

### Scenario 1: On-Premises AD to Azure AD Hybrid
```
Migration Phases:
═══════════════════════════════════════════════════════════════════

Phase 1: Assessment (2 weeks)
├── Inventory current AD structure
├── Document GPOs and custom configurations
├── Identify applications and authentication dependencies
├── Assess network connectivity requirements
└── Create migration plan

Phase 2: Preparation (2 weeks)
├── Deploy Azure AD Connect server
├── Configure firewall rules for hybrid connectivity
├── Test password hash sync
├── Implement Seamless SSO
└── Configure Conditional Access policies

Phase 3: Pilot Migration (2 weeks)
├── Select pilot user group (IT department)
├── Enable hybrid join for pilot devices
├── Test application access
├── Validate MFA enrollment
└── Document issues and resolutions

Phase 4: Production Migration (4-8 weeks)
├── Rolling deployment by department
├── Configure device hybrid join
├── Migrate applications to Azure AD authentication
├── Implement PIM for privileged access
└── Enable security features (Identity Protection)

Phase 5: Optimization (Ongoing)
├── Monitor sign-in logs and security alerts
├── Optimize Conditional Access policies
├── Implement Azure AD Access Reviews
└── Plan for passwordless authentication
```

### Scenario 2: AD Domain Consolidation
```powershell
# Domain Consolidation Automation Script
# Migrate users and computers from child domain to parent domain

# Step 1: Inventory Source Domain
$SourceDomain = "child.corp.local"
$TargetDomain = "corp.local"

$UsersToMigrate = Get-ADUser -Filter * -Server $SourceDomain -Properties *
$ComputersToMigrate = Get-ADComputer -Filter * -Server $SourceDomain -Properties *
$GroupsToMigrate = Get-ADGroup -Filter * -Server $SourceDomain -Properties *

# Step 2: Create Migration Mapping
$MigrationMap = @()
foreach ($User in $UsersToMigrate) {
    $MigrationMap += [PSCustomObject]@{
        SourceDN      = $User.DistinguishedName
        SourceSAM     = $User.SamAccountName
        TargetOU      = "OU=Migrated,OU=Users,DC=corp,DC=local"
        TargetSAM     = $User.SamAccountName
        SIDHistory    = $User.SID
    }
}

# Step 3: Use ADMT for migration (PowerShell wrapper)
function Invoke-ADMTUserMigration {
    param(
        [string]$SourceDomain,
        [string]$TargetDomain,
        [string]$SourceOU,
        [string]$TargetOU,
        [switch]$MigrateSIDHistory
    )
    
    $ADMTPath = "C:\Program Files\Active Directory Migration Tool\admt.exe"
    
    $Arguments = @(
        "USER"
        "/SD:$SourceDomain"
        "/TD:$TargetDomain"
        "/SO:$SourceOU"
        "/TO:$TargetOU"
        "/MSS:YES"  # Migrate SID History
        "/MCO:REPLACE"  # Migrate and replace conflicting objects
        "/TRP:YES"  # Translate roaming profiles
        "/RES:YES"  # Rename with suffix if conflict
    )
    
    & $ADMTPath $Arguments
}

# Step 4: Update GPO Links
function Update-GPOLinks {
    param([string]$NewOU)
    
    $GPOs = Get-GPO -All -Domain $SourceDomain
    foreach ($GPO in $GPOs) {
        $Links = Get-GPInheritance -Target "OU=Users,$SourceDomain"
        foreach ($Link in $Links.GpoLinks) {
            if ($Link.GpoId -eq $GPO.Id) {
                # Create new link in target domain
                New-GPLink -Guid $GPO.Id -Target $NewOU -Domain $TargetDomain
            }
        }
    }
}
```

### Scenario 3: Multi-Forest Trust Migration
```powershell
# Configure Forest Trust
$SourceForest = "partner.com"
$TargetForest = "corp.local"
$TrustPassword = Read-Host -AsSecureString "Enter Trust Password"

# Create Forest Trust
netdom trust $TargetForest /domain:$SourceForest /add /twoway /transitive /passwordt:$TrustPassword

# Verify Trust
Get-ADTrust -Filter * | Where-Object { $_.Target -eq $SourceForest }

# Configure Selective Authentication (more secure)
netdom trust $TargetForest /domain:$SourceForest /SelectiveAuth:YES

# Grant access to specific resources
$AllowedToAuthenticate = @(
    "CN=Partner-AppServers,OU=Groups,DC=partner,DC=com"
)

foreach ($Group in $AllowedToAuthenticate) {
    $Server = "CN=AppServer01,OU=Servers,DC=corp,DC=local"
    dsacls $Server /G "${Group}:CA;Allowed To Authenticate"
}
```

---

## 8. Interview Scenarios and Use Cases <a name="interview-scenarios"></a>

### Use Case 1: Enterprise AD Consolidation

**Challenge**: Consolidate 5 separate AD domains into single forest after company acquisitions

**Solution**:
```
Project Timeline: 18 months
─────────────────────────────────────────────────────────────────

Phase 1: Discovery (3 months)
├── Inventory all domains (users, computers, applications)
├── Document trust relationships
├── Map application dependencies
├── Identify conflicts (duplicate usernames, SID conflicts)
└── Create detailed migration plan

Phase 2: Infrastructure (2 months)
├── Deploy new forest root domain (corp.global)
├── Establish forest trusts with all existing domains
├── Deploy additional domain controllers
├── Configure Azure AD Connect for hybrid identity
└── Implement global DNS strategy

Phase 3: Application Migration (4 months)
├── Re-configure applications for new domain
├── Update Kerberos delegation settings
├── Migrate service accounts to gMSAs
├── Test application functionality
└── Update firewall rules and GPOs

Phase 4: User Migration (6 months)
├── Wave-based migration by business unit
├── SID History migration for file/share access
├── Profile migration using USMT
├── Group membership reconciliation
└── Email/Exchange migration coordination

Phase 5: Decommission (3 months)
├── Verify no dependencies on old domains
├── Demote old domain controllers
├── Remove trust relationships
├── Clean up DNS records
└── Archive old domain data

Results:
├── Reduced from 5 domains to 1 (80% reduction)
├── Single identity for all 15,000 users
├── Simplified GPO management (45 GPOs → 15)
├── Annual cost savings of $500K (reduced infrastructure)
└── Improved security posture (single policy enforcement)
```

### Use Case 2: Zero-Trust GPO Implementation

**Challenge**: Implement zero-trust security model using Group Policy

**Implementation**:
```powershell
# GPO Structure for Zero Trust
$ZeroTrustGPOs = @(
    @{
        Name = "ZT-Device-Compliance"
        Settings = @{
            # BitLocker enforcement
            "Computer\Policies\Administrative Templates\Windows Components\BitLocker Drive Encryption" = @{
                "Require additional authentication at startup" = "Enabled"
                "Choose drive encryption method and cipher strength" = "XTS-AES 256-bit"
            }
            # Windows Defender
            "Computer\Policies\Administrative Templates\Windows Components\Windows Defender Antivirus" = @{
                "Turn off Windows Defender Antivirus" = "Disabled"
                "Configure real-time protection" = "Enabled"
            }
        }
    },
    @{
        Name = "ZT-Network-Segmentation"
        Settings = @{
            # Windows Firewall
            "Block all inbound by default" = $true
            "Log dropped packets" = $true
            "IPsec enforcement" = "Required"
        }
    },
    @{
        Name = "ZT-Identity-Protection"
        Settings = @{
            # Credential Guard
            "Turn On Virtualization Based Security" = "Enabled"
            "Credential Guard Configuration" = "Enabled with UEFI lock"
            # Protected Users group enforcement
            "Add accounts to Protected Users" = $true
        }
    }
)

# Create and configure GPOs
foreach ($GPO in $ZeroTrustGPOs) {
    $NewGPO = New-GPO -Name $GPO.Name
    # Configure settings using Set-GPRegistryValue or LGPO
    # Link to appropriate OUs
    New-GPLink -Guid $NewGPO.Id -Target "OU=Corp,DC=corp,DC=local" -LinkEnabled Yes
}
```

### Common Interview Questions

**Q1: "How do you handle AD replication issues?"**

**Answer (Systematic Approach)**:
```powershell
# 1. Check replication status
repadmin /replsummary
repadmin /showrepl

# 2. Identify failing DCs
repadmin /replsummary /bysrc /bydest /sort:delta

# 3. Check for lingering objects
repadmin /removelingeringobjects <dest_dc> <source_dc_guid> <nc>

# 4. Force replication
repadmin /syncall /APed

# 5. Check SYSVOL replication (DFSR)
dfsrdiag pollad
dfsrmig /getglobalstate

# 6. Verify DNS
dcdiag /test:dns /e /v

# Common causes and fixes:
# - Time sync issues: w32tm /resync /force
# - Network connectivity: Test-NetConnection
# - USN rollback: Restore from backup
# - Lingering objects: repadmin /removelingeringobjects
```

**Q2: "Explain your approach to GPO troubleshooting"**

**Answer**:
```powershell
# 1. Get Resultant Set of Policy
gpresult /r
gpresult /h C:\Temp\GPResult.html

# 2. Check GPO application order
Get-GPInheritance -Target "OU=Servers,DC=corp,DC=local"

# 3. Verify WMI filtering
Get-GPO -Name "MyGPO" | Get-GPOReport -ReportType XML

# 4. Check security filtering
Get-GPPermission -Name "MyGPO" -All

# 5. Test WMI filter
Get-WmiObject -Query "SELECT * FROM Win32_OperatingSystem WHERE Version LIKE '10%'"

# 6. Enable GPO debug logging
# HKLM\Software\Microsoft\Windows NT\CurrentVersion\Diagnostics
# GPSvcDebugLevel = 0x30002 (DWORD)

# 7. Review event logs
Get-WinEvent -LogName "Microsoft-Windows-GroupPolicy/Operational" | 
    Where-Object { $_.TimeCreated -gt (Get-Date).AddHours(-1) }
```

**Q3: "How do you secure Active Directory?"**

**Answer (Tiered Administration Model)**:
```
Tier 0 - Domain Controllers and Identity
├── Dedicated admin accounts (T0-Admin_*)
├── Privileged Access Workstations (PAWs)
├── Protected Users group membership
├── Credential Guard enabled
├── No internet access
└── Just-in-time access via PIM

Tier 1 - Server Administration
├── Separate admin accounts (T1-Admin_*)
├── LAPS for local admin passwords
├── Restricted admin mode for RDP
├── AppLocker/WDAC policies
└── Network segmentation

Tier 2 - Workstation Administration
├── Standard user accounts
├── Local admin removed from users
├── UAC enforced
├── Windows Defender ATP
└── BitLocker encryption

Security Controls:
├── gMSA for service accounts
├── Fine-grained password policies
├── Audit policy (all privileged operations)
├── AdminSDHolder monitoring
└── Regular AD security assessments
```

---

## Quick Reference Commands

```powershell
# AD Health
dcdiag /v
repadmin /showrepl
nltest /dclist:corp.local

# User Management
Get-ADUser -Identity username -Properties *
Set-ADUser -Identity username -Enabled $false
Unlock-ADAccount -Identity username

# Group Management
Get-ADGroupMember -Identity "Domain Admins" -Recursive
Add-ADGroupMember -Identity "GroupName" -Members "Username"

# GPO Management
gpupdate /force
gpresult /r
Get-GPO -All | Select-Object DisplayName, GpoStatus

# Azure AD
Connect-AzureAD
Get-AzureADUser -ObjectId user@domain.com
Get-AzureADGroup -SearchString "GroupName"

# Hybrid Identity
Start-ADSyncSyncCycle -PolicyType Delta
Get-ADSyncScheduler
```
