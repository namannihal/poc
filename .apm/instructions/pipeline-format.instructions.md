---
name: pipeline-format
description: >-
  The LSEG canonical IaC + GitLab CI/CD pipeline format that MigrateKit emits.
  /generate-iac-scaffolding produces the terraform/ + environments/ layout;
  /generate-pipeline reshapes the IaC toolkit's CI output into the two-tier
  parent + child pipeline below. All app-specific values are placeholders in v0.
applyTo: "**/infrastructure/**"
---

# LSEG Canonical IaC + Pipeline Format

`/generate-iac-scaffolding` + `/generate-pipeline` emit the layout below. It mirrors
the production reference (Datacloud Ingestion `infrastructure/`). All app-specific
values are emitted as **`<PLACEHOLDER>`** tokens for manual completion in v0.
The `bams-upload` stage seen in some reference repos is intentionally **excluded**.

## Directory layout

```
<app>/infrastructure/
├── .gitlab-ci.yml
├── .gitignore
├── README.md
├── ci/
│   ├── module.yml
│   └── variables.yml
├── environments/
│   └── <env>/infra.tfvars          # one folder per environment (dev, ppr-01, prd-01, prd-02, ...)
└── terraform/
    ├── main.tf
    ├── data.tf
    ├── variable.tf
    └── providers.tf
```

## Parent `.gitlab-ci.yml`

- Includes the shared **tagger** template; defines `.deploy-shared` which triggers
  the child `ci/module.yml` with `strategy: depend` and variable forwarding.
- Stages: `tag`, `environment`.
- `0-create-tag` auto-tags on `main` push.
- One **deploy job per environment** with env-specific placeholders and `when: manual`.

See `templates/pipeline/.gitlab-ci.yml` for the full skeleton.

## Child `ci/module.yml`

- Includes **terraform-core** (`templates/azure.yml`) and **vault-service** templates
  + local `ci/variables.yml`.
- Stages: `vault`, `terraform-apply`, `terraform-destroy`, `terraform-state-unlock`
  (**no `bams-upload`**).
- Runs merge-request pipelines via a `workflow` rule.

See `templates/pipeline/ci/module.yml`.

## `ci/variables.yml`

Pinned template versions, terraform-core wiring, and GCF/Trivy compliance variables —
all app-specific fields as placeholders. See `templates/pipeline/ci/variables.yml`.

## Placeholder reference

| Placeholder | Meaning | Likely source |
|---|---|---|
| `<APP_ID>` / `<ORG_ID>` | LSEG app / org identifier | intake / SAD |
| `<env>` list | Environments (dev, ppr-01, prd-01, prd-02) | intake / SAD |
| `<AZURE_SUBSCRIPTION_NAME>` | Per-env subscription | platform onboarding |
| `<TF_STATE_RG>` / `<TF_STATE_STORAGE_ACCOUNT>` | Terraform remote-state backend | platform onboarding |
| `<KEYVAULT_NAME>` | Per-env Key Vault | platform onboarding |
| `<RUNNER_TAG>` | Per-env GitLab runner tag | platform onboarding |
| `<TF_CORE_TEMPLATE_VERSION>` / `<VAULT_SERVICE_TEMPLATE_VERSION>` / `<TAGGER_REF>` | Pinned shared-CI template refs | CI standards |
| `<APP_SLUG>` | Short app name used in state key | intake / SAD |

## `/generate-pipeline` procedure

1. Read the module plan and the generated `terraform/` + `environments/` from the
   IaC scaffold.
2. Ask the IaC toolkit's scaffolding logic for its CI output (reuse — do not author
   fresh Terraform/CI logic).
3. Reshape that output into the two-tier format above using the templates in
   `templates/pipeline/`, one `deploy-<env>` job per environment from the intake.
4. Emit app-specific values as placeholders (do not fetch real subscription / state
   / vault values in v0).
5. Update the manifest: `phases.pipeline.status = done`, list the written files.
