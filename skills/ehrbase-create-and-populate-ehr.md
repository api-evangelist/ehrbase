---
name: ehrbase-create-and-populate-ehr
description: >-
  Create an openEHR Electronic Health Record on an EHRbase server and commit the first
  clinical COMPOSITION into it, using an operational template that is already loaded.
generated: '2026-09-02'
method: generated
source: openapi/_original/ehrbase-api-openapi.json + https://docs.ehrbase.org/docs/EHRbase/openEHR-Introduction/Create-EHR
api: EHRbase openEHR REST API
base_url_note: >-
  EHRbase is self-hosted. Substitute your own deployment's base URL. The project's
  public sandbox is https://sandkiste.ehrbase.org/ehrbase - shared, resettable,
  unauthenticated, and never for real patient data.
operations:
  - createEhr
  - createEhrWithId
  - getEhrBySubject
  - getEhrById
  - getTemplatesClassic
  - getWebTemplate
  - createComposition
  - getComposition
---

# Create an EHR and commit a composition

## Before you start

- Know your base URL. There is no vendor host.
- Know your auth mode. The default is **none**; if the operator enabled it you will need
  HTTP Basic or an OAuth2 bearer JWT (`authentication/ehrbase-authentication.yml`).
- Nothing here is idempotent unless it says so. Read step 5 before you retry anything.

## 1. Decide whether the EHR already exists

An EHR is per-subject. Creating a second one for the same patient is the most common
mistake.

```
GET /rest/openehr/v1/ehr?subject_id={id}&subject_namespace={namespace}
```

`getEhrBySubject` returns **200** with the EHR or **404** if there is none. A 404 here is
the go-ahead, not an error.

## 2. Create the EHR

Two options, and the second is the one an agent should prefer:

- `createEhr` — `POST /rest/openehr/v1/ehr`. The server assigns the `ehr_id`. **Not
  safely retryable**: a retry after a timeout creates a second EHR.
- `createEhrWithId` — `PUT /rest/openehr/v1/ehr/{ehr_id}` with an `ehr_id` you generate.
  **Safely retryable**: `201` on create, `409` if it already exists, `400` if malformed.
  Treat `409` as success.

Optionally supply an `EHR_STATUS` in the body to bind the subject
(`EhrStatus.subject`), and set `is_queryable` / `is_modifiable`.

## 3. Find the template you are writing against

```
GET /rest/openehr/v1/definition/template/adl1.4
```

`getTemplatesClassic` lists the operational templates loaded on this server, with
`template_id`, `concept` and `archetype_id`. If the template you need is absent, load it
first — see the `ehrbase-manage-templates` skill.

Then fetch the **web template**, which is the only practical way to learn the paths:

```
GET /rest/openehr/v1/definition/template/adl1.4/{template_id}/webtemplate
```

`getWebTemplate` returns EHRbase's flattened projection of the template: every leaf node
with its `aqlPath`, input type, cardinality and terminology constraints. There is also
`getTemplateExample` (`.../{template_id}/example`) which returns a filled example
composition — start from that rather than building an RM tree by hand.

## 4. Commit the composition

```
POST /rest/openehr/v1/ehr/{ehr_id}/composition
Content-Type: application/openehr.wt.flat.schema+json
Prefer: return=representation
openEHR-AUDIT_DETAILS: <committer / change type / description>
```

`createComposition` returns **201** with the new `version_uid` in the `ETag` and
`Location` headers.

Use the **flat** media type (`application/openehr.wt.flat.schema+json`) unless you have
a reason not to. It takes a flat `path -> value` map keyed on the paths the web template
gave you, instead of a nested openEHR RM document. `application/json` (canonical RM) and
`application/xml` are also accepted.

Always send `openEHR-AUDIT_DETAILS`. It is persisted with the CONTRIBUTION and is the
provenance record for who committed what and why.

`Prefer: return=representation` echoes the stored composition back;
`Prefer: return=minimal` returns headers only.

## 5. Retry rules

`createComposition` has **no idempotency key**. If the request times out ambiguously, do
not resend it. Instead:

```
GET /rest/openehr/v1/ehr/{ehr_id}/contribution/{contribution_uid}
```

or list the EHR's contributions and check whether your write landed. Only then retry.

`createEhrWithId` is the exception — it is safe to resend, because a duplicate returns
`409` rather than creating anything.

## 6. Read it back

```
GET /rest/openehr/v1/ehr/{ehr_id}/composition/{versioned_object_uid}
```

`getComposition` accepts a `version_at_time` query parameter for a point-in-time read.

## Failure modes

| Status | Meaning | What to do |
|---|---|---|
| 400 | Composition failed template or terminology validation | Re-check against the web template; coded values must be inside the constrained ValueSet |
| 401 / 403 | Auth mode is on and you are unauthenticated or under-privileged | See `authentication/ehrbase-authentication.yml` |
| 404 | EHR or template does not exist | Resolve the EHR first (step 1) |
| 409 | EHR already exists | Success for `createEhrWithId` — use the existing one |

There is no error body schema. Branch on the status code alone.
