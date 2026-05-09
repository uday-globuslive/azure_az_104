# Module 23: Security and Compliance Automation

## Table of Contents
1. [CIS Benchmarks Implementation](#cis-benchmarks)
2. [Configuration Drift Detection and Remediation](#drift-detection)
3. [Vulnerability Management](#vulnerability-management)
4. [Prisma Cloud Integration](#prisma-cloud)
5. [Azure Security Center and Defender](#azure-security)
6. [Certificate Lifecycle Automation](#certificate-automation)
7. [Compliance as Code](#compliance-as-code)
8. [Interview Scenarios](#interview-scenarios)

---

## 1. CIS Benchmarks Implementation <a name="cis-benchmarks"></a>

### CIS Benchmark Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                    CIS Benchmark Framework                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Level 1 (Basic Security)                                       │
│  ├── Password policies                                          │
│  ├── Account lockout settings                                   │
│  ├── Audit policies                                             │
│  ├── User rights assignments                                    │
│  └── Basic firewall rules                                       │
│                                                                  │
│  Level 2 (Defense in Depth)                                     │
│  ├── All Level 1 controls                                       │
│  ├── Advanced audit configuration                               │
│  ├── AppLocker/WDAC policies                                    │
│  ├── Network isolation                                          │
│  └── Credential protection                                      │
│                                                                  │
│  Benchmarks Available:                                          │
│  ├── Windows Server 2019/2022                                   │
│  ├── Red Hat Enterprise Linux 8/9                               │
│  ├── Ubuntu 20.04/22.04                                         │
│  ├── Kubernetes                                                 │
│  ├── Docker                                                     │
│  ├── Azure                                                      │
│  └── Microsoft 365                                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Ansible CIS Hardening Playbook
```yaml
# playbooks/cis-hardening/windows-server-2022.yml
---
- name: Apply CIS Benchmark - Windows Server 2022 Level 1
  hosts: windows_servers
  gather_facts: yes
  vars:
    cis_level: 1
    remediate: true
    report_only: false
    
  tasks:
    # ============================================
    # 1. Account Policies
    # ============================================
    - name: "1.1.1 - Ensure 'Enforce password history' is set to '24 or more'"
      win_security_policy:
        section: System Access
        key: PasswordHistorySize
        value: 24
      tags: [cis, passwords, "1.1.1"]
      
    - name: "1.1.2 - Ensure 'Maximum password age' is set to '365 or fewer days'"
      win_security_policy:
        section: System Access
        key: MaximumPasswordAge
        value: 60
      tags: [cis, passwords, "1.1.2"]
      
    - name: "1.1.3 - Ensure 'Minimum password age' is set to '1 or more'"
      win_security_policy:
        section: System Access
        key: MinimumPasswordAge
        value: 1
      tags: [cis, passwords, "1.1.3"]
      
    - name: "1.1.4 - Ensure 'Minimum password length' is set to '14 or more'"
      win_security_policy:
        section: System Access
        key: MinimumPasswordLength
        value: 14
      tags: [cis, passwords, "1.1.4"]
      
    - name: "1.1.5 - Ensure 'Password must meet complexity requirements' is Enabled"
      win_security_policy:
        section: System Access
        key: PasswordComplexity
        value: 1
      tags: [cis, passwords, "1.1.5"]
      
    # ============================================
    # 1.2 Account Lockout Policy
    # ============================================
    - name: "1.2.1 - Ensure 'Account lockout duration' is set to '15 or more minutes'"
      win_security_policy:
        section: System Access
        key: LockoutDuration
        value: 15
      tags: [cis, lockout, "1.2.1"]
      
    - name: "1.2.2 - Ensure 'Account lockout threshold' is set to '5 or fewer'"
      win_security_policy:
        section: System Access
        key: LockoutBadCount
        value: 5
      tags: [cis, lockout, "1.2.2"]
      
    # ============================================
    # 2. Local Policies - User Rights Assignment
    # ============================================
    - name: "2.2.1 - Ensure 'Access Credential Manager as a trusted caller' is set to 'No One'"
      win_user_right:
        name: SeTrustedCredManAccessPrivilege
        users: []
      tags: [cis, user_rights, "2.2.1"]
      
    - name: "2.2.3 - Ensure 'Access this computer from the network' is set to required users"
      win_user_right:
        name: SeNetworkLogonRight
        users:
          - Administrators
          - Authenticated Users
      tags: [cis, user_rights, "2.2.3"]
      
    - name: "2.2.26 - Ensure 'Deny log on locally' includes 'Guests'"
      win_user_right:
        name: SeDenyInteractiveLogonRight
        users:
          - Guests
      tags: [cis, user_rights, "2.2.26"]
      
    # ============================================
    # 5. System Services
    # ============================================
    - name: "5.x - Disable unnecessary services"
      win_service:
        name: "{{ item }}"
        start_mode: disabled
        state: stopped
      loop:
        - Browser           # Computer Browser
        - IISADMIN          # IIS Admin Service (if not needed)
        - irmon             # Infrared Monitor
        - SharedAccess      # Internet Connection Sharing
        - LxssManager       # Windows Subsystem for Linux
        - FTPSVC            # FTP Publishing Service
        - RpcLocator        # Remote Procedure Call Locator
        - RemoteAccess      # Routing and Remote Access
        - SNMPTRAP          # SNMP Trap
        - SSDPSRV           # SSDP Discovery
        - upnphost          # UPnP Device Host
        - WMSvc             # Web Management Service
        - XboxGipSvc        # Xbox Accessory Management
      ignore_errors: yes
      tags: [cis, services]
      
    # ============================================
    # 9. Windows Firewall
    # ============================================
    - name: "9.1.1 - Ensure Windows Firewall: Domain: Firewall state is On"
      win_firewall:
        state: enabled
        profiles:
          - Domain
      tags: [cis, firewall, "9.1.1"]
      
    - name: "9.1.2 - Ensure Windows Firewall: Domain: Inbound connections is Block"
      win_firewall:
        profiles:
          - Domain
      win_shell: |
        Set-NetFirewallProfile -Profile Domain -DefaultInboundAction Block
      tags: [cis, firewall, "9.1.2"]
      
    # ============================================
    # 17. Advanced Audit Policy Configuration
    # ============================================
    - name: "17.1.1 - Ensure 'Audit Credential Validation' is set to 'Success and Failure'"
      win_audit_policy_system:
        subcategory: Credential Validation
        audit_type: success, failure
      tags: [cis, audit, "17.1.1"]
      
    - name: "17.2.1 - Ensure 'Audit Application Group Management' is set to 'Success and Failure'"
      win_audit_policy_system:
        subcategory: Application Group Management
        audit_type: success, failure
      tags: [cis, audit, "17.2.1"]
      
    # ============================================
    # 18. Administrative Templates
    # ============================================
    - name: "18.1.1.1 - Ensure 'Prevent enabling lock screen camera' is Enabled"
      win_regedit:
        path: HKLM:\SOFTWARE\Policies\Microsoft\Windows\Personalization
        name: NoLockScreenCamera
        data: 1
        type: dword
      tags: [cis, registry, "18.1.1.1"]
      
    - name: "18.3.1 - Ensure 'Configure SMB v1 client driver' is Disabled"
      win_regedit:
        path: HKLM:\SYSTEM\CurrentControlSet\Services\mrxsmb10
        name: Start
        data: 4
        type: dword
      tags: [cis, registry, "18.3.1"]
      
    - name: "18.4.4 - Ensure 'MSS: (DisableIPSourceRouting) IP source routing protection level'"
      win_regedit:
        path: HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters
        name: DisableIPSourceRouting
        data: 2
        type: dword
      tags: [cis, registry, "18.4.4"]
      
  handlers:
    - name: Reboot if required
      win_reboot:
        msg: "Rebooting for CIS hardening changes"
        reboot_timeout: 600
```

### Linux CIS Hardening
```yaml
# playbooks/cis-hardening/rhel8.yml
---
- name: Apply CIS Benchmark - RHEL 8 Level 1
  hosts: linux_servers
  become: yes
  vars:
    cis_level: 1
    
  tasks:
    # ============================================
    # 1.1 Filesystem Configuration
    # ============================================
    - name: "1.1.1.1 - Ensure mounting of cramfs filesystems is disabled"
      copy:
        dest: /etc/modprobe.d/cramfs.conf
        content: |
          install cramfs /bin/true
          blacklist cramfs
        mode: '0644'
      tags: [cis, filesystems, "1.1.1.1"]
      
    - name: "1.1.1.2 - Ensure mounting of squashfs filesystems is disabled"
      copy:
        dest: /etc/modprobe.d/squashfs.conf
        content: |
          install squashfs /bin/true
          blacklist squashfs
        mode: '0644'
      tags: [cis, filesystems, "1.1.1.2"]
      
    - name: "1.1.2 - Ensure /tmp is configured"
      mount:
        path: /tmp
        src: tmpfs
        fstype: tmpfs
        opts: "mode=1777,strictatime,noexec,nodev,nosuid"
        state: mounted
      tags: [cis, filesystems, "1.1.2"]
      
    # ============================================
    # 1.4 Secure Boot Settings
    # ============================================
    - name: "1.4.1 - Ensure permissions on bootloader config are configured"
      file:
        path: /boot/grub2/grub.cfg
        owner: root
        group: root
        mode: '0600'
      tags: [cis, boot, "1.4.1"]
      
    # ============================================
    # 3.1 Network Parameters
    # ============================================
    - name: "3.1.1 - Ensure IP forwarding is disabled"
      sysctl:
        name: net.ipv4.ip_forward
        value: '0'
        state: present
        sysctl_file: /etc/sysctl.d/99-cis.conf
        reload: yes
      tags: [cis, network, "3.1.1"]
      
    - name: "3.1.2 - Ensure packet redirect sending is disabled"
      sysctl:
        name: "{{ item }}"
        value: '0'
        state: present
        sysctl_file: /etc/sysctl.d/99-cis.conf
        reload: yes
      loop:
        - net.ipv4.conf.all.send_redirects
        - net.ipv4.conf.default.send_redirects
      tags: [cis, network, "3.1.2"]
      
    # ============================================
    # 4.2 Configure Logging
    # ============================================
    - name: "4.2.1.1 - Ensure rsyslog is installed"
      package:
        name: rsyslog
        state: present
      tags: [cis, logging, "4.2.1.1"]
      
    - name: "4.2.1.2 - Ensure rsyslog Service is enabled"
      service:
        name: rsyslog
        enabled: yes
        state: started
      tags: [cis, logging, "4.2.1.2"]
      
    # ============================================
    # 5.2 Configure SSH Server
    # ============================================
    - name: "5.2.4 - Ensure SSH access is limited"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^AllowGroups'
        line: 'AllowGroups sshusers'
      notify: restart sshd
      tags: [cis, ssh, "5.2.4"]
      
    - name: "5.2.5 - Ensure SSH LogLevel is appropriate"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^LogLevel'
        line: 'LogLevel INFO'
      notify: restart sshd
      tags: [cis, ssh, "5.2.5"]
      
    - name: "5.2.11 - Ensure SSH PermitRootLogin is disabled"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: 'PermitRootLogin no'
      notify: restart sshd
      tags: [cis, ssh, "5.2.11"]
      
    - name: "5.2.12 - Ensure SSH PermitEmptyPasswords is disabled"
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PermitEmptyPasswords'
        line: 'PermitEmptyPasswords no'
      notify: restart sshd
      tags: [cis, ssh, "5.2.12"]
      
    # ============================================
    # 5.4 User Accounts and Environment
    # ============================================
    - name: "5.4.1.1 - Ensure password expiration is 365 days or less"
      lineinfile:
        path: /etc/login.defs
        regexp: '^PASS_MAX_DAYS'
        line: 'PASS_MAX_DAYS   90'
      tags: [cis, passwords, "5.4.1.1"]
      
    - name: "5.4.1.2 - Ensure minimum days between password changes is configured"
      lineinfile:
        path: /etc/login.defs
        regexp: '^PASS_MIN_DAYS'
        line: 'PASS_MIN_DAYS   1'
      tags: [cis, passwords, "5.4.1.2"]
      
  handlers:
    - name: restart sshd
      service:
        name: sshd
        state: restarted
```

---

## 2. Configuration Drift Detection and Remediation <a name="drift-detection"></a>

### Drift Detection Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                  Drift Detection Pipeline                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐          │
│  │  Baseline   │───▶│  Scanner    │───▶│   Compare   │          │
│  │  (Desired   │    │ (Ansible/   │    │   Engine    │          │
│  │   State)    │    │  InSpec)    │    │             │          │
│  └─────────────┘    └─────────────┘    └──────┬──────┘          │
│                                               │                  │
│                                               ▼                  │
│                                        ┌─────────────┐          │
│                     ┌─────────────────▶│   Report    │          │
│                     │                  │  Generator  │          │
│                     │                  └─────────────┘          │
│                     │                                            │
│              ┌──────┴──────┐                                    │
│              │   Drift     │                                    │
│              │  Detected?  │                                    │
│              └──────┬──────┘                                    │
│                     │                                            │
│         ┌───────────┴───────────┐                               │
│         ▼                       ▼                                │
│  ┌─────────────┐         ┌─────────────┐                        │
│  │ Auto-       │         │   Manual    │                        │
│  │ Remediate   │         │   Review    │                        │
│  │ (Ansible)   │         │   (Ticket)  │                        │
│  └─────────────┘         └─────────────┘                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### InSpec Compliance Profile
```ruby
# profiles/windows-baseline/controls/security.rb

control 'cis-1.1.1' do
  impact 1.0
  title 'Ensure password history is configured'
  desc 'Check that password history is set to 24 or more'
  
  describe security_policy do
    its('PasswordHistorySize') { should be >= 24 }
  end
end

control 'cis-1.1.4' do
  impact 1.0
  title 'Ensure minimum password length'
  desc 'Minimum password length should be 14 or more'
  
  describe security_policy do
    its('MinimumPasswordLength') { should be >= 14 }
  end
end

control 'cis-9.1.1' do
  impact 1.0
  title 'Ensure Windows Firewall is enabled'
  desc 'Windows Firewall should be enabled for all profiles'
  
  describe command('netsh advfirewall show allprofiles state') do
    its('stdout') { should match /State\s+ON/ }
  end
end

control 'cis-18.3.1' do
  impact 0.7
  title 'Ensure SMBv1 is disabled'
  desc 'SMBv1 protocol should be disabled'
  
  describe registry_key('HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters') do
    its('SMB1') { should eq 0 }
  end
end

control 'windows-services' do
  impact 0.5
  title 'Check unnecessary services are disabled'
  
  %w[Browser RemoteAccess SSDPSRV].each do |svc|
    describe service(svc) do
      it { should_not be_enabled }
      it { should_not be_running }
    end
  end
end
```

### Automated Drift Remediation
```python
# drift_remediation.py
import subprocess
import json
import logging
from datetime import datetime
from typing import Dict, List
from dataclasses import dataclass, asdict

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@dataclass
class DriftFinding:
    control_id: str
    severity: str
    description: str
    current_state: str
    expected_state: str
    remediation_playbook: str
    auto_remediate: bool

class DriftRemediationEngine:
    def __init__(self, config_path: str):
        with open(config_path) as f:
            self.config = json.load(f)
        
        self.auto_remediation_threshold = self.config.get('auto_remediation_severity', 'medium')
        self.remediation_playbooks = self.config.get('playbooks', {})
    
    def scan_for_drift(self, target_hosts: List[str]) -> Dict[str, List[DriftFinding]]:
        """Run InSpec scan and parse results"""
        
        results = {}
        for host in target_hosts:
            # Run InSpec
            cmd = [
                'inspec', 'exec', 
                'profiles/windows-baseline',
                '-t', f'winrm://{host}',
                '--reporter', 'json'
            ]
            
            process = subprocess.run(cmd, capture_output=True, text=True)
            scan_results = json.loads(process.stdout)
            
            findings = []
            for control in scan_results.get('controls', []):
                if control['status'] == 'failed':
                    findings.append(DriftFinding(
                        control_id=control['id'],
                        severity=control['impact'],
                        description=control['title'],
                        current_state=str(control.get('actual', 'unknown')),
                        expected_state=str(control.get('expected', 'compliant')),
                        remediation_playbook=self.remediation_playbooks.get(control['id']),
                        auto_remediate=self._should_auto_remediate(control['impact'])
                    ))
            
            results[host] = findings
        
        return results
    
    def remediate_drift(self, findings: Dict[str, List[DriftFinding]]) -> Dict:
        """Execute remediation playbooks for drift findings"""
        
        remediation_results = {
            'timestamp': datetime.utcnow().isoformat(),
            'hosts': {}
        }
        
        for host, host_findings in findings.items():
            host_results = []
            
            for finding in host_findings:
                if finding.auto_remediate and finding.remediation_playbook:
                    result = self._run_remediation(host, finding)
                    host_results.append({
                        'control': finding.control_id,
                        'status': result['status'],
                        'message': result['message']
                    })
                else:
                    # Create ticket for manual review
                    ticket_id = self._create_ticket(host, finding)
                    host_results.append({
                        'control': finding.control_id,
                        'status': 'manual_review',
                        'ticket_id': ticket_id
                    })
            
            remediation_results['hosts'][host] = host_results
        
        return remediation_results
    
    def _run_remediation(self, host: str, finding: DriftFinding) -> Dict:
        """Execute Ansible playbook for remediation"""
        
        cmd = [
            'ansible-playbook',
            '-i', f'{host},',
            finding.remediation_playbook,
            '--tags', finding.control_id,
            '-e', f'target_host={host}'
        ]
        
        try:
            process = subprocess.run(cmd, capture_output=True, text=True, timeout=300)
            
            if process.returncode == 0:
                return {'status': 'remediated', 'message': 'Successfully remediated'}
            else:
                return {'status': 'failed', 'message': process.stderr}
        
        except subprocess.TimeoutExpired:
            return {'status': 'timeout', 'message': 'Remediation timed out'}
    
    def _should_auto_remediate(self, severity: float) -> bool:
        """Determine if finding should be auto-remediated based on severity"""
        thresholds = {'low': 0.3, 'medium': 0.5, 'high': 0.7, 'critical': 1.0}
        return severity <= thresholds.get(self.auto_remediation_threshold, 0.5)
    
    def _create_ticket(self, host: str, finding: DriftFinding) -> str:
        """Create ServiceNow ticket for manual remediation"""
        # Implementation would integrate with ServiceNow API
        logger.info(f"Creating ticket for {finding.control_id} on {host}")
        return f"INC{datetime.now().strftime('%Y%m%d%H%M%S')}"
```

---

## 3. Vulnerability Management <a name="vulnerability-management"></a>

### Vulnerability Scanning Pipeline
```yaml
# .github/workflows/vulnerability-scan.yml
name: Vulnerability Scanning Pipeline

on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM
  workflow_dispatch:

jobs:
  container-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'image'
          image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE }}:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
          
      - name: Fail on critical vulnerabilities
        run: |
          CRITICAL_COUNT=$(cat trivy-results.sarif | jq '[.runs[].results[] | select(.level == "error")] | length')
          if [ "$CRITICAL_COUNT" -gt 0 ]; then
            echo "Found $CRITICAL_COUNT critical vulnerabilities"
            exit 1
          fi

  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
          
      - name: Run OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'web-application'
          path: '.'
          format: 'HTML'
          
  infrastructure-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
          output_format: sarif
          
      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          soft_fail: false
```

### Vulnerability Remediation Automation
```python
# vulnerability_remediation.py
from dataclasses import dataclass
from typing import List, Optional
import requests
import json

@dataclass
class Vulnerability:
    cve_id: str
    severity: str
    cvss_score: float
    package: str
    current_version: str
    fixed_version: Optional[str]
    affected_systems: List[str]

class VulnerabilityRemediationService:
    def __init__(self, config: dict):
        self.config = config
        self.nvd_api = "https://services.nvd.nist.gov/rest/json/cves/2.0"
        
    def get_vulnerability_details(self, cve_id: str) -> dict:
        """Fetch CVE details from NVD"""
        response = requests.get(f"{self.nvd_api}?cveId={cve_id}")
        return response.json()
    
    def generate_remediation_plan(self, vulnerabilities: List[Vulnerability]) -> dict:
        """Generate prioritized remediation plan"""
        
        # Sort by CVSS score (critical first)
        sorted_vulns = sorted(vulnerabilities, key=lambda x: x.cvss_score, reverse=True)
        
        plan = {
            "critical": [],  # CVSS >= 9.0
            "high": [],      # CVSS >= 7.0
            "medium": [],    # CVSS >= 4.0
            "low": []        # CVSS < 4.0
        }
        
        for vuln in sorted_vulns:
            remediation = {
                "cve": vuln.cve_id,
                "package": vuln.package,
                "current_version": vuln.current_version,
                "target_version": vuln.fixed_version,
                "affected_systems": vuln.affected_systems,
                "remediation_steps": self._get_remediation_steps(vuln)
            }
            
            if vuln.cvss_score >= 9.0:
                plan["critical"].append(remediation)
            elif vuln.cvss_score >= 7.0:
                plan["high"].append(remediation)
            elif vuln.cvss_score >= 4.0:
                plan["medium"].append(remediation)
            else:
                plan["low"].append(remediation)
        
        return plan
    
    def _get_remediation_steps(self, vuln: Vulnerability) -> List[str]:
        """Generate remediation steps based on package type"""
        
        if vuln.fixed_version:
            return [
                f"Update {vuln.package} from {vuln.current_version} to {vuln.fixed_version}",
                "Run automated tests after update",
                "Deploy to staging environment",
                "Verify fix with vulnerability scan",
                "Deploy to production"
            ]
        else:
            return [
                f"No fix available for {vuln.cve_id}",
                "Evaluate workarounds or mitigating controls",
                "Consider removing or replacing affected component",
                "Document risk acceptance if no remediation possible"
            ]
    
    def execute_remediation(self, package: str, version: str, hosts: List[str]) -> dict:
        """Execute package update across affected systems"""
        
        # Generate Ansible playbook
        playbook = self._generate_update_playbook(package, version)
        
        # Execute via Ansible
        results = self._run_ansible(playbook, hosts)
        
        return results
```

---

## 4. Prisma Cloud Integration <a name="prisma-cloud"></a>

### Prisma Cloud Configuration
```yaml
# prisma-cloud-config.yaml
api:
  url: https://api.prismacloud.io
  access_key: ${PRISMA_ACCESS_KEY}
  secret_key: ${PRISMA_SECRET_KEY}

policies:
  # Runtime protection
  runtime:
    - name: "Block cryptomining"
      type: container
      action: block
      rule: |
        proc.cmdline contains "minerd" or 
        proc.cmdline contains "xmrig" or
        network.outbound.port in (3333, 4444, 5555)
    
    - name: "Detect container escape"
      type: container
      action: alert
      rule: |
        container.privileged = true and
        proc.name in ("nsenter", "unshare")
  
  # Vulnerability management
  vulnerability:
    - name: "Critical CVE blocking"
      severity: critical
      action: block
      grace_period_days: 0
    
    - name: "High CVE alerting"
      severity: high
      action: alert
      grace_period_days: 7
  
  # Compliance
  compliance:
    - framework: CIS
      version: "1.5.0"
      target: kubernetes
      action: alert
    
    - framework: PCI-DSS
      version: "3.2.1"
      action: alert

# Defender deployment
defenders:
  orchestrator: kubernetes
  namespace: twistlock
  console_address: https://console.prismacloud.io
  
  daemonset:
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
```

### Prisma API Integration
```python
# prisma_integration.py
import requests
from typing import Dict, List, Optional
import json

class PrismaCloudClient:
    def __init__(self, api_url: str, access_key: str, secret_key: str):
        self.api_url = api_url
        self.access_key = access_key
        self.secret_key = secret_key
        self.token = None
        
    def authenticate(self):
        """Authenticate and get JWT token"""
        response = requests.post(
            f"{self.api_url}/login",
            json={
                "username": self.access_key,
                "password": self.secret_key
            }
        )
        self.token = response.json()["token"]
        
    def _headers(self):
        return {
            "Authorization": f"Bearer {self.token}",
            "Content-Type": "application/json"
        }
    
    def get_vulnerabilities(self, severity: Optional[str] = None) -> List[Dict]:
        """Get container vulnerabilities"""
        params = {}
        if severity:
            params["severity"] = severity
            
        response = requests.get(
            f"{self.api_url}/api/v1/vulnerabilities",
            headers=self._headers(),
            params=params
        )
        return response.json()
    
    def get_compliance_status(self, framework: str = "CIS") -> Dict:
        """Get compliance posture"""
        response = requests.get(
            f"{self.api_url}/api/v1/compliance",
            headers=self._headers(),
            params={"framework": framework}
        )
        return response.json()
    
    def get_runtime_events(self, hours: int = 24) -> List[Dict]:
        """Get runtime security events"""
        response = requests.get(
            f"{self.api_url}/api/v1/audits/runtime/container",
            headers=self._headers(),
            params={"hours": hours}
        )
        return response.json()
    
    def create_custom_rule(self, rule: Dict) -> Dict:
        """Create custom runtime rule"""
        response = requests.post(
            f"{self.api_url}/api/v1/policies/runtime/container",
            headers=self._headers(),
            json=rule
        )
        return response.json()
    
    def get_drift_analysis(self) -> Dict:
        """Get configuration drift analysis"""
        response = requests.get(
            f"{self.api_url}/api/v1/coderepos/drift",
            headers=self._headers()
        )
        return response.json()

# Usage example
def main():
    client = PrismaCloudClient(
        api_url="https://api.prismacloud.io",
        access_key=os.environ["PRISMA_ACCESS_KEY"],
        secret_key=os.environ["PRISMA_SECRET_KEY"]
    )
    
    client.authenticate()
    
    # Get critical vulnerabilities
    critical_vulns = client.get_vulnerabilities(severity="critical")
    
    # Get compliance status
    compliance = client.get_compliance_status(framework="CIS")
    
    # Generate report
    report = {
        "timestamp": datetime.utcnow().isoformat(),
        "critical_vulnerabilities": len(critical_vulns),
        "compliance_score": compliance.get("score", 0),
        "drift_detected": len(client.get_drift_analysis().get("drifts", []))
    }
    
    print(json.dumps(report, indent=2))
```

---

## 5. Azure Security Center and Defender <a name="azure-security"></a>

### Microsoft Defender for Cloud Configuration
```hcl
# Defender for Cloud Terraform
resource "azurerm_security_center_subscription_pricing" "defender" {
  for_each = toset([
    "VirtualMachines",
    "SqlServers",
    "AppServices",
    "StorageAccounts",
    "KeyVaults",
    "Arm",
    "Dns",
    "Containers",
    "KubernetesService"
  ])
  
  tier          = "Standard"
  resource_type = each.value
}

resource "azurerm_security_center_auto_provisioning" "log_analytics" {
  auto_provision = "On"
}

resource "azurerm_security_center_workspace" "main" {
  scope        = "/subscriptions/${data.azurerm_subscription.current.subscription_id}"
  workspace_id = azurerm_log_analytics_workspace.security.id
}

# Custom security policies
resource "azurerm_subscription_policy_assignment" "cis_benchmark" {
  name                 = "cis-azure-benchmark"
  subscription_id      = data.azurerm_subscription.current.id
  policy_definition_id = "/providers/Microsoft.Authorization/policySetDefinitions/1a5bb27d-173f-493e-9568-eb56638dde4d"
  description          = "CIS Microsoft Azure Foundations Benchmark"
  display_name         = "CIS Azure Benchmark"
  
  parameters = jsonencode({
    effect = {
      value = "AuditIfNotExists"
    }
  })
}

# Security contacts
resource "azurerm_security_center_contact" "main" {
  email               = "security-team@corp.local"
  phone               = "+1-555-0100"
  alert_notifications = true
  alerts_to_admins    = true
}

# Workflow automation for alerts
resource "azurerm_logic_app_workflow" "security_alert" {
  name                = "security-alert-workflow"
  resource_group_name = azurerm_resource_group.security.name
  location            = azurerm_resource_group.security.location
  
  workflow_parameters = {
    "$connections" = jsonencode({
      defaultValue = {}
      type         = "Object"
    })
  }
}

resource "azurerm_security_center_automation" "alert_automation" {
  name                = "export-to-logic-app"
  resource_group_name = azurerm_resource_group.security.name
  location            = azurerm_resource_group.security.location
  
  scopes = [data.azurerm_subscription.current.id]
  
  source {
    event_source = "Alerts"
    rule_set {
      rule {
        property_path  = "Severity"
        operator       = "Equals"
        expected_value = "High"
        property_type  = "String"
      }
    }
  }
  
  action {
    type              = "LogicApp"
    resource_id       = azurerm_logic_app_workflow.security_alert.id
    trigger_url       = "https://..."
  }
}
```

---

## 6. Certificate Lifecycle Automation <a name="certificate-automation"></a>

### Certificate Management System
```python
# certificate_manager.py
from cryptography import x509
from cryptography.x509.oid import NameOID
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization
from datetime import datetime, timedelta
import hvac  # HashiCorp Vault client
from typing import Dict, List, Optional
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class CertificateManager:
    def __init__(self, vault_url: str, vault_token: str):
        self.vault_client = hvac.Client(url=vault_url, token=vault_token)
        
    def issue_certificate(
        self,
        common_name: str,
        san: List[str],
        validity_days: int = 365,
        key_size: int = 2048
    ) -> Dict:
        """Issue a new certificate from Vault PKI"""
        
        response = self.vault_client.secrets.pki.generate_certificate(
            name="web-server-role",
            common_name=common_name,
            alt_names=",".join(san),
            ttl=f"{validity_days * 24}h",
            mount_point="pki"
        )
        
        return {
            "certificate": response["data"]["certificate"],
            "private_key": response["data"]["private_key"],
            "ca_chain": response["data"]["ca_chain"],
            "serial_number": response["data"]["serial_number"],
            "expiration": response["data"]["expiration"]
        }
    
    def check_expiring_certificates(
        self,
        warning_days: int = 30
    ) -> List[Dict]:
        """Scan for certificates expiring soon"""
        
        expiring_certs = []
        
        # Get all certificates from Vault
        certs = self.vault_client.secrets.pki.list_certificates(
            mount_point="pki"
        )
        
        for serial in certs["data"]["keys"]:
            cert_data = self.vault_client.secrets.pki.read_certificate(
                serial=serial,
                mount_point="pki"
            )
            
            cert = x509.load_pem_x509_certificate(
                cert_data["data"]["certificate"].encode(),
                default_backend()
            )
            
            days_to_expiry = (cert.not_valid_after - datetime.utcnow()).days
            
            if days_to_expiry <= warning_days:
                expiring_certs.append({
                    "serial": serial,
                    "common_name": cert.subject.get_attributes_for_oid(NameOID.COMMON_NAME)[0].value,
                    "expiry_date": cert.not_valid_after.isoformat(),
                    "days_remaining": days_to_expiry
                })
        
        return expiring_certs
    
    def auto_renew_certificate(
        self,
        serial: str,
        validity_days: int = 365
    ) -> Dict:
        """Automatically renew a certificate"""
        
        # Get existing certificate
        old_cert_data = self.vault_client.secrets.pki.read_certificate(
            serial=serial,
            mount_point="pki"
        )
        
        old_cert = x509.load_pem_x509_certificate(
            old_cert_data["data"]["certificate"].encode(),
            default_backend()
        )
        
        # Extract details from old certificate
        common_name = old_cert.subject.get_attributes_for_oid(NameOID.COMMON_NAME)[0].value
        
        try:
            san_extension = old_cert.extensions.get_extension_for_oid(
                x509.oid.ExtensionOID.SUBJECT_ALTERNATIVE_NAME
            )
            san = [name.value for name in san_extension.value]
        except x509.ExtensionNotFound:
            san = []
        
        # Issue new certificate
        new_cert = self.issue_certificate(
            common_name=common_name,
            san=san,
            validity_days=validity_days
        )
        
        # Revoke old certificate
        self.vault_client.secrets.pki.revoke_certificate(
            serial_number=serial,
            mount_point="pki"
        )
        
        logger.info(f"Renewed certificate for {common_name}")
        
        return new_cert
    
    def deploy_certificate(
        self,
        certificate: Dict,
        targets: List[str],
        deployment_type: str = "ansible"
    ) -> Dict:
        """Deploy certificate to target systems"""
        
        if deployment_type == "ansible":
            # Generate Ansible playbook
            playbook = self._generate_deployment_playbook(certificate, targets)
            result = self._run_ansible(playbook)
        elif deployment_type == "kubernetes":
            result = self._deploy_to_kubernetes(certificate, targets)
        else:
            raise ValueError(f"Unknown deployment type: {deployment_type}")
        
        return result
```

---

## 7. Compliance as Code <a name="compliance-as-code"></a>

### OPA/Gatekeeper Policies
```rego
# policies/kubernetes/pod-security.rego
package kubernetes.admission

# Deny privileged containers
deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    container.securityContext.privileged
    msg := sprintf("Privileged containers are not allowed: %v", [container.name])
}

# Require resource limits
deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    not container.resources.limits.memory
    msg := sprintf("Container must have memory limits: %v", [container.name])
}

# Require specific labels
deny[msg] {
    input.request.kind.kind == "Pod"
    not input.request.object.metadata.labels["app.kubernetes.io/name"]
    msg := "Pod must have 'app.kubernetes.io/name' label"
}

# Deny latest tag
deny[msg] {
    input.request.kind.kind == "Pod"
    container := input.request.object.spec.containers[_]
    endswith(container.image, ":latest")
    msg := sprintf("Image with :latest tag not allowed: %v", [container.image])
}
```

### Azure Policy as Code
```hcl
# Azure Policy definitions
resource "azurerm_policy_definition" "require_tags" {
  name         = "require-cost-center-tag"
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = "Require Cost Center tag on resources"
  
  metadata = jsonencode({
    category = "Tags"
  })
  
  policy_rule = jsonencode({
    if = {
      allOf = [
        {
          field  = "type"
          equals = "Microsoft.Compute/virtualMachines"
        },
        {
          field  = "tags['CostCenter']"
          exists = "false"
        }
      ]
    }
    then = {
      effect = "deny"
    }
  })
}

resource "azurerm_policy_definition" "allowed_vm_sizes" {
  name         = "allowed-vm-sizes"
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = "Allowed VM sizes"
  
  parameters = jsonencode({
    allowedSizes = {
      type = "Array"
      metadata = {
        displayName = "Allowed VM sizes"
        description = "The list of allowed VM sizes"
      }
    }
  })
  
  policy_rule = jsonencode({
    if = {
      allOf = [
        {
          field  = "type"
          equals = "Microsoft.Compute/virtualMachines"
        },
        {
          not = {
            field = "Microsoft.Compute/virtualMachines/sku.name"
            in    = "[parameters('allowedSizes')]"
          }
        }
      ]
    }
    then = {
      effect = "deny"
    }
  })
}
```

---

## 8. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: Enterprise CIS Compliance Implementation

**Challenge**: Achieve 95%+ CIS compliance across 2,000 Windows servers

**Solution**:
```
Phase 1: Assessment (2 weeks)
├── Deploy InSpec to scan all servers
├── Baseline current compliance: 45%
├── Identify high-impact controls
└── Prioritize remediation by risk

Phase 2: Remediation (8 weeks)
├── Create Ansible hardening playbooks
├── Test in dev/staging environments
├── Rolling deployment by server tier
├── Automated re-scanning after each wave

Phase 3: Continuous Compliance (Ongoing)
├── Scheduled daily scans
├── Auto-remediation for drift
├── Dashboard and reporting
└── Integration with ServiceNow

Results:
├── Compliance: 45% → 98%
├── Audit findings: 2,400 → 45
├── Mean time to compliance: 4 hours
└── Drift incidents: 50/month → 2/month
```

### Common Interview Questions

**Q1: "How do you handle zero-day vulnerabilities?"**

**Answer**:
1. **Detection**: Subscribe to threat intelligence feeds
2. **Assessment**: Rapid scan for affected systems
3. **Containment**: Network isolation if critical
4. **Mitigation**: Apply workarounds before patches
5. **Remediation**: Emergency patch deployment
6. **Verification**: Re-scan and validate

**Q2: "Explain your approach to configuration drift"**

**Answer**:
1. Define desired state as code
2. Continuous scanning (InSpec/Ansible)
3. Real-time alerts on drift detection
4. Auto-remediation for approved changes
5. Manual review for unknown changes
6. Root cause analysis and prevention

---

## Quick Reference

```bash
# InSpec
inspec exec profile/ -t winrm://server
inspec exec profile/ -t ssh://server --reporter json

# Ansible hardening
ansible-playbook cis-hardening.yml --check
ansible-playbook cis-hardening.yml --tags "passwords,services"

# Trivy
trivy image myimage:latest
trivy fs --security-checks vuln,config ./

# Azure Security
az security assessment list
az security secure-score list
az security alert list

# Prisma Cloud
twistcli images scan myimage:latest
```
