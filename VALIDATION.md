# MigrateKit — Validation checklist

Manual acceptance checklist for the SAD-only bootstrap workflow. (This is a
prompt/agent package, so "tests" = steps you run and verify by hand.)

## 1. Install-time safety
- [ ] `apm install <org>/migrate-kit` creates **no** files/folders in the workspace.
- [ ] No `.migrate-kit/` folder exists until `/migrate-app` is run.

## 2. Bootstrap folder creation
- [ ] `/migrate-app demo-app` creates `.migrate-kit/demo-app/` with
      `migration.manifest.yaml`, `input/`, `intake/`, `arch/`, and
      `demo-app/infrastructure/`.
- [ ] Manifest has `app_name`, `app_slug: demo-app`, `workflow: sad-only`, all
      phases `not_started`, and an `updated` timestamp.

## 3. SAD placement
- [ ] With no SAD present, `/migrate-app demo-app` stops and instructs the user to
      place the SAD at `.migrate-kit/demo-app/input/source-sad.docx`.
- [ ] `/migrate-app demo-app ./path/to/sad.docx` copies it to
      `input/source-sad.docx` and records `sad_source`.
- [ ] Re-supplying a SAD path does **not** overwrite an existing `input/source-sad.*`.

## 4. Sequential execution
- [ ] With the SAD present, phases run in order:
      extract → analyse → module_plan → iac → validate → pipeline.
- [ ] A `.md` SAD skips the `extract` phase.

## 5. Dependency delegation
- [ ] extract / analyse / module_plan / iac delegate to `IaC_Terraform_Agent_4LMP`.
- [ ] If the toolkit is not resolvable, the flow stops and asks to install it
      (no re-implementation).

## 6. Decision gates preserved
- [ ] new-vs-migration prompt appears (module_plan) and is not auto-answered.
- [ ] CPF version review A/B/C appears (module_plan).
- [ ] Artifactory-vs-GitLab registry prompt appears (iac).
- [ ] mono / multi / micro topology prompt appears (iac).

## 7. Terraform validation
- [ ] `validate` runs `terraform init -backend=false` then `terraform validate` in
      `demo-app/infrastructure/`.
- [ ] Missing `terraform` CLI → phase `blocked` + install instruction.
- [ ] `terraform validate` failure → phase `blocked` + errors surfaced + stop.

## 8. Pipeline generation
- [ ] `pipeline` writes `.gitlab-ci.yml`, `ci/module.yml`, `ci/variables.yml` in
      canonical two-tier form, no `bams-upload` stage, app-specific values as
      placeholders.

## 9. Manifest updates
- [ ] After every phase the manifest updates status, artifacts, `sad_source`,
      `next_step`, and `updated`.

## 10. Resume & re-run
- [ ] `/migrate-app resume demo-app` continues from the first incomplete/blocked phase.
- [ ] A plain re-run `/migrate-app demo-app` behaves like resume (skips `done` phases).
- [ ] `/migrate-app demo-app --rerun analyse` re-runs only `analyse`.

## 11. Idempotency
- [ ] Re-running does not duplicate artifacts.
- [ ] Completed phases are not re-run unless via `--rerun`.

## 12. No AI Migrate / no /assess
- [ ] No command, prompt, instruction, or template references AI Migrate intake or `/assess`.
- [ ] A search for `ai migrate`, `/assess`, or `normalized intake` over the package returns nothing.
