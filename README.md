# Project Delivery Process

A comprehensive, end-to-end guide to our project delivery process. This repository documents the methodologies, standards, workflows, and best practices that support consistent, repeatable, and high-quality delivery across all phases of a project—from initial discovery and requirements analysis to development, testing, release management, and post-deployment operations. Designed as a living resource, it provides templates, checklists, governance models, and reference architectures to ensure teams can plan, execute, and deliver with clarity and alignment.


# Project Delivery Guide – GitHub to Azure (End-to-End)

This guide explains **how we deliver solutions using GitHub into Azure**, from the first conversation with a stakeholder to post-deployment operations. It’s designed as a **repeatable blueprint** for all projects.

---

## 0. Core Principles

Before we dive into steps, these principles apply everywhere:

1. **Everything-as-Code**

   * Application code
   * Infrastructure (Bicep/Terraform/ARM)
   * CI/CD workflows (GitHub Actions YAML)
   * Configuration (JSON/YAML settings)
2. **GitHub as the Source of Truth**

   * No manual changes in Azure that aren’t also captured in Git.
3. **Environment Parity**

   * dev → test → prod follow the same pattern, just with different scale/config.
4. **Security & Governance First**

   * Least privilege, OIDC to Azure, secrets in GitHub Encrypted Secrets / Key Vault.
5. **Automated Quality Gates**

   * PR checks, tests, security scans, policy checks.

---

## 1. Repository Foundations

### 1.1. Create / Structure the Repo

**Step 1 – Create the GitHub repo**

* Name: `org-solution-name` (e.g., `contoso-project-tracker`).
* Make it private by default.
* Initialize with a README.

**Step 2 – Create base folders**

```text
/.
├─ docs/           # Methodologies, standards, runbooks
├─ src/            # Application code (API, UI, Functions, etc.)
├─ infra/          # Azure IaC (Bicep/Terraform)
├─ .github/
│  └─ workflows/   # GitHub Actions
└─ .devcontainer/  # Optional: GitHub Codespaces dev config
```

**Step 3 – Branching strategy**

* `main` – production-ready, tagged releases.
* `develop` – integration branch for upcoming release (optional, if you prefer trunk you can skip).
* `feature/*` – short-lived branches for work items.
* `hotfix/*` – production fixes.

Configure **branch protection** on `main` (and `develop` if used):

* Require PRs
* Require status checks to pass (build, tests, lint)
* Require at least 1–2 reviewers
* No direct pushes

---

## 2. Discovery & Requirements in GitHub

### 2.1. Capture Work with Issues

**Step 1 – Define issue templates**

In `.github/ISSUE_TEMPLATE/` create templates like:

* `feature_request.md`
* `bug_report.md`
* `architecture_change.md`

Each template includes:

* Problem statement
* Business value
* Acceptance criteria
* Dependencies

**Step 2 – Labels**

Define labels such as:

* `phase:discovery`, `phase:design`, `phase:build`, `phase:test`, `phase:release`, `phase:ops`
* `type:feature`, `type:bug`, `type:tech-debt`
* `priority:high|medium|low`

**Step 3 – Project board**

Use a **GitHub Projects** board (e.g., “Delivery – Project Tracker”) with columns:

* Backlog
* Discovery
* Design
* In Development
* In Testing
* Ready for Release
* Done

Link issues to this board for traceability.

---

## 3. Architecture & Design

### 3.1. Document the Architecture in the Repo

**Step 1 – Create architecture docs**

In `docs/`:

* `02-architecture-overview.md`
* `03-reference-architectures.md`
* `adr/` for **Architecture Decision Records**

Each includes:

* Overall diagram (UI → APIM → Functions / App Service → Azure SQL / NoSQL)
* Non-functional requirements (availability, security, performance)
* Chosen Azure services and why

**Step 2 – Architecture Decision Records (ADRs)**

For significant decisions (e.g., “Use Azure Functions vs App Service”), create ADRs:

* Context
* Decision
* Alternatives
* Consequences

This gives future teams a paper trail of why things are how they are.

### 3.2. Define Azure Environments

Typical mapping:

* `dev` – for feature testing
* `test` / `qa` – for integration/UAT
* `prod` – live users

Define:

* Azure subscriptions or resource groups per environment
* Naming conventions, e.g.:

  * `rg-solution-dev`, `rg-solution-test`, `rg-solution-prod`
  * `func-solution-dev`, `sql-solution-dev`, etc.

Document this in `docs/04-environments-and-naming.md`.

---

## 4. Implementation – Code & Infrastructure

### 4.1. Infrastructure as Code (IaC)

