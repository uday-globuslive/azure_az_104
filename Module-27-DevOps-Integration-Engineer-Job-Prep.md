# Module 27 - DevOps Integration Engineer — Job-Specific Preparation
## (Bath / Claverton Office Role — December 2024 JD)

> **Purpose of This Module**: This module is written specifically for the DevOps Integration Engineer JD from the Bath IT Applications team. It maps every requirement to your existing study materials, explains the **gaps** in plain language, and gives you complete word-for-word interview answers aligned to what this specific employer is looking for.
>
> **Excluded (as requested)**: Azure Data Factory (ADF) and .NET / C# — skip those sections in the JD.

---

## What This Role Actually Wants (Plain English)

```
THE COMPANY HAS:
- A mix of old (legacy) integration systems
- Applications that don't talk to each other well
- A team that needs technical leadership, not just doers

THEY WANT SOMEONE WHO:
1. Has BUILT Azure integration from scratch (not just maintained it)
2. Can lead others technically (not just do the work alone)
3. Knows the full Azure integration stack deeply:
   Logic Apps → Service Bus → APIM → Functions → Event Grid
4. Can automate everything with CI/CD and DevOps
5. Bridges the gap between Dev team and Architecture team
6. Can explain complex things to non-technical people

KEY PHRASE IN THE JD:
"personally designed, built, and implemented Azure integration
and cloud architectures from the ground up"

This means they will directly ask: "Walk me through something you built."
You must have a strong, detailed, personal story ready.
```

---

## Your Study Map (JD Requirement → Module to Study)

| JD Requirement | Your Module | Status |
|---------------|-------------|--------|
| Azure Logic Apps | Module-13 (Integration Services) | ✅ Covered |
| Azure API Management | Module-13 (Integration Services) | ✅ Covered |
| Azure Service Bus | Module-13 (Integration Services) | ✅ Covered |
| Azure Functions | Module-13 (Integration Services) | ✅ Covered |
| Azure Event Grid | Module-13 (Integration Services) | ✅ Covered |
| Azure Landing Zones | Module-14 (Landing Zones & Governance) | ✅ Covered |
| Hub & Spoke networking | Module-12 (Advanced Networking) | ✅ Covered |
| CI/CD pipelines | Module-11 (Azure DevOps & CI/CD) | ✅ Covered |
| Azure DevOps | Module-11 & Module-15 | ✅ Covered |
| GitHub / GitHub Actions | Module-19 (GitOps & CI/CD Pipelines) | ✅ Covered |
| PowerShell | Module-16 (Development & Scripting) | ✅ Covered |
| Node.js / JavaScript | Module-16 (Development & Scripting) | ✅ Covered |
| Security & Governance | Module-14, Module-15 | ✅ Covered |
| Monitoring & Observability | Module-10, Module-22 | ✅ Covered |
| Architecture full design | Module-15 (Architecture & Interview Prep) | ✅ Covered |
| SQL Server basics | See Section 27.3 below | ⬇️ In this module |
| Agile methodology | See Section 27.4 below | ⬇️ In this module |
| Legacy migration/retirement | Module-11 (Migration) | ✅ Covered |
| Technical leadership / SME | See Section 27.5 below | ⬇️ In this module |

---

## 27.1 Role Breakdown — What You'll Actually Do Day-to-Day

### The Three Hats You'll Wear

```
HAT 1: INTEGRATION ARCHITECT
- Design how different systems/applications exchange data
- Choose between Service Bus vs Event Grid vs Logic Apps
- Define patterns: async vs sync, retry policies, error handling
- Review integration designs from other developers

HAT 2: DEVOPS ENGINEER  
- Write CI/CD pipelines (Azure DevOps or GitHub Actions)
- Infrastructure as Code (Terraform / Bicep / ARM)
- Automate deployments so nothing is done manually
- Monitor production integrations, respond to failures

HAT 3: TECHNICAL LEADER / SME
- Guide junior developers and offshore teams
- Participate in architecture review boards
- Advise project managers on technical feasibility and timelines
- Document integration patterns and standards
- "Own" the integration platform — it's yours to run and improve
```

