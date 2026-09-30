# Business value and data maturity

This reference guides the first design conversation: why the data work matters, who must benefit, and how much engineering the organization can support. It applies the book's Chapter 1 principles in original language; it is not a transcription or a substitute for talking to the actual stakeholders.

## 1. Define the work by its outcome

Data engineering is the engineering of repeatable systems that make source data dependable and usable for downstream decisions and products. Data movement, storage, and transformation are means; the outcome is that an identified consumer can act on information with appropriate confidence, timeliness, access, and cost.

Before drawing a platform, establish:

- **Business goal:** what result, risk reduction, customer experience, or operational decision should change?
- **Consumer and action:** which people or systems use the output, and what will they do differently? Identify an accountable business/data owner, not just the team requesting a table.
- **Measure and meaning:** what KPI or data product indicates success; who defines it; what entity and grain does it describe; which exclusions, time boundaries, units, and edge cases matter?
- **Service expectation:** how fresh, complete, available, explainable, and fast does the data need to be for that action? Separate a real deadline from an aspirational “real-time” label.
- **Evidence:** separate technical/data-product acceptance (such as correctness, freshness and usable access) from a downstream outcome or adoption indicator. Name the owner, baseline, target and review point for each; learn whether the data contributed without claiming causal impact that has not been established.
- **Constraints:** source authority and permission, privacy/security, locality, budget, existing systems, team skills, operational ownership, and delivery deadline.

The split between product acceptance and downstream outcome, along with explicit baselines, targets, review points and caution about causal claims, is Data Forge's practical measurement safeguard. It operationalizes the chapter's outcome-first and quick-win themes; Chapter 1 does not prescribe this measurement protocol.

If the request is only a technical symptom, trace it to a user impact. If there is no identifiable consumer or decision yet, propose a short discovery step rather than designing a large platform in search of a use case. Do not invent an ROI number or KPI definition; state an assumption and ask its owner to validate it.

## 2. Fit the approach to data maturity

Maturity is about how consistently the organization uses, trusts, integrates, and operates data—not its age, revenue, headcount, or cloud adoption. Diagnose the **specific workload and team** along independent signals rather than assigning the whole organization a single stage:

- **Business pull:** Is work mostly ad hoc, or is there a known decision, consumer, accountable sponsor and success measure?
- **Data readiness:** Are source ownership, business meaning, quality, permissions and change expectations understood well enough to reuse the data?
- **Repeatability:** Does delivery depend on manual, individual effort, or are changes versioned, tested, automated, observable and recoverable?
- **Reach and controls:** Is use confined to one team, or do multiple teams, customer-facing applications or ML workflows need discoverable data, reliable access boundaries and shared governance?

The authors offer a simplified progression from getting started, through scaling, to broader organizational use. Use that as an optional orientation, not a rubric or a required sequence. At an early point, it can help to secure a sponsor, clarify one valuable slice, assess its sources and build a dependable minimum foundation. Make visible progress, while recording shortcuts as technical debt with an owner or revisit trigger; an ambitious ML initiative without an adequate data/production foundation is a common failure mode. As recurring demand grows, formal practices, team throughput, automation, reliability and ML-supporting systems may become valuable. At broader scale, discoverability, controlled self-service, quality, lineage/governance and ongoing maintenance may matter more. The next capability should still follow a validated use case, not the label assigned to a company.

A team can combine these tendencies. One practitioner may favor managed or off-the-shelf abstractions for routine work and build a custom component for a genuinely distinctive, mission-critical need. Treat that as a choice about the work, not a personality type or hiring requirement. Maturity adjusts the **proportion** of specialization, automation and governance that is useful; baseline safety, privacy, correctness and clear responsibility remain operating principles of this skill suite, not a claim that the book sets a formal maturity-stage rule.

## 3. Balance the engineering objectives

Do not optimize one dimension in isolation. Make trade-offs explicit across:

- time to useful delivery and agility;
- correctness, security, privacy, and operational risk;
- total cost of ownership (including engineering/on-call time), opportunity cost, and expected business value;
- scalability for evidenced demand, not hypothetical maximum scale;
- simplicity, maintainability, reuse, and interoperability;
- reversibility and the cost of changing direction.

