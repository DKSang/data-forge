# Data sources, ownership and contracts

Assess a data source before choosing how to ingest it. This reference focuses on how information is created, what the source actually guarantees, who owns it, and what needs to be agreed at the producer boundary. It adapts Chapter 5 of *Fundamentals of Data Engineering*; connector implementation, polling/CDC mechanics, retries, replay and backfill belong in the later ingestion reference.

Use [01-business-value-and-data-maturity.md](01-business-value-and-data-maturity.md) to begin from the intended consumer and [02-lifecycle-and-undercurrents.md](02-lifecycle-and-undercurrents.md) for the cross-stage lifecycle. Source discovery does not itself authorize access or establish that a system is the business system of record.

## 1. Establish the source and its owners

For each field, entity or event needed by the use case, distinguish:

- **Source system:** where the data is first created or captured. It may be a business application, transaction database, device, process/log, human-maintained file, API, partner or third-party dataset.
- **Data owner/steward:** who can confirm the business meaning, permission to use it, acceptable quality and retention/change expectations.
- **System operator/provider:** who can explain availability, schema/deploy changes, access method, capacity limits and incident communication. This may be different from the business data owner.
- **System of record:** which source is authoritative for a particular fact or state. Several sources may be authoritative for different fields; verify the meaning with the domain owner instead of inferring it from a table name or connector.
- **Consumer and permitted purpose:** who will use the data and whether the source owner has authorized the intended use, especially for personal, sensitive, licensed or shared data.

Ask how the information is generated—not just where it can be downloaded. Is it native digital data or a digital copy of a human process? Does an application emit business events or only a current state? Which upstream systems and human steps can introduce delays, transformations or errors? Preserve these distinctions because two tables with similar columns can have different authority and change semantics.

## 2. Fingerprint source behavior, not product labels

| Source pattern | What to understand | Questions for the owner or source documentation |
| --- | --- | --- |
| **Application/transaction database** | Often optimized for operational record reads/writes and concurrency; it may expose current mutable state, historical versions, or both. Analytical scans can contend with production work. | What do key records mean? Which operations insert, update or delete? What transaction and consistency guarantees actually apply? What reads, rates or windows can production tolerate? Is a replica/export available? |
| **Analytical/OLAP system** | Often optimized for large analytical scans and aggregation rather than frequent single-record transactions. It may itself be an upstream source, for example for ML training or reverse-ETL workflows. | Who owns the business definitions and refresh state? What scan/query limits, freshness and retention apply? Will reads contend with other consumers or incur material cost, and can the lineage to operational sources be established? |
| **Files and human-generated data** | Files can be a first-class exchange mechanism, from structured exports to semi-structured documents. A person, scheduled job or another platform may create them. | Who produces the file; what makes it complete and current; what version/schema, encoding, naming and cadence are expected; how are corrections, duplicates and missing deliveries communicated? |
| **APIs, SaaS and third-party/shared data** | The interface may expose application-specific business semantics, access conditions and operational limits. “Has an API” does not mean the fields or contract are self-explanatory. | Who owns the API and grants access? What data may be used/shared, for which purposes and where? What is documented about schema/version changes, availability, quotas and support? Does the provider push updates, expose queryable endpoints, or offer files? |
| **Logs, messages, event and time-series sources** | A log may record actors, actions and timestamps; a queue may discard a message after delivery, while a retained stream/log may support replay. Device and event sources can be delayed, malformed or out of order. | What is recorded, at what resolution and with which identifiers/timestamps? Is this an event log or only a snapshot? How long is it retained? What ordering, replay and completeness does the owner actually guarantee? |

These are broad source shapes, not a database taxonomy or recommendations. Relational and nonrelational systems vary in schema enforcement, indexing, transactions, consistency, query behavior and update semantics. Even within NoSQL, key-value, document, wide-column, graph, search and time-series systems are not interchangeable. Use the actual system's documentation and a safe sample/profile rather than assuming behavior from a category.

## 3. Verify semantics, consistency and change history

For a stateful source, ask what a record means **now** and whether you need to reconstruct what it meant **before**. Sources commonly expose some combination of:

- a current-state record that can be overwritten;
- a versioned/insert-only history;
- periodic snapshots;
- change events/logs (CDC or application events);
- a mixture, where some mutations or deletions are not visible to consumers.

Do not treat CRUD as a promise that every action is externally observable. Verify whether updates, hard deletes, soft deletes, corrections and intermediate states are retained or emitted, how long any log/history remains available, and what happens during schema migrations. A timestamp column does not necessarily contain every intermediate change. A CDC feature name alone does not prove gap-free history, a particular retention window or end-to-end delivery guarantees.

For data correctness and feasibility, record source-specific facts about:

- generation rate, size/volume, peak patterns and delivery cadence;
- keys, grain, schema, nested/variable fields, joins and required upstream dependencies;
- nulls, malformed values, duplicates, missing or late data and consistency behavior;
- event time versus the time the source publishes, the consumer receives, or processing occurs;
- source read/scan impact, API quota, payload/query limits and response when a dependency is unavailable;
- retention, historical backfill availability, deletion duties, permissions, residency and external data-use terms.

Treat these as observations and owner-verified expectations, not as guarantees until the appropriate source contact or current technical documentation confirms them. The source evaluation informs the later ingestion choice; it does not prescribe batch, streaming, pull, push, snapshot or CDC by itself.

## 4. Agree a source contract with the producer

A source contract is an agreement between the source owner/provider and the downstream team about what is being delivered and how the parties coordinate. Keep it small enough to maintain, but explicit enough to prevent silent breaks. For this project, agree the relevant parts of:

1. **Scope and meaning:** entities/events, fields, grain, business definitions, keys and known limitations.
2. **Authorization:** approved purpose, consumers, columns/sensitivity, access method and any sharing/residency/retention constraints. Do not put secrets in the contract.
3. **Schema and change:** current schema/version, compatibility expectations, how breaking/ordinary changes are announced, who tests or coordinates them, and where the canonical description lives.
4. **Delivery expectations:** expected availability/freshness, completeness or quality thresholds, known source-side latency, any agreed extraction mode/cadence and source load limits. Leave the actual ingestion implementation to its own design.
5. **Ownership and support:** data steward, system/API operator, consumer/ingestion contact, escalation route, incident/change notification and review process.
6. **Evidence and exceptions:** how the source contract was verified, assumptions awaiting confirmation, known outages/quirks and how exceptions will be communicated.

A formal data contract product, service-level agreement (SLA) or fixed schema policy is not necessary for every small source. Choose a lightweight written agreement when it materially reduces ambiguity or breakage; distinguish a target or expectation (SLO) from a commitment the source owner has actually accepted (SLA). Chapter 5 uses both formal and informal collaboration examples—do not represent an unapproved expectation as a producer promise.

## 5. Keep the source assessment and ingestion design distinct

The output of this reference is a source profile and agreed producer expectations, not a connector plan. Hand confirmed source facts to the ingestion design:

- Source can expose only snapshots, with a read window → consider how freshness, full/delta scope and source load constrain the future ingestion choice.
- Source retains a log for a limited time → account for that window in the later capture/recovery plan.
- API quotas or operational DB load are tight → include them as non-negotiable constraints for a later cadence/connector comparison.
- Owner cannot confirm delete, key or schema semantics → mark that as an open decision; do not claim downstream history or completeness is guaranteed.

When designing ingestion, the later reference will compare implementation patterns such as snapshot/differential reads, push/pull, batch/micro-batch/stream and CDC. This source reference deliberately stops before choosing or implementing one.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 5 sections “Sources of Data: How Is Data Created?,” “Files and Unstructured Data,” “Application Databases (OLTP Systems),” “Logs,” “Messages and Streams,” “Types of Time,” “Databases,” “APIs,” “Data Sharing,” “Third-Party Data Sources,” “Whom You’ll Work With,” and relevant “Data Management”/undercurrent discussions. The source-profile and contract outline are Data Forge aids; the source shapes and examples are paraphrased. Database, API and product examples are illustrative and time-bound. No book prose, tables or figures are reproduced. See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
