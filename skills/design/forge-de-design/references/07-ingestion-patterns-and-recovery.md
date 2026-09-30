# Ingestion patterns and recovery

Ingestion moves data across a source boundary into a destination where it can be stored or processed. It is not the same as **integration**, which combines or relates data from different sources to create a new dataset. A data pipeline can include ingestion, storage, transformation and serving, and may loop across them more than once.

This reference adapts Chapter 7 of *Fundamentals of Data Engineering* into design and delivery questions. Choose patterns from consumer freshness, source behavior and recovery needs—not from fashion or a connector catalog. Use [05-source-generation-and-contracts.md](05-source-generation-and-contracts.md) for source meaning, permissions and producer commitments; [06-storage-and-data-layout.md](06-storage-and-data-layout.md) for target storage guarantees; and [decision-lenses.md](decision-lenses.md) for concise option comparisons.

## 1. Define the ingestion contract before the mechanism

Agree the relevant parts of the contract before choosing a connector, schedule or platform:

- **Purpose and destination:** who will use the data, for what action, and where it must become available. Can the same authorized data product serve more than one use instead of creating redundant copies? Which producer engineering team, business/data owner, downstream consumer and decision-maker need to agree on meaning, change notice and value?
- **Data minimization:** which fields/events are strictly needed for that purpose? Avoid collecting or persisting sensitive attributes that have no justified use; if sensitive identity is required, choose a control (such as tokenization or masking) with the privacy/security owner and an explicit threat model.
- **Freshness and completeness:** what end-to-end delay is acceptable, what amount of late/missing data is tolerable, and how will a consumer know the data is current or incomplete?
- **Source boundary:** what access, APIs/files/change logs, query limits, rate quotas, network paths and source-load budgets are permitted? Which source guarantees have been confirmed with its owner?
- **Payload:** event or state record, schema/data types, nesting, size/rate, format/encoding and relevant metadata. Can the destination deserialize it, and what transformations are allowed in flight?
- **Change and history:** snapshot versus changes, update/delete visibility, ordering, duplicates, retention/replay, and which history must be recoverable.
- **Operations:** who owns the pipeline and source relationship, detects late or failed delivery, handles errors, approves schema changes and coordinates reprocessing?

An assumption is not a producer guarantee. Mark unknowns and seek source-owner confirmation before asserting that a particular extraction mode, deletion signal or freshness target is available.

## 2. Choose along independent ingestion dimensions

Several dimensions are related but not interchangeable. A pipeline can, for example, poll a source on a schedule, pull a snapshot, buffer events asynchronously, then write bounded files for later batch analytics.

| Dimension | Options and meaning | Decision test |
| --- | --- | --- |
| **Boundedness** | A bounded set has a defined boundary (for example a period/file); an unbounded stream continues as new records/events arrive. | What natural or business boundary creates a complete unit? Does the consumer need continuous updates or a complete, reconcilable slice? |
| **Frequency** | Periodic batch, micro-batch (small batches on a short cadence), or continuous/near-real-time processing. | Which action improves if data arrives sooner? Compare end-to-end freshness, source/destination capacity, operating burden and cost. Real-time still has measurable delay. |
| **Batch boundary** | A batch may close by elapsed time or accumulated size/volume. | How does boundary choice trade latency for efficient transfers and manageable file/object sizes? Where does downstream availability begin? |
| **Transfer direction** | A source may **push** to a target; an ingestion process may **pull** from a source; a **poller** may check at intervals and then pull. Boundaries can mix these modes. | Which party can initiate transfer safely? What permissions, quotas, wake-up behavior, retries and source load follow from that choice? |
| **Consumer subscription** | Separately, a queue/stream consumer may pull records and acknowledge/process them, or a broker/provider may push to a listener. This is not the same choice as source-to-ingestion push/pull. | What consumer capacity, backpressure, acknowledgement, receiver durability, retry and failure behavior are required? Verify provider-specific delivery semantics; do not assume push means immediate/safe processing. |
| **Coupling / processing granularity** | A tightly dependent batch chain may wait for the prior step or whole batch; buffered/asynchronous event processing may let stages progress independently and absorb bursts. | What work is blocked by an upstream failure? What is the smallest durable unit for retry/recovery? What buffer/backlog and downstream lag can be supported? |

