# Storage and data layout

Storage is the set of places where data persists while it is created, moved, transformed, served, retained or deleted. A dataset can pass through several stores; “the storage choice” is often a small set of related choices rather than one destination. Start with how consumers and processes will write and retrieve data, then compare durability, consistency, access, lifecycle, operations and cost.

This reference adapts Chapter 6 of *Fundamentals of Data Engineering* in our own original wording. It gives a decision framework, not a provider catalog. Use [03-architecture-principles-and-patterns.md](03-architecture-principles-and-patterns.md) for architecture topology and [04-technology-selection.md](04-technology-selection.md) for comparing current products. Detailed file/serialization/codec choices belong in the later Appendix A reference; ingestion mechanics belong in Chapter 7.

## 1. Start from the data's use and access pattern

For each important dataset or intermediate, identify:

- **Purpose and consumers:** which process, analyst, application or model reads/writes it, and what decision or action relies on it?
- **Read pattern:** individual-key lookups, repeated small reads, range filters, large analytical scans, joins/aggregations, or replay through a time range?
- **Write/update pattern:** append-only events, bulk loads, frequent small changes, upserts/deletes, mutable state, or immutable retained objects?
- **Workload envelope:** size, growth, throughput, concurrency, latency, availability and recovery needs—including peaks, not just averages.
- **Data lifecycle:** how long the data remains valuable, how often it will be accessed over time, whether it can be regenerated, and whether policy requires retention, archival or deletion.
- **Boundaries:** sensitivity, access level, tenant isolation, data locality, producer/consumer proximity and who operates or restores the store.

Do not choose a store because it is fashionable or because one kind of access is fast. The right result may be a small local database plus files, an analytical warehouse, an object store with compute, a managed platform, or more than one tier. Use only the components justified by the workload.

## 2. Separate storage layers of the decision

It helps to distinguish three levels:

1. **Physical ingredients:** memory, disks/flash, CPU and network. They shape latency, throughput, capacity, volatility and failure behavior. Their exact prices and performance change; verify current hardware/service specifications rather than reusing book-era numbers.
2. **Storage systems:** local filesystems, network filesystems, block devices, object stores, databases, warehouses, caches or retained message logs. Each exposes particular read/write, consistency, durability and operational semantics.
3. **Storage abstractions:** the way a system organizes and manages data for work, such as a warehouse, lake, lakehouse, catalog or data platform. An abstraction may combine several storage systems and query/metadata capabilities.

At the physical level, magnetic disks have moving parts and generally favor sequential throughput and low cost per capacity; random lookups pay seek/rotation latency. Flash/SSD avoids mechanical seek and often improves random access/IOPS, usually with different capacity economics. RAM is much faster working memory but volatile; CPU cache is smaller and closer to the processor. Distributed storage also pays for CPU and network work, so parallelism can raise throughput while adding coordination and transfer costs. These are qualitative tendencies, not current product specifications—benchmark the actual workload.

A single product can span more than one level, and different vendors may implement the same abstraction differently. Ask what behavior the workload needs, then verify the actual implementation's guarantees.

## 3. Match storage system behavior to the workload

