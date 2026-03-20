# CIPP API — Full Documentation

## Overview

This repository is the **backend logic layer for the CIPP platform**.

It is a **PowerShell-based Azure Functions application** that exposes a large HTTP API and background-processing engine for Microsoft 365 / tenant administration workflows. It also includes:

- Azure Durable Functions orchestration
- Azure Queue processing
- Azure Timer processing
- Azure Table / storage-backed configuration and state
- bundled internal PowerShell modules
- local Docker-based development support
- GitHub Actions deployment and release workflows

This is **not** a simple single-script tool. It is a substantial function-app backend with many endpoints and helper modules.

Based on the code and deployment files, this app is primarily designed to be:

- **Azure-hosted**, as an **Azure Functions app**
- optionally run locally for development
- optionally containerized for local or custom-hosted testing

There is strong evidence of Azure hosting throughout the repo:
- `host.json`
- Azure Functions trigger folders
- `Dockerfile` based on Azure Functions PowerShell runtime
- GitHub Actions using `Azure/functions-action`
- Azure Blob release publishing workflows

---

## What it is

CIPP API is the **backend/API engine** for CIPP.

At a high level it:

1. receives HTTP API calls
2. routes requests to internal command handlers
3. executes PowerShell-based operational tasks
4. offloads work into Azure Queue and Durable Function workflows when needed
5. runs recurring timer-based maintenance and analysis
6. stores configuration and state in Azure-backed storage
7. exposes many operational endpoints through one central HTTP trigger

It appears to support a broad range of tenant-management capabilities such as:
- Microsoft 365 administration
- standards application
- best-practice analysis
- Intune and endpoint actions
- queue-based operations
- reporting and tenant analysis
- packaged app/deployment support
- update/release distribution metadata

---

## What it does

The codebase is organized around several Azure Function entry points:

- **HTTP trigger**
- **Queue trigger**
- **Durable orchestrator**
- **Timer trigger**
- **Activity trigger**

These all use a shared entrypoint module:

```text
Modules/CippEntrypoints/CippEntrypoints.psm1
```

The function folders confirm this structure:

- `CIPPHttpTrigger/function.json`
- `CIPPQueueTrigger/function.json`
- `CIPPOrchestrator/function.json`
- `CIPPTimer/function.json`
- `CIPPActivityFunction/function.json`

### Main HTTP behavior
The primary API surface is exposed through one catch-all HTTP route:

```json
"route": "{*CIPPEndpoint}"
```

That means the app behaves like a routed API gateway inside Azure Functions.

### Queue and orchestration behavior
Longer or scalable work is offloaded into:
- queue processing
- activity functions
- durable orchestrations

This lets the app handle larger multi-step operations beyond a single HTTP request.

### Timer behavior
A timer trigger runs every 15 minutes:

```json
"schedule": "0 0/15 * * * *"
```

This is used for recurring background processing and scheduled operational logic.

### Batch request support
The HTTP entrypoint includes special handling for:

```text
$batch
```

with a Graph-style batch request model. It validates:
- that batch requests exist
- that no more than 20 requests are sent per batch
- and executes them in parallel with a throttle limit

That means the API supports multi-request batching natively.

---

## High-level architecture

## Runtime platform
- **Azure Functions v4**
- **PowerShell 7.4**
- **Azure Durable Functions**
- **Azure Storage / queue-backed runtime**
- optional **Application Insights**

## Main runtime files
- `host.json` — function app runtime configuration
- `profile.ps1` — startup/bootstrap script
- `requirements.psd1` — intentionally empty because modules are bundled locally
- `Modules/CIPPCore` — primary shared module
- `Modules/CippEntrypoints` — trigger entrypoint handlers
- `Modules/CippExtensions` — extension logic
- `Modules/AzBobbyTables` — table/storage helper support

## Trigger folders
- `CIPPHttpTrigger`
- `CIPPQueueTrigger`
- `CIPPOrchestrator`
- `CIPPTimer`
- `CIPPActivityFunction`

## Additional content
- `Config/` — templates and configuration JSON
- `Tools/` — support scripts and helper tooling
- `.github/workflows/` — deployment and release automation
- `AddMSPApp/` and `AddChocoApp/` — packaged application assets
- `Tests/` — Pester-style PowerShell tests

---

## Technologies used

This project uses a broad stack.