Chapter 7 uses “synchronous/asynchronous” to contrast tightly coupled batch steps with independently progressing event-oriented stages. Do not silently equate that discussion with an API's synchronous/non-blocking call semantics; describe the actual coupling and failure behavior in the design.

### Batch and continuous processing can coexist

Batch is a convenient way to process a bounded part of ongoing data. It remains suitable for many reporting, historical, training and reconciliation jobs. Streaming can enable actions that require lower latency, but increases state, ordering, buffering, monitoring, failure and cost concerns. A system can process events continuously for a live use case while also retaining or batching them for durable history and later analysis.

Choose true continuous processing only when an identified consumer action justifies its extra complexity over batch or micro-batch. Do not infer that a dashboard labelled “real-time” requires subsecond ingestion; clarify the required freshness and action first.

## 3. Choose a change-capture pattern that matches source semantics

| Pattern | What it provides | Limitations to resolve before implementation |
| --- | --- | --- |
| **Full snapshot** | A complete current-state view for an agreed scope at a point in time; often simple to reason about. | Can consume source/query/network/storage capacity. “Missing from this file means deleted” is valid only if the snapshot is complete for the same confirmed scope and omission semantics. |
| **Differential/incremental read** | Only records considered changed since a prior read; can reduce repeated data movement. | Requires a reliable key, cursor/watermark or change indicator, overlap/gap policy and delete visibility. A `updated_at`-style field may reveal the latest changed row but not every intermediate state. |
| **Log-based/continuous CDC** | A sequence of database changes may preserve event-level history and lower-latency replication. | Requires source/log support, permissions, retention/position management, schema-change handling, capacity and owner coordination. Verify which operations and metadata are actually emitted. |
| **Replication** | Native synchronous replication may keep a like-for-like read replica closely aligned and offload some reads; asynchronous CDC replication may buffer changes and send them to several different targets with some lag. | These modes differ in coupling, consistency/lag, supported targets and source resource use; neither name guarantees a particular recovery or freshness property. Confirm guarantees, monitor replication lag, and coordinate/testing with source operators. |
| **Managed change connector** | A provider or connector may manage scheduling, capture, monitoring or target sync. | Validate source/target support, delete/history semantics, checkpointing, failure alerts, re-sync/backfill, pricing, credential handling and support boundary. “CDC supported” is not an end-to-end completeness guarantee. |
| **Export/file transfer** | The producer controls which data is exported and can isolate direct read access to the operational database. | Requires an agreed file contract, delivery signal, schema/encoding, completeness/correction behavior and secure transfer/retention. Large exports still create source load. |

Distinguish **latest state** from **all changes**: timestamp-based differential capture often returns the latest row version within a window. If consumers need every transaction/event, a retained event history or suitable CDC source may be required. For migration, test representative schemas and types before a full move; bulk data movement and redirecting existing pipeline connections are separate changes with separate cutover/rollback plans.

No pattern by itself solves idempotence, delete propagation, late corrections or a source log gap. Specify these in the contract and validate them against the actual source and destination.

## 4. Design reliability, buffering and recovery explicitly

Ingestion is a critical boundary: a source outage, poor data or stalled connector can leave every downstream stage stale or incomplete. For the slice at hand, define:

