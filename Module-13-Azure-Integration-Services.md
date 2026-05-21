# Module 13 - Azure Integration Services
## (Service Bus, Event Grid, API Management, Functions, Logic Apps)

> **Beginner Note**: Think of "integration" as connecting different applications and systems so they can talk to each other. Just like how WhatsApp connects you to your friends, Azure Integration Services connect different software applications together.

---

## Learning Objectives
By the end of this module, you will be able to:
- Understand what middleware/integration platforms are
- Explain Azure Service Bus and when to use it
- Explain Azure Event Grid and when to use it
- Understand Azure API Management (APIM)
- Build workflows with Azure Logic Apps
- Create serverless functions with Azure Functions
- Know which service to use in which scenario

---

## 13.1 What is Integration / Middleware?

### Real-World Analogy

Think about a busy restaurant:

```
WITHOUT integration (chaos):
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Customer │  │ Customer │  │ Customer │
│  shouts  │  │  shouts  │  │  shouts  │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │              │              │
     └──────────────┼──────────────┘
                    ▼
            ┌──────────────┐
            │    Kitchen   │  ← overwhelmed, chaos
            └──────────────┘

WITH integration (a waiter = middleware):
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Customer │  │ Customer │  │ Customer │
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │              │              │
     └──────────────┼──────────────┘
                    ▼
            ┌──────────────┐
            │    Waiter    │  ← middleware: collects, organizes, routes
            └──────┬───────┘
                   │
            ┌──────▼───────┐
            │   Kitchen    │  ← processes in order, organized
            └──────────────┘
```

**Middleware** = Software that sits between applications to help them communicate, translate data, route messages, and manage the flow.

### Why Integration Matters

| Problem | Integration Solution |
|---------|---------------------|
| App A uses JSON, App B uses XML | Middleware transforms the data |
| App B is slow and App A can't wait | Middleware queues messages |
| 100 apps all call App C directly | Middleware routes and distributes |
| App C goes down | Middleware holds messages until it's back |

### Types of Integration Patterns

```
1. SYNCHRONOUS (Request-Response):
   App A ──── sends request ────► App B
   App A ◄─── waits for reply ──── App B
   (Like a phone call - you wait for an answer)

2. ASYNCHRONOUS (Fire and Forget):
   App A ──── drops message in queue ────► Queue
   App A continues doing other work
   App B picks up message from queue later
   (Like sending an email - you don't wait)

3. EVENT-DRIVEN:
   Something happens (event) ──► Event published
   Multiple subscribers get notified
   (Like a news broadcast - many people receive at once)
```

---

## 13.2 Azure Service Bus

### What is Azure Service Bus?

Azure Service Bus is a **fully managed enterprise message broker**. Think of it as a super-reliable post office for your applications.

```
REAL-WORLD ANALOGY:
A Post Office (Service Bus)

App A (sender)                           App B (receiver)
┌──────────┐                            ┌──────────┐
│ "I need  │──── drops letter ────►     │ Picks up │
│ to send  │     in mailbox             │ letter   │
│ an order"│                            │ when     │
└──────────┘                            │ ready    │
                                        └──────────┘
        ┌─────────────────────────────────────┐
        │          POST OFFICE                 │
        │         (Service Bus)                │
        │                                      │
        │  ┌──────────────────────────────┐   │
        │  │    QUEUE (one receiver)       │   │
        │  │    ████████████████          │   │
        │  │    Message1 Message2 Message3 │   │
        │  └──────────────────────────────┘   │
        │                                      │
        │  ┌──────────────────────────────┐   │
        │  │    TOPIC (many receivers)     │   │
        │  │    ████████─────────►Sub A   │   │
        │  │    (one message)────────►Sub B│   │
        │  │             └────────────►Sub C│   │
        │  └──────────────────────────────┘   │
        └─────────────────────────────────────┘
```

### Queue vs Topic - Simple Explanation

| Feature | Queue | Topic |
|---------|-------|-------|
| **Receivers** | ONE receiver picks up each message | MULTIPLE receivers get a copy |
| **Analogy** | Mailbox - one person picks up mail | WhatsApp Broadcast - everyone gets a copy |
| **Use Case** | Order processing (one worker per order) | Notifications (send to email AND SMS AND dashboard) |

