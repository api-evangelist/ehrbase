---
title: "openEHR archetypes as SQL Tables"
url: "https://discourse.openehr.org/t/openehr-archetypes-as-sql-tables/11412?page=4#post_63"
date: "2026-09-11"
author: "@borut.jures Borut Jures"
feed_url: "https://discourse.openehr.org/posts.rss"
---
I made several attempts at using RM as close as possible to the native capabilities of the database(s). I settled on a database that perfectly supports implementing RM classes as native types. Simply put, LOCATABLE classes have their own tables, and other classes are composite types.
