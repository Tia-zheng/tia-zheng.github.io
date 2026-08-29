---
layout: post
title: "When adjustment removes the biology we wanted to study"
date: 2026-08-14 10:00:00-0400
description: Adjustment can control confounding or redefine the scientific question, and cross-fitting cannot repair noisy measurements or a misspecified model.
tags: causal-inference residualization cross-fitting simulation biomarkers biostatistics
categories: cancer-signature-series
series: beyond-the-tumor-cell
series_order: 7
related_posts: false
thumbnail: assets/img/blog/cancer-signature-series/simulation-stress-test.png
---

> **Beyond the Tumor Cell, Part 7 of 7.** [View the complete series.]({% link _pages/cancer-signature-series.md %})

Statistical adjustment is often described as “making a comparison fair.” We include stage, tumor subtype, stromal abundance, or purity so that the remaining association is less confounded.

That description is useful, but incomplete. Adjustment does not simply clean a result. It changes the question.

For our expression axis, the unadjusted question was:

> Is a fibroblast-rich multigene tumor state associated with progression-related outcome?

After removing CMS4-like, CAF, stromal, and purity components, the question became:

> Is the part of this score that cannot be predicted from our measured composition proxies still associated with outcome?

Both are legitimate. They are not the same estimand.

## Confounder, mediator, or part of the exposure?

Before adjusting for a variable, we should decide what role it may play.

A **confounder** is a common cause of the exposure and outcome. Failing to account for it can create a misleading association.

A **mediator** lies on the pathway from exposure to outcome. Adjusting for it removes part of the effect we may be trying to estimate.

A **component or proxy of the exposure** measures nearly the same underlying construct. Adjusting for it asks whether anything remains after subtracting shared information.

CMS4-like affinity, CAF abundance, stromal score, purity, and the frozen axis are all expression-derived descriptions of tumor composition or state. Their exact causal relationships are not known, and some share genes or tissue architecture. Treating them as ordinary confounders would be too simple.

This is the adjustment paradox: removing composition may reduce confounding, but composition may also be the main biology encoded by the score.

## Residualization as a subtraction problem

Residualization predicts the axis from measured composition variables, then subtracts the prediction:

$$
\text{residual axis}
= \text{observed axis}
- \widehat{\text{axis from CMS4, CAF, stroma, purity}}.
$$

The residual is the portion not explained by that prediction model. It is not automatically a pure tumor-cell-intrinsic signal. It can still contain unmeasured composition, measurement error, nonlinear relationships, noise, and biology not represented by the comparators.

The scale also matters. In this project, held-out residuals remained in units of one standard deviation of the **original** axis and were not standardized again. That allowed the residual hazard ratios to retain a common interpretation across cohorts. Re-standardizing every residual would make a weak, noisy remainder appear artificially large by defining one residual standard deviation as the exposure unit.

## Why cross-fitting was used

If the same cohort is used to learn the subtraction model and test the residual, the model can adapt to cohort-specific noise. Cross-fitting separates those roles.

For each GEO cohort:

1. Hold that cohort out.
2. Fit the axis-versus-composition model in the other ten cohorts.
3. Freeze the fitted coefficients.
4. Predict the axis in the held-out cohort.
5. Compute held-out residuals.
6. Join survival data only after residual construction and fit the Cox model.

Each patient's residual was therefore produced by a model that had not seen that cohort or any survival outcome. A ridge version was used as a sensitivity analysis to stabilize correlated predictors.

This design controls a specific problem: information leakage from the held-out cohort into nuisance-model training. It does not guarantee that the nuisance model is biologically correct.

## What the residual analysis found

The pooled GEO residual-axis hazard ratio was 1.12, with a 95% confidence interval from 0.77 to 1.64. The external TCGA residual-axis hazard ratio was 1.31, with a confidence interval from 0.63 to 2.74. The linear and ridge versions agreed closely.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/residual-axis-forest.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Forest plot of cross-fitted residual-axis Cox estimates across GEO cohorts and TCGA" %}

<div class="caption">
  Cross-fitted residual-axis estimates after removing measured CMS4-like, CAF, stromal, and purity components. The pooled estimates are uncertain and do not show a detectable independent residual association.
