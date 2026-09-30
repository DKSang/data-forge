# Serving contracts and data products

Serving is the point where prepared data becomes usable by a person, model, application, or operational process. Design backward from the consumer and the decision or action—not from a preferred platform or a generic “analytics” label. This reference adapts Chapter 9 of *Fundamentals of Data Engineering* into consumer-facing design questions for analytics, machine learning (ML), and reverse ETL.

Use [01-business-value-and-data-maturity.md](01-business-value-and-data-maturity.md) to clarify business value and consumers, [06-storage-and-data-layout.md](06-storage-and-data-layout.md) for storage properties, [07-ingestion-patterns-and-recovery.md](07-ingestion-patterns-and-recovery.md) for inbound acquisition and recovery, and [08-query-modeling-and-transformation.md](08-query-modeling-and-transformation.md) for data meaning, models, transformations, query execution, and persisted-output semantics. Serving concerns the consumer-facing contract and interface over prepared data; it does not prescribe a fixed architecture or require every output to be managed as a data product.

## 1. Start with the consumer and the action

“Serve analytics,” “serve ML,” and “make data available” do not yet define a design. Establish the consumer's capability and intended use before choosing an interface:

- **Consumer and owner:** Who uses the data or receives the output? Who owns the product or interface, and who can confirm the business meaning?
- **Decision or action:** Is the consumer exploring a question, monitoring a defined measure, generating a prediction, or taking an operational action? What changes if the data arrives sooner or is unavailable?
- **Required trust:** Which definitions, quality checks, lineage, freshness and completeness evidence must be visible for the consumer to use the result safely?
- **Workload:** What are the expected data volume, query or request shape, concurrency, latency, availability, and periods of peak use? Which are measured needs versus preferences?
- **Access capability:** Can consumers query a model or engine, or do they need a file, curated interface, API, or managed feed? What technical support can they provide themselves?
- **Constraints:** Which use, fields, destinations, retention, access boundaries, cost limits, or source/target dependencies constrain serving?
- **Acceptance:** What representative question, decision, or action will show that the interface is useful and its data is fit for that purpose?

Ask about freshness separately from display or query speed. If a consumer says “each morning,” clarify (1) what source or business cutoff the result must include (for example, complete through yesterday or through a stated time) and (2) whether it must be available by a particular time or morning is simply when they usually view it. Freshness is how current the data is; latency is how quickly a query, request, or prediction returns. A user may need fresh data but tolerate a slower query, or need a fast response from a periodically refreshed dataset. Record the end-to-end freshness need separately from query/request latency; do not assume an 8 a.m. viewing habit requires an 8 a.m. refresh.

## 2. Define a serving contract proportionate to the use

A consumer contract makes clear what is being offered and what consumers may rely on. A small, local use case may need only a short note; a widely reused or action-driving interface may need explicit ownership and service expectations. Do not turn this list into a mandatory enterprise checklist.

| Contract element | Questions to resolve |
| --- | --- |
| Purpose and audience | What use is supported, and which consumers or downstream systems are in scope? What uses are not supported? |
| Ownership and meaning | Who owns the interface and who owns its business definitions? What does each important field or measure mean? |
| Interface and shape | Is the consumer receiving a file, queryable dataset, stream, semantic/metrics layer, or application interface? What schema, keys, grain, or response shape does the consumer need? Use Chapter 8 for the underlying model and transformation decisions. |
| Freshness and completeness | How current and complete should a successful delivery or response be? How will consumers recognize late, partial, or stale data? |
| Quality and trust evidence | Which validation, reconciliation, provenance, lineage, or known limitation is relevant to this use? Where can the consumer inspect the status? |
| Service expectations | What availability, latency, concurrency, support hours, or response expectations matter? Agree measurable targets where the impact justifies them; do not invent a universal SLO. |
| Access and use | Which people or systems may use the data, for what approved purpose, and at what scope? Which fields or records are unnecessary for that purpose? |
| Change and support | Who communicates schema, meaning, or interface changes? What compatibility, deprecation, correction, and consumer-notification process is needed? |
| Lifecycle and cost | Who reviews continued use, retention, serving cost, and consumer adoption? When should the interface be changed or retired? |

