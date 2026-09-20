---
title: "Why is the DV_TEXT <value> element missing when mapping terminologies with ORDINAL?"
url: "https://discourse.openehr.org/t/why-is-the-dv-text-value-element-missing-when-mapping-terminologies-with-ordinal/11468#post_7"
date: "2026-09-19"
author: "@ian.mcnicoll Ian McNicoll"
feed_url: "https://discourse.openehr.org/posts.rss"
---
I think we agree on the issue and that the XSD should not demand a populated DV_CODED_TEXT.value The Ocean .opt generator gets around this by creating (a space) which complies with the in DV_TEXT. The PDF specs say Which is IMO correct and BMM C_ORDINAL: name: C_ORDINAL documentation: 'Constrainer class for Ordinal data.' ancestors: - C_DOMAIN_TYPE properties: list: !P_BMM_CONTAINER_PROPERTY name: list documentation: 'Value set of allowed Ordinals in the constraint.' type_def: container_type: List type: ORDINAL cardinality: lower: 0 upper_unbounded: true ORDINAL: name: ORDINAL documentation: '