### What They Mean by "Legacy Integration Migration"

```
LEGACY PLATFORM = usually one of these:
- BizTalk Server (Microsoft's old on-premises integration tool)
- MuleSoft
- IBM MQ / IBM Integration Bus
- Custom-built point-to-point integrations (spaghetti)

YOUR JOB:
1. Understand what the legacy platform does
2. Map each existing integration to its Azure equivalent:
   BizTalk Orchestration → Logic Apps
   BizTalk Message Queues → Service Bus
   BizTalk HTTP Receive Port → APIM
3. Build the new Azure version
4. Test side-by-side (run both temporarily)
5. Cut over, then retire the legacy system
```

---

## 27.2 Event-Driven Integration Patterns (Deep Level for This Role)

### The Patterns This Role Requires You to Know

#### Pattern 1: Request-Reply (Synchronous)

```
WHEN TO USE:
- User needs an immediate answer
- "Is this product in stock?" → Need answer NOW

FLOW:
Client App ──► APIM ──► Logic App / Function ──► Backend System
                                               ◄── Response

AZURE SERVICES:
- APIM handles the incoming request
- Logic App calls backend, waits, returns response
- Function App for lightweight transformations
```

#### Pattern 2: Fire and Forget (Async Command)

```
WHEN TO USE:
- Client doesn't need to wait for the result
- "Process this order" — it will be done eventually

FLOW:
Client App ──► APIM ──► Service Bus Queue ──► Logic App / Function
(gets 202 Accepted back immediately)         (processes independently)

WHY THIS IS BETTER THAN SYNC:
- Client is never blocked waiting
- Backend can scale independently
- Failures don't affect the client
- Service Bus retries automatically if processing fails
```

#### Pattern 3: Publish-Subscribe (Event Notification)

```
WHEN TO USE:
- Something happened and multiple systems need to know
- "Customer address changed" → CRM, billing, shipping all need this

FLOW:
Source System ──► Event Grid ──► Topic
                                  ├──► CRM Logic App
                                  ├──► Billing Function
                                  └──► Shipping Notification

WHY EVENT GRID:
- Push-based (not polling)
- Near real-time
- Scale to millions of events
- Each subscriber processes independently
```

#### Pattern 4: Aggregator (Correlation)

```
WHEN TO USE:
- Need to collect multiple responses before proceeding
- "Wait for all 3 suppliers to respond before placing final order"

AZURE IMPLEMENTATION:
Logic App (Stateful) with correlation ID:
1. Send request to Supplier1, Supplier2, Supplier3
2. Wait for all 3 responses (correlation on OrderId)
3. When all received → compare prices → place order

Logic Apps Stateful = perfect for this (state stored in Azure Storage)
```

#### Pattern 5: Compensating Transaction (Saga)

```
WHEN TO USE:
- Multi-step process where a later step can fail
- Need to "undo" earlier completed steps

EXAMPLE: Flight booking
Step 1: Reserve seat → SUCCESS
Step 2: Charge payment → SUCCESS  
Step 3: Add loyalty points → FAILS

SAGA RESPONSE:
Step 3 fails → trigger compensation:
Step 2b: Refund payment
Step 1b: Release seat
Send failure notification to customer

AZURE IMPLEMENTATION:
Logic App Stateful with error handling branches
or Azure Durable Functions (orchestration pattern)
```

---

## 27.3 SQL Server Basics for Azure Integration (Gap Filled)

### Why SQL Server Matters for This Role

```
As a DevOps Integration Engineer you will:
- Connect integrations TO databases (read/write data)
- Write SQL queries inside Logic Apps (SQL Server connector)
- Troubleshoot data issues in integrations
- Help migrate data alongside application migration

YOU DON'T NEED: Deep DBA skills
YOU DO NEED: Read queries, understand schemas, spot issues
```

