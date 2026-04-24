# Module 11 - Azure DevOps and CI/CD (Complete Guide: Beginner to Expert)

## Learning Objectives
By the end of this module, you will be able to:
- Understand DevOps principles and practices comprehensively
- Master Azure DevOps Services components
- Design and implement CI/CD pipelines from scratch
- Configure build and release pipelines for enterprise applications
- Implement release management and deployment strategies
- Work with Azure Repos for source control
- Manage work items and Agile planning
- Implement automated testing in pipelines
- Configure security and governance for pipelines
- Integrate with GitHub Actions and other tools

---

## 11.1 Introduction to DevOps

### What is DevOps?
DevOps is a set of practices, tools, and cultural philosophies that automate and integrate the processes between software development (Dev) and IT operations (Ops) teams. It emphasizes collaboration, automation, continuous integration, continuous delivery, and monitoring.

### DevOps Culture and Principles

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         DevOps Infinity Loop                                │
│                                                                             │
│              PLAN → CODE → BUILD → TEST                                    │
│            ↗                           ↘                                   │
│        MONITOR                            RELEASE                           │
│            ↖                           ↙                                   │
│              OPERATE ← DEPLOY ← QA                                         │
│                                                                             │
│   Development Side (Dev)         │         Operations Side (Ops)           │
│   • Plan                         │         • Release                        │
│   • Code                         │         • Deploy                         │
│   • Build                        │         • Operate                        │
│   • Test                         │         • Monitor                        │
└────────────────────────────────────────────────────────────────────────────┘
```

### Core DevOps Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Continuous Integration (CI)** | Frequently merge code changes into a central repository | Automated builds on every commit |
| **Continuous Delivery (CD)** | Automatically prepare code changes for release to production | Automated testing and staging deployments |
| **Continuous Deployment** | Automatically release to production after passing tests | Full automation of deployment pipeline |
| **Infrastructure as Code** | Manage infrastructure through code | ARM templates, Terraform, Bicep |
| **Monitoring & Logging** | Track application performance and user behavior | Azure Monitor, Application Insights |
| **Communication & Collaboration** | Break down silos between teams | Shared tools, practices, and responsibilities |
| **Microservices Architecture** | Design as collection of loosely coupled services | Containers, APIs, service mesh |

### DevOps Lifecycle Stages

| Stage | Description | Tools |
|-------|-------------|-------|
| **Plan** | Define requirements, track work | Azure Boards, Jira, Asana |
| **Code** | Write and review code | VS Code, Git, PR reviews |
| **Build** | Compile code, run unit tests | Azure Pipelines, MSBuild, npm |
| **Test** | Automated testing | Selenium, NUnit, Jest |
| **Release** | Package for deployment | Artifacts, NuGet, npm |
| **Deploy** | Push to environments | Azure Pipelines, Octopus |
| **Operate** | Manage infrastructure | Azure, Kubernetes, Ansible |
| **Monitor** | Track performance | Azure Monitor, Grafana |

### DevOps vs Traditional Development

| Aspect | Traditional | DevOps |
|--------|-------------|--------|
| **Deployment frequency** | Monthly/Quarterly | Multiple times daily |
| **Lead time** | Weeks/Months | Hours/Days |
| **Change failure rate** | 31-45% | 0-15% |
| **Recovery time** | Days/Weeks | Hours/Minutes |
| **Team structure** | Siloed | Cross-functional |
| **Testing** | Manual, at end | Automated, continuous |

#### 🏢 Real-World DevOps Transformation Use Cases:

**Large Enterprise DevOps Journey:**
```
Company: GlobalBank (Fortune 500, 10,000 developers)
Challenge: 6-month release cycles, 40% failure rate

Before DevOps (2020):
├── Releases: 4 per year (quarterly)
├── Lead time: 6 months from idea to production
├── Deployment: 72-hour weekend window, 20 people
├── Failure rate: 40% of releases had critical bugs
├── Recovery: 1-2 weeks to fix production issues
└── Team structure: Dev team throws code "over the wall" to Ops

After DevOps (2024):
├── Releases: 50+ per day (microservices)
├── Lead time: 2 days from commit to production
├── Deployment: Automated, zero-downtime, 0 people
├── Failure rate: 2% (caught in staging, auto-rollback)
├── Recovery: 15 minutes (automated rollback)
└── Team structure: Product teams own full lifecycle

Transformation Steps:
Year 1: Pilot with 3 teams
├── Implemented CI/CD for new microservice
├── Containerized application
├── Results: 10x faster deployments

Year 2: Scale to 50 teams
├── Platform team built internal developer platform
├── Standardized pipeline templates
├── Results: 80% reduction in deployment time