Make assumptions explicit. A target freshness is not guaranteed merely because an upstream pipeline is scheduled at that interval; verify the end-to-end path and the owners of each handoff. See Chapter 7 for acquisition and recovery behavior, and Chapter 8 for the semantics and maintenance of the prepared output.

### Data products are an optional packaging model

A **data product** is a useful way to organize a reusable, owned data interface around a consumer need. It can include an accountable owner, documented meaning, supported access path, quality/freshness signals, change expectations, and a feedback route. The product may be backed by one or more tables, files, views, APIs, streams, or other interfaces; it is not synonymous with a particular storage layer or platform.

Use this framing when multiple consumers need a stable, discoverable contract or when a product owner can support the interface over time. A one-off report or small local workflow may not need a formal data-product wrapper. A catalogue entry or “certified” label is not, by itself, proof that data is correct, current, complete, or suitable for a new purpose. Give consumers evidence and limitations they can assess.

## 3. Choose a serving interface by workload

Serving modes can coexist. Select the least complicated combination that meets the consumer's freshness, access, performance, trust, and ownership needs. The options below describe consumer-facing trade-offs, not implementation recipes.

| Serving mode | Often useful when | Trade-offs and questions |
| --- | --- | --- |
| **File or bounded export** | A consumer needs a portable snapshot, scheduled handoff, offline analysis, or a simple exchange with a known boundary. | Agree format and schema, delivery/completeness signal, version or partition boundary, correction behavior, secure destination, and cleanup. Files are easy to move but can become stale, duplicated, or ambiguous without an owner and manifest/status. Use Appendix A for detailed serialization and compression decisions when that reference is available. |
| **Queryable analytical store, warehouse, lake, or query engine** | One or many analysts or applications need interactive reads over prepared data; a local query engine may also serve one person over local files. | Compare consumer access capability, latency, concurrency/isolation, freshness behavior, data movement, query-time dependencies, operating burden, and cost. Warehouse- and lake/lakehouse-backed paths can serve similar consumer needs but differ in access and supporting-runtime requirements; evaluate the actual workload rather than relying on labels. Chapter 6 covers storage-system properties; Chapter 8 owns query plans and persistence mechanics. |
| **Federated or shared access** | Consumers can use data where it lives, or avoiding a copy has meaningful value. | Query-time availability, source load, network latency, permissions, revocation, data-location boundaries, and consumer access capability remain dependencies. Confirm what the provider permits consumers to query, retain, or redistribute. Federation does not make source access free or eliminate ownership. |
| **Stream or event output** | A consumer must react to continuing changes rather than wait for a bounded refresh. | State the consumer-visible freshness and delivery expectations, event meaning, ownership, retention needs, and failure/status route. Keep inbound capture, source-event ordering, replay, buffering, and ingestion recovery with Chapter 7; continuous query and transformation semantics belong with Chapter 8. Agree the serving output's consumer contract and target-specific delivery expectations rather than assuming they follow from upstream ingestion guarantees. Do not select a stream solely because a use case is called “real time.” |
| **Semantic or metrics layer** | Multiple consumers need a reusable presentation of business-approved measures and dimensions. | Name the definition owner, supported measures, grain/filters, version/change policy, access boundary, and how consumers validate results. A semantic layer can expose common meaning; it cannot settle a business disagreement or make an unowned calculation canonical. Chapter 8 covers defining and deriving the measure. |
| **API or application interface** | An application needs a bounded, purpose-specific way to request or receive data or predictions. | Define request/response or push shape, versioning, expected latency/availability, authorization boundary, error behavior, and interface owner. Tailored interfaces can reduce consumer coupling but create service, compatibility, and support commitments. Application implementation is outside this reference. |

