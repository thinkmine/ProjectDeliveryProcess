# Project Delivery Process

A comprehensive, end-to-end guide to our project delivery process. This repository documents the methodologies, standards, workflows, and best practices that support consistent, repeatable, and high-quality delivery across all phases of a project—from initial discovery and requirements analysis to development, testing, release management, and post-deployment operations. Designed as a living resource, it provides templates, checklists, governance models, and reference architectures to ensure teams can plan, execute, and deliver with clarity and alignment.



Below is a **fully fleshed-out, training-ready use case** with **design diagrams, architecture explanation, step-by-step implementation, and hands-on exercises**.
You can deliver this as a workshop, onboarding module, or internal training curriculum for architects, engineers, and DevOps teams.

---

# 🌐 **Training Use Case: End-to-End Project Delivery – Dynamic UI + APIM + Azure Functions + SQL + NoSQL + GitOps**

## 🎯 **Business Scenario**

Your organization wants to standardize how projects are delivered.
You will build a **reference implementation** that demonstrates a complete delivery lifecycle:

1. **Dynamic Bootstrap UI** – configuration-driven dynamic pages, forms, and tables.
2. **Azure API Management** – API gateway, security, policies.
3. **Azure Function App** – CRUD API for "Projects".
4. **Azure SQL + Cosmos DB** – dual-write data ingestion pipeline.
5. **GitOps CI/CD** – automated deployments of infrastructure + code.
6. **Observability & Operations** – logging, telemetry, error handling.

Teams trained on this use case can replicate this delivery pattern for any client.

---

# 📘 **1. Functional Use Case**

### **“Project Tracking Platform”**

This reference solution tracks projects delivered by the company:

* Create / update / delete project records
* View project list
* Search/filter by status
* Store records in Azure SQL
* Store denormalized metadata in Cosmos DB
* Provide ingestion endpoint for batch uploads
* Expose APIs through APIM
* Dynamically render UI pages based on JSON configuration

This models real enterprise patterns for governance, CRUD flows, data ingestion, secure API delivery, and GitOps.

---

# 🏛️ **2. Architecture Overview**

### **High-Level Architecture**

```
[Bootstrap UI] 
     ↓ REST
[API Management]
     ↓ Backend Route
[Azure Functions CRUD API]
     ↓ Writes
[Azure SQL]  <--->  [Cosmos DB NoSQL]
     ↑                   ↓
[Ingestion Function] <--- Batch/Events
```

### **GitOps Workflow**

```
Git Commit → PR → Pipeline Validate → Deploy Dev → Approve → Deploy Prod
```

---

# 🧩 **3. Design Components**

## **3.1 UI Requirements**

* HTML5 Bootstrap dynamic UI
* Driven entirely by JSON config files
* Pages load dynamically
* Tables auto-populate from `/api/projects`
* Forms submit to APIM endpoints
* No hard-coded HTML forms

---

## **3.2 API Requirements**

Endpoints:

| Method | Route          | Purpose                      |
| ------ | -------------- | ---------------------------- |
| GET    | /projects      | Get all projects             |
| GET    | /projects/{id} | Get single project           |
| POST   | /projects      | Create project               |
| PUT    | /projects/{id} | Update project               |
| DELETE | /projects/{id} | Delete project               |
| POST   | /ingest        | Ingest batch project records |

---

## **3.3 Data Requirements**

### SQL Table

```
ProjectId (PK)
Name
Owner
Status
StartDate
EndDate
LastUpdatedUtc
```

### Cosmos Container

```
id
name
status
metadata: {...}
lastSyncUtc
```

---

# 🛠️ **4. Step-by-Step Implementation Guide (Training)**

---

# 🎨 **STEP 1: Build Dynamic Bootstrap UI**

### 1.1 Create Folder Structure

```
ui/
  index.html
  assets/
    js/
      ui-config.json
      app.js
    css/
      styles.css
```

### 1.2 Create the Bootstrap Shell (`index.html`)

Includes:

* Nav bar
* Dynamic content container
* App.js script

### 1.3 Build the JSON-driven UI (`ui-config.json`)

Example:

```json
{
  "navigation": [
    { "id": "home", "label": "Home" },
    { "id": "projects", "label": "Projects" }
  ],
  "pages": {
    "projects": {
      "title": "Project Records",
      "form": {
        "endpoint": "/api/projects",
        "fields": [
          {"name": "projectId", "label": "Project ID","type": "text"},
          {"name": "name","label": "Name","type": "text"},
          {"name": "owner","label": "Owner","type": "text"},
          {"name": "status","label": "Status","type": "select","options":["Active","Closed"]}
        ]
      },
      "table": {
        "endpoint": "/api/projects",
        "columns": ["projectId","name","owner","status"]
      }
    }
  }
}
```

