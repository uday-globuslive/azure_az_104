# Module 16 - Development and Scripting for DevOps Engineers (Complete Guide)

## Learning Objectives
By the end of this module, you will be able to:
- Write PowerShell scripts for Azure automation
- Develop .NET applications for Azure
- Build Node.js applications for cloud deployment
- Work with Azure SQL and databases
- Create automation scripts for DevOps workflows
- Implement REST API integrations
- Debug and troubleshoot cloud applications

---

## 16.1 PowerShell for Azure DevOps

### PowerShell Fundamentals for Automation

```powershell
# Variables and data types
$resourceGroup = "MyResourceGroup"
$location = "eastus"
$tags = @{
    Environment = "Production"
    Owner = "DevOps"
    CostCenter = "IT-001"
}

# Arrays and hashtables
$vmNames = @("web01", "web02", "web03")
$vmConfig = @{
    Size = "Standard_D2s_v3"
    Image = "Win2022Datacenter"
    AdminUser = "azureadmin"
}

# Loops
foreach ($vmName in $vmNames) {
    Write-Host "Processing VM: $vmName"
}

$vmNames | ForEach-Object {
    Write-Host "VM: $_"
}

# Conditional logic
if ($environment -eq "Production") {
    $skuName = "Premium"
} elseif ($environment -eq "Staging") {
    $skuName = "Standard"
} else {
    $skuName = "Basic"
}

# Switch statement
switch ($environment) {
    "Production" { $replication = "GRS" }
    "Staging" { $replication = "ZRS" }
    default { $replication = "LRS" }
}
```

### Azure PowerShell Module

```powershell
# Install Azure PowerShell module
Install-Module -Name Az -Repository PSGallery -Force

# Connect to Azure
Connect-AzAccount

# Set subscription context
Set-AzContext -SubscriptionId "subscription-id"

# Get current context
Get-AzContext
```

#### 🏢 Real-World Scripting Use Cases:

**Automated Environment Provisioning:**
```
Company: SoftwareStartup (20 developers, on-demand environments)
Challenge: Developers need isolated environments spun up quickly

PowerShell Solution:
┌─────────────────────────────────────────────────────────────────┐
│  Script: New-DevEnvironment.ps1                                  │
│  Purpose: Create full dev environment in 10 minutes             │
│                                                                  │
│  Parameters:                                                     │
│  ├── -DeveloperName: Creates unique resource naming            │
│  ├── -ApplicationType: web, api, microservices                 │
│  └── -ExpirationDays: Auto-delete after N days                 │
│                                                                  │
│  Resources Created:                                              │
│  ├── Resource Group: rg-dev-{DeveloperName}-001                │
│  ├── App Service Plan: asp-dev-{DeveloperName}                 │
│  ├── Web App: app-dev-{DeveloperName}                          │
│  ├── SQL Database: sql-dev-{DeveloperName}                      │
│  ├── Storage Account: stdev{DeveloperName}                      │
│  └── Key Vault: kv-dev-{DeveloperName}                         │
│                                                                  │
│  Automation:                                                     │
│  ├── Azure Automation runbook scheduled nightly                │
│  ├── Checks tag: ExpirationDate                                 │
│  └── Deletes expired environments automatically                 │
│                                                                  │
│  Sample Usage:                                                   │
│  ./New-DevEnvironment.ps1 -DeveloperName "john" `               │ │
│      -ApplicationType "web" -ExpirationDays 14                  │
└─────────────────────────────────────────────────────────────────┘

Key Script Logic:
param(
    [Parameter(Mandatory=$true)]
    [string]$DeveloperName,
    
    [ValidateSet("web","api","microservices")]
    [string]$ApplicationType = "web",
    
    [int]$ExpirationDays = 7
)

$expiration = (Get-Date).AddDays($ExpirationDays).ToString("yyyy-MM-dd")
$tags = @{ Environment="Development"; Owner=$DeveloperName; Expiration=$expiration }

# Create Resource Group
$rg = New-AzResourceGroup -Name "rg-dev-$DeveloperName" -Location "eastus" -Tag $tags

# Create all resources in parallel using jobs
$jobs = @()
$jobs += Start-Job -ScriptBlock { 
    New-AzAppServicePlan -ResourceGroupName $using:rg.ResourceGroupName ...
}
$jobs += Start-Job -ScriptBlock {
    New-AzSqlServer -ResourceGroupName $using:rg.ResourceGroupName ...
}

Wait-Job -Job $jobs  # Wait for parallel completion

Results:
├── Environment creation: 10 minutes (was 2 days with tickets)
├── Auto-cleanup: $5,000/month savings (no forgotten resources)
├── Developer self-service: Yes (no infra team bottleneck)
└── Consistency: 100% (same script, same output)
```

**Cost Optimization Automation:**
```
Company: EnterpriseCorp (100+ subscriptions, $500K/month Azure spend)
Challenge: Reduce costs by right-sizing and cleaning up waste

Automated Cost Controls:
┌─────────────────────────────────────────────────────────────────┐
│  Script 1: Stop-NonProdVMs.ps1 (Nightly 7 PM)                   │
│  ├── Find all VMs with tag: Environment != "Production"        │
│  ├── Stop VMs (deallocate to save compute costs)               │
│  └── Result: $30,000/month savings                              │
│                                                                  │
│  Script 2: Find-UnusedResources.ps1 (Weekly Report)             │
│  ├── Unattached disks (no VM association)                       │
│  ├── Unassociated Public IPs                                    │
│  ├── Empty storage accounts (no blobs for 90 days)             │
│  ├── Unused App Service Plans (no apps)                        │
│  └── Result: $15,000/month in identified waste                  │
│                                                                  │
│  Script 3: Resize-UnderutilizedVMs.ps1 (Monthly)                │
│  ├── Query Metrics: Get VMs with <10% CPU for 30 days          │
│  ├── Recommend: Downsize from D4s_v3 to D2s_v3                  │
│  ├── Auto-resize: If approved via Teams workflow               │
│  └── Result: $20,000/month savings                              │
│                                                                  │
│  Script 4: Apply-Reservations.ps1 (Quarterly Analysis)          │
│  ├── Analyze: Stable workloads running 24/7                    │
│  ├── Recommend: 1-year or 3-year reservations                  │
│  └── Result: 40% savings on qualifying resources               │
└─────────────────────────────────────────────────────────────────┘

