# LSEG MigrateKit

**MigrateKit** orchestrates an application's Azure LMP migration lifecycle from
GitHub Copilot / VS Code. It is a **thin orchestrator** — it reuses the existing
[`IaC_Terraform_Agent_4LMP`](../../ai-migration-gitlab-repo/agents/lseg-agents/IaC_Terraform_Agent_4LMP)
toolkit rather than re-implementing IaC or pipeline generation.

Baseline **v0** wires a single capability plug (IaC + pipeline) behind:

- a **normalized intake contract** (SAD today, AI Migrate intake tomorrow — same schema),
- a **repo-file migration manifest** for stateful phase handoff,
- **APM packaging** so the whole thing installs with one command.

See the full PRD: [../baselin-agent/prd.md](../baselin-agent/prd.md).

---

## What you get

| Command | Purpose | Reuse mode |
|---|---|---|
| `/extract-sad-to-markdown` | Convert a SAD `.docx` to Markdown | Delegates to IaC toolkit |
| `/analyse-sad` | SAD (or normalized intake) → requirements brief | Delegates to IaC toolkit |
| `/map-cpf-modules` | Requirements → CPF module plan | Delegates to IaC toolkit |
| `/generate-iac-scaffolding` | Module plan → Terraform IaC scaffold (canonical layout) | Delegates to IaC toolkit |
| `/generate-pipeline` | Reshape the IaC toolkit's GitLab CI output into the LSEG canonical format | Reuses + extends IaC toolkit |

The orchestrator agent `@migrate-kit` runs these as a guided, resumable flow and
keeps `migration.manifest.yaml` up to date.

---

## Install (APM)

```bash
apm install lseg/migrate-kit
```

This resolves the `IaC_Terraform_Agent_4LMP` dependency transitively (see
[`apm.yml`](apm.yml)). After install, the commands appear in VS Code Copilot Chat.

> **v0 note:** app-specific pipeline values (subscriptions, TF state, Key Vault,
> runner tags, environments) are emitted as **placeholders** for manual completion.
> See [`.apm/instructions/pipeline-format.instructions.md`](.apm/instructions/pipeline-format.instructions.md).

---

## Package layout

```
migrate-kit/
├── apm.yml
├── README.md
└── .apm/
    ├── agents/
    │   └── migrate-kit.agent.md
    ├── prompts/
    │   ├── extract-sad-to-markdown.prompt.md
    │   ├── analyse-sad.prompt.md
    │   ├── map-cpf-modules.prompt.md
    │   ├── generate-iac-scaffolding.prompt.md
    │   └── generate-pipeline.prompt.md
    ├── instructions/
    │   ├── intake-contract.instructions.md
    │   └── pipeline-format.instructions.md
    └── templates/
        ├── migration.manifest.yaml
        ├── intake.normalized.md
        ├── intake.normalized.json
        └── pipeline/
            ├── .gitlab-ci.yml
            └── ci/
                ├── module.yml
                └── variables.yml
```

---

## Artifact model

All outputs are repo files under a per-app working folder:

```
.lseg-migration/<app-name>/
├── migration.manifest.yaml
├── intake/
│   ├── intake-report.md
│   └── intake.normalized.(md|json)
├── arch/
│   ├── sad-analysis.md
│   ├── <app-slug>-requirements.md
│   ├── <app-slug>-module-plan.md
│   └── <app-slug>-migration-report.md   # migration mode only
└── <app>/infrastructure/                # canonical IaC + pipeline
    ├── .gitlab-ci.yml
    ├── ci/{module.yml,variables.yml}
    ├── environments/<env>/infra.tfvars
    └── terraform/{main,data,variable,providers}.tf
```
