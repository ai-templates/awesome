# NDA Lifecycle Manager

Use case: you want to manage NDAs as a **data system** (not a folder), and extract **meaningful insights from document terms** - not only stale counters like “# signed” or “avg cycle time”.

Template link:

* [https://templates.elasticflow.app/workflows/nda-lifecycle-manager](https://templates.elasticflow.app/workflows/nda-lifecycle-manager)

Public API (OpenAPI):

* [https://api-public.elasticflow.app/swagger/doc/#/](https://api-public.elasticflow.app/swagger/doc/#/)

---

## What this template is

**NDA Lifecycle Manager** is a workflow template inside ElasticFlow that helps you:

* centralize NDAs into a single repository
* detect coverage gaps between real relationships and NDA protection
* track expirations/renewals with automation
* surface term-level issues (where clauses drift from your standards)
* turn findings into **signals and actions** (so teams know what to do next)

Important: the process runs **inside ElasticFlow**. Integrations (UI uploads or API) are only used to:

* upload data for analysis
* read results back (signals, summaries, structured outputs)

---

## Who this is for

This guide is written for two audiences at once:

* **Data-fluent operators** (Legal Ops, RevOps, Compliance, Procurement, BizOps): you think in keys, joins, quality checks, pipelines, dashboards.
* **Developers**: you want to push rows, update cells, run workflows, and pull results back programmatically.

If you don’t code, you can do the same through the ElasticFlow UI by uploading files and running the workflow manually.

---

## Why this approach gives insights you can’t get with “normal metrics”

Typical NDA reporting is stale and shallow:

* “how many NDAs exist”
* “how many were signed last month”
* “average days to signature”
* “how many expire soon”

Those metrics are useful but they miss the most expensive problems:

* which exact clauses cause negotiation delays
* what risk is actually created by deviations in definitions/exceptions
* where “coverage exists” but the terms are too weak to matter
* which counterparties repeatedly push specific unfavorable language
* what patterns predict escalation, delay, or risk acceptance

ElasticFlow becomes valuable when you treat NDA management like an analytic system:

* relationship universe (accounts/vendors/partners)
* NDA inventory with stable IDs and dates
* document term understanding (clause-level signals)
* action layer (routing, playbooks, renewals)

---

## Meaningful metrics (insight-level), not stale counters

These metrics become possible only when the system can connect:
**relationship → NDA record → document terms → workflow outcomes**.

### Clause Drift Rate (term deviations from your standard)

What it is:

* share of NDAs that deviate from your approved baseline clauses
* with detail about *which clauses drift* and *how*

What it tells you:

* which terms are repeatedly conceded
* where your template is weak or negotiation strategy is failing
* which segments (region, counterparty type, deal stage) cause deviations

How to use it:

* create “allowed deviation bands”
* set severity thresholds by deal value or risk tier
* update your template and fallback positions based on real drift patterns

### Negotiation Hotspots (what actually causes delays)

What it is:

* mapping between clause topics and negotiation friction:

  * increased iterations
  * longer cycle time
  * more escalations

What it tells you:

* the root causes of “cycle time”, not the average number

How to use it:

* build pre-approved fallback language for top hotspots
* route specific hotspot types to the right reviewer early
* reduce negotiation by changing your default template structure

### Risk Surface Coverage (coverage that reflects real protection)

What it is:

* coverage measured by the presence and quality of critical protections:

  * definition of confidential information
  * exclusions
  * term and survival
  * permitted disclosures/affiliates
  * remedies / liability boundaries
  * governing law / jurisdiction

What it tells you:

* two “signed NDAs” can create radically different real-world protection

How to use it:

* replace binary “covered/not covered” with risk-weighted coverage
* prioritize remediation where risk surface is thin

### Silent Expiration Exposure (expiration + active relationship)

What it is:

* NDAs expiring/expired that are tied to relationships still active:

  * active deals
  * ongoing projects
  * active vendor work
  * repeated confidential exchanges

What it tells you:

* where expiration is not a calendar event, but live exposure

How to use it:

* trigger renewal 90 days before expiry for high activity relationships
* escalate expirations only when activity makes it meaningful
* remove noise from “expiring soon” lists

### Fallback Acceptance Rate (what your counterparties actually accept)

What it is:

* how often your pre-approved fallback language is accepted
* segmented by clause type and segment

What it tells you:

* which negotiation positions are realistic
* what to standardize vs what to escalate

How to use it:

* improve your fallback library and templates
* predict negotiation effort by counterparty segment

---

## Data model mindset (recommended, even if you use UI-only)

You will move faster if you structure data around three entities.

### 1) Relationship (Account / Vendor / Partner)

Purpose: define “who we are dealing with” and connect activity to coverage.

Recommended fields (minimum):

* `relationship_id` (stable unique key)
* `relationship_name`
* `relationship_type` (customer/vendor/partner/prospect)
* `status` (active/inactive)
* optional: `owner`, `risk_tier`, `region`, `deal_value_band`

### 2) NDA record

Purpose: define existence, status, dates, and link to the document.

Recommended fields (minimum):

* `nda_id` (stable unique key)
* `relationship_id` (join key)
* `status` (draft/sent/signed/expired/terminated)
* `effective_date`
* `expiration_date` or `is_perpetual`
* optional: `template_type` (mutual/unilateral/custom), `governing_law`

### 3) Term signals / extracted structure

Purpose: represent what’s inside the document in a machine-friendly way.

Examples:

* extracted clause flags as JSON string
* risk markers and deviation categories
* evidence snippets (short)
* recommended actions

You can start with just Relationship + NDA record, then add term structure when ready.

---

## No-code (UI) workflow

If you don’t want to use the API, the UI flow is:

* Upload NDA inventory (CSV) and NDA documents (or links)
* Upload relationship universe (CRM export or vendor master)
* Run the NDA Lifecycle Manager workflow
* Review signals, summaries, and actions in the resulting table rows
* Export results or connect downstream tools as needed

Your job as a data-fluent operator:

* ensure stable IDs and clean joins
* define what “active relationship” means
* define what “active NDA” means (signed + not expired or perpetual)
* calibrate severities and thresholds to avoid alert fatigue

---

## Developer integration via Public API

The Public API provides:

* list workflows
* list table columns
* create rows
* update cells
* run workflows
* check workflow execution status
* read table rows back (including workflow outputs)

Auth:

* API key via header `x-api-key`

Base URL:

* `https://api-public.elasticflow.app`

### Integration pattern (recommended)

* Find workflow + its table
* Create a row (represents one analysis case or one NDA/relationship unit)
* Write inputs into cells (text, metadata, structured JSON)
* Run workflow for that row
* Poll execution until completed
* Read the same row back and parse output columns

This pattern is robust and works whether you upload one NDA at a time or batch many.

---

## API examples (copy/paste ready)

Set env:

```bash
export EF_API_KEY="YOUR_KEY"
export EF_BASE="https://api-public.elasticflow.app"
```

### 1) List workflows and find NDA Lifecycle Manager

```bash
curl -s "$EF_BASE/workflow" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

Pick:

* `workflow_id` from `workflow_list[].id`
* `table_id` from `workflow_list[].table.id`

### 2) Inspect the table schema (columns)

```bash
TABLE_ID="REPLACE_WITH_TABLE_ID"

curl -s "$EF_BASE/table/$TABLE_ID/column" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

You need `column_list[].id` for cell updates.

### 3) Create a row (a unit of work)

```bash
curl -s -X POST "$EF_BASE/table/$TABLE_ID/row" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

Save:

* `frontend_row_id`

### 4) Write inputs to cells (NDA text / metadata / extracted structure)

```bash
ROW_ID="REPLACE_WITH_FRONTEND_ROW_ID"
COLUMN_ID="REPLACE_WITH_COLUMN_ID"

curl -s -X PATCH "$EF_BASE/table/$TABLE_ID/cell" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "column_id": '"$COLUMN_ID"',
    "frontend_row_id": "'"$ROW_ID"'",
    "cell": {
      "d": "Counterparty: Acme Corp\nEffective: 2024-02-10\nExpiration: 2026-02-10\nNDA text or link: ...\nNotes: ...",
      "m": null
    }
  }'
```

Practical input strategy:

* one column for “raw NDA text or link”
* one column for “relationship context”
* one column for “structured JSON string with extracted clauses” (optional)
* let the workflow write outputs into dedicated result columns

### 5) Run the workflow for the row

```bash
WORKFLOW_ID="REPLACE_WITH_WORKFLOW_ID"

curl -s -X POST "$EF_BASE/workflow/$WORKFLOW_ID/run" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "frontend_row_id": "'"$ROW_ID"'"
  }'
