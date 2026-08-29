---
layout: post
title: "How independent must a biomarker be?"
date: 2026-08-13 10:00:00-0400
description: Prognostic association, biological meaning, and added clinical value are different standards, and a marker can meet one without meeting all three.
tags: biomarkers prognostic-models tumor-microenvironment CMS4 clinical-translation
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 6
related_posts: false
thumbnail: assets/img/blog/cancer-signature-series/tcga-module-forest.png
---

> **Beyond the Tumor Cell, Part 6 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

Suppose a gene-expression score is associated with shorter survival. Then we adjust for tumor stage and the association remains. Next we adjust for CMS4, cancer-associated fibroblasts, stromal abundance, and tumor purity, and the association becomes uncertain.

Was the score a real biomarker before adjustment? Did we discover that it was “just stroma”? Or did we subtract the biology that the score was designed to represent?

These questions cannot be answered until we define what kind of independence we want.

## Three different standards

The word _biomarker_ covers several scientific goals. At least three should be separated.

### 1. Prognostic association

A prognostic marker is associated with a future clinical outcome. The basic question is:

> Do patients with different score values experience different progression-related outcomes?

Our frozen 87-gene score met this standard retrospectively. The association appeared across 11 GEO cohorts and in locked TCGA transport.

### 2. Biological meaning

A marker can represent a coherent tissue state even if that state overlaps an established category. The question becomes:

> What cells and processes generate the score?

The single-cell, module, differential-expression, and spatial analyses consistently pointed toward fibroblast-rich stromal remodeling. That makes the score biologically interpretable, but not necessarily biologically novel.

### 3. Incremental clinical value

A clinically incremental marker must improve prediction beyond information already available to a clinician or model. The question is:

> Does adding the score improve prediction after stage and established molecular or stromal measures are already known?

This is the strongest standard. A significant univariable hazard ratio is not enough. We would want prespecified external validation, measures of discrimination and calibration, clinically relevant time horizons, and evidence that the added information could change a decision.

Our current study does not establish this third standard.

## “Independent of stage” is not “independent of biology”

Stage records the anatomical extent of disease. A transcriptomic score may provide information about tumor state that is not captured by stage. Among seven cohorts with usable stage, the original axis remained associated with outcome after stage adjustment: hazard ratio 1.26, with a 95% confidence interval from 1.06 to 1.50.

That is useful evidence, but stage was available for only 943 patients and 225 events. It does not prove clinical utility across all cohorts.

CMS4, CAF abundance, stromal score, and purity are different kinds of variables. They are close to the biological construct measured by the axis. The frozen score had median within-cohort rank correlations of 0.904 with CMS4-like affinity, 0.909 with CAF or a CAF proxy, 0.947 with a stromal proxy, and -0.858 with a purity proxy.

Those are not small overlaps. They tell us that the axis and the comparators are observing much of the same tissue state.

## Redundancy can still be scientifically informative

Imagine two thermometers built with different materials. If both track temperature closely, the second may add little predictive information once the first is known. It can still confirm that temperature is the underlying quantity.

Likewise, a stromal-dominant gene score can help us understand why a tumor group has poor outcomes even if it does not outperform CMS4 or a CAF measure. It may connect the bulk survival pattern to particular genes, cell compartments, or spatial regions. That is biological clarification rather than incremental prediction.

The mistake would be to switch standards after seeing the results: calling the score a novel independent biomarker when it is significant alone, then calling overlap with established biology a mechanistic discovery without acknowledging redundancy.

## What the comparator analysis found

The study used several complementary checks:

- Within-cohort correlations measured overlap with CMS4-like, CAF, stromal, and purity comparators.
- Nested Cox models asked whether adding the axis improved models that already contained stage, CMS4 affinity, and one stromal comparator.
- Cross-fitted residualization tested the portion of the axis not predicted by the four composition measures.
- Stage-adjusted models evaluated a familiar clinical covariate in the cohorts where it was available.

Adding the axis did not improve the primary CMS-adjusted model likelihood, and no CMS/stroma-residualized axis model reached the prespecified significance threshold. Collinearity was substantial, which is expected when predictors describe closely related tissue states.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/tcga-module-forest.png" class="img-fluid rounded z-depth-1" zoomable=true alt="TCGA Cox estimates for broad gene modules" %}

<div class="caption">
  Broad module estimates in TCGA illustrate that related biological components can have different effect sizes and uncertainty. They are secondary analyses, not replacements for the frozen primary score.
</div>

The correct conclusion is not that the original association disappeared or was false. The correct conclusion is that the study did not detect added prognostic information beyond the measured CMS4/stromal composition.

## How much independence is enough?

There is no universal threshold because the answer depends on the intended claim.

- **“The score is associated with outcome”** requires a reproducible association with honest uncertainty.
- **“The score marks a biological state”** requires localization, module behavior, orthogonal data, and mechanistic plausibility.
- **“The score is independent of a known subtype”** requires prespecified models that compare the score with that subtype.
- **“The score improves clinical prediction”** requires external validation of added discrimination, calibration, and decision value.
- **“The score identifies a causal target”** requires perturbation and mechanistic experiments.

A paper should choose the row it can support. Moving downward requires new evidence, not stronger adjectives.

## Independence can be the wrong scientific goal

If our biological question is “does a fibroblast-rich state matter?”, demanding independence from every fibroblast and stromal measure is strange. We would be asking the score to remain informative after removing the thing it represents.

If our clinical question is “should a hospital run this assay in addition to existing tests?”, that same independence becomes essential. Redundant information may not justify added cost or complexity.

The analysis is identical only on the surface. The scientific target, or **estimand**, has changed.

This is why the paper uses cautious language. It describes a stromal-dominant prognostic state, not a clinically independent biomarker. That wording preserves the result that is supported without implying the result we hoped to see.

## What Our Conclusions Support

**Supported:** The score is reproducibly associated with progression-related outcome and has a coherent fibroblast-rich, CMS4-like interpretation. Its stage-adjusted association persisted in the subset with stage data.

**Not supported:** The study does not show incremental prognostic value beyond CMS4 or generic stromal abundance, prospective clinical utility, or causal independence.

A biomarker does not need to be independent of all known biology to be meaningful. It does need to be independent enough for the specific claim we make about it.

**Previous:** [Kaplan-Meier and Cox models: why do we need both?]({{ '/blog/2026/kaplan-meier-and-cox/' | relative_url }})<br>
**Next:** [When adjustment removes the biology we wanted to study.]({{ '/blog/2026/when-adjustment-removes-biology/' | relative_url }})
