# Technology selection and evidence

Choose technology to realize an agreed architecture and a real use case—not to define the architecture after a product was selected. Chapter 4 of *Fundamentals of Data Engineering* offers criteria for making those choices; this reference turns them into a practical comparison process, without endorsing any vendor or treating its 2022 product examples as current.

Start with [03-architecture-principles-and-patterns.md](03-architecture-principles-and-patterns.md) to understand the needed system and boundaries. Use [decision-lenses.md](decision-lenses.md) for quick comparisons such as local versus cloud, batch versus streaming, or managed versus self-hosted. This reference adds a deeper way to evaluate actual candidate technologies. If the architecture or outcome is still unclear, return to design rather than scoring products prematurely.

## 1. Identify viable candidates in context

Technology is a socio-technical commitment: people must build, learn, operate, secure, integrate, pay for and eventually change it. Before comparing products, record:

- **Required capabilities:** which agreed requirements must the tool satisfy, and which are preferences?
- **Current system and interfaces:** what does it need to interoperate with, including schemas, identity, networking, metadata, monitoring and data export?
- **Team capacity:** who will build, administer, support and troubleshoot it? What skills exist today, how hard is training/hand-off, and who provides coverage when a specialist is unavailable?
- **Time to useful delivery:** how soon must a safe, valuable result reach users? How quickly can the team test, learn and iterate while preserving quality, reliability and security?
- **Operating boundary:** which work is handled by the team, a vendor or a managed service? What remains the team's responsibility for configuration, data, access, incidents and recovery?
- **Change horizon:** which capabilities are durable foundations and which product/tool choices may age quickly? Choose for current and near-term needs rather than speculative futures; preserve replaceable boundaries when the flexibility is worth its cost, and name what would trigger a review.

Use a shortlist of two or three **actually viable** options when a meaningful choice exists. Include a familiar/simple option and a managed or existing capability if they could meet the need. Do not create artificial candidates just to fill a comparison table. A vendor, open-source project or preferred language named by the user is an input—not approval of all its operational and economic consequences.

## 2. Compare the full cost and value

A license or hourly infrastructure price is not total cost. Estimate over an explicit time horizon and workload, separating one-time and recurring costs where useful:

- acquisition/subscription, infrastructure, storage, compute, data movement and network/egress;
- implementation, integration, migration, environment setup and testing;
- engineering, administration, on-call, incident response, upgrades, patching and support coverage;
- training, hiring/contracting, hand-off, documentation and operational tooling;
- security/privacy controls, backup/restore, compliance and cost of outages or bad data;
- switching, export, retraining and eventual exit/retirement.

Also consider **opportunity cost**: what valuable work or time-to-market is lost by building, operating, learning or waiting for this option instead of an alternative? Compare expected business value and risk reduction with total financial and nonfinancial cost; do not invent ROI numbers or assume that lower spend is always better. Chapter 4 cites “total opportunity cost of ownership” as a way to draw attention to foregone alternatives; treat that as a conceptual prompt, not a standardized accounting formula.

For consumption-based services, model plausible low, typical, peak and adverse usage if cost can vary materially. Note assumptions, price date/source, cost owner and a signal or threshold that would trigger review. Calculate a cost per useful unit only when the unit (a successful load, query, customer, or other outcome) helps make a decision.

FinOps is continuous: revisit real spend and value after workloads run, investigate meaningful variance and adjust resources, product choice or workload design. A fixed two-year review interval from an older source is not a universal cadence; use decision triggers and an appropriate review rhythm for this system.

## 3. Assess the main selection dimensions

