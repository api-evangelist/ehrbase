---
title: "Performance of an openEHR CDR based on a graph database"
url: "https://discourse.openehr.org/t/performance-of-an-openehr-cdr-based-on-a-graph-database/17224?page=2#post_37"
date: "2026-09-06"
author: "@borut.jures Borut Jures"
feed_url: "https://discourse.openehr.org/posts.rss"
---
We have tools that eliminate the need for assumptions. Engineers shouldn’t assume. I used the profiler to measure the actual time spent: 0,72% - parse AQL 0,37% - convert AQL to SQL 76,72% - execute SQL by the database