| Storage system shape | Often useful for | Properties to verify |
| --- | --- | --- |
| **Memory/cache** | Temporary working data or frequently reused results where very low latency or reduced load on a backend matters. CPU cache is very small/close to the processor; RAM is larger but volatile and must not be treated as durable without explicit persistence/replication guarantees. | Is it volatile? What is the eviction/TTL behavior? What happens on restart, node loss or stale cache? Which durable copy is authoritative? |
| **Local filesystem or attached disk** | A local project, a single host, temporary processing, or an application needing filesystem semantics and low-latency reads. | Is data tied to a machine/lifecycle? What are consistency, locking, backup, concurrent-write and disk-failure behaviors? Can a local processing disk be disposable because inputs/outputs persist elsewhere? |
| **Network filesystem / shared file storage** | Multiple processes or machines needing filesystem-style shared access. | What consistency/locking semantics apply across clients? What network latency, throughput, availability, access control and shared-failure domains exist? |
| **Block storage** | Systems, often databases or VMs, that require a device-like volume with random reads/writes and control over filesystem/database layout. | How is the volume attached and persisted across machine replacement? Which performance, replication, snapshot, zone and recovery properties are guaranteed? A volume attached to a running host may differ from ephemeral local storage. |
| **Object storage** | Durable, large-scale storage of file-like objects, often for data exchange, analytics, archival, or decoupled storage/compute. | Objects are commonly addressed by keys and updated by writing a replacement rather than editing bytes in place. Verify append/update, consistency, versioning, listing/prefix costs, range reads, request limits, lifecycle, deletion and restore semantics for the actual service. Large parallel reads/writes may fit better than frequent tiny transactional updates. |
| **Transactional database** | Mutable operational records, indexed lookups, concurrent transactions and low-latency state access. | Transaction/consistency guarantees, indexes, write contention, query impact, capacity, backup/restore, schema migration, replication and serving limits. Protect production workloads from analytical scans. |
| **Analytical/column-oriented system** | Large scans, aggregations and analytical queries over selected columns and partitions. | Query pruning, indexing/clustering/partition behavior, joins, concurrency, update/delete support, load pattern, scan/capacity cost and consistency. Do not assume it is an efficient transactional store for many small random updates. |
| **Retained message/event log** | Event ingestion, consumers at different speeds, retention and replay when supported. A stream-to-batch arrangement may fan events to lower-latency consumers while also persisting them for long-term queries and historical processing. | Retention, ordering/partitioning, replay window, throughput, state and source-of-truth boundary. A queue that removes a message after delivery is not equivalent to a retained/replayable log. Some systems expose a unified view over recent buffered data and persisted analytical data; verify its consistency and query behavior rather than assuming the handoff is seamless. Delivery/acknowledgement guarantees and replay implementation belong in Chapter 7. |

These are broad patterns, not product guarantees. Distributed storage can add throughput, capacity, redundancy and availability, but also replication delay, coordination, network failure modes and operational complexity. For either single-host or distributed storage, examine the failure boundary and test recovery appropriate to the data's value.

### Consistency, durability and availability are different

- **Consistency:** what a reader may observe after writes, across replicas or queries. Verify whether a read sees the latest write, an older value temporarily, or a defined snapshot; identify where configuration or query-level choices alter behavior.
- **Durability:** how likely acknowledged data is to survive failures over the required period. Replication, backups, snapshots and versioning provide different protections; replication alone does not necessarily protect against accidental or malicious deletion.
- **Availability:** whether the service/data can be reached when needed. Geographic replication may improve resilience but can affect latency, cost, sovereignty and consistency.

Do not collapse these into a vendor “nines” claim. Translate them into the consumer's tolerable stale reads, data loss, outage and restoration time. Name the failure modes that matter and how restoration will be verified.

For mutable or concurrent workloads, also define the transaction boundary: which records/operations must commit atomically, what isolation or conflict behavior applies to simultaneous writers, and whether a transaction spans multiple tables or systems. Do not assume a guarantee across separate stores or services; verify the actual transaction model and recovery behavior.

## 4. Choose a storage abstraction conditionally