### SQL You Must Know for Integration Work

```sql
-- 1. Read data (most common in integrations)
SELECT 
    OrderId,
    CustomerId,
    OrderDate,
    TotalAmount,
    Status
FROM Orders
WHERE Status = 'Pending'
  AND OrderDate >= DATEADD(day, -1, GETDATE())
ORDER BY OrderDate DESC;

-- 2. Check if record exists (before insert)
IF NOT EXISTS (SELECT 1 FROM Customers WHERE Email = 'user@example.com')
BEGIN
    INSERT INTO Customers (Email, Name, CreatedDate)
    VALUES ('user@example.com', 'John Smith', GETDATE())
END

-- 3. Update record status (common in order processing)
UPDATE Orders
SET Status = 'Processed',
    ProcessedDate = GETDATE(),
    ProcessedBy = 'IntegrationService'
WHERE OrderId = 12345
  AND Status = 'Pending';

-- 4. Join tables (when your integration needs related data)
SELECT 
    o.OrderId,
    c.Name AS CustomerName,
    c.Email,
    p.ProductName,
    oi.Quantity
FROM Orders o
INNER JOIN Customers c ON o.CustomerId = c.CustomerId
INNER JOIN OrderItems oi ON o.OrderId = oi.OrderId
INNER JOIN Products p ON oi.ProductId = p.ProductId
WHERE o.OrderId = 12345;

-- 5. Stored procedure call (Logic Apps calls these often)
EXEC ProcessOrder 
    @OrderId = 12345, 
    @UserId = 'integration-service'
```

### Azure SQL vs SQL Server on VM

| | Azure SQL Database | SQL Server on VM |
|--|---|---|
| **Management** | Microsoft manages patching, backups | You manage everything |
| **Scaling** | Elastic, instant | Manual, requires downtime |
| **Cost** | Pay for DTUs/vCores | Pay for VM + SQL license |
| **Private Endpoint** | ✅ Supported (recommended) | ✅ VNet integration |
| **Managed Identity auth** | ✅ No password needed | Limited |
| **Use when** | New projects, cloud-native | Lift-and-shift, legacy apps |

### Logic Apps SQL Connector (Integration-Specific)

```
IN LOGIC APPS, you can use the SQL Server connector to:
- Trigger: "When a row is modified in Orders table"
- Action: "Get rows from Products"
- Action: "Insert row into ProcessingLog"
- Action: "Execute stored procedure"

This means you can build integrations that:
1. Watch a database table for changes
2. Process those records through your integration
3. Call external APIs or other services
4. Write results back to the database

No code required — all visual in Logic Apps designer.
```

---

## 27.4 Agile Methodology for Technical Leads (Gap Filled)

### What Agile Actually Means in a DevOps Role

```
AGILE = Way of working where you deliver in small chunks
        instead of one big release at the end

WHY IT EXISTS:
Old way (Waterfall):
Plan for 6 months → Build for 12 months → Test for 3 months
→ 21 months later: "This isn't what we wanted"

Agile way:
Build for 2 weeks → Show it → Get feedback → Adjust → Repeat
→ Deliver value every 2 weeks → Always aligned with business
```

### Scrum Framework (Most Common)

```
SCRUM STRUCTURE:

┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCT BACKLOG                               │
│  All the work that needs to be done (prioritized list)          │
│  "Integrate CRM with billing system"                            │
│  "Build payment webhook endpoint"                               │
│  "Migrate legacy BizTalk orchestration to Logic Apps"           │
└────────────────────────────┬────────────────────────────────────┘
                             │ Sprint Planning
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SPRINT (2 weeks)                              │
│  Selected items from backlog committed to this sprint           │
│  Daily standup: What did I do? What will I do? Any blockers?    │
│  Sprint Review: Demo what was built to stakeholders             │
│  Retrospective: What went well? What to improve?                │
└─────────────────────────────────────────────────────────────────┘
```

### Agile Tools This JD Mentions

