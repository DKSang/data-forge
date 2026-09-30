# Security, privacy, and governance

Security and privacy protect people, business commitments, and the availability and integrity of data systems throughout the data lifecycle. Governance connects accountable people, policies, and technical controls so data is used and protected as agreed. Apply controls in proportion to the data, purpose, exposure, and impact of failure—not as a universal checklist or a reason to assume cloud, on-premises, or local systems are inherently safer. This reference is a Data Forge decision aid based on the Chapter 10 coverage plan; the source chapter text was not available in the repository for direct comparison.

For detailed lifecycle, source, ingestion, and serving design, consult the optional `forge-de-design` skill when installed; the approved project contract remains the source of design decisions. For implementation-stage checks, use the optional `forge-de-deliver` skill when installed. This reference remains usable on its own: rely on approved project context and current primary documentation without inferring unapproved design or authorization. The planned Appendix B will cover detailed cloud-networking trade-offs; this reference keeps network exposure at the control-boundary level.

## 1. Start with data, purpose, and risk

A control only makes sense against a data use and a plausible risk. Begin with the specific project slice, not an abstract compliance checklist:

- **Data:** Which fields, records, files, events, derived outputs, logs, and copies are involved? Which are personal, sensitive, confidential, regulated by contract, or otherwise consequential if exposed, altered, lost, or retained too long?
- **Purpose:** What approved task needs each data element? Who can confirm that the purpose and recipients are allowed? Is the data being reused for a new purpose, linked to another source, exported, or passed to an operational target?
- **People and systems:** Which humans, services, devices, providers, and downstream consumers can access or affect the data? Where are trust boundaries crossed?
- **Impact:** What could happen if data is disclosed, altered, unavailable, stale, or deleted incorrectly? Consider people and business operations, not only infrastructure.
- **Lifecycle and copies:** Where is data created, moved, transformed, cached, exported, backed up, used in development, and eventually removed? Which copies or derived datasets inherit the original sensitivity or use restrictions?
- **Existing controls and evidence:** What is already required by organizational policy, contract, provider responsibility, or law? What evidence shows the relevant control is in place and working, rather than merely configured or assumed?

Scale the review to likely harm and exposure. A small local analysis with synthetic data may need a brief access/retention decision; a shared service with sensitive records or production writeback may need named approvers, a threat review, more evidence, and a planned response. Mark unknown data classification, use rights, or provider responsibilities as unresolved; do not invent an authorization or claim that data is safe because it is inside a particular environment. Treat an unknown purpose or field need as an open decision, not a detail to silently assume. Before recommending field-level safeguards, ask concise questions such as: “What decision or task will the weekly review support?” and “Which fields are needed for that task—do you need names/emails to contact people, or can you use aggregate feedback?” Keep the question in chat; do not ask the user to paste sensitive rows as a substitute for an owner-confirmed purpose.

### Use a compact control record

For each material risk, capture only what helps an owner decide and maintain the control:

| Element | Question |
| --- | --- |
| Asset and purpose | Which data or system is in scope, and which approved use does it support? |
| Risk and boundary | What could go wrong, who or what is exposed, and at which handoff or copy? |
| Owner and approver | Who is accountable for the data/use, who operates the control, and who may approve an exception? |
| Control and evidence | What prevents, detects, or limits the event? What evidence or test would show the control is operating? |
| Residual risk | What can still happen, and who is authorized to accept that residual risk? |
| Review trigger | When should the decision be revisited—such as a new data use, recipient, system boundary, provider, incident, or material change? |

A completed checklist is not proof of security. Prefer a small set of controls with clear owners, evidence, and review triggers over a long unmaintained list.

## 2. Assign responsibility and shared ownership

Data protection is a people, process, and technology responsibility. The same person or team may hold several roles in a small project; name the decisions even when the roles are combined.

| Role or party | Decisions and responsibilities to clarify |
| --- | --- |
| **Business/data owner or steward** | Confirms meaning, sensitivity, approved purpose, permitted consumers, retention expectation, and how a use or definition change is approved. |
| **Source or producer owner** | Confirms source rights, contractual limits, delivery behavior, and source-side access or change obligations. See Chapter 5 for the source contract. |
| **Security, privacy, or legal owner** | Advises on risk, organizational controls, privacy requirements, jurisdiction-specific duties, exceptions, and incident escalation within their authority. Do not substitute an engineer's interpretation for their decision. |
| **Platform or service operator** | Explains identity boundaries, administrative access, patching, encryption, audit, network exposure, backup/restore, and provider responsibilities for the actual environment. |
| **Data engineering/build owner** | Implements only approved controls and tests their behavior within the authorized scope; reports gaps rather than claiming controls that were not checked. |
| **Consumer or recipient** | Uses the data for the agreed purpose, protects received copies, follows access and retention limits, and reports suspected misuse or exposure through the agreed route. |
| **Provider or partner** | Supplies only the security/privacy commitments confirmed in the applicable contract and current documentation; the customer's remaining responsibilities must be understood separately. |

