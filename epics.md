# Case Management Platform — Epics (V1: single application scope)

> **Superseded in scope by [`platform-epics.md`](./platform-epics.md).** This
> version models one case management application. If the goal is a
> platform that *builds and renders* case management applications
> (Appian/Pega/Camunda-style), see `platform-epics.md` instead — it keeps
> everything below as the case-instance slice of a larger system.

The source list had one row per *story*, with the story text sitting in the
"Epic Summary" column — so no epic actually had a real summary, and a couple
of stories carried stale authoring notes ("Same as 4th", "Going with 6th
Epic") that pointed at inconsistent targets. This document rewrites each
epic with an actual epic-level summary, regroups the underlying stories into
coherent capabilities, and calls out every place a reclassification,
merge, or dedup was made so it can be confirmed.

## Epic Summary Table

| # | Epic | Epic Summary |
|---|------|---------------|
| 1 | Case Foundation | Establishes the structural backbone of a case type: how it's templated, identified, tied into a business-entity hierarchy for rollup reporting, composed of configurable objects (asset sub-types, forms, rules), and exposed through a REST API for programmatic creation and retrieval. |
| 2 | Case Lifecycle | Defines and visualizes how a case type moves from open to close — stages and steps, the forms/workflows/agents/tasks attached to each step, exit and SLA rules — plus the progress bar, drilldown, and swimlane views that make the process legible, a preview mode distinct from live runtime, and the ability for agents to insert steps dynamically. |
| 3 | Case Explorer | The day-to-day command center for finding and triaging cases: configurable list columns per case type, inline task/quality signals, expandable rows that surface extraction and insight without leaving the list, full search/filter, and AI-assisted routing of incoming documents to the right case. |
| 4 | Case Intelligence | The AI layer over a case: a timestamped narrative of agent activity, count-based health/quality alerts, predictive SLA-breach probability, configurable AI enrichment on any field, a machine-readable state/action API for agentic consumption, a connected Case Map across the knowledge graph, and the handoff/HITL/checkpointing policies that govern agent–human collaboration. |
| 5 | Tasks & Communications | Where the work and correspondence of a case live: per-user task and work queues, a chronological case feed of agent events, automatic re-evaluation of case stage when a document is uploaded, and a fully versioned communication audit trail. |
| 6 | AI Assisted Blueprint Builder | Lets an author describe a step — or an object, such as a business entity — in natural language and have the system scaffold the matching app or object, resolved deterministically from curated, tenant-scoped templates and explicit configuration rather than open-ended generative synthesis. |
| 7 | Dashboards | The agentic-automation landing page: a single entry point that surfaces portfolio-level status across cases before a user drills into Case Explorer or an individual case. |
| 8 | AI Governance | The control layer that keeps AI trustworthy: explainability of agent decisions and scores, grounding and citation enforcement so every conclusion traces to a governed source, provenance, model/prompt registries, and kill-switch operations. |

---

## 1. Case Foundation

**Summary:** Establishes the structural backbone of a case type: how it's
templated, identified, tied into a business-entity hierarchy for rollup
reporting, composed of configurable objects, and exposed through a REST API.

**Capabilities:**
- **Case Template Design** — initiate case design from a template.
- **Case Identity & Metadata** — capture name, code, prefix, and sub-case
  alignment at setup.
- **Business Entity Hierarchy** — relate cases/sub-cases into a parent
  entity for rollup (e.g., Dealer Group → all cases for that dealer).
- **Case Object Model** — define the objects a case is built from: asset
  sub-types, forms, rules.
- **Case Management API** — `POST /cases` (create with type, entity ID,
  trigger mode, seed metadata) and `GET /cases/{id}` (full object: current
  stage, step states, history).

**Reclassification notes:**
- *"Ability to define business entity as top level"* moved to **AI Assisted
  Blueprint Builder**, per the source note "(Going with 6th Epic)" — the 6th
  epic in the original ordering is Blueprint Builder. Read literally, entity
  definition becomes an object you scaffold via the NLP-assisted builder
  rather than a standalone Foundation capability. Foundation keeps the
  *relationship/rollup* mechanics (cases → parent entity), since that's data
  modeling, not authoring. **Confirm this is the intended split** — it's a
  large behavioral change from "entity is defined here" to "entity is
  scaffolded there."
- *"Ability to define objects for the case… (Same as 4th)"* — the "4th" story
  in the source list is the entity-rollup story, which doesn't match this
  story's content (asset sub-types/forms/rules). Treated as a stale note and
  kept as its own capability, **Case Object Model**. Flag for confirmation
  in case "4th" referred to a numbering outside this list.

---

## 2. Case Lifecycle

**Summary:** Defines and visualizes how a case type moves from open to
close, and gives runtime flexibility for agents to extend it.

**Capabilities — Lifecycle Definition:**
- Define stages and steps.
- Define step attributes.
- Relate steps via workflow calls, including rejections and failures.
- Define exit/completion rules per step (human approve/reject, agent).
- Configure SLA targets and breach actions per stage.

**Capabilities — Step Composition:**
- Attach a form, workflow, agent, or task to a step.
- Drag/drop form editor, connected to cases at the step level.
- Enable/disable form UI features per step owner.

**Capabilities — Visualization & Preview:**
- Horizontal stage progress bar.
- Drilldown from a stage into its steps/processes.
- Swimlane diagram for the whole case lifecycle.
- Preview mode, distinct from live runtime.

**Capabilities — Runtime Flexibility:**
- Agents can add dynamic steps at runtime.

**Note:** This is by far the largest epic (13 source stories). Kept as one
epic for now since it's one cohesive "process design" concept, but it's a
candidate to split into **Lifecycle Designer** (definition + step
composition) and **Lifecycle Visualization** (progress bar / drilldown /
swimlane / preview) if it needs a separate delivery track.

---

## 3. Case Explorer

**Summary:** The day-to-day command center for finding, triaging, and
previewing cases without leaving the list.

**Capabilities:**
- Configure case list columns per case type.
- See open and overdue task counts inline.
- Inline row expansion shows extraction and InSight without navigation.
- Search and filter the case list.
- AI document-to-case association (route an incoming document to the
  correct case).

**Cross-epic note:** AI document-to-case *association* (this epic) and
"auto-process uploaded document and re-evaluate stage" (Tasks &
Communications) are sequential steps of one pipeline — association happens
first, processing/stage re-evaluation second. Kept in separate epics but
flagged as a dependency.

---

## 4. Case Intelligence

**Summary:** The AI layer over a case — insight, prediction, and the
governed mechanics of agent action.

**Capabilities:**
- Timestamped agent narrative with typed signal bullets.
- Count-based health check with clickable quality alerts.
- Predicted SLA breach probability with date range.
- Configure AI enrichment on any case field.
- Expose case state and actions as machine-readable APIs (for agentic /
  automation consumption).
- Case Map — see everything a case connects to in the knowledge graph:
  entity, document, sub-case, transaction, audit events, forms, tasks.
- Agent handoff and HITL policy design, workflow checkpointing.

**Reclassification notes:**
- This epic's API and Case Foundation's REST API are **not duplicates**:
  Foundation's is case CRUD/lifecycle state (`POST /cases`, `GET
  /cases/{id}`); this one exposes state + actions for agent/automation
  consumption. Recommend distinct backlog naming — e.g. "Case Management
  API" vs. "Agentic Action API" — so they don't get conflated.
- "Agent handoff and HITL policy design, workflow checkpointing" overlaps
  with Case Lifecycle's per-step exit rules (human approve/reject, agent)
  and with AI Governance's control mandate. Kept here as the home for the
  handoff *mechanics*, on the understanding that Lifecycle defines *where*
  checkpoints occur and Governance defines the *policy constraints* around
  them.

---

## 5. Tasks & Communications

**Summary:** Where the work and correspondence of a case live.

**Capabilities:**
- Track tasks / work queues at the user level.
- Switch to Case Feed for a chronological view of agent events.
- Auto-process an uploaded document and re-evaluate case stage.
- See complete communication history with all versions (audit trail).

---

## 6. AI Assisted Blueprint Builder

**Summary:** Describe a step — or an object — in natural language and get a
deterministically scaffolded app or object back.

**Capabilities:**
- NLP-assisted setup: an author describes a step in natural language.
- Convert that natural-language description into a scaffolded app or
  object.
- Deterministic, tenant-scoped resolution (D-04): any scaffold resolves via
  curated, tenant-scoped templates and explicit configuration, evaluated
  against an evidence contract — never open-ended, corpus-fed generation.
- *(Moved in from Case Foundation)* Define a business entity as a top-level
  object, authored via this same NLP-assisted scaffolding path.

**Reclassification note:** The source had two stories stating the same
tenant-scoped/non-generative guardrail ("Resolve any scaffold via curated,
tenant-scoped templates…" and "Keep the AI Blueprint Assistant tenant-scoped
(D-04)…"). Merged into one capability so the constraint is stated once.

---

## 7. Dashboards

**Summary:** The agentic-automation landing page.

**Capabilities:**
- Agentic Automation Dashboard / landing page.

**Note:** Currently a single-capability epic. Kept separate since a landing
page is a distinct product surface likely to grow its own set of widgets,
but it's thin enough that folding it into Case Intelligence is a reasonable
alternative if the team prefers fewer epics.

---

## 8. AI Governance

**Summary:** The control layer that keeps AI trustworthy.

**Capabilities:**
- Explainability of agent decisions and scores.
- Grounding and citation enforcement — every conclusion traces to a
  governed source (provenance).
- Registries, kill switch, and operations.

**Note:** Also thin (2 source stories) and conceptually close to Case
Intelligence's agent-handoff/HITL capability. Kept separate because
governance is a distinct, compliance-driven concern with its own
stakeholders (audit/risk), even though it will need to work hand-in-hand
with Intelligence's runtime agent behavior.

---

## Open Questions for Confirmation

1. Is moving "define business entity as top level" from Case Foundation to
   AI Assisted Blueprint Builder actually intended, or did the original
   "Going with 6th Epic" note mean something else (e.g., a 6th epic in a
   different, unshown list)?
2. What did "(Same as 4th)" on the case-object-model story actually refer
   to? Nothing in this list matches it — flagging in case there's a master
   backlog with different numbering.
3. Is Case Lifecycle's size (13 stories) a problem for planning/delivery,
   or fine to keep as one epic?
4. Are Dashboards and AI Governance meant to stay as standalone epics
   long-term, or are they placeholders that will absorb more scope later?