### Key Service Bus Features

| Feature | What it Does | Why it Matters |
|---------|-------------|----------------|
| **Message persistence** | Stores messages if receiver is offline | No message is lost |
| **FIFO** | First In, First Out ordering | Messages processed in order |
| **Dead-letter queue** | Failed messages go here for investigation | Nothing is silently lost |
| **Message lock** | Only one consumer processes each message | No duplicate processing |
| **Sessions** | Group related messages together | Process all messages for one order together |
| **Retry** | Automatically retries failed messages | Handles temporary failures |

### When to Use Service Bus

```
USE SERVICE BUS WHEN:
✓ You need guaranteed message delivery
✓ Order of messages matters
✓ Receiver might be temporarily down
✓ One receiver should handle each message (queue)
✓ Multiple apps need the same message (topic)
✓ You need transactions (all or nothing)

REAL EXAMPLES:
• E-commerce: Order placed → Service Bus Queue → Fulfillment service
• Banking: Payment transaction messages
• Ticket booking: Ensure seats aren't double-booked
```

### Creating Service Bus

```powershell
# Create Service Bus Namespace
New-AzServiceBusNamespace `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "mycompany-servicebus" `
    -Location "East US" `
    -SkuName "Standard"

# Create a Queue
New-AzServiceBusQueue `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "mycompany-servicebus" `
    -QueueName "order-processing" `
    -MaxDeliveryCount 3 `              # Try 3 times before dead-letter
    -LockDuration "00:01:00" `         # Lock message for 1 minute while processing
    -DefaultMessageTimeToLive "P7D"    # Message expires after 7 days

# Create a Topic
New-AzServiceBusTopic `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "mycompany-servicebus" `
    -TopicName "order-events"

# Create Subscriptions for the Topic
New-AzServiceBusSubscription `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "mycompany-servicebus" `
    -TopicName "order-events" `
    -SubscriptionName "email-notifications"

New-AzServiceBusSubscription `
    -ResourceGroupName "Integration-RG" `
    -NamespaceName "mycompany-servicebus" `
    -TopicName "order-events" `
    -SubscriptionName "inventory-update"
```

### Service Bus in Code (Simple Example)

```csharp
// SENDING a message (C# example)
var client = new ServiceBusClient("connection-string");
var sender = client.CreateSender("order-processing");

var order = new { OrderId = "ORD-123", Amount = 99.99 };
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));

await sender.SendMessageAsync(message);
Console.WriteLine("Order message sent to Service Bus!");

// RECEIVING a message
var receiver = client.CreateReceiver("order-processing");
var receivedMessage = await receiver.ReceiveMessageAsync();

Console.WriteLine($"Processing: {receivedMessage.Body}");
await receiver.CompleteMessageAsync(receivedMessage); // Mark as done
```

### Service Bus Tiers

| Tier | Features | Best For |
|------|---------|---------|
| **Basic** | Queues only, 256KB messages | Dev/Test |
| **Standard** | Queues + Topics, 256KB | Most apps |
| **Premium** | Large messages (100MB), VNet, dedicated | Enterprise, production |

---

## 13.3 Azure Event Grid

### What is Azure Event Grid?

Azure Event Grid is a **fully managed event routing service**. Think of it as a notification system - when something happens in Azure, Event Grid broadcasts it to whoever cares about that event.

```
REAL-WORLD ANALOGY:
Like a NEWSPAPER:

EVENT HAPPENS:          EVENT GRID:            SUBSCRIBERS:
┌─────────────┐        ┌─────────────┐        ┌─────────────┐
│ User uploads│        │ Event Grid  │        │  Function   │
│ a photo     │──────► │ (Publisher) │──────► │ (resize)    │
└─────────────┘        └──────┬──────┘        └─────────────┘
                              │               ┌─────────────┐
                              └─────────────► │  Logic App  │
                                              │ (notify)    │
                                              └─────────────┘
                                              ┌─────────────┐
                                              └─────────────►│  Webhook   │
                                                             │ (3rd party)│
                                                             └─────────────┘