Sample: VM Right-Sizing Script
$threshold = 10  # CPU percent
$days = 30

Get-AzVM | ForEach-Object {
    $metrics = Get-AzMetric -ResourceId $_.Id -MetricName "Percentage CPU" `
        -TimeGrain 01:00:00 -StartTime (Get-Date).AddDays(-$days)
    
    $avgCpu = ($metrics.Data.Average | Measure-Object -Average).Average
    
    if ($avgCpu -lt $threshold) {
        [PSCustomObject]@{
            VMName = $_.Name
            CurrentSize = $_.HardwareProfile.VmSize
            AvgCPU = [math]::Round($avgCpu, 2)
            Recommendation = "Consider downsizing"
        }
    }
} | Export-Csv "underutilized-vms.csv"

Total Results:
├── Monthly savings: $65,000 (13% reduction)
├── Annual savings: $780,000
├── Automation time: 0 hours (runs automatically)
└── Human time: 2 hours/month (review reports)
```

**Disaster Recovery Automation:**
```
Company: CriticalApps (Finance, RPO: 30 min, RTO: 2 hours)
Challenge: Automate failover to secondary region

DR Runbook:
┌─────────────────────────────────────────────────────────────────┐
│  Trigger: Azure Automation on health check failure             │
│                                                                  │
│  Step 1: Verify Primary is Actually Down                        │
│  ├── Multi-check: 5 failed health probes over 10 minutes       │
│  ├── Avoid: False positives (transient issues)                 │
│  └── If uncertain: Alert team, don't auto-failover             │
│                                                                  │
│  Step 2: Initiate Database Failover                             │
│  ├── Azure SQL: Invoke failover to geo-secondary               │
│  ├── Cosmos DB: Switch write region                            │
│  └── Wait: Replication catch-up (RPO verification)            │
│                                                                  │
│  Step 3: Update Traffic Routing                                 │
│  ├── Traffic Manager: Set primary endpoint to disabled         │
│  ├── Front Door: Update origin to secondary                    │
│  └── DNS: Lower TTL already in place (60 seconds)             │
│                                                                  │
│  Step 4: Scale Secondary Region                                 │
│  ├── App Service: Scale out to production capacity             │
│  ├── AKS: Scale node count to match primary                    │
│  └── Monitor: Ensure scaling completes                         │
│                                                                  │
│  Step 5: Notifications                                          │
│  ├── Teams/Slack: Alert operations team                        │
│  ├── Email: Notify stakeholders                                │
│  ├── PagerDuty: Incident created                               │
│  └── Status page: Update external status                       │
│                                                                  │
│  Step 6: Documentation                                           │
│  ├── Log: All actions with timestamps                          │
│  ├── Record: Actual RTO achieved                               │
│  └── Create: Post-mortem ticket                                │
└─────────────────────────────────────────────────────────────────┘

Failover Test Results (Quarterly DR drill):
├── Detection time: 2 minutes
├── Failover execution: 18 minutes
├── Total RTO achieved: 20 minutes (target: 2 hours) ✓
├── Data loss (RPO): 45 seconds (target: 30 min) ✓
└── Automation success: 100%
```

**.NET SDK Automation:**
```
Company: LogAnalytics Platform (Custom Azure integration)
Challenge: Complex automation beyond PowerShell capabilities

.NET Automation Service:
┌─────────────────────────────────────────────────────────────────┐
│  Use Case: Custom Resource Management API                       │
│                                                                  │
│  using Azure.ResourceManager;                                    │
│  using Azure.ResourceManager.Resources;                          │
│  using Azure.Identity;                                           │
│                                                                  │
│  // Authenticate using Managed Identity                          │
│  var credential = new DefaultAzureCredential();                 │
│  var client = new ArmClient(credential);                        │
│                                                                  │
│  // Custom business logic                                        │
│  public async Task<EnvResult> ProvisionEnvironment(EnvRequest req)  │
│  {                                                                │
│      // Validate request against business rules                 │
│      await ValidateBudget(req.CostCenter, req.EstimatedCost);   │
│      await ValidateApprovals(req.Requester, req.Level);         │
│                                                                  │
│      // Create resources with custom logic                      │
│      var rg = await CreateResourceGroup(req);                   │
│      var resources = await CreateResources(rg, req.Template);   │
│                                                                  │
│      // Post-creation automation                                 │
│      await ConfigureMonitoring(resources);                       │
│      await ConfigureBackup(resources);                           │
│      await NotifyStakeholders(req, resources);                  │
│                                                                  │
│      return new EnvResult { ResourceGroup = rg, ... };          │
│  }                                                                │
│                                                                  │
│  Why .NET over PowerShell:                                       │
│  ├── Complex business logic (budgets, approvals)               │
│  ├── Database integration (track all requests)                  │
│  ├── REST API exposure (self-service portal)                   │
│  ├── Unit testing (full test coverage)                         │
│  └── Performance (parallel operations at scale)                │
└─────────────────────────────────────────────────────────────────┘

Architecture:
┌─────────────────────────────────────────────────────────────────┐
│  Self-Service Portal (React)                                     │
│          │                                                        │
│          ▼                                                        │
│  API (ASP.NET Core + Azure SDK)                                  │
│          │                                                        │
│          ├──► Azure Resource Manager (create resources)         │
│          ├──► SQL Database (track requests)                     │
│          ├──► Service Bus (async long-running ops)              │
│          └──► Email/Teams (notifications)                       │
└─────────────────────────────────────────────────────────────────┘

Results:
├── Self-service IT: Reduced tickets by 80%
├── Provisioning time: 15 minutes (was 5 days)
├── Compliance: 100% (rules enforced in code)
└── Audit: Full history of all requests
```

### Resource Management Scripts

```powershell
# Create resources script
function New-AzureWebApp {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$AppName,
        
        [Parameter(Mandatory)]
        [string]$ResourceGroupName,
        
        [Parameter()]
        [string]$Location = "eastus",
        
        [Parameter()]
        [ValidateSet("Free", "Basic", "Standard", "Premium")]
        [string]$Tier = "Standard",
        
        [Parameter()]
        [hashtable]$Tags = @{}
    )
    
    begin {
        Write-Verbose "Starting web app deployment..."
    }
    
    process {
        try {
            # Create resource group if not exists
            $rg = Get-AzResourceGroup -Name $ResourceGroupName -ErrorAction SilentlyContinue
            if (-not $rg) {
                Write-Verbose "Creating resource group: $ResourceGroupName"
                $rg = New-AzResourceGroup -Name $ResourceGroupName -Location $Location -Tags $Tags
            }
            
            # Create App Service Plan
            $planName = "asp-$AppName"
            Write-Verbose "Creating App Service Plan: $planName"
            $plan = New-AzAppServicePlan `
                -ResourceGroupName $ResourceGroupName `
                -Name $planName `
                -Location $Location `
                -Tier $Tier `
                -NumberofWorkers 1 `
                -WorkerSize "Small"
            
            # Create Web App
            Write-Verbose "Creating Web App: $AppName"
            $webapp = New-AzWebApp `
                -ResourceGroupName $ResourceGroupName `
                -Name $AppName `
                -AppServicePlan $planName `
                -Location $Location
            
            # Configure app settings
            $appSettings = @{
                "ASPNETCORE_ENVIRONMENT" = "Production"
                "WEBSITE_TIME_ZONE" = "UTC"
            }
            Set-AzWebApp -ResourceGroupName $ResourceGroupName -Name $AppName -AppSettings $appSettings
            
            Write-Host "Web app created successfully: $($webapp.DefaultHostName)" -ForegroundColor Green
            return $webapp
        }
        catch {
            Write-Error "Failed to create web app: $_"
            throw
        }
    }
}

# Usage
$webapp = New-AzureWebApp `
    -AppName "mywebapp-001" `
    -ResourceGroupName "WebApp-RG" `
    -Location "eastus" `
    -Tier "Standard" `
    -Tags @{ Environment = "Production" } `
    -Verbose
```

### Automation Scripts

```powershell
# Automated deployment script
param (
    [Parameter(Mandatory)]
    [string]$Environment,
    
    [Parameter(Mandatory)]
    [string]$Version,
    
    [Parameter()]
    [string]$ConfigPath = ".\config"
)

# Load configuration
$config = Get-Content "$ConfigPath\$Environment.json" | ConvertFrom-Json

# Validate prerequisites
function Test-Prerequisites {
    $azModule = Get-Module -Name Az -ListAvailable
    if (-not $azModule) {
        throw "Azure PowerShell module not installed"
    }
    
    $context = Get-AzContext
    if (-not $context) {
        throw "Not logged in to Azure"
    }
    
    Write-Host "Prerequisites validated" -ForegroundColor Green
}

# Deploy infrastructure
function Deploy-Infrastructure {
    param ($Config)
    
    Write-Host "Deploying infrastructure for $($Config.environment)..." -ForegroundColor Cyan
    
    # Deploy ARM/Bicep template
    $deploymentParams = @{
        ResourceGroupName     = $Config.resourceGroup
        TemplateFile         = ".\templates\main.bicep"
        TemplateParameterObject = @{
            environment = $Config.environment
            location    = $Config.location
            appName     = $Config.appName
        }
    }
    
    $deployment = New-AzResourceGroupDeployment @deploymentParams -Verbose
    
    if ($deployment.ProvisioningState -eq "Succeeded") {
        Write-Host "Infrastructure deployed successfully" -ForegroundColor Green
        return $deployment.Outputs
    } else {
        throw "Infrastructure deployment failed: $($deployment.ProvisioningState)"
    }
}

# Deploy application
function Deploy-Application {
    param ($Config, $Version)
    
    Write-Host "Deploying application version $Version..." -ForegroundColor Cyan
    
    # Get web app
    $webapp = Get-AzWebApp -ResourceGroupName $Config.resourceGroup -Name $Config.appName
    
    # Deploy from package
    $packageUrl = "https://releases.example.com/app-$Version.zip"
    
    Publish-AzWebApp `
        -ResourceGroupName $Config.resourceGroup `
        -Name $Config.appName `
        -ArchivePath $packageUrl `
        -Force
    
    Write-Host "Application deployed successfully" -ForegroundColor Green
}

# Run health check
function Test-Deployment {
    param ($Config)
    
    $url = "https://$($Config.appName).azurewebsites.net/health"
    
    $maxRetries = 5
    $retryCount = 0
    
    while ($retryCount -lt $maxRetries) {
        try {
            $response = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 30
            if ($response.StatusCode -eq 200) {
                Write-Host "Health check passed" -ForegroundColor Green
                return $true
            }
        }
        catch {
            Write-Warning "Health check failed, retry $($retryCount + 1)/$maxRetries"
            Start-Sleep -Seconds 10
            $retryCount++
        }
    }
    
    throw "Health check failed after $maxRetries retries"
}

# Main execution
try {
    Test-Prerequisites
    $outputs = Deploy-Infrastructure -Config $config
    Deploy-Application -Config $config -Version $Version
    Test-Deployment -Config $config
    
    Write-Host "`nDeployment completed successfully!" -ForegroundColor Green
    Write-Host "Application URL: https://$($config.appName).azurewebsites.net"
}
catch {
    Write-Error "Deployment failed: $_"
    exit 1
}
```

### Azure CLI in Scripts

```powershell
# Using Azure CLI in PowerShell
function Invoke-AzureCLI {
    param (
        [Parameter(Mandatory)]
        [string]$Command
    )
    
    $output = az $Command.Split(' ') 2>&1
    
    if ($LASTEXITCODE -ne 0) {
        throw "Azure CLI command failed: $output"
    }
    
    return $output | ConvertFrom-Json
}