```
JIRA / ASANA = Backlog management tools
- Create user stories: "As a warehouse manager, I want to receive 
  a notification when stock falls below reorder level"
- Break into tasks: estimate hours, assign to team members
- Track progress: To Do → In Progress → In Review → Done
- Sprint boards: visual view of current sprint work

AZURE DEVOPS BOARDS = Microsoft's version of Jira
- Epics → Features → User Stories → Tasks
- Link work items to Git commits and pull requests
- "When I commit code, it auto-closes the work item"

GITHUB ISSUES + PROJECTS = Lightweight issue tracking
- Issues = bugs or features
- Projects = kanban board
- Milestones = release groupings
```

### Your Role in Agile as a Technical Lead

```
AS THE INTEGRATION SME YOU:

Sprint Planning:
- Estimate integration stories: "This Logic App will take 3 days"
- Identify dependencies: "We can't start this until API is ready"
- Flag technical risk: "This pattern is complex, add buffer"

Daily Standup:
- Report blockers quickly: "Waiting on access to test environment"
- Unblock others: "I'll help you with that connector config"

Sprint Review:
- Demo integration functionality to business stakeholders
- Explain what was built in non-technical terms

Technical Backlog Refinement:
- Break large integration stories into smaller tasks
- Write acceptance criteria (how do we know it's done?)
- "Integration must handle 1000 messages/hour without failure"

Retrospective:
- "Our deployment pipeline has no testing — let's add it"
- "We should document the integration patterns we used"
```

### User Story Format for Integration Work

```
FORMAT:
"As a [role], I want [action], so that [benefit]"

INTEGRATION EXAMPLES:

Story 1:
"As a Sales Manager, I want the CRM to automatically update
when an order is fulfilled in the ERP system, so that I don't
need to manually check order status."

Acceptance Criteria:
- CRM contact record updated within 60 seconds of ERP fulfillment
- Update contains: order number, fulfillment date, delivery estimate
- If CRM update fails, message goes to dead-letter queue and alert fires
- Works during business hours and overnight batch processing

Story 2:
"As a Finance Manager, I want an alert when a payment fails,
so that the accounts team can follow up immediately."

Acceptance Criteria:
- Alert sent within 5 minutes of payment failure
- Alert includes: customer name, order ID, amount, failure reason
- Alert sent via email AND Teams message
- Alert does not fire for retries — only final failure after 3 attempts
```

---

## 27.5 Technical Leadership — Being an SME (Gap Filled)

### What "SME" (Subject Matter Expert) Really Means

```
SME = You are THE person people come to for integration questions
      You own the platform, you set the standards, you resolve escalations

IN PRACTICE:
- Developer asks: "Which should I use, Service Bus or Event Grid?"
  → You give them a clear answer AND explain why
  
- Project Manager asks: "How long will this integration take?"
  → You estimate accurately and explain complexity
  
- Architect asks: "Is this integration design secure?"
  → You review it and provide feedback with recommendations
  
- Business asks: "Why did the integration fail at 2am?"
  → You investigate, fix, explain in plain language, prevent recurrence
```

### How to Review Integration Designs (Technical Governance)

```
CHECKLIST WHEN REVIEWING ANOTHER DEVELOPER'S INTEGRATION:

RESILIENCE:
□ Is there a retry policy? (Logic Apps built-in or custom)
□ Is there a dead-letter queue for failed messages?
□ Does it handle duplicate messages (idempotency)?
□ What happens if downstream service is unavailable?

SECURITY:
□ Are credentials in Key Vault? (NOT hardcoded)
□ Is authentication using Managed Identity where possible?
□ Is data encrypted in transit (HTTPS/AMQP TLS)?
□ Is sensitive data masked in logs?
□ Is the endpoint accessible only via Private Endpoint?

OBSERVABILITY:
□ Are there Application Insights traces?
□ Is there a structured log message at start, end, and error?
□ Are there metrics/alerts configured for failure rates?
□ Can you trace a message end-to-end with a correlation ID?

PERFORMANCE:
□ Is it processing messages in batches where appropriate?
□ Are there unnecessary polling loops? (Use push instead)
□ Is there unnecessary data being transferred? (Select only needed fields)

STANDARDS:
□ Does it follow the team's naming conventions?
□ Is it deployed via CI/CD pipeline? (Not manually)
□ Is there documentation for operations team?
□ Is there a runbook for common failure scenarios?
```