### 1.4 Create app.js

Loads UI config → builds pages dynamically → fetches table data → POST/PUT operations.

---

# 🏗️ **STEP 2: Deploy Azure Infrastructure (IaC – GitOps)**

All infra is deployed with **Bicep** or **Terraform**:

### Resources:

* Azure SQL Database + Server
* Cosmos DB
* Function App
* API Management
* Storage Account (for UI hosting)
* App Insights
* Key Vault

### GitOps Flow:

1. `infra/bicep/main.bicep`
2. `infra/bicep/parameters/dev.json`
3. GitHub Actions pipeline triggers on PR
4. `az deployment group create ...`

This step teaches:

* Parameterization
* Environment segregation
* What-if validation
* Role-based secret access

---

# 🧪 **STEP 3: Implement CRUD API (Azure Functions)**

### 3.1 Create Function App Project

Files:

```
functions/
  GetProjects.cs
  GetProjectById.cs
  CreateProject.cs
  UpdateProject.cs
  DeleteProject.cs
  IngestProjects.cs
  Services/
    SqlService.cs
    CosmosService.cs
  Models/
    Project.cs
```

### 3.2 SQL Logic (Insert/Update)

```csharp
var query = @"INSERT INTO Project (...) VALUES (...)";
```

### 3.3 Cosmos Logic (Upsert)

```csharp
await container.UpsertItemAsync(item);
```

### 3.4 Batch Ingestion Endpoint

Accepts JSON array of projects → loops → dual-write to SQL + Cosmos.

---

# 🌉 **STEP 4: API Management Integration**

### 4.1 Create API in APIM

* Name: `project-api`
* Version: v1

### 4.2 Import Function App

Through:

* OpenAPI URL
  or
* Direct function import

### 4.3 Apply Policies

* Rate-limit
* CORS
* Validate JWT (for training: allow anonymous or use APIM subscription key)
* Add correlation ID

Example inbound policy:

```xml
<set-header name="x-correlation-id" exists-action="override">
  <guid />
</set-header>
```

### 4.4 Test all operations in APIM Console.

---

# 🚀 **STEP 5: CI/CD GitOps Deployment**

### Pipelines Cover:

### 5.1 Infrastructure Pipeline

* Validate Bicep
* Deploy to dev
* Gate approval
* Deploy to prod

### 5.2 Function App Pipeline

* Build, test, publish
* Zip deploy to Function App
* Swap slots (canary pattern)

### 5.3 UI Pipeline

* Build & minify UI
* Deploy to Static Web App or Blob Static Website
* Add environment banners

### 5.4 Automated Testing Stage

* Post-deployment smoke tests
* API functional tests
* UI availability test using Playwright

---

# 🔄 **STEP 6: Data Ingestion Pipeline**

### Trigger:

* API call (POST /ingest)
* Optionally Event Grid → queue → ingestion function

### Ingestion Steps:

1. Validate batch
2. Loop through record list
3. Write to SQL
4. Upsert to Cosmos
5. Log each step (App Insights)
6. Return ingestion summary

### Example Response:

```json
{
  "received": 20,
  "processed": 20,
  "failed": 0,
  "durationMs": 243
}
```

---

# 🔍 **STEP 7: Monitoring & Observability**

### Metrics Instrumentation:

* App Insights (Requests, Dependencies, Failures)
* Ingestion-specific metrics
* Dashboard for:

  * API performance
  * SQL DTU/CPU
  * Cosmos RU throttling
  * APIM calls

### Alerts:

* Function error > 5%
* API latency > 1s
* SQL CPU > 80%
* Cosmos RU > 90%

---

# 🎓 **Training Modules Breakdown**

| Module                         | Description                                | Hands-on Lab                 |
| ------------------------------ | ------------------------------------------ | ---------------------------- |
| **1. Architecture & Design**   | Learn patterns used in enterprise delivery | diagram workshop             |
| **2. Dynamic UI**              | Build config-driven Bootstrap UI           | implement `ui-config.json`   |
| **3. Azure Functions CRUD**    | Build API backend                          | write Create/Update/Delete   |
| **4. SQL + Cosmos Dual-write** | Implement ingestion logic                  | ingest batch of sample data  |
| **5. APIM Front Door**         | Secure and expose functions                | import functions into APIM   |
| **6. GitOps CI/CD**            | Set up GitHub Actions/Azure DevOps         | deploy dev + prod            |
| **7. Monitoring**              | Observe dependencies & failures            | build App Insights dashboard |

