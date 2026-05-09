# Module 19: GitOps and CI/CD Pipelines

## Table of Contents
1. [GitOps Fundamentals](#gitops-fundamentals)
2. [Argo CD Deep Dive](#argocd)
3. [Flux CD Implementation](#flux)
4. [CI/CD Pipeline Design](#cicd-design)
5. [Azure DevOps Pipelines](#azure-devops)
6. [GitHub Actions](#github-actions)
7. [Pipeline Security and Compliance](#pipeline-security)
8. [Interview Scenarios](#interview-scenarios)

---

## 1. GitOps Fundamentals <a name="gitops-fundamentals"></a>

### What is GitOps?
GitOps is an operational framework that applies DevOps best practices for infrastructure automation using Git as the single source of truth.

### GitOps Principles
```
┌─────────────────────────────────────────────────────────────────┐
│                    GitOps Core Principles                        │
├─────────────────────────────────────────────────────────────────┤
│  1. Declarative Configuration                                   │
│     - Entire system described declaratively                     │
│     - Desired state stored in Git                               │
├─────────────────────────────────────────────────────────────────┤
│  2. Version Controlled & Immutable                              │
│     - Git as single source of truth                             │
│     - Complete audit trail                                      │
├─────────────────────────────────────────────────────────────────┤
│  3. Automatically Applied                                        │
│     - Approved changes auto-applied to system                   │
│     - No manual kubectl/oc commands                             │
├─────────────────────────────────────────────────────────────────┤
│  4. Continuously Reconciled                                      │
│     - Agents ensure system matches desired state                │
│     - Self-healing capabilities                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GitOps Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                     Developer Workflow                           │
├─────────────────────────────────────────────────────────────────┤
│  [Developer] → [Git Commit] → [PR Review] → [Merge to Main]     │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Git Repository (Source of Truth)              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │ Application │  │Infrastructure│  │  Policies   │              │
│  │   Configs   │  │    Configs   │  │   & RBAC    │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    GitOps Operator (Argo CD / Flux)              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │    Watch    │  │   Compare   │  │   Reconcile │              │
│  │     Git     │→ │ Desired vs  │→ │   Deploy    │              │
│  │   Changes   │  │   Actual    │  │   Changes   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Target Environment                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Dev Cluster│  │ Staging     │  │ Production  │              │
│  │             │  │  Cluster    │  │   Cluster   │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
└─────────────────────────────────────────────────────────────────┘
```

### Repository Structure for GitOps
```
gitops-repo/
├── apps/                          # Application configurations
│   ├── base/                      # Base configurations
│   │   ├── web-app/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── configmap.yaml
│   │   │   └── kustomization.yaml
│   │   └── api-service/
│   │       └── ...
│   └── overlays/                  # Environment-specific overlays
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── patches/
│       ├── staging/
│       │   ├── kustomization.yaml
│       │   └── patches/
│       └── production/
│           ├── kustomization.yaml
│           └── patches/
├── infrastructure/                 # Infrastructure configs
│   ├── namespaces/
│   ├── rbac/
│   ├── network-policies/
│   └── storage-classes/
├── clusters/                       # Cluster-specific configs
│   ├── dev-cluster/
│   │   └── kustomization.yaml
│   ├── staging-cluster/
│   │   └── kustomization.yaml
│   └── production-cluster/
│       └── kustomization.yaml
└── argocd/                        # Argo CD configurations
    ├── applications/
    ├── appprojects/
    └── repositories/
```

---

## 2. Argo CD Deep Dive <a name="argocd"></a>

### Argo CD Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Argo CD Components                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐                                                │
│  │    API      │  ← gRPC/REST API for UI & CLI                  │
│  │   Server    │                                                │
│  └─────────────┘                                                │
│         │                                                        │
│  ┌──────┴──────┐                                                │
│  │             │                                                 │
│  ▼             ▼                                                 │
│  ┌─────────────┐  ┌─────────────┐                               │
│  │ Repository  │  │Application  │                               │
│  │   Server    │  │ Controller  │                               │
│  │(Git sync)   │  │(Reconcile)  │                               │
│  └─────────────┘  └─────────────┘                               │
│                          │                                       │
│                          ▼                                       │
│                   ┌─────────────┐                               │
│                   │    Redis    │                                │
│                   │  (Caching)  │                                │
│                   └─────────────┘                                │
├─────────────────────────────────────────────────────────────────┤
│                   ApplicationSet Controller                      │
│           (For managing multiple applications)                   │
└─────────────────────────────────────────────────────────────────┘
```

### Argo CD Installation
```yaml
# argocd-install.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: argocd
---
# Install Argo CD with HA configuration
# kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/ha/install.yaml

# Custom configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Repository credentials
  repositories: |
    - url: https://github.com/org/gitops-repo.git
      passwordSecret:
        name: github-creds
        key: password
      usernameSecret:
        name: github-creds
        key: username
  
  # SSO configuration
  dex.config: |
    connectors:
      - type: microsoft
        id: microsoft
        name: Microsoft
        config:
          clientID: $AZURE_CLIENT_ID
          clientSecret: $AZURE_CLIENT_SECRET
          tenant: $AZURE_TENANT_ID
          redirectURI: https://argocd.corp.local/api/dex/callback
  
  # Resource tracking
  resource.customizations.health.argoproj.io_Application: |
    hs = {}
    hs.status = "Progressing"
    hs.message = ""
    if obj.status ~= nil then
      if obj.status.health ~= nil then
        hs.status = obj.status.health.status
        if obj.status.health.message ~= nil then
          hs.message = obj.status.health.message
        end
      end
    end
    return hs
```

### Application Definition
```yaml
# Application CR for web-app
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-application
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: production
  
  source:
    repoURL: https://github.com/org/gitops-repo.git
    targetRevision: main
    path: apps/overlays/production/web-app
    
    # Kustomize configuration
    kustomize:
      images:
        - registry.corp.local/web-app:v2.1.0
      namePrefix: prod-
      commonLabels:
        environment: production
  
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  
  # Health checks
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
  
  # Resource hooks
  info:
    - name: Documentation
      value: https://wiki.corp.local/web-app
```

### ApplicationSet for Multi-Cluster Deployment
```yaml
# ApplicationSet for deploying across multiple clusters
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: web-application-set
  namespace: argocd
spec:
  generators:
    # Matrix generator combining cluster and environment
    - matrix:
        generators:
          # Cluster generator
          - clusters:
              selector:
                matchLabels:
                  environment: production
          # List generator for applications
          - list:
              elements:
                - app: web-frontend
                  path: apps/web-frontend
                - app: api-gateway
                  path: apps/api-gateway
                - app: backend-service
                  path: apps/backend-service
  
  template:
    metadata:
      name: '{{name}}-{{app}}'
      namespace: argocd
      labels:
        cluster: '{{name}}'
        application: '{{app}}'
    spec:
      project: production
      source:
        repoURL: https://github.com/org/gitops-repo.git
        targetRevision: main
        path: '{{path}}/overlays/{{metadata.labels.environment}}'
        kustomize:
          commonLabels:
            cluster: '{{name}}'
      destination:
        server: '{{server}}'
        namespace: '{{app}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
---
# Git generator for pull request previews
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: pr-preview-apps
  namespace: argocd
spec:
  generators:
    - pullRequest:
        github:
          owner: org
          repo: application
          tokenRef:
            secretName: github-token
            key: token
        requeueAfterSeconds: 60
  template:
    metadata:
      name: 'preview-{{branch}}-{{number}}'
    spec:
      project: preview
      source:
        repoURL: https://github.com/org/application.git
        targetRevision: '{{head_sha}}'
        path: kubernetes/preview
      destination:
        server: https://kubernetes.default.svc
        namespace: 'preview-{{number}}'
      syncPolicy:
        automated:
          prune: true
        syncOptions:
          - CreateNamespace=true
```

### Argo CD App Projects
```yaml
# AppProject for production workloads
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production environment applications
  
  sourceRepos:
    - 'https://github.com/org/gitops-repo.git'
    - 'https://github.com/org/helm-charts.git'
  
  destinations:
    - namespace: 'production-*'
      server: https://kubernetes.default.svc
    - namespace: 'production-*'
      server: https://prod-cluster.corp.local
  
  # Allowed cluster resources
  clusterResourceWhitelist:
    - group: ''
      kind: Namespace
    - group: rbac.authorization.k8s.io
      kind: ClusterRole
    - group: rbac.authorization.k8s.io
      kind: ClusterRoleBinding
  
  # Namespace-scoped resources
  namespaceResourceWhitelist:
    - group: '*'
      kind: '*'
  
  # Blocked resources
  namespaceResourceBlacklist:
    - group: ''
      kind: ResourceQuota
    - group: ''
      kind: LimitRange
  
  # RBAC roles
  roles:
    - name: developer
      description: Developer access to production apps
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, allow
      groups:
        - developers
    
    - name: admin
      description: Full access to production apps
      policies:
        - p, proj:production:admin, applications, *, production/*, allow
        - p, proj:production:admin, repositories, *, production/*, allow
      groups:
        - platform-admins
  
  # Sync windows
  syncWindows:
    - kind: allow
      schedule: '0 8-18 * * 1-5'  # Weekdays 8am-6pm
      duration: 10h
      applications:
        - '*'
      manualSync: true
    - kind: deny
      schedule: '0 0-8,18-24 * * *'  # Outside business hours
      duration: 14h
      applications:
        - '*-critical'
```

### Argo CD Notifications
```yaml
# Notification configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Slack configuration
  service.slack: |
    token: $slack-token
  
  # Microsoft Teams configuration
  service.teams: |
    recipientUrls:
      deployments: $teams-webhook-url
  
  # Templates
  template.app-deployed: |
    message: |
      Application {{.app.metadata.name}} is now deployed.
      - Sync Status: {{.app.status.sync.status}}
      - Health Status: {{.app.status.health.status}}
      - Revision: {{.app.status.sync.revision}}
    teams:
      themeColor: "#00FF00"
      title: "Deployment Successful"
      summary: "{{.app.metadata.name}} deployed"
  
  template.app-health-degraded: |
    message: |
      ⚠️ Application {{.app.metadata.name}} health has degraded!
      - Health Status: {{.app.status.health.status}}
      - Message: {{.app.status.health.message}}
    teams:
      themeColor: "#FF0000"
      title: "Health Degraded"
      summary: "{{.app.metadata.name}} health issue"
  
  # Triggers
  trigger.on-deployed: |
    - when: app.status.operationState.phase in ['Succeeded'] and 
            app.status.health.status == 'Healthy'
      send: [app-deployed]
  
  trigger.on-health-degraded: |
    - when: app.status.health.status == 'Degraded'
      send: [app-health-degraded]

---
# Subscriptions
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  subscriptions: |
    - recipients:
        - slack:deployments
        - teams:deployments
      triggers:
        - on-deployed
        - on-health-degraded
      selector: app.kubernetes.io/instance=production
```

---

## 3. Flux CD Implementation <a name="flux"></a>

### Flux Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        Flux Components                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐                       │
│  │ Source Controller│  │ Kustomize       │                       │
│  │ (Git, Helm, OCI) │  │ Controller      │                       │
│  └─────────────────┘  └─────────────────┘                       │
│           │                    │                                 │
│           ▼                    ▼                                 │
│  ┌─────────────────┐  ┌─────────────────┐                       │
│  │ Helm Controller │  │ Notification    │                       │
│  │                 │  │ Controller      │                       │
│  └─────────────────┘  └─────────────────┘                       │
│           │                    │                                 │
│           └────────┬───────────┘                                │
│                    ▼                                             │
│  ┌─────────────────────────────────────┐                        │
│  │         Image Automation            │                        │
│  │   (Auto-update image tags in Git)   │                        │
│  └─────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
```

### Flux Bootstrap
```bash
# Bootstrap Flux on cluster
flux bootstrap github \
  --owner=org \
  --repository=gitops-repo \
  --branch=main \
  --path=clusters/production-cluster \
  --personal

# Bootstrap with Azure DevOps
flux bootstrap git \
  --url=https://dev.azure.com/org/project/_git/gitops-repo \
  --branch=main \
  --path=clusters/production-cluster \
  --username=git \
  --password=$AZURE_DEVOPS_PAT
```

### Flux Source Configuration
```yaml
# GitRepository source
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: gitops-repo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/org/gitops-repo.git
  ref:
    branch: main
  secretRef:
    name: github-credentials
  ignore: |
    # exclude all
    /*
    # include apps and infrastructure
    !/apps/
    !/infrastructure/
---
# HelmRepository source
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 1h
  url: https://charts.bitnami.com/bitnami
---
# OCIRepository for container images
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: oci-manifests
  namespace: flux-system
spec:
  interval: 5m
  url: oci://registry.corp.local/manifests
  ref:
    tag: latest
  secretRef:
    name: registry-credentials
```

### Flux Kustomization
```yaml
# Kustomization for infrastructure
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: gitops-repo
  path: ./infrastructure
  prune: true
  wait: true
  timeout: 5m
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: nginx-ingress-controller
      namespace: ingress-nginx
---
# Kustomization for applications (depends on infrastructure)
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: applications
  namespace: flux-system
spec:
  dependsOn:
    - name: infrastructure
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: gitops-repo
  path: ./apps/overlays/production
  prune: true
  wait: true
  timeout: 10m
  postBuild:
    substitute:
      ENVIRONMENT: production
      CLUSTER_NAME: prod-east
    substituteFrom:
      - kind: ConfigMap
        name: cluster-config
      - kind: Secret
        name: cluster-secrets
```

### Flux HelmRelease
```yaml
# HelmRelease for application deployment
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: web-application
  namespace: production
spec:
  interval: 5m
  chart:
    spec:
      chart: web-app
      version: '>=1.0.0 <2.0.0'
      sourceRef:
        kind: HelmRepository
        name: internal-charts
        namespace: flux-system
      interval: 1h
  
  values:
    replicaCount: 3
    image:
      repository: registry.corp.local/web-app
      tag: v2.1.0
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
  
  valuesFrom:
    - kind: ConfigMap
      name: web-app-values
      valuesKey: values.yaml
    - kind: Secret
      name: web-app-secrets
      valuesKey: secrets.yaml
  
  upgrade:
    remediation:
      retries: 3
      remediateLastFailure: true
    cleanupOnFail: true
  
  rollback:
    timeout: 5m
    cleanupOnFail: true
  
  test:
    enable: true
    timeout: 5m
```

### Flux Image Automation
```yaml
# Image repository scanning
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageRepository
metadata:
  name: web-app
  namespace: flux-system
spec:
  image: registry.corp.local/web-app
  interval: 1m
  secretRef:
    name: registry-credentials
---
# Image policy for automatic updates
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImagePolicy
metadata:
  name: web-app
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: web-app
  policy:
    semver:
      range: '>=1.0.0 <2.0.0'
  filterTags:
    pattern: '^v(?P<version>[0-9]+\.[0-9]+\.[0-9]+)$'
    extract: '$version'
---
# Image update automation
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: web-app-automation
  namespace: flux-system
spec:
  interval: 5m
  sourceRef:
    kind: GitRepository
    name: gitops-repo
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: flux@corp.local
        name: Flux Bot
      messageTemplate: |
        Auto-update: {{range .Updated.Images}}{{.}}{{end}}
    push:
      branch: main
  update:
    path: ./apps/overlays/production
    strategy: Setters
```

---

## 4. CI/CD Pipeline Design <a name="cicd-design"></a>

### Complete CI/CD Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                    CI/CD Pipeline Architecture                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│  │  Code   │───▶│  Build  │───▶│  Test   │───▶│ Security│       │
│  │ Commit  │    │   CI    │    │  Suite  │    │  Scan   │       │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘       │
│                                                      │           │
│                                                      ▼           │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│  │ Deploy  │◀───│  GitOps │◀───│ Approve │◀───│ Publish │       │
│  │ (Prod)  │    │  Sync   │    │  Gate   │    │ Artifact│       │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Pipeline Stages Definition
```yaml
# Generic pipeline stages
stages:
  - stage: Build
    jobs:
      - job: compile
        steps:
          - checkout
          - restore_dependencies
          - compile_code
          - unit_tests
          - code_coverage
      
      - job: container_build
        dependsOn: compile
        steps:
          - build_docker_image
          - push_to_registry
  
  - stage: Security
    dependsOn: Build
    jobs:
      - job: sast
        steps:
          - sonarqube_analysis
          - checkmarx_scan
      
      - job: dependency_scan
        steps:
          - snyk_analysis
          - dependabot_check
      
      - job: container_scan
        steps:
          - trivy_scan
          - prisma_cloud_scan
  
  - stage: Test
    dependsOn: Security
    jobs:
      - job: integration_tests
        steps:
          - deploy_to_test_env
          - run_integration_tests
          - cleanup_test_env
      
      - job: performance_tests
        steps:
          - deploy_to_perf_env
          - run_load_tests
          - analyze_results
  
  - stage: Deploy_Staging
    dependsOn: Test
    jobs:
      - job: update_gitops
        steps:
          - update_image_tag
          - create_pr
          - wait_for_sync
      
      - job: smoke_tests
        dependsOn: update_gitops
        steps:
          - run_smoke_tests
          - validate_health
  
  - stage: Deploy_Production
    dependsOn: Deploy_Staging
    condition: manual_approval
    jobs:
      - job: update_prod_gitops
        steps:
          - update_prod_image_tag
          - create_prod_pr
          - wait_for_sync
      
      - job: verify_deployment
        dependsOn: update_prod_gitops
        steps:
          - verify_health
          - run_smoke_tests
          - notify_stakeholders
```

---

## 5. Azure DevOps Pipelines <a name="azure-devops"></a>

### Complete Azure Pipeline
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - release/*
  paths:
    exclude:
      - '*.md'
      - 'docs/**'

pr:
  branches:
    include:
      - main
  autoCancel: true

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: production-vars
  - name: imageRepository
    value: 'web-application'
  - name: containerRegistry
    value: 'prodacr.azurecr.io'
  - name: dockerfilePath
    value: '$(Build.SourcesDirectory)/Dockerfile'
  - name: tag
    value: '$(Build.BuildId)'

stages:
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Build
        displayName: 'Build Application'
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'
            displayName: 'Install Node.js'
          
          - script: |
              npm ci
              npm run build
              npm run test:ci
            displayName: 'Install, Build, and Test'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFiles: '**/junit.xml'
              testRunTitle: 'Unit Tests'
            condition: always()
          
          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: '$(Build.SourcesDirectory)/coverage/cobertura-coverage.xml'
      
      - job: ContainerBuild
        displayName: 'Build Container'
        dependsOn: Build
        steps:
          - task: Docker@2
            displayName: 'Build and Push'
            inputs:
              command: buildAndPush
              repository: $(imageRepository)
              dockerfile: $(dockerfilePath)
              containerRegistry: $(dockerRegistryServiceConnection)
              tags: |
                $(tag)
                latest
          
          - task: AzureCLI@2
            displayName: 'Sign Container Image'
            inputs:
              azureSubscription: 'production-subscription'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az acr manifest sign \
                  --registry $(containerRegistry) \
                  --name $(imageRepository):$(tag)
  
  - stage: Security
    displayName: 'Security Scanning'
    dependsOn: Build
    jobs:
      - job: SAST
        displayName: 'Static Analysis'
        steps:
          - task: SonarQubePrepare@5
            inputs:
              SonarQube: 'SonarQube-Connection'
              scannerMode: 'CLI'
              configMode: 'manual'
              cliProjectKey: 'web-application'
          
          - task: SonarQubeAnalyze@5
          
          - task: SonarQubePublish@5
            inputs:
              pollingTimeoutSec: '300'
      
      - job: ContainerScan
        displayName: 'Container Vulnerability Scan'
        steps:
          - task: AzureCLI@2
            displayName: 'Scan with Defender'
            inputs:
              azureSubscription: 'production-subscription'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Get scan results from Microsoft Defender
                az security assessment list \
                  --query "[?contains(displayName, '$(imageRepository)')]" \
                  -o table
          
          - script: |
              docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy:latest image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                $(containerRegistry)/$(imageRepository):$(tag)
            displayName: 'Trivy Scan'
  
  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: Security
    jobs:
      - deployment: DeployStaging
        displayName: 'Deploy to Staging'
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: gitops-repo
                  persistCredentials: true
                
                - script: |
                    cd apps/overlays/staging/web-app
                    kustomize edit set image web-app=$(containerRegistry)/$(imageRepository):$(tag)
                    git config user.email "pipeline@corp.local"
                    git config user.name "Azure Pipeline"
                    git add .
                    git commit -m "Update web-app to $(tag)"
                    git push
                  displayName: 'Update GitOps Repo'
                
                - task: AzureCLI@2
                  displayName: 'Wait for Argo CD Sync'
                  inputs:
                    azureSubscription: 'production-subscription'
                    scriptType: 'bash'
                    scriptLocation: 'inlineScript'
                    inlineScript: |
                      # Wait for sync to complete
                      argocd app wait web-application-staging \
                        --timeout 300 \
                        --health \
                        --sync
      
      - job: SmokeTests
        displayName: 'Run Smoke Tests'
        dependsOn: DeployStaging
        steps:
          - script: |
              npm run test:smoke -- --url https://staging.corp.local
            displayName: 'Execute Smoke Tests'
  
  - stage: DeployProduction
    displayName: 'Deploy to Production'
    dependsOn: DeployStaging
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProduction
        displayName: 'Deploy to Production'
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: gitops-repo
                  persistCredentials: true
                
                - script: |
                    cd apps/overlays/production/web-app
                    kustomize edit set image web-app=$(containerRegistry)/$(imageRepository):$(tag)
                    git config user.email "pipeline@corp.local"
                    git config user.name "Azure Pipeline"
                    git add .
                    git commit -m "Update web-app to $(tag) [production]"
                    git push
                  displayName: 'Update Production GitOps Repo'
                
                - task: AzureCLI@2
                  displayName: 'Verify Deployment'
                  inputs:
                    azureSubscription: 'production-subscription'
                    scriptType: 'bash'
                    scriptLocation: 'inlineScript'
                    inlineScript: |
                      # Verify pods are running
                      kubectl get pods -n production -l app=web-app
                      
                      # Run health check
                      curl -f https://app.corp.local/health || exit 1
```

### Azure Pipeline Templates
```yaml
# templates/build-container.yml
parameters:
  - name: dockerfile
    type: string
    default: 'Dockerfile'
  - name: context
    type: string
    default: '.'
  - name: repository
    type: string
  - name: registry
    type: string

steps:
  - task: Docker@2
    displayName: 'Build Container Image'
    inputs:
      command: build
      repository: ${{ parameters.repository }}
      dockerfile: ${{ parameters.dockerfile }}
      buildContext: ${{ parameters.context }}
      tags: |
        $(Build.BuildId)
        $(Build.SourceVersion)
  
  - task: Docker@2
    displayName: 'Push Container Image'
    inputs:
      command: push
      repository: ${{ parameters.repository }}
      containerRegistry: ${{ parameters.registry }}
      tags: |
        $(Build.BuildId)
        $(Build.SourceVersion)
```

---

## 6. GitHub Actions <a name="github-actions"></a>

### Complete GitHub Actions Workflow
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, 'release/**']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run unit tests
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
      
      - name: Generate version
        id: version
        run: |
          VERSION=$(git describe --tags --always)
          echo "version=$VERSION" >> $GITHUB_OUTPUT

  security:
    name: Security Scanning
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Run CodeQL
        uses: github/codeql-action/analyze@v2

  container:
    name: Build Container
    runs-on: ubuntu-latest
    needs: [build, security]
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
            type=semver,pattern={{version}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Scan container
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: container
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
      - name: Checkout GitOps repo
        uses: actions/checkout@v4
        with:
          repository: org/gitops-repo
          token: ${{ secrets.GITOPS_TOKEN }}
          path: gitops
      
      - name: Update image tag
        run: |
          cd gitops/apps/overlays/staging
          kustomize edit set image app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
      
      - name: Commit and push
        run: |
          cd gitops
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Update staging to ${{ github.sha }}"
          git push
      
      - name: Wait for Argo CD sync
        uses: clowdhaus/argo-cd-action@main
        with:
          command: app wait
          options: --timeout 300 --health app-staging

  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: deploy-staging
    environment: 
      name: production
      url: https://app.corp.local
    
    steps:
      - name: Checkout GitOps repo
        uses: actions/checkout@v4
        with:
          repository: org/gitops-repo
          token: ${{ secrets.GITOPS_TOKEN }}
          path: gitops
      
      - name: Update production image
        run: |
          cd gitops/apps/overlays/production
          kustomize edit set image app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
      
      - name: Create PR
        uses: peter-evans/create-pull-request@v5
        with:
          path: gitops
          token: ${{ secrets.GITOPS_TOKEN }}
          commit-message: 'Deploy ${{ github.sha }} to production'
          title: 'Production Deploy: ${{ github.sha }}'
          body: |
            Automated production deployment
            - Image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            - Commit: ${{ github.sha }}
          branch: deploy/production-${{ github.sha }}
```

### Reusable Workflow
```yaml
# .github/workflows/deploy-environment.yml
name: Deploy to Environment

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image_tag:
        required: true
        type: string
    secrets:
      GITOPS_TOKEN:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    
    steps:
      - name: Checkout GitOps
        uses: actions/checkout@v4
        with:
          repository: org/gitops-repo
          token: ${{ secrets.GITOPS_TOKEN }}
      
      - name: Update manifest
        run: |
          cd apps/overlays/${{ inputs.environment }}
          kustomize edit set image app=registry.corp.local/app:${{ inputs.image_tag }}
      
      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Deploy ${{ inputs.image_tag }} to ${{ inputs.environment }}"
          git push
```

---

## 7. Pipeline Security and Compliance <a name="pipeline-security"></a>

### Secure Pipeline Practices
```yaml
# Security gates in pipeline
security_gates:
  - name: SAST Scan
    tool: SonarQube
    threshold:
      critical: 0
      high: 5
      medium: 20
    
  - name: Dependency Scan
    tool: Snyk
    threshold:
      critical: 0
      high: 0
    
  - name: Container Scan
    tool: Trivy
    threshold:
      critical: 0
      high: 5
    
  - name: Secret Scan
    tool: GitLeaks
    threshold:
      secrets_found: 0
    
  - name: IaC Scan
    tool: Checkov
    threshold:
      failed_checks: 10
```

### Signed Commits and Artifacts
```yaml
# GitHub Actions with signing
- name: Sign Container Image
  uses: sigstore/cosign-installer@main

- name: Sign and verify
  run: |
    cosign sign --key env://COSIGN_PRIVATE_KEY \
      ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
    
    cosign verify --key env://COSIGN_PUBLIC_KEY \
      ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
  env:
    COSIGN_PRIVATE_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}
    COSIGN_PUBLIC_KEY: ${{ secrets.COSIGN_PUBLIC_KEY }}
```

### SLSA Provenance
```yaml
# Generate SLSA provenance
- name: Generate SLSA Provenance
  uses: slsa-framework/slsa-github-generator/.github/workflows/generator_container_slsa3.yml@v1.9.0
  with:
    image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
    digest: ${{ needs.build.outputs.digest }}
```

---

## 8. Interview Scenarios <a name="interview-scenarios"></a>

### Scenario 1: GitOps Implementation at Scale

**Challenge**: Implement GitOps for 150+ microservices across 5 clusters

**Solution**:
```yaml
# ApplicationSet pattern for scale
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: all-services
spec:
  generators:
    - matrix:
        generators:
          - git:
              repoURL: https://github.com/org/services-catalog.git
              revision: HEAD
              files:
                - path: "services/*/config.json"
          - clusters:
              selector:
                matchLabels:
                  tier: production
  template:
    metadata:
      name: '{{cluster.name}}-{{service.name}}'
    spec:
      project: production
      source:
        repoURL: '{{service.repo}}'
        path: 'kubernetes/{{cluster.environment}}'
      destination:
        server: '{{cluster.server}}'
        namespace: '{{service.namespace}}'
```

**Key Achievements**:
- Deployment time reduced from 2 hours to 15 minutes
- Configuration drift eliminated across all clusters
- Self-service deployments enabled for 50+ teams

### Scenario 2: Zero-Downtime Pipeline Migration

**Challenge**: Migrate from Jenkins to Azure DevOps without disrupting deployments

**Approach**:
1. **Parallel Running** (2 weeks): Run both pipelines simultaneously
2. **Traffic Shifting** (2 weeks): Gradually shift teams to new pipeline
3. **Validation** (1 week): Verify all deployments successful
4. **Cutover** (1 day): Disable Jenkins pipelines
5. **Cleanup** (1 week): Decommission Jenkins infrastructure

### Common Interview Questions

**Q1: "How do you handle secrets in GitOps?"**

**Answer**:
1. **Sealed Secrets**: Encrypt secrets that can be stored in Git
2. **External Secrets Operator**: Sync from HashiCorp Vault, Azure Key Vault
3. **SOPS**: Mozilla's secret management tool
4. **Never store plain secrets in Git**

```yaml
# External Secrets Operator example
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: db-credentials
  data:
    - secretKey: password
      remoteRef:
        key: database-password
```

**Q2: "How do you implement rollbacks in GitOps?"**

**Answer**:
1. **Git Revert**: Revert the commit that introduced the change
2. **Argo CD Rollback**: Use `argocd app rollback` command
3. **History Tracking**: Argo CD maintains deployment history
4. **Automated Rollback**: Configure on health check failures

```bash
# Argo CD rollback
argocd app rollback web-app --to-revision 5

# Git revert
git revert HEAD
git push
```

**Q3: "Explain your CI/CD pipeline optimization strategies"**

**Answer**:
1. **Caching**: Dependency and layer caching
2. **Parallelization**: Run independent jobs concurrently
3. **Selective Builds**: Only build changed components
4. **Self-hosted Runners**: Reduce queue times
5. **Pipeline as Code**: Version controlled, reviewable pipelines

---

## Quick Reference Commands

```bash
# Argo CD
argocd app list
argocd app sync web-app
argocd app diff web-app
argocd app rollback web-app
argocd app history web-app

# Flux
flux get all
flux reconcile source git gitops-repo
flux reconcile kustomization apps
flux logs --follow

# Kustomize
kustomize build overlays/production
kustomize edit set image app=registry/app:v2

# GitHub Actions
gh workflow run ci-cd.yml
gh run list
gh run view <run-id>

# Azure DevOps
az pipelines run --name ci-cd
az pipelines build list
```
