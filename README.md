# Data Forge — Data Engineering Skills

> **Public source:** [DKSang/data-forge](https://github.com/DKSang/data-forge). Install commands below use this repository; this README does not claim a versioned release or a passing remote CI run.

Data Forge is a vendor-neutral collection of skills for AI coding agents working with data systems. It helps users design, deliver, and operate systems around the problem—not around a predetermined stack. It supports local, on-premises, cloud, and hybrid projects, from a small ingestion script to a platform serving multiple teams.

## Quick Start

### Install one skill with `npx skills`

```bash
npx skills@latest add DKSang/data-forge --skill forge-de-design
```

Replace `forge-de-design` with the name of the skill you want. Repeat for optional peer skills when a workflow spans multiple roles. Use the installer's prompts to select a supported agent or runtime.

### Install manually from source

If you already have a source checkout, skip the clone step. Otherwise, clone the public repository:

```bash
git clone https://github.com/DKSang/data-forge.git
```

Copy the **entire selected skill directory**—including `SKILL.md` and its `references/`—into the runtime's skills directory.

For example, with Claude Code:

```text
skills/design/forge-de-design/  →  .claude/skills/forge-de-design/
```

Each skill folder is a standalone install unit. Other runtimes may use a different skills directory and must support the `SKILL.md` format. Cloning or downloading this repository alone does not activate skills in a project repository.

## Skills

| Skill | Group | Use it when |
| --- | --- | --- |
| [forge-de](skills/setup/forge-de/SKILL.md) | setup | Starting or resuming a data initiative and deciding what to do next. |
| [forge-de-brain](skills/brain/forge-de-brain/SKILL.md) | brain | Retrieving or maintaining confirmed project definitions, contracts, and decisions. |
| [forge-de-design](skills/design/forge-de-design/SKILL.md) | design | Clarifying requirements, assessing sources, modeling data, and weighing trade-offs before documenting a stage. |
| [forge-de-deliver](skills/build/forge-de-deliver/SKILL.md) | build | Implementing or testing an explicitly requested change with a clear scope. |
| [forge-de-operate](skills/operate/forge-de-operate/SKILL.md) | operate | Maintaining data quality and trust, observability, security/privacy, incident readiness, and continual improvement. |

`forge-de` routes work; `forge-de-brain` maintains confirmed project context; `forge-de-design` aligns consequential choices stage by stage; `forge-de-deliver` implements agreed requests; and `forge-de-operate` supports ongoing operations. Each skill works within its own scope. Handoffs to peer skills are optional: install additional peers when you need cross-role workflows.

After installation, invoke a skill by name in a compatible runtime or describe the task and let the agent choose a skill.

## Working Principles and Safety

- Start with data users, business decisions, data quality, freshness, and team capabilities. Do not assume cloud, Spark, streaming, or a medallion architecture.
- Discuss consequential trade-offs before writing a design document. Obtain explicit confirmation for each relevant stage; do not create draft design or tracking files while a decision is unapproved.
- **Design approval, an implementation request, and authorization for remote or production effects are separate.** Approving a design stage does not authorize cloud-resource changes, write jobs, material costs, or destructive actions.
- Prefer synthetic data and local tests. Report what was verified and what remains unknown.

See [synthetic examples](examples/scenarios.md). Platform-specific skills can be added after a platform is selected; they do not replace the discussion of goals and trade-offs.

## Update and Uninstall

| Installation method | Update | Uninstall |
| --- | --- | --- |
| `npx skills` | `npx skills update forge-de-design` (replace with the installed skill name). | Run `npx skills remove`; follow the CLI's selection prompt if shown. |
| Manual copy | Replace the installed skill folder with the complete newer folder, including `references/`. | Delete only the installed skill folder. |

The CLI commands apply only to skills installed and managed by `npx skills`. Removing a skill does not remove design documents or project knowledge in the target repository; those belong to that project and follow its conventions.

## Development and Evaluation

Each skill has `name` and `description` frontmatter, a concise `SKILL.md` entrypoint, and focused guidance in `references/`. [BOOK-COVERAGE.md](BOOK-COVERAGE.md) tracks chapter concepts, Data Forge content approvals, and limits on direct source comparison. It is a coverage plan—not a claim that the full book has been covered.

Run the structural checks from a source checkout:

```bash
python -m unittest discover -s tests -v
```

The workflow in `.github/workflows/check.yml` is configured to run the same tests on pushes and pull requests; this describes the workflow configuration, not a verified passing remote run. Synthetic evaluation prompts are in `evals/`. Locally generated workspaces, transcripts, and benchmarks under `.claude/skills/*-workspace/` are excluded from the distributable skill package.

## License and Sources

Data Forge material is licensed under MIT only to the extent its contributors have the rights to license it; see [LICENSE](LICENSE). This license does not grant rights to the book or other third-party material. The repository contains original explanations of concepts from *Fundamentals of Data Engineering*; it does not distribute the book's text, images, or tables.
