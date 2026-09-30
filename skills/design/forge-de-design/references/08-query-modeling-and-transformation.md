# Query, data modeling and transformation

Queries retrieve or change data; models give it coherent structure and business meaning; transformations persist useful results for later consumers. These are related but not identical decisions. This reference adapts Chapter 8 of *Fundamentals of Data Engineering* as a Design + Build guide: agree on semantics and workload first, then implement and test with the selected engine.

Use [03-architecture-principles-and-patterns.md](03-architecture-principles-and-patterns.md) for broader system design, [06-storage-and-data-layout.md](06-storage-and-data-layout.md) for storage guarantees/layout, and [07-ingestion-patterns-and-recovery.md](07-ingestion-patterns-and-recovery.md) for input capture and stream recovery. Use [09-serving-contracts-and-data-products.md](09-serving-contracts-and-data-products.md) for consumer-facing serving patterns and semantic/metrics-layer design.

## 1. Begin with business meaning and query workload

Before choosing a model or writing a complex query, establish:

- **Business concept and decision:** what question/action does the result support, and who owns the terms, measures and rules? Identify the producer/subject-matter stakeholders, downstream consumers and approver who can validate definitions; query or schema authorship alone does not make a rule canonical.
- **Grain:** what does one row or event represent? What stable key(s), time meaning and relationships identify it? State this in words before drawing tables.
- **History and change:** should consumers see current state, every event/version, selected prior values, or a snapshot “as known at” a prior point? How should corrections, late data and deletions affect prior results?
- **Query workload:** lookup versus scan, common filters, joins, aggregations, concurrency, latency/freshness, data volume and cost. Which queries or decisions are important enough to measure?
- **Quality and evidence:** what inputs, definitions, edge cases and reconciliation examples would show the result is correct and useful? Who validates them?

A result can be syntactically valid but semantically wrong—for example, a many-to-many join can multiply revenue, or departments can use incompatible definitions of “active customer.” Have business owners and downstream consumers validate meanings; do not make a data engineer's inferred calculation a canonical KPI without agreement.

## 2. Model in layers of increasing detail

Treat modeling as a deliberate representation of business entities, relationships, rules and workflow, not just choosing column names. A useful progression is:

1. **Conceptual:** entities, relationships and business definitions; use this to expose terms that differ across teams.
2. **Logical:** attributes, keys, grain, relationships, history and data types independent of one specific engine where feasible.
3. **Physical:** tables/files/views, schemas, indexes, partitions, clustering, nested structures and platform configuration for a specific workload.

These levels are a useful vocabulary, not a required method or universal taxonomy. The important contract is what the data represents, its grain and how definitions relate to consumer decisions. See [01-business-value-and-data-maturity.md](01-business-value-and-data-maturity.md) for business ownership and [05-source-generation-and-contracts.md](05-source-generation-and-contracts.md) for source semantics.

### Grain, keys and joins

State the grain of each model explicitly (for example, one row per order line, customer-day or event). Choose keys that actually identify that grain and identify expected cardinality when joining. Validate assumptions such as uniqueness, one-to-many and many-to-many relationships with representative data. Repeated keys on both sides can cause **fan-out/row explosion**: each match multiplies the result, potentially distorting sums and increasing query cost. Never accept plausible-looking totals without reconciling them to known examples or controls.

A finer-grained model can often be aggregated later, but aggregated-away detail cannot generally be recovered. Retain the detail needed for agreed future analysis, traceability and policy; do not interpret “lowest possible grain” as permission to collect or retain unnecessary sensitive data.

## 3. Choose a modeling style by its responsibilities and trade-offs

The book surveys several established approaches. Treat these as options that may be combined, not competing universal standards or product requirements.