### Mentoring Developers and Offshore Teams

```
SITUATION: You have offshore developers who are new to Azure integrations

YOUR APPROACH:

1. CREATE REFERENCE ARCHITECTURE
   Build a "golden example" Logic App that demonstrates:
   - Correct error handling pattern
   - Correct logging with correlation IDs
   - Correct Key Vault secret retrieval
   - Correct retry and dead-letter handling
   "Copy this pattern, adapt for your use case"

2. CODE REVIEW WITH EXPLANATION
   Don't just say "this is wrong" — explain WHY
   "This hardcoded connection string is a security risk.
   Move it to Key Vault and access it via Managed Identity.
   Here's the documentation: [link]"

3. PAIR PROGRAMMING
   Work alongside them for the first integration
   "Let me show you how I approach this, then you do the next one"

4. DOCUMENTATION
   Write integration patterns in the team wiki
   Developers should find answers BEFORE asking you
   Your goal: reduce how many times you're asked the same question

5. ARCHITECTURE DECISION RECORDS (ADRs)
   When you make a design decision, document it:
   "Decision: Use Service Bus instead of HTTP callbacks
   Reason: Better resilience and retry handling
   Date: [date] | Author: [you]"
```

---

## 27.6 Interview Questions — Mapped to This Specific JD

### Q1: "Tell me about a complex integration you designed and built from scratch."

> **STAR Answer:**
>
> **Situation**: We had a monolithic application where multiple business units — sales, warehouse, finance — all shared a single database. Any change to one area risked breaking another. Each team waited weeks for their feature requests because everything was tightly coupled.
>
> **Task**: I was brought in to design and build an Azure integration platform from scratch. The objective was to decouple these domains so each could evolve independently, while maintaining real-time data consistency between them.
>
> **Action**: First, I established the foundation. I built an Azure Landing Zone — Management Group hierarchy, separate subscriptions for Production and Non-Production, and applied Azure Policy to enforce tags, region restrictions, and no-public-IPs in production. For networking, I implemented Hub-Spoke with a centralised Azure Firewall and VPN Gateway in the Hub, with spoke VNets for the application tier. Every PaaS service — Service Bus, APIM, Logic Apps runtime — was accessed via Private Endpoints only.
>
> For the integration layer, I designed three patterns. Commands — things that needed guaranteed processing — went through Azure Service Bus queues. Events — things that happened that multiple teams needed to know about — were published through Event Grid. Complex, multi-step workflows such as purchase order approval, which required manager sign-off via email before finance processing, were built as Logic Apps Stateful workflows using the Outlook connector.
>
> Azure API Management became the single entry point. I configured JWT validation policies so no backend service needed to implement its own authentication. Rate limiting and IP whitelisting were applied at the APIM layer. I deployed everything via Azure DevOps pipelines using ARM templates, so no environment was built manually.
>
> **Result**: The integration platform processed over 500 business events daily with zero message loss. Deployment time dropped from half a day of manual work to 15 minutes automated. The teams could deploy their own services independently without co-ordination, and when Finance needed a new integration, we had a reference pattern they could follow with no architecture review required.

---

### Q2: "What is the difference between Service Bus and Event Grid, and when would you use each?"