| Dimension | What to compare | Evidence or question to seek |
| --- | --- | --- |
| **Team fit and time to value** | Familiarity, staffing/coverage, learning curve, setup, delivery speed, feedback cycle and operations burden. | Can the actual team support the system after launch? What is the smallest safe proof of useful value, and what quality/security controls must it still satisfy? |
| **Interoperability** | Existing connectors and standards, schema/semantic compatibility, identity, networking, metadata, failure behavior, data movement and export options. | Test the critical producer-consumer path, not a marketing list. Can data and configuration be retrieved or migrated in usable form? Which integrations are proprietary or require custom maintenance? |
| **Technology lifetime and reversibility** | Project maturity, community/vendor continuity, roadmap transparency, support, ecosystem, lock-in and replacement cost. | What depends on this technology? Can components be swapped behind a contract? What would trigger a review, and what is the credible exit path? |
| **Location and deployment** | Local/on-prem/cloud/hybrid/multicloud needs, workload placement, source proximity, residency, connectivity, team operations and movement cost. | Which location constraints are real? Use [decision-lenses.md](decision-lenses.md) and verify current network, jurisdiction, provider and pricing facts. Prefer the simplest location that meets the need; justify hybrid or multicloud with a concrete benefit that exceeds added networking, security, integration and operations overhead. Do not assume a cloud label guarantees lower cost or easier operation. |
| **Build, buy or adopt** | Custom development versus an existing open-source, commercial/open-source or proprietary capability, including managed offerings. | Does custom work create a concrete differentiator? The authors lean toward adopting an available OSS/COSS solution for common work when custom code adds no advantage; treat that as their contextual preference, not a procurement rule. For OSS, who owns upgrades, security fixes, hosting and incidents? For a managed offer, what service levels, support, portability, costs and shared responsibilities remain? |
| **Monolith, modules and service boundaries** | Simplicity and fewer moving parts versus independent changes and possible failure isolation or scaling when component boundaries are genuinely independent. | Do boundaries match ownership and actual independent needs? Shared codebases, dependencies or release processes can preserve coupling and create a distributed monolith; splitting services alone does not guarantee autonomy, isolation or scale. What integration/orchestration work is added? Could a simpler cohesive component be split later if measured requirements emerge? See Chapter 3's architecture-pattern reference. |
| **Execution and packaging** | Execution model: managed/serverless versus servers. Packaging/runtime: containers versus native or other deployment forms. Containers can run on servers and may also underpin managed services; these are overlapping, not mutually exclusive axes. | For execution, measure workload shape, concurrency, duration, runtime/network limits, customization, control and cost at scale. Do expected and adverse event rates fit service limits? When do managed execution savings outweigh per-use, networking, cold-start or platform constraints? For packaging, consider dependency isolation and portability against image/build/patching and orchestration work. Who owns operation at each layer? |
| **Performance and benchmarks** | End-to-end latency, throughput, concurrency, reliability, resource use and price for the intended workload. | Does the test use representative data size/distribution, formats, queries, concurrency and cache state? Are options configured and tuned comparably? What is the cost and operational effort per useful result—not just peak speed? |

These dimensions are related but not interchangeable. For example, managed versus self-hosted is a sourcing/operating choice, while serverless versus servers is an execution model; one does not determine the other. Locality is one location constraint, while portability and interoperability concern how readily systems and data can move.

Chapter 4 also asks that the lifecycle's **undercurrents** inform technology choice. Review only those material to the candidates and workload, using [02-lifecycle-and-undercurrents.md](02-lifecycle-and-undercurrents.md) for the broader concepts:

- **Security and data management:** access/privacy controls, regulatory or residency fit, data quality, metadata/lineage, discoverability and retention/deletion needs.
- **DataOps and orchestration:** ability to deploy changes safely, test workflows, observe jobs/data, alert and respond; fit for dependencies, external readiness conditions, retry and backfill.
- **Software engineering and architecture:** testability, versioning, dependency isolation, portability, maintainability and how the choice fits the selected system boundaries.

This is a prompt to compare relevant capabilities and responsibilities, not a requirement to procure dedicated tools for each concern or to duplicate the full cross-cutting checklist.

## 4. Build evidence before committing

Prefer the cheapest safe evidence that can reject a weak option:

1. Verify non-negotiable feature, support, security, compliance, residency and contract claims in **current primary documentation** and with the responsible organization/vendor owner. Record what remains unverified.
2. Try a narrow integration or proof of concept with synthetic/non-sensitive data. Test the actual interfaces, permissions, schema changes, errors, recovery, export and developer/operations workflow.
3. Benchmark only when performance or unit economics could change the decision. Use a representative workload and comparable configurations; separate warm-cache from cold-start needs, report resource/cost use and acknowledge what will not generalize.
4. Include the responsible engineers and consumers in the review. A result that is technically fast but cannot be operated, secured, afforded or understood by its users is not a successful selection.

A product page, single peak benchmark, social proof, or proof that a feature exists is not by itself evidence that the option fits this system. Avoid spending months comparing choices when a reversible, low-risk pilot can answer the question; also avoid a pilot that touches production, customer data, creates cost or changes access without the required approval.

## 5. Record the decision after agreement

When the user has confirmed the relevant design stage, a concise technology decision record may capture:

- the architecture capability and constraints being implemented;
- candidates considered and why they were viable;
- the chosen option, assumptions and evidence (including date/source for volatile claims);
- TCO and opportunity-cost considerations, operations/security ownership and exit path;
- rejected alternatives and their decisive trade-offs;
- a trigger or review date tied to workload, cost, support, compliance or team changes.

Do not write or update the project design document until the user has explicitly approved that stage, following [documentation.md](documentation.md). Design approval is not approval to purchase, deploy, incur cost or run a production workload; those actions have their own permission and confirmation boundaries.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 4 sections “Team Size and Capabilities,” “Speed to Market,” “Interoperability,” “Cost Optimization and Business Value,” “Today Versus the Future: Immutable Versus Transitory Technologies,” “Location,” “Build Versus Buy,” “Monolith Versus Modular,” “Serverless Versus Servers,” “Optimization, Performance, and the Benchmark Wars,” and the concluding discussion of lifecycle undercurrents. The criteria are paraphrased and contextualized; provider names, prices, feature limits and two-year review suggestions in the book are not current endorsements or requirements. The full comparison process and evidence prompts are Data Forge adaptations. No book prose, tables, figures or diagrams are reproduced. See [the source note](sources.md) and repository-root `BOOK-COVERAGE.md` in the source checkout (not included in a single-skill install).
