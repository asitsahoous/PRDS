# Case Management Application Builder — Platform Epics (V2)

## Why this looks nothing like `epics.md`

`epics.md` modeled **a single case management application** — one case type,
its lifecycle, its explorer, its intelligence layer. This document models
**a platform that lets someone design and run many different case
management applications** — the way Appian, Pega, or Camunda work: a
Studio where you author a case type, and a runtime engine that renders and
executes whatever was authored, for any tenant, at scale.

That reframe adds whole layers that a single app never needs: a design-time
IDE as its own product surface, an object/data model that exists
independently of any one case type, a rendering runtime that is a distinct
system from the designer that produced the definition, environments and
release management, multi-tenancy and per-tenant security, an integration
layer for connecting to the outside world, and a marketplace for reusable
components. V1's epics don't disappear — they get absorbed as the
*case-instance* slice of a much bigger system.

## Research grounding

- **Appian** — Case Management Studio ships modular, prebuilt case patterns
  covering ~80% of common case needs; sits on a **Data Fabric** (a virtual
  layer unifying data from many sources without migration, modeled as
  Records with typed relationships); ships **Sites/Portals** for
  internal and external-facing experiences; **AI Copilot** + **Process
  Mining** for insight and continuous improvement; a unified
  **ALM/DevOps** experience spanning environments and governance, with
  AI actions themselves "governed, controllable, and tied to business
  impact." ([Appian low-code](https://appian.com/products/platform/low-code), [Case Management](https://appian.com/products/platform/process-automation/dynamic-case-management), [Data Fabric](https://docs.appian.com/suite/help/26.6/data-modeling-with-appian-records.html), [22.4 Portals/DevOps/Process Mining](https://appian.com/blog/2022/appian-22-4--codeless-data-fabric-plus-better-portals-devops-process-mining-security), [AI](https://appian.com/products/platform/artificial-intelligence))