**Step 1 – Define infrastructure in `infra/`**

Example with Bicep:

```text
infra/
├─ main.bicep
└─ parameters/
   ├─ dev.json
   ├─ test.json
   └─ prod.json
```

`main.bicep` provisions:

* Resource group (if not already)
* Storage account
* Azure Function App or Web App
* Azure SQL / Cosmos DB
* API Management
* Application Insights
* Key Vault (for secrets/config)

**Step 2 – Parameterize by environment**

Each `*.json` file sets:

* Resource names (with environment suffix)
* SKU tiers (Basic in dev, Premium in prod)
* Connection strings names

**Step 3 – Validate locally**

Use Azure CLI / Bicep:

```bash
az deployment group what-if ...
```

### 4.2. Application Code (`src/`)

Typical structure:

```text
src/
├─ api/              # Functions or Web API
├─ ui/               # HTML / SPA / Bootstrap UI
└─ shared/           # shared models, DTOs, utilities
```

**Guidelines:**

* Keep a **clear separation of concerns** (controllers/functions vs services vs data access).
* Make configuration external (app settings, Key Vault, environment variables).
* Add unit tests in `tests/` and wire them to CI.

---

## 5. Testing & Quality – GitHub Actions

### 5.1. Define Quality Gates

For each PR into `develop` / `main`:

* Build passes
* Unit tests pass
* (Optional) Integration tests
* Linting / static analysis
* Security scanning (CodeQL, Dependabot alerts)

### 5.2. Create CI Workflow

In `.github/workflows/ci.yml`:

**High-level steps:**

1. Trigger on `pull_request` and `push` to `develop` / `main`.
2. Check out code.
3. Set up language runtime (.NET, Node, Python, etc.).
4. Restore dependencies.
5. Build solution.
6. Run tests.
7. (Optional) Run linters, static analysis, CodeQL.

This workflow becomes a **required check** before merge.

---

## 6. Release Management – GitHub to Azure

### 6.1. Connect GitHub to Azure (Securely)

**Step 1 – Use OIDC (recommended)**
Set up a **federated credential** from GitHub to Azure:

* In Azure: create a **user-assigned managed identity** or a Service Principal.
* In GitHub: configure **OpenID Connect** using a GitHub environment (e.g., `dev`, `prod`) or repo-level trusted issuer.
* Grant the SP/MSI required roles (e.g., `Contributor` or more granular roles on the resource group).

This avoids storing long-lived client secrets in GitHub.

### 6.2. Define Environments in GitHub

In repo settings → **Environments**:

* Create `dev`, `test`, `prod`.
* For `prod`, enable:

  * Required reviewers for deployments.
  * Protection rules (e.g., wait timers, approvals).

Store environment-specific secrets (if needed):

* `AZURE_SUBSCRIPTION_ID`
* `AZURE_TENANT_ID`
* `AZURE_CLIENT_ID` (for SP)
* Any app config that isn’t in Key Vault.

---

### 6.3. Infrastructure Deployment Pipeline

Create `.github/workflows/infra-deploy.yml`:

**Trigger:**

* `push` to `main` and/or `develop`
* Or on release tags (e.g., `v*`)

**Steps (dev example):**

1. Checkout code.
2. Authenticate to Azure using OIDC.
3. Run `az deployment group create` or `az deployment sub create` with `main.bicep` and `parameters/dev.json`.
4. Publish deployment outputs (e.g., Function App name, APIM name) as workflow outputs or environment variables.

For **prod**, use:

* `on: workflow_dispatch` or tagged releases.
* `environment: prod` with approvals.

---

### 6.4. Application Deployment Pipeline

Create `.github/workflows/app-deploy.yml`:

**Trigger:**

* `push` to `develop` → deploy to `dev`.
* `push` to `main` / release tag → deploy to `prod`.

**Pipeline steps:**

1. **Build API**

   * Restore deps, build, run tests.
2. **Deploy API to Azure**

   * For Functions: `func azure functionapp publish` or `az functionapp deployment source config-zip`.
   * For Web Apps: `az webapp deploy` or `az webapp config container set` for containers.
3. **Build & deploy UI**

   * Build static assets.
   * Deploy to Azure Static Web Apps or Blob static site.
4. **Update APIM (if code-first)**

   * Use `az apim api import` or a Bicep deployment to update APIs/policies.

Use **GitHub environments**:

```yaml
environment: dev  # or prod
```

to enforce approvals, especially for prod.

---

## 7. Post-Deployment – Operations & Monitoring

### 7.1. Observability Setup

From IaC, ensure:

