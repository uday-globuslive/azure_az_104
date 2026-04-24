# Module 12 - Azure Integration Services (Complete Guide: Beginner to Expert)

## Learning Objectives
By the end of this module, you will be able to:
- Design and implement Azure Logic Apps for workflow automation
- Configure and manage Azure API Management (APIM)
- Implement Azure Service Bus for enterprise messaging
- Build serverless solutions with Azure Functions
- Create data integration pipelines with Azure Data Factory
- Implement event-driven architectures with Azure Event Grid
- Design enterprise integration patterns
- Build scalable, resilient integration solutions

---

## 12.1 Introduction to Azure Integration Services

### What are Azure Integration Services?
Azure Integration Services is a collection of services that enable connecting applications, data, and processes both within and across organizations. These services help build event-driven, API-led, and serverless integration solutions.

### Azure Integration Platform Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     Azure Integration Services                               │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        Event Sources                                  │   │
│  │   Azure Services  │  External Systems  │  IoT Devices  │  Users      │   │
│  └────────────────────────────┬──────────────────────────────────────────┘   │
│                               │                                              │
│                               ▼                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                                                                         │ │
│  │  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌────────────┐ │ │
│  │  │ Event Grid  │   │ Service Bus │   │   API Mgmt  │   │  Functions │ │ │
│  │  │ (Events)    │   │ (Messages)  │   │   (APIs)    │   │(Serverless)│ │ │
│  │  └─────────────┘   └─────────────┘   └─────────────┘   └────────────┘ │ │
│  │                                                                         │ │
│  │  ┌─────────────┐   ┌─────────────┐                                     │ │
│  │  │ Logic Apps  │   │Data Factory │                                     │ │
│  │  │ (Workflows) │   │(ETL/ELT)    │                                     │ │
│  │  └─────────────┘   └─────────────┘                                     │ │
│  │                                                                         │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                               │                                              │
│                               ▼                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      Target Systems                                   │   │
│  │   Azure Services  │  On-Premises  │  SaaS Apps  │  Databases         │   │
│  └───────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### When to Use Which Service

| Scenario | Service | Reason |
|----------|---------|--------|
| Workflow automation | Logic Apps | Visual designer, 400+ connectors |
| Event distribution | Event Grid | Reactive, event-driven |

#### 🏢 Real-World Integration Use Cases:

**Order Processing System (Multi-Service Integration):**
```
Company: GlobalRetail (E-commerce, 1M orders/day)
Challenge: Process orders across 10 different systems

Integration Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                    Order Processing Flow                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Customer ──► API Management ──► Order API (Functions)          │
│                     │                    │                       │
│                     │                    ▼                       │
│                     │           Service Bus Queue                │
│                     │           (order-processing)               │
│                     │                    │                       │
│                     ▼                    ▼                       │
│              ┌──────────────────────────────────────┐           │
│              │         Logic App Workflow            │           │
│              │                                       │           │
│              │  1. Validate inventory (SAP)          │           │
│              │  2. Process payment (Stripe API)      │           │
│              │  3. Update CRM (Salesforce)           │           │
│              │  4. Create shipment (FedEx API)       │           │
│              │  5. Send confirmation (SendGrid)      │           │
│              │  6. Update analytics (Cosmos DB)      │           │
│              └──────────────────────────────────────┘           │
│                              │                                   │
│                              ▼                                   │
│              Event Grid ──► Notify inventory system             │
│                         ──► Trigger reporting                    │
│                         ──► Update customer portal               │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Orders processed: 1M/day (peak: 50K/hour)
├── End-to-end time: 30 seconds (was 5 minutes)
├── Error rate: 0.01% (automatic retry)
└── Cost: $3,000/month (vs. $30,000 for VMs always running)
```

**Real-Time Data Pipeline (Event-Driven):**
```
Company: SmartCity Traffic (IoT sensors, 10M events/day)
Challenge: Process traffic data in real-time for congestion management

Event-Driven Architecture:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Traffic Sensors ──► IoT Hub ──► Event Grid                   │
│   (10K sensors)         │              │                        │
│                         │              │                        │
│                         ▼              ▼                        │
│              ┌─────────────────┐  ┌─────────────────┐          │
│              │  Stream         │  │  Azure          │          │
│              │  Analytics      │  │  Functions      │          │
│              │  (Aggregation)  │  │  (Alerts)       │          │
│              └────────┬────────┘  └────────┬────────┘          │
│                       │                    │                    │
│                       ▼                    ▼                    │
│              ┌─────────────────────────────────────┐           │
│              │         Power BI Dashboard           │           │
│              │  • Real-time traffic map             │           │
│              │  • Congestion hotspots              │           │
│              │  • Predicted travel times            │           │
│              └─────────────────────────────────────┘           │
│                                                                  │
│   Alert Scenarios:                                               │
│   • Accident detected → Notify emergency services               │
│   • Congestion > threshold → Update navigation apps             │
│   • Sensor offline → Create maintenance ticket                   │
└─────────────────────────────────────────────────────────────────┘

Scale: 10M events/day, <5 second latency
Cost: $2,000/month (consumption-based pricing)
```

**B2B API Platform:**
```
Company: InsuranceCorp (100 partner integrations)
Challenge: Expose APIs to partners with security and rate limiting

API Management Solution:
┌─────────────────────────────────────────────────────────────────┐
│                    API Management Gateway                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Partner Categories:                                             │
│  ├── Gold Partners: 10,000 requests/minute, priority support   │
│  ├── Silver Partners: 1,000 requests/minute                     │
│  └── Bronze Partners: 100 requests/minute                       │
│                                                                  │
│  Security Policies:                                              │
│  ├── OAuth 2.0 authentication required                          │
│  ├── IP whitelisting for production access                      │
│  ├── Request validation (schema enforcement)                    │
│  └── Response masking (hide sensitive data)                     │
│                                                                  │
│  API Products:                                                   │
│  ├── Quote API: Get insurance quotes                            │
│  │   └── Rate limit: 1000/minute                                │
│  ├── Claims API: Submit and track claims                        │
│  │   └── Rate limit: 100/minute                                 │
│  └── Policy API: Policy CRUD operations                         │
│      └── Rate limit: 500/minute                                 │
│                                                                  │
│  Backend Services (hidden from partners):                        │
│  ├── Legacy mainframe (SOAP) → Transformed to REST              │
│  ├── New microservices (AKS)                                    │
│  └── Third-party APIs (rate limiting, caching)                  │
└─────────────────────────────────────────────────────────────────┘

Business Value:
├── Partner onboarding: 1 day (was 3 weeks)
├── API calls: 50M/month
├── Revenue from API products: $500K/month
└── Self-service developer portal: 24/7 availability
```

