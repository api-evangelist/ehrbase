---
title: "AD shows wrong value set in Swedish"
url: "https://discourse.openehr.org/t/ad-shows-wrong-value-set-in-swedish/17323#post_4"
date: "2026-09-24"
author: "@emmanuel.eschmann Emmanuel Eschmann"
feed_url: "https://discourse.openehr.org/posts.rss"
---
I have no experience with the Forms tab in the Archetype Designer, but this problem may well have the same root cause as the behaviour we observed in the opt and Web templates generated from multilingual templates: If we used an archetype multiple times within a template, the DE and FR translations in the opt and Web templates (when EN was selected as the primary language) would only reflect the node name, description and comment of an archetype’s first occurrence in the template for each other instance of the same archetype. The specific node names, descriptions and comments for subsequent oc
