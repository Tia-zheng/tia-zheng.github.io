---
layout: page
title: Beyond the Tumor Cell
description: A seven-part guide to what a cancer transcriptomic signature can, and cannot, tell us.
permalink: /blog/cancer-signature-series/
nav: false
---

## What does a cancer signature really mean?

A tumor-expression signature can predict an outcome without being unique to cancer cells. It can separate patients without explaining a mechanism. It can remain biologically useful even when it adds little information beyond an established tumor subtype. Those distinctions are easy to lose when a paper moves quickly from genes, to clusters, to survival curves, to single-cell maps.

This seven-part series follows one colorectal cancer project from its original neural-stromal remodeling hypothesis to a more precise conclusion: the score marks a fibroblast-rich, CMS4-like tumor state associated with progression-related outcomes. Each post starts with a question that arose during the analysis and ends by separating what the data support from what they do not.

{% include figure.liquid loading="eager" path="assets/img/blog/cancer-signature-series/geo-heatmap.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Heatmap of selected neural-stromal genes across colorectal cancer samples" %}

<div class="caption">
  A multigene expression pattern can be visually coherent before we know which cells generate it, whether it predicts outcome, or whether it adds information beyond known biology. The series follows those questions in order.
</div>

{% assign signature_posts = site.posts | where: "series", "beyond-the-tumor-cell" | sort: "series_order" %}

## Reading order

<ol>
{% for post in signature_posts %}
  <li>
    <strong><a href="{{ post.url | relative_url }}">{{ post.title }}</a></strong><br>
    {{ post.description }}
  </li>
{% endfor %}
</ol>

## The recurring rule

Every post uses the same four questions:

1. **What is the scientific question?**
2. **What does the method actually measure?**
3. **What did this study find?**
4. **What would be an overclaim?**

The goal is not to make every result sound novel. The goal is to make every conclusion match the evidence.