```

### Service Bus vs Event Grid - Key Difference

```
SERVICE BUS = "Hey, do this task for me"
(Heavy messages, commands, guaranteed delivery, one-to-one or one-to-many)

EVENT GRID = "Hey, something just happened, FYI"
(Lightweight notifications, events, broadcast, near-real-time)

Example:
Service Bus: "Process this order" (command, needs doing)
Event Grid: "A new file was uploaded" (event, happened)
```

### Event Grid Concepts

| Term | Simple Explanation |
|------|--------------------|
| **Event Source** | Where the event comes from (Azure Storage, your app, etc.) |
| **Topic** | The channel where events are published |
| **Subscription** | "I want to receive events from this topic" |
| **Handler** | What receives and processes the event |
| **Filter** | "Only send me events matching this condition" |

### Built-in Event Sources

```
Azure services that automatically publish events to Event Grid:

┌─────────────────────────────────────────────────────────────┐
│                   Event Grid Event Sources                   │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Blob Storage│  │Resource Group│  │ Event Hub    │      │
│  │ "file.jpg    │  │ "VM created" │  │ "data stream │      │
│  │  uploaded"   │  │              │  │  captured"   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Service Bus  │  │ Container    │  │ Custom Topic │      │
│  │ "queue empty"│  │ Registry     │  │ "your own    │      │
│  │              │  │ "image push" │  │  events"     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Creating Event Grid

```powershell
# Create a custom Event Grid Topic
New-AzEventGridTopic `
    -ResourceGroupName "Integration-RG" `
    -Name "order-events-topic" `
    -Location "East US"

# Get the topic endpoint and key
$topic = Get-AzEventGridTopic -ResourceGroupName "Integration-RG" -Name "order-events-topic"
$keys = Get-AzEventGridTopicKey -ResourceGroupName "Integration-RG" -Name "order-events-topic"

Write-Host "Endpoint: $($topic.Endpoint)"
Write-Host "Key: $($keys.Key1)"

# Create an Event Subscription (subscribe an Azure Function to receive events)
New-AzEventGridSubscription `
    -ResourceGroupName "Integration-RG" `
    -TopicName "order-events-topic" `
    -EventSubscriptionName "function-subscription" `
    -EndpointType "AzureFunction" `
    -Endpoint "/subscriptions/{sub-id}/resourceGroups/Integration-RG/providers/Microsoft.Web/sites/myFunctionApp/functions/ProcessOrder"
```

### Publishing Events

```csharp
// Publishing a custom event to Event Grid
var client = new EventGridPublisherClient(
    new Uri("https://order-events-topic.eastus-1.eventgrid.azure.net"),
    new AzureKeyCredential("your-key")
);

var events = new List<EventGridEvent>
{
    new EventGridEvent(
        subject: "orders/ORD-123",
        eventType: "Order.Placed",
        dataVersion: "1.0",
        data: new { OrderId = "ORD-123", Amount = 99.99, Customer = "John Doe" }
    )
};

await client.SendEventsAsync(events);
Console.WriteLine("Event published to Event Grid!");
```

### When to Use Event Grid

```
USE EVENT GRID WHEN:
✓ React to events that happen in Azure (file uploaded, VM created)
✓ Fan-out notifications (one event → many handlers)
✓ Near-real-time notifications
✓ Serverless event-driven architectures
✓ IoT events, custom application events

REAL EXAMPLES:
• Resize image when photo uploaded to Blob Storage
• Send welcome email when new user registers
• Trigger CI/CD pipeline when code is pushed
• Alert team when Azure resource is deleted
```

---

## 13.4 Azure API Management (APIM)

### What is API Management?

An **API** is like a waiter in a restaurant — it takes your order (request) and brings back the food (response). **API Management** is like the restaurant manager who controls all the waiters, sets the rules, monitors orders, and protects the kitchen.

