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

1. **Resolve app + working folder.** Ensure `.migrate-kit/<app-slug>/` exists;
   read `migration.manifest.yaml`. If `phases.extract.status == done`, skip
   (idempotent) unless `--rerun extract` was passed.
2. **Locate the SAD** at `input/source-sad.docx` (or the path in `sad_source`). If
   the SAD is already Markdown (`input/source-sad.md`), skip extraction: copy it to
   `intake/source-sad.md` and mark the phase done.
3. **Delegate** to the IaC toolkit skill `extract-sad-to-markdown` (from dependency
   `<org>/agents/IaC_Terraform_Agent_4LMP`). If the toolkit is not resolvable,
   stop and ask the user to install the dependency — do not parse the `.docx` yourself.
4. **Place output** at `.migrate-kit/<app-slug>/intake/source-sad.md` (never modify
   the original in `input/`).
5. **Update manifest:** `phases.extract.status = done`, add the artifact, set
   `sad_source`, `next_step = /analyse-sad`, and `updated`.

## Output

- `intake/source-sad.md`