### Azure Functions
The core hosting/runtime platform.

Evidence:
- `host.json`
- function folders with `function.json`
- Azure Functions Docker base image
- GitHub deploy workflows using Azure Functions action

### PowerShell 7.4
Primary application language/runtime.

Evidence:
- `FUNCTIONS_WORKER_RUNTIME='powershell'`
- `FUNCTIONS_WORKER_RUNTIME_VERSION='7.4'`
- Docker image `mcr.microsoft.com/azure-functions/powershell:4-powershell7.4`

### Azure Durable Functions
Used for orchestrated and fan-out/fan-in background work.

Evidence:
- `orchestrationTrigger`
- `activityTrigger`
- durable client bindings
- orchestration logic in `Receive-CippOrchestrationTrigger`

### Azure Queue Storage
Used for deferred/background task execution.

Evidence:
- queue output bindings to `cippqueue`
- queue trigger input on `cippqueue`

### Azure Storage / Azurite
Used for function storage and local development.

Evidence:
- `.env.example` contains `AzureWebJobsStorage`
- local compose includes `azurite`
- queue/table/blob endpoints are wired in local config

### Application Insights
Optional telemetry and logging are supported if the environment variables are present.

Evidence:
- `APPLICATIONINSIGHTS_CONNECTION_STRING`
- `APPINSIGHTS_INSTRUMENTATIONKEY`
- startup loading of `Microsoft.ApplicationInsights.dll`

### PowerShell modules
Key bundled modules include:
- `CIPPCore`
- `CippEntrypoints`
- `CippExtensions`
- `AzBobbyTables`
- `AzureFunctions.PowerShell.Durable.SDK`
- `DNSHealth`
- `HuduAPI`
- `MicrosoftTeams`
- `PassPushPosh`

### Docker / Docker Compose
Used for local development and local multi-instance function testing.

Evidence:
- `Dockerfile`
- `docker-compose.yml`
- `nginx.conf`

### Nginx
Used in local compose to front multiple app replicas.

### GitHub Actions
Used for:
- Azure deployment
- release generation
- Azure Blob distribution uploads
- PR policy checks

### OpenAPI / API docs
The repository includes:
- `openapi.json`
- multiple generated OpenAPI specs under `Tools/`

### JSON template/config system
The app ships many JSON templates and policy/standard definition files under `Config/`.

---

## Repository structure

## Top-level important files

### `host.json`
Defines the Azure Functions host configuration, including:
- extension bundle version
- durable task settings
- concurrency controls
- function timeout of 10 minutes

### `profile.ps1`
Runs during startup and bootstraps the app by:
- writing startup diagnostics
- loading Application Insights if configured
- importing local modules
- retrieving authentication/keys
- checking/storing version information
- preparing environment state

### `requirements.psd1`
Intentionally empty. The repo bundles modules locally instead of relying on managed dependency download.

### `Dockerfile`
Builds the app into an Azure Functions PowerShell 7.4 container.

### `docker-compose.yml`
Provides local development orchestration with:
- Azurite
- the function app
- nginx
- multiple app replicas

### `.env.example`
Provides local development environment variables for the Docker/dev path.

### `openapi.json`
Documents a large set of HTTP endpoints.

---

## Trigger folders

### `CIPPHttpTrigger`
The main HTTP API trigger.

Important properties:
- `authLevel: "anonymous"`
- accepts `GET`, `POST`, `PATCH`, `PUT`, `DELETE`, `OPTIONS`
- catch-all route pattern
- has durable client input
- has queue output binding

### `CIPPQueueTrigger`
Processes queued tasks from `cippqueue`.

### `CIPPOrchestrator`
Receives orchestration trigger input for durable workflows.

### `CIPPTimer`
Runs every 15 minutes and can also enqueue work.

### `CIPPActivityFunction`
Executes durable activity work.

---

## Shared modules

### `Modules/CIPPCore`
Core module containing many public and private functions. This appears to be the main shared logic library for the application.

### `Modules/CippEntrypoints`
Contains the trigger entry functions:
- `Receive-CippHttpTrigger`
- `Receive-CippQueueTrigger`
- `Receive-CippOrchestrationTrigger`
- `Receive-CippTimerTrigger`
- `Receive-CippActivityTrigger`

### `Modules/CippExtensions`
Additional extension logic.

