---
name: generate-iac-scaffolding
description: >-
  MigrateKit wrapper. Generates the Terraform IaC scaffold by delegating to the
  IaC_Terraform_Agent_4LMP generate-iac-scaffolding prompt, then maps the output
  into the canonical <app-slug>/infrastructure/ layout and records it in the manifest.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /generate-iac-scaffolding (MigrateKit wrapper)

Reuse the IaC toolkit's scaffolding. Do not author Terraform generation logic.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`. If
   `phases.iac.status == done`, skip (idempotent) unless `--rerun iac`.
2. **Precondition:** `arch/<app-slug>-module-plan.md` must exist. If not, recommend
   `/map-cpf-modules` first.
3. **Delegate** to the IaC toolkit prompt `generate-iac-scaffolding` (dependency
   `<org>/agents/IaC_Terraform_Agent_4LMP`).
   - **Preserve its gates verbatim:** registry mode (Artifactory/GitLab) and
     topology (A/B/C). Surface to the user; never auto-answer.
   - If the toolkit's migration report is present, honour the developer-confirmed
     versions (do not override with newer tags).
4. **Map output to the canonical layout** per `pipeline-format.instructions.md`:
   ```
   <app-slug>/infrastructure/
   ├── terraform/{main,data,variable,providers}.tf
   └── environments/<env>/infra.tfvars     # one per environment from the SAD
   ```
   Do not emit the pipeline here — that is `/generate-pipeline`'s job. Do not run
   Terraform here either — validation is a separate MigrateKit phase.
5. **Update manifest:** `phases.iac.status = done`, list written files, set
   `next_step = validate` (MigrateKit runs `terraform init -backend=false` +
   `terraform validate`), and `updated`.

## Output

- `<app-slug>/infrastructure/terraform/*`
- `<app-slug>/infrastructure/environments/<env>/infra.tfvars`
