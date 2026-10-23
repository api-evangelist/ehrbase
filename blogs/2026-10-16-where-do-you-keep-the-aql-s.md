---
title: "Where do you keep the AQL's ?"
url: "https://discourse.openehr.org/t/where-do-you-keep-the-aqls/5052?page=2#post_26"
date: "2026-10-16"
author: "@pablo Pablo Pazos"
feed_url: "https://discourse.openehr.org/posts.rss"
---
Hi all, there has been some progress on our side too that touch this topic. Now we fully version all queries and store the previous versions for reference (we could rollback too). We also added a “break detection”: when a new version of an OPT is uploaded, it detects all loaded queries that might be affected by the changes in the OPT and shows a report mentioning the type of change and how it affects each existing query.
