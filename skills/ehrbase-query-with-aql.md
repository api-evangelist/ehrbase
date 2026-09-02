---
name: ehrbase-query-with-aql
description: >-
  Query clinical data out of an EHRbase server with the Archetype Query Language - ad-hoc
  execution, and registering and running named stored queries.
generated: '2026-09-02'
method: generated
source: openapi/_original/ehrbase-api-openapi.json + https://docs.ehrbase.org/docs/category/archetype-query-language
api: EHRbase openEHR REST API
operations:
  - executeAdHocQuery
  - executeAdHocQuery_1
  - putStoredQuery
  - putStoredQuery_1
  - getStoredQueryList
  - getStoredQueryList_1
  - getStoredQueryVersion
  - executeStoredQuery
  - executeStoredQuery_1
  - executeStoredQuery_2
  - executeStoredQuery_3
  - getWebTemplate
---

# Query an EHRbase server with AQL

AQL is openEHR's query language. It is **not** SQL and it is not a REST filter grammar —
you do not paginate an EHRbase collection, you write a query that returns what you want.

## 1. Learn the paths before you write the query

AQL selects on archetype paths, so you need them first:

```
GET /rest/openehr/v1/definition/template/adl1.4
GET /rest/openehr/v1/definition/template/adl1.4/{template_id}/webtemplate
```

`getWebTemplate` gives every leaf node with its `aqlPath`. Guessing paths is the main
reason AQL queries come back empty rather than erroring.

## 2. Run an ad-hoc query

```
POST /rest/openehr/v1/query/aql
Content-Type: application/json

{ "q": "SELECT c/uid/value FROM EHR e CONTAINS COMPOSITION c LIMIT 100",
  "query_parameters": { } }
```

`executeAdHocQuery_1` (POST) is the one to use. `executeAdHocQuery` (GET, with the query
in the `q` parameter) exists but will collide with URL length limits on any real query.

The response is a `QueryResponseData`: a `columns[]` describing each projection and a
`rows[]` of values, plus `meta` and the echoed `q`.

## 3. Bound the result set inside the query

There are **no** `limit`, `offset`, `page` or `cursor` HTTP parameters on this API. Use
AQL's own `LIMIT` and `OFFSET`. The operator may also have configured server-side default
and maximum result limits
(https://docs.ehrbase.org/docs/EHRbase/Explore/AQL/Configuration) — a truncated result is
possible, so always order deterministically with `ORDER BY` before paging with `OFFSET`.

## 4. Promote a query you will run repeatedly

Register it once:

```
PUT /rest/openehr/v1/definition/query/{qualified_query_name}/{version}
Content-Type: text/plain
```

`putStoredQuery` takes a `qualified_query_name` of the form `{namespace}::{name}` and a
semantic `{version}`. `putStoredQuery_1` is the unversioned variant. Registration is a
PUT and is therefore idempotent.

List and inspect:

```
GET /rest/openehr/v1/definition/query
GET /rest/openehr/v1/definition/query/{qualified_query_name}
GET /rest/openehr/v1/definition/query/{qualified_query_name}/{version}
```

Execute:

```
GET  /rest/openehr/v1/query/{qualified_query_name}/{version}?ehr_id=...
POST /rest/openehr/v1/query/{qualified_query_name}/{version}
```

Use POST (`executeStoredQuery_1`) when you have query parameters to bind; the body takes
`query_parameters`.

## 5. Parameterise, never interpolate

Bind values through `query_parameters` rather than string-building the AQL. Stored
queries plus bound parameters are the pattern to reach for when an agent is generating
queries on behalf of a user.

## Conformance note

EHRbase publishes an AQL conformance test-suite documentation repository covering
SELECT, WHERE, ORDER BY, LIMIT, FROM, PARAMETER and aggregate function behaviour:
https://github.com/ehrbase/conformance-testing-documentation — worth reading before
assuming a construct is supported.

## Failure modes

The query endpoints declare **only 200** in the published contract — there is no
documented error shape for a malformed query. Treat any non-200 as a client error, read
the raw body, and do not retry a query that failed deterministically.