| Modeling approach | What it emphasizes | Useful questions and trade-offs |
| --- | --- | --- |
| **Normalized relational model** | Separate entities and relationships to reduce redundancy and update anomalies while preserving integrity. | Are dependencies/keys clear? Are updates consistent? Will consumer joins and query complexity be acceptable? Highly normalized structures can be useful for integration or operational use, while analytics may need additional presentation models. |
| **Dimensional/star model (Kimball-style)** | Facts at an explicit event grain with descriptive dimensions; shared/conformed dimensions can align analysis across multiple facts. | Which event/measures form the fact, and what is the grain? Which attributes and time semantics belong in dimensions? This can be accessible for analytics, but requires shared definitions and deliberate handling of dimension history. |
| **Integrated enterprise model (Inmon-style)** | A warehouse organized around subject areas, integrating detailed data, retaining history and treating stored facts as stable; downstream marts/presentation models can serve particular consumers. The classic description calls it subject-oriented, integrated, nonvolatile and time-variant. | Is a shared, integrated enterprise view valuable and supported by ownership/governance? It can reduce divergent core definitions but may require broader coordination and more modeling/integration work before consumers get their preferred shape. Treat the four terms as the framework's intended characteristics; define actual update/history semantics explicitly. |
| **Data Vault-style historized integration** | Separate business keys, relationships and descriptive context—often described as hubs, links and satellites—while preserving source history and accommodating changing sources. | Are source lineage and historical traceability important? Can the team manage the more complex joins and build consumer models downstream? Keep hashing, load metadata and physical table conventions implementation-specific. |
| **Wide/nested/flexible model** | Denormalized or semi-structured records can reduce joins and adapt to flexible payloads or column-oriented query patterns. | Is the consumer workload suited to this shape? How will repeated definitions, array updates, sparse fields, schema evolution and business logic be governed? Flexibility does not remove modeling or stewardship. |

A company may keep detailed integrated/history data and derive dimensional, wide or consumer-specific models from it. A model may be technically correct yet unhelpful if users cannot discover or interpret it. Conversely, directly querying source data may be a useful short-term discovery path, but clarify metric consistency, production-system impact, access and how the approach would change if repeated use or workload grows.

The chapter's detailed modeling survey is primarily about **batch analytical** data. It describes streaming data modeling as less settled: unbounded events, evolving payloads and late changes make direct use of batch patterns (for example, maintaining a Type 2 historical dimension) nontrivial. Preserve the need for explicit semantics and schema-evolution agreements, but do not present Kimball, Data Vault, wide schemas or windowing as a universal streaming model. See §7 for query/transform behavior; the right model remains a consumer-and-workload decision.

### History is a business requirement

When attributes change, choose history semantics according to the questions consumers need answered:

- **Overwrite/current value:** only the latest value matters; prior values will not be available from this model.
- **Versioned history:** retain changed versions with effective/recorded times so consumers can reconstruct prior states.
- **Selected previous values:** preserve limited prior context where that is sufficient.

Dimensional-model conventions often call these Type 1, Type 2 and Type 3 slowly changing dimensions. They are familiar labels, not required behavior. Agree whether time means when a fact occurred, was observed, was recorded or became valid. Keep source capture of changes distinct from how a transformed model applies/retains them; Chapter 7 owns ingestion CDC mechanics.

## 4. Understand queries before optimizing them

A query can read data, define or change database objects, mutate records, control access or commit/rollback work. These operations have different permissions and side effects; verify the target engine’s supported syntax, transaction and access model before implementation.

A query engine typically validates a request and permissions, builds an execution strategy, optimizes it, and runs it against one or more data sources. The exact optimizer, available indexes/caches, transaction model and plan format vary by engine. Use the target engine's `EXPLAIN`/analysis tools and actual runtime metrics rather than assuming SQL text predicts cost.

Distributed query and transformation engines commonly divide work across workers. Data may be partitioned, exchanged/shuffled by keys, then aggregated or reduced; network transfer, intermediate storage and memory pressure/spill can dominate cost. Classic MapReduce is a historical model, but understanding scan, shuffle and aggregation helps reason about modern query plans too. Inspect the selected engine's actual plan rather than assuming a framework follows one implementation.

For representative, important queries, check:

- records and columns read versus required; partition/index/clustering pruning where available;
- join order/cardinality, many-to-many fan-out and correctness of aggregates;
- data shuffled, spilled, networked or materialized between stages;
- resource use, queueing, concurrency and contention with other queries/writes;
- latency and cost with realistic data size, distribution and cache state.

Use CTEs, views or modular query composition when they clarify logic and maintainability; a CTE is not guaranteed to be faster than a temporary or persisted table. Prejoining or denormalizing may help a repeated workload but creates a maintained result and potentially repeated business logic. Select only needed data when appropriate, while allowing full scans when the workload genuinely requires them. Treat indexes, pruning and caching as engine- and workload-specific tools, not automatic wins.