---

# 📦 **Deliverables for Students**

You can bundle these into the repo:

* Full source code for UI
* Function App CRUD templates
* SQL schema + stored procedures
* Cosmos container templates
* Bicep/Terraform infra templates
* CI/CD YAML samples
* APIM policy templates
* Step-by-step labs


---

## 1. Repository & Project Structure

**Goal:** One repo that holds **IaC, backend, frontend, and docs** in a way that supports GitOps and repeatable delivery.

**1.1. Root layout**

```text
project-delivery-guide/
├─ docs/
│  ├─ 01-introduction.md
│  ├─ 02-architecture-overview.md
│  ├─ 03-standards-and-conventions.md
│  ├─ 04-ci-cd-gitops.md
│  ├─ 05-operations-and-support.md
│  └─ checklists/
├─ infra/
│  ├─ bicep/          # or terraform
│  ├─ pipelines/      # pipeline YAML templates
│  └─ env/            # per-environment config (dev/test/prod)
├─ src/
│  ├─ ui/             # Bootstrap dynamic UI
│  ├─ functions/      # Azure Functions app (CRUD + ingestion)
│  └─ shared/         # shared models, DTOs, config
├─ db/
│  ├─ sql/
│  │  ├─ schema.sql
│  │  ├─ seed-data.sql
│  │  └─ stored-procs.sql
│  └─ nosql/
│     └─ containers.json   # Cosmos DB container definitions & sample docs
├─ .github/ or azure-pipelines/
│  └─ workflows/           # GitHub Actions or Azure DevOps pipelines
└─ README.md
```

**1.2. Governance & standards (docs folder)**

Add docs that describe:

* **Branching model:** `main`, `develop`, `feature/*`, `hotfix/*`.
* **Environment mapping:** `develop → dev`, `main → prod`.
* **Pull request rules:** required reviews, checks, test coverage gates.
* **Definition of Done:** code, tests, docs, pipeline passing, monitoring in place.

---

## 2. Dynamic Bootstrap HTML UI

**Goal:** A simple **config-driven UI** built with Bootstrap, rendered dynamically from JSON so it’s easy to reuse and extend.

### 2.1. UI structure

Under `src/ui/`:

```text
src/ui/
├─ index.html
├─ assets/
│  ├─ css/
│  │  └─ styles.css
│  └─ js/
│     ├─ ui-config.json
│     └─ app.js
```

### 2.2. `index.html` – Bootstrap shell

Key elements:

* Bootstrap CSS + JS references (CDN is fine).
* A **top nav bar** with links (Home, Data Explorer, Admin).
* A **content container** that the JS will populate based on `ui-config.json`.

Skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Project Delivery Demo UI</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
  <link rel="stylesheet" href="assets/css/styles.css">
</head>
<body>
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Delivery Demo</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
      <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav ms-auto" id="nav-links"></ul>
    </div>
  </div>
</nav>

<main class="container my-4">
  <div id="dynamic-content"></div>
</main>

<script src="assets/js/app.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 2.3. `ui-config.json` – config-driven pages

Example config that defines **pages**, **forms**, and **tables**:

```json
{
  "navigation": [
    { "id": "home", "label": "Home", "type": "page" },
    { "id": "records", "label": "Records", "type": "page" },
    { "id": "ingestion", "label": "Ingestion Monitor", "type": "page" }
  ],
  "pages": {
    "home": {
      "title": "Project Delivery Demo",
      "content": "<p>Welcome to the demo UI showcasing our standard delivery pattern.</p>"
    },
    "records": {
      "title": "Manage Records",
      "form": {
        "id": "recordForm",
        "submitLabel": "Save Record",
        "endpoint": "/api/records",
        "method": "POST",
        "fields": [
          { "name": "id", "label": "ID", "type": "text", "required": true },
          { "name": "name", "label": "Name", "type": "text", "required": true },
          { "name": "category", "label": "Category", "type": "text" },
          { "name": "status", "label": "Status", "type": "select",
            "options": ["Active", "Inactive"] }
        ]
      },
      "table": {
        "id": "recordTable",
        "endpoint": "/api/records",
        "columns": [
          { "field": "id", "label": "ID" },
          { "field": "name", "label": "Name" },
          { "field": "category", "label": "Category" },
          { "field": "status", "label": "Status" }
        ]
      }
    },
    "ingestion": {
      "title": "Ingestion Monitor",
      "content": "<p>Shows recent ingestion runs from SQL and NoSQL pipelines.</p>"
    }
  }
}
```

### 2.4. `app.js` – dynamic rendering

High-level steps:

1. **Fetch** `ui-config.json`.
2. Build nav items from `navigation`.
3. Render selected page:

   * Inject `title` and `content`.
   * If `form` exists, generate Bootstrap form fields.
   * If `table` exists, call backend `GET /api/records` via APIM and render table rows.
4. Handle **form submission** with `fetch()` to APIM endpoints.

This gives you a generic pattern you can reuse in other projects.

---

## 3. Data Model: Azure SQL & NoSQL

**Goal:** Simple, demo-friendly entities to show **dual-write** to SQL and NoSQL.

### 3.1. Azure SQL schema (in `db/sql/schema.sql`)

Example `Record` table:

```sql
CREATE TABLE [dbo].[Record] (
    [Id]          NVARCHAR(50) NOT NULL PRIMARY KEY,
    [Name]        NVARCHAR(200) NOT NULL,
    [Category]    NVARCHAR(100) NULL,
    [Status]      NVARCHAR(50) NOT NULL,
    [CreatedUtc]  DATETIME2 NOT NULL DEFAULT SYSDATETIMEOFFSET(),
    [UpdatedUtc]  DATETIME2 NULL
);
```

### 3.2. Sample seed data (in `db/sql/seed-data.sql`)

```sql
INSERT INTO [dbo].[Record] (Id, Name, Category, Status)
VALUES
('rec-001', 'Sample Record 1', 'Demo', 'Active'),
('rec-002', 'Sample Record 2', 'Demo', 'Inactive');
```

### 3.3. NoSQL (Cosmos DB) container

In `db/nosql/containers.json`:

```json
{
  "databaseId": "delivery-demo-db",
  "containers": [
    {
      "id": "records",
      "partitionKey": "/id",
      "throughput": 400,
      "sampleDocuments": [
        {
          "id": "rec-001",
          "name": "Sample Record 1",
          "category": "Demo",
          "status": "Active",
          "source": "SQL-sync",
          "lastUpdatedUtc": "2025-01-01T00:00:00Z"
        }
      ]
    }
  ]
}
```

---

## 4. Azure Function App – CRUD + Ingestion

**Goal:** HTTP-triggered **Function App** that:

* Exposes `/api/records` CRUD endpoints.
* Writes/reads from **Azure SQL** and **Cosmos DB**.
* Handles **ingestion events** (e.g., from Service Bus/Event Grid).

### 4.1. Functions project structure

```text
src/functions/
├─ RecordApi/
│  ├─ GetRecords.cs       # GET /api/records
│  ├─ GetRecordById.cs    # GET /api/records/{id}
│  ├─ CreateRecord.cs     # POST /api/records
│  ├─ UpdateRecord.cs     # PUT /api/records/{id}
│  └─ DeleteRecord.cs     # DELETE /api/records/{id}
├─ IngestionFunctions/
│  └─ IngestRecord.cs     # e.g., ServiceBus trigger or HTTP trigger
├─ Models/
│  └─ Record.cs
└─ shared config
```

