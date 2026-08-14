---
layout: post
title: "Kaplan-Meier and Cox models: why do we need both?"
date: 2026-08-12 10:00:00-0400
description: Kaplan-Meier curves show survival experience, while Cox models quantify associations and can use continuous predictors and clinical covariates.
tags: Kaplan-Meier Cox-regression survival-analysis biostatistics biomarkers
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 5
related_posts: false
thumbnail: assets/img/blog/cancer-signature-series/km-discovery.png
---

> **Beyond the Tumor Cell, Part 5 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

Kaplan-Meier curves and Cox regression often appear next to each other in cancer papers. They may show the same direction: one group has earlier events, and the Cox hazard ratio is greater than one. That makes them look redundant.

They are not. They use the same time-to-event data to answer different questions.

## Why ordinary averages are not enough

In a progression study, each patient contributes a follow-up time and an event indicator. Some patients progress during follow-up. Others have not progressed by their last visit. Their exact future progression time is unknown; their observation is **right-censored**.

Discarding censored patients would waste information and create bias. Treating their last follow-up time as an event time would also be wrong. Survival methods are designed to use the information we do have.

## Kaplan-Meier estimates a curve

The Kaplan-Meier estimator describes the probability of remaining event-free over time. At each observed event time, the curve is updated according to the number of events and the number of patients still at risk.

In simplified form,

$$
\hat S(t)=\prod_{t_i \leq t}
\left(1-\frac{d_i}{n_i}\right),
$$

where $d_i$ is the number of events at time $t_i$ and $n_i$ is the number still at risk just before that time.

A Kaplan-Meier plot is valuable because it lets us see:

- when curves begin to separate;
- whether separation is sustained;
- how much censoring occurs;
- whether few patients remain late in follow-up;
- whether a model summary hides a more complicated pattern.

The log-rank test then compares the full curves under a null hypothesis of no group difference.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/km-discovery.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Kaplan-Meier curves for exploratory high-axis and low-axis colorectal cancer groups" %}

<div class="caption">
  The exploratory high-axis group experienced earlier progression-related events. The curve is an intuitive display, but the split was selected during historical exploration and is not the primary validation analysis.
</div>

## Cox regression estimates an association

The Cox proportional-hazards model asks how a predictor changes the instantaneous event rate among patients still at risk:

$$
h(t\mid x)=h_0(t)\exp(\beta x).
$$

The quantity $\exp(\beta)$ is the hazard ratio. A hazard ratio of 1.33 per standard-deviation increase in a score means that, at a given time and under the model assumptions, patients one score standard deviation higher have an estimated 33% higher instantaneous event rate.

That is not the same as saying that 33% more patients will progress, that progression happens 33% sooner, or that the score causes progression.

Cox regression can do things a basic Kaplan-Meier comparison cannot:

1. Use the expression score as a continuous variable.
2. Estimate an effect size with a confidence interval.
3. Include stage or other measured covariates.
4. Produce cohort-specific estimates that can be pooled by meta-analysis.
5. Test interactions or more complex model specifications when the data support them.

## Why grouping can lose information

To draw two Kaplan-Meier curves, a continuous score is often divided into high and low groups. A patient just above the cutoff may be biologically similar to a patient just below it, yet the plot places them in different groups. Meanwhile, two patients at opposite ends of the high group are treated as equivalent.

A continuous Cox model uses the ordering and distance of the score. It avoids a cutoff selected to produce attractive separation and generally retains more statistical information.

Grouped displays are still useful. They communicate the time course and can reveal departures from model assumptions. The solution is not to choose one method, but to assign each method the right role.

## Their assumptions are also different

The log-rank test is most sensitive when hazards are roughly proportional over time. The standard Cox model explicitly assumes that the hazard ratio associated with a predictor is constant over time unless the model includes a time-varying effect.

Kaplan-Meier curves can help us notice obvious violations, such as curves that cross. Formal diagnostics are still needed for a Cox model. Neither method corrects confounding simply because it handles censoring correctly.

## How the two methods were used in this project

The historical analysis used PCA and $k$-means to form high- and low-axis groups, then displayed Kaplan-Meier curves and log-rank values. Those plots explain how the original signal was discovered. Because the dataset partition was chosen during a broad search that included survival behavior, they are exploratory.

The primary evidence used a continuous frozen 87-gene score:

- Eleven GEO cohorts were modeled separately with Cox regression.
- The cohort log-hazard ratios were combined using random-effects meta-analysis.
- The pooled hazard ratio was 1.33, with a 95% confidence interval from 1.19 to 1.49.
- A DFS-only analysis and endpoint-family exclusions checked whether the result depended on the mixed endpoint definitions.
- The locked continuous score was transported to TCGA, where the hazard ratio was 1.25 for progression-free interval.

This division of labor is deliberate. Kaplan-Meier shows what the grouped survival experience looks like. Cox estimates the continuous association. Meta-analysis asks whether that association is reproducible across cohorts.

## A note about “PFS”

The GEO cohorts did not all measure strict progression-free survival. Seven used disease-free survival, while others used recurrence-free, metastasis-free, metastasis/recurrence-free, or strict PFS definitions. We therefore call the combined outcome **progression-related** or **PFS-like**, not strict PFS.

This wording matters. A sophisticated model cannot make heterogeneous endpoint definitions identical. Statistical precision does not replace clinical-definition precision.

## When would the two results disagree?

A log-rank comparison may be nonsignificant while a continuous Cox model is informative, especially if an arbitrary cutoff weakens the grouped contrast. A grouped comparison can also appear strong after an outcome-optimized cutoff even when a prespecified continuous analysis is less convincing. Crossing curves may make a single Cox hazard ratio hard to interpret. Covariate adjustment can change the Cox estimate while the unadjusted Kaplan-Meier curves remain unchanged.

Disagreement is therefore diagnostic. It should prompt us to inspect cutoffs, assumptions, covariates, event counts, and the analysis plan rather than selecting whichever result looks better.

## What we can and cannot say

**Supported:** Kaplan-Meier curves provide an interpretable grouped display, while Cox models quantify continuous and adjusted associations. Using both can make the evidence easier to inspect.

**Not supported:** Agreement between a curve and a hazard ratio does not establish causality, eliminate confounding, or turn a PFS-like composite into strict PFS.

The two methods are partners, not duplicates: one helps us see the survival experience, and the other helps us estimate how a predictor relates to it.

**Previous:** [PCA and clustering find patterns, but do not prove prognosis.]({{ '/blog/2026/pca-clustering-patterns-not-prognosis/' | relative_url }})<br>
**Next:** [How independent must a biomarker be?]({{ '/blog/2026/how-independent-must-a-biomarker-be/' | relative_url }})
