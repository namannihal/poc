# MigrateKit

**MigrateKit** orchestrates an application's Azure LMP migration from a **SAD
document only**, inside GitHub Copilot / VS Code. It is a **thin orchestrator** —
it reuses the existing **`IaC_Terraform_Agent_4LMP`** toolkit for all migration
substance and adds a resumable, idempotent, manifest-tracked workflow around it.

You run one command, drop your SAD into a folder, and MigrateKit sequences the
phases end-to-end.

- **No AI Migrate, no `/assess`** in this version — SAD-only.
- **Install-time safe** — nothing touches your workspace during `apm install`;
  scaffolding happens only when you first run `/migrate-app`.

See [ARCHITECTURE.md](ARCHITECTURE.md) and the [VALIDATION.md](VALIDATION.md) checklist.

---

## Install (APM)

```bash
apm install <org>/migrate-kit
```

This resolves the `IaC_Terraform_Agent_4LMP` dependency transitively (see
[`apm.yml`](apm.yml)). After install, the commands appear in VS Code Copilot Chat.
**Installation makes no workspace changes.**

---

## Quick start

```text
# 1. Bootstrap the working folder for your app
/migrate-app my-app

# 2. MigrateKit tells you to drop your SAD here:
#    .migrate-kit/my-app/input/source-sad.docx

# 3. Run again to execute the migration phases
/migrate-app my-app
```

Or provide the SAD inline (copied into `input/`, never overwriting an existing one):

```text
/migrate-app my-app ./docs/my-app-sad.docx
```

Resume or re-run:

```text
/migrate-app resume my-app            # continue from the first incomplete phase
/migrate-app my-app --rerun analyse   # re-run one completed phase
```

`<app-slug>` is your app name lowercased with non-alphanumeric runs replaced by
`-` (e.g. `My App` -> `my-app`).

---

## Commands

| Command | Purpose | Reuse mode |
|---|---|---|
| `/migrate-app <app> [<sad-path>]` | Bootstrap, place SAD, run/continue the flow | Orchestrator |
| `/migrate-app resume <app>` | Resume from first incomplete phase | Orchestrator |
| `/migrate-app <app> --rerun <phase>` | Re-run one completed phase | Orchestrator |
| `/extract-sad-to-markdown` | SAD `.docx` → Markdown | Delegates to IaC toolkit |
| `/analyse-sad` | Extracted SAD → requirements brief | Delegates to IaC toolkit |
| `/map-cpf-modules` | Requirements → CPF module plan (gates) | Delegates to IaC toolkit |
| `/generate-iac-scaffolding` | Module plan → Terraform scaffold (gates) | Delegates to IaC toolkit |
| `/generate-pipeline` | Reshape toolkit CI → canonical GitLab pipeline | Reuses + extends |

The **validate** phase (`terraform init -backend=false` + `terraform validate`) is
MigrateKit-owned and runs between IaC scaffolding and pipeline generation.

---

## Phase sequence

`extract → analyse → module_plan → iac → validate → pipeline`

1. **extract** — SAD `.docx` → `intake/source-sad.md` (skipped if the SAD is `.md`).
2. **analyse** → `arch/<app-slug>-requirements.md`.
3. **module_plan** → `arch/<app-slug>-module-plan.md` (+ migration report). *Gates:
   new-vs-migration, CPF version review A/B/C.*
4. **iac** → `<app-slug>/infrastructure/terraform/*` + `environments/<env>/`.
   *Gates: Artifactory-vs-GitLab registry, mono/multi/micro topology.*
5. **validate** — `terraform init -backend=false` + `terraform validate` (missing
   CLI or failure → phase `blocked`, surfaced, stop).
6. **pipeline** → canonical two-tier GitLab CI (no `bams-upload`; placeholders).

It is **resumable** (continue from the first incomplete phase) and **idempotent**
(never overwrites your SAD, never duplicates artifacts, never re-runs a completed
phase unless you pass `--rerun <phase>`).

---

## Package layout

```
migrate-kit/
├── apm.yml
├── README.md
├── ARCHITECTURE.md
├── VALIDATION.md
└── .apm/
    ├── agents/
    │   └── migrate-kit.agent.md
    ├── prompts/
    │   ├── migrate-app.prompt.md
    │   ├── extract-sad-to-markdown.prompt.md
    │   ├── analyse-sad.prompt.md
    │   ├── map-cpf-modules.prompt.md
    │   ├── generate-iac-scaffolding.prompt.md
    │   └── generate-pipeline.prompt.md
    ├── instructions/
    │   ├── workflow.instructions.md
    │   └── pipeline-format.instructions.md
    └── templates/
        ├── migration.manifest.yaml
        └── pipeline/
            ├── .gitlab-ci.yml
            └── ci/
                ├── module.yml
                └── variables.yml
```

---

## Working folder & artifacts

All outputs are repo files under a per-app working folder (created on first run):

```
.migrate-kit/<app-slug>/
├── migration.manifest.yaml      # phase state — source of truth
├── input/
│   └── source-sad.docx          # you place this (or source-sad.md)
├── intake/
│   └── source-sad.md            # extracted SAD markdown
├── arch/
│   ├── <app-slug>-requirements.md
│   ├── <app-slug>-module-plan.md
│   └── <app-slug>-migration-report.md   # migration mode only
└── <app-slug>/infrastructure/           # canonical IaC + pipeline
    ├── .gitlab-ci.yml
    ├── ci/{module.yml,variables.yml}
    ├── environments/<env>/infra.tfvars
    └── terraform/{main,data,variable,providers}.tf
```

> App-specific pipeline values (subscriptions, TF state, Key Vault, runner tags,
> environments) are emitted as **placeholders** for manual completion. See
> [`.apm/instructions/pipeline-format.instructions.md`](.apm/instructions/pipeline-format.instructions.md).
