---
title: "Representing a business identifier for a procedure (EHDS header.identifier) in openEHR-EHR-ACTION.procedure"
url: "https://discourse.openehr.org/t/representing-a-business-identifier-for-a-procedure-ehds-header-identifier-in-openehr-ehr-action-procedure/17318#post_4"
date: "2026-09-23"
author: "@jkrafft Joshua Krafft"
feed_url: "https://discourse.openehr.org/posts.rss"
---
Thanks Ian, that helps. One nuance I’d like to check: I’d describe the EHDS/FHIR business identifier less as not allocated internally and more as allocated by the source system, but stable and meaningful once the data leaves that system. FHIR defines Procedure.identifier as “ Business identifiers assigned to this procedure by the performer or other systems which remain constant as the resource is updated and is propagated from server to server ”, and EHDS says “ unique within its system ”.