Year 3: Organization-wide
├── Legacy monolith strangler pattern
├── Full observability stack
├── Results: $10M annual savings in ops costs
```

**Startup Rapid Iteration:**
```
Company: HealthTech Startup (20 developers)
Challenge: Ship features fast while maintaining quality

CI/CD Pipeline:
┌───────────────────────────────────────────────────────────────┐
│  Developer commits to feature branch                          │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────┐                                          │
│  │   PR Created    │  ← Code review required (2 approvers)   │
│  └────────┬────────┘                                          │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              Automated CI Pipeline (15 min)              │  │
│  │  ├── Build: Compile TypeScript, Docker build            │  │
│  │  ├── Unit tests: 2,000 tests in parallel                │  │
│  │  ├── Integration tests: Test with real database         │  │
│  │  ├── Security scan: Snyk for vulnerabilities            │  │
│  │  ├── Code quality: SonarQube analysis                   │  │
│  │  └── Preview environment: Unique URL for testing        │  │
│  └────────┬────────────────────────────────────────────────┘  │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────┐                                          │
│  │  PR Merged      │  → Triggers CD                          │
│  └────────┬────────┘                                          │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              Automated CD Pipeline                       │  │
│  │  Stage → Canary (5%) → Progressive (25%, 50%, 100%)   │  │
│  │  Auto-rollback if error rate > 1%                       │  │
│  └─────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────┘

Results:
├── Deploy frequency: 15-20 times per day
├── Idea to production: 4 hours
├── Developer confidence: "Push on Friday? No problem!"
└── Zero downtime in 18 months
```

**Multi-Team Microservices Pipeline:**
```
Company: E-Commerce Platform (200 developers, 50 services)
Challenge: Coordinate releases across 50 independent teams

Architecture:
┌─────────────────────────────────────────────────────────────┐
│              Shared Pipeline Templates (YAML)               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Template: backend-service.yml                               │
│  Used by: 35 backend services                                │
│  Includes: Build → Test → Scan → Deploy to AKS             │
│                                                              │
│  Template: frontend-app.yml                                  │
│  Used by: 5 frontend apps                                    │
│  Includes: Build → Test → Deploy to CDN                     │
│                                                              │
│  Template: infrastructure.yml                                │
│  Used by: Platform team                                      │
│  Includes: Terraform plan/apply with approval               │
│                                                              │
│  Governance:                                                 │
│  ├── All pipelines must use approved templates              │
│  ├── Security scanning is mandatory (cannot skip)           │
│  ├── Production deploys require Change Advisory Board       │
│  └── Automated compliance reporting                          │
└─────────────────────────────────────────────────────────────┘

Team Independence:
├── Each team owns their service repository
├── Deploys independently (no coordination needed)
├── Uses shared templates (consistency)
└── Platform team maintains templates and shared infrastructure
```

---

## 11.2 Azure DevOps Services Overview

### What is Azure DevOps?
Azure DevOps is a comprehensive suite of development tools that supports the entire software development lifecycle. It provides integrated features for planning, developing, delivering, and operating software.

### Azure DevOps Components

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Azure DevOps Services                              │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │             │  │             │  │             │  │             │        │
│  │   Azure     │  │   Azure     │  │   Azure     │  │   Azure     │        │
│  │   Boards    │  │   Repos     │  │  Pipelines  │  │ Test Plans  │        │
│  │             │  │             │  │             │  │             │        │
│  │ • Work Items│  │ • Git Repos │  │ • Build     │  │ • Manual    │        │
│  │ • Backlogs  │  │ • TFVC      │  │ • Release   │  │ • Automated │        │
│  │ • Sprints   │  │ • Pull Req  │  │ • YAML      │  │ • Load Test │        │
│  │ • Queries   │  │ • Branches  │  │ • Classic   │  │ • Exploratory│       │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                                              │
│                         ┌─────────────┐                                     │
│                         │             │                                     │
│                         │   Azure     │                                     │
│                         │  Artifacts  │                                     │
│                         │             │                                     │
│                         │ • NuGet     │                                     │
│                         │ • npm       │                                     │
│                         │ • Maven     │                                     │
│                         │ • Python    │                                     │
│                         └─────────────┘                                     │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Azure DevOps vs Azure DevOps Server

| Feature | Azure DevOps (Cloud) | Azure DevOps Server (On-Prem) |
|---------|---------------------|------------------------------|
| **Hosting** | Microsoft-managed | Self-hosted |
| **Updates** | Automatic (every 3 weeks) | Manual |
| **Scaling** | Automatic | Manual configuration |
| **Backup** | Managed | Self-managed |
| **Cost** | Per-user or resource-based | License purchase |
| **Best for** | Most organizations | Regulatory/Compliance needs |

### Creating an Azure DevOps Organization

#### Step-by-Step Process
1. Navigate to https://dev.azure.com
2. Sign in with your Microsoft account
3. Click "New organization"
4. Choose organization name and region
5. Create your first project

```powershell
# Install Azure DevOps CLI extension
az extension add --name azure-devops

