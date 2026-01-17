# Contract Execution Time

This guide shows how to use and integrate ElasticFlow’s **Contract Execution Time** metric template in two modes:

- **No-code (UI):** upload data through the interface, run everything inside ElasticFlow, export results.
- **Developer (API):** push rows/cells, run the workflow, poll execution status, read results back.

Key principle: **analysis + workflow execution happen inside ElasticFlow**. Integrations are only for:
- uploading data for analysis
- retrieving outputs (results, signals, summaries) back to your systems

Links:
Template:
https://templates.elasticflow.app/workflows/metrics/contract-execution-time

Public API (Swagger):
https://api-public.elasticflow.app/swagger/doc/#/

---

## 1) What “Contract Execution Time” means

**Contract Execution Time** is the time between “contract is ready to sign” and “fully executed by all parties”. It’s “time stuck at the finish line”, and most of it is waiting: approvals, signatures, responses.

ElasticFlow frames the common causes as:

* unclear ownership (no one knows whose turn it is)
* no follow-up system (things sit in inboxes)
* too many approval loops (contracts bounce for more sign-offs)

---

## 2) What the template is designed to fix (the four bottleneck archetypes)

The template page groups execution-time pain into four recurring bottlenecks:

* **Approvals take too long** (legal/finance/exec handoffs, looping sign-offs, unclear approvers)
* **Signature is stalled** (counterparty delay, no visibility whether they opened it)
* **Too many redlines** (negotiation rounds, late exceptions, standard terms repeatedly questioned)
* **Handoffs slow everything** (status trapped in email, no shared owner, constant “where is this?”) 

The “How ElasticFlow Helps” sections emphasize three levers:

* shared visibility (“one dashboard”)
* automated reminders + escalation when stalled
* faster execution by reducing waiting time

---

## 3) Why this becomes insight-driven (not stale reporting)

Most teams only measure:

* average time to sign
* number signed
* number in signature

Those are outcome counters. Useful, but they don’t explain **why** you’re slow.

A data-first setup lets you answer decision questions:

* Which stage accounts for most delay: internal approvals, counterparty signature, redline negotiations, or handoffs? 
* Which contract types/value bands trigger approval loops?
* Which counterparties correlate with long signature waiting?
* Which reminder sequence actually moves signatures without being pushy? 

ElasticFlow’s own recommendation for quick progress is aligned with this: start with visibility + reminders first, then improve process and policy once you can see the real bottleneck. 

---

## 4) The meaningful metrics (what you should actually track)

These are the metrics that unlock real process changes. Start with the first two.

### A) Distribution, not average

Track:

* p50 / p75 / p90 execution time
* segmented by: contract type, value band, region, owner/team, counterparty type

Why:

* averages hide the tail; the tail is where deals slip.

### B) Stage contribution (“where time is spent”)

Define stage clocks such as:

* internal approval waiting time
* counterparty signature waiting time
* redline iteration time
* handoff latency (time between a status change and next action)

ElasticFlow explicitly suggests diagnosing bottlenecks by looking at where contracts spend the most time and fixing the dominant stage first. 

### C) Approval loop indicators

Track:

* number of approvals per contract
* re-approvals (same function re-approves)
* loop triggers (type/value/exception category)

Why:

* “approval loops” are one of the stated root causes; you can measure and then design routing/tiering rules. 

### D) Reminder and escalation effectiveness

Track:

* time from “sent for signature” → “signed”
* time-to-sign after each reminder
* escalation success rate

ElasticFlow’s recommended baseline sequence is 24h, 48h, then escalation at 72h. 

### E) Ownership clarity

Track:

* “missing owner” rate
* time to next action after stage transitions
* number of handoffs per contract

Why:

* unclear ownership and handoff delays are explicitly called out as core failure modes. 

---

## 5) Data model: what to upload (works for UI and API)

You’ll move fastest if you treat each contract as a record with stable IDs and consistent timestamps.

### Minimum fields (to compute Contract Execution Time)

* `contract_id` (stable unique ID)
* `counterparty` (normalized)
* `contract_type`
* `owner` (internal)
* `ready_to_sign_at` (timestamp)
* `fully_signed_at` (timestamp)

### Highly recommended fields (to diagnose bottlenecks)

* `sent_for_signature_at`
* `current_stage` (or stage history)
* approval events: approver, approved_at
* redline iteration markers: redline_round_count, last_redline_at
* `close_deadline` (if this impacts revenue timing)
* `value_band` (or estimated value)

Example “contract record” you can store as a JSON string (in a single cell if needed):

```json
{
  "contract_id": "CT-10293",
  "counterparty": "Acme Corp",
  "contract_type": "MSA",
  "owner": "LegalOps",
  "ready_to_sign_at": "2026-01-05T10:30:00Z",
  "sent_for_signature_at": "2026-01-05T12:00:00Z",
  "fully_signed_at": "2026-01-08T09:15:00Z",
  "value_band": "50k-100k",
  "close_deadline": "2026-01-10"
}
```

---

## 6) No-code workflow (UI): fastest path to value

If you don’t code, you can do everything in the interface.

### Step 1 — Upload contract records

* start with 50–200 contracts if you want insights quickly
* ensure stable `contract_id` and consistent timestamps

### Step 2 — Create one shared status view (“where every contract stands”)

ElasticFlow positions this as a primary benefit: one dashboard instead of status in email. 

### Step 3 — Turn on reminders + escalation

Start here if time is limited. ElasticFlow calls signature tracking + reminders the fastest win and gives a 24h/48h/72h sequence. 

### Step 4 — Choose the bottleneck pack (optional, but powerful)

The template page provides “starter packs” by bottleneck.

