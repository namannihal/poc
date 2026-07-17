---
name: analyse-sad
description: >-
  MigrateKit wrapper. Produces the requirements brief by delegating to the
  IaC_Terraform_Agent_4LMP analyse-sad prompt, reading the extracted SAD Markdown,
  then records the brief in the manifest.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent']
---

# /analyse-sad (MigrateKit wrapper)

Reuse the IaC toolkit's SAD analysis. Do not re-derive requirements logic.

## Steps

1. **Resolve app + working folder;** read `migration.manifest.yaml`. If
   `phases.analyse.status == done`, skip (idempotent) unless `--rerun analyse`.
2. **Resolve input** per `workflow.instructions.md`:
   - Use `intake/source-sad.md` (the extracted / Markdown SAD).
   - If it does not exist, stop and recommend `/extract-sad-to-markdown` first.
3. **Delegate** to the IaC toolkit prompt `analyse-sad` (dependency
   `<org>/agents/IaC_Terraform_Agent_4LMP`), pointing its input at
   `intake/source-sad.md` and its output at `.migrate-kit/<app-slug>/arch/`.
4. **Capture output** at `arch/<app-slug>-requirements.md`.
5. **Update manifest:** `phases.analyse.status = done`, add the artifact,
   set `next_step = /map-cpf-modules`, and `updated`.

## Output

- `arch/<app-slug>-requirements.md`