# Login to Azure DevOps
az devops login --organization https://dev.azure.com/YourOrganization

# Create a new project
az devops project create `
    --name "MyProject" `
    --org https://dev.azure.com/YourOrganization `
    --visibility private `
    --source-control git `
    --process Agile
```

### Project Settings and Configuration

```powershell
# List all projects
az devops project list --org https://dev.azure.com/YourOrganization

# Show project details
az devops project show --project MyProject

# Configure default project
az devops configure --defaults organization=https://dev.azure.com/YourOrganization project=MyProject
```

---

## 11.3 Azure Repos - Source Control

### What is Azure Repos?
Azure Repos provides Git repositories or Team Foundation Version Control (TFVC) for source control of your code.

### Git vs TFVC

| Feature | Git | TFVC |
|---------|-----|------|
| **Type** | Distributed | Centralized |
| **Branching** | Lightweight, fast | Server-based |
| **Offline work** | Full capability | Limited |
| **History** | Every developer has full history | Server has history |
| **Recommended for** | Most projects | Very large codebases |

### Git Repository Structure

```
┌─────────────────────────────────────────────────────────┐
│                   Azure Repos                            │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Remote Repository                   │   │
│  │                (Azure DevOps)                    │   │
│  │                                                  │   │
│  │  main ─────┬─── develop ─────┬─── feature/*     │   │
│  │            │                 │                   │   │
│  │            └─── release/* ───┴─── hotfix/*      │   │
│  └─────────────────────────────────────────────────┘   │
│                          │                              │
│                     git push/pull                       │
│                          │                              │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Local Repository                    │   │
│  │            (Developer Machine)                   │   │
│  │                                                  │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐  │   │
│  │  │ Working │→ │ Staging │→ │ Local Commits   │  │   │
│  │  │Directory│  │  Area   │  │                 │  │   │
│  │  └─────────┘  └─────────┘  └─────────────────┘  │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Creating and Cloning Repositories

```bash
# Create a new repository in Azure DevOps
az repos create --name "MyNewRepo" --project MyProject

# Clone repository
git clone https://dev.azure.com/YourOrganization/MyProject/_git/MyNewRepo

# Set up remote
git remote add origin https://dev.azure.com/YourOrganization/MyProject/_git/MyNewRepo
```

### Branching Strategies

#### GitFlow

```
┌─────────────────────────────────────────────────────────────────┐
│                        GitFlow Model                             │
│                                                                  │
│  main         ●────────────────●─────────────●──────── (stable)  │
│                               ↑              ↑                   │
│  release/*                    │     ●────────● (v1.0)            │
│                               │    ↗                             │
│  develop      ●───●───●───●───●────●───●───● (integration)      │
│                  ↑   ↑   ↗    ↑                                  │
│  feature/a       └───●   │    │                                  │
│                          │    │                                  │
│  feature/b          └────●    │                                  │
│                               │                                  │
│  hotfix/*                     └────●──── (emergency fix)         │
└─────────────────────────────────────────────────────────────────┘
```

#### Trunk-Based Development

```
┌─────────────────────────────────────────────────────────────────┐
│                  Trunk-Based Development                         │
│                                                                  │
│  main     ●───●───●───●───●───●───●───●───●───● (all commits)   │
│               ↑   ↑       ↑   ↑       ↑   ↑                      │
│  short-lived  └───●       │   │       │   │                      │
│  branches         └───────●   │       │   │                      │
│                               └───────●   │                      │
│                                           └───●                  │
│                                                                  │
│  Releases      [v1.0]         [v1.1]         [v1.2]             │
│                  ↓             ↓               ↓                 │
│                (tag)         (tag)           (tag)               │
└─────────────────────────────────────────────────────────────────┘
```

### Branch Policies

```powershell
# Create branch policy requiring PR with minimum reviewers
az repos policy approver-count create `
    --branch main `
    --repository-id <repo-id> `
    --blocking true `
    --enabled true `
    --minimum-approver-count 2 `
    --reset-on-source-push true

# Require build validation
az repos policy build create `
    --branch main `
    --repository-id <repo-id> `
    --build-definition-id <build-id> `
    --blocking true `
    --enabled true `
    --queue-on-source-update-only true
```

### Pull Request Workflow

```yaml
# Example PR workflow:
# 1. Create feature branch
# 2. Make changes
# 3. Push to remote
# 4. Create PR
# 5. Code review
# 6. Build validation
# 7. Approve and merge

# Create PR using CLI
az repos pr create `
    --repository MyRepo `
    --source-branch feature/my-feature `
    --target-branch main `
    --title "Add new feature" `
    --description "This PR adds the new feature..." `
    --reviewers user@company.com
```

---

## 11.4 Azure Pipelines - Build and Release

### What is Azure Pipelines?
Azure Pipelines is a cloud-based service that automatically builds and tests code projects and makes them available to other users. It supports any language and platform.

### Pipeline Types

| Type | Description | Use Case |
|------|-------------|----------|
| **YAML Pipelines** | Pipeline as code in YAML files | Modern, recommended approach |
| **Classic Build Pipelines** | GUI-based pipeline editor | Legacy, simpler setup |
| **Classic Release Pipelines** | GUI-based release management | Complex release scenarios |

### YAML Pipeline Structure

```yaml
# azure-pipelines.yml - Complete structure
trigger:                  # When to run
  - main
  - develop

pr:                       # Pull request triggers
  - main

pool:                     # Agent pool
  vmImage: 'ubuntu-latest'

variables:                # Global variables
  buildConfiguration: 'Release'

stages:                   # Pipeline stages
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - task: TaskName@version
            inputs:
              key: value
  
  - stage: Deploy
    dependsOn: Build
    jobs:
      - deployment: DeployJob
        environment: Production
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo Deploying
```

### Complete .NET Application Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/*
    exclude:
      - docs/*

pr:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'
  dotnetVersion: '8.0.x'

stages:
  # ==================== BUILD STAGE ====================
  - stage: Build
    displayName: 'Build Stage'
    jobs:
      - job: Build
        displayName: 'Build .NET Application'
        steps:
          # Install .NET SDK
          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              packageType: 'sdk'
              version: '$(dotnetVersion)'

          # Restore NuGet packages
          - task: DotNetCoreCLI@2
            displayName: 'Restore NuGet Packages'
            inputs:
              command: 'restore'
              projects: '$(solution)'
              feedsToUse: 'select'

          # Build solution
          - task: DotNetCoreCLI@2
            displayName: 'Build Solution'
            inputs:
              command: 'build'
              projects: '$(solution)'
              arguments: '--configuration $(buildConfiguration) --no-restore'

          # Run unit tests
          - task: DotNetCoreCLI@2
            displayName: 'Run Unit Tests'
            inputs:
              command: 'test'
              projects: '**/*Tests/*.csproj'
              arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'

          # Publish test results
          - task: PublishTestResults@2
            displayName: 'Publish Test Results'
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/*.trx'
              mergeTestResults: true

          # Publish code coverage
          - task: PublishCodeCoverageResults@1
            displayName: 'Publish Code Coverage'
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'

          # Publish artifact
          - task: DotNetCoreCLI@2
            displayName: 'Publish Application'
            inputs:
              command: 'publish'
              projects: '**/MyApp.csproj'
              arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

          - task: PublishBuildArtifacts@1
            displayName: 'Publish Build Artifact'
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  # ==================== TEST STAGE ====================
  - stage: Test
    displayName: 'Integration Test Stage'
    dependsOn: Build
    jobs:
      - job: IntegrationTests
        displayName: 'Run Integration Tests'
        steps:
          - task: DownloadBuildArtifacts@0
            inputs:
              buildType: 'current'
              downloadType: 'single'
              artifactName: 'drop'
              downloadPath: '$(System.ArtifactsDirectory)'

          - task: DotNetCoreCLI@2
            displayName: 'Run Integration Tests'
            inputs:
              command: 'test'
              projects: '**/*IntegrationTests/*.csproj'
              arguments: '--configuration $(buildConfiguration)'

  # ==================== DEPLOY TO DEV ====================
  - stage: DeployDev
    displayName: 'Deploy to Development'
    dependsOn: Test
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
    jobs:
      - deployment: DeployToDev
        displayName: 'Deploy to Dev Environment'
        environment: 'Development'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Azure App Service'
                  inputs:
                    azureSubscription: 'MyAzureSubscription'
                    appType: 'webApp'
                    appName: 'myapp-dev'
                    package: '$(Pipeline.Workspace)/drop/*.zip'

  # ==================== DEPLOY TO STAGING ====================
  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: DeployDev
    condition: succeeded()
    jobs:
      - deployment: DeployToStaging
        displayName: 'Deploy to Staging Environment'
        environment: 'Staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Staging'
                  inputs:
                    azureSubscription: 'MyAzureSubscription'
                    appType: 'webApp'
                    appName: 'myapp-staging'
                    package: '$(Pipeline.Workspace)/drop/*.zip'

                # Run smoke tests
                - task: PowerShell@2
                  displayName: 'Run Smoke Tests'
                  inputs:
                    targetType: 'inline'
                    script: |
                      $response = Invoke-WebRequest -Uri "https://myapp-staging.azurewebsites.net/health"
                      if ($response.StatusCode -ne 200) {
                        throw "Smoke test failed"
                      }

  # ==================== DEPLOY TO PRODUCTION ====================
  - stage: DeployProduction
    displayName: 'Deploy to Production'
    dependsOn: DeployStaging
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployToProduction
        displayName: 'Deploy to Production Environment'
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Production'
                  inputs:
                    azureSubscription: 'MyAzureSubscription'
                    appType: 'webApp'
                    appName: 'myapp-prod'
                    package: '$(Pipeline.Workspace)/drop/*.zip'
                    deploymentMethod: 'zipDeploy'
