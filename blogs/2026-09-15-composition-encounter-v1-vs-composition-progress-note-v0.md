---
title: "COMPOSITION.encounter.v1 vs COMPOSITION.progress_note.v0 for INPATIENT Daily Clinical Notes During an EPISODE"
url: "https://discourse.openehr.org/t/composition-encounter-v1-vs-composition-progress-note-v0-for-inpatient-daily-clinical-notes-during-an-episode/17300#post_2"
date: "2026-09-15"
author: "@ian.mcnicoll Ian McNicoll"
feed_url: "https://discourse.openehr.org/posts.rss"
---
Your approach seems pretty solid to me and I think Composition.progress_note is a better fit for ongoing documentation during an admission for all clinical progress notes not just nurses. Encounter is not wrong but feels a bit closer to a typical outpatient clinic/ GP consultation or Nurse visit. Having said that, I suspect these boundaries are a little blurred in some care-settings e.g a dialysis or chemotherapy visit and I probably would not over-rely on the Composition archetype to imply strong semantics.