# Examples
$resourceGroups = Invoke-AzureCLI "group list"
$webApps = Invoke-AzureCLI "webapp list --resource-group MyRG"

# Mixed PowerShell and CLI
$storageAccounts = az storage account list --query "[].name" -o tsv
foreach ($account in $storageAccounts) {
    $keys = az storage account keys list --account-name $account --query "[0].value" -o tsv
    Write-Host "Account: $account, Key: $($keys.Substring(0,10))..."
}
```

---

## 16.2 .NET Development for Azure

### .NET Web API Project Structure

```
MyWebApi/
├── src/
│   └── MyWebApi/
│       ├── Controllers/
│       │   ├── HealthController.cs
│       │   └── OrdersController.cs
│       ├── Models/
│       │   ├── Order.cs
│       │   └── ApiResponse.cs
│       ├── Services/
│       │   ├── IOrderService.cs
│       │   └── OrderService.cs
│       ├── Program.cs
│       ├── appsettings.json
│       └── MyWebApi.csproj
├── tests/
│   └── MyWebApi.Tests/
│       ├── OrdersControllerTests.cs
│       └── MyWebApi.Tests.csproj
└── MyWebApi.sln
```

### ASP.NET Core Web API

```csharp
// Program.cs
using Microsoft.ApplicationInsights.Extensibility;
using Azure.Identity;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add Azure Key Vault configuration
if (!builder.Environment.IsDevelopment())
{
    var keyVaultUri = new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/");
    builder.Configuration.AddAzureKeyVault(keyVaultUri, new DefaultAzureCredential());
}

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry();

