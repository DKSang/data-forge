# Data architecture principles and patterns

Architecture describes the system boundaries and choices that make data useful under real business, reliability, security, team and cost constraints. A diagram or tool list is only a partial implementation view. Start from the need and constraints; then compare a small number of viable architectures. This reference adapts Chapter 3 of *Fundamentals of Data Engineering* in original wording. Its principles and patterns are decision lenses, not universal standards or a prescribed stack.

For the preceding business/lifecycle framing, see [01-business-value-and-data-maturity.md](01-business-value-and-data-maturity.md) and [02-lifecycle-and-undercurrents.md](02-lifecycle-and-undercurrents.md). For comparing concrete tool choices, use [decision-lenses.md](decision-lenses.md) and [Chapter 4's technology-selection reference](04-technology-selection.md).

## 1. Keep architecture distinct from implementation

Express the requirement before naming a technology:

- **Need:** which business action, consumer, or product must be supported?
- **System qualities:** what latency, reliability/recovery, security, privacy, concurrency, data residency, interoperability and cost matter for that use?
- **Boundaries:** which data and capabilities belong to which source, platform, domain, team, or external customer? Who owns each contract?
- **Architecture:** which components and responsibilities must exist, how they connect, and what alternatives are viable?
- **Implementation:** which tools, services, code and configurations realize those responsibilities today?

Chapter 3 distinguishes operational architecture (the requirements, boundaries and system behavior to support) from technical architecture (how design and technology can realize it). Data Forge applies a related working sequence: first understand what and why, then compare how. That sequence is a design aid here—not a claim that Chapter 3 itself calls architecture “strategic” and tools “tactical.” In practice, a tool's constraints and capabilities may feed back into architecture; make that dependency visible and revisit the design when needed. Verify current product behavior rather than inheriting 2022 examples as recommendations.

Architecture responds to business needs that change. It is therefore a continuing activity: establish the current state and a useful target for the slice, implement a next step, learn from real usage and revisit when evidence or constraints change. Do not force a multi-year target diagram onto a small, reversible project.

## 2. Architecture review lenses

The authors provide a set of architecture principles. The following reorganizes them into practical review lenses rather than reproducing their list as a compliance checklist.

### Business fit, ownership and evolution

- Tie the design to a named outcome, consumer, source owner and operating owner. Involve the people who set business definitions, use the data, operate the source and respond to failures.
- Make important boundaries and decision rights explicit. Common components or standards help when they reduce repeated work and improve interoperability; they can harm when a central service blocks valid domain needs or shifts effort onto every team.
- Treat architecture as leadership and communication as much as technical design. A short rationale, clear assumptions, and an agreed sequence can prevent teams from implementing incompatible interpretations. Where someone has architectural responsibility, share expertise through mentoring or training so decisions do not become a central bottleneck; a separate architect role is not required for every team.
- Review architecture as the system evolves. Record what would trigger a change, rather than trying to forecast every future requirement at the outset.

### Failure, reliability and scale

- Consider credible failure modes at each important boundary: source unavailable or slow, partial data, a downstream system full, a credential revoked, a job failing midway, or a tenant/resource affecting another.
- Define recovery needs in terms of business impact. When important, ask what data loss and service interruption are tolerable, how recovery will be tested, who responds and what evidence indicates safe restoration. Use terms such as RTO/RPO only when they clarify a real requirement.
- Estimate ordinary, peak and plausible future workload; identify bottlenecks and the cost of scaling up, out, down or to zero. Do not add distribution or high availability without a requirement that justifies its coordination and operating burden.
- Distinguish **scalability** (capacity to grow), **elasticity** (capacity to adjust as demand varies), **availability** (being reachable) and **reliability** (consistently meeting expectations). Related, but not interchangeable qualities.

### Coupling, reuse and reversibility

- Identify both technical dependencies and organizational dependencies. A component boundary is useful when its contract is understandable and one part can change without surprising all consumers; extra services and interfaces also create maintenance and failure points.
- Assess common components by actual reuse, integration effort, service ownership, access control, performance and total cost. Prefer shared pieces where they reduce repeated work; keep local options where requirements genuinely differ.
- Compare the cost of switching with the value of flexibility. Prefer reversible choices when uncertainty is high and the penalty is modest. For a hard-to-reverse choice, explain viable alternatives, migration/exit cost, and what evidence would change the decision; do not create an ADR for every ordinary implementation detail.

### Security and cost are architectural concerns

- Do not rely only on a hardened network perimeter: include data sensitivity, identity and access boundaries, tenant isolation and privacy in the design. The Chapter 3 zero-trust lens questions implicit trust based only on network location; apply the current identity/resource model appropriate to the environment rather than copying a cloud recipe.
- Engineers are responsible for securing the systems they build and maintain, alongside provider responsibilities where managed services are involved. Identify the boundary for identity, configuration, data, application and infrastructure controls, and verify current provider and organizational requirements.
- Treat FinOps as an ongoing, cross-functional feedback practice among engineering, finance, technology and business—not merely a budget ceiling. When cost is material or variable, monitor spend and relevant unit costs, assign an owner for alerts, and adjust usage/design to increase value per cost. Include the risk of unbounded or abusive data access where data is shared publicly.
- Security and cost controls are proportional to use and impact, but should be explicit wherever a credible risk or material spend exists. Current regulatory duties and provider-specific shared-responsibility details must be verified with the relevant owners and current documentation.

## 3. Patterns are alternatives, not maturity badges

Choose a pattern by the requirements it satisfies and the consequences the team can operate. These patterns overlap, and names alone are not enough to select one.

| Pattern family | What it can make easier | Costs, failure modes and questions |
| --- | --- | --- |
| **Analytical warehouse** | Governed, queryable analytical data with managed schemas and a familiar SQL experience. | Where do transformations run? Does its data model, concurrency, sharing, update behavior and cost fit all intended consumers? Can upstream operational workloads remain protected? |
| **Data lake / file-oriented storage** | Retaining diverse or large source data and selecting compute or consumers later; useful where files/open formats and independent engines matter. | Low-cost storage alone does not make data discoverable or trustworthy. Who owns schemas/catalog, access, quality, update/deletion, lifecycle and recovery? Is the team prepared for file and metadata operations? |
| **Lakehouse or converged platform** | Combining flexible/object-style storage with more managed table, transaction or query capabilities; potentially serving more than one workload. | Which capabilities are actually available and interoperable? Does the convergence reduce real integration work or add a complex platform with unclear ownership? Verify current support, openness, consistency, security, migration and full cost. |
| **Modular stack versus consolidated platform** | Modular components can let teams choose a fit-for-purpose capability; a more consolidated platform can reduce integration and operating overhead. | Modularity increases interfaces, version compatibility, observability and coordination work. Consolidation can increase dependency or reduce autonomy. Compare end-to-end operations, not only feature lists. |
| **Central platform versus domain-oriented ownership** | Centralization can standardize controls and reduce duplication; domain ownership can keep definitions and decisions close to their producers and consumers. | Central models can become bottlenecks; domain models can fragment semantics, security and quality. A domain-oriented design needs accountable owners, usable data products, enabling platform capabilities and rules for cross-domain compatibility. It is an organizational choice as well as a technical one. |
| **Single application/monolith versus separated services** | A cohesive application can be simple to build and operate; separated services can support independent ownership, deployment, or scaling when boundaries are real. | Premature distribution adds network, deployment, monitoring, contracts and failure complexity; tightly interdependent services can become a distributed monolith. Separate technical layers from business-domain boundaries before splitting components. |
| **Event-driven integration** | Producers can publish events through a broker/router to consumers, reducing direct timing dependencies and enabling independent reactions. | Specify event ownership, schema, delivery/replay behavior, idempotence, observability and consumers. Events do not remove the need for contracts, failure handling or access control. |
| **Batch/stream processing topology** | Batch, micro-batch or continuous processing can each fit different deadlines and actions; the processing cadence is a separate choice from whether components communicate through events. | Two batch/stream paths may duplicate logic and diverge; a single streaming path may impose replay, retention, historical backfill and cost challenges. A unified programming model may help, but does not remove workload or operational differences. See [decision-lenses.md](decision-lenses.md) and the future ingestion reference for deeper cadence choices. |

### Common streaming and organization patterns

Use these names only when the trade-off is relevant:

- **Dual-path batch/stream (often called Lambda):** separate paths may meet both historical and low-latency use cases, but duplicate logic and reconciliation can create inconsistency and operational load.
- **Stream-and-replay (often called Kappa):** a retained event log can support historical recomputation through the same processing path, but replay duration, retention, throughput and large historical datasets can make this expensive or operationally difficult.
- **Unified bounded/unbounded processing:** one programming model may handle finite batches and ongoing event streams, often with windowing semantics. Confirm how the actual system represents time, late events, state and recovery; uniform API does not mean identical execution costs.
- **Domain-oriented data mesh:** consider only when domains can own products and semantics, consumers need cross-domain data, a platform can enable self-service, and federated rules can protect shared interoperability, security and quality. It is not simply “decentralize the lake” or a software product.
- **Brownfield and greenfield:** an existing system needs a migration path that recognizes why it exists and how to safely retire or coexist with it; a new system needs disciplined scope and evidence just as much as a migration. Incremental replacement may reduce risk when dependencies can be isolated and success measured.

For specialized environments such as IoT, explicitly account for device power/connectivity, buffering, late or malformed messages, and edge processing. Ask what response deadline the use case requires and how it affects persistence and serving: delayed reporting may fit buffered batch processing, while prompt alerts or device feedback may need a lower-latency path and a different storage/serving design. That is a workload-specific branch, not the default architecture for a general analytics platform.

## 4. Compare only viable architectures

For each consequential architecture option, state:

1. **Fit:** which confirmed need or constraint it addresses.
2. **Trade-offs:** effects on latency, reliability/recovery, security/privacy, data semantics, interoperability, team operations and total cost.
3. **Failure/coupling:** where a fault, schema change, noisy workload or unavailable owner can propagate.
4. **Change path:** what is easy or hard to replace; how migration and coexistence would work.
5. **Evidence:** what safe benchmark, proof of concept, source contract or stakeholder check could disprove the assumptions.

Recommend a narrow, understandable option when uncertainty and current scale are modest. Increase separation, redundancy, governance or customization when a real consumer requirement, risk or measured load justifies it. Ask the user to confirm the architecture decisions for this stage before writing the approved design document; selecting architecture does not approve a vendor, deploy, cloud spend or production run.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 3 sections “Data Architecture Defined,” “Good Data Architecture,” “Principles of Good Data Architecture,” “Major Architecture Concepts,” “Examples and Types of Data Architecture,” “Data Mesh,” and “Who’s Involved with Designing a Data Architecture?” The review lenses, comparison prompts and grouping of patterns are Data Forge adaptations. The authors' frameworks, illustrative patterns, terminology and 2022 technology landscape are perspectives to evaluate, not universal requirements. No book prose, tables, figures or diagrams are reproduced. For tool-specific feature, support, pricing and current implementation choices, consult current primary sources and the future technology-selection reference.