### `Modules/AzBobbyTables`
Azure storage/table helper logic.

---

## How it works

## Startup sequence

When the function app starts, `profile.ps1` runs.

It does the following:

1. writes startup information
2. conditionally loads Application Insights assemblies if telemetry is configured
3. imports bundled modules:
   - `CIPPCore`
   - `CippExtensions`
   - `AzBobbyTables`
4. optionally imports external durable SDK support
5. retrieves authentication/keys with `Get-CIPPAuthentication`
6. loads the current version from `version_latest.txt`
7. records startup/version state into a table entry

This means the app depends on successful startup initialization before request handling is reliable.

---

## HTTP request flow

The HTTP function calls:

```powershell
Receive-CippHttpTrigger
```

Key behaviors include:

### Cold-start logging
If the `x-ms-coldstart` header is present, the app logs a cold-start event.

### Function offload protection
The code checks a config table for an `OffloadFunctions` setting and may block direct API calls on processor-only function apps.

### Request normalization
The incoming request is converted through JSON and back into an object because of case-sensitivity concerns in newer PowerShell versions.

### Route handling
The catch-all route exposes the endpoint value as:

```powershell
$Request.Params.CIPPEndpoint
```

### `$batch` support
If the endpoint is `$batch`, the app:
- validates request count
- rejects more than 20 items
- processes in parallel
- returns Graph-style batched responses

### Normal API handling
For non-batch requests, it calls:

```powershell
New-CippCoreRequest
```

and returns a structured HTTP response.

---

## Queue request flow

The queue trigger:

```powershell
Receive-CippQueueTrigger
```

does the following:

1. receives a queue item
2. normalizes it through JSON
3. validates the command exists in `CIPPCore`
4. extracts:
   - `Cmdlet`
   - `Parameters`
5. executes the cmdlet dynamically

This means queue items can invoke internal command handlers asynchronously.

---

## Orchestration flow

The durable orchestrator:

```powershell
Receive-CippOrchestrationTrigger
```

supports different execution models:

- `FanOut`
- `Sequence`
- `NoScaling`
- default fallback behavior

Important behaviors:
- parses orchestration input
- sets durable custom status
- builds retry options
- optionally changes behavior on Premium SKU
- supports either direct batch input or a queue function that generates batch items
- invokes activity functions with retry behavior

This is a core part of how multi-step actions are scaled.

---

## Local development/runtime model

The repo includes local development support.

### Docker Compose stack
The provided compose file includes:
- `azurite`
- `cippapi`
- `nginx`

### Local storage emulator
Azurite is used to emulate Azure storage locally.

### Multiple replicas
The compose file declares:

```yaml
deploy:
  replicas: 3
```

for the function app service.

### Nginx fronting
Nginx listens on port `7071` and proxies to the backend app instances.

This suggests local load-balanced development/testing scenarios are supported.

---

## Hosting model

## Primary hosting model
This application is clearly designed to be hosted as an **Azure Function App**.

Evidence:
- Azure Functions runtime files
- Azure Functions Docker image
- Azure Functions GitHub deployment actions
- Azure-specific bindings and durable triggers

## Secondary/local hosting model
It can also be run:
- locally with Docker Compose
- potentially in custom container hosting, if desired

But the main production design is Azure Functions.

---

## How to run

## Option 1: Azure-hosted deployment
This is the intended production path.

Typical flow:
1. provision Azure Function App
2. configure required app settings/secrets/storage
3. deploy the repo contents
4. verify startup, storage, and module loading
5. validate endpoint behavior

## Option 2: Local Docker-based development
The repo includes all the necessary local container files.

### Steps
1. copy `.env.example` to a local env file
2. adjust environment variables if needed
3. run Docker Compose

Example:

```bash
docker compose up --build
```

### Local ports
Based on compose, nginx exposes:

```text
7071
```

So local access is expected via:
```text
http://localhost:7071
```

## Option 3: Azure Functions local tooling
The repository structure is compatible with Azure Functions local execution, but the shipped local path is clearly Docker-first due to included Azurite and nginx support.

---

## How to build

This app is not “built” in the traditional compiled-web-app sense. The practical build/deployment modes are:

### 1. Zip/package deployment to Azure Functions
The GitHub workflows deploy the repository root directly using:

```yaml
uses: Azure/functions-action@v1
```