```
WITHOUT APIM (chaotic):
App1 ──── calls ────► Backend Service 1
App2 ──── calls ────► Backend Service 2
App3 ──── calls ────► Backend Service 1  ← no control, duplicate code
AppN ──── calls ────► Backend Service N  ← security holes

WITH APIM (organized):
                    ┌─────────────────────────────────────┐
App1 ──────────────►│                                     │──► Backend 1
App2 ──────────────►│      API Management                 │──► Backend 2
App3 ──────────────►│   (Single entry point)              │──► Backend 3
AppN ──────────────►│                                     │──► Backend N
                    └─────────────────────────────────────┘
                    ↑ Controls: Auth, Rate Limiting,
                      Caching, Transformation, Analytics
```

### What APIM Does

| Feature | What it Means | Real Example |
|---------|--------------|--------------|
| **Gateway** | Single entry point for all APIs | All APIs go through one URL |
| **Authentication** | Verify who is calling | Check API key or OAuth token |
| **Rate Limiting** | Limit how many calls per second | Max 100 calls/minute per user |
| **Caching** | Store responses to avoid hitting backend | Same request = cached response |
| **Transformation** | Change request/response format | Convert XML to JSON |
| **Analytics** | Track who calls what, when | Dashboard with call counts |
| **Documentation** | Auto-generated developer portal | Developers can explore APIs |
| **Versioning** | Run v1 and v2 of API simultaneously | Old clients use v1, new use v2 |

### APIM Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    API Management Instance                        │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                   Developer Portal                        │  │
│   │          (Documentation + API testing for devs)           │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                     API Gateway                           │  │
│   │   Inbound Policies → Backend → Outbound Policies         │  │
│   │                                                           │  │
│   │   Policies: Auth, Rate limit, Transform, Cache, Log       │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                   │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │                     Management Plane                      │  │
│   │   (Azure Portal / REST API to configure everything)       │  │
│   └──────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### APIM Tiers

| Tier | Throughput | VNet | Use Case |
|------|-----------|------|---------|
| **Consumption** | Pay-per-call | No | Serverless, low traffic |
| **Developer** | 500 RPM | Yes | Testing, dev only |
| **Basic** | 1,000 RPM | No | Light production |
| **Standard** | 2,500 RPM | No | Medium production |
| **Premium** | 4,000 RPM | Yes | Enterprise, multi-region |

### Creating APIM

```powershell
# Create API Management instance (takes 30-40 minutes)
New-AzApiManagement `
    -ResourceGroupName "Integration-RG" `
    -Name "mycompany-apim" `
    -Location "East US" `
    -Organization "My Company" `
    -AdminEmail "admin@mycompany.com" `
    -Sku "Developer"        # Use Developer for testing
```

### APIM Policies (Important Concept)

Policies are rules applied to API calls. Written in XML.

```xml
<!-- Example APIM Policy -->
<policies>
    <!-- INBOUND: Applied when request comes IN -->
    <inbound>
        <!-- Check API Key -->
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401">
            <openid-config url="https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration"/>
        </validate-jwt>

        <!-- Rate limit: 5 calls per 15 seconds per key -->
        <rate-limit-by-key calls="5" renewal-period="15" counter-key="@(context.Subscription.Id)" />

        <!-- Cache GET requests for 60 seconds -->
        <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" />

        <!-- Forward to backend -->
        <set-backend-service base-url="https://mybackend.azurewebsites.net" />
    </inbound>

    <!-- BACKEND: When calling the backend service -->
    <backend>
        <forward-request />
    </backend>

    <!-- OUTBOUND: Applied to the response going OUT -->
    <outbound>
        <cache-store duration="60" />
        <!-- Add response header -->
        <set-header name="X-Powered-By" exists-action="delete" />
    </outbound>

    <!-- ON-ERROR: If something goes wrong -->
    <on-error>
        <set-status code="500" reason="Internal Server Error" />
    </on-error>
</policies>
```

### APIM + Service Bus + Event Grid = Integration Platform

```
External Apps/Mobile
        │
        ▼
┌───────────────┐
│     APIM      │  ← Entry point, security, rate limiting
└───────┬───────┘
        │
   ┌────▼────┐
   │Functions│  ← Process and transform data
   └────┬────┘
        │
 ┌──────▼──────┐
 │ Service Bus │  ← Queue for reliable async processing
 └──────┬──────┘
        │
 ┌──────▼──────┐
 │ Event Grid  │  ← Notify other services
 └──────┬──────┘
        │
 ┌──────▼──────┐
 │ Backend Apps│  ← Your actual business logic
 └─────────────┘
```

