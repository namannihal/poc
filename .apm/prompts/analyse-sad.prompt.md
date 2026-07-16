---
name: analyse-sad
description: >-
  MigrateKit wrapper. Produces the requirements brief by delegating to the
  IaC_Terraform_Agent_4LMP analyse-sad prompt. Accepts either a SAD (.md) or the
  normalized intake schema as input, then records the brief in the manifest.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /analyse-sad (MigrateKit wrapper)

Reuse the IaC toolkit's SAD analysis. Do not re-derive requirements logic.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`.
2. **Resolve intake** per `intake-contract.instructions.md`:
   - If `intake/intake.normalized.md` exists, use it as the input.
   - Else use `arch/sad-analysis.md` (or the SAD `.md` provided).
   - If only a `.docx` exists, stop and recommend `/extract-sad-to-markdown`.
3. **Delegate** to the IaC toolkit prompt `analyse-sad` (dependency
   `lseg/lseg-agents/IaC_Terraform_Agent_4LMP`), pointing its input at the resolved
   source and its output at `.lseg-migration/<app-name>/arch/`.
4. **Capture output** at `arch/<app-slug>-requirements.md`.
5. **Update manifest:** `phases.intake.status = done`,
   `phases.requirements.status = done`, add the artifact,
   set `next_step = /map-cpf-modules`.

## Output

- `arch/<app-slug>-requirements.md`