**Serverless ETL Pipeline:**
```
Company: MarketingAnalytics (Process data from 20 sources)
Challenge: Hourly data refresh from multiple SaaS apps

Data Factory Pipeline:
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   Sources:                     Transformations:                  │
│   ├── Salesforce CRM          ├── Data flows (Spark)           │
│   ├── HubSpot Marketing       ├── Mapping transforms           │
│   ├── Google Analytics        ├── Join/Aggregate               │
│   ├── Shopify Orders          └── Data quality rules           │
│   ├── Stripe Payments                                           │
│   └── 15 more...                                                │
│                                                                  │
│   Pipeline Orchestration:                                        │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Trigger: Every hour                                      │  │
│   │     │                                                      │  │
│   │     ▼                                                      │  │
│   │  ForEach source (parallel):                               │  │
│   │     │                                                      │  │
│   │     ├── Extract (incremental, last 1 hour)               │  │
│   │     │                                                      │  │
│   │     └── If errors > 0:                                    │  │
│   │         └── Retry 3 times, then alert                     │  │
│   │                                                            │  │
│   │  After all extractions:                                    │  │
│   │     │                                                      │  │
│   │     └── Transform (join, aggregate, cleanse)              │  │
│   │         │                                                  │  │
│   │         └── Load to Synapse Analytics                     │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Results:                                                       │
│   ├── Data freshness: 1 hour (was 24 hours)                    │
│   ├── Pipeline runtime: 15 minutes                              │
│   ├── Data volume: 500GB/day                                    │
│   └── Cost: $800/month (consumption-based)                      │
└─────────────────────────────────────────────────────────────────┘
```
| Enterprise messaging | Service Bus | Reliable, ordered messages |
| API publishing | API Management | Security, throttling, analytics |
| Serverless compute | Functions | Event-driven code execution |
| Data integration | Data Factory | ETL/ELT at scale |

---

## 12.2 Azure Logic Apps

### What is Azure Logic Apps?
Azure Logic Apps is a cloud-based platform for creating and running automated workflows that integrate apps, data, services, and systems.

### Logic Apps Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Logic App Workflow                           │
│                                                                      │
│  ┌─────────────────┐                                                │
│  │    Trigger      │  ← Starts the workflow                         │
│  │  (HTTP, Timer,  │    (Recurrence, HTTP request, Event Grid, etc.)│
│  │   Event Grid)   │                                                │
│  └────────┬────────┘                                                │
│           │                                                          │
│           ▼                                                          │
│  ┌─────────────────┐                                                │
│  │    Action 1     │  ← Get data from source                        │
│  │  (Get items)    │    (SQL, SharePoint, API, etc.)                │
│  └────────┬────────┘                                                │
│           │                                                          │
│           ▼                                                          │
│  ┌─────────────────┐                                                │
│  │   Condition     │  ← Branch logic                                │
│  │   (If/Then)     │                                                │
│  └────────┬────────┘                                                │
│     ┌─────┴─────┐                                                   │
│     ▼           ▼                                                   │
│  ┌──────┐   ┌──────┐                                                │
│  │ True │   │False │                                                │
│  │Action│   │Action│                                                │
│  └──────┘   └──────┘                                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Logic Apps Types

| Type | Hosting | Use Case |
|------|---------|----------|
| **Consumption** | Multi-tenant | Pay per execution, simple workflows |
| **Standard** | Single-tenant | Enterprise, complex workflows |
| **ISE (Integration Service Environment)** | Isolated | Compliance, dedicated capacity |

### Creating a Logic App

#### Using Azure Portal
1. Navigate to "Logic Apps"
2. Click "Add" → "Consumption" or "Standard"
3. Configure basics (Resource Group, Name, Region)
4. Use Logic App Designer to build workflow

#### Using Azure CLI

```bash
# Create Logic App (Consumption)
az logic workflow create \
    --resource-group "Integration-RG" \
    --name "OrderProcessingWorkflow" \
    --location "eastus" \
    --definition @workflow-definition.json

# Create Logic App (Standard)
az functionapp create \
    --resource-group "Integration-RG" \
    --name "StandardLogicApp" \
    --storage-account mystorageaccount \
    --plan MyAppServicePlan \
    --functions-version 4 \
    --runtime node
```

### Common Triggers

| Trigger Type | Description | Example |
|--------------|-------------|---------|
| **Recurrence** | Time-based schedule | Every 5 minutes |
| **HTTP Request** | Webhook endpoint | External system call |
| **Event Grid** | Azure events | Blob created |
| **Service Bus** | Message received | Queue message |
| **Blob Storage** | File operations | New file uploaded |
| **SQL Server** | Database changes | Row modified |

### Common Actions

| Action | Purpose |
|--------|---------|
| **HTTP** | Call any REST API |
| **Parse JSON** | Extract data from JSON |
| **Compose** | Create complex objects |
| **Filter Array** | Filter data collections |
| **For Each** | Loop through items |
| **Condition** | Branch logic |
| **Switch** | Multiple branches |
| **Scope** | Group actions |
| **Delay** | Wait for specified time |
| **Terminate** | End workflow |

### Logic App Workflow Definition (JSON)

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "manual": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "schema": {
                        "type": "object",
                        "properties": {
                            "orderId": {"type": "string"},
                            "customerEmail": {"type": "string"},
                            "amount": {"type": "number"}
                        }
                    }
                }
            }
        },
        "actions": {
            "Insert_Order": {
                "type": "ApiConnection",
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['sql']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/datasets/default/tables/@{encodeURIComponent(encodeURIComponent('[dbo].[Orders]'))}/items"
                },
                "runAfter": {}
            },
            "Send_Confirmation_Email": {
                "type": "ApiConnection",
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['office365']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/v2/Mail",
                    "body": {
                        "To": "@triggerBody()?['customerEmail']",
                        "Subject": "Order Confirmation",
                        "Body": "Your order @{triggerBody()?['orderId']} has been received."
                    }
                },
                "runAfter": {
                    "Insert_Order": ["Succeeded"]
                }
            }
        }
    }
}
```

### Error Handling in Logic Apps

```json
{
    "actions": {
        "Try_Action": {
            "type": "Scope",
            "actions": {
                "Risky_Operation": {
                    "type": "Http",
                    "inputs": {
                        "method": "POST",
                        "uri": "https://api.example.com/process"
                    }
                }
            }
        },
        "Catch_Errors": {
            "type": "Scope",
            "actions": {
                "Log_Error": {
                    "type": "Compose",
                    "inputs": "@result('Try_Action')"
                }
            },
            "runAfter": {
                "Try_Action": ["Failed", "Skipped", "TimedOut"]
            }
        }
    }
}
```

### Integration with DevOps

```yaml
# Deploy Logic App via Azure DevOps
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'MyAzureConnection'
    subscriptionId: '$(subscriptionId)'
    action: 'Create Or Update Resource Group'
    resourceGroupName: 'Integration-RG'
    location: 'East US'
    templateLocation: 'Linked artifact'
    csmFile: '$(System.DefaultWorkingDirectory)/templates/logicapp-template.json'
    csmParametersFile: '$(System.DefaultWorkingDirectory)/templates/logicapp-parameters.json'