For mutating queries, clarify transaction boundaries, commit/rollback behavior, concurrent writers/readers, isolation/consistency, and maintenance or retained-version implications. Specific behavior (including snapshot visibility, vacuum/cleanup, cache validity or time travel) must be verified against the selected engine and its current documentation.

## 5. Decide what to persist and where logic belongs

A query computes a result when requested. A **transformation** changes, enriches, models or combines data and persists the result—temporarily or durably—for downstream transformations or queries. Persisting can avoid repeating expensive work and create a reusable contract, but adds storage, refresh, correctness, lineage, ownership and staleness responsibilities.

Choose case by case:

- **View/query at read time:** current logic can remain virtual and easy to change. Views may also provide role-specific column/row access, a deduplicated current-state presentation, or a reusable common join/query pattern. But the underlying work may run each time a user queries it.
- **Materialized/persisted result:** precomputes all or part of a query for reuse, can standardize business logic and reduce repeated work, but needs refresh/update, retention, quality checks and a plan for upstream changes. Some engines may rewrite a matching query to use a materialized result; verify freshness and optimizer behavior rather than assuming it.
- **Federated/virtualized query:** can combine external systems without first copying all data, but performance, source impact, permissions, connectivity and availability depend on those sources at query time. Query pushdown may reduce returned data, but it is not a way to make analytical load on a production source free.
- **Hybrid:** persist expensive or shared portions and leave low-cost, stable or source-local work virtual if that fits freshness and ownership requirements.

Choose **ETL or ELT per pipeline**, not by an organization-wide slogan. Transform-before-load may be appropriate when source data needs filtering or protection before landing, target capabilities are limited, or a controlled schema is required. Load-then-transform may use target compute and retain a useful raw/less-processed copy, but requires a defined downstream transformation plan, access controls and lifecycle; unowned “we'll transform later” data can become unusable clutter. In lake/lakehouse/federated systems, these boundaries can blur.

### Maintain persisted outputs deliberately

If a transformation persists a result, agree its key/grain, ownership, freshness, lineage, backfill and update semantics. Common patterns include:

- **Rebuild/truncate-and-reload:** simple and reproducible for bounded, affordable datasets; can be costly or disrupt readers at scale.
- **Append/insert-only versions:** preserve history and support as-of/current-state views; consumers or downstream models must resolve versions and deletions correctly.
- **Upsert/merge:** maintain current state or corrections by matching on keys, but depend on uniqueness assumptions, transaction/update support and efficient store behavior. File/column-oriented stores may rewrite larger units for small changes.
- **Delete semantics:** a hard delete physically removes a record; a soft delete marks it unavailable to ordinary queries; an insert-only deletion/tombstone appends a version that declares the key deleted. Choose according to audit/history and policy. A tombstone or filtered soft delete does not by itself satisfy a legal or contractual erasure requirement; confirm deletion scope and verify physical/derived copies through lineage and the storage policy.

Choose based on source semantics, target behavior, history and deletion requirements, data volume, update frequency and recovery cost. Chapter 7's capture mechanism tells what changes arrive; this section decides how the **derived output** reflects them. Test duplicate reruns, corrections, deletes, key conflicts, partial failures and the ability to rebuild or restore as relevant.

## 6. Choose SQL, code or a combination

SQL is declarative and often concise for filtering, joins, aggregation and common data transformations. A general-purpose language or native engine API can be clearer for complex algorithms, specialized libraries, custom parsing or reusable logic. The meaningful question is not “SQL or code forever?” but which expression the team can understand, test, operate and optimize for this task.

- Prefer the target engine's native/declarative operations when they make the logic clearer and allow the engine to optimize it.
- Use code or a well-maintained library when SQL becomes awkward, opaque or a poor fit; isolate reusable business logic rather than duplicating it across scripts.
- Combining SQL and code can be effective if data/ownership boundaries, serialization costs and failure handling are explicit.
- Review query plans and representative performance before custom-tuning; custom code transfers optimization responsibility to its maintainers.
- Version control, review, automated tests and repeatable deployment apply to transformation definitions too, whether written as SQL, code or generated by a visual tool. Inspect generated logic when it affects correctness, privacy or performance.

Product names and engine APIs in the book are historical examples. Confirm support, performance, security and deployment behavior for the current selected system.

## 7. Queries and transformations over streaming data