```

Save:

* `workflow_execution.id`

### 6) Poll execution status

```bash
EXEC_ID="REPLACE_WITH_EXECUTION_ID"

curl -s "$EF_BASE/workflow_execution/$EXEC_ID" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

Watch:

* `status`: `NEW | IN_PROGRESS | COMPLETED | FAILED | REJECTED`
* `current_step_number` / `step_amount`

### 7) Read results back from the row

```bash
curl -s "$EF_BASE/table/$TABLE_ID/row?frontend_row_id=$ROW_ID" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

Your workflow results will typically appear as:

* human-readable summary cells
* signal cells (severity/category)
* recommended actions
* structured outputs (JSON string) for downstream BI

---

## Data quality checklist (prevents false signals)

### Identity and joins

* Relationship keys must be stable and unique
* NDA records must reference `relationship_id` consistently
* Never use names as keys (names are not unique)

### Dates

* one date format across all uploads
* no placeholder expiration unless intentional
* perpetual NDAs represented consistently (`is_perpetual=true` and empty expiration)

### Status semantics

Decide and document:

* what counts as “active relationship”
* what counts as “active NDA”
* how you handle re-execution (new NDA vs version)

### Dedupe strategy

Have explicit rules:

* if a counterparty signs multiple NDAs, do you keep all?
* which one is authoritative for coverage?
* how do you treat amendments?

---

## Operational playbooks (high leverage)

Attach simple, executable playbooks to the top signal categories.

### Expiration approaching

* renew / re-paper / accept lapse decision
* default renewal window: 90 days pre-expiry
* owner routing rules and escalation thresholds

### Coverage gap on active relationship

* pause sharing vs fast-track NDA
* minimum evidence required to mark as “active sharing”
* rapid path using approved template

### Term drift / mismatch

* define “allowed deviation bands”
* maintain fallback language library
* escalate only when deviation crosses risk tier thresholds

### Breach indicators

* escalation chain and incident process
* audit trail expectations
* internal comms template