```

---

## 12.3 Azure API Management (APIM)

### What is Azure API Management?
Azure API Management (APIM) is a hybrid, multi-cloud management platform for APIs across all environments. It helps organizations publish APIs to external, partner, and internal developers.

### APIM Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Azure API Management                                  │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                          API Gateway                                    │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │ │
│  │  │Rate Limiting│ │   Caching   │ │Transformation│ │  Security   │      │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘      │ │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐      │ │
│  │  │   Routing   │ │   Logging   │ │  Mocking    │ │  Monetization│     │ │
│  │  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘      │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│  ┌─────────────────┬───────────────┼───────────────┬─────────────────┐      │
│  │                 │               │               │                 │      │
│  ▼                 ▼               ▼               ▼                 ▼      │
│ ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐       │
│ │Backend 1│   │Backend 2│   │Function │   │ Logic   │   │External │       │
│ │ (App    │   │   (VM)  │   │  App    │   │  App    │   │   API   │       │
│ │Service) │   │         │   │         │   │         │   │         │       │
│ └─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘       │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                     Developer Portal                                    │ │
│  │  • API Documentation  • Interactive Console  • Subscription Keys       │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### APIM Components

| Component | Description |
|-----------|-------------|
| **API Gateway** | Entry point for API calls, applies policies |
| **Management Plane** | Configure APIs, policies, products |
| **Developer Portal** | Self-service portal for API consumers |

### APIM Tiers

| Tier | Features | Use Case |
|------|----------|----------|
| **Consumption** | Serverless, pay per call | Low traffic, prototyping |
| **Developer** | No SLA, single unit | Development, testing |
| **Basic** | No Developer Portal | Basic production |
| **Standard** | Full features | Production workloads |
| **Premium** | Multi-region, VNet | Enterprise, global |
| **Isolated** | Dedicated capacity | Compliance, high security |

### Creating APIM Instance

```powershell
# Create API Management instance
New-AzApiManagement `
    -ResourceGroupName "Integration-RG" `
    -Name "myapim" `
    -Location "East US" `
    -Organization "My Company" `
    -AdminEmail "admin@company.com" `
    -Sku "Standard" `
    -Capacity 1
```

```bash
# Azure CLI
az apim create \
    --name "myapim" \
    --resource-group "Integration-RG" \
    --location "eastus" \
    --publisher-name "My Company" \
    --publisher-email "admin@company.com" \
    --sku-name Standard
```

### Importing APIs

```bash
# Import from OpenAPI specification
az apim api import \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --api-id "my-api" \
    --path "myapi" \
    --specification-format "OpenAPI" \
    --specification-url "https://petstore.swagger.io/v2/swagger.json"

# Import from Azure Function
az apim api import \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --api-id "functions-api" \
    --path "functions" \
    --specification-format "OpenAPI" \
    --specification-path "./openapi.json" \
    --service-url "https://myfunctionapp.azurewebsites.net/api"
```

### API Policies

Policies are powerful XML-based expressions that modify API behavior.

```xml
<!-- Inbound policies - applied to requests -->
<policies>
    <inbound>
        <!-- Rate limiting -->
        <rate-limit calls="100" renewal-period="60" />
        
        <!-- Quota -->
        <quota calls="10000" renewal-period="86400" />
        
        <!-- IP restriction -->
        <ip-filter action="allow">
            <address-range from="10.0.0.0" to="10.0.0.255" />
        </ip-filter>
        
        <!-- Add header -->
        <set-header name="X-Request-ID" exists-action="override">
            <value>@(context.RequestId.ToString())</value>
        </set-header>
        
        <!-- Validate JWT -->
        <validate-jwt header-name="Authorization" require-scheme="Bearer">
            <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration" />
            <required-claims>
                <claim name="aud">
                    <value>api://myapi</value>
                </claim>
            </required-claims>
        </validate-jwt>
        
        <base />
    </inbound>
    
    <!-- Backend policies - applied before forwarding to backend -->
    <backend>
        <!-- Set backend URL dynamically -->
        <set-backend-service base-url="https://api.backend.com" />
        
        <base />
    </backend>
    
    <!-- Outbound policies - applied to responses -->
    <outbound>
        <!-- Transform response -->
        <set-body>@{
            var response = context.Response.Body.As<JObject>();
            response["timestamp"] = DateTime.UtcNow.ToString("o");
            return response.ToString();
        }</set-body>
        
        <!-- Cache response -->
        <cache-store duration="3600" />
        
        <base />
    </outbound>
    
    <!-- Error policies -->
    <on-error>
        <set-body>@{
            return new JObject(
                new JProperty("error", context.LastError.Message),
                new JProperty("code", context.LastError.Source)
            ).ToString();
        }</set-body>
        
        <base />
    </on-error>
</policies>
```

### Named Values (Configuration)

```bash
# Create named value
az apim nv create \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --named-value-id "backend-url" \
    --display-name "Backend URL" \
    --value "https://api.backend.com"

# Create secret from Key Vault
az apim nv create \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --named-value-id "api-key" \
    --display-name "API Key" \
    --secret true \
    --value "{{KeyVaultSecretName}}"
```

### Products and Subscriptions

```bash
# Create product
az apim product create \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --product-id "starter" \
    --product-name "Starter" \
    --description "Free tier with limited calls" \
    --subscription-required true \
    --approval-required false \
    --subscriptions-limit 1 \
    --state "published"

# Add API to product
az apim product api add \
    --resource-group "Integration-RG" \
    --service-name "myapim" \
    --product-id "starter" \
    --api-id "my-api"
```

---

## 12.4 Azure Service Bus

### What is Azure Service Bus?
Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics.

