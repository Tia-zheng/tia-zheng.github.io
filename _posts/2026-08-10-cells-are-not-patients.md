---
layout: post
title: "Why 433,804 cells are not 433,804 patients"
date: 2026-08-10 10:00:00-0400
description: Single cells are nested within people, so patient-level questions require patient-level replication rather than treating every cell as independent.
tags: single-cell pseudobulk statistics study-design colorectal-cancer
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 3
related_posts: false
thumbnail: assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png
---

> **Beyond the Tumor Cell, Part 3 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

Single-cell RNA sequencing produces tables with an astonishing number of columns or rows, depending on how the matrix is stored. A study may contain hundreds of thousands of cells. That scale creates a tempting but incorrect thought:

> If I measured 20,000 cells, do I have 20,000 independent biological samples?

Usually, no. If those cells came from 10 patients, then the study still has only 10 independently sampled patients for a patient-level question.

This is the difference between the **unit of observation** and the **unit of inference**.

## Observation is not the same as replication

A single cell is an observation. A gene count in that cell is real data. But cells from the same person share genetics, treatment history, tissue environment, collection procedure, and many technical effects. Two cells from one tumor are generally more related than two cells sampled from different patients.

Treating all cells as independent would be similar to measuring one student's blood pressure 1,000 times and claiming a study of 1,000 students. More measurements can improve our estimate for that student, but they do not create new people.

The hierarchy in this project looked like this:

- **Bulk transcriptomics:** the repeated observations were tumor-expression vectors. The independent biological unit was the patient within a cohort.
- **Single-cell RNA-seq:** the repeated observations were cells nested within patient, tissue, and cell type. The independent biological unit for the main comparisons was the patient.
- **Spatial transcriptomics:** the repeated observations were spots nested within tissue sections. The independent unit was the patient or section, depending on the claim.

The exact unit must follow the scientific question. If the question is “where is a gene expressed?”, cells can be plotted descriptively. If the question is “do colorectal cancer patients tend to have higher tumor scores than normal-tissue scores?”, patients must provide the replication.

## The pseudoreplication problem

Suppose Patient A contributes 30,000 fibroblasts and Patient B contributes 300. A cell-level test would allow Patient A to dominate the result simply because more of that tumor was captured. It would also calculate uncertainty as though all 30,300 cells came from independent people. The resulting value can look extremely significant even when the difference is driven by one patient.

This is called **pseudoreplication**: counting correlated measurements as independent replicates.

The problem is not fixed by having a balanced number of cells per group if those cells still come from very few patients. Nor is it fixed by plotting each cell as a dot. The number of dots in a figure is not necessarily the biological sample size.

## What pseudobulk does

For the patient-level single-cell analysis, we used a pseudobulk strategy. Within each dataset, counts were summed separately for every combination of:

$$
\text{patient} \times \text{tissue} \times \text{broad cell type}.
$$

For example, all eligible fibroblasts from one patient's tumor became one fibroblast-tumor pseudobulk profile. Fibroblasts from that patient's normal tissue became another profile. The same procedure was repeated for epithelial, myeloid, endothelial, and other broad compartments.

This approach has two benefits. It respects the patient as the independent unit, and it models count data at a level where established bulk-RNA methods work well. It does sacrifice cell-to-cell detail, but that detail is not the correct source of replication for a patient-level tumor-versus-normal claim.

The workflow also required a minimum number of cells per pseudobulk profile. The primary threshold was 50 cells, with 20- and 100-cell thresholds used as sensitivity analyses. This prevented an unstable profile based on only a handful of cells from being treated like a well-sampled profile.

## What happened in our two single-cell datasets

The project used two public colorectal cancer atlases:

- GSE178341 contained 370,115 cells from 62 patients.
- GSE132465 contained 63,689 cells from 23 patients.

Together they contained 433,804 cells, but only 85 patients. Even 85 is not the sample size for every comparison because not every patient had every tissue and cell type. The fibroblast/CAF tumor-normal comparison in GSE178341, for example, had only five matched patient pairs at the primary threshold.

That comparison showed a tumor-higher score, but its paired Wilcoxon p value was 0.0625. Reporting 5,231 fibroblasts as 5,231 independent replicates would have hidden the real uncertainty.

At the descriptive cell level, fibroblasts/CAFs had the highest median annotation score, followed by other stromal, endothelial, and myeloid compartments. This was useful localization evidence. It was not a patient-level survival analysis, and neither single-cell dataset contained the progression outcomes needed to make it one.

{% include figure.liquid path="assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png" class="img-fluid rounded z-depth-1" zoomable=true alt="GSE178341 single-cell colorectal cancer atlas" %}

<div class="caption">
  The GSE178341 atlas contains hundreds of thousands of plotted cells, but those cells are nested within 62 patients. Visual resolution at the cell level does not create patient-level replication.
</div>

## Paired data are especially valuable

When tumor and normal tissue come from the same patient, each person can act as their own control. A paired comparison asks whether the score tends to change within patients, rather than comparing two unrelated groups that may differ in age, genetics, collection site, or other factors.

We used paired tests when at least three matched patients were available. If there were too few pairs but at least five patients in each tissue group, an unpaired comparison was labeled exploratory. Otherwise, the result was marked non-estimable.

“Non-estimable” is not a failure to run a test. It is a scientifically honest statement that the available independent units cannot support that comparison.

## Spatial spots create the same issue

The spatial dataset retained 16,171 spots across four tissue sections. Yet those four sections came from only two patients. The spots are measurements of locations, not 16,171 independent people. They can show spatial organization within a section, but they cannot support population-level claims about colorectal cancer patients.

This is why the paper reported section-level correlations and descriptive patient summaries without a two-patient hypothesis test. More pixels, spots, or cells improve resolution within a specimen; they do not automatically improve generalizability across people.

## A practical checklist

Before running a statistical test on single-cell or spatial data, ask:

1. What entity was independently recruited or sampled?
2. Which observations share the same patient, sample, or tissue section?
3. At what level is the scientific claim being made?
4. Does the model account for the nested structure?
5. Could one patient dominate because they contributed more cells?

If the claim is about patients, the uncertainty must ultimately come from variation across patients.

## What Our Conclusions Support

**Supported:** Hundreds of thousands of cells provide rich cellular localization. Patient-level pseudobulk analysis can test tumor-normal state differences while preserving the patient as the inferential unit.

**Not supported:** The cell count is not the patient sample size. These single-cell datasets do not independently validate progression prognosis because they do not contain progression outcomes.

The basic rule is worth remembering: **measurement depth and biological replication are different resources**. A strong study needs to know which one it has.

**Previous:** [Are neural, stromal, and EMT programs really separate?]({{ '/blog/2026/neural-stromal-and-emt-labels/' | relative_url }})<br>
**Next:** [PCA and clustering find patterns, but do not prove prognosis.]({{ '/blog/2026/pca-clustering-patterns-not-prognosis/' | relative_url }})