// Add custom services
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddHttpClient<IExternalApiClient, ExternalApiClient>();

// Add health checks
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")!)
    .AddAzureBlobStorage(builder.Configuration["StorageConnectionString"]!);

var app = builder.Build();

// Configure pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.MapHealthChecks("/health");

app.Run();
```

### Controller Example

```csharp
// Controllers/OrdersController.cs
using Microsoft.AspNetCore.Mvc;

namespace MyWebApi.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(IOrderService orderService, ILogger<OrdersController> logger)
    {
        _orderService = orderService;
        _logger = logger;
    }

    [HttpGet]
    [ProducesResponseType(typeof(IEnumerable<Order>), StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<Order>>> GetOrders(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        _logger.LogInformation("Getting orders, page {Page}, size {Size}", page, pageSize);
        
        var orders = await _orderService.GetOrdersAsync(page, pageSize);
        return Ok(orders);
    }

    [HttpGet("{id}")]
    [ProducesResponseType(typeof(Order), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<Order>> GetOrder(Guid id)
    {
        var order = await _orderService.GetOrderAsync(id);
        
        if (order == null)
        {
            _logger.LogWarning("Order {OrderId} not found", id);
            return NotFound();
        }
        
        return Ok(order);
    }

    [HttpPost]
    [ProducesResponseType(typeof(Order), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<Order>> CreateOrder([FromBody] CreateOrderRequest request)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        try
        {
            var order = await _orderService.CreateOrderAsync(request);
            _logger.LogInformation("Order {OrderId} created", order.Id);
            
            return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
        }
        catch (ValidationException ex)
        {
            _logger.LogWarning(ex, "Validation failed for order creation");
            return BadRequest(ex.Message);
        }
    }

    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateOrder(Guid id, [FromBody] UpdateOrderRequest request)
    {
        var result = await _orderService.UpdateOrderAsync(id, request);
        
        if (!result)
        {
            return NotFound();
        }
        
        return NoContent();
    }

    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    public async Task<IActionResult> DeleteOrder(Guid id)
    {
        await _orderService.DeleteOrderAsync(id);
        return NoContent();
    }
}
```

### Azure SDK Integration

```csharp
// Services/AzureStorageService.cs
using Azure.Storage.Blobs;
using Azure.Identity;

public class AzureStorageService : IStorageService
{
    private readonly BlobServiceClient _blobServiceClient;
    private readonly ILogger<AzureStorageService> _logger;

    public AzureStorageService(IConfiguration configuration, ILogger<AzureStorageService> logger)
    {
        _logger = logger;
        
        // Using managed identity
        var blobUri = new Uri($"https://{configuration["StorageAccountName"]}.blob.core.windows.net");
        _blobServiceClient = new BlobServiceClient(blobUri, new DefaultAzureCredential());
    }

    public async Task<string> UploadFileAsync(string containerName, string fileName, Stream content)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        await containerClient.CreateIfNotExistsAsync();
        
        var blobClient = containerClient.GetBlobClient(fileName);
        await blobClient.UploadAsync(content, overwrite: true);
        
        _logger.LogInformation("File {FileName} uploaded to {Container}", fileName, containerName);
        
        return blobClient.Uri.ToString();
    }

    public async Task<Stream> DownloadFileAsync(string containerName, string fileName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        var blobClient = containerClient.GetBlobClient(fileName);
        
        var response = await blobClient.DownloadStreamingAsync();
        return response.Value.Content;
    }
}
```

```csharp
// Services/ServiceBusService.cs
using Azure.Messaging.ServiceBus;

public class ServiceBusService : IMessageService, IAsyncDisposable
{
    private readonly ServiceBusClient _client;
    private readonly ServiceBusSender _sender;
    private readonly ILogger<ServiceBusService> _logger;

    public ServiceBusService(IConfiguration configuration, ILogger<ServiceBusService> logger)
    {
        _logger = logger;
        _client = new ServiceBusClient(
            configuration["ServiceBusNamespace"],
            new DefaultAzureCredential());
        _sender = _client.CreateSender(configuration["ServiceBusQueue"]);
    }

    public async Task SendMessageAsync<T>(T message, string? correlationId = null)
    {
        var json = JsonSerializer.Serialize(message);
        var serviceBusMessage = new ServiceBusMessage(json)
        {
            ContentType = "application/json",
            CorrelationId = correlationId ?? Guid.NewGuid().ToString(),
            MessageId = Guid.NewGuid().ToString()
        };

        await _sender.SendMessageAsync(serviceBusMessage);
        _logger.LogInformation("Message sent with ID {MessageId}", serviceBusMessage.MessageId);
    }

    public async Task SendBatchAsync<T>(IEnumerable<T> messages)
    {
        using var batch = await _sender.CreateMessageBatchAsync();
        
        foreach (var message in messages)
        {
            var json = JsonSerializer.Serialize(message);
            if (!batch.TryAddMessage(new ServiceBusMessage(json)))
            {
                throw new InvalidOperationException("Message too large for batch");
            }
        }

        await _sender.SendMessagesAsync(batch);
        _logger.LogInformation("Batch of {Count} messages sent", batch.Count);
    }

    public async ValueTask DisposeAsync()
    {
        await _sender.DisposeAsync();
        await _client.DisposeAsync();
    }
}
```

---

## 16.3 Node.js Development for Azure

### Express.js API Project

```javascript
// package.json
{
  "name": "azure-api",
  "version": "1.0.0",
  "main": "dist/index.js",
  "scripts": {
    "build": "tsc",
    "start": "node dist/index.js",
    "dev": "ts-node-dev src/index.ts",
    "test": "jest",
    "lint": "eslint src/**/*.ts"
  },
  "dependencies": {
    "@azure/identity": "^4.0.0",
    "@azure/keyvault-secrets": "^4.7.0",
    "@azure/service-bus": "^7.9.0",
    "@azure/storage-blob": "^12.17.0",
    "applicationinsights": "^2.9.0",
    "express": "^4.18.2",
    "helmet": "^7.1.0",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.10.0",
    "typescript": "^5.3.0"
  }
}
```

### TypeScript API Implementation

```typescript
// src/index.ts
import express, { Request, Response, NextFunction } from 'express';
import helmet from 'helmet';
import * as appInsights from 'applicationinsights';
import { DefaultAzureCredential } from '@azure/identity';
import { SecretClient } from '@azure/keyvault-secrets';
import { logger } from './utils/logger';
import { orderRouter } from './routes/orders';
import { healthRouter } from './routes/health';

// Initialize Application Insights
if (process.env.APPLICATIONINSIGHTS_CONNECTION_STRING) {
  appInsights.setup()
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .start();
}

const app = express();
const port = process.env.PORT || 3000;

// Middleware
app.use(helmet());
app.use(express.json());

// Request logging
app.use((req: Request, res: Response, next: NextFunction) => {
  logger.info(`${req.method} ${req.path}`, {
    ip: req.ip,
    userAgent: req.get('User-Agent')
  });
  next();
});

// Routes
app.use('/health', healthRouter);
app.use('/api/orders', orderRouter);

// Error handling
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  logger.error('Unhandled error', { error: err.message, stack: err.stack });
  res.status(500).json({ error: 'Internal Server Error' });
});

// Load secrets and start server
async function start() {
  try {
    // Load secrets from Key Vault
    const keyVaultName = process.env.KEY_VAULT_NAME;
    if (keyVaultName) {
      const credential = new DefaultAzureCredential();
      const client = new SecretClient(
        `https://${keyVaultName}.vault.azure.net`,
        credential
      );
      
      const dbPassword = await client.getSecret('database-password');
      process.env.DB_PASSWORD = dbPassword.value;
      
      logger.info('Secrets loaded from Key Vault');
    }

    app.listen(port, () => {
      logger.info(`Server running on port ${port}`);
    });
  } catch (error) {
    logger.error('Failed to start server', { error });
    process.exit(1);
  }
}

start();
```

### Azure SDK Integration (Node.js)

```typescript
// src/services/storage.service.ts
import { BlobServiceClient, ContainerClient } from '@azure/storage-blob';
import { DefaultAzureCredential } from '@azure/identity';
import { logger } from '../utils/logger';

export class StorageService {
  private blobServiceClient: BlobServiceClient;

  constructor() {
    const accountName = process.env.STORAGE_ACCOUNT_NAME!;
    const credential = new DefaultAzureCredential();
    
    this.blobServiceClient = new BlobServiceClient(
      `https://${accountName}.blob.core.windows.net`,
      credential
    );
  }

  async uploadFile(
    containerName: string,
    blobName: string,
    content: Buffer | string
  ): Promise<string> {
    const containerClient = this.blobServiceClient.getContainerClient(containerName);
    await containerClient.createIfNotExists();
    
    const blobClient = containerClient.getBlockBlobClient(blobName);
    await blobClient.upload(content, Buffer.byteLength(content));
    
    logger.info(`File uploaded: ${blobName}`);
    return blobClient.url;
  }

  async downloadFile(containerName: string, blobName: string): Promise<Buffer> {
    const containerClient = this.blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlobClient(blobName);
    
    const response = await blobClient.download();
    const chunks: Buffer[] = [];
    
    for await (const chunk of response.readableStreamBody!) {
      chunks.push(Buffer.from(chunk));
    }
    
    return Buffer.concat(chunks);
  }

  async listFiles(containerName: string, prefix?: string): Promise<string[]> {
    const containerClient = this.blobServiceClient.getContainerClient(containerName);
    const files: string[] = [];
    
    for await (const blob of containerClient.listBlobsFlat({ prefix })) {
      files.push(blob.name);
    }
    
    return files;
  }
}
```

```typescript
// src/services/servicebus.service.ts
import { ServiceBusClient, ServiceBusMessage, ServiceBusSender } from '@azure/service-bus';
import { DefaultAzureCredential } from '@azure/identity';
import { logger } from '../utils/logger';

export class ServiceBusService {
  private client: ServiceBusClient;
  private senders: Map<string, ServiceBusSender> = new Map();

  constructor() {
    const namespace = process.env.SERVICE_BUS_NAMESPACE!;
    const credential = new DefaultAzureCredential();
    
    this.client = new ServiceBusClient(
      `${namespace}.servicebus.windows.net`,
      credential
    );
  }

  private getSender(queueName: string): ServiceBusSender {
    if (!this.senders.has(queueName)) {
      this.senders.set(queueName, this.client.createSender(queueName));
    }
    return this.senders.get(queueName)!;
  }

  async sendMessage<T>(queueName: string, body: T, properties?: Partial<ServiceBusMessage>): Promise<void> {
    const sender = this.getSender(queueName);
    
    const message: ServiceBusMessage = {
      body: JSON.stringify(body),
      contentType: 'application/json',
      messageId: crypto.randomUUID(),
      ...properties
    };

    await sender.sendMessages(message);
    logger.info(`Message sent to ${queueName}`, { messageId: message.messageId });
  }

  async sendBatch<T>(queueName: string, items: T[]): Promise<void> {
    const sender = this.getSender(queueName);
    const batch = await sender.createMessageBatch();

    for (const item of items) {
      const message: ServiceBusMessage = {
        body: JSON.stringify(item),
        contentType: 'application/json',
        messageId: crypto.randomUUID()
      };

      if (!batch.tryAddMessage(message)) {
        throw new Error('Message too large for batch');
      }
    }

    await sender.sendMessages(batch);
    logger.info(`Batch sent to ${queueName}`, { count: items.length });
  }

  async close(): Promise<void> {
    for (const sender of this.senders.values()) {
      await sender.close();
    }
    await this.client.close();
  }
}
```

---

## 16.4 SQL Server and Database Development

### SQL Server Management

```sql
-- Database creation
CREATE DATABASE MyAppDB
ON PRIMARY (
    NAME = N'MyAppDB',
    FILENAME = N'/var/opt/mssql/data/MyAppDB.mdf',
    SIZE = 100MB,
    MAXSIZE = 10GB,
    FILEGROWTH = 100MB
)
LOG ON (
    NAME = N'MyAppDB_Log',
    FILENAME = N'/var/opt/mssql/data/MyAppDB_log.ldf',
    SIZE = 50MB,
    MAXSIZE = 5GB,
    FILEGROWTH = 50MB
);

-- Create schema
CREATE SCHEMA Integration;
GO

-- Orders table
CREATE TABLE Integration.Orders (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWSEQUENTIALID(),
    OrderNumber VARCHAR(50) NOT NULL UNIQUE,
    CustomerEmail VARCHAR(255) NOT NULL,
    Status VARCHAR(50) NOT NULL DEFAULT 'Pending',
    TotalAmount DECIMAL(18,2) NOT NULL,
    CreatedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    ModifiedAt DATETIME2 NULL,
    CreatedBy VARCHAR(100) NOT NULL,
    
    INDEX IX_Orders_Status NONCLUSTERED (Status),
    INDEX IX_Orders_CustomerEmail NONCLUSTERED (CustomerEmail),
    INDEX IX_Orders_CreatedAt NONCLUSTERED (CreatedAt DESC)
);

-- Order Items table
CREATE TABLE Integration.OrderItems (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    OrderId UNIQUEIDENTIFIER NOT NULL FOREIGN KEY REFERENCES Integration.Orders(Id),
    ProductId INT NOT NULL,
    ProductName VARCHAR(255) NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(18,2) NOT NULL,
    
    INDEX IX_OrderItems_OrderId NONCLUSTERED (OrderId)
);

-- Audit table
CREATE TABLE Integration.AuditLog (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    TableName VARCHAR(100) NOT NULL,
    RecordId VARCHAR(100) NOT NULL,
    Action VARCHAR(20) NOT NULL,
    OldValues NVARCHAR(MAX) NULL,
    NewValues NVARCHAR(MAX) NULL,
    ChangedBy VARCHAR(100) NOT NULL,
    ChangedAt DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);
```

### Stored Procedures

```sql
-- Create order stored procedure
CREATE OR ALTER PROCEDURE Integration.CreateOrder
    @OrderNumber VARCHAR(50),
    @CustomerEmail VARCHAR(255),
    @TotalAmount DECIMAL(18,2),
    @CreatedBy VARCHAR(100),
    @OrderItems Integration.OrderItemsType READONLY,
    @OrderId UNIQUEIDENTIFIER OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    
    BEGIN TRY
        BEGIN TRANSACTION;
        
        -- Insert order
        SET @OrderId = NEWID();
        
        INSERT INTO Integration.Orders (Id, OrderNumber, CustomerEmail, TotalAmount, CreatedBy)
        VALUES (@OrderId, @OrderNumber, @CustomerEmail, @TotalAmount, @CreatedBy);
        
        -- Insert order items
        INSERT INTO Integration.OrderItems (OrderId, ProductId, ProductName, Quantity, UnitPrice)
        SELECT @OrderId, ProductId, ProductName, Quantity, UnitPrice
        FROM @OrderItems;
        
        -- Log audit
        INSERT INTO Integration.AuditLog (TableName, RecordId, Action, NewValues, ChangedBy)
        VALUES ('Orders', CAST(@OrderId AS VARCHAR(100)), 'INSERT', 
                (SELECT TOP 1 * FROM Integration.Orders WHERE Id = @OrderId FOR JSON PATH),
                @CreatedBy);
        
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
GO

-- Update order status
CREATE OR ALTER PROCEDURE Integration.UpdateOrderStatus
    @OrderId UNIQUEIDENTIFIER,
    @NewStatus VARCHAR(50),
    @ModifiedBy VARCHAR(100)
AS
BEGIN
    SET NOCOUNT ON;
    
    DECLARE @OldStatus VARCHAR(50);
    
    SELECT @OldStatus = Status FROM Integration.Orders WHERE Id = @OrderId;
    
    IF @OldStatus IS NULL
    BEGIN
        RAISERROR('Order not found', 16, 1);
        RETURN;
    END
    
    UPDATE Integration.Orders
    SET Status = @NewStatus,
        ModifiedAt = GETUTCDATE()
    WHERE Id = @OrderId;
    
    INSERT INTO Integration.AuditLog (TableName, RecordId, Action, OldValues, NewValues, ChangedBy)
    VALUES ('Orders', CAST(@OrderId AS VARCHAR(100)), 'UPDATE',
            JSON_MODIFY('{}', '$.Status', @OldStatus),
            JSON_MODIFY('{}', '$.Status', @NewStatus),
            @ModifiedBy);
END;
GO

-- Get orders with pagination
CREATE OR ALTER PROCEDURE Integration.GetOrders
    @Page INT = 1,
    @PageSize INT = 10,
    @Status VARCHAR(50) = NULL,
    @StartDate DATE = NULL,
    @EndDate DATE = NULL,
    @TotalCount INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Get total count
    SELECT @TotalCount = COUNT(*)
    FROM Integration.Orders
    WHERE (@Status IS NULL OR Status = @Status)
      AND (@StartDate IS NULL OR CAST(CreatedAt AS DATE) >= @StartDate)
      AND (@EndDate IS NULL OR CAST(CreatedAt AS DATE) <= @EndDate);
    
    -- Get paginated results
    SELECT 
        o.Id,
        o.OrderNumber,
        o.CustomerEmail,
        o.Status,
        o.TotalAmount,
        o.CreatedAt,
        o.ModifiedAt,
        (
            SELECT oi.ProductName, oi.Quantity, oi.UnitPrice
            FROM Integration.OrderItems oi
            WHERE oi.OrderId = o.Id
            FOR JSON PATH
        ) AS Items
    FROM Integration.Orders o
    WHERE (@Status IS NULL OR o.Status = @Status)
      AND (@StartDate IS NULL OR CAST(o.CreatedAt AS DATE) >= @StartDate)
      AND (@EndDate IS NULL OR CAST(o.CreatedAt AS DATE) <= @EndDate)
    ORDER BY o.CreatedAt DESC
    OFFSET (@Page - 1) * @PageSize ROWS
    FETCH NEXT @PageSize ROWS ONLY;
END;
GO
```

### Database Migration Scripts

```sql
-- Migration: Add processed timestamp
-- Version: 2024.01.15.001

IF NOT EXISTS (
    SELECT * FROM sys.columns 
    WHERE object_id = OBJECT_ID('Integration.Orders') 
    AND name = 'ProcessedAt'
)
BEGIN
    ALTER TABLE Integration.Orders
    ADD ProcessedAt DATETIME2 NULL;
    
    PRINT 'Column ProcessedAt added to Orders table';
END
GO

-- Migration: Create index for processing
IF NOT EXISTS (
    SELECT * FROM sys.indexes 
    WHERE name = 'IX_Orders_ProcessedAt' 
    AND object_id = OBJECT_ID('Integration.Orders')
)
BEGIN
    CREATE NONCLUSTERED INDEX IX_Orders_ProcessedAt
    ON Integration.Orders (ProcessedAt)
    WHERE ProcessedAt IS NULL;
    
    PRINT 'Index IX_Orders_ProcessedAt created';
END
GO
```

---

## 16.5 REST API Integration

### HTTP Client Implementation

```csharp
// C# - HttpClient with resilience
public class ExternalApiClient : IExternalApiClient
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<ExternalApiClient> _logger;

    public ExternalApiClient(HttpClient httpClient, ILogger<ExternalApiClient> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    public async Task<T?> GetAsync<T>(string endpoint, CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _httpClient.GetAsync(endpoint, cancellationToken);
            response.EnsureSuccessStatusCode();
            
            return await response.Content.ReadFromJsonAsync<T>(cancellationToken: cancellationToken);
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "HTTP GET failed for {Endpoint}", endpoint);
            throw;
        }
    }

    public async Task<TResponse?> PostAsync<TRequest, TResponse>(
        string endpoint, 
        TRequest body, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _httpClient.PostAsJsonAsync(endpoint, body, cancellationToken);
            response.EnsureSuccessStatusCode();
            
            return await response.Content.ReadFromJsonAsync<TResponse>(cancellationToken: cancellationToken);
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "HTTP POST failed for {Endpoint}", endpoint);
            throw;
        }
    }
}

