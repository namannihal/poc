---
name: generate-pipeline
description: >-
  MigrateKit command. Reshapes the IaC_Terraform_Agent_4LMP GitLab CI output into
  the LSEG canonical two-tier pipeline (parent .gitlab-ci.yml + ci/module.yml +
  ci/variables.yml) with app-specific values as placeholders. No BAMS-upload stage.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /generate-pipeline (MigrateKit)

Reuse and **extend** the IaC toolkit's CI generation. Do not author a fresh
pipeline engine. Follow `pipeline-format.instructions.md` and the skeletons in
`templates/pipeline/`.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`.
2. **Precondition:** `<app>/infrastructure/terraform/` must exist. If not, recommend
   `/generate-iac-scaffolding` first.
3. **Reuse:** obtain the IaC toolkit's GitLab CI output (its
   `generate-iac-scaffolding` step already produces CI). Treat that as the source
   of truth for job/stage intent.
4. **Reshape into the canonical two-tier format** using `templates/pipeline/`:
   - `<app>/infrastructure/.gitlab-ci.yml` — parent: `tag` + `environment` stages,
     `0-create-tag`, and one `deploy-<env>` job per environment from the intake.
   - `<app>/infrastructure/ci/module.yml` — child: `vault` -> `terraform-apply` ->
     `terraform-destroy` -> `terraform-state-unlock`. **No `bams-upload` stage.**
   - `<app>/infrastructure/ci/variables.yml` — pinned template versions + TF core +
     GCF/Trivy vars.
5. **Placeholders only:** emit `<APP_ID>`, `<AZURE_SUBSCRIPTION_NAME>`,
   `<TF_STATE_RG>`, `<TF_STATE_STORAGE_ACCOUNT>`, `<KEYVAULT_NAME>`, `<RUNNER_TAG>`,
   `<TF_CORE_TEMPLATE_VERSION>`, `<VAULT_SERVICE_TEMPLATE_VERSION>`, `<TAGGER_REF>`,
   `<APP_SLUG>`. Do not fetch real values in v0.
6. **Update manifest:** `phases.pipeline.status = done`, list written files,
   set `next_step = null` (baseline flow complete).

## Output

- `<app>/infrastructure/.gitlab-ci.yml`
- `<app>/infrastructure/ci/module.yml`
- `<app>/infrastructure/ci/variables.yml`
