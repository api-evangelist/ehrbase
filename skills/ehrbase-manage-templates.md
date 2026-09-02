---
name: ehrbase-manage-templates
description: >-
  Load, inspect and retire the openEHR operational templates that define what an EHRbase
  server will accept - including the web-template and example projections that make the
  API usable, and the admin-only deletion path.
generated: '2026-09-02'
method: generated
source: >-
  openapi/_original/ehrbase-api-openapi.json +
  https://docs.ehrbase.org/docs/EHRbase/openEHR-Introduction/Template +
  https://docs.ehrbase.org/docs/EHRbase/Explore/Admin-REST
api: EHRbase openEHR REST API + Admin API
operations:
  - getTemplatesClassic
  - createTemplateClassic
  - getTemplateClassic
  - getWebTemplate
  - getTemplateExample
  - getTemplatesNew
  - createTemplateNew
  - getTemplateNew
  - updateTemplate
  - deleteTemplate
  - deleteAllTemplates
---

# Manage operational templates

A template is the contract for clinical content. EHRbase will reject any composition that
does not validate against the operational template it names, so templates are the schema
layer of this API — and they are deployment-scoped, not per-EHR.

## 1. List what is loaded

```
GET /rest/openehr/v1/definition/template/adl1.4
```

`getTemplatesClassic` returns each template's `template_id`, `concept`, `archetype_id`
and `created_timestamp`. This endpoint is readable anonymously on the public sandbox —
it is the cheapest way to confirm you are talking to a live EHRbase.

## 2. Upload a template

```
POST /rest/openehr/v1/definition/template/adl1.4
Content-Type: application/xml
```

`createTemplateClassic` takes an ADL 1.4 operational template (OPT, XML) and returns
**201**.

ADL 2 has its own pair (`createTemplateNew` / `getTemplateNew` under
`/definition/template/adl2`), but note the caveat: the docs state the HIP distribution
supports every endpoint **except** ADL2 templates. Do not build on ADL 2 without
confirming your deployment.

## 3. Get the projections that make the template usable

Two EHRbase-specific extensions, and they are the reason integrating against EHRbase is
tolerable:

```
GET /rest/openehr/v1/definition/template/adl1.4/{template_id}/webtemplate
GET /rest/openehr/v1/definition/template/adl1.4/{template_id}/example
```

- `getWebTemplate` flattens the OPT into a node tree with `aqlPath`, input types,
  cardinality, validation rules and terminology constraints per leaf. This is what you
  read to learn the paths for both composition writes and AQL queries.
- `getTemplateExample` returns a fully populated example composition. Start from it.
  Building an openEHR RM document from scratch is not a good use of anyone's time.

Retrieve the raw OPT itself with `getTemplateClassic`
(`GET .../adl1.4/{template_id}`).

## 4. Replace or retire a template — admin only

These live on the Admin API, which is **disabled by default** and requires the admin
role. Every one of them returns 401 unauthenticated and 403 without the role.

```
PUT    /rest/admin/template/{template_id}    # updateTemplate
DELETE /rest/admin/template/{template_id}    # deleteTemplate  -> 202
DELETE /rest/admin/template/all              # deleteAllTemplates
```

**422 Unprocessable Entity is the response you should expect**, and it is the system
protecting you: a template that still has compositions referencing it cannot be deleted
or incompatibly replaced. Migrate or remove the referencing compositions first.

`deleteTemplate` returns **202 Accepted**, not 204 — the removal is asynchronous. Poll
`getTemplatesClassic` to confirm.

`deleteAllTemplates` does exactly what it says on a shared deployment. There is no undo.

## Guardrails for an agent

- Never call anything under `/rest/admin/**` on a production deployment without an
  explicit human instruction naming the template. These operations are physical and
  irreversible.
- Treat 422 as "correct refusal", not as a transient error to retry.
- A template upload changes what every future write on that server is allowed to
  contain. It is a schema migration, not a data write.