(Use C#, Node.js, or Python—whatever your team standard is; pattern is the same.)

### 4.2. CRUD function behavior

For **Create / Update**:

1. **Validate** request payload (id, name, status).
2. **Write to Azure SQL** (INSERT or UPDATE).
3. **Upsert to Cosmos** to keep a denormalized view.
4. Return 201/200 with record.

For **Delete**:

1. Soft delete in SQL (optional) or hard delete.
2. Delete doc in Cosmos.

For **Get**:

* `GET /api/records` – query SQL (and optionally join with Cosmos metadata).
* `GET /api/records/{id}` – read from SQL or Cosmos.

### 4.3. Ingestion function

A simple ingestion path:

* Trigger: `HttpTrigger` or `ServiceBusTrigger`.
* Input: new records batch.
* Logic:

  * Parse batch.
  * For each record, execute same dual-write logic as create (or reuse shared service).
  * Log ingestion metrics (count, latency).

---

## 5. Azure API Management (APIM)

**Goal:** Front-door for all APIs (records CRUD + ingestion), with consistent **policies, security, and observability**.

### 5.1. APIM resources via IaC

In `infra/bicep/` or `infra/terraform/`:

* Define:

  * `Microsoft.ApiManagement/service`
  * `api` resource (import from Function App OpenAPI or direct HTTP backend)
  * `products` (e.g., `internal`, `external`)
  * global policies (CORS, rate limit, logging)

### 5.2. APIM API design

Expose endpoints like:

* `GET /records`
* `GET /records/{id}`
* `POST /records`
* `PUT /records/{id}`
* `DELETE /records/{id}`
* `POST /ingestion/run`

Map them to backend Functions:

* Backend URL: `https://{functionapp}.azurewebsites.net/api/records`.
* Use **function key** or **managed identity** for backend auth.
* Add APIM policy to inject required headers.

### 5.3. Policies

At API or operation level:

* **Inbound:**

  * Validate JWT (if using Entra ID / B2C).
  * Check subscription key (`Ocp-Apim-Subscription-Key`).
  * CORS for UI origin.
* **Outbound:**

  * Add correlation IDs.
  * Strip backend headers.
* **On error:**

  * Normalize error response format.

---

## 6. CI/CD & GitOps

**Goal:** All infra and app deployments are **driven by Git changes**. Environments managed via **branches / folders**, not manual changes.

### 6.1. Infra GitOps (Bicep/Terraform)

Pattern:

1. **Infra code** in `infra/bicep/` with parameter files per env:

   ```text
   infra/bicep/
   ├─ main.bicep
   └─ parameters/
      ├─ dev.json
      ├─ test.json
      └─ prod.json
   ```

2. Pipeline (GitHub Actions or Azure DevOps) stages:

   * `validate` (lint, `what-if`).
   * `deploy-dev` on PR merge to `develop`.
   * `deploy-prod` on tagged release or `main` merge.

3. Use **Managed Identity** or **OIDC federated credential** for the pipeline to deploy infra.

### 6.2. App GitOps (Functions & UI)

Set up two pipelines (or two jobs):

1. **UI pipeline**:

   * Trigger: changes in `src/ui/**`.
   * Steps:

     * Build static assets (optional bundler).
     * Publish to:

       * Azure Static Web App **or**
       * Azure Storage static website **or**
       * App Service.
   * Use environment-specific config (API base URL, environment banner).

2. **Function App pipeline**:

   * Trigger: changes in `src/functions/**`.
   * Steps:

     * Restore → Build → Test.
     * Deploy:

       * Zip deploy to Function App.
       * Or container build & push to ACR, then update Function App to use new image.
   * Use app settings from Key Vault (connection strings, Cosmos endpoint, etc.).

### 6.3. GitOps workflow

* **Dev:**

  * Dev merges feature → `develop`.
  * Pipelines:

    * Deploy infra to `dev` subscription/resource group.
    * Deploy Function App & UI.
    * Smoke tests run.
* **Prod:**

  * Release branch or tag → `main`.
  * Same process for `prod` environment, gated by approvals.

Document this flow in `docs/04-ci-cd-gitops.md` with sequence steps.

---

## 7. Data Ingestion Solution (SQL + NoSQL)

**Goal:** Show how data flows **from UI / external systems → API → ingestion → SQL + NoSQL**.

### 7.1. Simple ingestion flow

1. **UI** submits new records via `POST /records` or `POST /ingestion/run`.
2. **APIM** validates and routes request to Function App.
3. **Ingestion Function**:

   * For each item:

     * Writes to **Azure SQL** `Record` table.
     * Upserts to **Cosmos DB** `records` container.
4. **Monitoring**:

   * Log events in App Insights (custom events: `IngestionRunStarted`, `IngestionRunCompleted`).
   * Expose summary via `GET /ingestion/status`.

### 7.2. Sample ingestion scenario

Add `docs/ingestion-scenario.md`:

* Input: CSV or JSON payload with sample records.
* Step-by-step:

  1. Upload CSV to UI or call ingestion endpoint via Postman.
  2. Pipeline writes records into both data stores.
  3. UI “Records” page shows combined view (e.g., status from SQL and additional flags from Cosmos).

---

## 8. Testing, Quality, and Operations

**Goal:** Bake **quality & ops** into the process, not as an afterthought.

### 8.1. Testing

* **Unit tests**:

  * For Function App business logic, repositories (SQL + Cosmos).
* **Integration tests**:

  * Spin up test database(s) or use test containers.
  * Hit APIM endpoints in a dev environment.
* **UI tests**:

  * Basic smoke tests (e.g., Playwright / Cypress) to ensure pages render and APIs respond.

### 8.2. Observability

* Enable **Application Insights**:

  * Track requests, dependencies (SQL/Cosmos), failures.
  * Add custom metrics for ingestion (records processed).
* Capture **logs** for:

  * Function App (structured logs).
  * APIM (diagnostic logs).

### 8.3. Runbooks & SRE

In `docs/05-operations-and-support.md`:

* How to **rollback** a bad deployment (revert commit/tag).
* How to **rotate keys** (APIM subscription key, SQL passwords if used, etc.).
* How to **scale** Function Apps and APIM.
* On-call / incident steps.

---

## 9. How to Present This in the Repo

