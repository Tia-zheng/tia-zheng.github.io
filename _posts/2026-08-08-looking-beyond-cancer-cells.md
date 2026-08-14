---
layout: post
title: "Why looking beyond cancer cells helps us understand cancer"
date: 2026-08-08 10:00:00-0400
description: A tumor is a tissue ecosystem, so a prognostic expression signal may come from fibroblasts, matrix, vessels, and immune cells as well as malignant cells.
tags: cancer-biology tumor-microenvironment transcriptomics colorectal-cancer
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 1
related_posts: false
featured: true
thumbnail: assets/img/blog/cancer-signature-series/geo-heatmap.png
---

> **Beyond the Tumor Cell, Part 1 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

When people hear _cancer biology_, they often picture a malignant cell accumulating mutations and dividing out of control. That picture is important, but incomplete. A tumor is also a tissue. It contains fibroblasts, immune cells, blood vessels, extracellular matrix, signaling molecules, and normal cells pushed into abnormal states. Cancer cells live inside that environment, change it, and depend on it.

This leads to the question that started our project:

> Why would we study stromal remodeling, neural-guidance signaling, or tissue communication when the disease is cancer?

The short answer is that these programs are not outside cancer biology. They are outside a **cancer-cell-only** view of cancer biology.

## A bulk tumor sample is a mixture

Bulk gene-expression data measure RNA from a piece of tissue. The result is usually one expression vector per tumor sample, but that vector combines RNA from many cell populations. A useful mental model is

$$
\text{bulk expression}
= \sum_c
(\text{fraction of cell type }c)
\times
(\text{expression within cell type }c).
$$

If a collagen gene is high in a bulk tumor, at least two explanations are possible. The tumor may contain more collagen-producing fibroblasts, or the fibroblasts already present may be expressing more collagen. Both can happen at once. Bulk data alone cannot fully separate them.

That matters because the non-malignant compartments are active participants in disease. Cancer-associated fibroblasts can reorganize extracellular matrix. Endothelial cells form and alter blood vessels. Myeloid and lymphoid cells can support or restrain antitumor immunity. Matrix density and tissue architecture can affect invasion, drug penetration, and cell migration. A signature produced by these compartments may therefore associate with progression even if the genes are not primarily expressed by malignant epithelial cells.

## Why begin with a cross-system gene axis?

Our starting gene set brought together stromal-remodeling, neural-guidance, and related signaling genes. It was assembled from bladder-cancer and broader cancer literature before the colorectal cancer outcome analysis. That history is important: the list was not a colorectal cancer signature selected to maximize survival separation.

Applying it to colorectal cancer was therefore a **cross-cancer hypothesis test**. We were asking whether a program motivated elsewhere would appear reproducibly in colorectal tumors and whether it would track progression-related outcomes.

There was a biological reason to try. Developmental and tissue-remodeling programs are frequently reused in disease. Molecules first studied in axon guidance may also influence cell migration, vascular patterning, or stromal signaling. EMT-related regulators may connect epithelial plasticity to matrix-rich tumor states. The name of the field in which a gene was discovered does not restrict the gene to that field.

But this reasoning creates a risk. If we start with an interesting label such as _neural-stromal axis_, we may be tempted to interpret any association as evidence for the entire proposed mechanism. The analysis therefore had to ask not only whether the score worked, but **what part of it was doing the work**.

## What the colorectal cancer data showed

The frozen 87-gene score was evaluated across 11 GEO cohorts containing 1,783 tumors and 441 progression-related events. A one-standard-deviation increase in the score was associated with shorter progression-related survival, with a pooled hazard ratio of 1.33. A locked application to 376 TCGA COAD/READ tumors gave a hazard ratio of 1.25 for progression-free interval.

Those results established a reproducible association. They did not yet identify its cellular source.

The next analyses changed the interpretation:

- The stromal-remodeling module reproduced most of the survival association.
- Removing stromal genes weakened the full-score association much more than removing neural-guidance genes.
- The score was strongly correlated with CMS4-like, cancer-associated fibroblast, stromal, and purity measures.
- A related 84-gene annotation score localized most strongly to fibroblast and other stromal compartments in single-cell data.
- After removing measured CMS4, CAF, stromal, and purity components using a cross-fitted model, no clear residual survival association remained.

The result was not “we found a neural mechanism.” It was more specific and better supported: the score captured a **fibroblast-rich, CMS4-like tumor state** associated with progression-related outcomes.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/cell-type-effects.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Immune and stromal score differences between low-axis and high-axis colorectal tumors" %}

<div class="caption">
  Bulk microenvironment comparisons showed higher CAF, endothelial, macrophage, and stromal-related scores in high-axis tumors. These comparisons help interpret the mixture; they do not identify a causal cell type by themselves.
</div>

## A more precise conclusion is not a weaker conclusion

It can feel disappointing when an exciting multicomponent hypothesis becomes “mostly stromal.” Scientifically, the opposite is true. The revised conclusion tells us what the data actually support.

The score still captures meaningful tumor biology. Stromal abundance and stromal-cell state are not technical nuisances. They are features of the tumor microenvironment. What changed is the level of novelty we can claim. The data support a reproducible microenvironmental state, but they do not show that the score is independent of established stromal biology or that it represents a tumor-cell-intrinsic neural program.

This distinction gives us three separate questions for the rest of the series:

1. **What biological program does the score represent?**
2. **What is the correct independent unit in each dataset?**
3. **Does the score add information beyond what we already know?**

The answers require different methods. No single heatmap, cluster plot, or survival curve can answer all three.

## What we can and cannot say

**Supported:** Tumors are mixtures of malignant and non-malignant compartments. The frozen score is associated with progression-related outcomes and is strongly stromal-dominant.

**Not supported:** The study does not demonstrate a new neural mechanism, prove causality, or establish a clinically deployable biomarker.

The larger lesson is simple: looking beyond malignant cells is not leaving cancer biology. It is treating cancer as the tissue-level disease that it is.

## Further reading

- Isella et al., [_Stromal contribution to the colorectal cancer transcriptome_](https://doi.org/10.1038/ng.3224).
- Guinney et al., [_The consensus molecular subtypes of colorectal cancer_](https://doi.org/10.1038/nm.3967).
- Sahai et al., [_A framework for advancing our understanding of cancer-associated fibroblasts_](https://doi.org/10.1038/s41568-019-0238-1).

**Next:** [Are neural, stromal, and EMT programs really separate?]({{ '/blog/2026/neural-stromal-and-emt-labels/' | relative_url }})