For managed services, verify the actual shared-responsibility boundary for each service and data path. A provider's certification, service name, or “managed” label does not show that the customer's identity settings, data use, logging, retention, or recovery are appropriate. Local and on-premises environments also have owners and boundaries—for example, device access, operating-system patching, backups, and physical handling.

Keep approval distinct from implementation. A data owner's permission to use data does not automatically grant an engineer access, approve an infrastructure change, authorize a production test, or permit sending data to another recipient. Record who can approve each decision and which effects need separate authorization.

## 3. Make privacy and permitted use operational

Privacy starts before collection and continues through reuse, sharing, retention, and deletion. Ask what is needed for the authorized purpose and what can be left out:

- **Minimize collection and exposure:** Collect, copy, expose, and retain only fields and records justified by the use. Prefer a less sensitive field, narrower population, aggregation, or synthetic data when it still meets the need.
- **Limit purpose and recipients:** State which users, services, and destinations may use the data and for what purpose. Reassess before linking sources, adding a new recipient, training a model, exporting data, or using derived outputs for a materially different purpose.
- **Identify sensitive attributes and inferences:** Direct identifiers are not the only risk. Combinations of fields, repeated observations, small groups, location, behavior, or model-derived scores can reveal sensitive information or enable re-identification.
- **Check sharing and location boundaries:** Confirm relevant source, contractual, organizational, and residency restrictions with the accountable owner before sharing, copying across tenants, or moving data across boundaries. Keep provider-specific network/location design in the appropriate source and networking guidance.
- **Protect non-production use:** Prefer synthetic fixtures. Do not copy production-sensitive rows into local notebooks, test exports, screenshots, logs, tickets, or evaluation workspaces without a separately authorized, narrowly scoped exception and suitable protections.
- **Preserve deletion and correction paths:** Know how a correction, opt-out, or approved deletion request can be traced to derived datasets and recipients. The authorized privacy/legal owner determines applicable obligations and exceptions; engineers verify only the technical scope they are responsible for.

Masking, tokenization, hashing, aggregation, or removal of direct identifiers may reduce exposure but does not by itself establish that data is anonymous or safe for every reuse. Consider linkage and re-identification risk in context, especially when combining datasets or sharing small cohorts. Do not promise anonymity, consent, or legal compliance without evidence and the appropriate owner’s determination.

### Resolve legal and policy questions with the right owner

Laws, contractual duties, data-subject rights, breach-notification rules, and retention requirements vary by jurisdiction, data type, organization, and time. This reference is not legal advice. When a design depends on a legal basis, notice/consent, cross-border transfer, retention period, individual request, or incident-notification deadline, record the question and have the authorized legal/privacy owner confirm the current requirement. Do not invent a generic number of days, a universal lawful basis, or a compliance conclusion.

## 4. Control identity and access through its lifecycle

Use the least privilege necessary for a defined task, for both people and workloads. Design access around named identities and data boundaries, not shared credentials or broad convenience roles.

Consider, where relevant:

- **Authentication and authorization:** Which identity is being verified, which actions and data are allowed, and where is the decision enforced? Verify the actual control path rather than assuming a network location or private workspace is sufficient.
- **Human access:** Which roles need access, at what scope, and for how long? Separate ordinary use from administration; use time-bounded or just-in-time elevation if supported and appropriate.
- **Workload/service identities:** Give jobs, applications, and integrations their own scoped identity where feasible. Avoid embedding a person's long-lived credential in a pipeline or notebook.
- **Provisioning and removal:** Who approves access, how is it granted, changed, reviewed, and revoked when a role, contract, purpose, or project ends? Include temporary, emergency, vendor, and support access where it exists.
- **Segregation and environment boundaries:** Which development, test, and production roles or tenants are separated? Can a developer inspect real data or trigger a production write from a lower-trust environment?
- **Access reviews and exceptions:** What evidence shows current access is still needed? Who owns an exception, what narrow scope and expiry apply, and when will it be reviewed or removed?

Least privilege is not only a role name. Check effective access through group membership, inherited permissions, service identities, sharing links, exports, and downstream copies where material. A central identity system or catalog helps only when the relevant assets and roles are actually governed by it.

Do not grant or change permissions as part of an unapproved design exercise. Follow the authorized implementation path and obtain separate approval for remote or production access changes.

## 5. Protect credentials, data, and systems

Use controls that match the data path and operating boundary. State the control objective and owner; verify product-specific configuration against current authoritative documentation during implementation.