// Configure with Polly for resilience
services.AddHttpClient<IExternalApiClient, ExternalApiClient>(client =>
{
    client.BaseAddress = new Uri(configuration["ExternalApi:BaseUrl"]!);
    client.DefaultRequestHeaders.Add("Accept", "application/json");
})
.AddTransientHttpErrorPolicy(policy => policy.WaitAndRetryAsync(3, retryAttempt =>
    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))))
.AddTransientHttpErrorPolicy(policy => policy.CircuitBreakerAsync(5, TimeSpan.FromMinutes(1)));
```

### PowerShell API Integration

```powershell
# REST API helper functions
function Invoke-ApiRequest {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Endpoint,
        
        [Parameter()]
        [ValidateSet("GET", "POST", "PUT", "DELETE")]
        [string]$Method = "GET",
        
        [Parameter()]
        [hashtable]$Body,
        
        [Parameter()]
        [hashtable]$Headers = @{},
        
        [Parameter()]
        [int]$RetryCount = 3,
        
        [Parameter()]
        [int]$RetryDelaySeconds = 2
    )
    
    $Headers["Content-Type"] = "application/json"
    $Headers["Accept"] = "application/json"
    
    $params = @{
        Uri = $Endpoint
        Method = $Method
        Headers = $Headers
        UseBasicParsing = $true
    }
    
    if ($Body) {
        $params["Body"] = $Body | ConvertTo-Json -Depth 10
    }
    
    $attempt = 0
    while ($attempt -lt $RetryCount) {
        try {
            $response = Invoke-RestMethod @params
            return $response
        }
        catch {
            $attempt++
            if ($attempt -eq $RetryCount) {
                throw "API request failed after $RetryCount attempts: $_"
            }
            Write-Warning "Request failed, retrying in $RetryDelaySeconds seconds..."
            Start-Sleep -Seconds $RetryDelaySeconds
        }
    }
}

