# Case Management App Builder — MVP Epics

Five epics. This is the smallest loop that proves the builder actually
works: design a case type → build a form → run a case → work a task → see
it in a list. Nothing here depends on anything not in this list.

Assumes single tenant, basic login already exists (not a product epic).

---

## Epic 1: Case Type Builder
Define what a case type is: its fields, and its stages/steps.

- Create a case type (name, code).
- Add fields to a case type (text, number, date, dropdown, reference).
- Add stages and steps to a case type, in order.
- Set a step's exit action: approve/reject (human) or auto-complete.
- Publish a case type so it can be used to create cases.

**Done when:** a user can define a case type end-to-end and publish it.

## Epic 2: Form Builder
Attach a data-entry form to a step.

- Create a form and attach it to one step.
- Add fields to the form, pulled from the case type's field list.
- Publish a form.

**Done when:** every step that needs human input has a form attached to it.

## Epic 3: Case Runtime
Create and progress an actual case from a published case type.

- Create a case instance from a published case type (UI + `POST /cases`).
- Render the current step's form for data entry.
- On step completion, evaluate its exit action and advance to the next
  step (or stop and wait if rejected).
- `GET /cases/{id}` returns current stage/step, field values, and history.

**Done when:** a case can be created and walked through every step to
completion.

## Epic 4: Case List
Find and check on cases.

- List cases of a type with a few key columns (id, stage, step, owner,
  updated).
- Search/filter the list.
- Open a case to see its current state and step history.

**Done when:** a user can find any case and see where it is.

## Epic 5: Tasks
Get the right work in front of the right person.

- When a case reaches a step needing human action, create a task.
- Assign the task to a user.
- Completing a task (approve/reject) advances the case (Epic 3).
- "My tasks" list per user.

**Done when:** a person can see their work, act on it, and the case moves.

---

## Explicitly deferred (not epics yet — one line each, not lost)

- **Multi-tenancy / per-tenant isolation** — build single-tenant first.
- **Rules/decision engine** — approve/reject/auto covers exit logic for now.
- **AI enrichment, Case Intelligence, agentic automation** — needs a
  working case model to enrich before it's useful.
- **AI-Assisted Blueprint Builder (NLP scaffolding)** — nice-to-have on
  top of a builder that already works manually.
- **AI Governance (explainability, grounding, kill switch)** — only
  relevant once there's AI in the loop.
- **Integration/connector studio** — no external systems until something
  needs one.
- **Sites/Portals (external-facing)** — internal Case List first.
- **ALM/environments/versioning pipelines** — one environment until
  there's a second team to isolate from.
- **Observability/process mining** — needs real case volume to be useful.
- **Extensibility/marketplace** — nothing to reuse yet.

If/when one of these becomes the actual bottleneck, promote it to an epic
with its own stories at that point — don't build it ahead of need.