Chapter 8 describes several query patterns for continuously changing data. A **fast-follower** pattern queries an analytics-oriented copy updated through CDC, accepting some lag while reducing analytical load on the production database. A **retained-event-log** pattern keeps events queryable across historical ranges (often associated with Kappa-style architectures). A streaming engine may also run continuously triggered or windowed computations. These are distinct ways to serve queries from changing data, not a single universal definition of “streaming query.”

Distinguish querying an evolving/current view from a **streaming transformation**, which enriches or reshapes events and emits output for downstream use. The boundary can blur when a windowed query publishes a derived stream.

When a use case truly requires continuous or low-latency processing, agree event time and key, window/session definition, trigger/output cadence, allowed lateness/watermark behavior, retained state, late corrections, schema evolution, replay, resource cost and failure recovery. Stream enrichment may join an event with a reference dataset; stream-to-stream joins may need buffers to handle different arrival delays, trading completeness against state/storage cost. Micro-batch may satisfy a freshness target more simply than processing every event individually; there is no universal winner. The chapter notes that a broadly accepted streaming data-modeling approach remains unsettled; do not assume batch models or SCD patterns transfer unchanged.

Use the ingestion reference for capture, ordering/duplicate/replay and event-retention requirements. Here, focus on query/transform semantics and the output contract. Verify windows, boundaries, late/out-of-order cases and state cleanup on the chosen runtime; do not infer exact behavior from generic pattern names.

## 8. Business logic, derived data and validation

Transformations often encode business rules and derived measures. A rule can be subtle and may change as teams refine definitions; copying it into many queries/pipelines risks drift, while a single derived table also needs ownership and update discipline. Record the authoritative definition owner, version/lineage and consumers; Chapter 9 covers consumer-facing semantic/metrics-layer serving in more depth.

For each important transform, agree representative examples and invariants: expected grain/keys, row counts or control totals, null/validity rules, uniqueness, join cardinality, date/time boundaries, and how known exceptions are handled. Reconcile derived outputs to trusted inputs or business-approved examples. When source meaning or rules change, identify which persisted outputs and downstream products need recomputation or notice.

Build tests with synthetic fixtures for ordinary and edge cases. For persisted transforms, verify schema and shape, business metrics, duplicate/update/delete handling, rerun behavior and rollback/recovery proportionate to impact. Data tests should check both source inputs and outputs: a query can run successfully while yielding an incorrect metric. Coordinate rules and schema changes with source engineers and downstream analysts/scientists/product owners. Apply least privilege to derived outputs, maintain lineage so changes and deletion requests can be traced, and link definitions/owners to discoverable metadata. For broader security, privacy and DataOps principles, see [02-lifecycle-and-undercurrents.md](02-lifecycle-and-undercurrents.md) and consult the optional `forge-de-operate` skill when installed; this reference focuses on transformation-specific design and tests.

## 9. Design and build handoff

**Design:** agree business definitions, grain, model choice and alternatives, query workload, persistence/update semantics, transformation location/tool rationale, quality evidence, privacy/access boundaries, cost and owner. Keep architecture choices and product-specific implementation details distinct. Ask the user to confirm this stage before writing its project design document.

**Build:** implement the approved contract and conventions; inspect the actual query plan or runtime evidence where relevant; test grain, keys, join fan-out, data rules, schema evolution, update/rebuild behavior, incremental bounds and recovery using representative synthetic data. Report what ran, what passed/failed and what remains unverified. External writes, production jobs, sensitive-data use or material spend retain their separate approval gates.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 8 sections “Queries,” “The Life of a Query,” “The Query Optimizer,” “Improving Query Performance,” “Data Modeling,” “Conceptual, Logical, and Physical Data Models,” “Normalization,” “Techniques for Modeling Batch Analytical Data,” “Modeling Streaming Data,” “Transformations,” “ETL, ELT, and Data Pipelines,” “SQL and Code-Based Transformation Tools,” “Update Patterns,” “Business Logic and Derived Data,” “MapReduce,” “After MapReduce,” “Materialized Views, Federation, and Query Virtualization,” “Streaming Transformations and Processing,” “Whom You’ll Work With,” and “Undercurrents.” Named frameworks and patterns are optional lenses; performance/product claims and forecasts in the 2022 text are not current guarantees. The Design/Build decision and validation prompts are Data Forge adaptations; explicit approval before project documentation or production-side effects is also this suite's operating policy, not a process prescribed in Chapter 8. No book prose, distinctive tables or figures are reproduced. See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
