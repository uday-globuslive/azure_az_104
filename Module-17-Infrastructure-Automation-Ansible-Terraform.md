# Module 17: Infrastructure Automation with Ansible and Terraform

## Table of Contents
1. [Introduction to Infrastructure Automation](#introduction)
2. [Ansible Deep Dive](#ansible-deep-dive)
3. [Terraform Deep Dive](#terraform-deep-dive)
4. [Scripting with Python, Go, and Bash](#scripting)
5. [Enterprise Automation Patterns](#enterprise-patterns)
6. [Real-World Use Cases](#use-cases)
7. [Interview Questions and Scenarios](#interview-scenarios)

---

## 1. Introduction to Infrastructure Automation <a name="introduction"></a>

### What is Infrastructure Automation?
Infrastructure automation is the practice of using code and tools to provision, configure, manage, and maintain IT infrastructure without manual intervention.

### Key Principles
- **Idempotency**: Running the same automation multiple times produces the same result
- **Declarative vs Imperative**: Defining desired state vs step-by-step instructions
- **Version Control**: All infrastructure code stored in Git
- **Immutable Infrastructure**: Replace rather than modify
- **Self-Service**: Enable teams to provision their own resources

### Automation Stack Overview
```
┌─────────────────────────────────────────────────────────────┐
│                    Self-Service Portal                       │
├─────────────────────────────────────────────────────────────┤
│              API Layer (REST/GraphQL)                        │
├─────────────────────────────────────────────────────────────┤
│     Orchestration (Ansible Tower/AWX, Terraform Cloud)       │
├─────────────────────────────────────────────────────────────┤
│  Configuration Mgmt │ Infrastructure │ Container Platform    │
│     (Ansible)       │  (Terraform)   │   (Kubernetes)        │
├─────────────────────────────────────────────────────────────┤
│  On-Premises (VMware, BareMetal) │ Cloud (Azure, GCP)        │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Ansible Deep Dive <a name="ansible-deep-dive"></a>

### Ansible Architecture
```
┌──────────────────┐
│  Control Node    │
│  (Ansible)       │
├──────────────────┤
│ - Playbooks      │
│ - Inventory      │
│ - Roles          │
│ - Collections    │
└────────┬─────────┘
         │ SSH/WinRM
    ┌────┴────┐
    ▼         ▼
┌───────┐ ┌───────┐
│Target │ │Target │
│Node 1 │ │Node 2 │
└───────┘ └───────┘
```

### Key Ansible Components

#### Inventory Management
```yaml
# inventory/production/hosts.yml
all:
  children:
    webservers:
      hosts:
        web01.corp.local:
          ansible_host: 10.0.1.10
        web02.corp.local:
          ansible_host: 10.0.1.11
      vars:
        http_port: 80
        
    databases:
      hosts:
        db01.corp.local:
          ansible_host: 10.0.2.10
          
    windows_servers:
      hosts:
        win01.corp.local:
          ansible_host: 10.0.3.10
          ansible_connection: winrm
          ansible_winrm_transport: kerberos
          ansible_winrm_server_cert_validation: ignore
```

#### Dynamic Inventory for Azure
```python
# azure_rm.py - Dynamic inventory script
#!/usr/bin/env python3
import json
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient

def get_azure_inventory():
    credential = DefaultAzureCredential()
    subscription_id = "your-subscription-id"
    compute_client = ComputeManagementClient(credential, subscription_id)
    
    inventory = {
        "_meta": {"hostvars": {}},
        "azure_vms": {"hosts": []}
    }
    
    for vm in compute_client.virtual_machines.list_all():
        inventory["azure_vms"]["hosts"].append(vm.name)
        inventory["_meta"]["hostvars"][vm.name] = {
            "ansible_host": get_vm_ip(vm),
            "vm_size": vm.hardware_profile.vm_size,
            "location": vm.location
        }
    
    return inventory

if __name__ == "__main__":
    print(json.dumps(get_azure_inventory(), indent=2))
```

### Advanced Playbook Patterns

#### Windows Server Configuration
```yaml
# playbooks/windows/configure_windows_server.yml
---
- name: Configure Windows Server Infrastructure
  hosts: windows_servers
  gather_facts: yes
  
  vars:
    domain_name: "corp.local"
    dns_servers:
      - 10.0.0.10
      - 10.0.0.11
    
  tasks:
    - name: Set hostname
      win_hostname:
        name: "{{ inventory_hostname_short }}"
      register: hostname_result
      
    - name: Configure DNS servers
      win_dns_client:
        adapter_names: '*'
        dns_servers: "{{ dns_servers }}"
        
    - name: Join domain
      win_domain_membership:
        dns_domain_name: "{{ domain_name }}"
        domain_admin_user: "{{ domain_admin }}"
        domain_admin_password: "{{ domain_admin_password }}"
        state: domain
      register: domain_join
      
    - name: Reboot if required
      win_reboot:
      when: domain_join.reboot_required or hostname_result.reboot_required
      
    - name: Install Windows Features
      win_feature:
        name:
          - Web-Server
          - Web-Mgmt-Tools
          - NET-Framework-45-Core
        state: present
        
    - name: Configure Windows Firewall
      win_firewall_rule:
        name: "Allow HTTP"
        localport: 80
        action: allow
        direction: in
        protocol: tcp
        state: present
        enabled: yes
```

#### CIS Benchmark Hardening Playbook
```yaml
# playbooks/security/cis_hardening.yml
---
- name: Apply CIS Benchmark Hardening
  hosts: all
  become: yes
  
  vars:
    cis_level: 1  # Level 1 or Level 2
    
  tasks:
    # Account Policies
    - name: Set password minimum length
      win_security_policy:
        section: System Access
        key: MinimumPasswordLength
        value: 14
      when: ansible_os_family == "Windows"
      
    - name: Set password complexity
      win_security_policy:
        section: System Access
        key: PasswordComplexity
        value: 1
      when: ansible_os_family == "Windows"
      
    # Linux hardening
    - name: Ensure SSH root login is disabled
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'
      when: ansible_os_family == "RedHat"
      notify: restart sshd
      
    - name: Set permissions on /etc/passwd
      file:
        path: /etc/passwd
        owner: root
        group: root
        mode: '0644'
      when: ansible_os_family == "RedHat"
      
    - name: Disable unused filesystems
      copy:
        dest: /etc/modprobe.d/CIS.conf
        content: |
          install cramfs /bin/true
          install freevxfs /bin/true
          install jffs2 /bin/true
          install hfs /bin/true
          install hfsplus /bin/true
          install udf /bin/true
      when: ansible_os_family == "RedHat"
      
  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted
```

### Ansible Roles Structure
```
roles/
└── windows_base/
    ├── defaults/
    │   └── main.yml
    ├── files/
    │   └── scripts/
    ├── handlers/
    │   └── main.yml
    ├── meta/
    │   └── main.yml
    ├── tasks/
    │   ├── main.yml
    │   ├── prerequisites.yml
    │   ├── configuration.yml
    │   └── hardening.yml
    ├── templates/
    │   └── config.j2
    ├── tests/
    │   └── test.yml
    └── vars/
        └── main.yml
```

### Ansible Tower/AWX for Enterprise
```yaml
# Job Template Configuration
job_template:
  name: "Deploy Windows Infrastructure"
  inventory: "Production"
  project: "Infrastructure Automation"
  playbook: "playbooks/windows/deploy.yml"
  credential: "Windows Domain Admin"
  extra_vars:
    environment: production
    deploy_type: rolling
  survey_enabled: true
  survey_spec:
    - question_name: "Target Environment"
      variable: "target_env"
      type: "multiplechoice"
      choices: ["dev", "staging", "production"]
      required: true
```

---

## 3. Terraform Deep Dive <a name="terraform-deep-dive"></a>

### Terraform Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Terraform Core                            │
├─────────────────────────────────────────────────────────────┤
│  Configuration Files (.tf) │ State Management │ Providers    │
├─────────────────────────────────────────────────────────────┤
│         Provider Plugins (Azure, AWS, vSphere, etc.)         │
├─────────────────────────────────────────────────────────────┤
│              Target Infrastructure APIs                       │
└─────────────────────────────────────────────────────────────┘
```

### Enterprise Terraform Structure
```
terraform/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── production/
├── modules/
│   ├── networking/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/
│   ├── storage/
│   └── security/
└── global/
    ├── iam/
    └── dns/
```

### Azure Infrastructure Module
```hcl
# modules/azure-infrastructure/main.tf

terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "${var.project_name}-${var.environment}-rg"
  location = var.location
  
  tags = local.common_tags
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "${var.project_name}-${var.environment}-vnet"
  address_space       = var.vnet_address_space
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  
  tags = local.common_tags
}

# Subnets
resource "azurerm_subnet" "subnets" {
  for_each = var.subnets
  
  name                 = each.key
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = [each.value.address_prefix]
  
  dynamic "delegation" {
    for_each = each.value.delegation != null ? [each.value.delegation] : []
    content {
      name = delegation.value.name
      service_delegation {
        name = delegation.value.service
      }
    }
  }
}

# Network Security Groups
resource "azurerm_network_security_group" "main" {
  for_each = var.subnets
  
  name                = "${each.key}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  
  dynamic "security_rule" {
    for_each = each.value.security_rules
    content {
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = security_rule.value.direction
      access                     = security_rule.value.access
      protocol                   = security_rule.value.protocol
      source_port_range          = security_rule.value.source_port_range
      destination_port_range     = security_rule.value.destination_port_range
      source_address_prefix      = security_rule.value.source_address_prefix
      destination_address_prefix = security_rule.value.destination_address_prefix
    }
  }
  
  tags = local.common_tags
}

# Windows Virtual Machines
resource "azurerm_windows_virtual_machine" "vm" {
  for_each = var.windows_vms
  
  name                = each.key
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = each.value.size
  admin_username      = var.admin_username
  admin_password      = var.admin_password
  
  network_interface_ids = [
    azurerm_network_interface.vm_nic[each.key].id
  ]
  
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = each.value.os_disk_type
  }
  
  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = each.value.windows_sku
    version   = "latest"
  }
  
  # Domain Join Extension
  dynamic "additional_capabilities" {
    for_each = each.value.join_domain ? [1] : []
    content {
      # Enable for hybrid benefit
    }
  }
  
  tags = merge(local.common_tags, each.value.tags)
}

# VM Extension for Domain Join
resource "azurerm_virtual_machine_extension" "domain_join" {
  for_each = { for k, v in var.windows_vms : k => v if v.join_domain }
  
  name                 = "DomainJoin"
  virtual_machine_id   = azurerm_windows_virtual_machine.vm[each.key].id
  publisher            = "Microsoft.Compute"
  type                 = "JsonADDomainExtension"
  type_handler_version = "1.3"
  
  settings = jsonencode({
    Name    = var.domain_name
    User    = var.domain_join_user
    Restart = "true"
    Options = "3"
  })
  
  protected_settings = jsonencode({
    Password = var.domain_join_password
  })
}

locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
    CostCenter  = var.cost_center
  }
}
```

### Variables Definition
```hcl
# modules/azure-infrastructure/variables.tf

variable "project_name" {
  description = "Name of the project"
  type        = string
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "location" {
  description = "Azure region"
  type        = string
  default     = "eastus"
}

variable "vnet_address_space" {
  description = "VNET address space"
  type        = list(string)
  default     = ["10.0.0.0/16"]
}

variable "subnets" {
  description = "Map of subnet configurations"
  type = map(object({
    address_prefix = string
    delegation     = optional(object({
      name    = string
      service = string
    }))
    security_rules = list(object({
      name                       = string
      priority                   = number
      direction                  = string
      access                     = string
      protocol                   = string
      source_port_range          = string
      destination_port_range     = string
      source_address_prefix      = string
      destination_address_prefix = string
    }))
  }))
}

variable "windows_vms" {
  description = "Map of Windows VM configurations"
  type = map(object({
    size         = string
    windows_sku  = string
    os_disk_type = string
    join_domain  = bool
    tags         = map(string)
  }))
  default = {}
}
```

### State Management with Azure Backend
```hcl
# backend.tf
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstateaccount"
    container_name       = "tfstate"
    key                  = "production.terraform.tfstate"
    use_azuread_auth     = true
  }
}
```

### Terraform Workspaces for Multi-Environment
```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new production

# Switch workspace
terraform workspace select production

# Use workspace in configuration
locals {
  environment = terraform.workspace
  
  env_config = {
    dev = {
      vm_size = "Standard_B2s"
      vm_count = 1
    }
    staging = {
      vm_size = "Standard_D2s_v3"
      vm_count = 2
    }
    production = {
      vm_size = "Standard_D4s_v3"
      vm_count = 4
    }
  }
  
  config = local.env_config[local.environment]
}
```

---

## 4. Scripting with Python, Go, and Bash <a name="scripting"></a>

### Python Automation Scripts

#### Azure Resource Management
```python
#!/usr/bin/env python3
"""
Azure Infrastructure Automation Script
Handles VM provisioning, scaling, and lifecycle management
"""

import os
import json
import logging
from datetime import datetime
from typing import List, Dict, Optional
from azure.identity import DefaultAzureCredential
from azure.mgmt.compute import ComputeManagementClient
from azure.mgmt.network import NetworkManagementClient
from azure.mgmt.resource import ResourceManagementClient
from concurrent.futures import ThreadPoolExecutor, as_completed

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class AzureInfrastructureManager:
    def __init__(self, subscription_id: str):
        self.subscription_id = subscription_id
        self.credential = DefaultAzureCredential()
        self.compute_client = ComputeManagementClient(
            self.credential, subscription_id
        )
        self.network_client = NetworkManagementClient(
            self.credential, subscription_id
        )
        self.resource_client = ResourceManagementClient(
            self.credential, subscription_id
        )
    
    def provision_vm(
        self,
        resource_group: str,
        vm_name: str,
        location: str,
        vm_size: str,
        image_reference: Dict,
        subnet_id: str,
        admin_username: str,
        admin_password: str,
        tags: Optional[Dict] = None
    ) -> Dict:
        """Provision a new Azure VM with networking"""
        
        logger.info(f"Provisioning VM: {vm_name}")
        
        # Create NIC
        nic_params = {
            "location": location,
            "ip_configurations": [{
                "name": f"{vm_name}-ipconfig",
                "subnet": {"id": subnet_id},
                "private_ip_allocation_method": "Dynamic"
            }]
        }
        
        nic_result = self.network_client.network_interfaces.begin_create_or_update(
            resource_group,
            f"{vm_name}-nic",
            nic_params
        ).result()
        
        # Create VM
        vm_params = {
            "location": location,
            "tags": tags or {},
            "hardware_profile": {"vm_size": vm_size},
            "storage_profile": {
                "image_reference": image_reference,
                "os_disk": {
                    "name": f"{vm_name}-osdisk",
                    "caching": "ReadWrite",
                    "create_option": "FromImage",
                    "managed_disk": {"storage_account_type": "Premium_LRS"}
                }
            },
            "os_profile": {
                "computer_name": vm_name,
                "admin_username": admin_username,
                "admin_password": admin_password,
                "windows_configuration": {
                    "provision_vm_agent": True,
                    "enable_automatic_updates": True
                }
            },
            "network_profile": {
                "network_interfaces": [{"id": nic_result.id}]
            }
        }
        
        vm_result = self.compute_client.virtual_machines.begin_create_or_update(
            resource_group,
            vm_name,
            vm_params
        ).result()
        
        logger.info(f"VM {vm_name} provisioned successfully")
        return {
            "vm_id": vm_result.id,
            "private_ip": nic_result.ip_configurations[0].private_ip_address
        }
    
    def bulk_provision_vms(
        self,
        vm_configs: List[Dict],
        max_workers: int = 5
    ) -> List[Dict]:
        """Provision multiple VMs in parallel"""
        
        results = []
        with ThreadPoolExecutor(max_workers=max_workers) as executor:
            futures = {
                executor.submit(self.provision_vm, **config): config["vm_name"]
                for config in vm_configs
            }
            
            for future in as_completed(futures):
                vm_name = futures[future]
                try:
                    result = future.result()
                    results.append({"vm_name": vm_name, "status": "success", **result})
                except Exception as e:
                    logger.error(f"Failed to provision {vm_name}: {e}")
                    results.append({"vm_name": vm_name, "status": "failed", "error": str(e)})
        
        return results
    
    def get_vm_compliance_status(self, resource_group: str) -> List[Dict]:
        """Check VMs for compliance issues"""
        
        compliance_results = []
        vms = self.compute_client.virtual_machines.list(resource_group)
        
        for vm in vms:
            issues = []
            
            # Check for required tags
            required_tags = ["Environment", "CostCenter", "Owner"]
            vm_tags = vm.tags or {}
            missing_tags = [tag for tag in required_tags if tag not in vm_tags]
            if missing_tags:
                issues.append(f"Missing tags: {missing_tags}")
            
            # Check disk encryption
            os_disk = vm.storage_profile.os_disk
            if not os_disk.encryption_settings:
                issues.append("OS disk not encrypted")
            
            # Check for managed identity
            if not vm.identity:
                issues.append("No managed identity configured")
            
            compliance_results.append({
                "vm_name": vm.name,
                "compliant": len(issues) == 0,
                "issues": issues
            })
        
        return compliance_results


if __name__ == "__main__":
    manager = AzureInfrastructureManager(os.environ["AZURE_SUBSCRIPTION_ID"])
    
    # Example: Bulk provision VMs
    vm_configs = [
        {
            "resource_group": "production-rg",
            "vm_name": f"web-server-{i:02d}",
            "location": "eastus",
            "vm_size": "Standard_D2s_v3",
            "image_reference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2022-datacenter-g2",
                "version": "latest"
            },
            "subnet_id": "/subscriptions/.../subnets/web-subnet",
            "admin_username": "adminuser",
            "admin_password": os.environ["VM_ADMIN_PASSWORD"],
            "tags": {"Environment": "Production", "Role": "WebServer"}
        }
        for i in range(1, 5)
    ]
    
    results = manager.bulk_provision_vms(vm_configs)
    print(json.dumps(results, indent=2))
```

### Go Automation for High-Performance Tasks

```go
// infrastructure-automation/main.go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"os"
	"sync"
	"time"

	"github.com/Azure/azure-sdk-for-go/sdk/azidentity"
	"github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/compute/armcompute"
)

type VMHealthStatus struct {
	VMName       string    `json:"vm_name"`
	ResourceGroup string   `json:"resource_group"`
	PowerState   string    `json:"power_state"`
	Healthy      bool      `json:"healthy"`
	CheckedAt    time.Time `json:"checked_at"`
	Issues       []string  `json:"issues,omitempty"`
}

type InfrastructureMonitor struct {
	subscriptionID string
	credential     *azidentity.DefaultAzureCredential
	computeClient  *armcompute.VirtualMachinesClient
}

func NewInfrastructureMonitor(subscriptionID string) (*InfrastructureMonitor, error) {
	cred, err := azidentity.NewDefaultAzureCredential(nil)
	if err != nil {
		return nil, fmt.Errorf("failed to create credential: %w", err)
	}

	clientFactory, err := armcompute.NewClientFactory(subscriptionID, cred, nil)
	if err != nil {
		return nil, fmt.Errorf("failed to create client factory: %w", err)
	}

	return &InfrastructureMonitor{
		subscriptionID: subscriptionID,
		credential:     cred,
		computeClient:  clientFactory.NewVirtualMachinesClient(),
	}, nil
}

func (m *InfrastructureMonitor) CheckVMHealth(ctx context.Context, resourceGroup, vmName string) VMHealthStatus {
	status := VMHealthStatus{
		VMName:        vmName,
		ResourceGroup: resourceGroup,
		CheckedAt:     time.Now(),
		Healthy:       true,
	}

	// Get VM instance view
	vm, err := m.computeClient.InstanceView(ctx, resourceGroup, vmName, nil)
	if err != nil {
		status.Healthy = false
		status.Issues = append(status.Issues, fmt.Sprintf("Failed to get VM status: %v", err))
		return status
	}

	// Check power state
	for _, instanceStatus := range vm.Statuses {
		if instanceStatus.Code != nil && *instanceStatus.Code == "PowerState/running" {
			status.PowerState = "running"
		}
	}

	if status.PowerState != "running" {
		status.Healthy = false
		status.Issues = append(status.Issues, "VM is not running")
	}

	return status
}

func (m *InfrastructureMonitor) BulkHealthCheck(ctx context.Context, vms []struct {
	ResourceGroup string
	VMName        string
}) []VMHealthStatus {
	results := make([]VMHealthStatus, len(vms))
	var wg sync.WaitGroup

	// Use worker pool pattern for concurrent checks
	semaphore := make(chan struct{}, 10) // Max 10 concurrent checks

	for i, vm := range vms {
		wg.Add(1)
		go func(index int, rg, name string) {
			defer wg.Done()
			semaphore <- struct{}{}        // Acquire
			defer func() { <-semaphore }() // Release

			results[index] = m.CheckVMHealth(ctx, rg, name)
		}(i, vm.ResourceGroup, vm.VMName)
	}

	wg.Wait()
	return results
}

func main() {
	subscriptionID := os.Getenv("AZURE_SUBSCRIPTION_ID")
	if subscriptionID == "" {
		log.Fatal("AZURE_SUBSCRIPTION_ID environment variable is required")
	}

	monitor, err := NewInfrastructureMonitor(subscriptionID)
	if err != nil {
		log.Fatalf("Failed to create monitor: %v", err)
	}

	ctx := context.Background()

	// Example: Check multiple VMs
	vms := []struct {
		ResourceGroup string
		VMName        string
	}{
		{"production-rg", "web-server-01"},
		{"production-rg", "web-server-02"},
		{"production-rg", "db-server-01"},
	}

	results := monitor.BulkHealthCheck(ctx, vms)

	output, _ := json.MarshalIndent(results, "", "  ")
	fmt.Println(string(output))
}
```

### Bash Automation Scripts

```bash
#!/bin/bash
# infrastructure-maintenance.sh
# Comprehensive infrastructure maintenance automation

set -euo pipefail

# Configuration
LOG_FILE="/var/log/infra-maintenance.log"
ALERT_WEBHOOK="${ALERT_WEBHOOK:-}"
PARALLEL_JOBS=5

# Logging function
log() {
    local level="$1"
    shift
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $*" | tee -a "$LOG_FILE"
}

# Send alert to monitoring system
send_alert() {
    local severity="$1"
    local message="$2"
    
    if [[ -n "$ALERT_WEBHOOK" ]]; then
        curl -s -X POST "$ALERT_WEBHOOK" \
            -H "Content-Type: application/json" \
            -d "{\"severity\": \"$severity\", \"message\": \"$message\"}"
    fi
}

# Check system health
check_system_health() {
    local hostname="$1"
    local issues=()
    
    # Check disk usage
    local disk_usage
    disk_usage=$(ssh -o ConnectTimeout=10 "$hostname" "df -h / | awk 'NR==2 {print \$5}' | tr -d '%'" 2>/dev/null) || {
        log "ERROR" "Failed to connect to $hostname"
        return 1
    }
    
    if [[ "$disk_usage" -gt 85 ]]; then
        issues+=("Disk usage at ${disk_usage}%")
    fi
    
    # Check memory usage
    local mem_usage
    mem_usage=$(ssh "$hostname" "free | awk '/Mem:/ {printf \"%.0f\", \$3/\$2 * 100}'")
    
    if [[ "$mem_usage" -gt 90 ]]; then
        issues+=("Memory usage at ${mem_usage}%")
    fi
    
    # Check for pending updates (RHEL/CentOS)
    local pending_updates
    pending_updates=$(ssh "$hostname" "yum check-update 2>/dev/null | grep -c '^[a-zA-Z]'" || echo "0")
    
    if [[ "$pending_updates" -gt 0 ]]; then
        issues+=("$pending_updates pending security updates")
    fi
    
    # Return results
    if [[ ${#issues[@]} -gt 0 ]]; then
        log "WARN" "$hostname has issues: ${issues[*]}"
        echo "${issues[*]}"
        return 1
    fi
    
    log "INFO" "$hostname is healthy"
    return 0
}

# Parallel health check
parallel_health_check() {
    local hosts_file="$1"
    local results_file="/tmp/health-check-results-$$.txt"
    
    log "INFO" "Starting parallel health check"
    
    cat "$hosts_file" | xargs -P "$PARALLEL_JOBS" -I {} bash -c '
        result=$(check_system_health "{}" 2>&1)
        echo "{}|$?|$result"
    ' >> "$results_file"
    
    # Process results
    local failed_hosts=()
    while IFS='|' read -r host status issues; do
        if [[ "$status" -ne 0 ]]; then
            failed_hosts+=("$host: $issues")
        fi
    done < "$results_file"
    
    if [[ ${#failed_hosts[@]} -gt 0 ]]; then
        send_alert "warning" "Health check failed for: ${failed_hosts[*]}"
    fi
    
    rm -f "$results_file"
}

# Certificate expiry check
check_certificates() {
    local cert_path="$1"
    local warning_days="${2:-30}"
    
    log "INFO" "Checking certificate expiry in $cert_path"
    
    find "$cert_path" -name "*.crt" -o -name "*.pem" | while read -r cert; do
        local expiry_date
        expiry_date=$(openssl x509 -enddate -noout -in "$cert" 2>/dev/null | cut -d= -f2)
        
        if [[ -n "$expiry_date" ]]; then
            local expiry_epoch
            expiry_epoch=$(date -d "$expiry_date" +%s)
            local current_epoch
            current_epoch=$(date +%s)
            local days_until_expiry=$(( (expiry_epoch - current_epoch) / 86400 ))
            
            if [[ "$days_until_expiry" -lt "$warning_days" ]]; then
                log "WARN" "Certificate $cert expires in $days_until_expiry days"
                send_alert "warning" "Certificate expiring: $cert ($days_until_expiry days)"
            fi
        fi
    done
}

# Configuration drift detection
check_config_drift() {
    local baseline_dir="$1"
    local current_dir="$2"
    
    log "INFO" "Checking for configuration drift"
    
    local drift_detected=false
    
    diff -rq "$baseline_dir" "$current_dir" 2>/dev/null | while read -r line; do
        log "WARN" "Config drift: $line"
        drift_detected=true
    done
    
    if [[ "$drift_detected" == "true" ]]; then
        send_alert "warning" "Configuration drift detected"
        return 1
    fi
    
    log "INFO" "No configuration drift detected"
    return 0
}

# Main execution
main() {
    local action="${1:-}"
    
    case "$action" in
        health-check)
            parallel_health_check "${2:-/etc/hosts.monitored}"
            ;;
        cert-check)
            check_certificates "${2:-/etc/ssl/certs}" "${3:-30}"
            ;;
        drift-check)
            check_config_drift "${2:-/etc/baseline}" "${3:-/etc/current}"
            ;;
        full)
            parallel_health_check "/etc/hosts.monitored"
            check_certificates "/etc/ssl/certs"
            check_config_drift "/etc/baseline" "/etc/current"
            ;;
        *)
            echo "Usage: $0 {health-check|cert-check|drift-check|full}"
            exit 1
            ;;
    esac
}

# Export functions for parallel execution
export -f check_system_health log send_alert

main "$@"
```

---

## 5. Enterprise Automation Patterns <a name="enterprise-patterns"></a>

### Self-Service Infrastructure Portal Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    Self-Service Portal (React/Vue)              │
├─────────────────────────────────────────────────────────────────┤
│                    API Gateway (Kong/Azure APIM)                │
├─────────────────────────────────────────────────────────────────┤
│  Orchestration Service │ Approval Workflow │ RBAC Service       │
│     (FastAPI/Go)       │   (ServiceNow)    │  (Keycloak)        │
├─────────────────────────────────────────────────────────────────┤
│           Message Queue (RabbitMQ/Azure Service Bus)            │
├─────────────────────────────────────────────────────────────────┤
│  Terraform Worker  │  Ansible Worker  │  Kubernetes Worker      │
│   (State Mgmt)     │   (Config Mgmt)  │   (Container Ops)       │
├─────────────────────────────────────────────────────────────────┤
│  On-Premises       │     Azure        │     Kubernetes          │
│  (VMware/Hyper-V)  │  (ARM/Bicep)     │  (OpenShift/AKS)        │
└─────────────────────────────────────────────────────────────────┘
```

### API-Driven Provisioning Workflow
```python
# api/services/provisioning_service.py
from fastapi import FastAPI, HTTPException, BackgroundTasks
from pydantic import BaseModel
from typing import Optional, List
import asyncio
import uuid

app = FastAPI()

class ProvisioningRequest(BaseModel):
    request_id: str = None
    environment: str
    resource_type: str
    specifications: dict
    requestor: str
    cost_center: str
    approval_required: bool = True

class ProvisioningService:
    def __init__(self):
        self.terraform_client = TerraformClient()
        self.ansible_client = AnsibleClient()
        self.approval_service = ApprovalService()
        self.notification_service = NotificationService()
    
    async def process_request(self, request: ProvisioningRequest) -> dict:
        """Process infrastructure provisioning request"""
        
        request.request_id = str(uuid.uuid4())
        
        # Step 1: Validate request
        validation_result = await self.validate_request(request)
        if not validation_result["valid"]:
            raise HTTPException(400, validation_result["errors"])
        
        # Step 2: Check approval requirements
        if request.approval_required:
            approval = await self.approval_service.request_approval(request)
            if not approval["approved"]:
                return {"status": "pending_approval", "request_id": request.request_id}
        
        # Step 3: Execute provisioning
        try:
            # Run Terraform for infrastructure
            tf_result = await self.terraform_client.apply(
                workspace=request.environment,
                variables=request.specifications
            )
            
            # Run Ansible for configuration
            if request.resource_type in ["vm", "server"]:
                ansible_result = await self.ansible_client.run_playbook(
                    playbook="configure_server.yml",
                    inventory=tf_result["inventory"],
                    extra_vars=request.specifications
                )
            
            # Notify completion
            await self.notification_service.notify(
                request.requestor,
                f"Provisioning complete: {request.request_id}"
            )
            
            return {
                "status": "completed",
                "request_id": request.request_id,
                "resources": tf_result["resources"]
            }
            
        except Exception as e:
            await self.notification_service.notify(
                request.requestor,
                f"Provisioning failed: {request.request_id} - {str(e)}"
            )
            raise HTTPException(500, str(e))

@app.post("/api/v1/provision")
async def provision_infrastructure(
    request: ProvisioningRequest,
    background_tasks: BackgroundTasks
):
    service = ProvisioningService()
    
    # Run in background for long operations
    background_tasks.add_task(service.process_request, request)
    
    return {
        "message": "Provisioning request submitted",
        "request_id": request.request_id,
        "status_url": f"/api/v1/provision/{request.request_id}/status"
    }
```

### Reusable Automation Modules

#### Terraform Module Registry Pattern
```hcl
# modules/vm-with-monitoring/main.tf
# Reusable VM module with built-in monitoring

variable "vm_config" {
  type = object({
    name           = string
    size           = string
    os_type        = string
    subnet_id      = string
    enable_backup  = bool
    monitoring     = object({
      enabled        = bool
      log_analytics_workspace_id = string
      alert_rules    = list(object({
        name       = string
        metric     = string
        threshold  = number
        severity   = number
      }))
    })
  })
}

resource "azurerm_windows_virtual_machine" "vm" {
  count = var.vm_config.os_type == "windows" ? 1 : 0
  
  name                = var.vm_config.name
  # ... VM configuration
}

resource "azurerm_virtual_machine_extension" "monitoring_agent" {
  count = var.vm_config.monitoring.enabled ? 1 : 0
  
  name                 = "AzureMonitorWindowsAgent"
  virtual_machine_id   = azurerm_windows_virtual_machine.vm[0].id
  publisher            = "Microsoft.Azure.Monitor"
  type                 = "AzureMonitorWindowsAgent"
  type_handler_version = "1.0"
  
  settings = jsonencode({
    workspaceId = var.vm_config.monitoring.log_analytics_workspace_id
  })
}

resource "azurerm_monitor_metric_alert" "alerts" {
  for_each = { for rule in var.vm_config.monitoring.alert_rules : rule.name => rule }
  
  name                = each.value.name
  resource_group_name = var.resource_group_name
  scopes              = [azurerm_windows_virtual_machine.vm[0].id]
  severity            = each.value.severity
  
  criteria {
    metric_namespace = "Microsoft.Compute/virtualMachines"
    metric_name      = each.value.metric
    aggregation      = "Average"
    operator         = "GreaterThan"
    threshold        = each.value.threshold
  }
}
```

---

## 6. Real-World Use Cases <a name="use-cases"></a>

### Use Case 1: Data Center to Cloud Migration Automation

**Scenario**: Migrate 500+ Windows servers from on-premises VMware to Azure

```yaml
# migration-playbook.yml
---
- name: Datacenter to Azure Migration
  hosts: localhost
  vars:
    migration_waves:
      - name: "Wave 1 - Non-Production"
        servers: "{{ groups['nonprod_servers'] }}"
        schedule: "2024-Q1"
      - name: "Wave 2 - Production Tier 2"
        servers: "{{ groups['prod_tier2'] }}"
        schedule: "2024-Q2"
  
  tasks:
    - name: Pre-migration assessment
      include_role:
        name: azure_migrate_assessment
      vars:
        discovery_scope: "{{ item.servers }}"
      loop: "{{ migration_waves }}"
      
    - name: Generate migration plan
      template:
        src: migration_plan.j2
        dest: "/reports/migration_plan_{{ item.name }}.md"
      loop: "{{ migration_waves }}"
      
    - name: Execute migration
      include_role:
        name: azure_migrate_execute
      vars:
        source_servers: "{{ item.servers }}"
        target_resource_group: "migrated-{{ item.name | lower | replace(' ', '-') }}"
        replication_policy: "crash-consistent"
        test_failover: true
      loop: "{{ migration_waves }}"
      when: migration_approved | default(false)
```

### Use Case 2: Automated Disaster Recovery Setup

**Scenario**: Implement automated DR for critical Windows infrastructure

```python
# disaster_recovery_automation.py
"""
Automated Disaster Recovery Setup and Testing
"""

import asyncio
from dataclasses import dataclass
from typing import List
from azure.mgmt.recoveryservices import RecoveryServicesClient
from azure.mgmt.recoveryservicesbackup import RecoveryServicesBackupClient

@dataclass
class DRConfiguration:
    primary_region: str
    dr_region: str
    recovery_vault_name: str
    replication_policy: dict
    critical_vms: List[str]
    rpo_minutes: int
    rto_minutes: int

class DisasterRecoveryManager:
    def __init__(self, subscription_id: str, dr_config: DRConfiguration):
        self.dr_config = dr_config
        # Initialize Azure clients
        
    async def setup_replication(self):
        """Configure ASR replication for all critical VMs"""
        
        tasks = []
        for vm_name in self.dr_config.critical_vms:
            tasks.append(self.enable_vm_replication(vm_name))
        
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        return {
            "total_vms": len(self.dr_config.critical_vms),
            "successful": sum(1 for r in results if not isinstance(r, Exception)),
            "failed": [str(r) for r in results if isinstance(r, Exception)]
        }
    
    async def execute_dr_drill(self):
        """Execute automated DR drill with validation"""
        
        print("Starting DR Drill...")
        
        # 1. Create isolated test network in DR region
        test_network = await self.create_test_network()
        
        # 2. Perform test failover
        failover_results = []
        for vm in self.dr_config.critical_vms:
            result = await self.test_failover(vm, test_network)
            failover_results.append(result)
        
        # 3. Validate services
        validation_results = await self.validate_dr_environment(failover_results)
        
        # 4. Cleanup test environment
        await self.cleanup_test_failover()
        
        # 5. Generate report
        return self.generate_dr_report(failover_results, validation_results)
    
    async def validate_dr_environment(self, failover_results):
        """Validate that DR environment is functional"""
        
        validations = {
            "network_connectivity": await self.check_network_connectivity(),
            "service_health": await self.check_service_health(),
            "data_integrity": await self.verify_data_integrity(),
            "rto_compliance": await self.measure_rto()
        }
        
        return validations
```

### Use Case 3: Compliance-Driven Infrastructure Automation

**Scenario**: Automated CIS benchmark enforcement and drift remediation

```yaml
# compliance-automation.yml
---
- name: CIS Compliance Automation
  hosts: all
  become: yes
  
  vars:
    cis_benchmark_version: "2.0.0"
    remediation_mode: "{{ lookup('env', 'REMEDIATION_MODE') | default('report') }}"
    
  pre_tasks:
    - name: Gather baseline compliance status
      include_role:
        name: compliance_scanner
      vars:
        output_format: json
        output_path: "/tmp/pre_scan_{{ inventory_hostname }}.json"
        
  tasks:
    # Section 1: Account Policies
    - name: "1.1.1 Ensure password expiration is 365 days or less"
      win_security_policy:
        section: System Access
        key: MaximumPasswordAge
        value: 60
      when:
        - ansible_os_family == "Windows"
        - remediation_mode == "enforce"
      tags: [cis, passwords, level1]
      
    - name: "1.1.2 Ensure minimum password length is 14 or more"
      win_security_policy:
        section: System Access
        key: MinimumPasswordLength
        value: 14
      when:
        - ansible_os_family == "Windows"
        - remediation_mode == "enforce"
      tags: [cis, passwords, level1]
      
    # Section 2: Local Policies
    - name: "2.3.1.1 Ensure 'Accounts: Administrator account status' is set to 'Disabled'"
      win_user:
        name: Administrator
        account_disabled: yes
      when:
        - remediation_mode == "enforce"
        - custom_admin_account_exists | default(true)
      tags: [cis, accounts, level1]
      
    # Section 5: Windows Firewall
    - name: "5.1.1 Ensure Windows Firewall: Domain: Firewall state is On"
      win_firewall:
        state: enabled
        profiles:
          - Domain
          - Private
          - Public
      when: remediation_mode == "enforce"
      tags: [cis, firewall, level1]
      
  post_tasks:
    - name: Gather post-remediation compliance status
      include_role:
        name: compliance_scanner
      vars:
        output_format: json
        output_path: "/tmp/post_scan_{{ inventory_hostname }}.json"
        
    - name: Generate compliance delta report
      template:
        src: compliance_report.j2
        dest: "/reports/compliance_{{ inventory_hostname }}_{{ ansible_date_time.date }}.html"
      delegate_to: localhost
```

---

## 7. Interview Questions and Scenarios <a name="interview-scenarios"></a>

### Behavioral Questions with STAR Format Answers

#### Q1: "Tell me about the most challenging automation project you've worked on"

**Situation**: At my previous organization, we had 2,000+ Windows servers across 5 data centers with inconsistent configurations, causing frequent security incidents and audit failures.

**Task**: I was tasked with implementing enterprise-wide configuration standardization and automated compliance enforcement within 6 months.

**Action**:
1. **Assessment Phase** (Month 1):
   - Deployed Ansible with dynamic inventory to discover all servers
   - Created baseline configuration audit playbooks
   - Identified 47 different configuration patterns across the estate

2. **Design Phase** (Month 2):
   - Designed modular Ansible roles for each server type (web, app, database)
   - Implemented CIS benchmark Level 1 as baseline
   - Created drift detection automation using scheduled scans

3. **Implementation Phase** (Months 3-5):
   - Rolled out in waves: Dev → QA → Staging → Production
   - Built self-service portal for teams to request configuration changes
   - Integrated with ServiceNow for change management

4. **Optimization Phase** (Month 6):
   - Implemented real-time drift detection with immediate remediation
   - Created dashboards showing compliance status across all environments

**Result**:
- Reduced configuration drift incidents by 94%
- Achieved 99.2% CIS compliance across all servers
- Reduced audit preparation time from 3 weeks to 2 days
- Automated 85% of routine server configuration tasks

#### Q2: "Describe your most innovative solution to an infrastructure problem"

**Situation**: Our organization was spending $2M annually on manual certificate management, with frequent outages due to expired certificates across 15,000+ endpoints.

**Task**: Design and implement an automated certificate lifecycle management system.

**Action**:
1. **Discovery and Analysis**:
   - Built Python scanner to inventory all certificates across infrastructure
   - Discovered 23,000 certificates with varying renewal processes

2. **Architecture Design**:
   - Designed microservices-based certificate management platform
   - Integrated with HashiCorp Vault for certificate issuance
   - Created Kubernetes operators for automatic renewal

3. **Implementation**:
   ```python
   # Certificate lifecycle automation
   class CertificateLifecycleManager:
       def __init__(self):
           self.vault_client = VaultClient()
           self.notification_service = NotificationService()
           
       async def monitor_and_renew(self):
           certificates = await self.scan_all_certificates()
           
           for cert in certificates:
               days_to_expiry = (cert.expiry - datetime.now()).days
               
               if days_to_expiry <= 30:
                   # Auto-renew
                   new_cert = await self.vault_client.issue_certificate(
                       common_name=cert.common_name,
                       san=cert.san
                   )
                   await self.deploy_certificate(cert.endpoint, new_cert)
                   
               elif days_to_expiry <= 60:
                   # Notify team
                   await self.notification_service.alert(
                       f"Certificate expiring: {cert.common_name}"
                   )
   ```

4. **Integration**:
   - Ansible playbooks for certificate deployment to Windows/Linux servers
   - Terraform modules for cloud certificate management
   - GitOps workflow for certificate policies

**Result**:
- Zero certificate-related outages in 18 months (previously 12/year)
- Reduced certificate management effort by 95%
- Saved $1.8M annually in operational costs
- Patent filed for the auto-discovery algorithm

#### Q3: "What was your biggest achievement in platform reliability?"

**Situation**: Our production Kubernetes clusters were experiencing frequent incidents with MTTR averaging 45 minutes, impacting customer SLAs.

**Task**: Reduce MTTR to under 10 minutes and improve overall platform reliability.

**Action**:
1. **Observability Enhancement**:
   - Implemented unified observability stack (Prometheus, Grafana, Jaeger)
   - Created SLO-based alerting with error budgets
   - Built automated runbooks for common issues

2. **Self-Healing Automation**:
   ```yaml
   # Kubernetes self-healing operator
   apiVersion: chaos.io/v1alpha1
   kind: SelfHealingPolicy
   metadata:
     name: pod-recovery
   spec:
     triggers:
       - type: PodCrashLoopBackOff
         threshold: 3
         window: 5m
       - type: HighMemoryUsage
         threshold: 95
         duration: 2m
     actions:
       - type: RestartPod
         maxRetries: 2
       - type: ScaleUp
         increment: 1
         maxReplicas: 10
       - type: NotifyOnCall
         channel: slack
         escalation: pagerduty
   ```

3. **Automated Incident Response**:
   - Created incident response automation with PagerDuty integration
   - Built ChatOps bot for quick diagnostics
   - Implemented automated rollback on deployment failures

4. **Chaos Engineering**:
   - Introduced chaos testing in staging environments
   - Identified and fixed 34 potential failure scenarios
   - Created resilience testing as part of CI/CD pipeline

**Result**:
- MTTR reduced from 45 minutes to 7 minutes
- Platform availability improved from 99.5% to 99.95%
- Reduced on-call incidents by 60%
- Team velocity increased due to less firefighting

### Technical Deep-Dive Questions

#### Q4: "How would you design automation for a hybrid cloud environment?"

**Answer Framework**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Unified Control Plane                         │
│              (Terraform Cloud / Ansible Tower)                   │
├─────────────────────────────────────────────────────────────────┤
│                    GitOps Repository                             │
│         (Infrastructure definitions in Git)                      │
├───────────────────┬─────────────────────┬───────────────────────┤
│    On-Premises    │       Azure         │     Kubernetes        │
│    ┌──────────┐   │   ┌──────────┐      │   ┌──────────┐        │
│    │ VMware   │   │   │ Azure RM │      │   │ OpenShift│        │
│    │ Provider │   │   │ Provider │      │   │ Provider │        │
│    └──────────┘   │   └──────────┘      │   └──────────┘        │
├───────────────────┴─────────────────────┴───────────────────────┤
│                    Shared Services Layer                         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                │
│  │  Vault  │ │ Consul  │ │Monitoring│ │  CMDB   │                │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘                │
└─────────────────────────────────────────────────────────────────┘
```

**Key Design Principles**:
1. Single source of truth in Git
2. Provider abstraction for multi-cloud
3. Shared state management
4. Consistent RBAC across environments
5. Unified monitoring and alerting

#### Q5: "Explain how you would implement zero-downtime deployments"

**Answer with Implementation**:

```yaml
# Zero-downtime deployment strategy
deployment_strategies:
  rolling_update:
    description: "Gradual replacement of instances"
    use_case: "Stateless applications"
    terraform_example: |
      resource "azurerm_linux_virtual_machine_scale_set" "app" {
        upgrade_mode = "Rolling"
        
        rolling_upgrade_policy {
          max_batch_instance_percent              = 20
          max_unhealthy_instance_percent          = 20
          max_unhealthy_upgraded_instance_percent = 5
          pause_time_between_batches              = "PT2M"
        }
        
        health_probe_id = azurerm_lb_probe.health.id
      }
      
  blue_green:
    description: "Parallel environments with traffic switch"
    use_case: "Critical applications requiring instant rollback"
    implementation:
      - Deploy new version to green environment
      - Run smoke tests on green
      - Switch load balancer to green
      - Keep blue for instant rollback
      - Decommission blue after validation period
      
  canary:
    description: "Gradual traffic shift with monitoring"
    use_case: "High-traffic applications"
    traffic_progression:
      - { percentage: 1, duration: "10m", gates: ["error_rate < 1%"] }
      - { percentage: 10, duration: "30m", gates: ["latency_p99 < 500ms"] }
      - { percentage: 50, duration: "1h", gates: ["slo_compliance > 99%"] }
      - { percentage: 100, duration: "final" }
```

### Scenario-Based Questions

#### Q6: "You discover that 30% of your servers have drifted from baseline. What's your approach?"

**Structured Response**:

1. **Immediate Assessment** (Day 1):
   - Run full compliance scan to identify exact drift
   - Categorize drift by severity (security, performance, functional)
   - Identify root causes (manual changes, failed automation, etc.)

2. **Risk Prioritization** (Day 2-3):
   - Security-related drift: Immediate remediation
   - Performance drift: Schedule within 1 week
   - Cosmetic drift: Include in next maintenance window

3. **Remediation Approach**:
   ```bash
   # Drift remediation workflow
   ansible-playbook remediate_drift.yml \
     --limit "drifted_servers" \
     --extra-vars "remediation_mode=enforce" \
     --extra-vars "create_backup=true" \
     --check  # Dry run first
   ```

4. **Prevention Measures**:
   - Implement real-time drift detection
   - Block direct server access (require automation)
   - Add drift check to CI/CD pipeline

5. **Reporting and Metrics**:
   - Track drift metrics over time
   - Report on root causes
   - Measure time-to-remediation

---

## Key Takeaways for Interview Success

1. **Always use STAR format** for behavioral questions
2. **Quantify results** wherever possible (%, time saved, cost reduced)
3. **Show end-to-end thinking** from problem to solution to maintenance
4. **Demonstrate depth** in Ansible, Terraform, and scripting
5. **Connect automation to business value** (compliance, reliability, cost)
6. **Highlight self-service and API-driven approaches**
7. **Emphasize observability and continuous improvement**

---

## Quick Reference Commands

```bash
# Ansible
ansible-playbook -i inventory/ playbook.yml --check
ansible-vault encrypt secrets.yml
ansible-galaxy collection install azure.azcollection

# Terraform
terraform init -backend-config=backend.tfvars
terraform plan -var-file=production.tfvars
terraform apply -auto-approve
terraform state list

# Azure CLI
az vm list --query "[].{name:name,rg:resourceGroup}" -o table
az group deployment create --template-file main.bicep

# Compliance
ansible-playbook cis_audit.yml --tags "level1,scored"
```