```

### Node.js Application Pipeline

```yaml
# azure-pipelines.yml for Node.js
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '18.x'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          # Use Node.js version
          - task: NodeTool@0
            displayName: 'Install Node.js'
            inputs:
              versionSpec: '$(nodeVersion)'

          # Install dependencies
          - script: npm ci
            displayName: 'Install Dependencies'

          # Run linting
          - script: npm run lint
            displayName: 'Run ESLint'

          # Run tests
          - script: npm test -- --coverage
            displayName: 'Run Tests'

          # Build application
          - script: npm run build
            displayName: 'Build Application'

          # Archive build output
          - task: ArchiveFiles@2
            displayName: 'Archive Build'
            inputs:
              rootFolderOrFile: '$(System.DefaultWorkingDirectory)/dist'
              includeRootFolder: false
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/$(Build.BuildId).zip'

          # Publish artifact
          - task: PublishBuildArtifacts@1
            displayName: 'Publish Artifact'
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'
```

---

## 11.5 Pipeline Templates and Reusability

### Creating Pipeline Templates

Templates allow you to define reusable content, logic, and parameters.

#### Step Template

```yaml
# templates/build-steps.yml
parameters:
  - name: buildConfiguration
    type: string
    default: 'Release'
  - name: projectPath
    type: string

