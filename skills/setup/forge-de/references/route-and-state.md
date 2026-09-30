# Route and state

Read existing approved project documents as evidence of what has been agreed; check that they are still current. A missing document is **not** proof that no conversation happened, and a note marked draft is **not** sign-off. If the conversation has agreed a decision but not the documentation, summarize it and seek the explicit approval needed before writing that stage's document.

| Request | Route |
| --- | --- |
| Retrieve, reconcile, or persist a confirmed project definition, source contract, design decision, or lesson | `forge-de-brain`; read the target project's current knowledge convention first and do not promote an assumption or unresolved stage decision |
| New initiative, unclear goal, architecture options, data modeling, source choice, batch/stream debate, quality strategy | `forge-de-design` |
| Add or change ingestion/transform/model/serving/quality code, tests, or configuration after the necessary decisions are clear | `forge-de-deliver` |
| Existing pipeline fails with known expected behavior | Diagnose read-only first; use `forge-de-deliver` for authorized fix/test; return to design only if root cause requires a new architecture choice |
| Ongoing data quality/freshness, observability, access/security/privacy/governance, incident readiness, retention, recovery, or cost concern | `forge-de-operate` for read-only-first assessment and ownership/escalation; route any code change to `forge-de-deliver` and keep remote/production authorization separate |
| Ask what a concept means or request a quick comparison | Answer concisely; route to design only if making a consequential project decision |
| Ask for a dashboard or report over existing data without changing the data platform | Use an appropriate visualization/reporting skill; use design here only if definitions, serving, or data contracts must be established |

A stage may be `unexplored`, `under discussion`, `agreed in conversation`, `documented with approval`, or `reopened`. This is a reasoning aid, **not** a reason to create a tracking file. If earlier decisions conflict with fresh evidence, explain the difference and reopen only affected parts.

For a small request, decide only what it depends on. For example, a local SQLite import can proceed after source permissions, target grain, update semantics, and acceptance checks are clear; it need not wait for an enterprise governance handbook. For an urgent request, ask a compact set of high-impact questions, offer a reversible recommendation, and confirm the slice. Avoid postponing all value until every lifecycle stage has been specified.