### Service Bus Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Azure Service Bus                                    │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                        Namespace                                         ││
│  │                                                                          ││
│  │  ┌─────────────────────────────┐  ┌──────────────────────────────────┐ ││
│  │  │          Queues              │  │           Topics                  │ ││
│  │  │                              │  │                                   │ ││
│  │  │  Sender ──► Queue ──► Receiver│  │  Publisher ──► Topic            │ ││
│  │  │                              │  │                  │               │ ││
│  │  │  Point-to-Point              │  │           ┌──────┴──────┐        │ ││
│  │  │  One consumer per message    │  │           ▼             ▼        │ ││
│  │  │                              │  │     Subscription   Subscription  │ ││
│  │  │  ┌────────────────────┐      │  │           │             │        │ ││
│  │  │  │ MSG │ MSG │ MSG    │      │  │     Subscriber1   Subscriber2    │ ││
│  │  │  └────────────────────┘      │  │                                  │ ││
│  │  └─────────────────────────────┘  └──────────────────────────────────┘ ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
```

### Service Bus Tiers

| Tier | Features | Use Case |
|------|----------|----------|
| **Basic** | Queues only | Simple messaging |
| **Standard** | Queues + Topics | General production |
| **Premium** | Dedicated capacity | Mission-critical |

### Queue vs Topic

| Feature | Queue | Topic |
|---------|-------|-------|
| **Pattern** | Point-to-point | Publish-subscribe |
| **Consumers** | Single | Multiple (subscriptions) |
| **Message delivery** | One consumer | Copy to each subscription |
| **Use case** | Task processing | Notifications, broadcasting |

### Creating Service Bus Resources

```powershell
# Create namespace
New-AzServiceBusNamespace `
    -ResourceGroupName "Integration-RG" `
    -Name "myservicebus" `
    -Location "East US" `
    -SkuName "Standard"

# Create queue
New-AzServiceBusQueue `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "myservicebus" `
    -Name "orders" `
    -EnablePartitioning $false `
    -MaxSizeInMegabytes 5120 `
    -DefaultMessageTimeToLive "P14D" `
    -LockDuration "PT5M" `
    -MaxDeliveryCount 10 `
    -EnableDeadLetteringOnMessageExpiration $true

# Create topic
New-AzServiceBusTopic `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "myservicebus" `
    -Name "notifications" `
    -MaxSizeInMegabytes 5120

# Create subscription
New-AzServiceBusSubscription `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "myservicebus" `
    -TopicName "notifications" `
    -Name "email-notifications" `
    -MaxDeliveryCount 10
```

### Working with Service Bus (.NET)

```csharp
// Azure.Messaging.ServiceBus NuGet package
using Azure.Messaging.ServiceBus;

// Connection string from Azure portal or Key Vault
string connectionString = "Endpoint=sb://myservicebus.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=...";

// SENDING MESSAGES
await using var client = new ServiceBusClient(connectionString);
ServiceBusSender sender = client.CreateSender("orders");

// Send single message
var message = new ServiceBusMessage("Order: 12345")
{
    ContentType = "application/json",
    MessageId = Guid.NewGuid().ToString(),
    CorrelationId = "correlation-123",
    Subject = "NewOrder",
    ApplicationProperties = { { "Priority", "High" } }
};
await sender.SendMessageAsync(message);

// Send batch
var messages = new List<ServiceBusMessage>
{
    new ServiceBusMessage("Order 1"),
    new ServiceBusMessage("Order 2"),
    new ServiceBusMessage("Order 3")
};
await sender.SendMessagesAsync(messages);

// RECEIVING MESSAGES
ServiceBusReceiver receiver = client.CreateReceiver("orders");

// Receive single message
ServiceBusReceivedMessage receivedMessage = await receiver.ReceiveMessageAsync();
string body = receivedMessage.Body.ToString();

// Complete message (remove from queue)
await receiver.CompleteMessageAsync(receivedMessage);

// Abandon message (release lock, retry)
await receiver.AbandonMessageAsync(receivedMessage);

// Dead-letter message (move to DLQ)
await receiver.DeadLetterMessageAsync(receivedMessage, "Invalid format");

// PROCESSOR (recommended for production)
ServiceBusProcessor processor = client.CreateProcessor("orders", new ServiceBusProcessorOptions
{
    MaxConcurrentCalls = 5,
    AutoCompleteMessages = false
});

processor.ProcessMessageAsync += async args =>
{
    string body = args.Message.Body.ToString();
    Console.WriteLine($"Received: {body}");
    
    // Process message...
    
    await args.CompleteMessageAsync(args.Message);
};

processor.ProcessErrorAsync += args =>
{
    Console.WriteLine($"Error: {args.Exception.Message}");
    return Task.CompletedTask;
};

await processor.StartProcessingAsync();
```

### Message Sessions (Ordered Processing)

```csharp
// Sessions guarantee FIFO within a session
var message = new ServiceBusMessage("Order item 1")
{
    SessionId = "order-123"  // All messages with same SessionId processed in order
};

// Receive session messages
ServiceBusSessionReceiver sessionReceiver = 
    await client.AcceptSessionAsync("order-queue", "order-123");

ServiceBusReceivedMessage sessionMessage = await sessionReceiver.ReceiveMessageAsync();
```

### Dead Letter Queue

```csharp
// Receive from dead letter queue
ServiceBusReceiver dlqReceiver = client.CreateReceiver(
    "orders", 
    new ServiceBusReceiverOptions { SubQueue = SubQueue.DeadLetter }
);

ServiceBusReceivedMessage deadMessage = await dlqReceiver.ReceiveMessageAsync();
Console.WriteLine($"Dead letter reason: {deadMessage.DeadLetterReason}");
Console.WriteLine($"Description: {deadMessage.DeadLetterErrorDescription}");
```

### Subscription Filters

```powershell
# SQL filter
New-AzServiceBusRule `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "myservicebus" `
    -TopicName "notifications" `
    -SubscriptionName "high-priority" `
    -Name "HighPriorityFilter" `
    -SqlExpression "Priority = 'High'"

# Correlation filter
New-AzServiceBusRule `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "myservicebus" `
    -TopicName "notifications" `
    -SubscriptionName "email-events" `
    -Name "EmailFilter" `
    -ContentType "application/json" `
    -CorrelationFilterProperty @{EventType = "Email"}
```

---

## 12.5 Azure Functions

### What is Azure Functions?
Azure Functions is a serverless compute service that lets you run event-triggered code without having to explicitly provision or manage infrastructure.

### Functions Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Azure Functions                                     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         Triggers                                        │ │
│  │  HTTP │ Timer │ Blob │ Queue │ Service Bus │ Event Grid │ Cosmos DB   │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                      Function App                                       │ │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                 │ │
│  │  │  Function 1  │  │  Function 2  │  │  Function 3  │                 │ │
│  │  │   (HTTP)     │  │   (Timer)    │  │  (Queue)     │                 │ │
│  │  └──────────────┘  └──────────────┘  └──────────────┘                 │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                        Output Bindings                                  │ │
│  │  Blob │ Queue │ Table │ Cosmos DB │ SendGrid │ Event Grid │ HTTP     │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Hosting Plans

| Plan | Scaling | Use Case |
|------|---------|----------|
| **Consumption** | Automatic, event-driven | Sporadic workloads |
| **Premium** | Pre-warmed, VNet | Enterprise, low latency |
| **Dedicated (App Service)** | Manual/auto | Predictable workloads |
| **Container Apps** | Kubernetes-based | Containerized functions |

### Creating Function App

```bash
# Create storage account (required)
az storage account create \
    --name "myfuncstorageacct" \
    --resource-group "Integration-RG" \
    --location "eastus" \
    --sku Standard_LRS