A serving design may combine these modes—for example, a scheduled export for analysts and a narrowly scoped API for an application. Avoid creating multiple copies or interfaces unless each has a consumer, owner, and reason to exist.

### Compare viable options, not labels

When more than one interface can meet the need, compare the same workload across realistic consumer-facing modes—such as direct query on a prepared dataset, federated/shared access, or a bounded export—not only capacity or caching variants within one mode. Include only options viable under the stated constraints; note why an obvious alternative is a poor fit. Use the following dimensions:

- **Freshness and latency:** What end-to-end delay is acceptable, and which step dominates it?
- **Concurrency and scale:** How many consumers or requests arrive together, and what happens at peaks?
- **Availability and dependency:** Which source, network, engine, or owner must be healthy at query or request time?
- **Consistency and semantics:** Do consumers see the same agreed definitions, and how are corrections or version changes represented?
- **Access and exposure:** Can access be restricted to the intended audience and purpose without creating unnecessary copies or disclosures?
- **Consumer capability:** Can the audience query and interpret the data, or do they need a curated interface or explanation?
- **Cost and operating burden:** What compute, storage, transfer, support, and failure-response responsibilities accompany the interface?
- **Reversibility:** How difficult is it to change the interface or migrate consumers if needs change?

For a consequential serving choice, compare two or three genuinely different consumer-facing modes in a compact table; for each, state when it fits, its main benefit, freshness/latency, concurrency and access capability, query-time dependencies, cost or complexity, failure/operating burden, owner/support needs, and how easily consumers could move away. Mark unknowns instead of inventing them, and briefly explain why an obvious alternative does not fit. Use [decision-lenses.md](decision-lenses.md) for fuller comparison prompts. Compute isolation, caching, or precomputation can refine a selected mode, but are not substitutes for comparing serving interfaces unless they materially change the consumer-facing contract.

Recommend one option with its assumptions and the evidence that would change the recommendation. If one straightforward mode meets the agreed requirements, do not manufacture alternatives.

## 4. Serve analytics with discoverable meaning and proportionate self-service

Analytical consumers need more than access to rows. They need to discover a supported dataset, understand its grain and definitions, recognize its freshness and limitations, and know who can answer questions. Provide documentation and examples that are appropriate to the audience and impact.

When several teams use a common metric, expose the business-approved definition through a reusable semantic or metrics interface where that reduces drift. Keep its owner and change process visible. Do not duplicate metric logic independently across reports or declare a calculation authoritative simply because it appears in a central tool. For metric meaning, grain, joins, transformations, and validation, use [08-query-modeling-and-transformation.md](08-query-modeling-and-transformation.md). Treat a missing or disputed KPI definition as an unresolved dependency for its business owner and Chapter 8—not a definition to invent during serving design.

Self-service is a supported access model, not unrestricted access. Agree, as relevant:

- which audiences and use cases are allowed;
- the documented, supported interface and how to request access;
- what meaning, freshness, completeness, and quality status consumers can inspect;
- proportionate query, export, and cost guardrails;
- how consumers report suspected errors, request changes, or learn of breaking changes.

Balance enablement with the data's sensitivity, contractual purpose, consumer capability, and risk of an incorrect action. Detailed security, privacy, and governance controls belong to Chapter 10; the serving design still has to state the access needs and boundaries that Chapter 10 must address.

## 5. Make ML data handoffs explicit

Data serving for ML has at least two distinct consumers: a process that assembles data for training or evaluation, and a model or application that consumes inputs or emits predictions at inference time. They may have different freshness, volume, latency, and availability needs. Do not assume one interface serves both well.

For the data handoff, agree only the parts that affect the selected serving design:

- **Purpose and owner:** Which model or decision process consumes the data? Who owns the input definition and the downstream use?
- **Input and label contract:** Which fields, features, labels, keys, and time meanings are expected? Link feature derivation and canonical definitions to Chapter 8 rather than redefining them here.
- **Time validity:** What was knowable at the time of a training example or prediction? Could future information leak into historical examples, or could online inputs differ materially from the training representation?
- **Freshness and delivery:** Does training use a reproducible bounded snapshot or periodic refresh? Does inference require a batch input or a lower-latency interface? What delay or incomplete-input behavior is acceptable?
- **Version and lineage:** Can a consumer identify the data/definition version used for a training run or prediction and reproduce or investigate the handoff, subject to retention and privacy policy?
- **Quality and failure visibility:** Which missing, invalid, stale, or out-of-range inputs should block, warn, or route the request for a fallback? Who owns the decision when the input contract is not met?
- **Output contract:** If predictions or scores are served onward, who consumes them, what do they mean, how current are they, and what happens when the model or input is unavailable?

Treat leakage, inconsistent training-versus-inference inputs, and unreproducible datasets as risks to investigate, not as a mandate to adopt a feature store or specific ML architecture. This reference does not design algorithms, model training/evaluation methods, model governance, or a full MLOps lifecycle; use the relevant ML owner or specialized guidance for those decisions.

## 6. Treat reverse ETL as a guarded outbound contract

Reverse ETL sends prepared or derived data from an analytical environment to an operational application or process so a recipient can take an action. It is a serving path with an external write boundary—not merely another analytical query, and not permission to update a target system.

Before choosing an implementation, define the proposed write contract:

- **Purpose and decision:** What specific action should the receiving system or person take? Is sending data back necessary, and could a read-only or human-reviewed path meet the need?
- **Target and authority:** Which system and target owner are involved? Who can authorize the permitted write scope, fields, population, and purpose? Is the target an approved destination for this data?
- **Identity and matching:** Which stable key maps a prepared record to the intended target entity? How will ambiguous, unmatched, merged, or stale identities be handled without updating the wrong entity?
- **Eligibility and timing:** Which rows or values are eligible, how fresh must they be, and how are opt-outs, corrections, deletions, or changed decisions reflected?
- **Write semantics:** Is the operation an insert, update, upsert, status change, or other action? How are duplicate attempts, retries, partial failures, conflicts, and target rate limits handled? Repeating a write should not silently cause an unintended repeated action.
- **Reconciliation and recovery:** How will sent, accepted, rejected, skipped, and corrected records be counted or traced? Who investigates differences? Can the flow be paused, safely resumed, or rolled back/corrected when the target permits it?
- **Traceability and support:** What minimum audit/provenance is needed to explain what was sent, when, and under whose authority, without copying unnecessary sensitive values into logs? Who owns incidents and target-side coordination?

A design should identify failure modes and a safe stop/correction path proportionate to the consequences. Do not imply that an analytical model's correctness guarantees that a target action is safe. Do not prescribe a vendor connector, target schema, or API implementation here; verify current target behavior with its owner during authorized implementation planning.

**Approval boundaries are separate:** agreeing on a design does not authorize creating a sync job, granting access, changing a remote setting, sending data, or writing to a production system. A request to implement is also not necessarily permission for each remote or production side effect. Before separately authorized implementation, confirm the business purpose and eligible population (not just the target fields), destination and environment, allowed objects/fields/actions, target owner or approver, run bounds, and a recovery/stop plan proportionate to impact. If the user says the production authorization is absent, do not treat the implementation request as that missing authorization; ask for the required confirmation before creating or running the job. Route explicit implementation requests to the optional `forge-de-deliver` skill when installed and hand over the approved contract; otherwise pass the approved scope to the authorized implementer. Do not implement the build within this Design reference. A build-stage approval does not by itself authorize a production write or another remote side effect. Test behavior with synthetic data or an approved non-production target first.

## 7. State serving-level service expectations and handoffs

A serving interface should make relevant success and failure conditions visible to the owner and consumer. At design time, specify only signals that matter for the agreed contract, such as:

- last successful refresh or delivery and observed data age;
- completeness or a known partial-data condition;
- interface availability, request/query latency, or consumer-visible errors;
- validation status for the checks that protect this use;
- owner, support route, and the condition that warrants escalation.

