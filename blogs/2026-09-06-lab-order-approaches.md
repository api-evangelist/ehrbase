---
title: "Lab order approaches"
url: "https://discourse.openehr.org/t/lab-order-approaches/17256#post_3"
date: "2026-09-06"
author: "@Seref Seref Arikan"
feed_url: "https://discourse.openehr.org/posts.rss"
---
chunlan.ma: After around 20 years working with this type of clinical system, I haven’t seen many requirements where the application needs to reconstruct the laboratory state from every individual status event Not only that, but the number of versions of a composition has been a constant pain on my backside from an analytics point of view. When you’re feeding an openEHR system from an external system that can raise clinical events (such as pathology), you are at the mercy of that external system: they’ll fire HL7 messages whenever they want to, so they set the granularity and scope of (versioni
