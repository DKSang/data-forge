# Working on Data Forge

This repository authors reusable, vendor-neutral data-engineering skills under `skills/<group>/<skill>/`, following the organization of DKSang/fabric-engineering-skills. Keep each `SKILL.md` lean and route detailed guidance to owner-local `references/`. Build and review references one topic at a time against the book's chapter concepts; maintain `BOOK-COVERAGE.md` so no major principle is silently lost. Do not bind the suite to a platform, require a fixed medallion architecture, or copy the source book into this repository.

The central behavior to preserve: discuss consequential choices and trade-offs until explicit agreement **for the relevant stage** before writing that stage's design document in the target project. Design approval is separate from an implementation request and from authorization for remote/production side effects. Do not create draft stage files or tracking files before stage approval. Avoid needless gates for small, already-defined fixes.

When changing a skill, run local structural checks and synthetic scenarios including a local project, a cloud project, withheld approval, and an unapproved remote action. Never point test agents at live production systems. Keep generated evaluation workspaces out of the distributable repository. Read `README.md` for skill roles, usage and provenance.