---

## 13.5 Azure Functions

### What are Azure Functions?

Azure Functions let you run **small pieces of code (functions) without managing any servers**. You write the code, Azure handles everything else — servers, scaling, OS patches.

```
TRADITIONAL (Server-based):
┌─────────────────────────────────────────────────────┐
│                   SERVER (always running)            │
│  RAM: used even when idle                            │
│  CPU: used even when idle                            │
│  Cost: $100/month even if no traffic                 │
│  YOU manage: OS updates, scaling, availability       │
└─────────────────────────────────────────────────────┘

SERVERLESS (Azure Functions):
Function only runs when triggered
┌──────────┐   Triggered!   ┌──────────────┐
│ Event    │───────────────►│  Code runs   │  ← Lasts milliseconds to minutes
│ (trigger)│                │  (function)  │
└──────────┘                └──────────────┘
Cost: Pay ONLY for execution time
Scale: Automatic, instant
Management: ZERO - Azure handles everything
```

### Triggers - What Starts a Function?

| Trigger Type | What Starts It | Example |
|--------------|---------------|---------|
| **HTTP Trigger** | An HTTP request | REST API call |
| **Timer Trigger** | A schedule (like cron) | Run every day at 9 AM |
| **Service Bus Trigger** | Message in Service Bus queue | New order in queue |
| **Event Grid Trigger** | An Event Grid event | File uploaded |
| **Blob Trigger** | File added to blob storage | New CSV file uploaded |
| **Queue Trigger** | Message in Storage Queue | Simple message queue |
| **Cosmos DB Trigger** | Document changed in DB | Data change feed |

### Bindings - Easy Data Connections

Bindings let your function **read from and write to Azure services without writing connection code**.

```csharp
// WITHOUT bindings (lots of manual code):
public async Task ProcessOrder(string orderId)
{
    // Manually connect to Service Bus
    var sbClient = new ServiceBusClient("connection-string");
    var receiver = sbClient.CreateReceiver("orders");
    var message = await receiver.ReceiveMessageAsync();

    // Manually write to Cosmos DB
    var cosmosClient = new CosmosClient("connection-string");
    var container = cosmosClient.GetContainer("db", "orders");
    await container.CreateItemAsync(new { id = orderId });
}

// WITH bindings (clean, simple):
[FunctionName("ProcessOrder")]
public async Task ProcessOrder(
    [ServiceBusTrigger("orders", Connection = "ServiceBusConnection")] string orderMessage,  // INPUT binding
    [CosmosDB("db", "orders", Connection = "CosmosConnection")] IAsyncCollector<object> outputDocs,  // OUTPUT binding
    ILogger log)
{
    var order = JsonSerializer.Deserialize<Order>(orderMessage);
    await outputDocs.AddAsync(order);  // That's it! Azure handles the connection
    log.LogInformation($"Processed order: {order.Id}");
}
```

### Function App Example - HTTP API

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;

public class OrderFunctions
{
    // This function responds to HTTP GET /api/orders/{id}
    [Function("GetOrder")]
    public HttpResponseData GetOrder(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "orders/{id}")] HttpRequestData req,
        string id,
        FunctionContext context)
    {
        var logger = context.GetLogger("GetOrder");
        logger.LogInformation($"Getting order {id}");

        // Your business logic here
        var order = new { Id = id, Status = "Processing", Amount = 99.99 };

        var response = req.CreateResponse(System.Net.HttpStatusCode.OK);
        response.WriteAsJsonAsync(order);
        return response;
    }

    // This function runs every day at 9 AM UTC
    [Function("DailyReport")]
    public void DailyReport(
        [TimerTrigger("0 0 9 * * *")] TimerInfo timer,
        FunctionContext context)
    {
        var logger = context.GetLogger("DailyReport");
        logger.LogInformation($"Running daily report at {DateTime.UtcNow}");
        // Generate and send daily report
    }
}
```

### Creating Function App

```powershell
# Create Storage Account (required for Function App)
New-AzStorageAccount `
    -ResourceGroupName "Integration-RG" `
    -Name "myfuncstorageacct" `
    -Location "East US" `
    -SkuName "Standard_LRS"