steps:
  - task: DotNetCoreCLI@2
    displayName: 'Restore'
    inputs:
      command: 'restore'
      projects: '${{ parameters.projectPath }}'

  - task: DotNetCoreCLI@2
    displayName: 'Build'
    inputs:
      command: 'build'
      projects: '${{ parameters.projectPath }}'
      arguments: '--configuration ${{ parameters.buildConfiguration }}'

  - task: DotNetCoreCLI@2
    displayName: 'Test'
    inputs:
      command: 'test'
      projects: '**/*Tests.csproj'
```

#### Job Template

```yaml
# templates/deploy-job.yml
parameters:
  - name: environment
    type: string
  - name: azureSubscription
    type: string
  - name: appName
    type: string

jobs:
  - deployment: Deploy${{ parameters.environment }}
    displayName: 'Deploy to ${{ parameters.environment }}'
    environment: ${{ parameters.environment }}
    strategy:
      runOnce:
        deploy:
          steps:
            - task: AzureWebApp@1
              displayName: 'Deploy to Azure Web App'
              inputs:
                azureSubscription: '${{ parameters.azureSubscription }}'
                appType: 'webApp'
                appName: '${{ parameters.appName }}'
                package: '$(Pipeline.Workspace)/**/*.zip'
```

#### Stage Template

```yaml
# templates/deploy-stage.yml
parameters:
  - name: stageName
    type: string
  - name: dependsOn
    type: object
    default: []
  - name: environment
    type: string
  - name: azureSubscription
    type: string
  - name: appName
    type: string
  - name: condition
    type: string
    default: 'succeeded()'

stages:
  - stage: ${{ parameters.stageName }}
    displayName: 'Deploy to ${{ parameters.environment }}'
    dependsOn: ${{ parameters.dependsOn }}
    condition: ${{ parameters.condition }}
    jobs:
      - template: deploy-job.yml
        parameters:
          environment: ${{ parameters.environment }}
          azureSubscription: ${{ parameters.azureSubscription }}
          appName: ${{ parameters.appName }}
```

#### Using Templates

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - template: templates/build-steps.yml
            parameters:
              buildConfiguration: 'Release'
              projectPath: '**/*.csproj'

  - template: templates/deploy-stage.yml
    parameters:
      stageName: DeployDev
      dependsOn: [Build]
      environment: 'Development'
      azureSubscription: 'MySubscription'
      appName: 'myapp-dev'

  - template: templates/deploy-stage.yml
    parameters:
      stageName: DeployProd
      dependsOn: [DeployDev]
      environment: 'Production'
      azureSubscription: 'MySubscription'
      appName: 'myapp-prod'
      condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
```

---

## 11.6 Variables, Secrets, and Variable Groups

### Variable Types