# Usage examples
$token = "Bearer $accessToken"

# GET request
$orders = Invoke-ApiRequest `
    -Endpoint "https://api.example.com/orders" `
    -Headers @{ Authorization = $token }

# POST request
$newOrder = Invoke-ApiRequest `
    -Endpoint "https://api.example.com/orders" `
    -Method "POST" `
    -Headers @{ Authorization = $token } `
    -Body @{
        customerEmail = "customer@example.com"
        items = @(
            @{ productId = 1; quantity = 2 }
            @{ productId = 5; quantity = 1 }
        )
    }
```

---

## 16.6 Testing and Debugging

### Unit Testing (.NET)

```csharp
// OrdersControllerTests.cs
using Microsoft.AspNetCore.Mvc;
using Moq;
using Xunit;

public class OrdersControllerTests
{
    private readonly Mock<IOrderService> _orderServiceMock;
    private readonly Mock<ILogger<OrdersController>> _loggerMock;
    private readonly OrdersController _controller;

    public OrdersControllerTests()
    {
        _orderServiceMock = new Mock<IOrderService>();
        _loggerMock = new Mock<ILogger<OrdersController>>();
        _controller = new OrdersController(_orderServiceMock.Object, _loggerMock.Object);
    }

    [Fact]
    public async Task GetOrder_ExistingId_ReturnsOrder()
    {
        // Arrange
        var orderId = Guid.NewGuid();
        var expectedOrder = new Order { Id = orderId, Status = "Completed" };
        
        _orderServiceMock
            .Setup(s => s.GetOrderAsync(orderId))
            .ReturnsAsync(expectedOrder);

        // Act
        var result = await _controller.GetOrder(orderId);

        // Assert
        var okResult = Assert.IsType<OkObjectResult>(result.Result);
        var returnedOrder = Assert.IsType<Order>(okResult.Value);
        Assert.Equal(orderId, returnedOrder.Id);
    }