### Credentials, secrets, and cryptographic keys

- Keep passwords, tokens, certificates, connection strings, and private keys out of source control, notebooks, screenshots, error messages, and logs.
- Limit who or what can retrieve a secret; prefer a supported secret-management mechanism over embedding long-lived values in code or configuration files.
- Define who issues, grants, rotates or renews, revokes, and audits access to each credential or key. Confirm what happens when an owner or provider relationship changes.
- Investigate suspected exposure promptly through the security owner; do not paste the exposed value into a ticket or chat to “verify” it.
- Distinguish key custody and service configuration from the abstract fact that a system advertises encryption. Confirm which data and copies are protected and who can access key material.

Do not prescribe a specific vault, key-management product, algorithm, or rotation interval here. Those choices depend on the environment, supported controls, threat model, policy, and current provider behavior.

### Data in transit and at rest

Identify where sensitive data is stored, transmitted, or temporarily cached. Confirm that the chosen services protect it in the relevant states, that access to protected copies and keys is restricted, and that the protection covers exports, logs, backups, and derived outputs as required. Encryption does not replace authentication, authorization, data minimization, or endpoint security.

For inbound transfer and ingestion modes, consult the optional `forge-de-design` skill when installed for the source-to-destination design boundary; otherwise rely on the approved source/target contract. For cloud network paths, private connectivity, regions, and egress, defer detailed trade-offs to the planned Appendix B and current provider guidance. Verify the actual path and exposure; a private endpoint or encrypted connection alone does not establish that the data is appropriately authorized or scoped.

### Patching, configuration, and exposure

Name who tracks security updates and configuration drift for each relevant component—application, runtime, operating system, managed service, network boundary, and dependency. Decide how vulnerabilities are assessed, prioritized, tested, deployed, and escalated according to impact and organizational policy. Confirm which tasks a provider performs and which remain with the organization. Avoid fixed patch deadlines or hardening settings unless the authorized policy or current system requirement supplies them.

## 6. Audit and monitor without creating another exposure

Audit and monitoring should help answer a defined question: who or what accessed a sensitive boundary, what changed, which control failed, or who needs to respond? Select the events and retention needed for the purpose and risk.

- Identify the system, identity, access/change event, time, outcome, and correlation information necessary for accountability and investigation.
- Assign an owner to review or alert on relevant events and define a path for suspicious access, privilege changes, policy exceptions, or missing telemetry.
- Protect logs from unauthorized changes and access; avoid recording secrets, full sensitive payloads, or identifiers that are not needed to investigate.
- Align log access and retention with privacy, policy, and incident needs. Logs are data copies too and may reveal user behavior or sensitive values.
- Test that alerts reach the intended owner and distinguish an unavailable log source from “no suspicious activity.”

Monitoring is not a substitute for prevention, and an empty alert queue is not evidence that access is correct. Keep implementation-specific event names, retention settings, and detection rules with the selected platform's current documentation and the organization's operational owners.

## 7. Govern retention, deletion, backup, and restoration

Agree the lifecycle for each material dataset and copy, not only the primary table. A useful inventory follows data through source extracts, raw or staged copies, curated outputs, reports, ML training/evaluation sets, caches, exports, logs, and backups where applicable.

For each relevant asset, determine:

- who sets and approves the purpose and retention expectation;
- what event starts or changes the retention period, and what policy, contract, or legal hold must be confirmed;
- how an approved correction or deletion is propagated to derived datasets and known recipients;
- which backups, immutable copies, or logs are exempt, delayed, or subject to a separate expiry process, and who can confirm that policy;
- what evidence can show that the requested technical deletion or expiry completed, and what copies remain outside the team's control.

Distinguish hiding a row from deleting it, and distinguish deletion from a primary table from removal in downstream copies or backups. Do not promise complete erasure when a copy, provider, legal hold, or backup lifecycle has not been verified. Keep detailed source history and derived-output delete mechanics with their respective source, ingestion, storage, and transformation references; this reference owns the cross-lifecycle control and accountable handoff.

Backups help recover from accidental deletion, corruption, or service failure, but they also contain data that must be access-controlled and retired according to the confirmed policy. Define who can create, read, change, and restore backups; what recovery need they serve; how restore is tested safely; and whether the restored environment preserves required access and privacy controls. Select recovery objectives and test depth with system and business owners based on impact rather than importing universal values.

## 8. Prepare for security and privacy incidents

An incident plan needs named people, authority, and a workable first response—not just a monitoring product. At the level of this reference, clarify:

- which events trigger escalation, such as suspected disclosure, exposed credential, unexpected privileged access, data loss, integrity compromise, or an unauthorized outbound transfer;
- who can contain access or pause a job, who preserves evidence, who coordinates with the source/provider, and who can approve recovery;
- how to contact security, privacy, legal, system, data, and business owners, including an after-hours or provider escalation route where relevant;
- where investigation records and artifacts can be stored without spreading sensitive payloads or credentials further;
- how to restore safe operation, validate the recovered data/control, communicate to affected stakeholders, and capture follow-up actions;
- who determines any required external, regulator, customer, or individual notification and timing under the applicable jurisdiction and contract.

Escalate suspected disclosure promptly through the organization's incident route. Do not make a legal notification decision, promise there is no impact, or run destructive containment/restore actions without the appropriate authority. If containment or recovery requires a production access, key, encryption, or deletion change, obtain explicit, scoped authorization from the authorized incident/security owner before execution. When implementation is requested, state the handoff explicitly: after scope authorization, the implementation belongs to the optional `forge-de-deliver` skill when installed; otherwise follow the project's authorized implementation process; this Operate reference does not execute the production action. Neither an incident ticket nor prior design approval alone authorizes the side effect.

**Response pattern when action is not authorized:** a concise opening can say, “No production change has been made, and this reference does not execute production actions.” Then direct prompt escalation to the organization's incident route; name the authorized incident/security owner who must approve the exact target and effects; and state that, after approval, implementation is handed to `forge-de-deliver` rather than executed from this reference. Do not request customer rows, credentials, or secrets in chat.

Detailed contact trees, on-call instructions, severity matrices, forensic procedures, and organization-specific runbooks belong in the responsible operations/security program, not this vendor-neutral reference. Exercises should use synthetic scenarios and avoid production data unless separately authorized.

## 9. Review controls as the system changes

Security and privacy controls decay when purposes, owners, identities, providers, data copies, or architecture change. Revisit relevant decisions when:

- a new data field, population, recipient, sharing route, or use is proposed;
- access roles, service identities, credentials, providers, endpoints, or trust boundaries change;
- data becomes linkable to another source or yields a new sensitive inference;
- a retention/deletion rule, legal/contractual instruction, or backup path changes;
- a vulnerability, incident, audit finding, failed restore, or unexpected data exposure reveals a gap;
- the service is retired, ownership changes, or a consumer no longer needs access.

Record material exceptions with their decision-maker, reason, data/systems affected, compensating controls, expiry or review trigger, and exit/remediation owner. Escalate exceptions that exceed the decision-maker's authority. Avoid maintaining an approval as a permanent substitute for reviewing whether it is still justified.

## 10. Handoffs and scope boundaries

Security, privacy, and governance concerns cross the lifecycle, but the detailed design remains with the owner of each boundary:

- **Chapter 2** maps the lifecycle and introduces security and data management as undercurrents. Use it to orient a cross-stage review, not as a detailed control catalog.
- **Chapter 5** establishes source meaning, source ownership, permitted use, and producer agreements. This reference turns confirmed constraints into lifecycle control and ownership questions; it does not create source rights.
- **Chapter 7** chooses and verifies ingestion mechanics, source limits, transport path, and recovery. This reference covers shared control objectives for identities, credentials, and data protection that apply to that path.
- **Chapter 9** defines the consumer-facing serving contract, access boundary, and reverse-ETL/writeback contract. This reference adds cross-lifecycle privacy/security controls; it does not authorize the outbound action.
- **Appendix B** is planned to cover cloud-network topology and related latency, failure-domain, locality, and transfer-cost trade-offs. This reference only identifies network exposure and when that deeper review is needed.
- **`forge-de-deliver`** implements and tests an explicitly approved design with proportional checks. Design approval does not authorize production access, permission changes, remote jobs, real sensitive-data tests, outbound sharing, or other consequential effects.

**Operate handoff:** `forge-de-operate` uses this user-approved reference for security, privacy, and governance guidance. This document is not evidence that organizational controls, an incident program, or a legal duty have been implemented; apply the approved project and organizational processes to maintain those controls.

## Source and scope note

The repository coverage plan assigns Chapter 10, “Security and Privacy,” to `10-security-privacy-and-governance.md` under Operate and lists minimization, people/process/technology, least privilege, shared responsibility, credentials/keys, patching, encryption, audit/monitoring, network exposure, backup/restore, retention/deletion, incident readiness, and jurisdiction-specific legal duties. The Data Forge reference content was approved by the user on 2026-09-30. The Chapter 10 book text was not present in the repository, so direct source comparison remains unverified. The control-record template, risk-proportionate prompts, privacy examples, role mapping, and handoff language are Data Forge applications based on the coverage plan and current references, not claims that the book prescribes this structure. Do not reproduce the book's prose, figures, or distinctive tables. The optional provenance note and repository-root `BOOK-COVERAGE.md` are maintained in the source checkout; neither is included in a standalone Operate install.