### 2. Container build
The app can be containerized using the supplied `Dockerfile`.

Example:

```bash
docker build -t cipp-api .
```

### 3. Local compose build
The compose stack will build the function app automatically.

Example:

```bash
docker compose up --build
```

---

## Deployment and release automation

The repo includes several workflows.

### Azure Function deployment
Examples:
- `dev_api.yml`
- `master_cipptpd3d.yml`
- `master_cipptsm7y.yml`

These deploy to Azure Function Apps using:
- OIDC Azure login in one case
- publish profiles in others

### Release publishing
Examples:
- `publish_release.yml`
- `publish_prerelease.yml`
- `upload_dev.yml`

These:
- read `version_latest.txt`
- create release zips
- publish GitHub releases
- upload release artifacts to Azure Blob Storage

This means the repo includes both deployment automation and product/release distribution automation.

---

## Dependencies

## Runtime dependencies

### Azure Functions PowerShell runtime
Required.

### PowerShell 7.4
Required by the configured runtime/container settings.

### Azure Storage
Required for Azure Functions runtime state and bindings.

### Durable Functions support
Required for orchestrator/activity paths.

### Bundled PowerShell modules
The app depends on bundled local modules instead of managed dependencies.

These include at minimum:
- `CIPPCore`
- `CippExtensions`
- `AzBobbyTables`

### Optional Application Insights
Used when telemetry variables are configured.

---

## Infrastructure/runtime dependencies

The app also depends on:
- Azure Function App hosting
- Azure Storage account
- queue/table/blob availability
- any tenant auth/secrets retrieved at startup
- potentially Key Vault or equivalent secret retrieval path

The code in `profile.ps1` indicates startup auth retrieval is important.

---

## Configuration

## Local development environment variables
From `.env.example`, the local stack expects:

- `FUNCTIONS_WORKER_RUNTIME`
- `FUNCTIONS_WORKER_RUNTIME_VERSION`
- `AzureWebJobsStorage`
- `DEV_SKIP_BPA_TIMER`
- `DEV_SKIP_DOMAIN_TIMER`
- `FUNCTIONS_EXTENSION_VERSION`
- `NonLocalHostAzurite`

### Example local values
The sample file uses:
- PowerShell worker runtime
- runtime version 7.4
- Azurite-based storage endpoints
- optional timer-skip flags for local dev

## Azure app settings
Based on the code, likely important Azure settings include:
- `APPLICATIONINSIGHTS_CONNECTION_STRING`
- `APPINSIGHTS_INSTRUMENTATIONKEY`
- `WEBSITE_HOSTNAME`
- `WEBSITE_SITE_NAME`
- `WEBSITE_SKU`
- `AzureWebJobsStorage`

There are likely more auth and platform-specific settings retrieved indirectly through internal functions at startup.

---

## Security model

### HTTP auth level
The HTTP trigger is configured as:

```json
"authLevel": "anonymous"
```

This means Azure Functions host-level auth is not enforcing request authentication itself.

Important implication:
- security is likely enforced inside the application logic or by upstream platform controls
- you should verify the real authentication model before exposing it directly

### Secrets and auth retrieval
`profile.ps1` indicates keys/auth are retrieved at startup, including possible Key Vault access.

### Telemetry
Application Insights is optional, not mandatory.

---

## How to amend or add to it

This app is large, so safe extension matters.

## Add a new HTTP endpoint
Because routing is centralized through the catch-all HTTP trigger, adding a new endpoint usually means:
1. define or extend request handling in the internal routing logic
2. implement the underlying cmdlet/function in `CIPPCore` or an extension module
3. update OpenAPI docs if needed
4. add tests

The actual HTTP trigger itself usually does **not** need another Azure Function folder; it already catches all routes.

## Add a new queue-based operation
To add async work:
1. implement the cmdlet in `CIPPCore`
2. enqueue a queue item with:
   - `Cmdlet`
   - `Parameters`
3. allow `Receive-CippQueueTrigger` to execute it
4. add validation/logging/tests

## Add a new durable workflow
To add an orchestration path:
1. define the orchestration input model
2. generate batch items or queue-function results
3. implement corresponding activity behavior
4. use `FanOut`, `Sequence`, or `NoScaling` as appropriate
5. test retry and batching behavior