# Create Function App (Consumption)
az functionapp create \
    --name "myfunctionapp" \
    --resource-group "Integration-RG" \
    --storage-account "myfuncstorageacct" \
    --consumption-plan-location "eastus" \
    --runtime "dotnet-isolated" \
    --runtime-version "8" \
    --functions-version 4

# Create Function App (Premium)
az functionapp plan create \
    --name "MyPremiumPlan" \
    --resource-group "Integration-RG" \
    --location "eastus" \
    --sku EP1 \
    --is-linux

az functionapp create \
    --name "mypremiumfunc" \
    --resource-group "Integration-RG" \
    --storage-account "myfuncstorageacct" \
    --plan "MyPremiumPlan" \
    --runtime "dotnet-isolated" \
    --runtime-version "8" \
    --functions-version 4
```

### Function Code Examples (.NET)

#### HTTP Trigger

```csharp
// HTTP Trigger - REST API endpoint
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.Extensions.Logging;
using System.Net;

public class HttpTriggerFunction
{
    private readonly ILogger _logger;

    public HttpTriggerFunction(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HttpTriggerFunction>();
    }

    [Function("GetOrders")]
    public async Task<HttpResponseData> GetOrders(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "orders/{id?}")] HttpRequestData req,
        string? id)
    {
        _logger.LogInformation($"Processing order request for ID: {id}");

        var response = req.CreateResponse(HttpStatusCode.OK);
        await response.WriteAsJsonAsync(new { OrderId = id, Status = "Completed" });
        return response;
    }

    [Function("CreateOrder")]
    public async Task<HttpResponseData> CreateOrder(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = "orders")] HttpRequestData req)
    {
        var order = await req.ReadFromJsonAsync<OrderRequest>();
        
        // Process order...
        
        var response = req.CreateResponse(HttpStatusCode.Created);
        await response.WriteAsJsonAsync(new { OrderId = Guid.NewGuid(), Status = "Created" });
        return response;
    }
}
```

#### Service Bus Trigger

```csharp
// Service Bus Queue Trigger
public class ServiceBusQueueFunction
{
    private readonly ILogger _logger;

    public ServiceBusQueueFunction(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<ServiceBusQueueFunction>();
    }

    [Function("ProcessOrder")]
    public async Task ProcessOrder(
        [ServiceBusTrigger("orders", Connection = "ServiceBusConnection")] ServiceBusReceivedMessage message,
        ServiceBusMessageActions messageActions)
    {
        _logger.LogInformation($"Processing message: {message.MessageId}");
        
        try
        {
            var order = message.Body.ToObjectFromJson<Order>();
            
            // Process order logic...
            
            await messageActions.CompleteMessageAsync(message);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error processing message");
            await messageActions.DeadLetterMessageAsync(message, ex.Message);
        }
    }
}
```

#### Timer Trigger

```csharp
// Timer Trigger - Scheduled execution
public class TimerFunction
{
    private readonly ILogger _logger;

    public TimerFunction(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<TimerFunction>();
    }

    [Function("CleanupJob")]
    public void Run(
        [TimerTrigger("0 0 * * * *")] TimerInfo timerInfo)  // Every hour
    {
        _logger.LogInformation($"Cleanup job ran at: {DateTime.Now}");
        
        if (timerInfo.IsPastDue)
        {
            _logger.LogWarning("Timer is running late!");
        }
        
        // Cleanup logic...
    }
}

// CRON expressions:
// "0 */5 * * * *"     - Every 5 minutes
// "0 0 * * * *"       - Every hour
// "0 0 0 * * *"       - Every day at midnight
// "0 0 9 * * 1-5"     - 9 AM, Monday-Friday
// "0 0 0 1 * *"       - First day of every month
```

#### Blob Trigger with Output Binding

```csharp
// Blob Trigger with multiple outputs
public class BlobProcessFunction
{
    [Function("ProcessImage")]
    [BlobOutput("processed/{name}", Connection = "StorageConnection")]
    public async Task<byte[]> ProcessImage(
        [BlobTrigger("uploads/{name}", Connection = "StorageConnection")] byte[] imageData,
        string name,
        [QueueOutput("image-processed")] out string queueMessage)
    {
        // Process image (resize, etc.)
        var processedImage = ResizeImage(imageData);
        
        // Output to queue
        queueMessage = $"Processed: {name}";
        
        // Return to blob output
        return processedImage;
    }
}
```

### Function Configuration (local.settings.json)

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet-isolated",
    "ServiceBusConnection": "Endpoint=sb://...",
    "CosmosDBConnection": "AccountEndpoint=...",
    "APPINSIGHTS_INSTRUMENTATIONKEY": "..."
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*"
  }
}
```

### Durable Functions (Orchestration)

```csharp
// Orchestrator function for complex workflows
public class OrderOrchestration
{
    [Function("OrderOrchestrator")]
    public async Task<OrderResult> RunOrchestrator(
        [OrchestrationTrigger] TaskOrchestrationContext context)
    {
        var order = context.GetInput<Order>();
        
        // Step 1: Validate order
        var isValid = await context.CallActivityAsync<bool>("ValidateOrder", order);
        if (!isValid)
        {
            return new OrderResult { Status = "Invalid" };
        }
        
        // Step 2: Reserve inventory (with retry)
        var retryOptions = new TaskRetryOptions(
            firstRetryInterval: TimeSpan.FromSeconds(5),
            maxNumberOfAttempts: 3);
        
        var reserved = await context.CallActivityAsync<bool>(
            "ReserveInventory", 
            order,
            new TaskActivityOptions { Retry = retryOptions });
        
        // Step 3: Process payment
        var paymentResult = await context.CallActivityAsync<PaymentResult>("ProcessPayment", order);
        
        // Step 4: Send confirmation (fan-out)
        var notificationTasks = new List<Task>
        {
            context.CallActivityAsync("SendEmail", order),
            context.CallActivityAsync("SendSMS", order),
            context.CallActivityAsync("UpdateCRM", order)
        };
        await Task.WhenAll(notificationTasks);
        
        return new OrderResult { Status = "Completed", OrderId = order.Id };
    }

    [Function("ValidateOrder")]
    public bool ValidateOrder([ActivityTrigger] Order order)
    {
        // Validation logic
        return order.Items.Any() && order.TotalAmount > 0;
    }

    [Function("ReserveInventory")]
    public async Task<bool> ReserveInventory([ActivityTrigger] Order order)
    {
        // Reserve inventory logic
        return true;
    }

    [Function("ProcessPayment")]
    public async Task<PaymentResult> ProcessPayment([ActivityTrigger] Order order)
    {
        // Payment processing logic
        return new PaymentResult { Success = true };
    }
}
```

