---
layout: post
title: "PCA and clustering find patterns, but do not prove prognosis"
date: 2026-08-11 10:00:00-0400
description: PCA summarizes variation and clustering assigns groups; neither method knows whether those groups matter for patient outcomes.
tags: PCA clustering dimensionality-reduction unsupervised-learning survival-analysis
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 4
related_posts: false
thumbnail: assets/img/blog/cancer-signature-series/pca-discovery.png
---

> **Beyond the Tumor Cell, Part 4 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

A PCA plot with two colored groups can feel like a result by itself. If the points separate cleanly, it is natural to think that we have discovered two biological types of cancer. But the plot has answered a narrower question:

> Can these samples be arranged along major directions of expression variation, and does a chosen grouping align with those directions?

It has not yet answered whether the groups are reproducible, clinically relevant, causally meaningful, or useful for prediction.

## Start with the expression matrix

Imagine a matrix with patients as rows and genes as columns. Each patient is a point in a space with one dimension per gene. With 87 genes, that space has 87 dimensions and cannot be viewed directly.

Principal-component analysis rotates the coordinate system to find directions that capture as much variation as possible. The first principal component, PC1, captures the largest linear source of variation; PC2 captures the largest remaining direction orthogonal to PC1.

PCA is **unsupervised** because it does not use progression time or event status. It sees expression variation, not clinical outcome.

That is both its strength and its limitation. PCA can reveal dominant structure without being told what to find. But the dominant structure could reflect biology, platform effects, tissue composition, sample quality, or some combination of them.

## Clustering answers another question

K-means clustering assigns samples to a chosen number of groups. With $k=2$, it searches for two centers and assigns each sample to the nearer center. It minimizes within-cluster squared distance:

$$
\sum_{k=1}^{K}\sum_{i \in C_k}
\lVert x_i - \mu_k \rVert^2.
$$

The algorithm will return two groups whenever we request two groups. That does not prove that nature contains exactly two tumor states. Setting $k=3$ will produce three groups. Choosing $k$ is a modeling decision that should be justified by stability, interpretability, and the scientific goal.

In our exploratory workflow, $k=2$ represented high- and low-axis states, while $k=3$ was examined as a sensitivity analysis. The orientation was determined from aggregate axis expression so that the higher-expression cluster was labeled high axis.

## What the exploratory plot showed

In the selected discovery arm, PC1 explained 31.7% of the evidence-gene variance and PC2 explained 5.6%. The two $k=2$ clusters were largely separated along PC1.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/pca-discovery.png" class="img-fluid rounded z-depth-1" zoomable=true alt="PCA of evidence genes with high-axis and low-axis clusters" %}

<div class="caption">
  Evidence-gene PCA in the exploratory discovery arm. Separation along PC1 shows a strong expression pattern, but PCA itself does not use or test progression outcomes.
</div>

This is evidence that the selected genes capture coordinated variation. It does not tell us why that variation exists. Later single-cell and comparator analyses suggested that much of PC1 reflected stromal and fibroblast-rich tissue composition.

That is a good example of why a principal component should not be named too quickly. “PC1” is mathematically safe. “Neural progression axis” is a biological claim that requires evidence beyond the plot.

## Heatmaps, PCA, and UMAP are different views

These methods are often grouped together because they produce visual summaries, but they preserve different information.

- **Heatmap:** shows gene-by-sample expression patterns. It does not establish independent clusters or prognosis.
- **PCA:** summarizes major linear variation. It does not establish causality or clinical relevance.
- **K-means:** assigns samples to a requested number of groups. It does not prove that the chosen $k$ is biologically true.
- **UMAP/t-SNE:** displays local neighborhoods in two dimensions. It does not guarantee reliable global distances or patient-level significance.

For single-cell data, UMAP or t-SNE can help locate cell populations. For the bulk analysis in this paper, PCA summarized sample-level expression structure. These uses should not be blended: a point on the bulk PCA is a tumor sample, while a point on a single-cell UMAP is usually a cell.

## “Unsupervised” does not protect the whole workflow from bias

PCA and K-means did not use outcome data internally. However, the historical workflow evaluated many six-versus-six dataset partitions and selected a partition partly according to survival behavior. The final displayed discovery and assessment arms therefore came from an outcome-informed search.

This is a subtle but important point. An unsupervised algorithm can sit inside a supervised **selection process**. If we try many unsupervised clusterings and keep the one with the strongest survival difference, the reported survival value is optimistic.

For that reason, the paper retained these plots as exploratory provenance rather than calling the second arm independent validation.

## Why the frozen continuous score became primary

The more defensible primary analysis did not use PCA coordinates or a selected $k$-means partition. It used a frozen score defined before the final survival analysis:

1. Standardize each of the 87 genes within each cohort.
2. Give every gene the same positive direction.
3. Average the standardized values.
4. Fit a continuous-score Cox model separately in each cohort.
5. Pool cohort estimates with random-effects meta-analysis.

The score is easier to transport because a new patient receives a numeric value without rerunning clustering on a combined dataset. It also avoids throwing away information by converting every patient into only “high” or “low.”

Most importantly, the primary evidence came from 11 cohort-specific estimates, leave-one-cohort-out checks, endpoint-family sensitivity analyses, and locked TCGA transport. That evidence is not equivalent to a single attractive PCA plot.

## A map, a grouping, and a test

It helps to separate the workflow into three stages:

**Map:** PCA, heatmaps, and UMAP show structure.

**Grouping or scoring:** K-means assigns categories; a frozen formula assigns a continuous score.

**Clinical test:** Kaplan-Meier, log-rank, and Cox analyses ask whether the assigned state is associated with time-to-event outcomes.

Confusion appears when evidence from one stage is used to claim success at another. Clear separation at the map stage is not a survival result. A small survival value in the same data used to choose a map is not external validation.

## What we can and cannot say

**Supported:** The evidence genes contain a strong coordinated expression pattern, and the frozen continuous score is associated with progression-related outcomes across cohorts.

**Not supported:** PCA does not prove that there are exactly two natural cancer types, and the outcome-selected historical partition is not prospectively locked validation.

Unsupervised methods are powerful discovery tools. Their output becomes scientific evidence only when the downstream claim is tested with the correct design.

**Previous:** [Why 433,804 cells are not 433,804 patients.]({{ '/blog/2026/cells-are-not-patients/' | relative_url }})<br>
**Next:** [Kaplan-Meier and Cox models: why do we need both?]({{ '/blog/2026/kaplan-meier-and-cox/' | relative_url }})