| Abstraction | What it often provides | Decision questions and trade-offs |
| --- | --- | --- |
| **Warehouse** | Managed analytical tables, query execution, schema/metadata and controls suited to structured reporting or analytics. | Does its data model, update behavior, concurrency, integration and economics fit the consumers? Where do transformations run? Can unstructured/raw data or portability requirements be met without awkward duplication? |
| **Data lake / object-oriented data store** | A place to retain diverse files/objects, potentially in open formats and consumed by multiple compute engines. | Who owns schemas/catalog, access, data quality, update/delete, lineage and retention? Without discoverability and stewardship, flexible storage can become difficult to trust or use. Evaluate total operating cost—not only storage price. |
| **Lakehouse** | A management/table layer over file/object storage that can add schema, transaction, history, update/delete or rollback capabilities while retaining file access. | Which capabilities are truly supported across the chosen engines? How do metadata, atomicity, concurrency, deletion, version retention and recovery work? Does openness reduce interchange cost, or does the platform create new lock-in/operational responsibilities? Verify current formats and support. |
| **Data platform** | An integrated set of storage, query, metadata, sharing and related services. | Which components solve requirements the team actually has? Can non-included tools interoperate? What are the boundaries for data ownership, cost, security, export and exit? Integration convenience may trade off against platform dependency. |
| **Catalog/metadata layer** | A way to make datasets, definitions, owners, lineage, relationships and access discoverable across systems. | Which metadata is automatically captured versus curated by people? Who maintains correctness? Does the catalog integrate with the real producers/consumers and support business descriptions as well as technical lineage? A catalog is a management capability, not a replacement for clear ownership or quality. |

These categories can overlap or be assembled differently. “Lake,” “warehouse,” “lakehouse” and “platform” are not mutually exclusive labels with identical capabilities across providers. Compare the needed semantics, lifecycle and interfaces, then test the actual system. Avoid a data swamp by ensuring retained data has owners, descriptions, access policy and a known consumer or explicit retention purpose.

## 5. Plan layout, metadata and compute placement

**Row versus column access.** Row-oriented layouts commonly suit individual-record access and updates; column-oriented layouts can reduce reads for large analytical scans and may compress similar values efficiently. Modern systems mix techniques, and some formats/engines support both transactional and analytical behavior. Confirm the actual query plans and update patterns instead of assuming a label predicts performance.

**Indexes, partitions and clustering.** Indexes or sorted/clustered layouts can speed selected lookups or reduce scans, but cost storage, write/update work and maintenance. Partitioning may help when consumers commonly filter by a bounded key such as time; excessive or high-cardinality partitions can increase metadata and planning overhead. Base layout on query evidence, and revisit it as the workload changes.

**Schema and metadata.** Schema-on-write enforces structure before data is accepted, which can improve consistency and downstream usability but requires handling valid evolution. Schema-on-read allows more flexible writes, while placing more responsibility on readers and governance. Schema can describe structured records, nested/semi-structured data and other useful properties; it is not limited to relational columns. Record business meaning, technical schema/version, owner, lineage, operational freshness and reference-data dependencies as relevant.

**Compute-storage placement.** Keeping compute close to storage can reduce data movement or speed repeated/local work; separating them can let a team scale or release compute independently while data persists. Hybrid designs commonly use durable shared storage with local or memory caches/intermediate storage for processing. Compare network bandwidth/latency, serialization, caching, egress, concurrency, ephemeral-compute startup and durability. Cache/temporary data must not silently become the only durable copy.

**Sharing and tenants.** Single-tenant storage can simplify isolation and allow independent schemas, but multiplies resources, administration and cross-tenant analytics work. Multi-tenant storage can simplify common views and resource use, but requires rigor around row/column/cell access, noisy-neighbor behavior, shared schema changes and blast radius. Choose isolation from explicit privacy, regulatory, customer and operations requirements; verify enforcement rather than relying on conventions in application code alone. Check whether tenant lifecycle tasks—provisioning/deprovisioning, export, retention/deletion, and restore—can be performed per tenant with the required isolation and auditability.

## 6. Manage data access temperature and retention

Storage lifecycle follows both usefulness and access frequency. A practical profile distinguishes:

- **Frequently accessed:** quick reads matter; storage and query capacity may cost more, but repeated retrieval may be cheaper/faster.
- **Occasionally accessed:** a lower-cost tier may fit if retrieval delays/fees and cache behavior are acceptable.
- **Rarely accessed/archive:** lower storage cost may come with slower retrieval, minimum-duration charges, restore steps or reduced resilience; test an actual restore before relying on it.

Do not copy named vendor-tier prices or retrieval times from a 2022 book. Check current service terms. Consider lifecycle policies, caches, spillover and versioning with attention to access patterns and failure recovery.

For each copy, decide **why it is retained and until when**:

- business value and whether the source can recreate it;
- consumer history, audit, backup/restore or reproducibility needs;
- sensitivity, jurisdiction, contractual and current legal retention/deletion obligations;
- versioning/snapshots, deletion propagation, catalog/lineage, and how to prove deletion or restoration;
- storage, retrieval, replication and operational cost over its full lifecycle.

“Keep everything in case it is useful” can create expense, risk, clutter and unmanageable deletion obligations. Conversely, deleting data that is expensive or impossible to recreate can harm consumers or recovery. Assign an owner and approved policy; do not infer retention duration or legal requirements from the book.

## 7. Storage operations and cross-team responsibilities

Storage decisions cross team boundaries. Clarify whether data engineering can provision/change the store or depends on infrastructure, security, cloud/platform or database teams; agree the request/review path, accountable operator and incident handoff. A managed service shifts some infrastructure work to a provider, but it does not remove responsibility for configuring data access, classifying data, retention, usage, recovery or monitoring the service contract.

Use proportionate operational controls:

- **Security:** grant least privilege to users and workloads; protect data in transit and at rest; use row/column/cell controls where needed and supported; audit access to sensitive/shared data. Verify actual isolation and provider/customer responsibilities.
- **Data management:** connect schemas and business definitions to owners, metadata/catalog, lineage, data quality, version history, sharing, retention and deletion. Versioning can support recovery or reproducibility, but it also multiplies stored data and does not replace backup or deletion policy.
- **DataOps and monitoring:** observe both the storage service (capacity, latency, errors, availability, cost and access patterns) and the data (freshness, volume and logical anomalies). Define alert owners and response/recovery steps. Monitoring rules and catalog/lineage are useful only if someone maintains and acts on them.
- **Orchestration and software engineering:** represent storage dependencies in repeatable pipelines; use infrastructure/configuration as code where it reduces drift and enables review; make temporary compute/cache behavior explicit; test backup, restore, schema change and deletion behavior proportionate to impact.
- **FinOps:** track material spend or unit-cost signals and who responds to spikes, retrieval/egress growth or storage growth. Avoid adding an expensive monitoring/automation stack for a low-risk local workload without value evidence.

This is a storage-specific handoff prompt, not a complete security, governance or DataOps playbook. For broader cross-cutting principles, see [02-lifecycle-and-undercurrents.md](02-lifecycle-and-undercurrents.md); verify current organizational policy and provider capabilities.

## 8. Review and verify a storage proposal

For a concrete workload, present only the relevant questions and candidate approaches:

1. What must be persisted, for whom, and how will it be read, written, updated and deleted?
2. Which latency, throughput, concurrency, consistency, durability, availability and recovery properties are required—and which are assumptions?
3. What storage boundary is authoritative, and which copies are caches, intermediates, replicas or archives?
4. How will schema, metadata, access, tenant isolation, sharing, lineage, retention and deletion work?
5. What are the financial and operational costs, including network movement, retrieval, backup, monitoring and support?
6. What representative query/load/recovery test would prove the choice is adequate? What result would make the team reconsider?

When evidence is missing, run a bounded synthetic or non-production test with representative read/write patterns. Verify guarantees from current primary documentation and an actual restore/query, where safe. Do not run a production write, delete data, incur material cloud cost or expose sensitive data without the required approval. State what the test did not prove.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 6 sections “Raw Ingredients of Data Storage,” “Data Storage Systems,” “Data Engineering Storage Abstractions,” “Big Ideas and Trends in Storage,” “Data Storage Lifecycle and Data Retention,” “Single-Tenant Versus Multitenant Storage,” “Whom You’ll Work With,” and “Undercurrents.” This reference generalizes the chapter’s workload-led storage questions and behaviors; hardware figures, provider examples, pricing, availability figures and product feature claims are time-bound and intentionally not repeated as current guidance. Serialization/compression format detail is reserved for Appendix A; connector and replay mechanics for Chapter 7; query/model decisions for Chapter 8. No book prose, distinctive tables or figures are reproduced. See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