---

## 12.6 Azure Data Factory

### What is Azure Data Factory?
Azure Data Factory is a cloud-based data integration service that allows you to create data-driven workflows (pipelines) for orchestrating data movement and transforming data at scale.

### Data Factory Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Azure Data Factory                                    │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                        Data Sources                                     │ │
│  │  SQL │ Blob │ Cosmos │ Oracle │ SAP │ REST APIs │ Files │ Salesforce  │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         Pipeline                                        │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐               │ │
│  │  │  Copy    │→│Transform │→│  Data    │→│  Stored  │               │ │
│  │  │ Activity │  │ (Data   │  │  Flow   │  │  Proc    │               │ │
│  │  │          │  │  Flows) │  │         │  │          │               │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘               │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                       Data Destinations                                 │ │
│  │  Azure SQL │ Synapse │ Data Lake │ Blob │ Cosmos DB │ Data Warehouse  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  Triggers: Schedule │ Event │ Tumbling Window │ Manual                  ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  Integration Runtime: Azure │ Self-hosted │ Azure-SSIS                  ││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Pipeline** | Logical grouping of activities |
| **Activity** | Processing step (copy, transform, etc.) |
| **Dataset** | Named view of data |
| **Linked Service** | Connection to data store |
| **Trigger** | Determines when pipeline runs |
| **Integration Runtime** | Compute infrastructure |

### Creating Data Factory

```bash
# Create Data Factory
az datafactory create \
    --resource-group "Integration-RG" \
    --factory-name "mydatafactory" \
    --location "eastus"

# Create linked service (Azure SQL)
az datafactory linked-service create \
    --resource-group "Integration-RG" \
    --factory-name "mydatafactory" \
    --linked-service-name "AzureSqlLinkedService" \
    --properties @linked-service.json
```

### Pipeline Definition (JSON)

```json
{
    "name": "CopyPipeline",
    "properties": {
        "activities": [
            {
                "name": "CopyFromBlobToSQL",
                "type": "Copy",
                "inputs": [
                    {
                        "referenceName": "BlobDataset",
                        "type": "DatasetReference"
                    }
                ],
                "outputs": [
                    {
                        "referenceName": "SqlDataset",
                        "type": "DatasetReference"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "SqlSink",
                        "writeBehavior": "upsert",
                        "upsertSettings": {
                            "useTempDB": true,
                            "keys": ["Id"]
                        }
                    },
                    "enableStaging": false
                }
            },
            {
                "name": "StoredProcedure",
                "type": "SqlServerStoredProcedure",
                "dependsOn": [
                    {
                        "activity": "CopyFromBlobToSQL",
                        "dependencyConditions": ["Succeeded"]
                    }
                ],
                "typeProperties": {
                    "storedProcedureName": "sp_ProcessData",
                    "storedProcedureParameters": {
                        "BatchId": {
                            "type": "String",
                            "value": "@pipeline().RunId"
                        }
                    }
                },
                "linkedServiceName": {
                    "referenceName": "AzureSqlLinkedService",
                    "type": "LinkedServiceReference"
                }
            }
        ],
        "parameters": {
            "inputPath": {
                "type": "String"
            },
            "outputTable": {
                "type": "String"
            }
        }
    }
}
```

### Data Flows (Transformation)

```json
{
    "name": "TransformDataFlow",
    "properties": {
        "type": "MappingDataFlow",
        "typeProperties": {
            "sources": [
                {
                    "dataset": {
                        "referenceName": "SourceDataset",
                        "type": "DatasetReference"
                    },
                    "name": "source1"
                }
            ],
            "sinks": [
                {
                    "dataset": {
                        "referenceName": "SinkDataset",
                        "type": "DatasetReference"
                    },
                    "name": "sink1"
                }
            ],
            "transformations": [
                {
                    "name": "filter1",
                    "description": "Filter active records"
                },
                {
                    "name": "derivedColumn1",
                    "description": "Add calculated columns"
                },
                {
                    "name": "aggregate1",
                    "description": "Aggregate by region"
                }
            ],
            "scriptLines": [
                "source(allowSchemaDrift: true) ~> source1",
                "source1 filter(IsActive == true) ~> filter1",
                "filter1 derive(TotalAmount = Quantity * UnitPrice) ~> derivedColumn1",
                "derivedColumn1 aggregate(groupBy(Region), TotalSales = sum(TotalAmount)) ~> aggregate1",
                "aggregate1 sink(allowSchemaDrift: true) ~> sink1"
            ]
        }
    }
}
```

---

## 12.7 Azure Event Grid

### What is Azure Event Grid?
Azure Event Grid is a fully managed event routing service that enables event-driven architectures using a publish-subscribe model.

### Event Grid Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Azure Event Grid                                    │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                       Event Publishers                                  │ │
│  │  Azure Services │ Custom Apps │ Storage │ Resource Groups │ IoT Hub   │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                          Topics                                         │ │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐        │ │
│  │  │  System Topic   │  │  Custom Topic   │  │ Partner Topic   │        │ │
│  │  │ (Azure builtin) │  │  (Your apps)    │  │ (Third party)   │        │ │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────┘        │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                    Event Subscriptions                                  │ │
│  │  ┌──────────────────────────────────────────────────────────────────┐  │ │
│  │  │  Filters (by event type, by subject prefix/suffix, advanced)     │  │ │
│  │  └──────────────────────────────────────────────────────────────────┘  │ │
│  └───────────────────────────────┬────────────────────────────────────────┘ │
│                                  │                                           │
│                                  ▼                                           │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                      Event Handlers                                     │ │
│  │  Functions │ Logic Apps │ Webhooks │ Event Hubs │ Service Bus │ Queue  │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Event Schema

```json
{
    "topic": "/subscriptions/{subscription-id}/resourceGroups/Integration-RG/providers/Microsoft.Storage/storageAccounts/mystorageaccount",
    "subject": "/blobServices/default/containers/uploads/blobs/image.jpg",
    "eventType": "Microsoft.Storage.BlobCreated",
    "eventTime": "2024-01-15T10:30:00.000Z",
    "id": "unique-event-id",
    "data": {
        "api": "PutBlob",
        "clientRequestId": "request-id",
        "requestId": "request-id",
        "eTag": "0x8D...",
        "contentType": "image/jpeg",
        "contentLength": 524288,
        "blobType": "BlockBlob",
        "url": "https://mystorageaccount.blob.core.windows.net/uploads/image.jpg"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
}
```