> "I think of it in terms of intent. Service Bus is for commands — messages where the sender is saying 'I need you to do something, and I need to know it was done.' It guarantees delivery, supports sessions for ordered processing, has dead-letter queues for failures, and supports competing consumers for scaling. I use Service Bus when the message must be processed exactly once, in order, with retries — things like payment processing or order fulfilment.
>
> Event Grid is for events — notifications where the sender is saying 'something happened, in case anyone cares.' It's push-based, near real-time, and designed for fan-out — one event going to multiple subscribers simultaneously. I use Event Grid when several systems need to react to the same occurrence — for example, when a customer record is updated, both the CRM and the billing system and the notification service should all be informed in parallel.
>
> The practical rule: if you need guaranteed processing by exactly one consumer, use Service Bus. If you need to notify multiple consumers simultaneously and they can act independently, use Event Grid. In a real architecture they often work together — Service Bus handles the reliable processing pipeline, Event Grid handles the downstream notification broadcast after processing is complete."

---

### Q3: "How do you secure an Azure integration platform?"

> "Security in integration has several layers. At the identity layer, I never use service account passwords. Every Logic App, Function App, and service connection uses Managed Identity to authenticate to Key Vault, Service Bus, and storage. There are no credentials stored in code or configuration files.
>
> At the network layer, all PaaS services are deployed with Private Endpoints — no public endpoints. Service Bus, APIM, and storage are only reachable from within the VNet. The APIM instance sits in a subnet with a Network Security Group restricting inbound traffic to application subnets only. Outbound traffic from integration services goes through an Azure Firewall with an allow-list of approved destinations.
>
> At the application layer, APIM validates JWT tokens from Azure AD before any request reaches a backend service. This means authentication is centralised — backends don't implement their own auth. APIM policies also scrub sensitive headers and mask PII in logs.
>
> For operational security, I use Azure Defender for Integration Services and enable audit logging in Azure Monitor. Every secret access in Key Vault is logged. Any anomaly — unexpected secret access, spike in failed requests — triggers an alert. And all infrastructure is deployed via CI/CD pipeline, so there are no manual deployments and every change has an audit trail in Azure DevOps."

---

### Q4: "How would you approach migrating a legacy BizTalk integration platform to Azure?"

> "I'd approach it in four phases.
>
> **Discovery first.** I'd audit every existing BizTalk orchestration, receive port, and send port. Map each one to understand: what data flows in, what transformation happens, where does it go, what triggers it, what are the SLAs. This becomes my migration inventory.
>
> **Foundation second.** Before migrating a single integration, I'd build the Azure Landing Zone — Management Groups, subscriptions, policies, Hub-Spoke networking, centralised logging. Trying to migrate applications before the foundation is ready creates rework.
>
> **Parallel migration third.** I'd migrate integrations by complexity — start with simple, low-risk ones to build team confidence and validate the patterns. Simple HTTP receive port becomes an APIM endpoint. Message queue becomes Service Bus. Orchestration becomes Logic App Stateful workflow. For each, I'd run the old and new in parallel, compare outputs, validate, then cut over. Critical business integrations stay on BizTalk until their Azure replacement is fully validated.
>
> **Decommission fourth.** Only once all integrations are live on Azure and have proven stable for an agreed period — typically 30 to 90 days — do we power down the BizTalk servers and reclaim those costs.
>
> Throughout, I'd document every decision as an Architecture Decision Record and build a runbook for each integration so the operations team can support it without needing me for every issue."

---

### Q5: "How do you ensure your CI/CD pipeline for Logic Apps meets production governance standards?"

> "A mature Logic Apps CI/CD pipeline has four stages.
>
> **Build stage**: Export the Logic App definition as ARM template or Bicep. Validate the template syntax. Run static analysis on any custom code in Functions. Store the built artifact — the parameterised template — in Azure Artifacts.
>
> **Dev deploy**: Apply the template to the dev environment using service principal with least-privilege access. Run integration tests — actually invoke the Logic App trigger and verify the output and any written records match expectation. These run automatically on every pull request.
>
> **Stage deploy with gated approval**: Deploy to staging, which mirrors production configuration. Run smoke tests and any load tests. A designated approver — architect or senior engineer — reviews the deployment summary before production is approved. This gate is configured in Azure DevOps Environments.
>
> **Production deploy**: Zero-downtime deployment using Logic App versioning or slot-equivalent patterns. Monitor the first 30 minutes — alert thresholds are temporarily lowered to catch any regression quickly. Rollback is a one-click pipeline trigger that redeploys the previous artifact.
>
> Infrastructure — VNet, private endpoints, NSGs — is managed separately via Terraform with its own pipeline. Application and infrastructure pipelines are deliberately separate so a Logic App update never accidentally changes network configuration."