* Application Insights is linked to Function App / Web App.
* APIM diagnostic logs are enabled (to Log Analytics).
* SQL / Cosmos diagnostics as needed.

Document in `docs/06-monitoring-and-alerting.md`:

* Standard dashboards:

  * API latency & failure rate
  * Function/App exceptions
  * Ingestion success vs failures
* Standard alerts:

  * 5xx errors threshold
  * Dependency failures (SQL/Cosmos) threshold
  * High CPU/DTU usage

### 7.2. Runbooks & Support

Create runbooks in `docs/runbooks/`:

* **Deployment rollback**

  * Revert to previous tag/commit
  * Redeploy via GitHub Action “Re-run workflow with previous ref”
* **Configuration changes**

  * Change config in IaC / appsettings
  * PR → merge → pipeline → deployment (no manual portal edits)
* **Incident response**

  * How to triage alerts
  * How to capture logs/telemetry
  * Escalation paths

---

## 8. Templates, Checklists & Standards

### 8.1. Templates

Provide ready-to-use templates in `templates/` or `docs/templates/`:

* Issue templates (feature, bug, spike).
* PR template (description, testing, risks).
* ADR template.
* Bicep/Terraform module templates.
* GitHub Action workflow templates (CI, infra deploy, app deploy).

### 8.2. Checklists

Maintain checklists (one per phase) under `docs/checklists/`:

1. **Discovery Checklist**

   * Problem statement documented.
   * Stakeholders identified.
   * Success metrics defined.
2. **Design Checklist**

   * Architecture diagram reviewed.
   * ADRs for key decisions.
   * Security/non-functional requirements documented.
3. **Build Checklist**

   * Coding standards followed.
   * Unit tests created.
   * Config externalized.
4. **Test Checklist**

   * Test cases mapped to requirements.
   * Integration tests passing.
   * Performance tests (if required).
5. **Release Checklist**

   * Infra deployed via pipeline.
   * Deployment dry-run in dev/test.
   * GitHub environments & approvals configured.
6. **Ops Checklist**

   * Alerts configured.
   * Runbooks in place.
   * Handover to operations documented.

---

## 9. Example Delivery Flow (End-to-End)

Putting it all together, a typical feature lifecycle looks like:
Below is an expanded, **real-world enterprise version** of the **Discovery → Design → Build → Merge & Dev Deployment → Release to Prod** lifecycle.
This is written so you can drop it directly into your **Project Delivery Guide**, under `docs/01-introduction.md` or `docs/03-methodologies-and-standards.md`.

Each phase includes:

* **Purpose**
* **Inputs & Outputs**
* **Roles involved**
* **Process steps**
* **Best practices**
* **Checklists**
* **GitHub & Azure deliverables**

---

# 🔍 **1. DISCOVERY PHASE**

*“What are we solving? Why does it matter?”*

## **Purpose**

Understand the business problem, constraints, success criteria, and expected outcomes.
This aligns stakeholders, identifies scope, and seeds architecture direction.

## **Primary Goals**

* Define the **problem statement**
* Identify **stakeholders & owners**
* Understand **business objectives & KPIs**
* Document **high-level requirements**
* Analyze **current-state systems**
* Identify risks, assumptions, dependencies
* Provide the first **delivery roadmap**

## **Key Inputs**

* Stakeholder interviews
* Business strategy docs
* Existing systems & architecture
* Compliance & security requirements

## **Key Outputs**

* Discovery Summary
* Requirements Backlog (GitHub Issues)
* High-Level Architecture Candidate
* Initial Scope + Constraints
* High-Level Project Plan

## **Roles**

* Product Owner
* Solution Architect
* Business Analyst
* Engineering Lead
* Security/Compliance SME

---

## ✔ **Discovery Workflow (Step-by-Step)**

1. **Kickoff session**

   * Define vision, business outcomes, timeline, success criteria.

2. **Gather functional requirements**

   * Use GitHub Issues (`type:feature`, `phase:discovery`).

3. **Gather non-functional requirements (NFRs)**
   Examples:

   * Security
   * Availability
   * Data residency
   * Performance
   * Cost

4. **Document current-state pain points**

   * Manual processes
   * Lack of visibility
   * Scalability issues
   * Integration gaps

5. **Identify systems & integration points**

   * Legacy apps
   * APIs
   * Identity providers
   * Databases

6. **Create the high-level architecture**

   * UI → APIM → Functions → SQL/Cosmos
   * Include rough sizing

7. **Create the initial delivery roadmap**

   * Sprint 1 → Sprint N
   * Build → Test → UAT → Launch

