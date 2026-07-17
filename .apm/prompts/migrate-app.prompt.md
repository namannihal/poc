---
name: migrate-app
description: >-
  MigrateKit orchestrator command. Bootstraps a per-app working folder, takes the
  SAD you place in input/, then runs the migration phases in order (extract ->
  analyse -> CPF map -> Terraform -> validate -> canonical GitLab pipeline) by
  delegating to IaC_Terraform_Agent_4LMP. Resumable, idempotent, manifest-tracked.
  Usage: `/migrate-app <app>` | `/migrate-app <app> <sad-path>` |
  `/migrate-app resume <app>` | `/migrate-app <app> --rerun <phase>`.
mode: agent
tools: ['read', 'edit', 'search', 'execute', 'agent', 'todo']
---

# /migrate-app (MigrateKit orchestrator)

Drive an application's SAD-only migration from a single command. You **own only**
orchestration — folder creation, SAD placement, phase sequencing, path mapping,
manifest updates, and canonical pipeline reshaping. All migration substance is
produced by the reused `IaC_Terraform_Agent_4LMP` toolkit.

Load `workflow.instructions.md` and `pipeline-format.instructions.md` before acting.

> **Install-time safety:** nothing here runs during `apm install`. The workspace is
> only ever touched when the user explicitly runs `/migrate-app`. Never scaffold or
> write files at install time.
>
> **Scope of this version:** SAD-only. Do **not** use AI Migrate intake or `/assess`.

---

## Argument forms

| Invocation | Behaviour |
|---|---|
| `/migrate-app <app>` | Bootstrap (if new) then run/continue phases. If the folder exists, behaves like resume. |
| `/migrate-app <app> <path-to-sad.(docx\|md)>` | Bootstrap, copy the SAD into `input/`, then run. |
| `/migrate-app resume <app>` | Resume from the first incomplete phase. |
| `/migrate-app <app> --rerun <phase>` | Re-run one already-completed phase in place, then stop. |

`<phase>` is one of `extract | analyse | module_plan | iac | validate | pipeline`.

**App name & slug.** `<app>` is a single token (quote it if it contains spaces).
`<app-slug>` = the name lowercased, every run of non-alphanumeric characters
replaced by a single `-`, leading/trailing `-` trimmed
(e.g. `My App` -> `my-app`, `app-1234` -> `app-1234`).

---

## Step 1 — Resolve app + bootstrap the working folder

Create (idempotently — never clobber existing content) under the current workspace:

```
.migrate-kit/<app-slug>/
├── migration.manifest.yaml      # seeded from templates/migration.manifest.yaml
├── input/                       # you place the SAD here
├── intake/                      # extracted SAD markdown
├── arch/                        # requirements + module plan (+ migration report)
└── <app-slug>/infrastructure/   # canonical Terraform + GitLab pipeline
```

If `migration.manifest.yaml` is missing, seed it from
`templates/migration.manifest.yaml` and set `app_name`, `app_slug`, `updated`.

## Step 2 — Place / locate the SAD

- If a SAD path argument was given: **copy** it to `input/source-sad.docx`
  (or `input/source-sad.md` if Markdown). **Never overwrite** an existing
  `input/source-sad.*` — if one exists, keep the original and say so.
- If no SAD path was given and `input/` has no `source-sad.*`: **stop** and tell
  the user:
  > Place your SAD at `.migrate-kit/<app-slug>/input/source-sad.docx`
  > (or `source-sad.md`), then run `/migrate-app <app>` again.
- Record the SAD location in `sad_source`.

## Step 3 — Run the phases in order (resumable)

Find the **first incomplete phase** from the manifest and run forward from there.
Skip any phase whose status is `done` (idempotent) unless it was named in
`--rerun`. Order and delegation:

| # | Phase | Command / action | Delegates to (IaC toolkit) |
|---|---|---|---|
| 1 | `extract` | `/extract-sad-to-markdown` | `extract-sad-to-markdown` (skip if SAD already `.md`) |
| 2 | `analyse` | `/analyse-sad` | `analyse-sad` |
| 3 | `module_plan` | `/map-cpf-modules` | `map-cpf-modules` — **decision gates** |
| 4 | `iac` | `/generate-iac-scaffolding` | `generate-iac-scaffolding` — **decision gates** |
| 5 | `validate` | MigrateKit runs Terraform (Step 4) | — (MigrateKit-owned) |
| 6 | `pipeline` | `/generate-pipeline` | reshape toolkit CI to canonical format |

After **each** phase, update the manifest: `phases.<phase>.status`, `artifacts`,
`sad_source`, `next_step`, `updated`.

**Decision gates (phases 3 and 4) — never auto-answer.** Preserve all IaC gates:
new-vs-migration; CPF module version review (A/B/C); Artifactory-vs-GitLab
registry; mono-repo vs multi-repo vs micro-stack topology. When a gate is reached,
surface the choices **verbatim** and pause. If the user answers in the same
conversation, continue forward; otherwise the flow is resumable later with
`/migrate-app resume <app>`.

## Step 4 — Validate phase (MigrateKit-owned)

In `<app-slug>/infrastructure/`:

1. Check the `terraform` CLI is available. If not, set `phases.validate.status =
   blocked`, tell the user to install Terraform, and stop.
2. Run `terraform init -backend=false`, then `terraform validate`.
3. On success: `phases.validate.status = done`, `next_step = /generate-pipeline`.
4. On failure: `phases.validate.status = blocked`, record the errors, and **stop**
   for the user to fix. Do not auto-edit Terraform to force a pass.

## Step 5 — Complete

When `pipeline` is done, set `next_step = null` and report the artifact tree.

---

## Idempotency & resume rules

- Never overwrite the original SAD in `input/`.
- Never duplicate artifacts — write in place at deterministic paths.
- Never re-run a `done` phase unless it is named in `--rerun`.
- Resume always continues from the first incomplete (or `blocked`) phase.

## Delegation boundary

If `IaC_Terraform_Agent_4LMP` is not resolvable (not installed / not a workspace
root), stop and ask the user to install the dependency. Do **not** reproduce SAD
parsing, requirements, CPF mapping, or Terraform / CI generation yourself.