    [Fact]
    public async Task GetOrder_NonExistingId_ReturnsNotFound()
    {
        // Arrange
        var orderId = Guid.NewGuid();
        _orderServiceMock
            .Setup(s => s.GetOrderAsync(orderId))
            .ReturnsAsync((Order?)null);

        // Act
        var result = await _controller.GetOrder(orderId);

        // Assert
        Assert.IsType<NotFoundResult>(result.Result);
    }

    [Fact]
    public async Task CreateOrder_ValidRequest_ReturnsCreatedResult()
    {
        // Arrange
        var request = new CreateOrderRequest
        {
            CustomerEmail = "test@example.com",
            Items = new List<OrderItemRequest>
            {
                new() { ProductId = 1, Quantity = 2 }
            }
        };
        
        var createdOrder = new Order
        {
            Id = Guid.NewGuid(),
            Status = "Pending"
        };

        _orderServiceMock
            .Setup(s => s.CreateOrderAsync(It.IsAny<CreateOrderRequest>()))
            .ReturnsAsync(createdOrder);

        // Act
        var result = await _controller.CreateOrder(request);

        // Assert
        var createdResult = Assert.IsType<CreatedAtActionResult>(result.Result);
        Assert.Equal(nameof(OrdersController.GetOrder), createdResult.ActionName);
    }
}
```

### Integration Testing (Node.js)

```typescript
// tests/orders.test.ts
import request from 'supertest';
import { app } from '../src/index';
import { db } from '../src/database';