# Create App Service Plan (Consumption = serverless)
New-AzFunctionAppPlan `
    -ResourceGroupName "Integration-RG" `
    -Name "myFunctionPlan" `
    -Location "East US" `
    -Sku "Y1" `        # Y1 = Consumption (pay-per-execution)
    -WorkerType "dotnet"

# Create Function App
New-AzFunctionApp `
    -ResourceGroupName "Integration-RG" `
    -Name "mycompany-functions" `
    -StorageAccountName "myfuncstorageacct" `
    -PlanName "myFunctionPlan" `
    -Runtime "dotnet" `
    -RuntimeVersion "6"
```

### Function Plans Comparison

| Plan | Description | Scaling | Cost |
|------|------------|---------|------|
| **Consumption** | Pay per execution, scale to zero | Auto, instant | Very low (free tier included) |
| **Premium** | Pre-warmed instances, no cold start, VNet | Auto | Medium |
| **Dedicated (App Service)** | Always-on VMs | Manual/auto | Fixed VM cost |

### When to Use Azure Functions

```
USE AZURE FUNCTIONS WHEN:
✓ Processing messages from queues (Service Bus, Storage Queue)
✓ Running scheduled jobs (reports, cleanup)
✓ Responding to file uploads (image resize, CSV import)
✓ Building lightweight APIs
✓ Glue code between services
✓ Event-driven processing

REAL EXAMPLES:
• Process order message from Service Bus
• Resize images uploaded to Blob Storage
• Send daily email reports
• Transform data between formats
• Webhook endpoints for third-party services
```

---

## 13.6 Azure Logic Apps

### What are Logic Apps?

Azure Logic Apps are **visual workflows** — like flowcharts that run in the cloud. You connect apps and services with a drag-and-drop designer, **no coding required**.

```
VISUAL WORKFLOW EXAMPLE:

[When email arrives]
         │
         ▼
[Check: has attachment?]
    YES │          NO │
        ▼             ▼
[Save attachment    [Mark as read]
 to SharePoint]          │
        │                │
        ▼                ▼
[Post message to    [Done]
 Teams channel]
        │
        ▼
[Send reply email]
```

### Functions vs Logic Apps

| Feature | Azure Functions | Logic Apps |
|---------|----------------|------------|
| **How you build** | Write code | Drag-and-drop designer |
| **Who uses it** | Developers | Developers + Non-developers |
| **Best for** | Complex logic, data processing | Workflows, integration, orchestration |
| **Connectors** | Manual code | 400+ pre-built connectors |
| **Monitoring** | App Insights | Built-in run history |

### 400+ Connectors Available

```
A few examples of what Logic Apps can connect to out-of-the-box:

Microsoft:           Third-Party:          Social:
• SharePoint         • Salesforce           • Twitter/X
• Outlook/Exchange   • ServiceNow           • LinkedIn
• Teams              • SAP                  • Facebook
• OneDrive           • Workday
• Azure SQL          • Zendesk              Protocols:
• Service Bus        • DocuSign             • HTTP/HTTPS
• Event Grid         • Slack                • FTP/SFTP
• Cosmos DB          • GitHub               • SOAP
                     • Twilio               • REST
                     • Stripe
```

### Example Logic App - Order Processing Workflow

```
Trigger: HTTP Request received (new order)
    │
    ▼
Action: Send message to Service Bus queue (order-queue)
    │
    ▼
Action: Create record in Azure SQL Database
    │
    ▼
Condition: Order amount > $500?
   YES │                    NO │
       ▼                       ▼
Action: Send approval      Action: Auto-approve
  email via Outlook          and continue
       │
       ▼
Condition: Approved?
   YES │          NO │
       ▼             ▼
Action: Process    Action: Cancel order
  payment            and notify customer
       │
       ▼
Action: Send confirmation email
       │
       ▼
Action: Post to Teams channel "New order processed"
```