- **Pega** — strict hierarchy: **Case → Stages → Processes → Steps**,
  configured in a low-code Case Designer; a hard structural split between
  **Work-classes** (the case/process) and **Data-classes** (the object
  model) — i.e., process and data are separate first-class layers, not one
  blob. ([Pega Case Management](https://docs.pega.com/bundle/platform/page/platform/case-management/case-management-overview.html), [MyKnowTech](https://myknowtech.com/blog/code-vault/case-management-case-stages-processes-and-steps/))
- **Camunda** — a process/case engine decomposes into four subsystems: a
  command-driven public API, the BPMN/DMN execution core, an async job
  executor, and a persistence layer — with *separate* observability
  tooling (Operate/Optimize) watching engine health, distinct from any
  single case's health. ([Camunda architecture](https://eitt.academy/knowledge-base/camunda-bpm-complete-business-process-management-guide/), [DMN engine](https://camunda.com/platform/decision-engine/))
- **CMMN** (OMG's Case Management Model & Notation standard) — Stages are
  "episodes" of a case; **Milestones** are achievable progress targets with
  no work directly attached; **Sentries** are entry/exit criteria attached
  to tasks and stages (this *is* "exit/completion rules," under its formal
  name); **Discretionary** tasks/stages are not part of the plan by
  default but a case worker can add them at runtime — direct precedent for
  "agents add dynamic steps at runtime." ([Visual Paradigm CMMN guide](https://www.visual-paradigm.com/guide/cmmn/what-is-cmmn/), [OMG CMMN spec](https://www.omg.org/spec/CMMN/1.0/PDF))
- **General low-code architecture** — the common pattern is a metadata
  repository (forms/workflows/data models as structured JSON/XML) plus a
  runtime interpreter that renders and executes that metadata, an
  integration layer of connectors/adapters, and multi-tenant
  infrastructure underneath. Design-time and runtime are architecturally
  separate systems that share only the metadata contract between them.
  ([metadata-driven pattern](https://www.claysys.com/blog/metadata-driven-application-development/))

## The four pillars

| Pillar | Question it answers |
|---|---|
| **Design-Time (Studio)** | How does someone author a case type, its data, its process, its forms, its rules — without writing code? |
| **Runtime (Engine)** | How does a case instance actually get created, executed, and rendered from what was authored? |
| **Intelligence** | What does the platform tell you about a case, and what can its agents do on your behalf, under what controls? |
| **Platform** | What makes this safe and operable for many tenants, many apps, and many releases at once? |

---

## Epic Summary Table

| Pillar | # | Epic | Summary |
|---|---|---|---|
| Design-Time | 1 | **Object & Data Model Studio** | Define the entities, fields, and relationships an app is built from — independent of any single case type — and roll cases up into parent business entities. This is the "data fabric" layer: the schema every other epic reads and writes against. |
| Design-Time | 2 | **Case Type & App Studio** | The low-code authoring surface itself: start from a template, name/code/prefix an app, compose it from data objects, forms, workflows, rules, and agents, and manage draft/published versions of the definition. |
| Design-Time | 3 | **Process & Lifecycle Designer** | Model a case type's lifecycle as stages, milestones, and steps, with entry/exit sentries, SLA targets, and step-to-step routing (including rejections and failures) — plus the swimlane/progress visualizations that make the design legible. |
| Design-Time | 4 | **Forms & Experience Designer** | A drag-drop form builder wired to lifecycle steps, with per-step UI feature toggles and a preview mode that never touches live data. |
| Design-Time | 5 | **Rules & Decision Designer** | Define the decision tables, expressions, and exit/approval conditions (human or agent) that steps evaluate — the DMN-equivalent layer referenced by, but decoupled from, the Lifecycle Designer. |
| Design-Time | 6 | **AI-Assisted Blueprint Builder** | Describe a step or object in natural language and get a scaffolded app/object back, resolved deterministically from curated, tenant-scoped templates — never open-ended generative synthesis. |
| Design-Time | 7 | **Integration & Connector Studio** | Build and reuse connectors to external systems, expose/consume webhooks, and manage the credentials and contracts that let a case type talk to the outside world. |
| Runtime | 8 | **Case Runtime Engine** | Instantiate a case from its type definition and drive it through the stage/step state machine at execution time — including discretionary steps agents add on the fly — exposed as a REST API (`POST /cases`, `GET /cases/{id}`) for anything that needs to create or query a case programmatically. |
| Runtime | 9 | **Forms Rendering Runtime** | Take a form definition authored in the Experience Designer and render it live for a real case instance, enforcing the same feature toggles and exit rules the designer configured — the runtime counterpart to Epic 4. |
| Runtime | 10 | **Task & Work Management** | Per-user/per-queue task tracking, SLA breach actions, and the chronological case feed of agent and human events. |
| Runtime | 11 | **Case Explorer & Experience Sites** | The internal case list/grid (configurable columns, inline task/quality signals, search/filter, inline preview) and the external-facing portal surface for customers/constituents interacting with a case. |
| Intelligence | 12 | **Case Intelligence & Agentic Automation** | AI enrichment on any field, predictive SLA-breach probability, a Case Map across the connected knowledge graph, agent handoff/HITL/checkpointing policy, and the agentic-automation landing dashboard. |
| Intelligence | 13 | **AI Governance & Trust** | Explainability of agent decisions and scores, grounding/citation enforcement so every conclusion traces to a governed source, provenance, model/prompt registries, and kill-switch operations. |
| Platform | 14 | **Identity, Security & Multi-Tenancy** | Tenant isolation, role/attribute-based access down to the object and field level, and SSO — the boundary that makes it safe to run many tenants' apps on one platform. |
| Platform | 15 | **Application Lifecycle Management & Governance** | Environments (dev/test/prod), versioning and change sets, deployment pipelines, and an audit trail of changes to app *definitions* themselves (distinct from audit of case *data*). |
| Platform | 16 | **Observability, Analytics & Process Intelligence** | Platform- and engine-level health monitoring, operational reporting across case types, and process mining to find bottlenecks in how cases actually move versus how they were designed to move. |
| Platform | 17 | **Extensibility & Marketplace** | A plugin/widget SDK and a template/component marketplace so capabilities built for one app (or one tenant) can be reused across others. |

---

## Design-Time Pillar (the Studio)

### 1. Object & Data Model Studio
- Define entities, fields, and typed relationships (one-to-many, etc.)
  independent of any single case type.
- Relate cases/sub-cases into a parent business entity for rollup (e.g.,
  Dealer Group → all cases for that dealer).
- Define case-scoped objects: asset sub-types, forms, rules, as
  references into this shared model.
- **Why it's separate from Case Type Studio:** Pega's Work-/Data-class
  split and Appian's Data Fabric both treat "what the data looks like" as
  a layer independent of "what a specific case type does with it" — so
  one entity definition can be reused across many case types.

### 2. Case Type & App Studio
- Initiate case design from a template.
- Capture case identity: name, code, prefix, sub-case alignment.
- Compose a case type from data objects (Epic 1), forms (Epic 4),
  workflows (Epic 3), rules (Epic 5), and agents (Epic 6/12).
- Draft vs. published versions of a case type definition, promoted
  through environments (see Epic 15).

### 3. Process & Lifecycle Designer
- Define stages, milestones, and steps.
- Define entry/exit sentries per step (human approve/reject, agent) —
  CMMN's formal term for "exit/completion rules."
- Configure SLA targets and breach actions per stage.
- Relate steps via workflow calls, including rejections and failures.
- Define step attributes.
- Author-time visualizations: horizontal stage progress bar, drilldown
  from stage into steps, swimlane diagram of the whole lifecycle.
- Mark steps/stages as **discretionary** — available for an agent or
  case worker to add at runtime, not part of the default plan (feeds
  Epic 8's dynamic-step capability).

### 4. Forms & Experience Designer
- Drag/drop form editor connected to case steps.
- Enable/disable form UI features per step owner.
- Preview mode, fully isolated from live runtime data.

### 5. Rules & Decision Designer
- Decision tables and expressions that steps evaluate for routing,
  enrichment, and exit conditions.
- Kept as its own epic (rather than folded into Lifecycle) because,
  like Camunda's separate DMN engine, decision logic is reused across
  many steps and case types and versions independently of the process
  that calls it.

### 6. AI-Assisted Blueprint Builder
- NLP-assisted setup: describe a step, or an object such as a business
  entity, in natural language.
- Convert that description into a scaffolded app or object.
- Deterministic, tenant-scoped resolution (D-04): resolve every scaffold
  from curated, tenant-scoped templates and explicit configuration against
  an evidence contract — never open-ended, corpus-fed generation.

### 7. Integration & Connector Studio
- Build/reuse connectors to external systems (databases, SaaS APIs, core
  systems of record).
- Expose and consume webhooks.
- Manage connection credentials and versioned integration contracts.
- **New vs. V1:** nothing in the original list covered how a case type
  talks to the outside world — every enterprise case-management platform
  (Appian's Connected Systems, Camunda's connectors) treats this as
  first-class, separate from the case's own object model.

---

## Runtime Pillar (the Engine)

### 8. Case Runtime Engine
- Instantiate a case from its type definition (template → live instance).
- Drive the stage/step state machine at execution time: entry/exit
  sentries, SLA timers, routing, rejections/failures.
- Allow agents to add discretionary/dynamic steps at runtime.
- `POST /cases` (create with type, entity ID, trigger mode, seed
  metadata) / `GET /cases/{id}` (current stage, step states, history).
- **Why separate from the Lifecycle Designer:** the designer produces a
  definition; the engine is the thing that actually executes it for a
  real case, at scale, across tenants — the same separation Camunda
  draws between its BPMN core and everything upstream of deployment.

### 9. Forms Rendering Runtime
- Render a form definition (Epic 4) live against a real case instance.
- Enforce the same per-step feature toggles and exit rules the designer
  configured, at runtime, for the actual step owner.
- **This is the literal answer to "render the case management
  application runtime too"** — a metadata interpreter that turns a form
  definition into a working UI, the same pattern used by every
  metadata-driven low-code platform.

### 10. Task & Work Management
- Per-user task/work queues.
- SLA breach actions surfaced to the right owner.
- Chronological case feed of agent and human events.
- Complete communication history with all versions (audit trail).

### 11. Case Explorer & Experience Sites
- Configurable case list columns per case type; open/overdue task counts
  inline; inline row expansion for extraction/insight; search and filter.
- AI document-to-case association at intake.
- **New vs. V1:** an internal-only case list is not the same product
  surface as an external-facing portal for the customer/constituent
  involved in the case — Appian ships these as a distinct "Sites/Portals"
  capability, and a builder platform needs both: an internal Explorer and
  a way to expose a rendered case experience to people outside the org.

---

## Intelligence Pillar

### 12. Case Intelligence & Agentic Automation
- Timestamped agent narrative with typed signal bullets.
- Count-based health check with clickable quality alerts.
- Predicted SLA breach probability with date range.
- Configurable AI enrichment on any case field.
- Case state/actions exposed as machine-readable APIs for agentic
  consumption (distinct from Epic 8's case-management CRUD API).
- Case Map: everything a case connects to in the knowledge graph —
  entity, document, sub-case, transaction, audit events, forms, tasks.
- Agent handoff and HITL policy design, workflow checkpointing.
- Agentic Automation Dashboard / landing page.

### 13. AI Governance & Trust
- Explainability of agent decisions and scores.
- Grounding and citation enforcement — every conclusion traces to a
  governed source (provenance).
- Model/prompt registries, kill switch, and operations.

---

## Platform Pillar

### 14. Identity, Security & Multi-Tenancy
- Tenant isolation for data, configuration, and users.
- Role/attribute-based access control down to object and field level.
- SSO and identity federation.
- **New vs. V1:** V1 assumed one app for one organization. A builder
  platform that renders many tenants' apps cannot exist without this as
  a first-class epic — every platform researched (Appian, Pega, generic
  low-code architecture) treats multi-tenancy as foundational
  infrastructure, not a feature of any one app.

### 15. Application Lifecycle Management & Governance
- Environments: dev/test/prod (or per-tenant sandboxes).
- Versioning and change sets for app definitions; promotion pipelines.
- Audit trail of changes to the *definition* (who changed what stage,
  form, or rule, and when) — distinct from the case-data audit trail in
  Epic 10.
- **New vs. V1:** Appian's "unified ALM/DevOps experience" and Pega's
  rule-versioning are core to how these platforms let many teams build
  many apps without stepping on each other; nothing in V1 addressed how
  an app definition itself gets built, tested, and released.

### 16. Observability, Analytics & Process Intelligence
- Platform/engine health monitoring (queue depth, job executor failures,
  render latency) — distinct from any single case's health.
- Operational reporting across case types and tenants.
- Process mining: compare how cases actually move against how the
  lifecycle was designed, to find bottlenecks and design drift.
- **New vs. V1:** V1's "health check" (now under Epic 12) is about one
  case. This epic is about the platform and the population of cases
  running on it — the same distinction Camunda draws between an
  application's own logs and Operate/Optimize watching the engine.

### 17. Extensibility & Marketplace
- Plugin/custom-widget SDK.
- Template and component marketplace so a form, connector, or rule set
  built once can be reused across apps and tenants.
- **New vs. V1:** every mature low-code platform researched has this as
  a distinct capability (Appian's App Market, Pega's rule reuse across
  the class hierarchy) — without it, every app rebuilds the same pieces.

---

## What actually changed vs. V1

- **V1 assumed the app already existed.** V2 assumes you're building the
  thing that builds and runs the app — so Epics 1, 2, 7, 14, 15, 16, and
  17 didn't need to exist in V1 and are the biggest true gaps it had.
- **Design-time and runtime are now separate epics, on purpose**, mirroring
  every platform researched: Lifecycle Designer (3) vs. Case Runtime Engine
  (8); Forms Designer (4) vs. Forms Rendering Runtime (9). V1 conflated
  "define the lifecycle" and "run the lifecycle" into one epic.
- **CMMN gives V1's more ambiguous stories real names**: "exit/completion
  rules" are sentries; "agents add dynamic steps at runtime" is exactly
  CMMN's discretionary task/stage mechanic. Worth adopting this vocabulary
  going forward since it's a formal standard, not a house term.
- **Case Explorer split into internal (Explorer) vs. external (Sites)**,
  since a builder platform has to serve both an internal caseworker view
  and an external customer/constituent-facing rendering of the same case.
- **Rules/Decisions pulled out of Lifecycle into its own epic**, matching
  Camunda's separate DMN engine — decision logic outlives and is reused
  across the process steps that call it.
- **AI Governance, AI Assisted Blueprint Builder, and Dashboards from V1
  survive largely intact** — they were already platform-shaped concerns,
  not single-app concerns, which is why they didn't need to move.

## Open questions for confirmation

1. Does "render the case management type application runtime too" mean a
   full custom UI renderer (Epic 9, forms-as-metadata → live UI), or is a
   configuration-driven wrapper around an existing UI framework
   acceptable for v1 of the builder?
2. Is multi-tenancy a hard requirement from day one, or can Epic 14 start
   as single-tenant with tenant isolation added later? This materially
   changes how early the Object & Data Model Studio needs to be tenant-aware.
3. Do you want Sites/Portals (external-facing) in scope for the first
   build, or is an internal-only Case Explorer sufficient initially, with
   Epic 11's external half deferred?
4. Should Rules & Decision Designer (Epic 5) be a full DMN-style decision
   table engine, or a lighter expression/condition builder to start?