## Add new shared functionality
Place reusable logic in:
- `Modules/CIPPCore/Public`
- `Modules/CIPPCore/Private`
- or `Modules/CippExtensions`

Because `CIPPCore.psm1` imports all public/private scripts from those directories when running from source, that is the natural extension point.

## Add or amend templates
The `Config/` folder contains many JSON templates. To extend standards/templates:
1. add the new JSON template file
2. follow existing naming/schema conventions
3. update any lookup or reference logic that expects it

## Add tests
The repo already contains PowerShell tests under `Tests/`, so new functionality should follow the same pattern and add Pester tests in the relevant area.

## Add release/deploy changes
The repo already uses GitHub Actions extensively. Deployment/release changes should be made in:
- `.github/workflows/`
- release zip generation logic
- blob upload workflow definitions

---

## Troubleshooting with rectification

## 1. Function app starts but endpoints fail immediately

### Cause
Common causes:
- startup module import failed
- authentication/secret retrieval failed in `profile.ps1`
- storage access is broken
- missing required environment variables

### Fix
Check startup logs first. Specifically verify:
- `CIPPCore`, `CippExtensions`, and `AzBobbyTables` import successfully
- Application Insights load is not throwing unexpected issues
- `Get-CIPPAuthentication` completes
- `version_latest.txt` is present and readable
- `AzureWebJobsStorage` is valid

If module imports fail, verify the `Modules/` folder was deployed intact.

---

## 2. Azure Function deployment succeeds but the app does not behave correctly

### Cause
Possible causes:
- partial/incomplete deployment artifact
- wrong runtime version
- missing app settings
- storage misconfiguration
- startup script failures hidden after deployment

### Fix
Confirm:
- Functions runtime is v4
- PowerShell runtime is 7.4
- app settings match expectations
- storage account is reachable
- deployment included the full repo, not a partial subfolder

Because this repo deploys from root, deploying only selected folders can break it.

---

## 3. HTTP API returns 500

### Cause
Possible causes:
- `New-CippCoreRequest` failed internally
- request route is invalid
- downstream cmdlet threw an exception
- authentication/tenant context was not loaded
- config table lookup failed

### Fix
- inspect Azure Function logs
- inspect Application Insights if configured
- test a known-good endpoint first
- test whether the same route works locally
- verify backing Azure storage tables/config exist

Also verify that the route is reaching the correct `CIPPEndpoint` value.

---

## 4. `$batch` requests fail

### Cause
Possible causes:
- malformed batch body
- missing `requests` array
- too many requests in a single batch
- one or more inner requests failed during parallel execution

### Fix
Validate the request body shape. The code expects:
- a body containing `requests`
- no more than 20 requests

Reduce the batch size if needed.

If only some items fail, inspect the returned per-item batch response because the code returns Graph-style individual statuses.

---

## 5. Queue-triggered work does not execute

### Cause
Possible causes:
- `cippqueue` storage queue not available
- storage connection invalid
- queue item shape is wrong
- referenced cmdlet does not exist in `CIPPCore`

### Fix
Verify:
- queue storage is working
- `AzureWebJobsStorage` is valid
- queue item contains:
  - `Cmdlet`
  - `Parameters`
- the cmdlet exists and is imported

The queue handler explicitly checks command existence and skips unknown commands.

---

## 6. Durable orchestrations do not complete

### Cause
Possible causes:
- invalid orchestration input
- no batch items generated
- activity function failure
- retry settings exhausted
- Premium/Non-Premium scaling assumptions differ

### Fix
Inspect:
- orchestration input payload
- whether `Batch` exists
- whether `QueueFunction` returned any items
- whether activity functions are throwing errors
- whether the chosen durable mode is appropriate

For debugging, start with smaller sequence-based workflows before large fan-out runs.

---

## 7. Timer jobs cause noise or side effects in local development

### Cause
Timer triggers run every 15 minutes by default.

### Fix
Use the included local flags from `.env.example` to disable certain timer behaviors during development, such as:
- `DEV_SKIP_BPA_TIMER`
- `DEV_SKIP_DOMAIN_TIMER`

Also ensure your local env file is actually being loaded by Compose.

---

## 8. Local Docker environment starts but requests fail

### Cause
Possible causes:
- Azurite not healthy
- nginx proxying issue
- compose env vars missing
- container image built but function app not initialized correctly

