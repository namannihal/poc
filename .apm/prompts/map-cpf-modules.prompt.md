---
name: map-cpf-modules
description: >-
  MigrateKit wrapper. Maps services to CPF Terraform modules by delegating to the
  IaC_Terraform_Agent_4LMP map-cpf-modules prompt. Preserves the toolkit's
  new-vs-migration mode and version review (A/B/C) gates, then records outputs.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /map-cpf-modules (MigrateKit wrapper)

Reuse the IaC toolkit's CPF mapping. Do not re-implement module selection or
version resolution.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`. If
   `phases.module_plan.status == done`, skip (idempotent) unless
   `--rerun module_plan`.
2. **Precondition:** `arch/<app-slug>-requirements.md` must exist. If not, recommend
   `/analyse-sad` first.
3. **Delegate** to the IaC toolkit prompt `map-cpf-modules` (dependency
   `<org>/agents/IaC_Terraform_Agent_4LMP`).
   - **Preserve its human gates verbatim:** new-vs-migration detection and the
     version review choices **A / B / C**. Surface them to the user; never auto-answer.
4. **Capture outputs** in `.migrate-kit/<app-slug>/arch/`:
   - `arch/<app-slug>-module-plan.md` (always)
   - `arch/<app-slug>-migration-report.md` (migration mode only)
5. **Update manifest:** `phases.module_plan.status = done`, add artifacts,
   record `mode` (`new` | `migration`), set `next_step = /generate-iac-scaffolding`,
   and `updated`.

## Output

- `arch/<app-slug>-module-plan.md`
- `arch/<app-slug>-migration-report.md` (migration mode only)