8. **Convert findings into GitHub Issues & Epics**

   * Epic = high-level feature
   * Issues = tasks/sub-features

---

## ✔ **Discovery Checklist**

* [ ] Business problem documented
* [ ] Stakeholders identified
* [ ] Functional & non-functional requirements collected
* [ ] Constraints captured
* [ ] Risks, assumptions documented
* [ ] High-level architecture created
* [ ] GitHub issues created for all major requirements
* [ ] Project board created
* [ ] Approval from sponsor

---

# 🏗 **2. DESIGN PHASE**

*“How will we solve the problem?”*

## **Purpose**

Translate high-level requirements into a detailed, implementable architecture and delivery plan.

## **Primary Goals**

* Finalize architecture
* Validate solution approach
* Define APIs, data models, workflows
* Create IaC modules
* Prepare for development

## **Key Inputs**

* Discovery artifacts
* Requirements backlog
* Security/compliance constraints

## **Key Outputs**

* Detailed Architecture Package
* Sequence Diagrams
* Infrastructure Designs (Bicep)
* API Contracts (OpenAPI)
* Data Models (SQL/Cosmos)
* GitHub project structure
* Solution Delivery Plan

## **Roles**

* Solution Architect
* Security Architect
* Data Architect
* Engineering Lead
* DevOps Engineer

---

## ✔ **Design Workflow (Step-by-Step)**

1. **System Architecture**

   * Define all Azure resources
   * Choose hosting model (Functions, App Service, AKS)
   * Choose storage (SQL, Cosmos, Blob)
   * Identity design (Entra ID, APIM JWT, Managed Identity)

2. **API Design (OpenAPI)**

   * Define endpoints (CRUD + ingestion)
   * Define request/response schemas
   * Error handling model

3. **Data Model Design**

   * SQL schema
   * Cosmos container structure
   * Indexing strategies
   * Partitioning

4. **Workflow & Sequence Diagrams**

   * CRUD flow
   * Ingestion pipeline
   * CI/CD flow
   * Authentication sequence

5. **Infrastructure Modules**

   * Break Bicep into modular structure
   * Define param files for each environment

6. **Security Controls**

   * APIM rate limiting
   * Managed identity access
   * NSG/Firewall considerations
   * Key Vault integration

7. **Define GitHub Delivery Structure**

   * Branching strategy
   * Environments
   * Deployment pipelines
   * Approval gates

---

## ✔ **Design Checklist**

* [ ] Architecture diagram completed
* [ ] API contracts completed
* [ ] Data models approved
* [ ] Bicep modules ready
* [ ] CI/CD architecture approved
* [ ] Security reviewed
* [ ] GitHub workflows drafted
* [ ] Architecture Decision Records created
* [ ] Design sign-off completed

---

# 🔨 **3. BUILD PHASE**

*“Build the solution according to standards and automation.”*

## **Purpose**

Develop the application code, infrastructure templates, pipelines, and documentation.

## **Primary Goals**

* Code implementation of APIs & UI
* Create Infrastructure-as-Code
* Automated CI/CD via GitHub Actions
* Unit & integration tests
* Complete documentation

## **Key Inputs**

* Approved design
* Architecture docs
* Bicep templates
* API contracts

## **Key Outputs**

* Function App code
* UI code
* Bicep IaC
* GitHub Actions pipelines
* Unit tests
* Integration tests
* Developer documentation

## **Roles**

* Developers
* DevOps Engineers
* QA Engineers
* Cloud Engineers

---

## ✔ **Build Workflow (Step-by-Step)**

1. **Develop Azure Functions**

   * Implement CRUD
   * Implement ingestion
   * Use configuration/environment variables

2. **Develop Bootstrap UI**

   * JSON-driven UI
   * Dynamic forms/tables
   * Build & minify

3. **Implement SQL Code**

   * Tables
   * Views
   * Procs
   * Index optimizations

4. **Implement Cosmos Containers**

   * Define partition key
   * Define indexing policy

5. **Write Unit Tests**

   * Function logic
   * Repository layers

6. **Write Integration Tests**

   * API tests
   * SQL/Cosmos tests

7. **Implement GitHub Actions**

   * CI workflow
   * Infra deployment workflow
   * Function App deployment workflow
   * Static Web App deployment workflow

8. **Create environment variables & secrets**

   * Azure OIDC credentials
   * SQL connection strings
   * Cosmos keys

9. **Developer Documentation**

   * README updates
   * Code comments
   * ADR updates

---

## ✔ **Build Checklist**