- **Load and backpressure:** normal/peak event rate, payload size, bursts, database scans, backlog growth, destination write capacity and how a buffer absorbs spikes. Test restart/catch-up after an outage, not only steady-state throughput.
- **Durability and redundancy:** which accepted data must survive a process/worker/zone failure, for how long, and where is the recoverable copy? Balance failure impact against infrastructure, storage, on-call and recovery cost.
- **Progress and retry:** what identifies the last safely committed unit/position? What is retried after partial failure? Is a repeated batch/event harmless, deduplicated, or potentially double-counted? Keep checkpoints and outputs consistent where required.
- **Late and out-of-order events:** distinguish event time from arrival/processing time; define which late data is accepted, how long the window remains open and what is recalculated or corrected after cutoff.
- **Retention and replay:** how far back can source history or a retained stream be replayed? Does retention cover outage repair, backfill and investigation? A short message TTL or an expired source log can make recovery impossible.
- **Error isolation:** malformed, oversized, unauthorized, expired or unsupported records should have an observable disposition (for example, quarantine/dead-letter with reason and owner) rather than silently blocking or disappearing. Bound retries and reprocessing to avoid poison-message loops or uncontrolled cost.
- **Quality and status:** monitor uptime, end-to-end latency, records/bytes, backlog, failures, schema change and data-quality signals. Make current/stale/partial status visible to consumers; define alert owner and escalation route.

These are design requirements first and implementation checks second. Chapter 7 describes at-least-once delivery and possible ordering/duplicate issues; do not promise “exactly once” end-to-end solely because a product uses that label. The actual result depends on source, checkpoint, transformation and destination semantics.

## 5. Treat payload and schema as part of the contract

Assess the payload's **kind** (table, event, image, document, etc.), **shape** (fields, nesting, dimensions), **size/rate**, **schema/data types**, encoding/serialization and metadata. The receiver must be able to decode and interpret what it receives. A technically successful transfer that leaves data unusable is not a successful ingestion.

When schemas change, distinguish compatible evolution from breaking changes such as removing/renaming fields or changing types. Agree who owns schema versions, how changes are announced, what may be accepted automatically, how incompatible records are surfaced, and which downstream consumers need notice. A schema registry or automated detection can help where supported, but it does not replace producer coordination or compatibility policy.

Use metadata that makes the delivery interpretable—source and schema version, event/ingest times, batch/window, relevant identifiers, and provenance—according to the use case and privacy constraints. As an interim selection check until the Appendix A reference is built, confirm that both source and destination support the chosen format, that types/nesting/encoding survive transfer, and that representative files parse with errors surfaced. Weakly typed text exports such as CSV require explicit delimiter, quoting/escaping, encoding and schema expectations; test malformed rows rather than relying on auto-detection for production. Keep codec catalogs and compression/performance detail in the planned Appendix A reference.

## 6. Select an ingestion approach by modality

| Modality | Useful when | Risks and checks |
| --- | --- | --- |
| **Direct database read** | The source permits queries and a clear snapshot/incremental contract exists. | Check query cost, indexes, concurrency, paging, schema, access and source impact. Never expose an application database directly to the public internet; use an approved private path or tightly scoped intermediary where needed. Parallel readers can speed transfer while increasing production load. |
| **Native export / file drop / secure transfer** | A producer can define and deliver a controlled export, or systems exchange files by agreement. | Secure transport, completeness marker, encoding/schema, naming, late replacement/correction behavior and cadence. SFTP/SCP are transfer protocols, not ingestion strategies by themselves; SSH may also provide a constrained tunnel. Avoid plain FTP for sensitive data. Manual download should be treated as an operational dependency, not invisible automation. |
| **API or webhook** | The provider exposes a supported interface or pushes events to a consumer endpoint. | Provider-specific semantics, pagination/rate limits, authentication, versioning, retries, endpoint durability and error handling. Webhooks require a maintained receiver and buffering/recovery path. |
| **CDC / event queue / retained stream** | Change history or low-latency event flow is required and the source supports it. | Permissions/log retention, order/duplicate behavior, schema evolution, message size/TTL, backlog, replay and on-call burden. |
| **Managed connector** | A maintained connector covers the required source, target and semantics, reducing undifferentiated plumbing. | Validate support scope, update/delete capture, resync/backfill, observability, credentials, provider incident visibility, cost and exit. Use custom code for real gaps, not merely because it is possible. |
| **Shared data access** | A provider exposes controlled read access without creating a physical copy. | Clarify continued access rights, revocation, freshness, tenant/row/column policies, provenance and what consumers may persist or redistribute. This is access/integration, not necessarily physical ingestion. |
| **One-time bulk transfer** | A large migration or initial historical load makes online transfer impractical. | Treat this as a migration event, not the recurring ingestion design; verify chain of custody, integrity checks, cutover, repeat/delta strategy and current cost/availability. |

