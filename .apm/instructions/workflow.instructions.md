---
name: workflow
description: >-
  The MigrateKit SAD-only migration workflow: folder bootstrap, slug rules, the
  phase sequence and their artifacts, the manifest schema, idempotency/resume
  rules, decision gates, and Terraform validation behaviour. This version does not
  use AI Migrate intake or /assess.
applyTo: "**/.migrate-kit/**"
---

# MigrateKit Workflow (SAD-only)

MigrateKit orchestrates a migration from a **SAD document only**. It owns folder
creation, SAD placement, phase sequencing, path mapping, manifest updates, and
canonical pipeline reshaping. All substance is delegated to
`IaC_Terraform_Agent_4LMP`. **No AI Migrate intake. No `/assess`.**

## Slug rules

`<app-slug>` = app name lowercased; every run of non-alphanumeric characters
becomes a single `-`; leading/trailing `-` trimmed.
Examples: `My App` -> `my-app`, `Payments_API v2` -> `payments-api-v2`,
`app-1234` -> `app-1234`.

## Working folder (bootstrapped on first `/migrate-app <app>`)

```
.migrate-kit/<app-slug>/
├── migration.manifest.yaml
├── input/                       # raw SAD: source-sad.docx (or .md)
├── intake/                      # extracted SAD markdown: source-sad.md
├── arch/                        # <app-slug>-requirements.md, -module-plan.md, -migration-report.md
└── <app-slug>/infrastructure/   # terraform/ + ci/ + environments/ + .gitlab-ci.yml
```

Bootstrapping happens **only** when the user runs `/migrate-app` — never at
`apm install` time.

## Phases (order + outputs)

| # | Phase | Owner | Output |
|---|---|---|---|
| 1 | `extract` | IaC toolkit | `intake/source-sad.md` (skip if SAD already `.md`) |
| 2 | `analyse` | IaC toolkit | `arch/<app-slug>-requirements.md` |
| 3 | `module_plan` | IaC toolkit (**gates**) | `arch/<app-slug>-module-plan.md` (+ `-migration-report.md`) |
| 4 | `iac` | IaC toolkit (**gates**) | `<app-slug>/infrastructure/terraform/*`, `environments/<env>/infra.tfvars` |
| 5 | `validate` | MigrateKit | `terraform init -backend=false` + `terraform validate` |
| 6 | `pipeline` | MigrateKit (reshape) | `<app-slug>/infrastructure/.gitlab-ci.yml`, `ci/module.yml`, `ci/variables.yml` |

## Manifest schema

`migration.manifest.yaml` fields: `app_name`, `app_slug`, `workflow: sad-only`,
`sad_source`, `phases.<phase>.{status, artifacts}` (`module_plan` also has `mode`),
`next_step`, `updated`. Phase status is one of
`not_started | in_progress | done | blocked`. Update it after **every** phase.

## Idempotency & resume

- Never overwrite `input/source-sad.*`.
- Never duplicate artifacts; write in place.
- Never re-run a `done` phase unless named in `--rerun <phase>`.
- Resume from the first `not_started` / `in_progress` / `blocked` phase.

## Decision gates (never auto-answered)

Preserve every IaC toolkit gate and surface choices verbatim:

- new deployment vs migration (`module_plan`);
- CPF module version review — A / B / C (`module_plan`);
- Artifactory vs GitLab registry (`iac`);
- mono-repo vs multi-repo vs micro-stack topology (`iac`).

## Terraform validation

Run in `<app-slug>/infrastructure/`. If the `terraform` CLI is missing → phase
`blocked` + tell the user to install it. On `validate` failure → phase `blocked`,
surface errors, stop (no forced auto-fix).

## Delegation boundary

If `IaC_Terraform_Agent_4LMP` is unresolvable, stop and ask the user to install
it. Never re-implement extraction, analysis, CPF mapping, or Terraform / CI logic.