</div>

“Not detected” is the right phrase. The intervals are wide enough to include moderate effects. The result does not prove that the true residual effect is exactly zero, and it does not invalidate the original stromal-state association.

It narrows the conclusion: this dataset does not provide clear evidence that the axis adds prognosis beyond the measured composition variables.

## Why we simulated the method

In patient data, the true composition-independent effect is unknown. We cannot calculate bias because we do not know the correct answer. Simulation solves this by creating data with a known data-generating process.

The benchmark preserved the 11 GEO cohort sizes and overall event structure. It compared six methods across 13 scenarios with 500 repetitions each, producing 39,000 pooled meta-analysis fits.

Under an ideal null with a correctly measured composition variable and an appropriate linear model, cross-fitted linear residualization had a false-positive rate of 0.038, close to the nominal 0.05. When the composition comparators were noisy, the rate increased to 0.102. When the composition relationship was nonlinear, it increased to 0.248.

{% include figure.liquid path="assets/img/blog/cancer-signature-series/simulation-false-positive-rate.png" class="img-fluid rounded z-depth-1" zoomable=true alt="False-positive rates for adjustment methods under simulated scenarios" %}

<div class="caption">
  Simulation reveals the boundary of the method. Cross-fitting controls leakage under the ideal null, but noisy composition measurements and nonlinear relationships can still produce false-positive residual associations.
</div>

The simulation does not prove which scenario describes the real tumors. It shows what must be true for the method to behave well and where it can fail.

## Adjustment cannot create a perfect measurement

There are three reasons a residual may be misleading even with correct cross-fitting.

**Measurement error:** CAF and purity estimates are themselves imperfect. Subtracting an imperfect proxy leaves part of the original composition signal behind.

**Model misspecification:** A linear model cannot fully remove a curved or interactive relationship.

**Shared construction:** If an axis and comparator reuse genes or reflect the same tissue architecture, their statistical separation may not correspond to separable biological processes.

Cross-fitting addresses none of these by itself. It is a design tool, not a universal correction.

## Did adjustment remove “too much”?

That depends on the question.

For a clinical incremental-value question, removing known stromal information is appropriate. A new assay should demonstrate what it adds.

For a tissue-state question, the original axis is the relevant exposure. Removing stroma would discard the phenomenon of interest.

For a causal question, neither analysis is sufficient without a defensible causal model and stronger experimental or longitudinal evidence.

The safest interpretation reports both analyses and names their targets. The original score captures a prognostic microenvironmental state. The residual analysis asks for additional information beyond measured composition and does not detect it clearly.

## What Our Conclusions Support

**Supported:** Cross-fitted residualization prevents the held-out cohort and its outcomes from shaping the nuisance model. Under the stated linear model, the analysis did not detect a composition-independent residual association.

**Not supported:** Cross-fitting does not prove that composition was measured without error, that the nuisance relationship was correctly specified, or that the original stromal-state association was false. The residual result is not evidence for an exactly zero biological effect.

## Five lessons from the full series

1. Cancer biology extends beyond malignant cells to the tissue ecosystem.
2. Pathway labels are hypotheses; reused genes do not prove a unified mechanism.
3. Cells and spatial spots are measurements, not independent patients.
4. PCA, clustering, Kaplan-Meier, and Cox models answer different questions.
5. Adjustment is meaningful only after we specify what we are trying to estimate.

The final scientific story is therefore more modest than the starting idea, but more reliable. We found a reproducible, stromal-dominant colorectal cancer state associated with progression-related outcomes. We did not establish a tumor-intrinsic neural mechanism, a composition-independent prognostic signal, or a clinical assay ready for use.

That is not an incomplete ending. It is what successful analysis should do: transform an interesting hypothesis into the most precise claim the evidence can support.

**Previous:** [How independent must a biomarker be?]({{ '/blog/2026/how-independent-must-a-biomarker-be/' | relative_url }})<br>
**Return to:** [Beyond the Tumor Cell: the complete series.]({% link _pages/cancer-signature-series.md %})