| Type | Scope | Use Case |
|------|-------|----------|
| **Pipeline Variables** | Single pipeline | Build-specific settings |
| **Variable Groups** | Multiple pipelines | Shared configuration |
| **Runtime Variables** | Job execution | Dynamic values |
| **Template Parameters** | Template instantiation | Customization |

### Defining Variables

```yaml
# Inline variables
variables:
  buildConfiguration: 'Release'
  vmImageName: 'ubuntu-latest'

# Variable groups
variables:
  - group: MyVariableGroup
  - name: localVariable
    value: 'local value'

# Variables from templates
variables:
  - template: variables/common-variables.yml
```

### Secret Variables

```yaml
# Reference secret from Azure Key Vault
variables:
  - group: MyKeyVaultSecrets  # Variable group linked to Key Vault

# Use in pipeline
steps:
  - script: echo $(databasePassword)  # Masked in logs
    env:
      DB_PASSWORD: $(databasePassword)  # Passed as env variable
```

### Variable Group with Key Vault

```powershell
# Create variable group linked to Key Vault
az pipelines variable-group create `
    --name "Production-Secrets" `
    --authorize true `
    --organization https://dev.azure.com/YourOrg `
    --project MyProject `
    --variables dummy=dummy

# Link to Key Vault (done via Azure Portal or REST API)
```

### Dynamic Variables

```yaml
steps:
  # Set variable in one step
  - script: |
      echo "##vso[task.setvariable variable=myDynamicVar]dynamicValue"
      echo "##vso[task.setvariable variable=secretVar;issecret=true]secretValue"
    displayName: 'Set Dynamic Variables'

  # Use in another step
  - script: echo $(myDynamicVar)
    displayName: 'Use Dynamic Variable'

  # Pass to another job (must be output variable)
  - script: |
      echo "##vso[task.setvariable variable=myOutputVar;isOutput=true]outputValue"
    name: setVarStep
```

---

## 11.7 Environments and Deployment Strategies

### What are Environments?
Environments represent targets for deployment (Dev, Staging, Production) and provide features like approvals, gates, and deployment history.

### Creating Environments

```yaml
# Define deployment job with environment
stages:
  - stage: Deploy
    jobs:
      - deployment: DeployWeb
        environment: 'Production'  # Creates environment if doesn't exist
        strategy:
          runOnce:
            deploy:
              steps:
                - script: echo Deploying
```

### Environment Approvals

Configure in Azure DevOps UI:
1. Go to Pipelines → Environments
2. Select environment
3. Click "..." → Approvals and checks
4. Add approvers

### Deployment Strategies

#### 1. RunOnce Strategy (Default)

```yaml
strategy:
  runOnce:
    preDeploy:
      steps:
        - script: echo Pre-deploy
    deploy:
      steps:
        - script: echo Deploying
    routeTraffic:
      steps:
        - script: echo Routing traffic
    postRouteTraffic:
      steps:
        - script: echo Post-routing
    on:
      failure:
        steps:
          - script: echo Failed
      success:
        steps:
          - script: echo Succeeded
```

#### 2. Rolling Deployment

```yaml
strategy:
  rolling:
    maxParallel: 2  # Deploy to 2 targets at a time
    preDeploy:
      steps:
        - script: echo Pre-deploy
    deploy:
      steps:
        - script: echo Deploying to $(Agent.MachineName)
    on:
      failure:
        steps:
          - script: echo Rolling back
```

#### 3. Canary Deployment

```yaml
strategy:
  canary:
    increments: [10, 20]  # Deploy to 10%, then 20%
    preDeploy:
      steps:
        - script: echo Pre-deploy
    deploy:
      steps:
        - script: echo Deploying to $(Strategy.Action) percent
    routeTraffic:
      steps:
        - script: echo Routing $(Strategy.WeightExpression) percent
    postRouteTraffic:
      pool: server
      steps:
        - task: Delay@1
          inputs:
            delayForMinutes: '5'  # Wait for metrics
    on:
      failure:
        steps:
          - script: echo Rolling back
      success:
        steps:
          - script: echo Canary successful
```

### Blue-Green Deployment with Azure

```yaml
# Blue-Green deployment using App Service slots
stages:
  - stage: DeployToStaging
    jobs:
      - deployment: DeployToStaging
        environment: 'Production-Staging'
        strategy:
          runOnce:
            deploy:
              steps:
                # Deploy to staging slot
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'MySubscription'
                    appType: 'webApp'
                    appName: 'myapp'
                    deployToSlotOrASE: true
                    slotName: 'staging'
                    package: '$(Pipeline.Workspace)/**/*.zip'

  - stage: SwapSlots
    dependsOn: DeployToStaging
    jobs:
      - deployment: SwapToProduction
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                # Swap staging to production
                - task: AzureAppServiceManage@0
                  inputs:
                    azureSubscription: 'MySubscription'
                    action: 'Swap Slots'
                    webAppName: 'myapp'
                    sourceSlot: 'staging'
                    targetSlot: 'production'
```