### Creating Event Grid Resources

```bash
# Create custom topic
az eventgrid topic create \
    --name "myeventtopic" \
    --resource-group "Integration-RG" \
    --location "eastus"

# Create system topic for Storage
az eventgrid system-topic create \
    --name "storage-events" \
    --resource-group "Integration-RG" \
    --source "/subscriptions/{sub}/resourceGroups/Integration-RG/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
    --topic-type "Microsoft.Storage.StorageAccounts" \
    --location "eastus"

# Create event subscription (to Azure Function)
az eventgrid event-subscription create \
    --name "blob-created-subscription" \
    --source-resource-id "/subscriptions/{sub}/resourceGroups/Integration-RG/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
    --endpoint "https://myfunctionapp.azurewebsites.net/runtime/webhooks/eventgrid?functionName=ProcessBlob&code=..." \
    --endpoint-type "webhook" \
    --included-event-types "Microsoft.Storage.BlobCreated" \
    --subject-begins-with "/blobServices/default/containers/uploads/"
```

### Publishing Custom Events

```csharp
// Using Azure.Messaging.EventGrid NuGet package
using Azure;
using Azure.Messaging.EventGrid;

// Create client
string topicEndpoint = "https://myeventtopic.eastus-1.eventgrid.azure.net/api/events";
string topicKey = "your-topic-key";
var client = new EventGridPublisherClient(new Uri(topicEndpoint), new AzureKeyCredential(topicKey));

// Create and publish event
var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "orders/12345",
        eventType: "Order.Created",
        dataVersion: "1.0",
        data: new
        {
            OrderId = "12345",
            CustomerId = "C001",
            Amount = 199.99,
            Items = new[] { "Product A", "Product B" }
        })
};

await client.SendEventsAsync(events);
```

### Handling Events in Azure Function

```csharp
public class EventGridTriggerFunction
{
    private readonly ILogger _logger;

    public EventGridTriggerFunction(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<EventGridTriggerFunction>();
    }

    [Function("ProcessBlobCreated")]
    public void Run([EventGridTrigger] EventGridEvent eventGridEvent)
    {
        _logger.LogInformation($"Event Type: {eventGridEvent.EventType}");
        _logger.LogInformation($"Subject: {eventGridEvent.Subject}");
        _logger.LogInformation($"Data: {eventGridEvent.Data}");
        
        if (eventGridEvent.EventType == "Microsoft.Storage.BlobCreated")
        {
            var data = eventGridEvent.Data.ToObjectFromJson<BlobCreatedEventData>();
            _logger.LogInformation($"Blob URL: {data.Url}");
            
            // Process the blob...
        }
    }
}
```

---

## 12.8 Enterprise Integration Patterns

### Common Integration Patterns

#### 1. Message Queue Pattern

```
┌──────────┐     ┌───────────────┐     ┌──────────┐
│ Producer │ ──► │ Service Bus   │ ──► │ Consumer │
│          │     │    Queue      │     │          │
└──────────┘     └───────────────┘     └──────────┘

Use cases:
- Task distribution
- Load leveling
- Decoupling systems
```

#### 2. Publish-Subscribe Pattern

```
                    ┌──────────────┐ ──► Consumer 1
                   ↗│ Subscription │
┌──────────┐      │ └──────────────┘
│Publisher │ ──► Topic
└──────────┘      │ ┌──────────────┐
                   ↘│ Subscription │ ──► Consumer 2
                    └──────────────┘

Use cases:
- Event notification
- Broadcasting
- Fan-out scenarios
```

#### 3. Request-Reply Pattern

```
┌──────────┐     Request      ┌──────────┐
│  Client  │ ─────────────►  │  Service │
│          │     Reply        │          │
│          │ ◄─────────────  │          │
└──────────┘                  └──────────┘

Implementation with Service Bus:
- Use correlationId for matching
- Reply to ReplyTo queue
```

#### 4. Saga Pattern (Distributed Transactions)

```
┌──────────────────────────────────────────────────────────────┐
│                    Saga Orchestrator                          │
│                                                               │
│  Step 1: Create Order  ─────────────────────────────────────►│
│          Success? ────► Step 2: Reserve Inventory            │
│                                  Success? ────► Step 3: Payment
│                                                    Success? ──│─► Complete
│                                                    Failure? ──│─► Compensate
│                                  Failure? ────────────────────├─► Compensate
│          Failure? ────────────────────────────────────────────┴─► Compensate
└──────────────────────────────────────────────────────────────┘

Implementation: Durable Functions or Logic Apps
```

### Complete Integration Solution Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Order Processing System Architecture                      │
│                                                                              │
│  ┌──────────┐                                                               │
│  │  Client  │                                                               │
│  │  Apps    │                                                               │
│  └────┬─────┘                                                               │
│       │                                                                      │
│       ▼                                                                      │
│  ┌──────────────┐                                                           │
│  │ API Mgmt     │  (Rate limiting, Authentication, Logging)                │
│  └──────┬───────┘                                                           │
│         │                                                                    │
│         ▼                                                                    │
│  ┌──────────────┐                                                           │
│  │ Azure        │  (Validate, Create Order, Return Response)               │
│  │ Functions    │                                                           │
│  └──────┬───────┘                                                           │
│         │                                                                    │
│         ▼                                                                    │
│  ┌──────────────┐                                                           │
│  │ Event Grid   │  (Publish OrderCreated event)                            │
│  └──────┬───────┘                                                           │
│         │                                                                    │
│    ┌────┴────┬─────────────┬─────────────┐                                  │
│    ▼         ▼             ▼             ▼                                  │
│  ┌────┐   ┌────────┐   ┌────────┐   ┌────────┐                             │
│  │Inv.│   │Payment │   │ Email  │   │ Data   │                             │
│  │Svc │   │ Svc    │   │ Svc    │   │Factory │                             │
│  └────┘   └────────┘   └────────┘   └────────┘                             │
│                                          │                                   │
│                                          ▼                                   │
│                                    ┌──────────┐                             │
│                                    │Data Lake │                             │
│                                    │Analytics │                             │
│                                    └──────────┘                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 12.9 DevOps for Integration Services

### CI/CD for Logic Apps

