# Module 24: Network Infrastructure Management

## Table of Contents
1. [Network Architecture Fundamentals](#network-fundamentals)
2. [Azure Virtual Networking](#azure-vnet)
3. [Firewall Management](#firewall-management)
4. [Load Balancing and Traffic Management](#load-balancing)
5. [DNS Management](#dns-management)
6. [Network Security and Microsegmentation](#network-security)
7. [Hybrid Network Connectivity](#hybrid-connectivity)
8. [Network Automation with Ansible](#network-automation)
9. [Interview Scenarios](#interview-scenarios)

---

## 1. Network Architecture Fundamentals <a name="network-fundamentals"></a>

### Enterprise Network Topology
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Enterprise Network Architecture                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                           INTERNET                                    │   │
│  └───────────────────────────────┬──────────────────────────────────────┘   │
│                                  │                                           │
│                    ┌─────────────┴─────────────┐                            │
│                    │      Edge Firewall        │                            │
│                    │   (Azure Firewall/Palo)   │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│  ┌───────────────────────────────┴───────────────────────────────────────┐  │
│  │                          DMZ / Perimeter                               │  │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐               │  │
│  │   │ WAF/App GW  │    │ API Gateway │    │ Bastion     │               │  │
│  │   └─────────────┘    └─────────────┘    └─────────────┘               │  │
│  └───────────────────────────────┬───────────────────────────────────────┘  │
│                                  │                                           │
│  ┌───────────────────────────────┴───────────────────────────────────────┐  │
│  │                         Internal Network                               │  │
│  │                                                                        │  │
│  │  ┌─────────────────────────┐    ┌─────────────────────────┐           │  │
│  │  │    Web Tier (NSG)       │    │    App Tier (NSG)       │           │  │
│  │  │  10.1.1.0/24            │    │  10.1.2.0/24            │           │  │
│  │  │  ├── Web Server 1       │───▶│  ├── App Server 1       │           │  │
│  │  │  ├── Web Server 2       │    │  ├── App Server 2       │           │  │
│  │  │  └── Web Server 3       │    │  └── App Server 3       │           │  │
│  │  └─────────────────────────┘    └───────────┬─────────────┘           │  │
│  │                                              │                         │  │
│  │                         ┌────────────────────┴────────────────────┐   │  │
│  │                         │        Data Tier (NSG)                  │   │  │
│  │                         │  10.1.3.0/24                            │   │  │
│  │                         │  ├── SQL Primary                        │   │  │
│  │                         │  ├── SQL Secondary                      │   │  │
│  │                         │  └── Redis Cache                        │   │  │
│  │                         └─────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                  │                                           │
│                    ┌─────────────┴─────────────┐                            │
│                    │    ExpressRoute / VPN     │                            │
│                    │   (Hybrid Connectivity)   │                            │
│                    └─────────────┬─────────────┘                            │
│                                  │                                           │
│  ┌───────────────────────────────┴───────────────────────────────────────┐  │
│  │                      On-Premises Datacenter                            │  │
│  │   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐               │  │
│  │   │ AD Domain   │    │ File Server │    │ Legacy Apps │               │  │
│  │   │ Controllers │    │             │    │             │               │  │
│  │   └─────────────┘    └─────────────┘    └─────────────┘               │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### IP Address Planning
```yaml
# network-ip-planning.yaml
enterprise_network:
  cloud:
    azure:
      address_space: "10.0.0.0/8"
      regions:
        eastus:
          hub_vnet: "10.1.0.0/16"
          spoke_vnets:
            production: "10.2.0.0/16"
            staging: "10.3.0.0/16"
            development: "10.4.0.0/16"
        westus:
          hub_vnet: "10.10.0.0/16"
          spoke_vnets:
            production: "10.11.0.0/16"
            dr_site: "10.12.0.0/16"
  
  on_premises:
    datacenter_primary:
      address_space: "172.16.0.0/12"
      vlans:
        servers: "172.16.0.0/16"
        workstations: "172.17.0.0/16"
        management: "172.18.0.0/24"
        dmz: "172.19.0.0/24"
    
    datacenter_dr:
      address_space: "172.20.0.0/14"
  
  reserved_ranges:
    vpn_clients: "192.168.100.0/24"
    container_networks: "10.244.0.0/16"
    service_mesh: "10.245.0.0/16"
```

---

## 2. Azure Virtual Networking <a name="azure-vnet"></a>

### Hub-Spoke Network Architecture
```hcl
# Hub VNet
resource "azurerm_virtual_network" "hub" {
  name                = "hub-vnet"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  address_space       = ["10.1.0.0/16"]
  
  tags = {
    Environment = "Production"
    Type        = "Hub"
  }
}

# Hub Subnets
resource "azurerm_subnet" "firewall" {
  name                 = "AzureFirewallSubnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.1.0.0/26"]
}

resource "azurerm_subnet" "gateway" {
  name                 = "GatewaySubnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.1.1.0/27"]
}

resource "azurerm_subnet" "bastion" {
  name                 = "AzureBastionSubnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.1.2.0/26"]
}

resource "azurerm_subnet" "management" {
  name                 = "management"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.1.3.0/24"]
}

# Spoke VNets (Production)
resource "azurerm_virtual_network" "spoke_prod" {
  name                = "spoke-prod-vnet"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  address_space       = ["10.2.0.0/16"]
}

resource "azurerm_subnet" "prod_web" {
  name                 = "web-tier"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.spoke_prod.name
  address_prefixes     = ["10.2.1.0/24"]
  
  delegation {
    name = "appservice"
    service_delegation {
      name = "Microsoft.Web/serverFarms"
      actions = [
        "Microsoft.Network/virtualNetworks/subnets/action"
      ]
    }
  }
}

resource "azurerm_subnet" "prod_app" {
  name                 = "app-tier"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.spoke_prod.name
  address_prefixes     = ["10.2.2.0/24"]
  
  service_endpoints = [
    "Microsoft.Sql",
    "Microsoft.Storage",
    "Microsoft.KeyVault"
  ]
}

resource "azurerm_subnet" "prod_data" {
  name                 = "data-tier"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.spoke_prod.name
  address_prefixes     = ["10.2.3.0/24"]
  
  private_endpoint_network_policies_enabled = true
}

resource "azurerm_subnet" "prod_aks" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.network.name
  virtual_network_name = azurerm_virtual_network.spoke_prod.name
  address_prefixes     = ["10.2.16.0/20"]  # Large subnet for AKS nodes
}

# VNet Peering Hub to Spoke
resource "azurerm_virtual_network_peering" "hub_to_spoke_prod" {
  name                         = "hub-to-spoke-prod"
  resource_group_name          = azurerm_resource_group.network.name
  virtual_network_name         = azurerm_virtual_network.hub.name
  remote_virtual_network_id    = azurerm_virtual_network.spoke_prod.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  allow_gateway_transit        = true
}

resource "azurerm_virtual_network_peering" "spoke_prod_to_hub" {
  name                         = "spoke-prod-to-hub"
  resource_group_name          = azurerm_resource_group.network.name
  virtual_network_name         = azurerm_virtual_network.spoke_prod.name
  remote_virtual_network_id    = azurerm_virtual_network.hub.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  use_remote_gateways          = true
}
```

### Network Security Groups
```hcl
# NSG for Web Tier
resource "azurerm_network_security_group" "web_tier" {
  name                = "web-tier-nsg"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_network_security_rule" "allow_https_inbound" {
  name                        = "Allow-HTTPS-Inbound"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "443"
  source_address_prefix       = "Internet"
  destination_address_prefix  = "VirtualNetwork"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.web_tier.name
}

resource "azurerm_network_security_rule" "allow_appgw_probe" {
  name                        = "Allow-AppGW-Probe"
  priority                    = 110
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "65200-65535"
  source_address_prefix       = "GatewayManager"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.web_tier.name
}

resource "azurerm_network_security_rule" "deny_all_inbound" {
  name                        = "Deny-All-Inbound"
  priority                    = 4096
  direction                   = "Inbound"
  access                      = "Deny"
  protocol                    = "*"
  source_port_range           = "*"
  destination_port_range      = "*"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.web_tier.name
}

# NSG for App Tier
resource "azurerm_network_security_group" "app_tier" {
  name                = "app-tier-nsg"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_network_security_rule" "allow_web_to_app" {
  name                        = "Allow-Web-to-App"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "8080"
  source_address_prefix       = "10.2.1.0/24"  # Web tier subnet
  destination_address_prefix  = "VirtualNetwork"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.app_tier.name
}

# NSG for Data Tier
resource "azurerm_network_security_group" "data_tier" {
  name                = "data-tier-nsg"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_network_security_rule" "allow_app_to_sql" {
  name                        = "Allow-App-to-SQL"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_range      = "1433"
  source_address_prefix       = "10.2.2.0/24"  # App tier subnet
  destination_address_prefix  = "VirtualNetwork"
  resource_group_name         = azurerm_resource_group.network.name
  network_security_group_name = azurerm_network_security_group.data_tier.name
}

# Subnet NSG Associations
resource "azurerm_subnet_network_security_group_association" "web" {
  subnet_id                 = azurerm_subnet.prod_web.id
  network_security_group_id = azurerm_network_security_group.web_tier.id
}

resource "azurerm_subnet_network_security_group_association" "app" {
  subnet_id                 = azurerm_subnet.prod_app.id
  network_security_group_id = azurerm_network_security_group.app_tier.id
}

resource "azurerm_subnet_network_security_group_association" "data" {
  subnet_id                 = azurerm_subnet.prod_data.id
  network_security_group_id = azurerm_network_security_group.data_tier.id
}
```

---

## 3. Firewall Management <a name="firewall-management"></a>

### Azure Firewall Configuration
```hcl
# Azure Firewall
resource "azurerm_firewall" "hub" {
  name                = "hub-firewall"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  sku_name            = "AZFW_VNet"
  sku_tier            = "Premium"
  
  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.firewall.id
    public_ip_address_id = azurerm_public_ip.firewall.id
  }
  
  firewall_policy_id = azurerm_firewall_policy.main.id
}

# Firewall Policy
resource "azurerm_firewall_policy" "main" {
  name                     = "hub-firewall-policy"
  resource_group_name      = azurerm_resource_group.network.name
  location                 = azurerm_resource_group.network.location
  sku                      = "Premium"
  threat_intelligence_mode = "Alert"
  
  dns {
    proxy_enabled = true
    servers       = ["168.63.129.16"]  # Azure DNS
  }
  
  intrusion_detection {
    mode = "Alert"
    
    signature_overrides {
      id    = "2024897"
      state = "Deny"
    }
    
    traffic_bypass {
      name                  = "SecretBypass"
      protocol              = "TCP"
      destination_addresses = ["10.1.3.0/24"]
      destination_ports     = ["443"]
      source_addresses      = ["10.2.0.0/16"]
    }
  }
}

# Application Rules
resource "azurerm_firewall_policy_rule_collection_group" "application" {
  name               = "application-rules"
  firewall_policy_id = azurerm_firewall_policy.main.id
  priority           = 200
  
  application_rule_collection {
    name     = "AllowMicrosoft"
    priority = 100
    action   = "Allow"
    
    rule {
      name = "WindowsUpdate"
      protocols {
        type = "Https"
        port = 443
      }
      source_addresses  = ["10.0.0.0/8"]
      destination_fqdns = [
        "*.microsoft.com",
        "*.windowsupdate.com",
        "*.update.microsoft.com"
      ]
    }
    
    rule {
      name = "AzureServices"
      protocols {
        type = "Https"
        port = 443
      }
      source_addresses = ["10.0.0.0/8"]
      destination_fqdn_tags = [
        "AzureActiveDirectory",
        "AzureMonitor",
        "AzureBackup"
      ]
    }
  }
  
  application_rule_collection {
    name     = "AllowContainerRegistry"
    priority = 200
    action   = "Allow"
    
    rule {
      name = "ACR"
      protocols {
        type = "Https"
        port = 443
      }
      source_addresses  = ["10.2.16.0/20"]  # AKS subnet
      destination_fqdns = [
        "*.azurecr.io",
        "*.blob.core.windows.net",
        "mcr.microsoft.com"
      ]
    }
  }
}

# Network Rules
resource "azurerm_firewall_policy_rule_collection_group" "network" {
  name               = "network-rules"
  firewall_policy_id = azurerm_firewall_policy.main.id
  priority           = 100
  
  network_rule_collection {
    name     = "AllowDNS"
    priority = 100
    action   = "Allow"
    
    rule {
      name                  = "DNS"
      protocols             = ["UDP"]
      source_addresses      = ["10.0.0.0/8"]
      destination_addresses = ["168.63.129.16"]
      destination_ports     = ["53"]
    }
  }
  
  network_rule_collection {
    name     = "AllowNTP"
    priority = 200
    action   = "Allow"
    
    rule {
      name                  = "NTP"
      protocols             = ["UDP"]
      source_addresses      = ["10.0.0.0/8"]
      destination_addresses = ["*"]
      destination_ports     = ["123"]
    }
  }
  
  network_rule_collection {
    name     = "AllowOnPremises"
    priority = 300
    action   = "Allow"
    
    rule {
      name                  = "ToOnPrem"
      protocols             = ["TCP", "UDP"]
      source_addresses      = ["10.0.0.0/8"]
      destination_addresses = ["172.16.0.0/12"]
      destination_ports     = ["*"]
    }
  }
}

# DNAT Rules (Port Forwarding)
resource "azurerm_firewall_policy_rule_collection_group" "dnat" {
  name               = "dnat-rules"
  firewall_policy_id = azurerm_firewall_policy.main.id
  priority           = 50
  
  nat_rule_collection {
    name     = "InboundNAT"
    priority = 100
    action   = "Dnat"
    
    rule {
      name                = "HTTPS-to-Web"
      protocols           = ["TCP"]
      source_addresses    = ["*"]
      destination_address = azurerm_public_ip.firewall.ip_address
      destination_ports   = ["443"]
      translated_address  = "10.2.1.10"  # Web server
      translated_port     = "443"
    }
  }
}
```

### On-Premises Firewall (Palo Alto) Automation
```python
# palo_alto_automation.py
from panos.firewall import Firewall
from panos.policies import SecurityRule, NatRule
from panos.objects import AddressObject, ServiceObject
from typing import List, Dict
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class PaloAltoManager:
    def __init__(self, hostname: str, api_key: str):
        self.fw = Firewall(hostname, api_key=api_key)
        
    def create_security_rule(
        self,
        name: str,
        source_zones: List[str],
        destination_zones: List[str],
        source_addresses: List[str],
        destination_addresses: List[str],
        applications: List[str],
        services: List[str],
        action: str = "allow",
        log_end: bool = True
    ) -> SecurityRule:
        """Create a security rule on the firewall"""
        
        rule = SecurityRule(
            name=name,
            fromzone=source_zones,
            tozone=destination_zones,
            source=source_addresses,
            destination=destination_addresses,
            application=applications,
            service=services,
            action=action,
            log_end=log_end
        )
        
        self.fw.add(rule)
        rule.create()
        
        logger.info(f"Created security rule: {name}")
        return rule
    
    def create_address_object(
        self,
        name: str,
        value: str,
        obj_type: str = "ip-netmask",
        description: str = ""
    ) -> AddressObject:
        """Create an address object"""
        
        obj = AddressObject(
            name=name,
            value=value,
            type=obj_type,
            description=description
        )
        
        self.fw.add(obj)
        obj.create()
        
        logger.info(f"Created address object: {name}")
        return obj
    
    def bulk_create_rules_from_config(self, rules_config: List[Dict]):
        """Create multiple rules from configuration"""
        
        for rule_cfg in rules_config:
            # Create address objects if needed
            for addr in rule_cfg.get('source_addresses', []):
                if '/' in addr or addr.count('.') == 3:
                    obj_name = f"addr-{addr.replace('/', '-').replace('.', '-')}"
                    try:
                        self.create_address_object(obj_name, addr)
                    except Exception:
                        pass  # Object may already exist
            
            # Create the security rule
            self.create_security_rule(
                name=rule_cfg['name'],
                source_zones=rule_cfg['source_zones'],
                destination_zones=rule_cfg['destination_zones'],
                source_addresses=rule_cfg.get('source_addresses', ['any']),
                destination_addresses=rule_cfg.get('destination_addresses', ['any']),
                applications=rule_cfg.get('applications', ['any']),
                services=rule_cfg.get('services', ['application-default']),
                action=rule_cfg.get('action', 'allow')
            )
        
        # Commit changes
        self.commit()
    
    def commit(self, description: str = "Automated commit"):
        """Commit configuration changes"""
        self.fw.commit(sync=True, description=description)
        logger.info("Configuration committed successfully")

# Example usage
if __name__ == "__main__":
    manager = PaloAltoManager(
        hostname="192.168.1.1",
        api_key="YOUR_API_KEY"
    )
    
    rules = [
        {
            "name": "Allow-Web-to-App",
            "source_zones": ["web-tier"],
            "destination_zones": ["app-tier"],
            "source_addresses": ["10.2.1.0/24"],
            "destination_addresses": ["10.2.2.0/24"],
            "applications": ["web-browsing", "ssl"],
            "services": ["application-default"]
        },
        {
            "name": "Allow-App-to-DB",
            "source_zones": ["app-tier"],
            "destination_zones": ["data-tier"],
            "source_addresses": ["10.2.2.0/24"],
            "destination_addresses": ["10.2.3.0/24"],
            "applications": ["mssql-db"],
            "services": ["tcp/1433"]
        }
    ]
    
    manager.bulk_create_rules_from_config(rules)
```

---

## 4. Load Balancing and Traffic Management <a name="load-balancing"></a>

### Azure Load Balancer
```hcl
# Standard Load Balancer
resource "azurerm_lb" "internal" {
  name                = "internal-lb"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  sku                 = "Standard"
  
  frontend_ip_configuration {
    name                          = "internal-frontend"
    subnet_id                     = azurerm_subnet.prod_app.id
    private_ip_address_allocation = "Static"
    private_ip_address            = "10.2.2.100"
  }
}

resource "azurerm_lb_backend_address_pool" "app_servers" {
  loadbalancer_id = azurerm_lb.internal.id
  name            = "app-servers-pool"
}

resource "azurerm_lb_probe" "http" {
  loadbalancer_id     = azurerm_lb.internal.id
  name                = "http-probe"
  protocol            = "Http"
  port                = 8080
  request_path        = "/health"
  interval_in_seconds = 5
  number_of_probes    = 2
}

resource "azurerm_lb_rule" "http" {
  loadbalancer_id                = azurerm_lb.internal.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 8080
  frontend_ip_configuration_name = "internal-frontend"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.app_servers.id]
  probe_id                       = azurerm_lb_probe.http.id
  idle_timeout_in_minutes        = 4
  enable_tcp_reset               = true
}
```

### Application Gateway with WAF
```hcl
# Application Gateway v2 with WAF
resource "azurerm_application_gateway" "main" {
  name                = "app-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  
  sku {
    name     = "WAF_v2"
    tier     = "WAF_v2"
    capacity = 2
  }
  
  gateway_ip_configuration {
    name      = "gateway-ip-config"
    subnet_id = azurerm_subnet.appgw.id
  }
  
  frontend_port {
    name = "https-port"
    port = 443
  }
  
  frontend_ip_configuration {
    name                 = "public-frontend"
    public_ip_address_id = azurerm_public_ip.appgw.id
  }
  
  ssl_certificate {
    name                = "wildcard-cert"
    key_vault_secret_id = azurerm_key_vault_certificate.wildcard.secret_id
  }
  
  backend_address_pool {
    name         = "web-backend"
    ip_addresses = ["10.2.1.10", "10.2.1.11", "10.2.1.12"]
  }
  
  backend_http_settings {
    name                  = "http-settings"
    cookie_based_affinity = "Disabled"
    port                  = 443
    protocol              = "Https"
    request_timeout       = 30
    probe_name            = "health-probe"
    
    pick_host_name_from_backend_address = false
    host_name                           = "api.corp.local"
  }
  
  probe {
    name                                      = "health-probe"
    protocol                                  = "Https"
    path                                      = "/health"
    interval                                  = 10
    timeout                                   = 10
    unhealthy_threshold                       = 3
    pick_host_name_from_backend_http_settings = true
    
    match {
      status_code = ["200-399"]
    }
  }
  
  http_listener {
    name                           = "https-listener"
    frontend_ip_configuration_name = "public-frontend"
    frontend_port_name             = "https-port"
    protocol                       = "Https"
    ssl_certificate_name           = "wildcard-cert"
  }
  
  request_routing_rule {
    name                       = "routing-rule"
    rule_type                  = "PathBasedRouting"
    http_listener_name         = "https-listener"
    url_path_map_name          = "path-map"
    priority                   = 100
  }
  
  url_path_map {
    name                               = "path-map"
    default_backend_address_pool_name  = "web-backend"
    default_backend_http_settings_name = "http-settings"
    
    path_rule {
      name                       = "api-path"
      paths                      = ["/api/*"]
      backend_address_pool_name  = "api-backend"
      backend_http_settings_name = "http-settings"
    }
  }
  
  waf_configuration {
    enabled                  = true
    firewall_mode            = "Prevention"
    rule_set_type            = "OWASP"
    rule_set_version         = "3.2"
    file_upload_limit_mb     = 100
    request_body_check       = true
    max_request_body_size_kb = 128
    
    disabled_rule_group {
      rule_group_name = "REQUEST-920-PROTOCOL-ENFORCEMENT"
      rules           = [920350]  # Disable specific rule if needed
    }
    
    exclusion {
      match_variable          = "RequestHeaderNames"
      selector_match_operator = "StartsWith"
      selector                = "x-custom-"
    }
  }
  
  # Enable diagnostics
  tags = {
    Environment = "Production"
  }
}
```

### Traffic Manager
```hcl
# Traffic Manager for Global Load Balancing
resource "azurerm_traffic_manager_profile" "main" {
  name                   = "global-traffic-manager"
  resource_group_name    = azurerm_resource_group.network.name
  traffic_routing_method = "Performance"
  
  dns_config {
    relative_name = "app-global"
    ttl           = 60
  }
  
  monitor_config {
    protocol                     = "HTTPS"
    port                         = 443
    path                         = "/health"
    interval_in_seconds          = 30
    timeout_in_seconds           = 10
    tolerated_number_of_failures = 3
    
    custom_header {
      name  = "host"
      value = "api.corp.local"
    }
  }
}

# Endpoints
resource "azurerm_traffic_manager_azure_endpoint" "eastus" {
  name               = "eastus-endpoint"
  profile_id         = azurerm_traffic_manager_profile.main.id
  target_resource_id = azurerm_public_ip.appgw_eastus.id
  weight             = 100
  priority           = 1
}

resource "azurerm_traffic_manager_azure_endpoint" "westus" {
  name               = "westus-endpoint"
  profile_id         = azurerm_traffic_manager_profile.main.id
  target_resource_id = azurerm_public_ip.appgw_westus.id
  weight             = 100
  priority           = 2
}
```

---

## 5. DNS Management <a name="dns-management"></a>

### Azure DNS Zones
```hcl
# Public DNS Zone
resource "azurerm_dns_zone" "public" {
  name                = "corp.com"
  resource_group_name = azurerm_resource_group.network.name
}

# Private DNS Zone
resource "azurerm_private_dns_zone" "internal" {
  name                = "corp.local"
  resource_group_name = azurerm_resource_group.network.name
}

# Link Private DNS to VNets
resource "azurerm_private_dns_zone_virtual_network_link" "hub" {
  name                  = "hub-link"
  resource_group_name   = azurerm_resource_group.network.name
  private_dns_zone_name = azurerm_private_dns_zone.internal.name
  virtual_network_id    = azurerm_virtual_network.hub.id
  registration_enabled  = true
}

resource "azurerm_private_dns_zone_virtual_network_link" "spoke_prod" {
  name                  = "spoke-prod-link"
  resource_group_name   = azurerm_resource_group.network.name
  private_dns_zone_name = azurerm_private_dns_zone.internal.name
  virtual_network_id    = azurerm_virtual_network.spoke_prod.id
  registration_enabled  = true
}

# DNS Records
resource "azurerm_dns_a_record" "www" {
  name                = "www"
  zone_name           = azurerm_dns_zone.public.name
  resource_group_name = azurerm_resource_group.network.name
  ttl                 = 300
  records             = [azurerm_public_ip.appgw.ip_address]
}

resource "azurerm_dns_cname_record" "api" {
  name                = "api"
  zone_name           = azurerm_dns_zone.public.name
  resource_group_name = azurerm_resource_group.network.name
  ttl                 = 300
  record              = azurerm_traffic_manager_profile.main.fqdn
}

# Private Endpoint DNS Zone Groups
resource "azurerm_private_dns_zone" "sql" {
  name                = "privatelink.database.windows.net"
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_private_dns_zone" "blob" {
  name                = "privatelink.blob.core.windows.net"
  resource_group_name = azurerm_resource_group.network.name
}

resource "azurerm_private_dns_zone" "keyvault" {
  name                = "privatelink.vaultcore.azure.net"
  resource_group_name = azurerm_resource_group.network.name
}
```

### Hybrid DNS Configuration
```yaml
# ansible/playbooks/configure-dns-forwarding.yml
---
- name: Configure DNS Forwarding for Hybrid Environment
  hosts: dns_servers
  become: yes
  vars:
    azure_dns_ip: "10.1.0.4"  # Azure Firewall DNS proxy
    on_prem_domain: "corp.local"
    azure_private_zones:
      - "privatelink.database.windows.net"
      - "privatelink.blob.core.windows.net"
      - "privatelink.vaultcore.azure.net"
      - "privatelink.azurecr.io"
      
  tasks:
    - name: Configure conditional forwarders (Windows DNS)
      win_dns_record:
        name: "{{ item }}"
        type: "forwarder"
        zone: "{{ item }}"
        value: "{{ azure_dns_ip }}"
        state: present
      loop: "{{ azure_private_zones }}"
      when: ansible_os_family == "Windows"
      
    - name: Configure BIND conditional forwarding (Linux)
      blockinfile:
        path: /etc/named.conf
        block: |
          {% for zone in azure_private_zones %}
          zone "{{ zone }}" {
              type forward;
              forward only;
              forwarders { {{ azure_dns_ip }}; };
          };
          {% endfor %}
      when: ansible_os_family == "RedHat"
      notify: restart named
      
    - name: Configure Unbound conditional forwarding
      blockinfile:
        path: /etc/unbound/unbound.conf.d/azure-forward.conf
        block: |
          {% for zone in azure_private_zones %}
          forward-zone:
              name: "{{ zone }}"
              forward-addr: {{ azure_dns_ip }}
          {% endfor %}
        create: yes
      when: ansible_os_family == "Debian"
      notify: restart unbound
      
  handlers:
    - name: restart named
      service:
        name: named
        state: restarted
        
    - name: restart unbound
      service:
        name: unbound
        state: restarted
```

---

## 6. Network Security and Microsegmentation <a name="network-security"></a>

### Zero Trust Network Architecture
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Zero Trust Network Segmentation                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Principle: Never Trust, Always Verify                                      │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    Identity Layer                                    │   │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │   │
│  │   │ Entra ID    │  │ MFA/CAP     │  │ Workload ID │                 │   │
│  │   │ (Users)     │  │ (Policies)  │  │ (Services)  │                 │   │
│  │   └─────────────┘  └─────────────┘  └─────────────┘                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                    │                                         │
│                                    ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    Network Layer                                     │   │
│  │                                                                      │   │
│  │   Microsegmentation Zones:                                           │   │
│  │                                                                      │   │
│  │   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐     │   │
│  │   │ Zone A   │    │ Zone B   │    │ Zone C   │    │ Zone D   │     │   │
│  │   │ (Web)    │───▶│ (API)    │───▶│ (Data)   │    │ (Admin)  │     │   │
│  │   │          │    │          │    │          │    │          │     │   │
│  │   │ NSG:     │    │ NSG:     │    │ NSG:     │    │ NSG:     │     │   │
│  │   │ Allow 443│    │ Allow    │    │ Allow    │    │ Allow    │     │   │
│  │   │ from     │    │ 8080     │    │ 1433     │    │ 3389/22  │     │   │
│  │   │ Internet │    │ from A   │    │ from B   │    │ from     │     │   │
│  │   │          │    │          │    │          │    │ Bastion  │     │   │
│  │   └──────────┘    └──────────┘    └──────────┘    └──────────┘     │   │
│  │                                                                      │   │
│  │   Inter-Zone Traffic: Explicit Allow Only                           │   │
│  │   Default: Deny All                                                  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Application Security Groups
```hcl
# Application Security Groups for logical grouping
resource "azurerm_application_security_group" "web_servers" {
  name                = "asg-web-servers"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_application_security_group" "app_servers" {
  name                = "asg-app-servers"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_application_security_group" "db_servers" {
  name                = "asg-db-servers"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

# NSG Rules using ASGs
resource "azurerm_network_security_group" "microseg" {
  name                = "microsegmentation-nsg"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
}

resource "azurerm_network_security_rule" "web_to_app" {
  name                                       = "Allow-Web-to-App"
  priority                                   = 100
  direction                                  = "Inbound"
  access                                     = "Allow"
  protocol                                   = "Tcp"
  source_port_range                          = "*"
  destination_port_range                     = "8080"
  source_application_security_group_ids      = [azurerm_application_security_group.web_servers.id]
  destination_application_security_group_ids = [azurerm_application_security_group.app_servers.id]
  resource_group_name                        = azurerm_resource_group.network.name
  network_security_group_name                = azurerm_network_security_group.microseg.name
}

resource "azurerm_network_security_rule" "app_to_db" {
  name                                       = "Allow-App-to-DB"
  priority                                   = 200
  direction                                  = "Inbound"
  access                                     = "Allow"
  protocol                                   = "Tcp"
  source_port_range                          = "*"
  destination_port_range                     = "1433"
  source_application_security_group_ids      = [azurerm_application_security_group.app_servers.id]
  destination_application_security_group_ids = [azurerm_application_security_group.db_servers.id]
  resource_group_name                        = azurerm_resource_group.network.name
  network_security_group_name                = azurerm_network_security_group.microseg.name
}
```

### Kubernetes Network Policies
```yaml
# Kubernetes Network Policies for Pod Microsegmentation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-tier-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: web
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
        - podSelector:
            matchLabels:
              app: nginx-ingress
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: app
      ports:
        - protocol: TCP
          port: 8080
    - to:
        - namespaceSelector:
            matchLabels:
              name: kube-system
        - podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-tier-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: app
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: web
      ports:
        - protocol: TCP
          port: 8080
  egress:
    - to:
        - podSelector:
            matchLabels:
              tier: database
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - namespaceSelector:
            matchLabels:
              name: kube-system
      ports:
        - protocol: UDP
          port: 53

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-tier-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      tier: database
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: app
      ports:
        - protocol: TCP
          port: 5432
```

---

## 7. Hybrid Network Connectivity <a name="hybrid-connectivity"></a>

### ExpressRoute Configuration
```hcl
# ExpressRoute Circuit
resource "azurerm_express_route_circuit" "main" {
  name                  = "corp-expressroute"
  resource_group_name   = azurerm_resource_group.network.name
  location              = azurerm_resource_group.network.location
  service_provider_name = "Equinix"
  peering_location      = "Silicon Valley"
  bandwidth_in_mbps     = 1000
  
  sku {
    tier   = "Premium"
    family = "MeteredData"
  }
}

# ExpressRoute Gateway
resource "azurerm_virtual_network_gateway" "expressroute" {
  name                = "expressroute-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  type                = "ExpressRoute"
  sku                 = "ErGw2AZ"
  
  ip_configuration {
    name                          = "gateway-config"
    public_ip_address_id          = azurerm_public_ip.er_gateway.id
    private_ip_address_allocation = "Dynamic"
    subnet_id                     = azurerm_subnet.gateway.id
  }
}

# Connection
resource "azurerm_virtual_network_gateway_connection" "expressroute" {
  name                       = "er-connection"
  resource_group_name        = azurerm_resource_group.network.name
  location                   = azurerm_resource_group.network.location
  type                       = "ExpressRoute"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.expressroute.id
  express_route_circuit_id   = azurerm_express_route_circuit.main.id
  
  routing_weight = 10
}

# Private Peering
resource "azurerm_express_route_circuit_peering" "private" {
  peering_type                  = "AzurePrivatePeering"
  express_route_circuit_name    = azurerm_express_route_circuit.main.name
  resource_group_name           = azurerm_resource_group.network.name
  peer_asn                      = 65001
  primary_peer_address_prefix   = "172.16.100.0/30"
  secondary_peer_address_prefix = "172.16.100.4/30"
  vlan_id                       = 100
  shared_key                    = var.peering_shared_key
}
```

### Site-to-Site VPN
```hcl
# VPN Gateway
resource "azurerm_virtual_network_gateway" "vpn" {
  name                = "vpn-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  type                = "Vpn"
  vpn_type            = "RouteBased"
  sku                 = "VpnGw2AZ"
  active_active       = true
  enable_bgp          = true
  
  bgp_settings {
    asn = 65515
    
    peering_addresses {
      ip_configuration_name = "vnetGatewayConfig1"
      apipa_addresses       = ["169.254.21.1"]
    }
    
    peering_addresses {
      ip_configuration_name = "vnetGatewayConfig2"
      apipa_addresses       = ["169.254.22.1"]
    }
  }
  
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
}

# Local Network Gateway (On-Premises)
resource "azurerm_local_network_gateway" "onprem" {
  name                = "onprem-gateway"
  resource_group_name = azurerm_resource_group.network.name
  location            = azurerm_resource_group.network.location
  gateway_address     = var.onprem_vpn_ip
  address_space       = ["172.16.0.0/12"]
  
  bgp_settings {
    asn                 = 65001
    bgp_peering_address = "172.16.0.1"
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
  shared_key                 = var.vpn_shared_key
  enable_bgp                 = true
  
  ipsec_policy {
    dh_group         = "DHGroup14"
    ike_encryption   = "AES256"
    ike_integrity    = "SHA256"
    ipsec_encryption = "AES256"
    ipsec_integrity  = "SHA256"
    pfs_group        = "PFS2048"
    sa_datasize      = 102400000
    sa_lifetime      = 27000
  }
}
```

---

## 8. Network Automation with Ansible <a name="network-automation"></a>

### Ansible Network Automation
```yaml
# ansible/playbooks/network-provisioning.yml
---
- name: Provision Network Infrastructure
  hosts: localhost
  connection: local
  vars:
    azure_subscription_id: "{{ lookup('env', 'AZURE_SUBSCRIPTION_ID') }}"
    resource_group: "network-rg"
    location: "eastus"
    
  tasks:
    - name: Create Resource Group
      azure.azcollection.azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
        state: present
        
    - name: Create Virtual Network
      azure.azcollection.azure_rm_virtualnetwork:
        resource_group: "{{ resource_group }}"
        name: "prod-vnet"
        address_prefixes:
          - "10.2.0.0/16"
        state: present
        
    - name: Create Subnets
      azure.azcollection.azure_rm_subnet:
        resource_group: "{{ resource_group }}"
        virtual_network_name: "prod-vnet"
        name: "{{ item.name }}"
        address_prefix: "{{ item.prefix }}"
        state: present
      loop:
        - { name: "web-tier", prefix: "10.2.1.0/24" }
        - { name: "app-tier", prefix: "10.2.2.0/24" }
        - { name: "data-tier", prefix: "10.2.3.0/24" }
        
    - name: Create Network Security Groups
      azure.azcollection.azure_rm_securitygroup:
        resource_group: "{{ resource_group }}"
        name: "{{ item.name }}"
        rules:
          - name: "{{ item.rule_name }}"
            protocol: Tcp
            destination_port_range: "{{ item.port }}"
            access: Allow
            priority: 100
            direction: Inbound
            source_address_prefix: "{{ item.source }}"
        state: present
      loop:
        - { name: "web-nsg", rule_name: "Allow-HTTPS", port: "443", source: "Internet" }
        - { name: "app-nsg", rule_name: "Allow-Web", port: "8080", source: "10.2.1.0/24" }
        - { name: "data-nsg", rule_name: "Allow-SQL", port: "1433", source: "10.2.2.0/24" }

---
# ansible/playbooks/cisco-router-config.yml
- name: Configure Cisco Routers
  hosts: cisco_routers
  gather_facts: no
  connection: network_cli
  
  vars:
    azure_networks:
      - "10.1.0.0/16"
      - "10.2.0.0/16"
    vpn_peer_ip: "52.168.100.1"
    
  tasks:
    - name: Configure BGP
      cisco.ios.ios_bgp_global:
        config:
          as_number: 65001
          bgp:
            log_neighbor_changes: true
            router_id: "{{ ansible_host }}"
        state: merged
        
    - name: Configure BGP Neighbors
      cisco.ios.ios_bgp_address_family:
        config:
          as_number: 65001
          address_family:
            - afi: ipv4
              safi: unicast
              neighbors:
                - neighbor_address: "{{ vpn_peer_ip }}"
                  remote_as: 65515
                  activate: true
        state: merged
        
    - name: Configure Static Routes for Azure
      cisco.ios.ios_static_routes:
        config:
          - address_families:
              - afi: ipv4
                routes:
                  - dest: "{{ item }}"
                    next_hops:
                      - interface: Tunnel0
        state: merged
      loop: "{{ azure_networks }}"
      
    - name: Save Configuration
      cisco.ios.ios_config:
        save_when: modified
```

---

## 9. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: Hub-Spoke Network Migration

**Challenge**: Migrate 50 applications from flat network to hub-spoke architecture

**Solution**:
```
Phase 1: Assessment and Planning (4 weeks)
├── Map all application dependencies
├── Document current traffic flows
├── Design IP addressing scheme
├── Create migration waves by application tier

Phase 2: Hub Infrastructure (2 weeks)
├── Deploy hub VNet with Azure Firewall
├── Configure ExpressRoute/VPN connectivity
├── Set up Azure Bastion for secure access
├── Implement centralized DNS

Phase 3: Spoke Deployment (6 weeks)
├── Create spoke VNets per environment
├── Configure VNet peering
├── Implement NSG rules per tier
├── Set up route tables through firewall

Phase 4: Application Migration (8 weeks)
├── Wave 1: Non-production workloads
├── Wave 2: Internal applications
├── Wave 3: External-facing applications
├── Wave 4: Critical business applications

Results:
├── Security posture: Improved (centralized inspection)
├── Visibility: Complete traffic logging
├── Cost: Optimized egress via central firewall
└── Compliance: Achieved network segmentation
```

### Scenario 2: Zero Trust Network Implementation

**Challenge**: Implement zero trust networking for 500-server environment

**Solution**:
```yaml
implementation_phases:
  phase1_identity:
    - Deploy Azure AD for all service accounts
    - Implement managed identities for Azure resources
    - Enable MFA for all administrative access
    - Configure Conditional Access policies
    
  phase2_microsegmentation:
    - Define security zones by application tier
    - Implement ASGs for logical grouping
    - Create NSG rules with explicit allow
    - Deploy Kubernetes Network Policies
    
  phase3_inspection:
    - Deploy Azure Firewall for east-west traffic
    - Enable TLS inspection for encrypted traffic
    - Implement threat intelligence feeds
    - Configure IDS/IPS rules
    
  phase4_monitoring:
    - Deploy NSG Flow Logs
    - Enable Azure Firewall logs
    - Create traffic analytics dashboards
    - Set up alerting for anomalies

results:
  lateral_movement: "Blocked (no default trust)"
  attack_surface: "Reduced 80%"
  visibility: "100% traffic logged"
  compliance: "PCI-DSS network segmentation achieved"
```

### Common Interview Questions

**Q1: "Explain the difference between NSG and Azure Firewall"**

**Answer**:
| Feature | NSG | Azure Firewall |
|---------|-----|----------------|
| Layer | L3/L4 | L3-L7 |
| Scope | Subnet/NIC | VNet/Hub |
| FQDN Filtering | No | Yes |
| Threat Intel | No | Yes |
| TLS Inspection | No | Yes |
| NAT | No | Yes |
| Cost | Free | Pay per hour |

**Q2: "How would you troubleshoot connectivity issues?"**

**Answer**:
1. Check NSG flow logs for denied traffic
2. Verify route tables (UDR)
3. Use Network Watcher IP flow verify
4. Check Azure Firewall logs
5. Verify DNS resolution
6. Test with Connection Monitor
7. Review VNet peering status

---

## Quick Reference

```bash
# Azure CLI - Networking
az network vnet create -g rg -n vnet --address-prefix 10.0.0.0/16
az network nsg rule list -g rg --nsg-name nsg -o table
az network vnet peering list -g rg --vnet-name vnet
az network nic show-effective-route-table -g rg -n nic
az network watcher show-next-hop -g rg --vm myvm --source-ip 10.0.0.4 --dest-ip 10.1.0.4

# Network Watcher
az network watcher test-ip-flow -g rg --vm myvm --direction Inbound \
    --local 10.0.0.4:80 --remote 10.1.0.5:* --protocol TCP

# Firewall
az network firewall show -g rg -n firewall --query "ipConfigurations[0].privateIpAddress"
az network firewall nat-rule collection list -g rg -f firewall

# DNS
az network private-dns zone list -g rg
az network private-dns record-set list -g rg -z privatelink.database.windows.net
```
