# Two projects, one decision process

These are fictional examples, not prescribed architectures. The same questions yield different choices. Lines marked “agreed” represent an explicit user confirmation in the example, not something the skill may infer from silence.

## Small local sales import

**Request:** “A three-person team receives a daily CSV of order lines. We need a weekly sales report on a laptop; only about 50,000 rows per month.”

**Dialogue before documentation:** Clarify revenue definition (refunds and tax), line-item grain, who owns the CSV, permitted data use, changes to files, report deadline, and whether historical corrections arrive. Compare a tested Python/SQL import to SQLite or DuckDB with a managed cloud pipeline only if collaboration or scale actually demands it. A daily local import may satisfy the freshness need; layers and distributed processing add little here.

**Stage decision:** The user explicitly agrees on the metric definition, authorized files, local storage/update semantics and daily freshness for this slice. Only then record the approved design in the **sales project's** documentation. If the user asks to implement it, add an idempotent import, synthetic tests for duplicate files/refunds and a clear failure report. No cloud credentials needed.

## Cloud customer-activity product

**Request:** “Several product teams need customer activity within ten minutes of an action, with deletion requests and restricted fields; data arrives from operational DB and SaaS API.”

**Dialogue before documentation:** Determine which consumer truly needs ten minutes, what actions it drives, who owns the two sources and deletion semantics, region/privacy boundary, expected event rate, API limits, reconciliation and incident owner. Compare frequent batches and a change/event stream on end-to-end latency, missed deletes, replay, on-call load and cost. Choose managed or self-hosted technology only after considering team capacity and existing constraints.

**Stage decision:** Each affected stage—outcome, source feasibility, architecture, flow, serving/operations—is confirmed for the scoped slice before its document is written. If the API owner has not granted access, source feasibility stays open and no claim of production readiness is made. A separate implementation request authorizes code; deploying infrastructure, running cloud jobs or processing real personal data requires its own explicit permission.
