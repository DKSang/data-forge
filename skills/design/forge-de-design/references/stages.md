# Decision areas and exit criteria

Use these prompts selectively. The goal is enough shared understanding for the requested slice, not a complete enterprise questionnaire. Lifecycle stages can recur or overlap. Apply security, privacy, ownership, quality, observability, cost, orchestration, and maintainability throughout.

## 1. Outcome and consumer

- Who will use the result, what decision or action changes, and what does success look like? Which dashboard, application, model, or export actually needs the data?
- Which KPI definitions have an owner? What is the business entity and grain (order, order line, customer per day)? Which dimensions, time zones, currencies, and historical comparisons matter?
- What freshness is **required** versus merely preferred? What latency-to-decision, accuracy, query concurrency, retention and availability are acceptable?
- What current baseline and acceptance measure can verify value? Who can confirm that a metric matches the business meaning?

**Ready to confirm:** consumer/action, definitions or an owner to resolve them, practical freshness, and an observable success criterion. When definitions are disputed, record the dispute in chat and seek the right stakeholder; don't declare a canonical KPI unilaterally.

## 2. Sources and feasibility

- Where and how is data generated: OLTP tables, CDC, SaaS APIs, events, logs, files? Who owns and can grant extraction rights? Is the source authoritative for each field?
- What are volume, rate, size, update/delete semantics, keys, schema evolution, late arrivals, duplicates, out-of-order records, historical depth, and source uptime? Can metadata/samples be viewed safely?
- What rate limits, source load budgets, contracts, licensing, locality, compliance and personal-data restrictions apply? What happens when the source changes?
- What is verified versus estimated? What small, safe sample or test would reduce uncertainty? Never copy real sensitive rows into design notes.

**Ready to confirm:** authorized source and extraction boundary, material data risks, ownership/contact, expected change behavior, and a safe feasibility check.

## 3. Architecture and modeling

- Trace intended consumption backward through serving, transformation, ingestion, source generation and storage. Where does persistence actually pay off? What can remain a small local database or file pipeline?
- Model the concepts first, then logical entities/relationships, then physical storage/indexing/partitioning. State grain, keys, temporal semantics, history strategy, and common query paths.
- Compare simplicity, portability, durability, latency, operational burden, TCO, team skill, and growth. What is expensive to reverse? Can a narrower reversible choice meet current demand?
- Consider raw/staging/curated or warehouse/lake/lakehouse **only if** they solve an identified need; avoid assuming all three layers, a centralized platform, or distributed compute.

**Ready to confirm:** data flow, justified storage/compute and model, key invariants, constraints, and what would trigger an architecture review.

## 4. Data flow and authoritative meaning

- Choose batch, micro-batch or streaming from the agreed freshness and action. Specify extraction, cursor/CDC, retries, idempotency, replay/backfill, deletes, event time, schema drift, source throttling and orchestration ownership as relevant.
- Specify standardization, deduplication rule, validation and business transformation with testable examples. Consider ETL versus ELT per pipeline and where a durable transformed result is worth maintaining.
- Distinguish source-of-record for raw facts, curated dataset for a use case, and authoritative KPI/semantic definition. Name steward, lineage and correction process; multiple physical stores can still share governed meaning.

**Ready to confirm:** flow and update semantics, recovery path, quality rules, authoritative definitions, and expected outputs for representative cases.

## 5. Serving and reliable operations

- Who can discover/query/export data, by what interface and at what concurrency/latency? Which self-service guardrails, documentation and permissions are needed?
- Agree freshness/quality SLOs and checks: schema, completeness, uniqueness, referential integrity, volume anomalies and business reconciliations where applicable. Choose monitoring, alert ownership, incident response and tested restoration proportionate to impact.
- Address least privilege, secrets, encryption, retention/deletion, lineage, jurisdiction, infrastructure and human operational cost; assign ownership and a cadence for revisiting decisions.

**Ready to confirm:** consumer contract, access boundary, detectable failure conditions, response owner, recovery expectations, cost guardrails and feedback loop.

## Shortcut for an existing system

If only one stage needs work, inspect upstream/downstream dependencies and confirm the affected decision, not all five stages afresh. If evidence contradicts an earlier agreement, reopen just the affected parts and explain the blast radius.
