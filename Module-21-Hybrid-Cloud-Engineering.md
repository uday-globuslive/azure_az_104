# Module 21: Hybrid Cloud Engineering (On-Premises, Azure, Multi-Cloud)

## Table of Contents
1. [Hybrid Cloud Architecture](#hybrid-architecture)
2. [On-Premises Infrastructure](#on-prem)
3. [Azure Hybrid Connectivity](#azure-connectivity)
4. [Bare Metal and VMware Integration](#baremetal-vmware)
5. [Multi-Cloud Strategies](#multi-cloud)
6. [Azure Arc for Hybrid Management](#azure-arc)
7. [Storage and Data Strategies](#storage)
8. [Interview Scenarios](#interview-scenarios)

---

## 1. Hybrid Cloud Architecture <a name="hybrid-architecture"></a>

### Enterprise Hybrid Architecture
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Unified Management Layer                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Azure Arc  │  │  Terraform  │  │   Ansible   │  │ Kubernetes  │        │
│  │             │  │   Cloud     │  │   Tower     │  │   (ACM)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Connectivity Layer                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐              │
│  │   ExpressRoute  │  │   Site-to-Site  │  │   Azure VPN     │              │
│  │   (Private)     │  │   VPN           │  │   Point-to-Site │              │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘              │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Compute Platforms                                   │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐  ┌──────────────┐  │
│  │  On-Premises  │  │    Azure      │  │  Azure Stack  │  │   Edge       │  │
│  │  Data Center  │  │    Cloud      │  │     HCI       │  │   Locations  │  │
│  │ ─────────────│  │ ─────────────│  │ ─────────────│  │ ────────────│  │
│  │ • VMware     │  │ • Azure VMs   │  │ • Local Azure │  │ • IoT Hubs   │  │
│  │ • Hyper-V    │  │ • AKS         │  │ • K8s Hybrid  │  │ • Azure Arc  │  │
│  │ • Bare Metal │  │ • App Service │  │ • Storage     │  │ • Edge K8s   │  │
│  │ • OpenShift  │  │ • Functions   │  │ • VMs         │  │              │  │
│  └───────────────┘  └───────────────┘  └───────────────┘  └──────────────┘  │
├─────────────────────────────────────────────────────────────────────────────┤
│                          Shared Services                                     │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │
│  │Identity │ │   DNS   │ │Monitoring│ │ Backup  │ │Security │ │ Network │   │
│  │(Entra)  │ │(Hybrid) │ │(Azure   │ │(Hybrid) │ │(Defender│ │ (Hybrid)│   │
│  │         │ │         │ │ Monitor)│ │         │ │ Cloud)  │ │         │   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Connectivity Patterns
```
Pattern 1: Hub-Spoke with On-Premises
═══════════════════════════════════════════════════════════════════

                    ┌─────────────────┐
                    │   On-Premises   │
                    │   Data Center   │
                    └────────┬────────┘
                             │
                    ExpressRoute / VPN
                             │
                    ┌────────┴────────┐
                    │    Hub VNet     │
                    │  ┌───────────┐  │
                    │  │ Firewall  │  │
                    │  │ VPN GW    │  │
                    │  │ Bastion   │  │
                    │  └───────────┘  │
                    └────────┬────────┘
           ┌─────────────────┼─────────────────┐
           │                 │                 │
    ┌──────┴──────┐   ┌──────┴──────┐   ┌──────┴──────┐
    │ Spoke VNet  │   │ Spoke VNet  │   │ Spoke VNet  │
    │ (Prod)      │   │ (Dev)       │   │ (DMZ)       │
    └─────────────┘   └─────────────┘   └─────────────┘


Pattern 2: Multi-Region Active-Active
═══════════════════════════════════════════════════════════════════

    ┌─────────────────┐              ┌─────────────────┐
    │ On-Premises     │              │ On-Premises     │
    │ (Primary)       │              │ (DR Site)       │
    └────────┬────────┘              └────────┬────────┘
             │                                │
    ExpressRoute                     ExpressRoute
             │                                │
    ┌────────┴────────┐              ┌────────┴────────┐
    │   Azure East    │◄────────────►│   Azure West    │
    │   Region        │  Global VNet │   Region        │
    │                 │   Peering    │                 │
    └─────────────────┘              └─────────────────┘
```

---

## 2. On-Premises Infrastructure <a name="on-prem"></a>

### VMware Integration
```powershell
# PowerCLI for VMware Automation
# Connect to vCenter
Connect-VIServer -Server vcenter.corp.local -Credential $Credentials

# Create VM from Template
$VMParams = @{
    Name = "web-server-01"
    Template = "Windows2022-Template"
    VMHost = "esxi-host-01.corp.local"
    Datastore = "Production-SAN"
    Location = "Production"
    NetworkName = "VLAN-100-Production"
}

$NewVM = New-VM @VMParams

# Configure VM resources
Set-VM -VM $NewVM -NumCpu 4 -MemoryGB 16 -Confirm:$false

# Add additional disk
New-HardDisk -VM $NewVM -CapacityGB 100 -StorageFormat Thick -Datastore "Production-SAN"

# Configure network
Get-NetworkAdapter -VM $NewVM | Set-NetworkAdapter -NetworkName "VLAN-100-Production" -Confirm:$false

# Start VM
Start-VM -VM $NewVM

# Apply customization specification
$Spec = Get-OSCustomizationSpec -Name "Windows-Domain-Join"
Set-VM -VM $NewVM -OSCustomizationSpec $Spec -Confirm:$false
```

### Terraform VMware Provider
```hcl
# VMware vSphere Provider Configuration
terraform {
  required_providers {
    vsphere = {
      source  = "hashicorp/vsphere"
      version = "~> 2.0"
    }
  }
}

provider "vsphere" {
  user                 = var.vsphere_user
  password             = var.vsphere_password
  vsphere_server       = var.vsphere_server
  allow_unverified_ssl = true
}

# Data sources
data "vsphere_datacenter" "dc" {
  name = "Datacenter"
}

data "vsphere_compute_cluster" "cluster" {
  name          = "Production-Cluster"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_datastore" "datastore" {
  name          = "Production-SAN"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_network" "network" {
  name          = "VLAN-100-Production"
  datacenter_id = data.vsphere_datacenter.dc.id
}

data "vsphere_virtual_machine" "template" {
  name          = "Windows2022-Template"
  datacenter_id = data.vsphere_datacenter.dc.id
}

# Create VM
resource "vsphere_virtual_machine" "vm" {
  count            = var.vm_count
  name             = "web-server-${format("%02d", count.index + 1)}"
  resource_pool_id = data.vsphere_compute_cluster.cluster.resource_pool_id
  datastore_id     = data.vsphere_datastore.datastore.id
  folder           = "Production/Web-Servers"

  num_cpus = 4
  memory   = 16384
  guest_id = data.vsphere_virtual_machine.template.guest_id

  network_interface {
    network_id   = data.vsphere_network.network.id
    adapter_type = data.vsphere_virtual_machine.template.network_interface_types[0]
  }

  disk {
    label            = "disk0"
    size             = data.vsphere_virtual_machine.template.disks.0.size
    eagerly_scrub    = false
    thin_provisioned = true
  }

  disk {
    label            = "disk1"
    size             = 100
    unit_number      = 1
    thin_provisioned = true
  }

  clone {
    template_uuid = data.vsphere_virtual_machine.template.id

    customize {
      windows_options {
        computer_name    = "web-server-${format("%02d", count.index + 1)}"
        admin_password   = var.admin_password
        join_domain      = "corp.local"
        domain_admin_user     = var.domain_admin
        domain_admin_password = var.domain_password
      }

      network_interface {
        ipv4_address = "10.0.100.${count.index + 10}"
        ipv4_netmask = 24
      }

      ipv4_gateway    = "10.0.100.1"
      dns_server_list = ["10.0.0.10", "10.0.0.11"]
    }
  }

  tags = [
    vsphere_tag.environment["production"].id,
    vsphere_tag.application["web"].id
  ]
}
```

### Hyper-V Automation
```powershell
# Hyper-V VM Deployment
$VMParams = @{
    Name = "web-server-01"
    MemoryStartupBytes = 16GB
    Generation = 2
    NewVHDPath = "D:\Hyper-V\VMs\web-server-01\disk0.vhdx"
    NewVHDSizeBytes = 100GB
    SwitchName = "Production-Switch"
    Path = "D:\Hyper-V\VMs"
}

# Create VM
$VM = New-VM @VMParams

# Configure VM
Set-VM -Name $VM.Name -ProcessorCount 4 -DynamicMemory -MemoryMaximumBytes 32GB

# Add additional disk
Add-VMHardDiskDrive -VMName $VM.Name -Path "D:\Hyper-V\VMs\web-server-01\disk1.vhdx" -ControllerType SCSI

# Configure networking
Add-VMNetworkAdapter -VMName $VM.Name -SwitchName "Management-Switch" -Name "Management"
Set-VMNetworkAdapterVlan -VMName $VM.Name -VMNetworkAdapterName "Management" -Access -VlanId 10

# Enable Secure Boot with Microsoft template
Set-VMFirmware -VMName $VM.Name -EnableSecureBoot On -SecureBootTemplate MicrosoftWindows

# Enable TPM
Set-VMKeyProtector -VMName $VM.Name -NewLocalKeyProtector
Enable-VMTPM -VMName $VM.Name

# Mount ISO and start
Set-VMDvdDrive -VMName $VM.Name -Path "D:\ISOs\Windows2022.iso"
Start-VM -Name $VM.Name

# Create Hyper-V Replica
Enable-VMReplication -VMName $VM.Name `
    -ReplicaServerName "dr-hyperv.corp.local" `
    -ReplicaServerPort 443 `
    -AuthenticationType Certificate `
    -CertificateThumbprint $CertThumbprint `
    -CompressionEnabled $true `
    -RecoveryHistory 24

Start-VMInitialReplication -VMName $VM.Name
```

---

## 3. Azure Hybrid Connectivity <a name="azure-connectivity"></a>

### ExpressRoute Configuration
```hcl
# ExpressRoute with Terraform
resource "azurerm_express_route_circuit" "main" {
  name                  = "corp-expressroute"
  resource_group_name   = azurerm_resource_group.network.name
  location              = azurerm_resource_group.network.location
  service_provider_name = "Equinix"
  peering_location      = "Washington DC"
  bandwidth_in_mbps     = 1000
  
  sku {
    tier   = "Premium"
    family = "MeteredData"
  }
  
  tags = local.common_tags
}

# ExpressRoute Gateway
resource "azurerm_virtual_network_gateway" "expressroute" {
  name                = "expressroute-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  
  type     = "ExpressRoute"
  vpn_type = "PolicyBased"
  
  sku      = "ErGw2AZ"
  
  ip_configuration {
    name                          = "vnetGatewayConfig"
    public_ip_address_id          = azurerm_public_ip.expressroute_gw.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.gateway.id
  }
}

# Connection
resource "azurerm_virtual_network_gateway_connection" "expressroute" {
  name                       = "expressroute-connection"
  resource_group_name        = azurerm_resource_group.network.name
  location                   = azurerm_resource_group.network.location
  
  type                       = "ExpressRoute"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.expressroute.id
  express_route_circuit_id   = azurerm_express_route_circuit.main.id
  
  routing_weight = 10
}
```

### Site-to-Site VPN
```hcl
# VPN Gateway
resource "azurerm_virtual_network_gateway" "vpn" {
  name                = "vpn-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  
  type     = "Vpn"
  vpn_type = "RouteBased"
  
  active_active = true
  enable_bgp    = true
  sku           = "VpnGw2AZ"
  
  ip_configuration {
    name                          = "vnetGatewayConfig1"
    public_ip_address_id          = azurerm_public_ip.vpn_gw_1.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.gateway.id
  }
  
  ip_configuration {
    name                          = "vnetGatewayConfig2"
    public_ip_address_id          = azurerm_public_ip.vpn_gw_2.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.gateway.id
  }
  
  bgp_settings {
    asn = 65515
    
    peering_addresses {
      ip_configuration_name = "vnetGatewayConfig1"
      apipa_addresses       = ["169.254.21.1"]
    }
    
    peering_addresses {
      ip_configuration_name = "vnetGatewayConfig2"
      apipa_addresses       = ["169.254.21.2"]
    }
  }
}

# Local Network Gateway (On-premises)
resource "azurerm_local_network_gateway" "onprem" {
  name                = "onprem-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  
  gateway_address = "203.0.113.1"  # On-premises VPN device public IP
  address_space   = ["10.0.0.0/8", "192.168.0.0/16"]
  
  bgp_settings {
    asn                 = 65000
    bgp_peering_address = "10.0.0.1"
  }
}

# VPN Connection
resource "azurerm_virtual_network_gateway_connection" "s2s" {
  name                       = "onprem-connection"
  resource_group_name        = azurerm_resource_group.network.name
  location                   = azurerm_resource_group.network.location
  
  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.vpn.id
  local_network_gateway_id   = azurerm_local_network_gateway.onprem.id
  
  shared_key = var.vpn_shared_key
  
  enable_bgp = true
  
  ipsec_policy {
    dh_group         = "DHGroup14"
    ike_encryption   = "AES256"
    ike_integrity    = "SHA256"
    ipsec_encryption = "AES256"
    ipsec_integrity  = "SHA256"
    pfs_group        = "PFS2048"
    sa_lifetime      = 27000
  }
}
```

### Azure Private Link
```hcl
# Private Endpoint for Azure SQL
resource "azurerm_private_endpoint" "sql" {
  name                = "sql-private-endpoint"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  subnet_id           = azurerm_subnet.private_endpoints.id
  
  private_service_connection {
    name                           = "sql-privateserviceconnection"
    private_connection_resource_id = azurerm_mssql_server.main.id
    subresource_names              = ["sqlServer"]
    is_manual_connection           = false
  }
  
  private_dns_zone_group {
    name                 = "sql-dns-zone-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.sql.id]
  }
}

# Private DNS Zone
resource "azurerm_private_dns_zone" "sql" {
  name                = "privatelink.database.windows.net"
  resource_group_name = azurerm_resource_group.main.name
}

# Link DNS Zone to VNet
resource "azurerm_private_dns_zone_virtual_network_link" "sql" {
  name                  = "sql-dns-link"
  resource_group_name   = azurerm_resource_group.main.name
  private_dns_zone_name = azurerm_private_dns_zone.sql.name
  virtual_network_id    = azurerm_virtual_network.main.id
  registration_enabled  = false
}

# DNS Forwarder on-premises (Windows DNS)
# Add conditional forwarder for privatelink.* domains to Azure DNS (168.63.129.16)
```

---

## 4. Bare Metal and VMware Integration <a name="baremetal-vmware"></a>

### Ansible for Bare Metal Provisioning
```yaml
# playbooks/baremetal-provision.yml
---
- name: Bare Metal Server Provisioning
  hosts: baremetal_servers
  gather_facts: no
  
  vars:
    pxe_server: "10.0.0.5"
    kickstart_url: "http://{{ pxe_server }}/ks/rhel8-base.cfg"
    
  tasks:
    - name: Configure IPMI settings
      ipmi_boot:
        name: "{{ inventory_hostname }}"
        bootdev: pxe
        uefiboot: yes
        persistent: no
        user: "{{ ipmi_user }}"
        password: "{{ ipmi_password }}"
      delegate_to: localhost
      
    - name: Power cycle server
      ipmi_power:
        name: "{{ inventory_hostname }}"
        state: reset
        user: "{{ ipmi_user }}"
        password: "{{ ipmi_password }}"
      delegate_to: localhost
      
    - name: Wait for installation to complete
      wait_for:
        host: "{{ ansible_host }}"
        port: 22
        delay: 600
        timeout: 3600
      delegate_to: localhost
      
    - name: Gather facts after install
      setup:
      
    - name: Apply base configuration
      include_role:
        name: server_baseline
        
    - name: Configure storage
      include_role:
        name: storage_config
      vars:
        raid_level: 10
        lvm_volumes:
          - name: root
            size: 50G
          - name: var
            size: 100G
          - name: data
            size: 500G

---
# roles/storage_config/tasks/main.yml
- name: Create RAID array
  community.general.mdadm:
    name: /dev/md0
    devices:
      - /dev/sda
      - /dev/sdb
      - /dev/sdc
      - /dev/sdd
    level: "{{ raid_level }}"
    state: present
    
- name: Create volume group
  lvg:
    vg: vg_data
    pvs: /dev/md0
    
- name: Create logical volumes
  lvol:
    vg: vg_data
    lv: "{{ item.name }}"
    size: "{{ item.size }}"
  loop: "{{ lvm_volumes }}"
  
- name: Create filesystems
  filesystem:
    fstype: xfs
    dev: "/dev/vg_data/{{ item.name }}"
  loop: "{{ lvm_volumes }}"
```

### VMware Cloud on Azure Integration
```hcl
# Azure VMware Solution (AVS)
resource "azurerm_vmware_private_cloud" "avs" {
  name                = "avs-private-cloud"
  resource_group_name = azurerm_resource_group.avs.name
  location            = azurerm_resource_group.avs.location
  
  sku_name = "av36"
  
  management_cluster {
    size = 3
  }
  
  network_subnet_cidr         = "10.100.0.0/22"
  internet_connection_enabled = false
  nsxt_password               = var.nsxt_password
  vcenter_password            = var.vcenter_password
  
  tags = local.common_tags
}

# ExpressRoute connection to AVS
resource "azurerm_vmware_express_route_authorization" "avs" {
  name             = "avs-expressroute-auth"
  private_cloud_id = azurerm_vmware_private_cloud.avs.id
}

resource "azurerm_virtual_network_gateway_connection" "avs" {
  name                       = "avs-connection"
  resource_group_name        = azurerm_resource_group.network.name
  location                   = azurerm_resource_group.network.location
  
  type                            = "ExpressRoute"
  virtual_network_gateway_id      = azurerm_virtual_network_gateway.expressroute.id
  express_route_circuit_id        = azurerm_vmware_private_cloud.avs.circuit[0].express_route_id
  authorization_key               = azurerm_vmware_express_route_authorization.avs.express_route_authorization_key
}
```

---

## 5. Multi-Cloud Strategies <a name="multi-cloud"></a>

### Multi-Cloud Architecture Patterns
```
Pattern 1: Active-Active Multi-Cloud
═══════════════════════════════════════════════════════════════════

         ┌───────────────────────────────────────────┐
         │            Global Load Balancer           │
         │         (Traffic Manager / DNS)           │
         └─────────────────┬─────────────────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
    ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
    │  Azure  │       │  On-Prem │       │  Other  │
    │ Region  │       │   DC     │       │  Cloud  │
    └────┬────┘       └────┬────┘       └────┬────┘
         │                 │                 │
    ┌────┴────┐       ┌────┴────┐       ┌────┴────┐
    │  App    │       │  App    │       │  App    │
    │ Cluster │       │ Cluster │       │ Cluster │
    └─────────┘       └─────────┘       └─────────┘


Pattern 2: Burst to Cloud
═══════════════════════════════════════════════════════════════════

    ┌─────────────────────────────────────────────────────────┐
    │                    Load Balancer                         │
    └─────────────────────────┬───────────────────────────────┘
                              │
              Normal Load     │     High Load
         ┌────────────────────┼────────────────────┐
         │                    │                    │
    ┌────┴────────────────────┴───┐        ┌───────┴───────┐
    │      On-Premises (Base)     │   ◄──► │  Cloud Burst  │
    │      Always Running         │        │  Scale Out    │
    │      Fixed Capacity         │        │  Auto-scale   │
    └─────────────────────────────┘        └───────────────┘
```

### Terraform Multi-Cloud Configuration
```hcl
# providers.tf - Multi-cloud providers
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    vsphere = {
      source  = "hashicorp/vsphere"
      version = "~> 2.0"
    }
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.azure_subscription_id
}

provider "vsphere" {
  user                 = var.vsphere_user
  password             = var.vsphere_password
  vsphere_server       = var.vsphere_server
  allow_unverified_ssl = true
}

# modules/compute/main.tf
# Unified compute abstraction
variable "deployment_target" {
  type        = string
  description = "azure, vsphere, or hybrid"
  validation {
    condition     = contains(["azure", "vsphere", "hybrid"], var.deployment_target)
    error_message = "Must be azure, vsphere, or hybrid."
  }
}

resource "azurerm_linux_virtual_machine" "vm" {
  count = var.deployment_target == "azure" || var.deployment_target == "hybrid" ? var.vm_count : 0
  
  name                = "${var.name_prefix}-azure-${count.index}"
  resource_group_name = var.resource_group_name
  location            = var.location
  size                = var.azure_vm_size
  # ... rest of configuration
}

resource "vsphere_virtual_machine" "vm" {
  count = var.deployment_target == "vsphere" || var.deployment_target == "hybrid" ? var.vm_count : 0
  
  name             = "${var.name_prefix}-onprem-${count.index}"
  resource_pool_id = var.vsphere_resource_pool_id
  datastore_id     = var.vsphere_datastore_id
  # ... rest of configuration
}

# Output unified inventory
output "compute_inventory" {
  value = {
    azure_vms = [for vm in azurerm_linux_virtual_machine.vm : {
      name       = vm.name
      private_ip = vm.private_ip_address
      location   = "azure"
    }]
    vsphere_vms = [for vm in vsphere_virtual_machine.vm : {
      name       = vm.name
      private_ip = vm.default_ip_address
      location   = "onprem"
    }]
  }
}
```

### Kubernetes Multi-Cluster with Submariner
```yaml
# Connect on-premises OpenShift with AKS
# Install Submariner on both clusters

# Broker installation (on hub cluster)
subctl deploy-broker --kubeconfig hub-config

# Join cluster 1 (on-premises OpenShift)
subctl join broker-info.subm \
  --kubeconfig onprem-config \
  --clusterid onprem-cluster \
  --natt=false

# Join cluster 2 (AKS)
subctl join broker-info.subm \
  --kubeconfig aks-config \
  --clusterid aks-cluster \
  --natt=true

# Export service for cross-cluster access
kubectl annotate service my-service \
  submariner.io/export=true

# Access from other cluster
# my-service.my-namespace.svc.clusterset.local
```

---

## 6. Azure Arc for Hybrid Management <a name="azure-arc"></a>

### Azure Arc Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                         Azure Portal                             │
│              (Unified Management Interface)                      │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │Azure Policy │ │Azure Monitor│ │   Defender  │ │Azure Update ││
│  │             │ │             │ │  for Cloud  │ │  Manager    ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘│
├─────────────────────────────────────────────────────────────────┤
│                        Azure Arc                                 │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐       │
│  │ Arc-enabled    │ │ Arc-enabled    │ │ Arc-enabled    │       │
│  │ Servers        │ │ Kubernetes     │ │ Data Services  │       │
│  └───────┬────────┘ └───────┬────────┘ └───────┬────────┘       │
└──────────┼──────────────────┼──────────────────┼────────────────┘
           │                  │                  │
           ▼                  ▼                  ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│   On-Premises    │ │   Edge Sites     │ │   Other Clouds   │
│   Data Center    │ │                  │ │                  │
│  ┌────┐ ┌────┐   │ │  ┌────┐ ┌────┐  │ │  ┌────┐ ┌────┐   │
│  │ VM │ │ VM │   │ │  │K8s │ │ IoT│  │ │  │ VM │ │ K8s│   │
│  └────┘ └────┘   │ │  └────┘ └────┘  │ │  └────┘ └────┘   │
│  ┌────┐ ┌────┐   │ │                  │ │                  │
│  │K8s │ │SQL │   │ │                  │ │                  │
│  └────┘ └────┘   │ │                  │ │                  │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### Arc-Enabled Servers
```bash
# Install Azure Connected Machine Agent
# Download and run installation script

# Windows
$env:SUBSCRIPTION_ID = "your-subscription-id"
$env:RESOURCE_GROUP = "arc-servers-rg"
$env:LOCATION = "eastus"
$env:TENANT_ID = "your-tenant-id"

# Download the installation script
Invoke-WebRequest -Uri https://aka.ms/azcmagent-windows -OutFile "$env:TEMP\install_windows_azcmagent.ps1"

# Install the agent
& "$env:TEMP\install_windows_azcmagent.ps1"

# Connect to Azure
azcmagent connect `
    --subscription-id $env:SUBSCRIPTION_ID `
    --resource-group $env:RESOURCE_GROUP `
    --location $env:LOCATION `
    --tenant-id $env:TENANT_ID

# Linux
curl -L https://aka.ms/azcmagent-linux | bash

azcmagent connect \
    --subscription-id "${SUBSCRIPTION_ID}" \
    --resource-group "${RESOURCE_GROUP}" \
    --location "${LOCATION}" \
    --tenant-id "${TENANT_ID}"
```

### Arc-Enabled Kubernetes
```bash
# Connect Kubernetes cluster to Azure Arc
az connectedk8s connect \
    --name "onprem-k8s-cluster" \
    --resource-group "arc-k8s-rg" \
    --location "eastus" \
    --correlation-id "unique-id"

# Enable GitOps
az k8s-configuration flux create \
    --name "gitops-config" \
    --cluster-name "onprem-k8s-cluster" \
    --resource-group "arc-k8s-rg" \
    --cluster-type connectedClusters \
    --scope cluster \
    --url "https://github.com/org/gitops-repo" \
    --branch main \
    --kustomization name=infra path=./infrastructure prune=true \
    --kustomization name=apps path=./apps prune=true dependsOn=["infra"]

# Enable Azure Monitor for containers
az k8s-extension create \
    --name "azuremonitor-containers" \
    --cluster-name "onprem-k8s-cluster" \
    --resource-group "arc-k8s-rg" \
    --cluster-type connectedClusters \
    --extension-type Microsoft.AzureMonitor.Containers

# Enable Azure Policy
az k8s-extension create \
    --name "azure-policy" \
    --cluster-name "onprem-k8s-cluster" \
    --resource-group "arc-k8s-rg" \
    --cluster-type connectedClusters \
    --extension-type Microsoft.PolicyInsights
```

### Arc Terraform Configuration
```hcl
# Azure Arc resources
resource "azurerm_arc_machine" "server" {
  name                = "onprem-server-01"
  resource_group_name = azurerm_resource_group.arc.name
  location            = azurerm_resource_group.arc.location
  kind                = "SCVMM"  # or VMware, AWS, GCP
  
  tags = {
    Environment = "Production"
    OS          = "Windows"
  }
}

# Apply Azure Policy to Arc servers
resource "azurerm_resource_group_policy_assignment" "arc_monitoring" {
  name                 = "enable-vm-insights"
  resource_group_id    = azurerm_resource_group.arc.id
  policy_definition_id = "/providers/Microsoft.Authorization/policySetDefinitions/..."
  
  parameters = jsonencode({
    logAnalyticsWorkspace = {
      value = azurerm_log_analytics_workspace.main.id
    }
  })
}

# Arc-enabled Kubernetes
resource "azurerm_arc_kubernetes_cluster" "main" {
  name                         = "onprem-k8s"
  resource_group_name          = azurerm_resource_group.arc.name
  location                     = azurerm_resource_group.arc.location
  agent_public_key_certificate = filebase64("agent-cert.pem")
  
  identity {
    type = "SystemAssigned"
  }
}

# Flux GitOps Configuration
resource "azurerm_arc_kubernetes_flux_configuration" "main" {
  name       = "gitops-config"
  cluster_id = azurerm_arc_kubernetes_cluster.main.id
  namespace  = "flux-system"
  scope      = "cluster"
  
  git_repository {
    url                      = "https://github.com/org/gitops-repo"
    reference_type           = "branch"
    reference_value          = "main"
    sync_interval_in_seconds = 60
  }
  
  kustomizations {
    name                       = "infrastructure"
    path                       = "./infrastructure"
    sync_interval_in_seconds   = 120
    retry_interval_in_seconds  = 60
    prune                      = true
  }
  
  kustomizations {
    name                       = "applications"
    path                       = "./apps"
    sync_interval_in_seconds   = 120
    depends_on                 = ["infrastructure"]
    prune                      = true
  }
}
```

---

## 7. Storage and Data Strategies <a name="storage"></a>

### Hybrid Storage Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    Data Tiering Strategy                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Hot Tier (Frequently Accessed)                                  │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ On-Premises │  │  Azure      │                               │
│  │ NVMe/SSD    │  │ Premium SSD │                               │
│  └─────────────┘  └─────────────┘                               │
│         │                │                                       │
│         ▼                ▼                                       │
│  Warm Tier (Occasional Access)                                   │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ On-Premises │  │  Azure      │                               │
│  │ SAN/NAS     │  │ Standard SSD│                               │
│  └─────────────┘  └─────────────┘                               │
│         │                │                                       │
│         ▼                ▼                                       │
│  Cold Tier (Archive)                                             │
│  ┌─────────────────────────────┐                                │
│  │      Azure Blob Storage     │                                │
│  │    (Cool / Archive Tier)    │                                │
│  └─────────────────────────────┘                                │
└─────────────────────────────────────────────────────────────────┘
```

### Azure File Sync
```powershell
# Install Azure File Sync Agent
$DownloadURL = "https://aka.ms/afs/agent"
$InstallerPath = "$env:TEMP\StorageSyncAgent.msi"
Invoke-WebRequest -Uri $DownloadURL -OutFile $InstallerPath
Start-Process msiexec.exe -ArgumentList "/i $InstallerPath /qn" -Wait

# Register Server with Storage Sync Service
$StorageSyncServiceRG = "storage-sync-rg"
$StorageSyncServiceName = "corp-storage-sync"
$SubscriptionId = "your-subscription-id"
$TenantId = "your-tenant-id"

# Login and register
Login-AzStorageSyncServer `
    -SubscriptionId $SubscriptionId `
    -ResourceGroupName $StorageSyncServiceRG `
    -StorageSyncServiceName $StorageSyncServiceName

# Create Server Endpoint
$SyncGroupName = "file-share-sync"
$ServerEndpointPath = "D:\Shares\Department"

New-AzStorageSyncServerEndpoint `
    -ResourceGroupName $StorageSyncServiceRG `
    -StorageSyncServiceName $StorageSyncServiceName `
    -SyncGroupName $SyncGroupName `
    -ServerResourceId (Get-AzStorageSyncServer -ResourceGroupName $StorageSyncServiceRG -StorageSyncServiceName $StorageSyncServiceName).ResourceId `
    -ServerLocalPath $ServerEndpointPath `
    -CloudTiering `
    -VolumeFreeSpacePercent 20 `
    -TierFilesOlderThanDays 30
```

---

## 8. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: Designing Hybrid Architecture for Financial Institution

**Challenge**: Design hybrid infrastructure for bank with strict compliance requirements

**Solution Architecture**:
```
Requirements:
├── Data sovereignty (certain data must stay on-premises)
├── PCI-DSS compliance
├── 99.99% availability
├── Disaster recovery < 1 hour RTO

Architecture:
┌──────────────────────────────────────────────────────────────────┐
│                    Primary Data Center                            │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │           Core Banking (On-Premises Only)                │    │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐         │    │
│  │  │  Core  │  │ Oracle │  │ HSMs   │  │ Vault  │         │    │
│  │  │Banking │  │  DB    │  │        │  │        │         │    │
│  │  └────────┘  └────────┘  └────────┘  └────────┘         │    │
│  └──────────────────────────────────────────────────────────┘    │
│                           │                                       │
│               ExpressRoute (Private Peering)                      │
│                           │                                       │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │                Azure (Hybrid Workloads)                   │    │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐         │    │
│  │  │  API   │  │Analytics│  │ Web    │  │ Mobile │         │    │
│  │  │Gateway │  │ (Azure │  │  Apps  │  │ Backend│         │    │
│  │  │        │  │Synapse)│  │        │  │        │         │    │
│  │  └────────┘  └────────┘  └────────┘  └────────┘         │    │
│  └──────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────┘

Key Design Decisions:
├── Core banking stays on-premises (compliance)
├── ExpressRoute with encryption for connectivity
├── Azure Arc for unified management
├── Private endpoints for all Azure services
├── Key Vault with HSM backing
└── Cross-region replication for Azure workloads
```

### Scenario 2: Multi-Cloud Disaster Recovery

**Challenge**: Implement DR strategy across on-premises and Azure

**Solution**:
```yaml
DR Strategy: Active-Passive with Automated Failover
─────────────────────────────────────────────────────────

Primary Site (On-Premises):
  Infrastructure:
    - VMware vSphere cluster (6 hosts)
    - 200 VMs (Windows/Linux)
    - SAN storage with replication
    - OpenShift cluster (production)
  
  RPO: 15 minutes (async replication)
  
Secondary Site (Azure):
  Infrastructure:
    - Azure Site Recovery vault
    - Azure VMs (warm standby)
    - Azure Kubernetes Service
    - Azure SQL (geo-replicated)
  
  RTO: 1 hour

Automation:
  - Runbooks for failover orchestration
  - DNS failover via Traffic Manager
  - Application-aware failover ordering
  - Automated testing (monthly DR drills)

Implementation:
  1. ASR for VM replication
  2. SQL AlwaysOn availability groups
  3. Storage Account geo-replication
  4. Kubernetes stateful set replication
  5. Monitoring with Azure Monitor alerts
```

### Common Interview Questions

**Q1: "How do you ensure consistency across hybrid environments?"**

**Answer**:
1. **Infrastructure as Code**: Single Terraform codebase with provider abstraction
2. **GitOps**: Argo CD/Flux managing all Kubernetes clusters
3. **Configuration Management**: Ansible with unified inventory (dynamic)
4. **Monitoring**: Azure Monitor + Arc for unified observability
5. **Security**: Azure Policy with Arc enforcement
6. **Identity**: Azure AD with hybrid join

**Q2: "Explain your approach to hybrid network security"**

**Answer**:
```
Network Security Layers:
├── Perimeter
│   ├── Azure Firewall Premium (IDPS, TLS inspection)
│   ├── On-premises next-gen firewall
│   └── DDoS Protection Standard
├── Connectivity
│   ├── ExpressRoute with encryption
│   ├── Site-to-Site VPN (backup)
│   └── Private endpoints for all PaaS
├── Segmentation
│   ├── Network Security Groups
│   ├── Application Security Groups
│   └── Microsegmentation (VMware NSX / Azure)
├── Monitoring
│   ├── Network Watcher
│   ├── Traffic Analytics
│   └── Defender for Cloud
└── Identity
    ├── Just-in-time VM access
    ├── Privileged Identity Management
    └── Conditional Access
```

**Q3: "How do you handle data residency in hybrid cloud?"**

**Answer**:
1. **Classification**: Tag data with residency requirements
2. **Policy**: Azure Policy to prevent data leaving region
3. **Architecture**: Separate landing zones per region
4. **Encryption**: Customer-managed keys in specific regions
5. **Monitoring**: DLP policies and alerts
6. **Compliance**: Regular audits and attestations

---

## Quick Reference Commands

```bash
# Azure Arc
azcmagent show
azcmagent connect --service-principal-id <id>
az connectedk8s list

# ExpressRoute
az network express-route show -n circuit-name -g rg-name
az network express-route peering list -g rg-name --circuit-name circuit

# VPN
az network vnet-gateway list -g rg-name
az network vpn-connection show -n connection-name -g rg-name

# Multi-cloud Terraform
terraform workspace list
terraform plan -var-file=azure.tfvars
terraform plan -var-file=vsphere.tfvars

# Kubernetes multi-cluster
kubectl config get-contexts
kubectl config use-context onprem-cluster
subctl show all
```