### Fix
Check containers in order:
1. `azurite`
2. `cippapi`
3. `nginx`

Then verify:
- storage connection string points to Azurite
- nginx can reach `cippapi`
- port `7071` is exposed correctly
- `FUNCTIONS_WORKER_RUNTIME` and version variables are set

---

## 9. Local storage/bindings fail

### Cause
The function app requires valid Azure Functions storage wiring even in local mode.

### Fix
Use the provided Azurite connection string pattern from `.env.example`.

If running outside compose, update the storage connection string so it points to a reachable Azurite instance or real Azure storage.

---

## 10. Cold starts or slow startup

### Cause
This app loads several modules, optional telemetry assemblies, auth, and version/config state at startup.

### Fix
- expect heavier cold starts than a trivial function app
- keep Application Insights enabled for visibility
- use Premium hosting if cold-start impact matters operationally
- verify startup logs for slow imports or auth delays
- ensure storage latency is acceptable

---

## 11. Function app rejects API calls on certain instances

### Cause
The code supports an offload/processor model and may reject API calls if a config flag is set and the instance is marked as processor-only.

### Fix
Check:
- `OffloadFunctions` config entry
- environment variable `CIPP_PROCESSOR`

If an app instance is intended only for background processing, direct API calls may correctly return forbidden.

---

## 12. Module import failures in Azure

### Cause
Possible causes:
- module files missing in deployment
- bad packaging/deployment artifact
- encoding/file corruption
- path assumptions broken

### Fix
- verify the deployed `Modules/` tree exists in the function app content
- redeploy from repo root
- confirm `profile.ps1` is present
- avoid trimming “unused” files from deployment package

This repository is not safe to deploy as a minimal subset unless you understand module dependencies.

---

## 13. Application Insights shows no telemetry

### Cause
Telemetry is only initialized if:
- `APPLICATIONINSIGHTS_CONNECTION_STRING`
- or `APPINSIGHTS_INSTRUMENTATIONKEY`

is set.

### Fix
Add one of those settings to the Azure Function App configuration and restart the app.

Also verify the DLL under:
```text
Shared/AppInsights/Microsoft.ApplicationInsights.dll
```
is present in the deployed artifact.

---

## 14. Release or update workflows fail

### Cause
Possible causes:
- `version_latest.txt` missing
- invalid Azure secrets
- publish profile mismatch
- blob storage secrets missing
- workflow branch expectations not met

### Fix
Check:
- repository secrets
- branch name conventions
- release tagging logic
- Azure Blob connection string
- whether the workflow expects `master`, `dev`, or `pre-release`

---

## 15. API appears exposed because `authLevel` is anonymous

### Cause
The HTTP trigger is explicitly configured as anonymous.

### Fix
Do **not** assume Azure Functions host auth is protecting the app.

You should verify and, if needed, enforce protection through:
- platform/network restrictions
- upstream auth gateway
- application-layer authentication checks
- private endpoints / IP restrictions / front door / reverse proxy

This is an important operational security check.

---

## Azure-specific troubleshooting guide

Because this app is clearly Azure-hosted, here is the Azure-focused rectification section.

## Azure App Settings checklist
When the app misbehaves in Azure, verify:

- `AzureWebJobsStorage`
- `FUNCTIONS_WORKER_RUNTIME`
- `FUNCTIONS_EXTENSION_VERSION`
- any CIPP-specific auth/secret settings
- `APPLICATIONINSIGHTS_CONNECTION_STRING` or `APPINSIGHTS_INSTRUMENTATIONKEY`
- any environment flags used by your deployment mode

## Azure Function runtime checks
Confirm the Function App is:
- on the correct Functions runtime version
- using the intended PowerShell worker version
- not missing storage configuration
- not running a stale deployment

## Azure Storage checks
This app depends heavily on storage-backed bindings and state. Verify:
- queue access works
- table access works
- blob access works where expected
- firewall/private networking is not blocking storage

## Azure deployment checks
From the workflows, deployment is done from repository root. If you deploy differently:
- make sure all function folders are included
- make sure `Modules/` is included
- make sure `profile.ps1`, `host.json`, `Config/`, and `Shared/` content are included

## Azure scaling checks
Because the app uses Durable Functions and queue processing:
- check whether scale-out changes behavior
- test fan-out orchestration carefully
- verify Premium-vs-non-Premium expectations where orchestration mode changes

