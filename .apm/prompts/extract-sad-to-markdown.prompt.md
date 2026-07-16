---
name: extract-sad-to-markdown
description: >-
  MigrateKit wrapper. Converts a SAD .docx to Markdown by delegating to the
  IaC_Terraform_Agent_4LMP extract-sad-to-markdown skill, then records the output
  in the migration manifest. Run this only when the SAD is a Word document.
mode: agent
tools: ['read', 'edit', 'search', 'execute']
---

# /extract-sad-to-markdown (MigrateKit wrapper)

Reuse the IaC toolkit's SAD extraction. Do not re-implement `.docx` parsing.

## Steps

1. **Resolve app + working folder.** Ensure `.lseg-migration/<app-name>/` exists;
   read `migration.manifest.yaml`.
2. **Locate the SAD `.docx`** (from the user argument or the app's `arch/` folder).
3. **Delegate** to the IaC toolkit skill `extract-sad-to-markdown` (from dependency
   `lseg/lseg-agents/IaC_Terraform_Agent_4LMP`). If the toolkit is not resolvable,
   stop and ask the user to install the dependency — do not parse the `.docx` yourself.
4. **Place output** at `.lseg-migration/<app-name>/arch/sad-analysis.md`.
5. **Update manifest:** `phases.intake.status = in_progress`, add the artifact,
   set `next_step = /analyse-sad`.

## Output

- `arch/sad-analysis.md`
