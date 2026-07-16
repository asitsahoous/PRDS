# Case Management App Builder — Must-Have Epics

`mvp-epics.md` proves the loop works for one app. This is what's actually
needed to be a **platform** — build multiple apps and deploy them — without
sliding back into the 17-epic version. 7 epics, ~28 stories.

"Must" = you can't call this "build and deploy an application platform"
without it. Everything else is listed at the bottom, deferred on purpose.

---

## Epic 1: Data & Object Model
Entities/fields that exist independent of any one app, so apps can reuse them.

- Define an object with typed fields (text, number, date, boolean,
  dropdown, reference to another object).
- Define a relationship between two objects (one-to-many).
- Reuse an object across more than one app.
- Define a business entity as a rollup parent for cases (e.g., Dealer →
  its Cases).

## Epic 2: App & Process Designer
Define what an application does.

- Create an app/case type (name, code, prefix), from scratch or a template.
- Attach objects (Epic 1) to the app.
- Define stages and steps, in order.
- Set each step's exit rule: human approve/reject, or auto-complete.
- Define step routing, including reject/failure paths.
- Set an SLA target per stage (due date + breach flag).
- Save as draft; validate before publish.

## Epic 3: Form Designer
Define how a person interacts with a step.

- Create a form, bound to one step.
- Drag/drop fields onto the form from the app's object model.
- Set field properties: required, read-only, visible-if.
- Preview the form with sample data before publishing.

## Epic 4: Runtime Engine
Actually run what was designed.

- Create a case instance from a published app (`POST /cases` + UI action).
- Render the bound form live for the case's current step.
- On submit, evaluate the step's exit rule and advance or halt the case.
- Persist full case state and step history (`GET /cases/{id}`).
- Support system/agent auto-completion of a step, not just human.

## Epic 5: Deployment & Versioning
This is the "and deploy" part — without it, it's a design tool, not a platform.

- Publish an app: draft → runnable version.
- Promote a published app across environments (e.g., staging → production).
- Keep version history of an app.
- Running cases stay on the app version they started on when a new
  version is published (define and enforce this rule explicitly).

## Epic 6: Case Work Management
How people actually use a deployed app day to day.

- List cases of an app: id, stage, step, owner, updated, SLA status.
- Search/filter the case list.
- Open a case: current state, field values, full step history.
- Auto-create a task when a case reaches a step needing human action.
- Assign a task to a user or queue.
- "My tasks" view; completing a task advances the case (Epic 4).

## Epic 7: Access Control
Who can build apps vs. who can work cases vs. who can see what.

- Users and roles.
- Restrict app editing (Epics 1-5) to builder roles.
- Restrict case/field visibility by role.
- Wire roles to login/session (assumes an auth provider exists — this is
  just the authorization layer on top of it).

---

## Deferred on purpose (one line each)

- **AI features** (enrichment, agentic automation, NLP blueprint builder,
  AI governance) — layer on top once the platform runs real apps.
- **Separate rules/decision engine (DMN-style)** — step exit rules cover
  "must" logic; a dedicated decision-table engine is an upgrade, not a
  requirement.
- **Multi-tenancy** — this scope is "many apps, one org." Tenant isolation
  is a different, later problem.
- **Integration/connector studio** — no external system needs wiring until
  a real app needs one.
- **Sites/Portals (external-facing)** — internal Case Work Management first.
- **Observability / process mining** — needs real case volume to matter.
- **Marketplace/extensibility SDK** — nothing to reuse yet.

Promote any of these to an epic when a real app you're deploying actually
needs it — not before.