Choose targets and alert thresholds from the consumer impact and evidence, not from a universal template. A stale dashboard, a late training snapshot, and a failed operational write have different consequences and owners. Decide how a consumer should respond to an unavailable, stale, or invalid result instead of silently presenting it as current.

This reference defines serving-level expectations and ownership questions, not a full operations manual. Chapter 7 owns inbound ingestion recovery; Chapter 8 owns transformation/query correctness and persisted-output maintenance. The active `forge-de-operate` skill handles ongoing cross-product observability, incident response, on-call, restoration, and continual improvement. The optional `forge-de-operate` skill provides related Chapter 10 security, privacy, and governance context when installed; it does not prove an organization's controls are implemented. Detailed cloud-networking trade-offs remain assigned to the planned Appendix B. Treat these as design/operation handoffs, not evidence that a control already exists.

## 8. Common failure patterns

| Failure pattern | Why it fails | Better design question |
| --- | --- | --- |
| “Real-time” is selected without a decision that needs it. | Lower latency adds cost, dependencies, state, support, and failure modes without necessarily changing an outcome. | Which consumer action improves at the proposed freshness, and what simpler cadence would still work? |
| A dataset is called trusted or certified with no inspectable evidence. | Consumers cannot tell who owns its meaning, whether it is current, or which limits apply. | What owner, definition, lineage, validation, and status can this consumer verify? |
| Each report implements a slightly different KPI. | Similar names can hide divergent filters, grain, time handling, or business rules. | Who owns the definition, and how will consumers reuse the approved version? |
| Self-service means broad access to all fields and records. | Access can exceed the purpose, expose unnecessary data, and undermine trust. | What supported interface and audience-specific boundary enable the intended use? |
| Federation is assumed to be current, cheap, and isolated from source impact. | Query-time dependency, latency, source load, permission changes, and network costs remain. | Which source guarantees and consumer behaviors have been verified? |
| ML training and inference inputs are assumed to be interchangeable. | Time, freshness, version, or feature differences can make a handoff misleading or unreproducible. | What data version and time meaning does each consumer need, and how is a mismatch detected? |
| Reverse ETL retries or replays writes without a target contract. | Duplicate or conflicting actions may occur, while failures and corrections become hard to reconcile. | What makes an operation repeat-safe, and who can stop, reconcile, and correct it? |
| A serving target is treated as production authorization. | Design intent does not grant permission to expose data or cause operational side effects. | Which separate approval covers implementation and each consequential remote action? |

## 9. Design-to-build handoff

**Design conversation:** agree the consumer and action, required trust and service expectations, output/interface contract, viable serving options and trade-offs, owner, access boundary, acceptance evidence, and unresolved assumptions. For ML or reverse ETL, name the data handoff or write boundary and its downstream owner. Keep source acquisition in Chapter 7 and data semantics/preparation in Chapter 8. Discuss consequential choices until the user confirms the scope and decision for this stage; only then create that stage's project design document if documentation is wanted.

**Build handoff and acceptance:** when the user explicitly requests implementation, hand the approved contract and acceptance criteria to `forge-de-deliver` (or its sibling `SKILL.md` if skill invocation is unavailable); this Design reference does not implement the system. Propose bounded tests that verify consumer-visible freshness/status and relevant success, stale/partial, invalid-input, or failure behavior. For ML, include the agreed input/version/time contract; for reverse ETL, specify tests for duplicate, conflict, rejection, reconciliation, and stop/correction paths using synthetic data or an approved non-production target. Report what should be verified and what remains unverified. Remote actions, production writes, permission changes, sensitive-data use, and material cost require their own applicable authorization.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 9, “Serving Data for Analytics, Machine Learning, and Reverse ETL.” The consumer contracts, comparison prompts, service-expectation scaffold, and guarded writeback questions are Data Forge applications, not copied book templates. Data products and serving modes are options, not universal requirements; operational, security, privacy, legal, vendor, and pricing details must be checked against the current environment and relevant owners. No book prose, figures, or distinctive tables are reproduced. See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