### Creating Logic App (Azure Portal)

```
1. Azure Portal → Create a resource → Logic App
2. Fill in: Name, Resource Group, Location
3. Open Logic App Designer
4. Choose trigger (e.g., "When a HTTP request is received")
5. Add actions (drag and drop)
6. Save and enable
7. Test by sending HTTP request
```

### Logic App in Code (ARM Template snippet)

```json
{
  "definition": {
    "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
    "triggers": {
      "When_a_message_is_received": {
        "type": "ServiceBusTrigger",
        "inputs": {
          "host": { "connection": { "name": "@parameters('$connections')['servicebus']['connectionId']" } },
          "method": "get",
          "path": "/@{encodeURIComponent('order-queue')}/messages/head"
        }
      }
    },
    "actions": {
      "Send_email": {
        "type": "ApiConnection",
        "inputs": {
          "host": { "connection": { "name": "@parameters('$connections')['outlook']['connectionId']" } },
          "method": "post",
          "path": "/Mail",
          "body": {
            "To": "team@company.com",
            "Subject": "New Order Received",
            "Body": "@{triggerBody()}"
          }
        }
      }
    }
  }
}
```

---

## 13.7 Choosing the Right Integration Service

### Decision Guide

```
WHAT DO YOU NEED TO DO?
          │
          ▼
Is it about MESSAGING between apps?
   YES → Is it one receiver or many?
              ONE  → Service Bus QUEUE
              MANY → Service Bus TOPIC
          │
          ▼
Is it about reacting to EVENTS (something happened)?
   YES → Azure EVENT GRID
          │
          ▼
Is it an API that needs managing?
   YES → Azure API MANAGEMENT
          │
          ▼
Is it code/logic that runs in response to triggers?
   YES → Azure FUNCTIONS (code) or LOGIC APPS (no-code)
          │
          ▼
Is it a multi-step workflow connecting many apps?
   YES → Azure LOGIC APPS
```

### Combined Architecture (Real Enterprise Example)

```
┌─────────────────────────────────────────────────────────────────────┐
│              ATT Solutions Integration Platform                      │
│                                                                      │
│  External Clients          APIM                   Internal Services │
│  ┌──────────────┐         ┌─────┐                 ┌──────────────┐  │
│  │ Mobile App   │────────►│     │                 │ Order Svc    │  │
│  └──────────────┘         │ API │────────────────►│ Inventory Svc│  │
│  ┌──────────────┐         │ MGT │    Functions    │ Payment Svc  │  │
│  │ Web App      │────────►│     │────────────────►│ Shipping Svc │  │
│  └──────────────┘         │(APIM│                 └──────────────┘  │
│  ┌──────────────┐         │  )  │                                   │
│  │ Partner API  │────────►│     │  Service Bus                      │
│  └──────────────┘         └──┬──┘  ┌──────────────────────────┐    │
│                               │     │  order-queue             │    │
│                               └────►│  payment-queue           │    │
│                                     │  notification-topic      │    │
│                                     └──────────┬───────────────┘    │
│                                                │                     │
│                                    ┌───────────▼──────────┐         │
│                                    │    Event Grid        │         │
│                                    │  (broadcasts events) │         │
│                                    └───────────┬──────────┘         │
│                                                │                     │
│                          ┌─────────────────────┼──────────────┐     │
│                          ▼                     ▼              ▼     │
│                    ┌──────────┐         ┌──────────┐    ┌──────────┐│
│                    │ Function │         │Logic App │    │ Function ││
│                    │(Resize   │         │(Approval │    │(Send SMS)││
│                    │ images)  │         │ workflow)│    │          ││
│                    └──────────┘         └──────────┘    └──────────┘│
└─────────────────────────────────────────────────────────────────────┘
```

---

## Hands-on Exercises

### Exercise 1: Create Service Bus and Send Messages

