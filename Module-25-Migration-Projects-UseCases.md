# Module 25: Migration Projects and Use Cases

## Table of Contents
1. [Migration Strategy Framework](#migration-framework)
2. [Datacenter to Cloud Migration](#datacenter-migration)
3. [Active Directory Consolidation and Migration](#ad-migration)
4. [Application Modernization](#app-modernization)
5. [VM to Container Migration](#vm-to-container)
6. [Legacy System Migration](#legacy-migration)
7. [Multi-Phase Migration Planning](#migration-planning)
8. [Interview Scenarios](#interview-scenarios)

---

## 1. Migration Strategy Framework <a name="migration-framework"></a>

### The 6 R's of Migration
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Cloud Migration Strategies (6 R's)                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  1. REHOST (Lift and Shift)                                          │   │
│  │     • Quickest migration path                                        │   │
│  │     • Minimal code changes                                           │   │
│  │     • IaaS approach (VMs to Cloud VMs)                               │   │
│  │     • Good for: Legacy apps, time-critical migrations                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  2. REPLATFORM (Lift and Reshape)                                    │   │
│  │     • Minor optimizations                                            │   │
│  │     • Leverage managed services (e.g., RDS instead of self-hosted)   │   │
│  │     • Moderate changes, significant benefits                         │   │
│  │     • Good for: Databases, middleware                                │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  3. REPURCHASE (Replace)                                             │   │
│  │     • Move to SaaS solution                                          │   │
│  │     • Replace custom with commercial                                 │   │
│  │     • Good for: CRM, HR systems, email                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  4. REFACTOR (Re-architect)                                          │   │
│  │     • Full application redesign                                      │   │
│  │     • Cloud-native approach (microservices, serverless)              │   │
│  │     • Maximum cloud benefits, highest effort                         │   │
│  │     • Good for: Strategic applications, scalability needs            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  5. RETIRE                                                           │   │
│  │     • Decommission applications                                      │   │
│  │     • Reduce portfolio complexity                                    │   │
│  │     • Good for: Redundant, legacy, low-value apps                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  6. RETAIN                                                           │   │
│  │     • Keep in current environment                                    │   │
│  │     • Not ready or not feasible to migrate                           │   │
│  │     • Good for: Mainframes, compliance restrictions                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Migration Assessment Framework
```python
# migration_assessment.py
from dataclasses import dataclass
from typing import List, Dict, Optional
from enum import Enum
import json

class MigrationStrategy(Enum):
    REHOST = "rehost"
    REPLATFORM = "replatform"
    REPURCHASE = "repurchase"
    REFACTOR = "refactor"
    RETIRE = "retire"
    RETAIN = "retain"

@dataclass
class Application:
    name: str
    owner: str
    technology_stack: List[str]
    dependencies: List[str]
    business_criticality: str  # low, medium, high, critical
    complexity: str  # low, medium, high
    compliance_requirements: List[str]
    current_infrastructure: Dict
    monthly_cost: float
    users: int
    data_sensitivity: str

@dataclass
class MigrationRecommendation:
    application: Application
    recommended_strategy: MigrationStrategy
    target_architecture: str
    estimated_effort_days: int
    estimated_cost: float
    risks: List[str]
    prerequisites: List[str]
    migration_wave: int

class MigrationAssessor:
    def __init__(self):
        self.strategy_weights = {
            "business_criticality": {"low": 1, "medium": 2, "high": 3, "critical": 4},
            "complexity": {"low": 1, "medium": 2, "high": 3}
        }
    
    def assess_application(self, app: Application) -> MigrationRecommendation:
        """Assess application and recommend migration strategy"""
        
        # Calculate scores for each strategy
        scores = {
            MigrationStrategy.REHOST: self._score_rehost(app),
            MigrationStrategy.REPLATFORM: self._score_replatform(app),
            MigrationStrategy.REFACTOR: self._score_refactor(app),
            MigrationStrategy.REPURCHASE: self._score_repurchase(app),
            MigrationStrategy.RETIRE: self._score_retire(app),
            MigrationStrategy.RETAIN: self._score_retain(app)
        }
        
        # Get recommended strategy
        recommended = max(scores, key=scores.get)
        
        return MigrationRecommendation(
            application=app,
            recommended_strategy=recommended,
            target_architecture=self._get_target_architecture(app, recommended),
            estimated_effort_days=self._estimate_effort(app, recommended),
            estimated_cost=self._estimate_cost(app, recommended),
            risks=self._identify_risks(app, recommended),
            prerequisites=self._get_prerequisites(app, recommended),
            migration_wave=self._assign_wave(app)
        )
    
    def _score_rehost(self, app: Application) -> float:
        """Score suitability for lift-and-shift"""
        score = 50  # Base score
        
        # Favorable factors
        if app.complexity == "low":
            score += 20
        if "windows" in app.technology_stack or "linux" in app.technology_stack:
            score += 15
        if app.business_criticality in ["low", "medium"]:
            score += 10
        
        # Unfavorable factors
        if len(app.dependencies) > 5:
            score -= 15
        if "mainframe" in app.technology_stack:
            score -= 30
        
        return score
    
    def _score_replatform(self, app: Application) -> float:
        """Score suitability for replatforming"""
        score = 50
        
        # Good candidates for replatforming
        if any(db in app.technology_stack for db in ["sql", "oracle", "mysql", "postgresql"]):
            score += 25
        if app.complexity == "medium":
            score += 15
        
        return score
    
    def _score_refactor(self, app: Application) -> float:
        """Score suitability for refactoring"""
        score = 30  # Lower base - high effort
        
        # Good candidates for refactoring
        if app.business_criticality in ["high", "critical"]:
            score += 30
        if any(stack in app.technology_stack for stack in ["java", "dotnet", "python"]):
            score += 20
        if app.users > 10000:
            score += 15
        
        return score
    
    def _score_repurchase(self, app: Application) -> float:
        """Score suitability for SaaS replacement"""
        score = 40
        
        # Good candidates for SaaS
        if any(cat in app.name.lower() for cat in ["crm", "hr", "email", "erp"]):
            score += 35
        if app.business_criticality in ["low", "medium"]:
            score += 15
        
        return score
    
    def _score_retire(self, app: Application) -> float:
        """Score suitability for retirement"""
        score = 20
        
        # Retirement candidates
        if app.users < 10:
            score += 40
        if app.business_criticality == "low":
            score += 30
        
        return score
    
    def _score_retain(self, app: Application) -> float:
        """Score suitability for retaining on-prem"""
        score = 30
        
        # Keep on-prem indicators
        if "data_residency" in app.compliance_requirements:
            score += 40
        if "mainframe" in app.technology_stack:
            score += 35
        if any(req in app.compliance_requirements for req in ["hipaa", "pci"]):
            score += 20
        
        return score
    
    def _get_target_architecture(self, app: Application, strategy: MigrationStrategy) -> str:
        """Define target architecture based on strategy"""
        architectures = {
            MigrationStrategy.REHOST: "Azure VMs with managed disks",
            MigrationStrategy.REPLATFORM: "Azure SQL Database + App Service",
            MigrationStrategy.REFACTOR: "AKS with microservices",
            MigrationStrategy.REPURCHASE: "SaaS solution",
            MigrationStrategy.RETIRE: "Decommission",
            MigrationStrategy.RETAIN: "On-premises (modernize in place)"
        }
        return architectures.get(strategy, "TBD")
    
    def _estimate_effort(self, app: Application, strategy: MigrationStrategy) -> int:
        """Estimate migration effort in days"""
        base_effort = {
            MigrationStrategy.REHOST: 10,
            MigrationStrategy.REPLATFORM: 30,
            MigrationStrategy.REFACTOR: 90,
            MigrationStrategy.REPURCHASE: 20,
            MigrationStrategy.RETIRE: 5,
            MigrationStrategy.RETAIN: 0
        }
        
        effort = base_effort.get(strategy, 30)
        
        # Adjust for complexity
        complexity_multiplier = {"low": 0.8, "medium": 1.0, "high": 1.5}
        effort *= complexity_multiplier.get(app.complexity, 1.0)
        
        # Adjust for dependencies
        effort += len(app.dependencies) * 2
        
        return int(effort)
    
    def _assign_wave(self, app: Application) -> int:
        """Assign to migration wave"""
        if app.business_criticality == "low" and app.complexity == "low":
            return 1  # Early adopters
        elif app.business_criticality in ["low", "medium"]:
            return 2  # Foundation wave
        elif app.business_criticality == "high":
            return 3  # Core business
        else:
            return 4  # Critical systems
```

---

## 2. Datacenter to Cloud Migration <a name="datacenter-migration"></a>

### Migration Project Plan
```yaml
# datacenter-migration-plan.yaml
project:
  name: "Datacenter Exit to Azure"
  duration: "18 months"
  scope:
    servers: 500
    applications: 150
    data_volume: "200TB"
    datacenters: 2

phases:
  phase1_discovery:
    duration: "8 weeks"
    activities:
      - name: "Deploy Azure Migrate appliance"
        deliverables:
          - "Server inventory"
          - "Dependency mapping"
          - "Performance data"
      - name: "Application portfolio assessment"
        deliverables:
          - "Application catalog"
          - "Business owner mapping"
          - "Criticality classification"
      - name: "Network assessment"
        deliverables:
          - "Bandwidth requirements"
          - "Latency analysis"
          - "Firewall rules documentation"
    
  phase2_foundation:
    duration: "6 weeks"
    activities:
      - name: "Landing Zone deployment"
        deliverables:
          - "Hub VNet with Azure Firewall"
          - "ExpressRoute connectivity"
          - "Identity integration"
      - name: "Security baseline"
        deliverables:
          - "Azure Policy assignments"
          - "Microsoft Defender for Cloud"
          - "Log Analytics workspace"
      - name: "Operations setup"
        deliverables:
          - "Monitoring dashboards"
          - "Backup policies"
          - "Update management"
    
  phase3_pilot:
    duration: "4 weeks"
    wave_scope: "10 non-critical servers"
    activities:
      - name: "Pilot migration"
        steps:
          - "Replicate VMs with Azure Migrate"
          - "Test failover"
          - "Application validation"
          - "Performance testing"
          - "Production cutover"
      - name: "Process refinement"
        deliverables:
          - "Runbook updates"
          - "Team training"
          - "Risk register update"
    
  phase4_migration_waves:
    waves:
      - wave: 1
        duration: "6 weeks"
        servers: 50
        strategy: "Rehost"
        criteria:
          - "Low complexity"
          - "No external dependencies"
          - "Non-production workloads"
        
      - wave: 2
        duration: "8 weeks"
        servers: 100
        strategy: "Rehost + Replatform"
        criteria:
          - "Medium complexity"
          - "Internal applications"
          - "SQL Server databases"
        
      - wave: 3
        duration: "10 weeks"
        servers: 150
        strategy: "Rehost + Replatform"
        criteria:
          - "Business applications"
          - "External integrations"
          - "Complex dependencies"
        
      - wave: 4
        duration: "12 weeks"
        servers: 150
        strategy: "Rehost + Replatform + Refactor"
        criteria:
          - "Critical applications"
          - "High availability requirements"
          - "Complex architectures"
        
      - wave: 5
        duration: "6 weeks"
        servers: 50
        strategy: "Final migration"
        criteria:
          - "Domain controllers"
          - "Final holdouts"
          - "Datacenter exit"

  phase5_optimization:
    duration: "Ongoing"
    activities:
      - "Right-sizing recommendations"
      - "Reserved Instance purchases"
      - "Modernization planning"
```

### Azure Migrate Automation
```python
# azure_migrate_automation.py
from azure.identity import DefaultAzureCredential
from azure.mgmt.migrate import AzureMigrateV2
from azure.mgmt.recoveryservices import RecoveryServicesClient
import json
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class ServerMigration:
    server_name: str
    source_ip: str
    target_resource_group: str
    target_vnet: str
    target_subnet: str
    target_vm_size: str
    os_disk_type: str
    data_disks: List[Dict]

class MigrationOrchestrator:
    def __init__(self, subscription_id: str, migrate_project: str):
        self.credential = DefaultAzureCredential()
        self.subscription_id = subscription_id
        self.migrate_project = migrate_project
        self.migrate_client = AzureMigrateV2(
            self.credential, 
            subscription_id
        )
        
    def get_discovered_servers(self) -> List[Dict]:
        """Get all discovered servers from Azure Migrate"""
        servers = []
        
        # Query discovered machines
        machines = self.migrate_client.machines.list_by_project(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project
        )
        
        for machine in machines:
            server_info = {
                "name": machine.display_name,
                "ip_addresses": machine.ip_addresses,
                "operating_system": machine.operating_system_details.os_name,
                "cores": machine.number_of_cores,
                "memory_mb": machine.memory_in_mb,
                "disks": [
                    {
                        "name": disk.name,
                        "size_gb": disk.max_size_in_bytes / (1024**3)
                    }
                    for disk in machine.disks
                ],
                "dependencies": self._get_dependencies(machine.id)
            }
            servers.append(server_info)
        
        return servers
    
    def _get_dependencies(self, machine_id: str) -> List[str]:
        """Get application dependencies for a machine"""
        dependencies = self.migrate_client.dependencies.list_by_machine(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project,
            machine_name=machine_id
        )
        
        return [dep.destination_machine_name for dep in dependencies]
    
    def create_migration_group(
        self, 
        group_name: str, 
        servers: List[str]
    ) -> Dict:
        """Create a migration group for batch processing"""
        
        group = self.migrate_client.groups.create(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project,
            group_name=group_name,
            properties={
                "machines": servers
            }
        )
        
        return {
            "name": group.name,
            "id": group.id,
            "machine_count": len(servers)
        }
    
    def start_replication(
        self, 
        server: ServerMigration
    ) -> Dict:
        """Start replication for a server"""
        
        replication_config = {
            "properties": {
                "machineId": f"/subscriptions/{self.subscription_id}/...",
                "targetResourceGroupId": f"/subscriptions/{self.subscription_id}/resourceGroups/{server.target_resource_group}",
                "targetNetworkId": f"/subscriptions/{self.subscription_id}/.../virtualNetworks/{server.target_vnet}",
                "targetSubnetName": server.target_subnet,
                "targetVmName": server.server_name,
                "targetVmSize": server.target_vm_size,
                "targetOsDiskType": server.os_disk_type,
                "diskType": "Standard_LRS",
                "licenseType": "WindowsServer"  # Azure Hybrid Benefit
            }
        }
        
        # Start replication
        result = self.migrate_client.replicating_machines.create(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project,
            machine_name=server.server_name,
            body=replication_config
        )
        
        return {
            "name": result.name,
            "status": result.properties.migration_state,
            "replication_progress": result.properties.replication_progress_percentage
        }
    
    def execute_test_failover(
        self, 
        server_name: str, 
        test_network: str
    ) -> Dict:
        """Execute test failover to validate migration"""
        
        result = self.migrate_client.replicating_machines.test_migrate(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project,
            machine_name=server_name,
            properties={
                "testNetworkId": test_network
            }
        )
        
        return {
            "status": "test_failover_initiated",
            "job_id": result.id
        }
    
    def execute_migration(
        self, 
        server_name: str,
        shutdown_source: bool = True
    ) -> Dict:
        """Execute final migration cutover"""
        
        result = self.migrate_client.replicating_machines.migrate(
            resource_group_name="migrate-rg",
            project_name=self.migrate_project,
            machine_name=server_name,
            properties={
                "shutdownSourceMachine": shutdown_source
            }
        )
        
        return {
            "status": "migration_initiated",
            "job_id": result.id
        }
```

---

## 3. Active Directory Consolidation and Migration <a name="ad-migration"></a>

### AD Migration Scenarios
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Active Directory Migration Scenarios                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Scenario 1: Forest Consolidation                                           │
│  ┌─────────────┐    ┌─────────────┐         ┌─────────────┐                │
│  │ Forest A    │    │ Forest B    │   ──▶   │ Target      │                │
│  │ corp.local  │    │ acme.local  │         │ Forest      │                │
│  │ 5,000 users │    │ 3,000 users │         │ newcorp.com │                │
│  └─────────────┘    └─────────────┘         │ 8,000 users │                │
│                                              └─────────────┘                │
│                                                                              │
│  Scenario 2: Domain Upgrade (In-Place)                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Server 2012 R2 DCs  ──▶  Server 2019 DCs  ──▶  Server 2022 DCs     │   │
│  │ DFL/FFL: 2012 R2    ──▶  DFL/FFL: 2016    ──▶  DFL/FFL: 2016       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  Scenario 3: Hybrid Identity (On-Prem + Azure AD)                           │
│  ┌─────────────┐                          ┌─────────────┐                  │
│  │ On-Prem AD  │    Azure AD Connect      │ Azure AD    │                  │
│  │ corp.local  │ ◄─────────────────────▶  │ (Entra ID)  │                  │
│  │             │    Password Hash Sync    │             │                  │
│  └─────────────┘    or PTA/Federation     └─────────────┘                  │
│                                                                              │
│  Scenario 4: Cloud-Only Migration                                           │
│  ┌─────────────┐                          ┌─────────────┐                  │
│  │ On-Prem AD  │    ADMT + Cloud Move     │ Azure AD    │                  │
│  │ corp.local  │ ────────────────────▶    │ (Entra ID)  │                  │
│  │             │    Retire On-Prem        │ AAD DS      │                  │
│  └─────────────┘                          └─────────────┘                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### AD Migration Automation
```powershell
# ad-migration-automation.ps1

# Active Directory Migration Toolkit (ADMT) Automation

param(
    [string]$SourceDomain,
    [string]$TargetDomain,
    [string]$MigrationOU,
    [string]$LogPath = "C:\ADMigration\Logs"
)

# Import required modules
Import-Module ActiveDirectory
Import-Module ADMT  # ADMT PowerShell module

class ADMigrationManager {
    [string]$SourceDomain
    [string]$TargetDomain
    [string]$LogPath
    
    ADMigrationManager([string]$source, [string]$target, [string]$logPath) {
        $this.SourceDomain = $source
        $this.TargetDomain = $target
        $this.LogPath = $logPath
    }
    
    # Pre-migration assessment
    [hashtable] AssessSourceDomain() {
        $assessment = @{
            Users = (Get-ADUser -Filter * -Server $this.SourceDomain).Count
            Groups = (Get-ADGroup -Filter * -Server $this.SourceDomain).Count
            Computers = (Get-ADComputer -Filter * -Server $this.SourceDomain).Count
            GPOs = (Get-GPO -All -Domain $this.SourceDomain).Count
            ServiceAccounts = (Get-ADServiceAccount -Filter * -Server $this.SourceDomain).Count
            Trusts = (Get-ADTrust -Filter * -Server $this.SourceDomain).Count
        }
        
        # Check for SIDHistory
        $usersWithSIDHistory = Get-ADUser -Filter * -Properties SIDHistory -Server $this.SourceDomain |
            Where-Object { $_.SIDHistory.Count -gt 0 }
        $assessment.UsersWithSIDHistory = $usersWithSIDHistory.Count
        
        return $assessment
    }
    
    # Migrate users with SID history
    [void] MigrateUsers([string]$sourceOU, [string]$targetOU) {
        Write-Host "Starting user migration from $sourceOU"
        
        $users = Get-ADUser -Filter * -SearchBase $sourceOU -Server $this.SourceDomain
        
        foreach ($user in $users) {
            try {
                # Using ADMT cmdlet
                $migrationResult = Invoke-ADMTUserMigration `
                    -SourceDomain $this.SourceDomain `
                    -TargetDomain $this.TargetDomain `
                    -SourceOU $sourceOU `
                    -TargetOU $targetOU `
                    -UserName $user.SamAccountName `
                    -MigrateSIDHistory $true `
                    -FixGroupMembership $true `
                    -TranslateRoamingProfile $true `
                    -UpdateUserRights $true
                
                Write-Host "Migrated user: $($user.SamAccountName)" -ForegroundColor Green
                $this.LogMigration("User", $user.SamAccountName, "Success")
            }
            catch {
                Write-Host "Failed to migrate: $($user.SamAccountName) - $_" -ForegroundColor Red
                $this.LogMigration("User", $user.SamAccountName, "Failed: $_")
            }
        }
    }
    
    # Migrate groups
    [void] MigrateGroups([string]$sourceOU, [string]$targetOU) {
        Write-Host "Starting group migration"
        
        $groups = Get-ADGroup -Filter * -SearchBase $sourceOU -Server $this.SourceDomain
        
        foreach ($group in $groups) {
            try {
                $migrationResult = Invoke-ADMTGroupMigration `
                    -SourceDomain $this.SourceDomain `
                    -TargetDomain $this.TargetDomain `
                    -SourceOU $sourceOU `
                    -TargetOU $targetOU `
                    -GroupName $group.SamAccountName `
                    -MigrateSIDHistory $true `
                    -FixGroupMembership $true `
                    -MigrateClosedGroups $true
                
                Write-Host "Migrated group: $($group.SamAccountName)" -ForegroundColor Green
            }
            catch {
                Write-Host "Failed to migrate group: $($group.SamAccountName) - $_" -ForegroundColor Red
            }
        }
    }
    
    # Migrate computer accounts
    [void] MigrateComputers([string[]]$computers, [string]$targetOU) {
        foreach ($computer in $computers) {
            try {
                $migrationResult = Invoke-ADMTComputerMigration `
                    -SourceDomain $this.SourceDomain `
                    -TargetDomain $this.TargetDomain `
                    -Computer $computer `
                    -TargetOU $targetOU `
                    -TranslateLocalGroups $true `
                    -TranslateFilesAndFolders $true `
                    -RestartDelay 5
                
                Write-Host "Migrated computer: $computer" -ForegroundColor Green
            }
            catch {
                Write-Host "Failed to migrate computer: $computer - $_" -ForegroundColor Red
            }
        }
    }
    
    # Security translation for file servers
    [void] TranslateSecurityOnFileServer([string]$server, [string]$sharePath) {
        Write-Host "Translating security on $server\$sharePath"
        
        Invoke-ADMTSecurityTranslation `
            -SourceDomain $this.SourceDomain `
            -TargetDomain $this.TargetDomain `
            -Computer $server `
            -TranslateFiles $true `
            -TranslateLocalGroups $true `
            -TranslateUserRights $true `
            -TranslateRegistry $true `
            -Path $sharePath
    }
    
    [void] LogMigration([string]$type, [string]$name, [string]$status) {
        $logEntry = @{
            Timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
            Type = $type
            Name = $name
            Status = $status
        }
        $logEntry | Export-Csv -Path "$($this.LogPath)\migration-log.csv" -Append -NoTypeInformation
    }
}

# Main execution
$migrator = [ADMigrationManager]::new($SourceDomain, $TargetDomain, $LogPath)

# Run assessment
Write-Host "=== Pre-Migration Assessment ===" -ForegroundColor Cyan
$assessment = $migrator.AssessSourceDomain()
$assessment | Format-Table -AutoSize

# Migration execution (example)
# $migrator.MigrateUsers("OU=Users,DC=corp,DC=local", "OU=MigratedUsers,DC=newcorp,DC=com")
# $migrator.MigrateGroups("OU=Groups,DC=corp,DC=local", "OU=MigratedGroups,DC=newcorp,DC=com")
```

---

## 4. Application Modernization <a name="app-modernization"></a>

### Modernization Patterns
```yaml
# modernization-patterns.yaml
patterns:
  
  strangler_fig:
    description: "Gradually replace legacy functionality"
    use_case: "Monolithic applications"
    approach:
      - "Identify bounded contexts"
      - "Extract functionality to microservices"
      - "Route traffic incrementally"
      - "Retire legacy components"
    example:
      before: "Monolithic .NET Framework 4.5 app"
      after: "Microservices on AKS"
      timeline: "12-18 months"
    
  lift_and_shift_then_modernize:
    description: "Migrate first, optimize later"
    use_case: "Time-critical migrations"
    approach:
      - "Rehost to Azure VMs"
      - "Establish cloud baseline"
      - "Incrementally modernize"
      - "Move to PaaS/containers"
    example:
      phase1: "VMs in Azure"
      phase2: "Containerize applications"
      phase3: "Move to AKS"
      phase4: "Add cloud-native features"
    
  database_first:
    description: "Modernize data layer first"
    use_case: "Database-intensive applications"
    approach:
      - "Migrate to Azure SQL/PostgreSQL"
      - "Implement read replicas"
      - "Add caching layer"
      - "Modernize application tier"
    benefits:
      - "Reduced maintenance"
      - "Improved performance"
      - "Built-in HA/DR"
    
  api_gateway_pattern:
    description: "Decouple clients from backends"
    use_case: "Multiple client applications"
    implementation:
      - "Deploy Azure API Management"
      - "Abstract backend services"
      - "Implement versioning"
      - "Add security policies"
```

### Containerization Workflow
```yaml
# Dockerfile generation for .NET application
# dockerfile-generator.py output

# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy project files
COPY ["WebApp/WebApp.csproj", "WebApp/"]
RUN dotnet restore "WebApp/WebApp.csproj"

# Copy source and build
COPY . .
WORKDIR "/src/WebApp"
RUN dotnet build "WebApp.csproj" -c Release -o /app/build

# Stage 2: Publish
FROM build AS publish
RUN dotnet publish "WebApp.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Stage 3: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Security: Run as non-root
RUN adduser --disabled-password --gecos '' appuser
USER appuser

# Copy published files
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# Expose port
EXPOSE 8080

# Entry point
ENTRYPOINT ["dotnet", "WebApp.dll"]
```

### Kubernetes Deployment
```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: modernized-webapp
  namespace: production
  labels:
    app: webapp
    version: v2.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        version: v2.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      serviceAccountName: webapp-sa
      securityContext:
        runAsNonRoot: true
        fsGroup: 1000
      containers:
        - name: webapp
          image: myacr.azurecr.io/webapp:2.0.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
              protocol: TCP
          env:
            - name: ASPNETCORE_ENVIRONMENT
              value: "Production"
            - name: ConnectionStrings__Database
              valueFrom:
                secretKeyRef:
                  name: webapp-secrets
                  key: db-connection-string
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 1000m
              memory: 1Gi
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          volumeMounts:
            - name: config
              mountPath: /app/config
              readOnly: true
      volumes:
        - name: config
          configMap:
            name: webapp-config
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: webapp
                topologyKey: topology.kubernetes.io/zone
```

---

## 5. VM to Container Migration <a name="vm-to-container"></a>

### VM Assessment for Containerization
```python
# vm_to_container_assessment.py
from dataclasses import dataclass
from typing import List, Dict
from enum import Enum

class ContainerizationFitness(Enum):
    EXCELLENT = "excellent"  # Easy to containerize
    GOOD = "good"           # Some modifications needed
    MODERATE = "moderate"   # Significant work required
    POOR = "poor"           # Not recommended
    NOT_SUITABLE = "not_suitable"  # Cannot containerize

@dataclass
class VMWorkload:
    name: str
    os: str
    applications: List[str]
    state_type: str  # stateless, stateful
    ports: List[int]
    storage_type: str  # local, shared, database
    startup_time_seconds: int
    dependencies: List[str]
    config_complexity: str

class ContainerizationAssessor:
    def __init__(self):
        # Applications that are container-friendly
        self.container_friendly = [
            "nginx", "apache", "tomcat", "nodejs", "python",
            "dotnet", "java", "go", "ruby", "php"
        ]
        
        # Applications that are NOT container-friendly
        self.not_containerizable = [
            "active_directory", "exchange", "sharepoint",
            "sql_server_cluster", "oracle_rac", "sap"
        ]
    
    def assess_vm(self, vm: VMWorkload) -> Dict:
        """Assess VM for containerization suitability"""
        
        score = 100
        issues = []
        recommendations = []
        
        # Check for non-containerizable apps
        for app in vm.applications:
            if any(nc in app.lower() for nc in self.not_containerizable):
                score -= 50
                issues.append(f"Application '{app}' is not suitable for containerization")
        
        # Check state
        if vm.state_type == "stateful":
            score -= 20
            issues.append("Stateful application requires persistent volume strategy")
            recommendations.append("Use StatefulSet with PVC for persistent data")
        
        # Check storage
        if vm.storage_type == "shared":
            score -= 15
            issues.append("Shared storage requires migration to Azure Files or PVC")
            recommendations.append("Consider ReadWriteMany storage class")
        
        # Check startup time
        if vm.startup_time_seconds > 60:
            score -= 10
            issues.append("Long startup time may affect scaling")
            recommendations.append("Optimize application startup or adjust HPA settings")
        
        # Check config complexity
        if vm.config_complexity == "high":
            score -= 15
            issues.append("Complex configuration requires ConfigMaps/Secrets strategy")
            recommendations.append("Externalize configuration using ConfigMaps")
        
        # Determine fitness
        if score >= 80:
            fitness = ContainerizationFitness.EXCELLENT
        elif score >= 60:
            fitness = ContainerizationFitness.GOOD
        elif score >= 40:
            fitness = ContainerizationFitness.MODERATE
        elif score >= 20:
            fitness = ContainerizationFitness.POOR
        else:
            fitness = ContainerizationFitness.NOT_SUITABLE
        
        return {
            "vm_name": vm.name,
            "fitness": fitness.value,
            "score": score,
            "issues": issues,
            "recommendations": recommendations,
            "migration_effort": self._estimate_effort(score),
            "target_architecture": self._recommend_architecture(vm, fitness)
        }
    
    def _estimate_effort(self, score: int) -> str:
        if score >= 80:
            return "1-2 weeks"
        elif score >= 60:
            return "2-4 weeks"
        elif score >= 40:
            return "4-8 weeks"
        else:
            return "8+ weeks or not recommended"
    
    def _recommend_architecture(
        self, 
        vm: VMWorkload, 
        fitness: ContainerizationFitness
    ) -> str:
        if fitness == ContainerizationFitness.NOT_SUITABLE:
            return "Rehost to Azure VM or consider alternative modernization"
        elif vm.state_type == "stateful":
            return "AKS with StatefulSet and Azure Disk PVC"
        else:
            return "AKS Deployment with HPA and Azure Container Registry"
```

---

## 6. Legacy System Migration <a name="legacy-migration"></a>

### Mainframe Migration
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Mainframe to Azure Migration                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Source: IBM z/OS Mainframe                                                 │
│  ├── COBOL applications                                                     │
│  ├── CICS transactions                                                      │
│  ├── DB2 databases                                                          │
│  └── JCL batch jobs                                                         │
│                                                                              │
│  Migration Approaches:                                                       │
│                                                                              │
│  1. REHOST (Emulation)                                                      │
│     ┌─────────────┐         ┌─────────────────────────────────┐            │
│     │ Mainframe   │  ──▶    │ Azure VMs + Emulation Software  │            │
│     │ COBOL/CICS  │         │ (Micro Focus, Raincode)         │            │
│     └─────────────┘         └─────────────────────────────────┘            │
│     Effort: Low | Risk: Low | Cost Savings: Medium                          │
│                                                                              │
│  2. REFACTOR (Automated Conversion)                                         │
│     ┌─────────────┐         ┌─────────────────────────────────┐            │
│     │ COBOL       │  ──▶    │ .NET/Java on App Service/AKS   │            │
│     │ Source      │  Tool   │ (Modern programming languages)  │            │
│     └─────────────┘         └─────────────────────────────────┘            │
│     Effort: Medium | Risk: Medium | Cost Savings: High                      │
│                                                                              │
│  3. REPLACE (Packaged Software)                                             │
│     ┌─────────────┐         ┌─────────────────────────────────┐            │
│     │ Custom      │  ──▶    │ SaaS/COTS Solution              │            │
│     │ Applications│         │ (SAP, Oracle ERP)               │            │
│     └─────────────┘         └─────────────────────────────────┘            │
│     Effort: High | Risk: High | Cost Savings: Variable                      │
│                                                                              │
│  Data Migration:                                                             │
│     DB2 ──▶ Azure SQL Database                                              │
│     VSAM ──▶ Azure Cosmos DB / Azure SQL                                    │
│     IMS DB ──▶ Azure SQL / PostgreSQL                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 7. Multi-Phase Migration Planning <a name="migration-planning"></a>

### Migration Wave Planning
```python
# migration_wave_planner.py
from dataclasses import dataclass, field
from typing import List, Dict, Set
from collections import defaultdict
import networkx as nx

@dataclass
class Application:
    id: str
    name: str
    criticality: str  # low, medium, high, critical
    complexity: str   # low, medium, high
    dependencies: List[str] = field(default_factory=list)
    data_sensitivity: str = "internal"
    downtime_tolerance: str = "hours"  # minutes, hours, days

class MigrationWavePlanner:
    def __init__(self):
        self.dependency_graph = nx.DiGraph()
        self.applications: Dict[str, Application] = {}
        
    def add_application(self, app: Application):
        """Add application and its dependencies to the graph"""
        self.applications[app.id] = app
        self.dependency_graph.add_node(app.id)
        
        for dep in app.dependencies:
            self.dependency_graph.add_edge(dep, app.id)
    
    def plan_waves(self, max_apps_per_wave: int = 10) -> List[Dict]:
        """Plan migration waves based on dependencies and criticality"""
        
        # Topological sort for dependency ordering
        try:
            sorted_apps = list(nx.topological_sort(self.dependency_graph))
        except nx.NetworkXUnfeasible:
            raise ValueError("Circular dependencies detected")
        
        waves = []
        current_wave = []
        migrated = set()
        
        for app_id in sorted_apps:
            app = self.applications.get(app_id)
            if not app:
                continue
            
            # Check if all dependencies are migrated
            deps_migrated = all(dep in migrated for dep in app.dependencies)
            
            if deps_migrated:
                # Check wave capacity and grouping rules
                if len(current_wave) < max_apps_per_wave:
                    if self._can_add_to_wave(current_wave, app):
                        current_wave.append(app)
                        migrated.add(app_id)
                    else:
                        # Start new wave
                        if current_wave:
                            waves.append(self._create_wave_plan(len(waves) + 1, current_wave))
                        current_wave = [app]
                        migrated.add(app_id)
                else:
                    waves.append(self._create_wave_plan(len(waves) + 1, current_wave))
                    current_wave = [app]
                    migrated.add(app_id)
        
        # Add remaining applications
        if current_wave:
            waves.append(self._create_wave_plan(len(waves) + 1, current_wave))
        
        return waves
    
    def _can_add_to_wave(self, wave: List[Application], app: Application) -> bool:
        """Check if application can be added to current wave"""
        
        # Don't mix critical and non-critical in same wave
        wave_criticality = {a.criticality for a in wave}
        if wave_criticality and app.criticality == "critical" and "critical" not in wave_criticality:
            return False
        
        # Don't exceed complexity threshold
        complexity_scores = {"low": 1, "medium": 2, "high": 3}
        wave_complexity = sum(complexity_scores.get(a.complexity, 1) for a in wave)
        if wave_complexity + complexity_scores.get(app.complexity, 1) > 15:
            return False
        
        return True
    
    def _create_wave_plan(self, wave_number: int, apps: List[Application]) -> Dict:
        """Create detailed wave plan"""
        
        return {
            "wave": wave_number,
            "applications": [
                {
                    "id": app.id,
                    "name": app.name,
                    "criticality": app.criticality,
                    "complexity": app.complexity,
                    "dependencies": app.dependencies
                }
                for app in apps
            ],
            "total_apps": len(apps),
            "estimated_duration_days": self._estimate_wave_duration(apps),
            "risk_level": self._assess_wave_risk(apps),
            "recommended_window": self._recommend_window(apps)
        }
    
    def _estimate_wave_duration(self, apps: List[Application]) -> int:
        """Estimate wave duration in days"""
        base_days = {
            "low": 2,
            "medium": 5,
            "high": 10
        }
        
        total_days = sum(base_days.get(app.complexity, 5) for app in apps)
        
        # Add buffer for integration testing
        total_days += len(apps) * 2
        
        return total_days
    
    def _assess_wave_risk(self, apps: List[Application]) -> str:
        """Assess overall risk level for wave"""
        criticalities = [app.criticality for app in apps]
        
        if "critical" in criticalities:
            return "high"
        elif "high" in criticalities:
            return "medium"
        else:
            return "low"
    
    def _recommend_window(self, apps: List[Application]) -> str:
        """Recommend migration window"""
        risk = self._assess_wave_risk(apps)
        
        if risk == "high":
            return "Weekend (Saturday 10 PM - Sunday 6 AM)"
        elif risk == "medium":
            return "Off-hours (Friday 10 PM - Saturday 6 AM)"
        else:
            return "Business hours with monitoring"

# Example usage
def main():
    planner = MigrationWavePlanner()
    
    # Add applications
    apps = [
        Application("app1", "Web Frontend", "low", "low", []),
        Application("app2", "API Gateway", "medium", "medium", ["app1"]),
        Application("app3", "Order Service", "high", "medium", ["app2"]),
        Application("app4", "Payment Service", "critical", "high", ["app3"]),
        Application("app5", "Reporting", "low", "low", ["app3"]),
        Application("app6", "Analytics", "medium", "medium", ["app5"]),
    ]
    
    for app in apps:
        planner.add_application(app)
    
    waves = planner.plan_waves(max_apps_per_wave=3)
    
    for wave in waves:
        print(f"\n=== Wave {wave['wave']} ===")
        print(f"Applications: {wave['total_apps']}")
        print(f"Duration: {wave['estimated_duration_days']} days")
        print(f"Risk Level: {wave['risk_level']}")
        print(f"Window: {wave['recommended_window']}")
        for app in wave['applications']:
            print(f"  - {app['name']} ({app['criticality']})")

if __name__ == "__main__":
    main()
```

---

## 8. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: Datacenter Exit Project

**Challenge**: Complete datacenter exit within 18 months for 500 servers

**STAR Response**:
```
SITUATION:
- 500 servers across 2 datacenters
- Lease expiring in 18 months
- 150 applications, mixed criticality
- $2M/year datacenter costs

TASK:
- Lead migration to Azure
- Zero critical application downtime
- Maintain compliance requirements
- Train operations team

ACTION:
1. Assessment Phase (8 weeks)
   - Deployed Azure Migrate appliances
   - Mapped 2,400 dependencies
   - Categorized all applications
   - Created migration waves

2. Foundation Phase (6 weeks)
   - Built Azure Landing Zone
   - Established ExpressRoute (10 Gbps)
   - Configured Azure AD Connect
   - Deployed monitoring stack

3. Migration Execution (12 months)
   - Wave 1: 50 non-critical servers (6 weeks)
   - Wave 2: 100 internal apps (8 weeks)
   - Wave 3: 150 business apps (10 weeks)
   - Wave 4: 150 critical systems (12 weeks)
   - Wave 5: Final migration (6 weeks)

4. Cutover Management
   - Weekend migrations for critical systems
   - Automated validation testing
   - Immediate rollback capability
   - 24/7 war room during critical migrations

RESULT:
- Completed 3 months ahead of schedule
- Zero critical incidents during migration
- 40% cost reduction (Azure Reserved Instances)
- 99.99% uptime maintained
- Datacenter lease terminated early (saved $500K)
```

### Scenario 2: AD Forest Consolidation

**Challenge**: Merge 3 AD forests after acquisition

**STAR Response**:
```
SITUATION:
- Parent company: 15,000 users in corp.local
- Acquired Company A: 5,000 users in acme.local
- Acquired Company B: 3,000 users in beta.local
- Multiple forest trusts causing complexity
- Need single identity for cloud services

TASK:
- Consolidate to single forest
- Migrate 8,000 users with SID history
- Translate all file server permissions
- Zero disruption to business operations

ACTION:
1. Trust Establishment
   - Created forest trusts between all domains
   - Verified Kerberos authentication
   - Tested cross-forest access

2. Pilot Migration
   - Migrated 100 users from each source
   - Validated SID history
   - Tested application access
   - Documented issues and solutions

3. Batch Migration
   - Migrated users in groups of 500
   - Off-hours migrations (Friday evening)
   - Automated security translation
   - Immediate validation scripts

4. Computer Migration
   - Domain rejoins via SCCM task sequence
   - Profile migration with USMT
   - Local group translation

5. Cleanup
   - Deprecated source domains
   - Removed forest trusts
   - Decommissioned old DCs

RESULT:
- Migrated 23,000 users successfully
- 48-hour RTO for any rollback
- Single identity for M365 deployment
- Reduced AD infrastructure by 60%
- Eliminated $300K/year in licensing
```

### Common Interview Questions

**Q1: "How do you handle migration risks?"**

**Answer**:
1. **Risk Assessment**: Identify risks in discovery phase
2. **Mitigation Planning**: Create rollback procedures
3. **Testing**: Extensive testing in staging
4. **Wave Approach**: Start with low-risk applications
5. **Monitoring**: Real-time monitoring during cutover
6. **Communication**: Clear escalation paths

**Q2: "What's your approach to application dependency mapping?"**

**Answer**:
1. Deploy Azure Migrate with dependency agent
2. Collect 30+ days of traffic data
3. Identify hard dependencies (synchronous)
4. Identify soft dependencies (async/batch)
5. Create dependency graphs
6. Group applications for migration waves
7. Validate with application owners

---

## Quick Reference

```bash
# Azure Migrate
az migrate project create -g rg -n project --location eastus
az migrate assessment create -g rg --project-name project --name assessment

# AD Migration (PowerShell)
Install-WindowsFeature RSAT-AD-Tools
Get-ADUser -Filter * -Properties SIDHistory | Where-Object {$_.SIDHistory}
Move-ADObject -Identity "CN=User,OU=Source,DC=corp,DC=local" -TargetPath "OU=Target,DC=newcorp,DC=com"

# Container Migration
az acr import --name myacr --source docker.io/library/nginx:latest
kubectl create deployment nginx --image=myacr.azurecr.io/nginx:latest

# Database Migration
az dms create --resource-group rg --name dms-instance --sku-name Premium_4vCores
az dms project create -g rg --service-name dms-instance --name sql-migration
```
