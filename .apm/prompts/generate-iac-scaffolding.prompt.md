---
name: generate-iac-scaffolding
description: >-
  MigrateKit wrapper. Generates the Terraform IaC scaffold by delegating to the
  IaC_Terraform_Agent_4LMP generate-iac-scaffolding prompt, then maps the output
  into the canonical <app>/infrastructure/ layout and records it in the manifest.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /generate-iac-scaffolding (MigrateKit wrapper)

Reuse the IaC toolkit's scaffolding. Do not author Terraform generation logic.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`.
2. **Precondition:** `arch/<app-slug>-module-plan.md` must exist. If not, recommend
   `/map-cpf-modules` first.
3. **Delegate** to the IaC toolkit prompt `generate-iac-scaffolding` (dependency
   `lseg/lseg-agents/IaC_Terraform_Agent_4LMP`).
   - **Preserve its gates verbatim:** registry mode (Artifactory/GitLab) and
     topology (A/B/C). Surface to the user; never auto-answer.
   - If the toolkit's migration report is present, honour the developer-confirmed
     versions (do not override with newer tags).
4. **Map output to the canonical layout** per `pipeline-format.instructions.md`:
   ```
   <app>/infrastructure/
   ├── terraform/{main,data,variable,providers}.tf
   └── environments/<env>/infra.tfvars     # one per environment from intake
   ```
   Do not emit the pipeline here — that is `/generate-pipeline`'s job.
5. **Validation gate (mandatory):** run
   `terraform init -backend=false` + `terraform validate`; resolve all errors
   before finishing.
6. **Update manifest:** `phases.iac.status = done`, list written files,
   set `next_step = /generate-pipeline`.

## Output

- `<app>/infrastructure/terraform/*`
- `<app>/infrastructure/environments/<env>/infra.tfvars`