```powershell
# Step 1: Create namespace
New-AzResourceGroup -Name "Integration-Lab-RG" -Location "East US"

New-AzServiceBusNamespace `
    -ResourceGroupName "Integration-Lab-RG" `
    -NamespaceName "mylab-servicebus-$(Get-Random -Maximum 9999)" `
    -Location "East US" `
    -SkuName "Standard"

# Step 2: Create queue
New-AzServiceBusQueue `
    -ResourceGroupName "Integration-Lab-RG" `
    -NamespaceName "mylab-servicebus-1234" `    # use your actual name
    -QueueName "lab-orders" `
    -MaxDeliveryCount 3

# Step 3: Get connection string
Get-AzServiceBusKey `
    -ResourceGroupName "Integration-Lab-RG" `
    -NamespaceName "mylab-servicebus-1234" `
    -AuthorizationRuleName "RootManageSharedAccessKey"
```

### Exercise 2: Create Event Grid and Subscribe

```powershell
# Create a custom topic
New-AzEventGridTopic `
    -ResourceGroupName "Integration-Lab-RG" `
    -Name "lab-events" `
    -Location "East US"

# Create storage account to receive events (via queue)
New-AzStorageAccount `
    -ResourceGroupName "Integration-Lab-RG" `
    -Name "labeventstorage" `
    -Location "East US" `
    -SkuName "Standard_LRS"

# Subscribe storage queue to receive events
# (Configure in Azure Portal - Event Grid → Subscriptions → Create)
Write-Host "Go to Azure Portal → lab-events topic → Event Subscriptions → New"
```

### Exercise 3: Create a Simple Azure Function

```powershell
# Create Function App
New-AzStorageAccount `
    -ResourceGroupName "Integration-Lab-RG" `
    -Name "labfuncstorage$(Get-Random -Maximum 9999)" `
    -Location "East US" `
    -SkuName "Standard_LRS"

New-AzFunctionApp `
    -ResourceGroupName "Integration-Lab-RG" `
    -Name "mylab-functions-$(Get-Random -Maximum 9999)" `
    -StorageAccountName "labfuncstorage1234" `   # use your actual name
    -Runtime "dotnet" `
    -RuntimeVersion "6" `
    -Location "East US" `
    -OSType "Windows" `
    -FunctionsVersion "4"

Write-Host "Function App created! Go to Azure Portal to add functions"
Write-Host "Portal → Function App → Functions → + Create → HTTP trigger"
```

---

## Summary

| Service | What it is | When to Use |
|---------|------------|-------------|
| **Service Bus** | Enterprise message broker | Reliable messaging, commands, transactions |
| **Event Grid** | Event routing service | Notifications, react to Azure events |
| **API Management** | API gateway + management | Secure, manage, expose APIs |
| **Azure Functions** | Serverless code execution | Event-driven code, scheduled jobs |
| **Logic Apps** | Visual workflow builder | No-code workflows, app integration |

---

## Interview Answers

**Q: What is Azure Service Bus?**
> Azure Service Bus is a fully managed enterprise message broker that provides reliable, asynchronous message queuing. It supports queues (one receiver) and topics (multiple subscribers). Used for decoupling applications, ensuring message delivery even when receivers are temporarily unavailable.

**Q: Difference between Service Bus and Event Grid?**
> Service Bus is for commands/messages that need to be processed — guaranteed delivery, ordered, one-to-one or one-to-many. Event Grid is for notifications about things that happened — lightweight events, near-real-time, broadcast to many subscribers. Think Service Bus = "do this task", Event Grid = "this just happened".

**Q: What is API Management?**
> APIM is a gateway that sits in front of your APIs to provide security (auth, rate limiting), transformation, caching, analytics, and documentation. It creates a single entry point for all your APIs and protects your backend services.

**Q: When would you use Logic Apps vs Functions?**
> Logic Apps for visual workflows, connecting many services with pre-built connectors (Outlook, Salesforce, SAP, etc.), good for non-developers. Azure Functions for code-based processing, complex logic, data transformation — when you need more control.

---

## Additional Resources

- [Azure Service Bus Documentation](https://docs.microsoft.com/azure/service-bus-messaging/)
- [Azure Event Grid Documentation](https://docs.microsoft.com/azure/event-grid/)
- [Azure API Management](https://docs.microsoft.com/azure/api-management/)
- [Azure Functions](https://docs.microsoft.com/azure/azure-functions/)
- [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/)