describe('Orders API', () => {
  beforeEach(async () => {
    await db.query('DELETE FROM orders');
  });

  afterAll(async () => {
    await db.close();
  });

  describe('GET /api/orders', () => {
    it('should return empty array when no orders exist', async () => {
      const response = await request(app)
        .get('/api/orders')
        .expect(200);

      expect(response.body).toEqual([]);
    });

    it('should return orders with pagination', async () => {
      // Create test orders
      await db.query(`
        INSERT INTO orders (customer_email, status) 
        VALUES ('test@example.com', 'Pending')
      `);

      const response = await request(app)
        .get('/api/orders?page=1&pageSize=10')
        .expect(200);

      expect(response.body).toHaveLength(1);
      expect(response.body[0].customer_email).toBe('test@example.com');
    });
  });

  describe('POST /api/orders', () => {
    it('should create order with valid data', async () => {
      const orderData = {
        customerEmail: 'new@example.com',
        items: [
          { productId: 1, quantity: 2, unitPrice: 10.00 }
        ]
      };

      const response = await request(app)
        .post('/api/orders')
        .send(orderData)
        .expect(201);

      expect(response.body.id).toBeDefined();
      expect(response.body.customerEmail).toBe(orderData.customerEmail);
      expect(response.body.status).toBe('Pending');
    });

    it('should return 400 for invalid data', async () => {
      const response = await request(app)
        .post('/api/orders')
        .send({ invalidField: 'test' })
        .expect(400);

      expect(response.body.error).toBeDefined();
    });
  });
});
```

### PowerShell Testing (Pester)

```powershell
# Tests/Deploy-WebApp.Tests.ps1
BeforeAll {
    . $PSScriptRoot/../Scripts/Deploy-WebApp.ps1
}

Describe "New-AzureWebApp" {
    BeforeAll {
        # Mock Azure commands
        Mock New-AzResourceGroup { return @{ ResourceGroupName = "Test-RG" } }
        Mock New-AzAppServicePlan { return @{ Name = "TestPlan" } }
        Mock New-AzWebApp { return @{ DefaultHostName = "test.azurewebsites.net" } }
        Mock Set-AzWebApp { }
    }

    It "Creates resource group if not exists" {
        Mock Get-AzResourceGroup { return $null }
        
        $result = New-AzureWebApp -AppName "testapp" -ResourceGroupName "Test-RG"
        
        Should -Invoke New-AzResourceGroup -Times 1
    }

    It "Does not create resource group if exists" {
        Mock Get-AzResourceGroup { return @{ ResourceGroupName = "Test-RG" } }
        
        $result = New-AzureWebApp -AppName "testapp" -ResourceGroupName "Test-RG"
        
        Should -Invoke New-AzResourceGroup -Times 0
    }

    It "Creates web app with correct tier" {
        $result = New-AzureWebApp -AppName "testapp" -ResourceGroupName "Test-RG" -Tier "Premium"
        
        Should -Invoke New-AzAppServicePlan -ParameterFilter { $Tier -eq "Premium" }
    }

    It "Returns web app URL" {
        $result = New-AzureWebApp -AppName "testapp" -ResourceGroupName "Test-RG"
        
        $result.DefaultHostName | Should -Be "test.azurewebsites.net"
    }
}

Describe "Test-Prerequisites" {
    It "Throws when not logged in" {
        Mock Get-AzContext { return $null }
        
        { Test-Prerequisites } | Should -Throw "*Not logged in*"
    }

    It "Passes when logged in" {
        Mock Get-AzContext { return @{ Subscription = @{ Id = "test" } } }
        Mock Get-Module { return @{ Name = "Az" } }
        
        { Test-Prerequisites } | Should -Not -Throw
    }
}
```

---

## 16.7 Summary and Best Practices

### Development Best Practices

| Area | Best Practice |
|------|---------------|
| **Error Handling** | Use structured exceptions, log errors properly |
| **Configuration** | Externalize config, use environment variables |
| **Secrets** | Never hardcode, use Key Vault |
| **Logging** | Structured logging with correlation IDs |
| **Testing** | Unit tests, integration tests, code coverage |
| **Performance** | Async/await, connection pooling, caching |

### Key Takeaways

1. **PowerShell** - Essential for Azure automation and DevOps scripts
2. **.NET** - Primary platform for enterprise Azure applications
3. **Node.js** - Great for APIs, serverless, and microservices
4. **SQL** - Foundation for data persistence and queries
5. **Testing** - Critical for reliable deployments

---

## Additional Resources

- [Azure PowerShell Documentation](https://docs.microsoft.com/powershell/azure/)
- [Azure SDK for .NET](https://docs.microsoft.com/dotnet/azure/)
- [Azure SDK for JavaScript](https://docs.microsoft.com/javascript/azure/)
- [Azure SQL Documentation](https://docs.microsoft.com/azure/azure-sql/)
