---
name: intake-contract
description: >-
  The normalized intake contract for MigrateKit. Defines the single Intake Schema
  that every downstream command consumes, and the adapters that map supported
  sources (SAD document, AI Migrate intake report, native /assess) into it.
applyTo: "**/.lseg-migration/**"
---

# MigrateKit Intake Contract

Downstream commands (`/analyse-sad`, `/map-cpf-modules`, `/generate-iac-scaffolding`,
`/generate-pipeline`) must read only the **normalized Intake Schema** — never a
specific tool's raw format. New sources are added by writing a new adapter, with
**zero changes** to the commands.

## Supported sources (v0)

1. **SAD document** (`.docx`/`.md`) — the IaC toolkit's current native input.
2. **AI Migrate Intake report** (external) — via the adapter below. Field mapping
   is TBC until the report format is provided; emit `open_questions` for anything
   unmapped rather than guessing.
3. **Native `/assess` output** — reserved; not implemented in v0 but contract-compatible.

## Normalized Intake Schema

Write both a human file and a machine sidecar:

- `intake/intake.normalized.md` — YAML front-matter + readable summary.
- `intake/intake.normalized.json` — same fields, machine-readable.

Required fields:

| Field | Meaning |
|---|---|
| `app_name` / `app_slug` | Application identity |
| `app_id` / `org_id` | LSEG identifiers (if known) |
| `tech_stack` | Languages, frameworks, runtimes |
| `services` | Azure/cloud services in scope |
| `dependencies` | Upstream/downstream systems |
| `integrations` | External integrations |
| `data_stores` | Databases, storage |
| `environments` | Target environments (e.g. dev, ppr-01, prd-01, prd-02) |
| `target_platform` | Landing zone / target |
| `nfrs` | Non-functional requirements |
| `risks` | Known risks |
| `open_questions` | Anything unresolved or unmapped |

## Adapter rules

- **SAD adapter:** if the source is a SAD, do not pre-normalize here — pass it to
  `/analyse-sad` (which delegates to the IaC toolkit). Record `source: sad` and the
  file path in the manifest.
- **AI Migrate adapter:** map the intake report fields onto the schema above.
  Unmapped values become `open_questions`. Record `source: ai-migrate`.
- **Contract rule:** `/analyse-sad` may consume the SAD directly **or** the
  normalized schema. All later commands consume only the artifact folder + manifest.

## Precedence

If multiple sources are present, prefer an explicit normalized schema, then the AI
Migrate report, then the SAD. Always record the chosen `source` in the manifest.
