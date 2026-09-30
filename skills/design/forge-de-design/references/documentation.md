# Documentation after stage approval

A document is a record of an agreed decision, not a way to manufacture agreement. First discuss and summarize the stage **in chat**; write only after the user explicitly confirms its decision and scope. Do not create a draft file, status record, empty directory, or ADR in the target project while the stage remains open. An implementation plan in chat is not a substitute for design consent.

For a project with no established layout, suggest `docs/data-engineering/<initiative>/<stage>.md` **after approval**; honor the project's existing convention instead when present. One concise document per approved stage is enough. Do not create empty files for later stages. If the user asks for a different artifact format, use the appropriate available tool and its publishing/confirmation rules.

Suggested contents (adapt to the stage and project):

1. **Outcome and scope:** consumer, decision/action and success measure; in/out of scope.
2. **Evidence and constraints:** verified facts with source/date where useful, plus clearly labeled assumptions and unknowns.
3. **Decision:** agreed design, owner and date of confirmation; data grain, interfaces and reliability/freshness contract when relevant.
4. **Reasoning:** meaningful alternatives, trade-offs, costs and reason for the choice; distinguish architecture from tool selections.
5. **Risks and checks:** security/privacy, data quality, failure/recovery, cost, how to verify the result and triggers for revisiting the decision.
6. **Dependencies:** links to earlier approved stage documents and downstream decisions still pending. Do not state an unapproved stage as decided.

A compact stage summary may be enough for a small slice; avoid a long template filled with “TBD.” Use an ADR only when the decision is difficult to reverse, real alternatives existed, and future maintainers need the rationale. Never store secrets, private rows, production credentials, or the full source book.

When a decision changes, show the prior decision, new evidence, blast radius and proposed revision **in chat**, seek renewed approval for the affected stage, then update only the approved document(s). Link to evidence rather than silently rewriting history.
