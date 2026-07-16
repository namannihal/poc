---
app_name: "<app-name>"
app_slug: "<app-slug>"
app_id: "<APP_ID>"
org_id: "<ORG_ID>"
source: "sad"                 # sad | ai-migrate | assess
tech_stack: []                # e.g. ["Java 17", "Spring Boot", "PostgreSQL 15"]
services: []                  # Azure/cloud services in scope
dependencies: []              # upstream/downstream systems
integrations: []              # external integrations
data_stores: []               # databases, storage
environments: []              # e.g. ["dev", "ppr-01", "prd-01", "prd-02"]
target_platform: "Azure LMP"
nfrs: []                      # non-functional requirements
risks: []                     # known risks
open_questions: []            # unresolved / unmapped items
---

# Normalized Intake — <app-name>

> Produced by a MigrateKit intake adapter (see `intake-contract.instructions.md`).
> Downstream commands read this file, never a tool-specific raw format.

## Summary

<one-paragraph description of the application and its migration scope>

## Notes

- Populate every front-matter field above from the source.
- Anything that cannot be mapped from the source goes into `open_questions`.