---

### Q6: "Describe how you would set up monitoring for a production integration platform."

> "I implement monitoring at three levels.
>
> **Platform level**: Azure Monitor for infrastructure health — CPU, memory, storage queues depth, Service Bus dead-letter count. Alerts fire when dead-letter messages exceed a threshold (meaning something is failing silently), or when message processing lag exceeds SLA.
>
> **Application level**: Application Insights integration with every Logic App and Function App. Every run is traced. I add custom tracking properties — correlation ID, business entity type, entity ID — so I can query 'show me all integration activity for Order 12345' in Log Analytics in a single KQL query. This is critical when a business user says 'my order hasn't updated' — I can find the exact message and see exactly where it got stuck.
>
> **Business level**: Custom dashboards in Azure Workbooks that show business metrics — orders processed per hour, payment failure rate, average processing time — using the Application Insights data. This means the operations manager can see business health without needing to understand Azure Monitor. Alerts at this level use dynamic thresholds — if Sunday at 3am normally processes 5 orders per hour, an alert fires when that drops to zero. But Monday at 9am it expects 200 per hour, so the thresholds adjust automatically.
>
> For on-call support, I set up Azure Monitor action groups with different severity routing — P1 integration failures page on-call immediately, P2 failures send a Teams message, P3 warnings create a ticket automatically in Jira."

---

### Q7: "How do you handle errors and retries in Azure Logic Apps?"

> "Error handling in Logic Apps has three layers.
>
> **Built-in retry policy**: Every Logic App action has a configurable retry policy. For HTTP calls, I typically set exponential backoff — retry after 10 seconds, then 30, then 60 — with a maximum of 4 retries. For Service Bus sends, I use the default exponential backoff. The key is not using fixed interval retries — they create thundering herd problems when a downstream service comes back online.
>
> **Scope-based error handling**: I wrap related actions in a Scope block, then add a parallel branch that runs 'if Scope failed.' In that failure branch, I capture the error message, add a structured log entry to Application Insights with correlation ID and all relevant context, drop a message on a dead-letter handling queue for human review, and send a Teams alert for critical failures.
>
> **Dead-letter strategy**: Any message that exhausts all retries lands in a dead-letter queue in Service Bus. I build a companion Logic App — a dead-letter reprocessor — that an operator can trigger to resubmit specific messages after the root cause is fixed. This means no data is ever lost and there's a clear, auditable recovery path without developer involvement.
>
> The principle is: fail loudly, recover gracefully, never lose data silently."

---

### Q8: "How do you explain a technical architecture decision to a non-technical stakeholder?"

> "I avoid technical terms entirely and focus on the business outcome and the risk trade-off.
>
> For example, if I'm recommending Event Grid over a polling mechanism, I wouldn't say 'Event Grid provides push-based event routing with near-real-time delivery at sub-second latency versus a polling interval of N seconds.'
>
> Instead I'd say: 'Currently the system checks for updates every 5 minutes — like checking your email manually every 5 minutes. With this change, the system gets notified instantly when something happens, like a push notification on your phone. For the business, this means finance sees payment confirmations within seconds instead of waiting up to 5 minutes. It also reduces cost because the system stops making thousands of unnecessary checks per day.'
>
> I try to answer three questions they actually care about: what does this mean for me, what does it cost, and what could go wrong. If I can answer those three clearly, I've communicated the decision effectively. The technical detail is available in the architecture document for anyone who wants it."

---

## 27.7 Preparation Checklist for This Interview

### Technical Knowledge Checklist

