---
name: ehrbase-update-composition-safely
description: >-
  Update or delete an existing openEHR COMPOSITION on EHRbase without losing a concurrent
  writer's change - the If-Match / version_uid optimistic-concurrency flow, 412 recovery,
  and what "delete" actually does.
generated: '2026-09-02'
method: generated
source: >-
  openapi/_original/ehrbase-api-openapi.json + conventions/ehrbase-conventions.yml +
  https://specifications.openehr.org/releases/ITS-REST/latest/
api: EHRbase openEHR REST API
operations:
  - getComposition
  - updateComposition
  - deleteComposition
  - retrieveVersionedCompositionByVersionedObjectUid
  - retrieveVersionedCompositionRevisionHistoryByEhr
  - retrieveVersionOfCompositionByVersionUid
  - retrieveVersionOfCompositionByTime
  - updateEhrStatus
  - updateDirectory
  - deleteDirectory
---

# Update a composition without clobbering someone else

## The mental model

openEHR never mutates. An update **appends a new version** to a version tree, and every
prior version stays permanently addressable. Two identifiers matter:

- `versioned_object_uid` — the stable identity of the clinical object.
- `version_uid` — one specific version, shaped
  `{versioned_object_uid}::{creating_system_id}::{version_tree_id}`.

You address the object by the former and prove you are up to date with the latter.

## 1. Read the current head

```
GET /rest/openehr/v1/ehr/{ehr_id}/composition/{versioned_object_uid}
```

`getComposition` returns the composition; the current `version_uid` comes back in the
`ETag` header.

## 2. Update with a precondition

```
PUT /rest/openehr/v1/ehr/{ehr_id}/composition/{versioned_object_uid}
If-Match: {version_uid you just read}
Content-Type: application/openehr.wt.flat.schema+json
Prefer: return=representation
openEHR-AUDIT_DETAILS: <committer / change type / description>
```

`updateComposition` returns **200** with the new `version_uid`.

**`If-Match` is not optional in practice.** It is the only thing standing between two
concurrent writers and a silent lost update.

## 3. Recover from 412

A **412 Precondition Failed** means someone committed a version between your read and
your write. The recovery is always the same three steps, and never a blind retry:

1. `GET .../versioned_composition/{versioned_object_uid}/revision_history` to see what
   landed (`retrieveVersionedCompositionRevisionHistoryByEhr`).
2. Read the new head, or a specific version with
   `GET .../versioned_composition/{versioned_object_uid}/version/{version_uid}`.
3. Re-apply your change on top of it and PUT again with the **new** `version_uid` in
   `If-Match`.

The same 412 flow governs `updateEhrStatus`, `updateDirectory` and `deleteDirectory`.

## 4. Understand what delete does

```
DELETE /rest/openehr/v1/ehr/{ehr_id}/composition/{preceding_version_uid}
```

`deleteComposition` returns **204** and performs a **logical** delete: a DELETED version
is appended to the tree. The prior content is still readable through
`retrieveVersionOfCompositionByVersionUid`. This is reversible in the sense that nothing
is destroyed — but EHRbase publishes no reversal window and no "undelete" operation, so
restoring means re-committing the content yourself.

The **physical** delete lives on the Admin API
(`DELETE /rest/admin/ehr/{ehr_id}/composition/{composition_id}`), is admin-role only,
and is **not reversible**. Do not reach for it to fix a mistake.

## 5. Time travel

Every versioned surface supports a point-in-time read:

- `retrieveVersionOfCompositionByTime` — `.../version?version_at_time=<ISO8601>`
- `retrieveVersionOfEhrStatusByTime`
- `getFolderInDirectoryVersionAtTime`

Use these to answer "what did the record say on date X" rather than reconstructing it.

## What this API does NOT give you

- No `Idempotency-Key`. Concurrency safety comes from `If-Match`, full stop.
- No error body. 412 arrives as a bare status code.
- No bulk update. One composition per request.
- No transaction spanning multiple compositions on the open-source distribution — group
  them into a CONTRIBUTION (`POST /rest/openehr/v1/ehr/{ehr_id}/contribution`) if they
  must commit together.