A small, local, well-tested solution may be the right foundation; a distributed or managed platform may be warranted by measured volume, availability, collaboration, or governance needs. Custom code or infrastructure is justified when it provides a concrete product, regulatory, performance, or competitive advantage that a simpler option cannot. For hard-to-reverse choices, document the alternatives and revisit conditions after the user agrees to that stage.

## 4. Work across business, data, and engineering

Data engineering connects producers and consumers; the boundary is collaborative, not a universal org chart. A source may be produced by application/software teams, an operational platform team, a SaaS provider, or a partner. Consumers may be analysts, data scientists, ML engineers, product teams, operational systems, or customers. Map who owns source behavior, business definitions, access approval, data-product quality, downstream use and production support. Project/product managers, architects, operations/SRE, security and business leadership may also shape scope and handoffs.

Clarify whether the output is **internal-facing**, **external/customer-facing**, or both. External application use can introduce materially different query concurrency, workload limits, tenant isolation, access/security and availability needs; it may also create a feedback loop where application activity is processed and fed back to that application. Do not impose these concerns on a simple internal report, but ask about them whenever a customer-facing path exists.

Data engineering builds and operates dependable data inputs and interfaces so downstream roles can focus on analysis, experimentation, decisions and models. Engineers should understand analytics, BI and ML consumption well enough to design useful contracts and help productionize repeatable preparation, while making explicit who owns analysis, dashboards, KPI meaning, models and business decisions. Role boundaries can overlap; clarify responsibility for this project rather than assuming the data engineer owns every adjacent function.

Treat Agile, DevOps and DataOps as collaboration and feedback practices, not as software purchases. Set expectations for review, releases, monitoring, incident response and learning with the people who will operate and use the result.

Production-grade coding and software-engineering capability remain important even when managed services reduce low-level work. Make changes reviewable, test transformations, automate repeatable tasks, handle failures and keep code/configuration understandable. SQL is especially useful for declarative data work; combine it with Python or another suitable language when that makes the logic clearer or more maintainable. Low-code and managed interfaces can reduce implementation effort, but they do not remove the need to understand and engineer reliable production behavior.

## 5. Discovery conversation and stage output

Keep the first conversation short enough to make progress. Ask only questions that can change the scope or design, such as:

1. Which person/team makes which decision, and what is difficult today?
2. What does the key measure mean—including grain, units, time window, exclusions, and owner?
3. How fresh and reliable must the result be, and what harm occurs when it is late or wrong?
4. What source is authoritative, who owns it, and what extraction/use is authorized?
5. What can the current team build and operate; what constraints (privacy, region, deadline, budget) are fixed?
6. What is the smallest useful deliverable and how will its technical acceptance and downstream value be assessed separately?
7. Is the output internal-facing or customer/application-facing? If both, what differ in concurrency, tenant boundaries, query limits, access and feedback flow?

Reflect back verified facts, assumptions, open questions, a narrow recommendation, its trade-offs, and the two types of success evidence (product acceptance and downstream outcome/adoption). Ask the user to confirm the scope and decisions for this stage. **Only after explicit agreement**, and when a project document is wanted, record a concise value brief containing the problem, consumer/action, measure definition and owner, freshness/quality expectations, constraints, acceptance evidence, and unresolved items. Do not record an assumption as an approved business definition.

## Source and scope note

Conceptual basis: Joe Reis and Matt Housley, *Fundamentals of Data Engineering*, 1st ed. (O’Reilly Media, 2022), Chapter 1 sections “Data Engineering Defined,” “Data Maturity and the Data Engineer,” “Data Engineering Skills and Activities,” “The Continuum of Data Engineering Roles, from A to B,” “Internal-Facing Versus External-Facing Data Engineers,” “Data Engineers and Other Technical Roles,” and “Business Responsibilities.” This guide uses those ideas to create an original multi-signal maturity diagnostic; it does not reproduce the book’s stage table, labels as a taxonomy, passages, figures, or tables. Specific approval gates, safety wording, and the product-acceptance-versus-outcome measurement scaffold are Data Forge operating practices, not frameworks attributed to Chapter 1. See also this suite's [source note](sources.md).
