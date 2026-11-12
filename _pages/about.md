---
layout: about
title: About
permalink: /
subtitle: ""

profile:
  align: right
  image: tia-zheng-lab.jpg
  image_circular: false
  more_info: ""

selected_papers: false # includes a list of papers marked as "selected={true}"
social: false # includes social icons at the bottom of the page

announcements:
  enabled: false # includes a list of news items
  scrollable: true # adds a vertical scroll bar if there are more than 3 news items
  limit: 5 # leave blank to include all the news in the `_news` folder

latest_posts:
  enabled: true
  scrollable: true # adds a vertical scroll bar if there are more than 3 new posts items
  limit: 3 # leave blank to include all the blog posts
---

<style>
  .post .profile {
    width: 220px;
    max-width: 40%;
    margin: 0 0 1rem 1.5rem;
  }

  @media (max-width: 575.98px) {
    .post .profile {
      float: none !important;
      width: 180px;
      max-width: 100%;
      margin: 0 auto 1.5rem;
    }
  }
</style>

I am a student researcher at Western Reserve Academy in Hudson, Ohio. This is my personal website and blog on bioinformatics, cancer biology, and data science. I’m interested in how mathematical modeling and machine learning can help us understand why the same type of cancer can progress differently from one patient to the next.

Most of this site grew out of my colorectal cancer research in the summer of 2026. I studied a pattern of activity across 87 genes and wanted to know how it related to patient outcomes. I also wanted to know which cells in a tumor contributed to that pattern. Looking at individual cells helped me trace much of it to fibroblasts, which build and maintain connective tissue.

You can read the [project summary and preprint]({{ '/projects/' | relative_url }}). On [the blog]({{ '/blog/' | relative_url }}), I write about questions I had along the way, including how to compare patients when one tumor sample contributes far more cells than another.