* If approvals take too long:

  * Automated Contract Review Workflow
  * Contract Compliance Monitor
  * Clause Risk Analyzer 

* If signature is stalled:

  * reminder sequence + pending signatures dashboard
  * (listed on the page) Contract Negotiation Intelligence, Renewal Risk Monitor

* If too many redlines:

  * identify standard vs risky redlines
  * track redline patterns to improve templates

* If handoffs slow everything:

  * single status dashboard
  * stage-change notifications
  * ensure required steps aren’t skipped

---

## 7) Developer integration (Public API): upload → run → fetch results

Auth:

* header `x-api-key: <your_key>`

The Public API model is table-driven:

* a workflow is linked to a table
* you create a row
* you write input values into cells
* you run workflow for that row
* the workflow writes outputs back into table cells
* you read the row back and extract outputs

### API endpoints you’ll use (from OpenAPI you shared)

* `GET /workflow` — list workflows
* `GET /table/{table_id}/column` — list columns (to find `column_id`)
* `POST /table/{table_id}/row` — create row (returns `frontend_row_id`)
* `PATCH /table/{table_id}/cell` — update cell value
* `POST /workflow/{workflow_id}/run` — run workflow for a row
* `GET /workflow_execution/{workflow_execution_id}` — poll status
* `GET /table/{table_id}/row?frontend_row_id=...` — read row back

### End-to-end example (bash)

```bash
export EF_API_KEY="YOUR_KEY"
export EF_BASE="https://api-public.elasticflow.app"

# 1) List workflows (find workflow_id + table_id for Contract Execution Time)
curl -s "$EF_BASE/workflow" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"

# 2) List table columns (find column_id for your input fields)
TABLE_ID="REPLACE_WITH_TABLE_ID"
curl -s "$EF_BASE/table/$TABLE_ID/column" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"

# 3) Create a row
ROW_ID=$(
  curl -s -X POST "$EF_BASE/table/$TABLE_ID/row" \
    -H "x-api-key: $EF_API_KEY" \
    -H "Accept: application/json" \
  | python -c 'import sys, json; print(json.load(sys.stdin)["frontend_row_id"])'
)
echo "ROW_ID=$ROW_ID"

# 4) Write an input cell (store structured JSON as a string)
COLUMN_ID="REPLACE_WITH_COLUMN_ID"
curl -s -X PATCH "$EF_BASE/table/$TABLE_ID/cell" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "column_id": '"$COLUMN_ID"',
    "frontend_row_id": "'"$ROW_ID"'",
    "cell": {
      "d": "{\"contract_id\":\"CT-10293\",\"counterparty\":\"Acme Corp\",\"contract_type\":\"MSA\",\"owner\":\"LegalOps\",\"ready_to_sign_at\":\"2026-01-05T10:30:00Z\",\"sent_for_signature_at\":\"2026-01-05T12:00:00Z\",\"fully_signed_at\":\"2026-01-08T09:15:00Z\"}",
      "m": null
    }
  }'

# 5) Run workflow
WORKFLOW_ID="REPLACE_WITH_WORKFLOW_ID"
EXEC_ID=$(
  curl -s -X POST "$EF_BASE/workflow/$WORKFLOW_ID/run" \
    -H "x-api-key: $EF_API_KEY" \
    -H "Content-Type: application/json" \
    -H "Accept: application/json" \
    -d '{"frontend_row_id":"'"$ROW_ID"'"}' \
  | python -c 'import sys, json; print(json.load(sys.stdin)["workflow_execution"]["id"])'
)
echo "EXEC_ID=$EXEC_ID"

# 6) Poll status (simple example)
curl -s "$EF_BASE/workflow_execution/$EXEC_ID" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"

# 7) Read row back (outputs are in cells)
curl -s "$EF_BASE/table/$TABLE_ID/row?frontend_row_id=$ROW_ID" \
  -H "x-api-key: $EF_API_KEY" \
  -H "Accept: application/json"
```

### What outputs to pull back (recommended)

Treat outputs as two layers:

* Human layer (for operators):

  * bottleneck summary (“approvals looping”, “signature stalled”, “handoff delay”)
  * next action and owner
  * reminder/escalation suggestion

* Data layer (for BI/automation):

  * computed execution time
  * stage breakdown (if available)
  * SLA risk flag (at risk of missing close date)
  * normalized bottleneck label (for segmentation)

You can write these back into your CRM/warehouse, but keep ElasticFlow as the system of analysis and operational signals.

---

## 8) Definitions and data quality checks (don’t skip)

The metric becomes misleading if definitions drift.

### Define these explicitly

* What does “ready to sign” mean in your org?

  * after legal approval?
  * after business approval?
  * after final redlines accepted?

* What does “fully signed” mean?

  * all parties executed (not “sent”)
  * include amendments or not?

### Enforce consistency

* time zones: pick one strategy
* IDs: `contract_id` must be stable and unique
* counterparty naming: normalize to avoid duplicates

### Minimum acceptance test (recommended)

* pick 20 contracts and verify the workflow’s computed execution time matches your manual calculation
* verify that “stalled signature” cases are the ones you’d actually chase
* verify that reminder sequence aligns with your process (24h/48h/72h baseline is a good start) 

---

## 9) Practical starting plan (week 1)

ElasticFlow’s guidance is to start with the fastest win: signature tracking + reminders.

A pragmatic week-1 plan:

* Day 1: upload contract records + confirm ID/timestamp consistency
* Day 2: build “single status view” and assign owners
* Day 3: enable reminder sequence (24h, 48h, escalate at 72h) 
* Day 4–7: segment execution time by type/owner/counterparty; pick the dominant bottleneck stage and address it first 