```
AZURE INTEGRATION (study from Module 13):
□ Can explain Service Bus queue vs topic with an example
□ Can explain Event Grid sources and subscribers with an example
□ Can explain APIM policies with a security example
□ Can explain Logic Apps stateful vs stateless
□ Can explain Azure Functions triggers and bindings
□ Know when to use which service (decision guide)

AZURE LANDING ZONE (study from Module 14):
□ Can explain Management Group hierarchy and why it matters
□ Can explain Azure Policy effects (Deny, Audit, DeployIfNotExists)
□ Can explain why separate subscriptions for Prod and Non-Prod
□ Can explain PIM and just-in-time access

NETWORKING (study from Module 12):
□ Can explain Hub-Spoke topology and its benefits
□ Can explain what a Private Endpoint is and why it's needed
□ Can explain VPN Gateway vs ExpressRoute

CI/CD (study from Modules 11 and 19):
□ Can explain what a CI/CD pipeline does step by step
□ Can explain blue/green or canary deployment concepts
□ Can explain how infrastructure is deployed via Terraform

SECURITY (study from Modules 14 and 15):
□ Can explain Managed Identity + Key Vault pattern
□ Can explain zero-trust network principles
□ Can explain Azure RBAC and least-privilege principle

SQL (this module, Section 27.3):
□ Can write a SELECT with a WHERE clause
□ Can explain when to use Azure SQL vs SQL on VM
□ Can explain how Logic Apps connects to SQL
```

### Behavioural Checklist

```
STORIES TO PREPARE (STAR format):
□ A complex integration you designed from scratch
□ A time you migrated a legacy system to Azure
□ A time you mentored or led a junior developer
□ A time an integration failed in production and how you resolved it
□ A time you had to explain a complex decision to a non-technical person
□ A time you improved a team's DevOps maturity
□ A time you disagreed with an architectural decision and how you handled it
```

### Questions to Ask the Interviewer

```
These show you are thinking like a technical leader, not just a candidate:

1. "What is the current state of the integration platform — is it on Azure already 
   or are we building from scratch?"

2. "Which legacy integration platform are you looking to retire, and what is the 
   rough timeline for that transition?"

3. "How many integrations are currently running in production, and what does 
   the on-call support process look like?"

4. "How mature is the DevOps practice today — are there existing CI/CD pipelines 
   for integration deployments, or is that something I'd be establishing?"

5. "How is the relationship between the Development DevOps team and the 
   Architecture DevOps team structured — is this role expected to bridge them?"

6. "What does success look like in the first 90 days for this role?"
```

---

## 27.8 The One Paragraph Introduction (Memorize This)

> "I'm an Azure Integration and DevOps Engineer with hands-on experience designing and implementing cloud-native integration platforms from scratch. My background covers the full Azure integration stack — Logic Apps, API Management, Service Bus, Event Grid, and Functions — and I've used these to build event-driven architectures that decouple tightly coupled systems and modernise legacy integration platforms. I've built Azure Landing Zones with Hub-Spoke networking, enforced governance through Policy and RBAC, and automated everything through Azure DevOps CI/CD pipelines with Terraform for infrastructure. I work across both the technical delivery and the architecture side — I'm equally comfortable writing a Logic App expression, reviewing an architecture diagram, or explaining a design decision to a business stakeholder. I'm specifically looking for a role where I can take ownership of the integration platform and help the team build it in a way that scales."

---

## Summary

| Topic | Where to Study |
|-------|---------------|
| Integration Services (deep) | Module-13 |
| Landing Zones | Module-14 |
| Advanced Networking | Module-12 |
| CI/CD and Azure DevOps | Module-11, Module-19 |
| Architecture Design + full interview answer | Module-15 |
| Node.js / PowerShell scripting | Module-16 |
| SQL basics for integration | This module, Section 27.3 |
| Agile and Scrum | This module, Section 27.4 |
| SME and technical leadership | This module, Section 27.5 |
| JD-specific interview Q&A | This module, Section 27.6 |