---

## 11.8 Azure Artifacts

### What is Azure Artifacts?
Azure Artifacts enables teams to share code artifacts (packages) across projects and organizations.

### Supported Package Types

| Package Type | Ecosystem | Use Case |
|--------------|-----------|----------|
| **NuGet** | .NET | .NET libraries |
| **npm** | Node.js | JavaScript packages |
| **Maven** | Java | Java libraries |
| **Gradle** | Java | Java libraries |
| **Python (pip)** | Python | Python packages |
| **Universal Packages** | Any | Binary files, tools |

### Publishing NuGet Packages

```yaml
# Build and publish NuGet package
steps:
  - task: DotNetCoreCLI@2
    displayName: 'Pack NuGet Package'
    inputs:
      command: 'pack'
      packagesToPack: '**/MyLibrary.csproj'
      versioningScheme: 'byBuildNumber'

  - task: NuGetCommand@2
    displayName: 'Push to Azure Artifacts'
    inputs:
      command: 'push'
      packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
      nuGetFeedType: 'internal'
      publishVstsFeed: 'MyFeed'
```

### Publishing npm Packages

```yaml
steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'

  - task: Npm@1
    displayName: 'npm publish'
    inputs:
      command: 'publish'
      workingDir: '$(System.DefaultWorkingDirectory)'
      publishRegistry: 'useFeed'
      publishFeed: 'MyProject/MyFeed'
```

### Consuming Packages

```yaml
# nuget.config
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="AzureArtifacts" value="https://pkgs.dev.azure.com/YourOrg/MyProject/_packaging/MyFeed/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

```yaml
# .npmrc for npm
registry=https://pkgs.dev.azure.com/YourOrg/MyProject/_packaging/MyFeed/npm/registry/
always-auth=true
```

---

## 11.9 Service Connections and Security

### What are Service Connections?
Service connections provide secure connections to external services like Azure, GitHub, Docker Hub, etc.

### Common Service Connection Types

| Type | Purpose |
|------|---------|
| **Azure Resource Manager** | Deploy to Azure |
| **GitHub** | Access GitHub repos |
| **Docker Registry** | Push/pull container images |
| **Kubernetes** | Deploy to Kubernetes |
| **SSH** | Connect via SSH |
| **Generic** | Custom integrations |

### Creating Azure Service Connection

```powershell
# Create service principal
$sp = New-AzADServicePrincipal -DisplayName "AzureDevOps-ServiceConnection"

# Get credentials
$spPassword = $sp.PasswordCredentials.SecretText
$spAppId = $sp.AppId
$tenantId = (Get-AzContext).Tenant.Id
$subscriptionId = (Get-AzContext).Subscription.Id

# Assign Contributor role
New-AzRoleAssignment `
    -ApplicationId $spAppId `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/$subscriptionId"
```

### Pipeline Security Best Practices

```yaml
# 1. Use pipeline permissions
trigger: none  # Only manual or scheduled triggers

# 2. Require approval for production
# (Configure in environment settings)

# 3. Limit who can edit pipeline
# (Configure in Azure DevOps security settings)

# 4. Use template validation
resources:
  repositories:
    - repository: templates
      type: git
      name: SharedTemplates
      ref: refs/heads/main

extends:
  template: secure-pipeline-template.yml@templates

# 5. Use checkout with limited depth
steps:
  - checkout: self
    clean: true
    fetchDepth: 1
    lfs: false
```

---

## 11.10 GitHub Actions Integration

### GitHub Actions vs Azure Pipelines

| Feature | Azure Pipelines | GitHub Actions |
|---------|----------------|----------------|
| **Hosting** | Azure DevOps | GitHub |
| **Syntax** | YAML | YAML |
| **Marketplace** | Tasks | Actions |
| **Self-hosted runners** | Yes | Yes |
| **Matrix builds** | Yes | Yes |

### GitHub Actions Workflow

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  AZURE_WEBAPP_NAME: myapp
  DOTNET_VERSION: '8.0.x'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Restore
        run: dotnet restore

      - name: Build
        run: dotnet build --no-restore --configuration Release

      - name: Test
        run: dotnet test --no-build --configuration Release

      - name: Publish
        run: dotnet publish -c Release -o ./publish

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: webapp
          path: ./publish

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: ${{ steps.deploy.outputs.webapp-url }}
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: webapp

      - name: Deploy to Azure Web App
        id: deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}-staging
          slot-name: production
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_STAGING }}

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: ${{ steps.deploy.outputs.webapp-url }}
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: webapp

      - name: Deploy to Azure Web App
        id: deploy
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE_PROD }}
```

