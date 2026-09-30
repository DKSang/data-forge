---
name: forge-de-operate
description: Help operate and improve trusted data products across local, on-prem, cloud, and hybrid environments. Use whenever the user asks about ongoing data quality or freshness, observability, SLOs, access/security/privacy/governance controls, retention/deletion, backup/restore, incident readiness, operational ownership, cost, or continual improvement. Diagnose read-only first; do not infer permission for remote or production actions.
---

# Operate trusted data systems

Operations keep a data product useful after implementation: its data remains fit for the agreed purpose, consumers can detect problems, and owners can respond and improve controls. Start from the relevant system and current evidence rather than assuming a particular platform, medallion architecture, or enterprise checklist. Operate covers ongoing health and control responsibilities; it does not replace stage-specific design approval or the implementation skill.

## When to use Operate

Use this skill for questions about an existing or approved system's:

- data quality, freshness, completeness, availability, consumer trust, and service expectations;
- monitoring, observability, alert ownership, incident readiness, restoration, and lessons learned;
- access reviews, security/privacy controls, governance follow-through, retention/deletion, and backup/restore;
- operational ownership, support boundaries, costs, and proportionate improvement.

If a requirement is not yet decided, route it to `forge-de-design`. If the user requests code or a bounded configuration change, hand it to `forge-de-deliver` after confirming scope. Use `forge-de-brain` only for retrieving or maintaining user-approved project facts, not to turn an operational assumption into policy.

## Operating workflow

1. **Establish the system and authority.** Identify the user’s project, environment, owner, consumer, and the operational concern. Read approved design/operations records and safe metadata; distinguish current evidence from an old plan or assumption. If ownership, access, or incident authority is unclear, ask the responsible person rather than inferring it.
2. **Start read-only.** Inspect only authorized status, logs, lineage, test results, configuration summaries, and synthetic or approved evidence needed to diagnose. Do not retrieve unnecessary sensitive rows, expose credentials, or use production writes as discovery. If evidence is missing, state what cannot be verified and propose a safe next step.
3. **Assess impact and urgency.** Describe which consumers, decisions, data, or controls may be affected; distinguish confirmed facts from hypotheses; identify stale/partial/invalid behavior and any data-use or security boundary. For a suspected security/privacy incident, prompt escalation through the organization's incident route and authorized owners.
4. **Choose the smallest safe response.** Offer read-only analysis, a synthetic/local reproduction, an owner review, or a bounded implementation request as appropriate. Compare operational options when there is a real choice and state recovery, reversibility, and cost implications. Do not invent an SLO, legal obligation, or service guarantee.
5. **Confirm the action gate.** Before a remote job, production write, permission or configuration change, deletion, external data transfer, material spend, or other consequential effect, identify the exact target, scope, side effects, approver, verification, and rollback/stop plan; obtain the required explicit authorization. An approved design, incident ticket, Brain record, or request to “fix it” is not by itself authorization for every side effect.
6. **Hand implementation to Deliver.** For code, configuration managed as code, or a bounded system change, pass the approved scope and operational acceptance criteria to `forge-de-deliver`. Keep execution and remote/production effect approvals separate. Do not claim an alert, restore, access review, or mitigation was completed unless it was actually authorized, performed, and verified.
7. **Report and improve.** Summarize evidence inspected, impact, owners, decisions, actions actually taken, results, unresolved risks, and the next review trigger. Update operational records only in the project's established location and only when documentation is requested and authorized. Feed confirmed durable facts to Brain only after their status and provenance are clear.

## Reference routing

Load focused guidance only when it applies:

- Use [10-security-privacy-and-governance.md](references/10-security-privacy-and-governance.md) for cross-lifecycle security, privacy, governance, ownership, control evidence, retention/deletion, and incident-readiness decisions. It is vendor-neutral guidance, not proof that an organization's controls or legal obligations are satisfied.
- Use the optional `forge-de-design` skill for lifecycle and cross-cutting DataOps framing when installed; otherwise assess from the approved project context.
- Use `forge-de-design` for ingestion boundaries, recovery requirements, consumer-facing serving, and reverse-ETL contracts when installed; otherwise use the approved project contracts and this skill's operational scope.
- The optional `forge-de-deliver` skill owns implementation-stage verification. Detailed query/transformation correctness belongs to the Chapter 8 Design/Build guidance.
- The planned Appendix B will hold detailed cloud-networking trade-offs. Use current provider documentation and the responsible owner for product-specific behavior.

## Authority and safety boundaries

- A security/privacy discussion or control recommendation is not legal advice, a compliance certification, or approval to alter a real system. Send jurisdiction-specific questions to the authorized legal/privacy owner and use current, authoritative sources.
- Do not ask users to paste sensitive records, secrets, or credentials into chat. Prefer synthetic examples and minimum-necessary evidence.
- Do not create runbooks, control attestations, policy statements, or status trackers that imply approval or implementation when none exists.
- Stage-specific design-document approval, implementation requests, and authorization for remote or production effects are distinct. Keep each permission explicit and limited to its stated scope.
- If an incident is suspected, route promptly to the organization's incident/security owners. Do not make notification decisions or destructive changes beyond the authority and scope granted by those owners.
