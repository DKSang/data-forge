---
name: forge-de-brain
description: Retrieve, reconcile, and maintain durable project-specific data-engineering knowledge such as user-approved definitions, source contracts, decisions, and lessons. Use whenever a user asks what the project has already agreed, wants a verified fact remembered or corrected, or needs prior context reconciled before design/build/operate work. Never turn guesses, tentative discussion, or unapproved stage decisions into durable facts; never store secrets or unnecessary sensitive data.
---

# Keep project knowledge reliable

A project knowledge record helps only when its status, source, scope, and owner remain clear. This skill retrieves and maintains confirmed project context; it is not an approval authority, design-document generator, implementation tool, incident handler, or substitute for the target project’s own conventions.

## Use Brain when

- The user asks what is already known or agreed about this project.
- A confirmed definition, source contract, architecture decision, operating constraint, or lesson should be saved for future work.
- New evidence conflicts with an existing record or suggests an approved fact has become stale.
- Another skill needs confirmed project context and the repository already has an accepted source of truth.

For a new consequential design choice, use `forge-de-design`; for an implementation request, use `forge-de-deliver`; for continuing control health or operational response, use `forge-de-operate`. Brain can pass them verified context, but cannot approve their work on the user's behalf.

## Workflow

1. **Identify the target project and its knowledge conventions.** Inspect the user-named project read-only. Look for its existing decision records, contracts, README guidance, naming, and location. Prefer the project's established source of truth; do not assume a `reference/`, `docs/`, or memory path. Never save ordinary project facts into the Data Forge repository or global assistant memory.
2. **Separate retrieval from persistence.** Read and summarize existing approved context without changing it when the user only asks a question. A request to store, revise, or retire a fact is a durable project change; act only within the specific file/location and content the user authorized.
3. **Check status and provenance.** Distinguish owner-confirmed facts from observations, assumptions, recommendations, and open questions. Capture the decision or definition, scope, accountable owner, confirmation/effective date when known, evidence or source reference, affected consumers/stages, and review trigger if useful. Do not infer approval from a draft label, an old decision, a tool choice, or silence.
4. **Handle conflict without silently choosing.** Compare the records and their source, scope, date, and authority. If two credible sources disagree, preserve the approved record, mark the conflict as unresolved in the conversation, and ask the accountable owner to resolve it. Do not promote a tentative claim to an approved fact or overwrite history for convenience.
5. **Confirm durable changes.** If the user has not clearly authorized the exact fact and target location, summarize what would be recorded and ask for confirmation before writing. If the facts are uncertain, ask which owner or evidence can verify them; do not create a “temporary” status file as a substitute for an unapproved decision.
6. **Write minimally in the existing convention.** Preserve unrelated content and record only necessary, non-sensitive knowledge. Prefer a short, searchable entry with a clear status and provenance over a duplicate narrative. Do not store credentials, secrets, raw customer rows, or personal data that is not necessary for the approved project purpose. Use synthetic examples in training and tests.
7. **Verify and report.** Re-read the changed record after writing. State what was added or changed, where it lives, which facts remain uncertain, and whether any conflicting or stale records still need owner review. Do not claim that a record was written unless the write and verification actually succeeded.

## Keep approval boundaries distinct

- Confirming a fact for Brain does not approve a design stage or authorize creation of that stage's project design document.
- Approving a design document does not authorize implementation, access changes, remote jobs, production writes, or meaningful spend.
- An implementation request and authorization for each consequential external or production side effect remain separate.
- If an updated fact changes a previously approved design, explain the affected scope and return the design decision to `forge-de-design` before rewriting its project document.

Keep unapproved stage choices in chat, not in Brain files, memory, draft ADRs, or tracking records. A project can have missing knowledge without needing a new file to advertise that absence.

## Handoffs

- Route unresolved business meaning, architecture, source, serving, or control choices to `forge-de-design` or `forge-de-operate` as appropriate.
- Route implementation and verification of an approved, bounded change to `forge-de-deliver`.
- Pass along the exact approved fact and its provenance; do not convert it into broader authority or unstated technical guarantees.