* [ ] Code meets standards
* [ ] All tests passed
* [ ] Pipelines defined
* [ ] All docs updated
* [ ] No unapproved secrets
* [ ] Code scanning passed (CodeQL/Dependabot)

---

# 🔀 **4. MERGE & DEV DEPLOYMENT PHASE**

*“Merge changes through PR and automatically deploy to dev.”*

## **Purpose**

Validate changes end-to-end in a dev environment using automated CI/CD.

## **Primary Goals**

* Peer-reviewed code merges
* Automatically deploy to dev
* Automated smoke tests
* Enable QA testing

## **Key Inputs**

* Feature branch
* Build artifacts
* IaC modules

## **Key Outputs**

* Updated dev environment
* Deployment logs
* Release notes
* QA sign-off (optional)

## **Roles**

* Developer
* Reviewer/Architect
* DevOps Engineer
* QA Engineer

---

## ✔ **Merge & Dev Deploy Workflow (Step-by-Step)**

1. **Developer opens Pull Request**

   * PR template auto-populates
   * Links issues (Closes #123)
   * Code scanning starts

2. **CI Pipeline Runs Automatically**

   * Build code
   * Unit tests
   * Linting
   * Security scan
   * Bicep validation (“what-if”)

3. **Peer Review**

   * Reviewers check architecture, code quality
   * Comments resolved

4. **Merge into `develop`**

   * Automatically triggers deployment pipeline

5. **Infrastructure Deploys to Dev**

   * Bicep deploy
   * APIM import/updates
   * Key Vault sync
   * Monitor diagnostic settings

6. **Application Deploys to Dev**

   * Function App deployment
   * Static UI deployment
   * Database migrations (if applicable)

7. **Post-Deployment Smoke Tests**

   * API reaches APIM
   * UI loads
   * SQL/Cosmos connectivity
   * Ingestion works

8. **Update release notes**

   * What changed
   * What needs testing

---

## ✔ **Merge & Dev Deployment Checklist**

* [ ] PR approved by required reviewers
* [ ] CI success
* [ ] Dev deployment success
* [ ] Smoke tests passed
* [ ] QA notified
* [ ] Issues linked & moved on project board

---

# 🚀 **5. RELEASE TO PROD PHASE**

*“Promote tested, approved code to production with safeguards.”*

## **Purpose**

Safely deploy stable functionality to production using controlled approvals and GitOps.

## **Primary Goals**

* Production stability
* Zero-touch automated deployments
* Observability & rollback readiness

## **Key Inputs**

* Successful dev deployment
* QA/UAT approval
* Release notes

## **Key Outputs**

* Production deployment
* Deployment documentation
* Post-deployment validation
* Monitoring confirmation

## **Roles**

* Engineering Lead
* DevOps
* Security
* Product Owner
* Support/Operations

---

## ✔ **Release to Prod Workflow (Step-by-Step)**

1. **Create Release PR or Tag**

   * PR from `develop → main`
   * Or create GitHub Release: `v1.3.0`

2. **CI Runs Again**

   * Build
   * Tests
   * Static analysis
   * Bicep validation

3. **Approval Gate Triggered**

   * GitHub environment: `prod`
   * Requires:

     * Approvers: Architects, DevOps, Product Owner
     * Optional wait timer
     * Change ticket link

4. **Infrastructure Deployment to Prod**

   * Execute Bicep with `prod.json`
   * APIM configuration
   * DNS changes if needed

5. **Application Deployment**

   * Function App slot swap (blue-green)
   * Static UI upload
   * SQL migrations (pre-approved)

6. **Post-Deployment Verification**

   * Health check
   * API operational via APIM
   * Logging & telemetry active
   * No SQL deadlocks / Cosmos throttling

7. **Finalize Deployment**

   * Close release milestone
   * Tag code
   * Update docs/runbooks

8. **Handover to Operations**

   * Notify support teams
   * Provide dashboards & alerts
   * QA final confirmation

---

## ✔ **Release to Prod Checklist**

* [ ] All UAT tests passed
* [ ] Prod approval gate satisfied
* [ ] IA & code security validated
* [ ] Prod deploy succeeded
* [ ] Telemetry validated
* [ ] Rollback plan available
* [ ] Release notes published
* [ ] Stakeholders notified

---

   * Monitoring dashboards & alerts active.
   * Incidents handled via runbooks.
   * Changes & fixes follow the same GitHub workflow.

---

If you’d like, I can next:

* Turn this into a **repo-ready README + separate `docs/` structure**, or
* Add **sample GitHub Actions YAMLs** for infra & app deployment specifically to Azure Functions + APIM + Static Web Apps.




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