Any data transmitted across a network should use encrypted transport and an approved, access-controlled path; encryption does not replace authentication, authorization or data minimization. Where applicable, keep intra-cloud traffic on approved private endpoints/network paths and use an approved VPN or dedicated private connection for cloud-to-on-premises transfers. Verify the current organizational policy and actual protocol/service configuration.

Shell/CLI workflows and legacy file-transfer or web interfaces may be adequate for small or constrained sources, but they still need ownership, security, repeatability and failure visibility. Treat SSH as a transport/tunneling protocol, while SFTP/SCP are file-transfer mechanisms that may use SSH; neither alone defines the ingestion contract or error/recovery behavior. Avoid unencrypted transfer for sensitive data. Web scraping is a last-resort source approach: first look for an authorized API or dataset; assess terms, privacy, load/rate limits, and ongoing page-structure maintenance. Do not evade access controls or create harmful traffic.

The source-to-destination path can combine modalities. For example, a source can push into a buffer, consumers can process events asynchronously, and a separate sink can create durable files for batch analytics. Draw the actual handoffs and owners instead of labeling the entire system “streaming.”

## 7. Design and delivery responsibilities

**Design conversation:** confirm the consumer/use case, source contract, freshness/completeness, pattern dimensions, failure/recovery needs, payload/schema policy, security/data-use boundary, whether each sensitive field is necessary, operating owner, cost and evidence. Include the source engineering/operator contact, business/data owner, downstream consumer and decision-maker relevant to the contract. Compare viable options and recommend one with assumptions; receive explicit confirmation for this stage before writing the project's design document.

**Implementation and verification:** after an implementation request, read this design and the approved source contract. Build a bounded slice; default to deterministic synthetic data for representative success and failure cases, and do not persist unnecessary sensitive fields. Use cleansed real data only if the intended purpose is authorized and the de-identification/re-identification risk and approvals have been verified; masking obvious identifiers alone does not establish safety. Include snapshot/delta boundaries, cursor gaps, duplicate/repeated delivery, corrections/deletes, late/out-of-order events, schema changes, bursts/backlog, partial failures, replay limits, DLQ/quarantine and recovery only where relevant to the selected pattern. State exactly what was tested and what remains unverified. If production debugging ever requires sensitive records, treat it as a separately approved exception with at least two authorized approvers, narrow issue-specific scope, minimum necessary access and an expiry; do not normalize production-data use in development.

Remote jobs, production writes, changes to source settings/permissions, external services, destructive actions or meaningful costs require the appropriate separate authorization. A design approval or a safe synthetic test is not that permission.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 7 sections “What Is Data Ingestion?,” “Data Pipelines Defined,” “Key Engineering Considerations for the Ingestion Phase,” “Batch Ingestion Considerations,” “Message and Stream Ingestion Considerations,” “Ways to Ingest Data,” “Whom You’ll Work With,” and “Undercurrents.” The decision prompts and delivery checks are Data Forge applications, not copied templates from the book. Tool/vendor examples, limits, prices and product recommendations in a 2022 text are not current endorsements. No book prose, figures or tables are reproduced. Detailed serialization/compression belongs to Appendix A; source behavior/contracts are in [05-source-generation-and-contracts.md](05-source-generation-and-contracts.md). See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
