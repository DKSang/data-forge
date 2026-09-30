# Compare options without choosing a stack first

Start with the required outcome and constraints, then compare actual viable choices. Include the cost of operating and changing the system, not just vendor pricing or peak benchmarks. Technical choices here are examples, not a catalog of defaults.

| Decision | Ask / compare | Warning sign |
| --- | --- | --- |
| Local / on-prem / cloud / hybrid | Residency, network and source location, team support, elasticity, existing contracts, operational ownership, direct + indirect TCO | “Cloud is always scalable” or “local is always cheaper” without workload and ops assumptions |
| File/OLTP DB / warehouse / lake / lakehouse | Consumer query patterns, concurrency, update/delete needs, governance/catalog, sharing, size, portability and maintenance | Selecting lakehouse merely because the raw/staging/curated diagram has three boxes |
| Single process / distributed engine | Data size relative to memory, shuffle/join needs, concurrency, failure recovery, local simplicity, skills | Adding Spark to a dataset that SQLite/DuckDB/SQL can handle safely |
| Batch / micro-batch / streaming | Decision deadline versus end-to-end latency, event-time ordering, state, replay, infrastructure, source ability | Equating “real-time dashboard” with a need for millisecond ingestion |
| Poll / API pull / CDC / events | Source impact, delete capture, read permissions, rate limits, completeness, ordering and replay | Assuming CDC is available or a timestamp cursor catches deletes |
| ETL / ELT | Where validation and access control occur, compute capacity, privacy boundary, reuse and debugging | Declaring one approach correct for the whole company |
| Direct model / dimensional / normalized / vault-style | Grain, update history, query patterns, entity consistency, consumers, complexity of maintenance | Selecting a modeling school before the business terms and grain are known |
| One curated copy / several domain-owned products | Ownership, contracts, common semantics, latency and cross-domain integration | Equating one physical database with one agreed business definition |
| Managed / self-hosted / build | Reliability/SLA, team capacity, lock-in, upgrade path, security and lifecycle cost | Comparing license cost but omitting incident and maintenance labor |

For each important choice, present a compact comparison in chat:

- **Decision and constraint:** what must work and which constraints are verified versus assumed.
- **Options:** normally 2–3, with meaningful upside, downside, failure/recovery burden and reversibility. No invented options when only one is viable.
- **Recommendation:** why it meets the current needs and what evidence might change the answer.
- **Validation:** a safe experiment or measurement with an explicit threshold and owner if uncertain.

Use proportionality: a one-person local pipeline may need an idempotent script, a small DB, tests and alerting rather than Kafka, an orchestrator and three storage layers. A high-volume regulated cross-region system may justify more separation, automation and controls. Neither example predetermines a user's actual architecture.