## Azure monitoring checks
Prefer enabling Application Insights and inspecting:
- startup failures
- dependency failures
- long-running requests
- failed orchestrations
- queue trigger failures

---

## Outputs and API surface

The API surface is large, and `openapi.json` plus the OpenAPI files under `Tools/` are the best evidence of breadth.

Examples shown in the OpenAPI include endpoints such as:
- `/AddGroup`
- `/AddChocoApp`
- `/AddWin32ScriptApp`
- `/RemoveUser`

This suggests the API covers a wide administrative surface rather than one narrow feature.

Because the HTTP route is catch-all, logical endpoints are implemented inside request-routing logic rather than as one Azure Function folder per endpoint.

---

## Testing

The repo includes a `Tests/` folder with PowerShell tests for areas such as:
- Alerts
- Endpoint
- Standards

This indicates automated testing is already part of the project structure and should be extended when adding new features.

---

## Security and operational considerations

## Important: anonymous HTTP trigger
The biggest security design note visible from the repo is the anonymous HTTP trigger.

That does **not** necessarily mean the app is unsecured, but it does mean you must confirm where authentication is really enforced.

## Large operational surface area
This is a large admin backend with many capabilities. A misconfiguration can affect:
- background tasks
- tenant administration
- standards enforcement
- queue execution

## Startup complexity
This app has a non-trivial startup path. Module import failures or missing secrets can break the whole app before requests are handled.

## Durable/queue complexity
This app supports asynchronous and fan-out operations. Debugging requires checking:
- HTTP request path
- queue contents
- orchestration status
- activity execution
- storage health

---

## Known issues and risks visible from the uploaded repo

Based on the uploaded code structure, notable operational risks include:

1. the HTTP trigger is configured as `anonymous`, so auth boundaries must be verified carefully
2. the app is heavily dependent on Azure storage and startup initialization
3. module deployment completeness is critical
4. local/runtime behavior can vary between Premium and non-Premium modes
5. batch mode can create concurrency and troubleshooting complexity
6. Docker/local development uses multiple components, so local debugging is not trivial
7. the API surface is large, which increases regression risk when making changes
8. release/deployment automation is branch- and secret-dependent

---

## Recommended improvements

## High priority
- document the real authentication boundary explicitly
- document required Azure app settings in one canonical file
- document minimum Azure resources required for deployment
- add a root README if one is not maintained elsewhere
- add a local developer quickstart guide
- add endpoint-routing documentation tied to OpenAPI

## Medium priority
- add clearer module dependency mapping
- add a troubleshooting runbook for queue/orchestration failures
- add health-check endpoint guidance
- add deployment validation checklist
- add environment config validation on startup

## Nice to have
- generate a cleaner developer architecture document from trigger -> module -> cmdlet flow
- document how `New-CippCoreRequest` maps endpoints to internal functions
- provide example local env files for common scenarios
- add a supported-hosting matrix

---

## Suggested way to amend/add functionality safely

For this specific repository, the safest extension pattern is:

1. **Add logic in modules, not trigger folders**, unless you are adding a genuinely new trigger type
2. **Keep HTTP routing centralized**
3. **Write tests in `Tests/`**
4. **Update OpenAPI docs**
5. **Validate local startup**
6. **Validate queue/orchestration side effects**
7. **Deploy to dev Azure Function App first**
8. **Verify logs and telemetry before production rollout**

---

## Final assessment

This project is a **substantial Azure Functions backend** for the CIPP platform.

It is best described as:
- an **Azure-hosted PowerShell Function App**
- with **HTTP, queue, timer, and durable orchestration**
- plus **bundled modules, templates, and deployment automation**
- and **optional Docker-based local development**

It is **definitely Azure-oriented**, not just a generic server-run script.

Its strengths are:
- mature Azure Functions structure
- modular PowerShell organization
- durable and queue-based scaling support
- local development support
- release/deployment automation
- large operational feature surface

Its main operational complexities are:
- startup dependency chain
- Azure storage reliance
- anonymous trigger configuration
- large routing surface
- orchestration/debugging complexity

For documentation and maintenance purposes, this should be treated as:
- a **platform backend**
- not a small standalone utility
- and one that benefits greatly from clear Azure configuration and operational runbooks.
