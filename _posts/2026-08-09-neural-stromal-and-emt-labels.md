---
layout: post
title: "Are neural, stromal, and EMT programs really separate?"
date: 2026-08-09 10:00:00-0400
description: Biological labels are useful maps, but shared genes and reused developmental programs do not automatically prove a unified mechanism.
tags: cancer-biology EMT neural-guidance stromal-biology biomarkers
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 2
related_posts: false
thumbnail: assets/img/blog/cancer-signature-series/gene-expression-atlas.png
---

> **Beyond the Tumor Cell, Part 2 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

Scientists divide biology into named systems: neural guidance, stromal remodeling, epithelial-mesenchymal transition, angiogenesis, immune signaling. These labels help us organize a huge literature. The danger is forgetting that cells do not read our category names.

A receptor can guide an axon during development and influence tumor-cell migration in an adult tissue. A transcription factor can participate in embryonic patterning and later regulate epithelial plasticity. A matrix gene can reflect both the number of fibroblasts and the state of those fibroblasts. Shared genes across named pathways may reveal genuine biological reuse, but they can also make our categories look more independent than they are.

This is why a pathway label should be treated as a **hypothesis about function**, not as proof of a mechanism.

## First, a necessary correction about VIM and ZEB

It would be inaccurate to describe _VIM_, _ZEB1_, or _ZEB2_ as neuronal genes in this study.

_VIM_ encodes vimentin, an intermediate-filament protein widely used as a mesenchymal marker. _ZEB1_ and _ZEB2_ encode transcription factors that can regulate epithelial-mesenchymal transition and epithelial plasticity. These genes also have roles in development, and developmental processes can include neural-crest biology, but that does not make their expression evidence of neurons or neuronal differentiation.

In our paper, _VIM_, _ZEB1_, and _ZEB2_ belong to an EMT/stromal-remodeling interpretation. The “neural” label comes mainly from a different subset of genes associated with axon guidance or neural-related signaling.

This distinction matters because a gene's historical association is not the same as its function in a particular tumor. We need expression localization, perturbation experiments, and direct mechanistic evidence before moving from “this gene has a neural-related annotation” to “this tumor uses a neural mechanism.”

## Why neural-guidance genes can appear outside neurons

During development, cells must move, recognize boundaries, form connections, and build organized tissues. Axon-guidance systems are one solution to those problems. Tumors face related physical tasks: cells migrate through matrix, vessels reorganize, and different compartments exchange signals.

It is therefore plausible that guidance molecules participate in cancer without producing a neural phenotype. For example, a guidance receptor may affect endothelial patterning or cell motility. That is biological reuse across tissues.

But plausibility is not evidence that all neural-guidance genes form one active pathway in a tumor. To support that stronger statement, we would want several kinds of evidence to agree:

1. The relevant ligands and receptors are expressed in appropriate cell types.
2. The cells are spatially close enough to communicate.
3. Downstream targets show pathway activity.
4. Perturbing the pathway changes a cancer phenotype.
5. The result replicates in independent patients or models.

Our focused ligand-receptor screen did not meet that standard. Twelve stromal-to-epithelial pairs had minimal expression support in two single-cell datasets, but none satisfied the combined expression, spatial, outcome, and prior-evidence criteria. Co-expression alone was not treated as communication.

## Use ablation to ask which label carries the signal

One way to study an overlapping signature is **module ablation**. Instead of asking whether the full score is associated with outcome, we remove or isolate parts of it and repeat the analysis.

The complete frozen score had a pooled hazard ratio of 1.33 per standard-deviation increase. The stromal-only module had a very similar hazard ratio of 1.34. In contrast, the full score after removal of stromal genes had a hazard ratio of 1.10. Removing neural-guidance genes had much less effect on the overall result.

This does not prove that every stromal gene is causal. It tells us that the reproducible survival association is carried mainly by the stromal-remodeling component. The “neural-stromal” name describes the history of the gene list better than the dominant result.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/module-forest.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Forest plot of fine gene modules and progression-related survival" %}

<div class="caption">
  Module-level estimates are heterogeneous. CAF activation and matrix-remodeling modules show stronger adverse associations than several neural-related modules. These exploratory estimates help locate the signal; they do not establish separate causal pathways.
</div>

## A single famous marker is not the whole program

_VIM_ provided a useful negative control. It clearly separated tumors into VIM-high and VIM-low groups, so the grouping procedure worked. Yet the groups did not differ significantly in progression-related survival: the log-rank value was 0.313, and continuous Cox analysis was also nonsignificant.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/vim-qc.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Quality control showing separation of VIM-high and VIM-low colorectal tumors" %}

{% include figure.liquid path="assets/img/blog/cancer-signature-series/vim-survival.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Kaplan-Meier and Cox results for the VIM single-gene control" %}

<div class="caption">
  VIM expression produced clean high and low groups, but those groups did not reproduce the multigene survival association. A visually successful classification is not automatically a prognostic result.
</div>

The lesson is not that _VIM_ is biologically unimportant. It is that one canonical marker did not explain the multigene pattern. Removing _ZEB1_ and _ZEB2_ also left an exploratory five-gene EMT/stromal set with survival separation, but that set was defined after reviewing the data and therefore cannot be presented as independently validated.

## Expression maps show reuse, not identity

Single-cell maps also show why one label is rarely enough. _LUM_ and _COL1A1_ are concentrated in particular stromal regions, _VIM_ is much broader, _ZEB2_ appears across several compartments, and _NRP2_ has another distribution. Their co-membership in a broad conceptual axis does not mean they are produced by the same cells or perform the same task.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/gene-expression-atlas.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Single-cell atlas maps for LUM, SNAI2, ZEB2, VIM, COL1A1, and NRP2" %}

<div class="caption">
  Different members of the proposed axis occupy different parts of the single-cell atlas. Shared membership in a gene list should not be confused with a single cellular source or mechanism.
</div>

## What Our Conclusions Support

**Supported:** Developmental, EMT, vascular, and stromal programs reuse molecular tools. In this dataset, the multigene survival association is stromal-dominant and is not reducible to _VIM_ alone.

**Not supported:** The data do not show that _VIM_ or _ZEB1/2_ are neuronal markers, that the tumor has undergone neuronal differentiation, or that the neural-related genes constitute one causal pathway.

The most useful role of labels is to generate tests. Once the tests are complete, the labels should bend to the evidence, not the other way around.

**Previous:** [Why looking beyond cancer cells helps us understand cancer.]({{ '/blog/2026/looking-beyond-cancer-cells/' | relative_url }})<br>
**Next:** [Why 433,804 cells are not 433,804 patients.]({{ '/blog/2026/cells-are-not-patients/' | relative_url }})
