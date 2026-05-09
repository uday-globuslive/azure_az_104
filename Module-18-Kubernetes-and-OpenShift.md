# Module 18: Kubernetes and OpenShift Platform Engineering

## Table of Contents
1. [Kubernetes Architecture and Core Concepts](#kubernetes-architecture)
2. [OpenShift Enterprise Features](#openshift-features)
3. [OpenShift Virtualization](#openshift-virtualization)
4. [Multi-Cluster Management](#multi-cluster)
5. [OpenShift on Azure (ARO)](#aro)
6. [Operators and Custom Controllers](#operators)
7. [Production Best Practices](#best-practices)
8. [Interview Scenarios and Use Cases](#interview-scenarios)

---

## 1. Kubernetes Architecture and Core Concepts <a name="kubernetes-architecture"></a>

### Kubernetes Architecture Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                      Control Plane                               │
├───────────────┬───────────────┬───────────────┬─────────────────┤
│  API Server   │   Scheduler   │ Controller    │     etcd        │
│               │               │   Manager     │   (State Store) │
└───────┬───────┴───────────────┴───────────────┴─────────────────┘
        │
        │ kubelet communication
        ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Worker Nodes                               │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                          │
│  │ kubelet │  │ kube-   │  │Container│                          │
│  │         │  │ proxy   │  │ Runtime │                          │
│  └─────────┘  └─────────┘  └─────────┘                          │
│                                                                  │
│  ┌──────────────────────────────────────────┐                   │
│  │                  Pods                     │                   │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐        │                   │
│  │  │ App │ │ App │ │ App │ │ App │        │                   │
│  │  └─────┘ └─────┘ └─────┘ └─────┘        │                   │
│  └──────────────────────────────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘
```

### Core Kubernetes Objects

#### Deployments and StatefulSets
```yaml
# Production-grade Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-application
  namespace: production
  labels:
    app: web-application
    version: v2.1.0
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: web-application
  template:
    metadata:
      labels:
        app: web-application
        version: v2.1.0
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      serviceAccountName: web-application-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
      
      containers:
      - name: web-application
        image: registry.corp.local/web-app:v2.1.0
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 8443
          name: https
        
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: connection-string
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log-level
        
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
          readOnly: true
        - name: tls-certs
          mountPath: /app/certs
          readOnly: true
      
      volumes:
      - name: config-volume
        configMap:
          name: app-config
      - name: tls-certs
        secret:
          secretName: app-tls-secret
      
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-application
            topologyKey: kubernetes.io/hostname
      
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: web-application
```

#### Services and Networking
```yaml
# Service with multiple port definitions
apiVersion: v1
kind: Service
metadata:
  name: web-application
  namespace: production
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  selector:
    app: web-application
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
---
# NetworkPolicy for microsegmentation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-application-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web-application
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
          app: api-gateway
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

### RBAC Configuration
```yaml
# Service Account for application
apiVersion: v1
kind: ServiceAccount
metadata:
  name: web-application-sa
  namespace: production
---
# ClusterRole for cross-namespace operations
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deployment-manager
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods", "pods/log", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create"]
---
# RoleBinding to namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-manager-binding
  namespace: production
subjects:
- kind: Group
  name: platform-engineers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: deployment-manager
  apiGroup: rbac.authorization.k8s.io
```

---

## 2. OpenShift Enterprise Features <a name="openshift-features"></a>

### OpenShift vs Kubernetes Comparison
```
┌────────────────────────┬────────────────────────┬────────────────────────┐
│       Feature          │      Kubernetes        │       OpenShift        │
├────────────────────────┼────────────────────────┼────────────────────────┤
│ Installation           │ Manual / Tools         │ Automated Installer    │
│ Web Console            │ Dashboard (basic)      │ Full-featured Console  │
│ CI/CD                  │ External tools         │ Built-in Pipelines     │
│ Security               │ Configure manually     │ SCCs, Built-in Auth    │
│ Networking             │ CNI plugins            │ OVN-Kubernetes         │
│ Image Registry         │ External               │ Integrated Registry    │
│ Developer Experience   │ kubectl only           │ oc CLI + Console       │
│ Logging/Monitoring     │ Deploy separately      │ Integrated EFK/Prom    │
│ Service Mesh           │ Istio (manual)         │ OpenShift Service Mesh │
│ Virtualization         │ KubeVirt (manual)      │ OpenShift Virtualization│
└────────────────────────┴────────────────────────┴────────────────────────┘
```

### Security Context Constraints (SCCs)
```yaml
# Custom SCC for specific workloads
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: custom-restricted-scc
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegeEscalation: false
allowPrivilegedContainer: false
allowedCapabilities: null
defaultAddCapabilities: null
fsGroup:
  type: MustRunAs
  ranges:
  - min: 1000
    max: 65535
readOnlyRootFilesystem: true
requiredDropCapabilities:
- ALL
runAsUser:
  type: MustRunAsRange
  uidRangeMin: 1000
  uidRangeMax: 65535
seLinuxContext:
  type: MustRunAs
supplementalGroups:
  type: RunAsAny
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
users:
- system:serviceaccount:production:web-application-sa
groups: []
```

### OpenShift Projects and Resource Quotas
```yaml
# Project template with resource quotas
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    name: ${PROJECT_NAME}
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}

- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: compute-quota
    namespace: ${PROJECT_NAME}
  spec:
    hard:
      requests.cpu: "4"
      requests.memory: 8Gi
      limits.cpu: "8"
      limits.memory: 16Gi
      pods: "20"
      persistentvolumeclaims: "10"
      secrets: "50"
      configmaps: "50"

- apiVersion: v1
  kind: LimitRange
  metadata:
    name: default-limits
    namespace: ${PROJECT_NAME}
  spec:
    limits:
    - type: Container
      default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "100m"
        memory: "256Mi"
      max:
        cpu: "2"
        memory: "4Gi"
      min:
        cpu: "50m"
        memory: "64Mi"

parameters:
- name: PROJECT_NAME
  required: true
- name: PROJECT_DISPLAYNAME
  required: true
- name: PROJECT_DESCRIPTION
- name: PROJECT_REQUESTING_USER
  required: true
```

### OpenShift Builds and ImageStreams
```yaml
# BuildConfig with S2I
apiVersion: build.openshift.io/v1
kind: BuildConfig
metadata:
  name: web-application
  namespace: development
spec:
  source:
    type: Git
    git:
      uri: https://github.com/org/web-application.git
      ref: main
    contextDir: /
  strategy:
    type: Source
    sourceStrategy:
      from:
        kind: ImageStreamTag
        namespace: openshift
        name: nodejs:18-ubi8
      env:
      - name: NPM_MIRROR
        value: https://registry.npmjs.org
  output:
    to:
      kind: ImageStreamTag
      name: web-application:latest
  triggers:
  - type: GitHub
    github:
      secret: github-webhook-secret
  - type: ConfigChange
  - type: ImageChange
---
# ImageStream for version tracking
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: web-application
  namespace: development
spec:
  lookupPolicy:
    local: true
  tags:
  - name: latest
    from:
      kind: DockerImage
      name: registry.corp.local/web-application:latest
    importPolicy:
      scheduled: true
    referencePolicy:
      type: Local
```

---

## 3. OpenShift Virtualization <a name="openshift-virtualization"></a>

### Architecture Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                    OpenShift Cluster                             │
├─────────────────────────────────────────────────────────────────┤
│                OpenShift Virtualization Operator                 │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐  ┌─────────────────────┐               │
│  │   VM Workloads      │  │ Container Workloads │               │
│  │  ┌───────────────┐  │  │  ┌───────────────┐  │               │
│  │  │ Windows VM    │  │  │  │    Pod        │  │               │
│  │  │ (virt-launcher)│  │  │  │  (Container) │  │               │
│  │  └───────────────┘  │  │  └───────────────┘  │               │
│  │  ┌───────────────┐  │  │  ┌───────────────┐  │               │
│  │  │ Linux VM      │  │  │  │    Pod        │  │               │
│  │  │ (virt-launcher)│  │  │  │  (Container) │  │               │
│  │  └───────────────┘  │  │  └───────────────┘  │               │
│  └─────────────────────┘  └─────────────────────┘               │
├─────────────────────────────────────────────────────────────────┤
│              KubeVirt + CDI + Networking                         │
├─────────────────────────────────────────────────────────────────┤
│                  Worker Nodes (Bare Metal)                       │
└─────────────────────────────────────────────────────────────────┘
```

### VirtualMachine Definitions
```yaml
# Windows Server VM
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: windows-server-2022
  namespace: vm-workloads
  labels:
    app: windows-server
    os: windows
spec:
  running: true
  template:
    metadata:
      labels:
        kubevirt.io/vm: windows-server-2022
    spec:
      domain:
        cpu:
          cores: 4
          sockets: 1
          threads: 2
        memory:
          guest: 8Gi
        devices:
          disks:
          - name: rootdisk
            disk:
              bus: virtio
          - name: cloudinitdisk
            cdrom:
              bus: sata
          interfaces:
          - name: default
            masquerade: {}
          - name: secondary
            bridge: {}
        machine:
          type: q35
        resources:
          requests:
            memory: 8Gi
            cpu: "4"
      networks:
      - name: default
        pod: {}
      - name: secondary
        multus:
          networkName: vm-network
      volumes:
      - name: rootdisk
        dataVolume:
          name: windows-server-2022-disk
      - name: cloudinitdisk
        sysprep:
          configMap:
            name: windows-sysprep
---
# DataVolume for Windows installation
apiVersion: cdi.kubevirt.io/v1beta1
kind: DataVolume
metadata:
  name: windows-server-2022-disk
  namespace: vm-workloads
spec:
  source:
    http:
      url: "http://image-server.corp.local/images/windows-server-2022.qcow2"
  pvc:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 100Gi
    storageClassName: ocs-storagecluster-ceph-rbd
```

### VM Migration and Live Migration
```yaml
# VirtualMachineInstanceMigration for live migration
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstanceMigration
metadata:
  name: windows-server-migration
  namespace: vm-workloads
spec:
  vmiName: windows-server-2022
---
# MigrationPolicy for automated migrations
apiVersion: migrations.kubevirt.io/v1alpha1
kind: MigrationPolicy
metadata:
  name: production-migration-policy
spec:
  selectors:
    namespaceSelector:
      matchLabels:
        environment: production
  allowAutoConverge: true
  bandwidthPerMigration: 64Mi
  completionTimeoutPerGiB: 800
  allowPostCopy: false
```

### VM to Container Bridge Services
```yaml
# Service exposing VM to cluster
apiVersion: v1
kind: Service
metadata:
  name: windows-server-service
  namespace: vm-workloads
spec:
  selector:
    kubevirt.io/vm: windows-server-2022
  ports:
  - name: rdp
    port: 3389
    targetPort: 3389
  - name: winrm
    port: 5985
    targetPort: 5985
  - name: winrm-ssl
    port: 5986
    targetPort: 5986
---
# Route for external access
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: windows-rdp
  namespace: vm-workloads
spec:
  host: windows-rdp.apps.cluster.corp.local
  port:
    targetPort: rdp
  to:
    kind: Service
    name: windows-server-service
```

---

## 4. Multi-Cluster Management <a name="multi-cluster"></a>

### Red Hat Advanced Cluster Management (ACM)
```
┌─────────────────────────────────────────────────────────────────┐
│                    Hub Cluster (ACM)                             │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│  │ Governance  │ │ Application │ │  Cluster    │               │
│  │   Policy    │ │  Lifecycle  │ │  Lifecycle  │               │
│  └─────────────┘ └─────────────┘ └─────────────┘               │
├─────────────────────────────────────────────────────────────────┤
│                    Managed Clusters                              │
├───────────────┬───────────────┬───────────────┬─────────────────┤
│  On-Premises  │    Azure      │  Development  │   Disaster      │
│   Cluster     │    (ARO)      │   Cluster     │   Recovery      │
│   (prod)      │   (prod)      │    (dev)      │   Cluster       │
└───────────────┴───────────────┴───────────────┴─────────────────┘
```

### Cluster Set and Placement
```yaml
# ManagedClusterSet for grouping clusters
apiVersion: cluster.open-cluster-management.io/v1beta2
kind: ManagedClusterSet
metadata:
  name: production-clusters
spec:
  clusterSelector:
    selectorType: LabelSelector
    labelSelector:
      matchLabels:
        environment: production
---
# ManagedClusterSetBinding to namespace
apiVersion: cluster.open-cluster-management.io/v1beta2
kind: ManagedClusterSetBinding
metadata:
  name: production-clusters
  namespace: application-workloads
spec:
  clusterSet: production-clusters
---
# Placement for workload distribution
apiVersion: cluster.open-cluster-management.io/v1beta1
kind: Placement
metadata:
  name: production-placement
  namespace: application-workloads
spec:
  clusterSets:
  - production-clusters
  predicates:
  - requiredClusterSelector:
      labelSelector:
        matchExpressions:
        - key: environment
          operator: In
          values:
          - production
        - key: region
          operator: In
          values:
          - us-east
          - us-west
  tolerations:
  - key: cluster.open-cluster-management.io/unreachable
    operator: Exists
    tolerationSeconds: 300
```

### Policy-Based Governance
```yaml
# Policy for cluster compliance
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-namespace-security
  namespace: policies
spec:
  remediationAction: enforce
  disabled: false
  policy-templates:
  - objectDefinition:
      apiVersion: policy.open-cluster-management.io/v1
      kind: ConfigurationPolicy
      metadata:
        name: policy-pod-security-restricted
      spec:
        remediationAction: enforce
        severity: high
        namespaceSelector:
          include:
          - production-*
        object-templates:
        - complianceType: musthave
          objectDefinition:
            apiVersion: v1
            kind: Namespace
            metadata:
              labels:
                pod-security.kubernetes.io/enforce: restricted
                pod-security.kubernetes.io/audit: restricted
                pod-security.kubernetes.io/warn: restricted
---
# PlacementBinding to apply policy
apiVersion: policy.open-cluster-management.io/v1
kind: PlacementBinding
metadata:
  name: binding-policy-namespace-security
  namespace: policies
placementRef:
  name: production-placement
  kind: Placement
  apiGroup: cluster.open-cluster-management.io
subjects:
- name: policy-namespace-security
  kind: Policy
  apiGroup: policy.open-cluster-management.io
```

---

## 5. OpenShift on Azure (ARO) <a name="aro"></a>

### ARO Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure Region                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Virtual Network                        │    │
│  │  ┌─────────────────┐  ┌─────────────────┐               │    │
│  │  │  Control Plane  │  │   Worker Nodes  │               │    │
│  │  │    Subnet       │  │     Subnet      │               │    │
│  │  │                 │  │                 │               │    │
│  │  │ ┌───┐ ┌───┐ ┌───┐│ │ ┌───┐ ┌───┐ ┌───┐│               │    │
│  │  │ │M1 │ │M2 │ │M3 ││ │ │W1 │ │W2 │ │W3 ││               │    │
│  │  │ └───┘ └───┘ └───┘│ │ └───┘ └───┘ └───┘│               │    │
│  │  └─────────────────┘  └─────────────────┘               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │Azure Storage │  │ Azure DNS    │  │ Private Link │           │
│  └──────────────┘  └──────────────┘  └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

### ARO Deployment with Terraform
```hcl
# ARO cluster deployment
resource "azurerm_resource_group" "aro" {
  name     = "aro-cluster-rg"
  location = "eastus"
}

resource "azurerm_virtual_network" "aro" {
  name                = "aro-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.aro.location
  resource_group_name = azurerm_resource_group.aro.name
}

resource "azurerm_subnet" "control_plane" {
  name                 = "control-plane-subnet"
  resource_group_name  = azurerm_resource_group.aro.name
  virtual_network_name = azurerm_virtual_network.aro.name
  address_prefixes     = ["10.0.1.0/24"]
  
  private_link_service_network_policies_enabled = false
}

resource "azurerm_subnet" "worker" {
  name                 = "worker-subnet"
  resource_group_name  = azurerm_resource_group.aro.name
  virtual_network_name = azurerm_virtual_network.aro.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_redhat_openshift_cluster" "aro" {
  name                = "aro-cluster"
  location            = azurerm_resource_group.aro.location
  resource_group_name = azurerm_resource_group.aro.name
  
  cluster_profile {
    domain       = "example"
    version      = "4.14.16"
    pull_secret  = file("pull-secret.json")
  }
  
  network_profile {
    pod_cidr     = "10.128.0.0/14"
    service_cidr = "172.30.0.0/16"
  }
  
  main_profile {
    vm_size   = "Standard_D8s_v3"
    subnet_id = azurerm_subnet.control_plane.id
  }
  
  worker_profile {
    vm_size      = "Standard_D4s_v3"
    disk_size_gb = 128
    node_count   = 3
    subnet_id    = azurerm_subnet.worker.id
  }
  
  api_server_profile {
    visibility = "Private"
  }
  
  ingress_profile {
    visibility = "Private"
  }
  
  service_principal {
    client_id     = var.sp_client_id
    client_secret = var.sp_client_secret
  }
}
```

### ARO Integration with Azure Services
```yaml
# Azure Workload Identity configuration
apiVersion: v1
kind: ServiceAccount
metadata:
  name: azure-workload-identity
  namespace: application
  annotations:
    azure.workload.identity/client-id: "00000000-0000-0000-0000-000000000000"
    azure.workload.identity/tenant-id: "00000000-0000-0000-0000-000000000000"
---
# Pod using Azure Workload Identity
apiVersion: v1
kind: Pod
metadata:
  name: azure-app
  namespace: application
  labels:
    azure.workload.identity/use: "true"
spec:
  serviceAccountName: azure-workload-identity
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: AZURE_CLIENT_ID
      value: "00000000-0000-0000-0000-000000000000"
```

---

## 6. Operators and Custom Controllers <a name="operators"></a>

### Operator Pattern Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     Kubernetes API Server                        │
├─────────────────────────────────────────────────────────────────┤
│  Custom Resource │    Watch Events    │  Status Updates         │
│    Definitions   │                    │                         │
├─────────────────────────────────────────────────────────────────┤
│                        Operator                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │ Controller  │  │  Reconciler │  │   Webhooks  │              │
│  │   Manager   │  │    Logic    │  │  (Mutating/ │              │
│  │             │  │             │  │  Validating)│              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
├─────────────────────────────────────────────────────────────────┤
│               Managed Resources (Deployments, Services, etc.)    │
└─────────────────────────────────────────────────────────────────┘
```

### Custom Resource Definition
```yaml
# CRD for Application deployment
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.platform.corp.local
spec:
  group: platform.corp.local
  names:
    kind: Application
    listKind: ApplicationList
    plural: applications
    singular: application
    shortNames:
    - app
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required:
            - image
            - replicas
            properties:
              image:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 10
              environment:
                type: string
                enum: ["development", "staging", "production"]
              resources:
                type: object
                properties:
                  cpu:
                    type: string
                  memory:
                    type: string
              autoscaling:
                type: object
                properties:
                  enabled:
                    type: boolean
                  minReplicas:
                    type: integer
                  maxReplicas:
                    type: integer
                  targetCPUUtilization:
                    type: integer
          status:
            type: object
            properties:
              phase:
                type: string
              replicas:
                type: integer
              readyReplicas:
                type: integer
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    lastTransitionTime:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
    subresources:
      status: {}
    additionalPrinterColumns:
    - name: Replicas
      type: integer
      jsonPath: .spec.replicas
    - name: Ready
      type: integer
      jsonPath: .status.readyReplicas
    - name: Phase
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
```

### Operator Implementation (Go)
```go
// controllers/application_controller.go
package controllers

import (
	"context"
	"fmt"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/types"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"

	platformv1 "github.com/corp/platform-operator/api/v1"
)

type ApplicationReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

func (r *ApplicationReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	logger := log.FromContext(ctx)
	logger.Info("Reconciling Application", "name", req.NamespacedName)

	// Fetch the Application instance
	app := &platformv1.Application{}
	err := r.Get(ctx, req.NamespacedName, app)
	if err != nil {
		if errors.IsNotFound(err) {
			logger.Info("Application resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		return ctrl.Result{}, err
	}

	// Check if Deployment exists
	deployment := &appsv1.Deployment{}
	err = r.Get(ctx, types.NamespacedName{Name: app.Name, Namespace: app.Namespace}, deployment)
	if err != nil && errors.IsNotFound(err) {
		// Create Deployment
		deployment = r.deploymentForApplication(app)
		logger.Info("Creating Deployment", "name", deployment.Name)
		err = r.Create(ctx, deployment)
		if err != nil {
			return ctrl.Result{}, err
		}
		return ctrl.Result{Requeue: true}, nil
	} else if err != nil {
		return ctrl.Result{}, err
	}

	// Update Deployment if spec changed
	if needsUpdate(app, deployment) {
		deployment.Spec.Replicas = &app.Spec.Replicas
		deployment.Spec.Template.Spec.Containers[0].Image = app.Spec.Image
		
		err = r.Update(ctx, deployment)
		if err != nil {
			return ctrl.Result{}, err
		}
	}

	// Update Application status
	app.Status.Phase = "Running"
	app.Status.Replicas = deployment.Status.Replicas
	app.Status.ReadyReplicas = deployment.Status.ReadyReplicas
	
	err = r.Status().Update(ctx, app)
	if err != nil {
		return ctrl.Result{}, err
	}

	return ctrl.Result{}, nil
}

func (r *ApplicationReconciler) deploymentForApplication(app *platformv1.Application) *appsv1.Deployment {
	labels := map[string]string{
		"app":        app.Name,
		"managed-by": "platform-operator",
	}
	replicas := app.Spec.Replicas

	deployment := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      app.Name,
			Namespace: app.Namespace,
			Labels:    labels,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &replicas,
			Selector: &metav1.LabelSelector{
				MatchLabels: labels,
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: labels,
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{{
						Name:  app.Name,
						Image: app.Spec.Image,
						Ports: []corev1.ContainerPort{{
							ContainerPort: 8080,
						}},
					}},
				},
			},
		},
	}

	// Set Application as owner
	ctrl.SetControllerReference(app, deployment, r.Scheme)
	return deployment
}

func (r *ApplicationReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&platformv1.Application{}).
		Owns(&appsv1.Deployment{}).
		Complete(r)
}
```

---

## 7. Production Best Practices <a name="best-practices"></a>

### High Availability Configuration
```yaml
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-application-pdb
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web-application
---
# Priority Class for critical workloads
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-production
value: 1000000
globalDefault: false
description: "High priority for production workloads"
---
# Deployment with HA configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: critical-application
spec:
  replicas: 3
  template:
    spec:
      priorityClassName: high-priority-production
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: critical-application
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchLabels:
                app: critical-application
            topologyKey: kubernetes.io/hostname
```

### Resource Management
```yaml
# Vertical Pod Autoscaler
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-application-vpa
  namespace: production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-application
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
---
# Horizontal Pod Autoscaler with custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-application-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-application
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: 1000
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

### Security Hardening
```yaml
# Pod Security Standards
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
---
# Secure Pod Configuration
apiVersion: v1
kind: Pod
metadata:
  name: secure-application
  namespace: production
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: secure-app:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: cache
      mountPath: /app/cache
  volumes:
  - name: tmp
    emptyDir: {}
  - name: cache
    emptyDir: {}
```

---

## 8. Interview Scenarios and Use Cases <a name="interview-scenarios"></a>

### Scenario 1: Large-Scale Kubernetes Platform Migration

**Challenge**: Migrate 200+ applications from legacy VMs to OpenShift cluster

**Solution Architecture**:
```
Phase 1: Assessment (4 weeks)
├── Application discovery and classification
├── Dependency mapping
├── Containerization feasibility analysis
└── Priority matrix creation

Phase 2: Foundation (6 weeks)
├── OpenShift cluster deployment
├── CI/CD pipeline setup (GitOps)
├── Monitoring and logging stack
└── Security policies and SCCs

Phase 3: Migration Waves (16 weeks)
├── Wave 1: Stateless applications (low risk)
├── Wave 2: Stateful applications with replication
├── Wave 3: Legacy applications with modernization
└── Wave 4: Mission-critical applications

Phase 4: Optimization (4 weeks)
├── Performance tuning
├── Cost optimization
├── Automation refinement
└── Knowledge transfer
```

**Key Success Metrics**:
- 98% application migration success rate
- 40% reduction in infrastructure costs
- 60% improvement in deployment frequency
- Zero critical incidents during migration

### Scenario 2: OpenShift Virtualization for Windows Workloads

**Challenge**: Consolidate Windows VMs with containerized workloads on single platform

**Implementation**:
```yaml
# Windows VM with integration to containerized backend
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: legacy-windows-app
spec:
  running: true
  template:
    spec:
      domain:
        devices:
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 4Gi
      networks:
      - name: default
        pod: {}
      volumes:
      - name: rootdisk
        dataVolume:
          name: windows-2019-disk
---
# Service to connect VM to containerized services
apiVersion: v1
kind: Service
metadata:
  name: legacy-app-bridge
spec:
  selector:
    kubevirt.io/vm: legacy-windows-app
  ports:
  - port: 8080
    targetPort: 8080
---
# Containerized microservice consuming VM service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: modern-frontend
spec:
  template:
    spec:
      containers:
      - name: frontend
        env:
        - name: BACKEND_URL
          value: "http://legacy-app-bridge:8080"
```

### Scenario 3: Multi-Cluster DR with ACM

**Challenge**: Implement automated disaster recovery across regions

**Solution**:
```yaml
# Application placement for DR
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
  name: dr-placement
spec:
  clusterConditions:
  - type: ManagedClusterConditionAvailable
    status: "True"
  clusterReplicas: 2
  clusterSelector:
    matchExpressions:
    - key: region
      operator: In
      values:
      - us-east
      - us-west
---
# Subscription for DR deployment
apiVersion: apps.open-cluster-management.io/v1
kind: Subscription
metadata:
  name: critical-app-subscription
spec:
  channel: channel-namespace/application-channel
  placement:
    placementRef:
      name: dr-placement
      kind: PlacementRule
  overrides:
  - clusterName: us-east-cluster
    clusterOverrides:
    - path: spec.replicas
      value: 3
  - clusterName: us-west-cluster
    clusterOverrides:
    - path: spec.replicas
      value: 1  # Standby mode
```

### Common Interview Questions

#### Q1: "How do you handle stateful workloads in Kubernetes?"

**Answer**:
1. **StatefulSets** for ordered deployment and stable network identities
2. **Persistent Volumes** with appropriate storage class
3. **Pod Disruption Budgets** for maintenance windows
4. **Backup strategies** using Velero or native CSI snapshots
5. **Data replication** at application level (e.g., database clustering)

#### Q2: "Explain your approach to Kubernetes security"

**Answer**:
1. **Pod Security Standards** (restricted by default)
2. **Network Policies** for microsegmentation
3. **RBAC** with principle of least privilege
4. **Image scanning** in CI/CD pipeline
5. **Runtime security** with Falco or similar
6. **Secrets management** with external secrets operator
7. **Regular security audits** using kube-bench

#### Q3: "How do you troubleshoot a pod that won't start?"

**Answer (systematic approach)**:
```bash
# 1. Check pod status
kubectl describe pod <pod-name>

# 2. Check events
kubectl get events --sort-by='.lastTimestamp'

# 3. Check logs (if container started)
kubectl logs <pod-name> --previous

# 4. Check resource constraints
kubectl top nodes
kubectl describe resourcequota

# 5. Check node status
kubectl describe node <node-name>

# 6. Check image pull issues
kubectl get events | grep -i pull

# 7. Debug with ephemeral container
kubectl debug <pod-name> -it --image=busybox
```

---

## Quick Reference Commands

```bash
# OpenShift specific
oc login --token=<token> --server=<api-url>
oc new-project production
oc adm policy add-scc-to-user restricted -z myserviceaccount
oc get clusteroperators

# Kubernetes
kubectl get pods -A -o wide
kubectl top pods --sort-by=memory
kubectl rollout restart deployment/myapp
kubectl scale deployment myapp --replicas=5

# Troubleshooting
kubectl describe pod <pod> | grep -A 10 Events
kubectl logs <pod> -c <container> --previous
kubectl exec -it <pod> -- /bin/sh

# Cluster management
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>
```