```yaml
# azure-pipelines.yml for Logic Apps
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  azureSubscription: 'MyAzureSubscription'
  resourceGroup: 'Integration-RG'
  location: 'eastus'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - task: CopyFiles@2
            inputs:
              sourceFolder: '$(Build.SourcesDirectory)/logicapps'
              contents: '**'
              targetFolder: '$(Build.ArtifactStagingDirectory)'

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'logicapps'

  - stage: DeployDev
    jobs:
      - deployment: Deploy
        environment: 'Development'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureResourceManagerTemplateDeployment@3
                  inputs:
                    deploymentScope: 'Resource Group'
                    azureResourceManagerConnection: '$(azureSubscription)'
                    subscriptionId: '$(subscriptionId)'
                    action: 'Create Or Update Resource Group'
                    resourceGroupName: '$(resourceGroup)-dev'
                    location: '$(location)'
                    templateLocation: 'Linked artifact'
                    csmFile: '$(Pipeline.Workspace)/logicapps/template.json'
                    csmParametersFile: '$(Pipeline.Workspace)/logicapps/parameters.dev.json'
```

### CI/CD for Azure Functions

```yaml
# azure-pipelines.yml for Functions
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - task: UseDotNet@2
            inputs:
              version: '8.0.x'

          - script: dotnet build --configuration Release
            displayName: 'Build'

          - script: dotnet test --configuration Release
            displayName: 'Test'

          - task: DotNetCoreCLI@2
            displayName: 'Publish'
            inputs:
              command: 'publish'
              publishWebProjects: false
              projects: '**/*.csproj'
              arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'

          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'functions'

  - stage: Deploy
    jobs:
      - deployment: Deploy
        environment: 'Production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureFunctionApp@1
                  inputs:
                    azureSubscription: 'MySubscription'
                    appType: 'functionApp'
                    appName: 'myfunctionapp'
                    package: '$(Pipeline.Workspace)/functions/**/*.zip'
                    deploymentMethod: 'zipDeploy'
```

### CI/CD for API Management

```yaml
# Deploy API to APIM
- task: AzurePowerShell@5
  displayName: 'Deploy API to APIM'
  inputs:
    azureSubscription: 'MySubscription'
    ScriptType: 'InlineScript'
    Inline: |
      $context = New-AzApiManagementContext -ResourceGroupName "Integration-RG" -ServiceName "myapim"
      
      # Import API from OpenAPI
      Import-AzApiManagementApi `
          -Context $context `
          -SpecificationFormat "OpenAPI" `
          -SpecificationPath "$(Pipeline.Workspace)/api/openapi.yaml" `
          -Path "myapi" `
          -ApiId "my-api"
      
      # Apply policy
      Set-AzApiManagementPolicy `
          -Context $context `
          -ApiId "my-api" `
          -PolicyFilePath "$(Pipeline.Workspace)/api/policies/api-policy.xml"
    azurePowerShellVersion: 'LatestVersion'
```

---

## 12.10 Monitoring and Troubleshooting

### Monitoring Integration Services

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Integration Monitoring Architecture                     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                        Azure Monitor                                    │ │
│  │                                                                         │ │
│  │  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐     │ │
│  │  │   Log Analytics  │  │ Application      │  │     Alerts       │     │ │
│  │  │   Workspace      │  │    Insights      │  │                  │     │ │
│  │  │                  │  │                  │  │ • Metric alerts  │     │ │
│  │  │ • Diagnostic logs│  │ • Traces         │  │ • Log alerts     │     │ │
│  │  │ • Activity logs  │  │ • Dependencies   │  │ • Action groups  │     │ │
│  │  │ • Custom logs    │  │ • Failures       │  │                  │     │ │
│  │  └──────────────────┘  └──────────────────┘  └──────────────────┘     │ │
│  │                              │                                          │ │
│  │                              ▼                                          │ │
│  │  ┌───────────────────────────────────────────────────────────────────┐ │ │
│  │  │                     Dashboards & Workbooks                         │ │ │
│  │  └───────────────────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Metrics to Monitor

| Service | Key Metrics |
|---------|-------------|
| **Logic Apps** | Runs, success rate, latency, failures |
| **Functions** | Executions, duration, failures, memory |
| **Service Bus** | Messages, active connections, DLQ |
| **API Management** | Requests, latency, errors, bandwidth |
| **Event Grid** | Published events, delivery successes/failures |
| **Data Factory** | Pipeline runs, activity duration, failures |

### KQL Queries for Integration

```kusto
// Logic App failures in last 24 hours
AzureDiagnostics
| where ResourceType == "WORKFLOWS"
| where status_s == "Failed"
| where TimeGenerated > ago(24h)
| summarize count() by resource_workflowName_s, error_message_s
| order by count_ desc

// Function App execution times
FunctionAppLogs
| where TimeGenerated > ago(1h)
| where Category == "Function.MyFunction"
| summarize avg(DurationMs), max(DurationMs), count() by bin(TimeGenerated, 5m)
| render timechart

// Service Bus dead letter queue messages
AzureDiagnostics
| where ResourceType == "NAMESPACES"
| where OperationName == "Send"
| where isnotnull(deadletterreason_s)
| summarize count() by deadletterreason_s, Resource

// API Management failed requests
ApiManagementGatewayLogs
| where TimeGenerated > ago(1h)
| where ResponseCode >= 400
| summarize count() by ApiId, OperationId, ResponseCode
| order by count_ desc
```

---

## 12.11 Summary and Best Practices

### Integration Best Practices

| Area | Best Practice |
|------|---------------|
| **Architecture** | Use event-driven patterns, decouple systems |
| **Reliability** | Implement retry policies, dead-letter queues |
| **Security** | Use managed identities, encrypt data in transit |
| **Monitoring** | Enable diagnostics, set up alerts |
| **DevOps** | Infrastructure as code, CI/CD pipelines |
| **Performance** | Use async patterns, optimize batch sizes |

### Key Takeaways for DevOps Integration Engineers

1. **Choose the right service** for each integration scenario
2. **Design for failure** with retry policies and error handling
3. **Implement observability** with comprehensive logging and monitoring
4. **Automate deployments** using Infrastructure as Code
5. **Follow security best practices** with managed identities and encryption
6. **Build reusable patterns** for common integration scenarios

---

## Practice Questions

1. When would you use Event Grid vs Service Bus?
2. How do you implement a saga pattern using Azure services?
3. What are the different hosting plans for Azure Functions?
4. How do you secure APIs in API Management?
5. Explain the difference between Consumption and Standard Logic Apps.
6. How do you handle dead-letter messages in Service Bus?
7. What are the key components of a Data Factory pipeline?

---

## Additional Resources

- [Azure Integration Services Documentation](https://docs.microsoft.com/azure/integration-services/)
- [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/)
- [Azure Functions Best Practices](https://docs.microsoft.com/azure/azure-functions/functions-best-practices)
- [API Management Policies Reference](https://docs.microsoft.com/azure/api-management/api-management-policies)