---

## 11.11 Hands-On Lab: Complete CI/CD Pipeline

### Lab Scenario
Create a complete CI/CD pipeline for a .NET Web API that:
1. Builds and tests on every commit
2. Deploys to Development on develop branch
3. Deploys to Production with approval on main branch

### Step 1: Create Project Structure

```
MyWebApi/
├── src/
│   └── MyWebApi/
│       ├── Controllers/
│       │   └── WeatherController.cs
│       ├── Program.cs
│       └── MyWebApi.csproj
├── tests/
│   └── MyWebApi.Tests/
│       ├── WeatherControllerTests.cs
│       └── MyWebApi.Tests.csproj
├── azure-pipelines.yml
└── MyWebApi.sln
```

### Step 2: Create Pipeline File

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop

pr:
  branches:
    include:
      - main

variables:
  - name: buildConfiguration
    value: 'Release'
  - name: vmImageName
    value: 'ubuntu-latest'
  - name: azureSubscription
    value: 'MyAzureSubscription'

stages:
  # ===== BUILD STAGE =====
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Build
        displayName: 'Build Job'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: UseDotNet@2
            displayName: 'Install .NET SDK'
            inputs:
              version: '8.0.x'

          - script: dotnet build --configuration $(buildConfiguration)
            displayName: 'dotnet build'

          - script: dotnet test --configuration $(buildConfiguration) --logger trx --results-directory $(Agent.TempDirectory)
            displayName: 'dotnet test'

          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'VSTest'
              testResultsFiles: '**/*.trx'
              searchFolder: '$(Agent.TempDirectory)'

          - script: dotnet publish src/MyWebApi/MyWebApi.csproj -c $(buildConfiguration) -o $(Build.ArtifactStagingDirectory)
            displayName: 'dotnet publish'

          - publish: $(Build.ArtifactStagingDirectory)
            artifact: drop

  # ===== DEPLOY TO DEV =====
  - stage: DeployDev
    displayName: 'Deploy to Development'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
    jobs:
      - deployment: Deploy
        displayName: 'Deploy to Dev'
        environment: Development
        pool:
          vmImage: $(vmImageName)
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Azure App Service'
                  inputs:
                    azureSubscription: $(azureSubscription)
                    appType: 'webAppLinux'
                    appName: 'mywebapi-dev'
                    package: '$(Pipeline.Workspace)/drop'

  # ===== DEPLOY TO PROD =====
  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: Deploy
        displayName: 'Deploy to Production'
        environment: Production  # Configure approval in Azure DevOps
        pool:
          vmImage: $(vmImageName)
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  displayName: 'Deploy to Azure App Service'
                  inputs:
                    azureSubscription: $(azureSubscription)
                    appType: 'webAppLinux'
                    appName: 'mywebapi-prod'
                    package: '$(Pipeline.Workspace)/drop'
```

---

## 11.12 Summary and Best Practices

### DevOps Integration Engineer Best Practices

| Category | Best Practice |
|----------|---------------|
| **Pipeline Design** | Use YAML pipelines, templates, and infrastructure as code |
| **Security** | Use service connections, variable groups with Key Vault, and least privilege |
| **Testing** | Implement unit, integration, and smoke tests in pipelines |
| **Deployment** | Use environments, approvals, and deployment strategies |
| **Monitoring** | Integrate Application Insights and Azure Monitor |
| **Documentation** | Document pipelines, environments, and deployment procedures |

### Key Takeaways for DevOps Engineers

1. **Pipeline as Code**: Always use YAML pipelines for version control
2. **Environment Management**: Create distinct environments with appropriate approvals
3. **Secret Management**: Never hardcode secrets; use Key Vault integration
4. **Template Reuse**: Create templates for common patterns
5. **Deployment Strategies**: Choose appropriate strategy (rolling, canary, blue-green)
6. **Monitoring**: Always include health checks and monitoring

---

## Practice Questions

1. What is the difference between Continuous Delivery and Continuous Deployment?
2. How do you implement a blue-green deployment in Azure DevOps?
3. Explain the structure of a YAML pipeline.
4. What are the benefits of using pipeline templates?
5. How do you secure secrets in Azure Pipelines?
6. What is the purpose of environments in Azure DevOps?
7. How do you implement quality gates in your deployment pipeline?
8. Compare Azure Pipelines with GitHub Actions.

---

## Additional Resources

- [Azure DevOps Documentation](https://docs.microsoft.com/azure/devops/)
- [YAML Pipeline Schema Reference](https://docs.microsoft.com/azure/devops/pipelines/yaml-schema)
- [Azure DevOps Labs](https://www.azuredevopslabs.com/)
- [GitHub Actions Documentation](https://docs.github.com/actions)
