---
layout: post
title: Dimensionality reduction and sparse regression in single-cell cancer analysis
date: 2026-06-24 15:30:00-0400
description: A readable guide to PCA, t-SNE, UMAP, linear regression, and sparse survival modeling using colorectal cancer single-cell analysis as an example.
tags: single-cell bioinformatics dimensionality-reduction regression cancer-biology
categories: methods
related_posts: false
thumbnail: assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png
---

Single-cell RNA sequencing lets us measure thousands of genes in hundreds of thousands of cells. That is powerful, but it also creates a practical problem: no one can look directly at a 20,000-dimensional gene-expression table and immediately understand what the cells are doing.

This post is about two families of methods that help with that problem:

1. **Dimensionality reduction**, which turns many gene measurements into a two-dimensional or low-dimensional view that we can inspect.
2. **Regression and sparse regression**, which connect molecular measurements to an outcome, such as patient survival.

I will explain the intuition first, then show how these ideas appeared in my colorectal cancer single-cell analysis using public `GSE178341` data. The goal is not to prove every theorem. The goal is to understand what each method is trying to preserve, what its math means, and how the result should be interpreted in a real analysis.

## The experiment in one paragraph

The analysis folder `single_cell/GSE178341_crc_immune_hubs` reproduces and extends a public-data workflow for colorectal cancer single-cell RNA-seq. The detailed-subgroup setup contains **87 cell subgroups** and **370,115 cells** across epithelial, stromal, immune, plasma, mast, and B-cell compartments. The survival-linked analysis uses **84 CRC-axis genes**, **1,719 bulk PFS samples**, and **383 PFS events**. For visualization, the workflow uses the public paper-style t-SNE coordinates and treats UMAP as an additional generated view when public UMAP coordinates are not available.

## Part 1: PCA, t-SNE, and UMAP

Imagine every cell as a point. If we measure 20,000 genes, then each cell lives in a 20,000-dimensional space. Cells with similar expression profiles should be close to each other; cells with different states should be farther apart.

Mathematically, the expression matrix can be written as

$$
X \in \mathbb{R}^{n \times p},
$$

where $$n$$ is the number of cells and $$p$$ is the number of genes. A dimensionality-reduction method creates a lower-dimensional representation

$$
Z \in \mathbb{R}^{n \times d},
\quad d \ll p.
$$

For visualization, $$d = 2$$ is common. For modeling, $$d$$ may be 10, 30, or 50. The hard part is deciding what information should be preserved when we compress the data.

### PCA: finding the main axes of variation

Principal component analysis (PCA) is the simplest and most important starting point. It asks: which direction through the data explains the most variation?

Before PCA, we usually center the data:

$$
X_c = X - \bar{X}.
$$

The covariance matrix is

$$
\Sigma = \frac{1}{n - 1} X_c^\top X_c.
$$

PCA finds directions $$v_1, v_2, \ldots$$ that solve

$$
\Sigma v_j = \lambda_j v_j.
$$

The vector $$v_j$$ is a principal component direction, and $$\lambda_j$$ is the amount of variance explained by that direction. The coordinate of cell $$i$$ on component $$j$$ is

$$
z_{ij} = x_i^\top v_j.
$$

Another way to write PCA is with singular value decomposition:

$$
X_c = UDV^\top.
$$

Here, the columns of $$V$$ are the principal directions, and the singular values in $$D$$ tell us how much signal each direction carries.

The intuition is like rotating a camera. PCA does not bend the data. It rotates the coordinate system so the first axis points along the strongest linear pattern, the second axis points along the next strongest independent pattern, and so on.

In the single-cell workflow, PCA was used inside the official Scissor branch during Seurat preprocessing. For each detailed cell subgroup, the script normalized expression, selected variable genes, scaled them, ran `RunPCA`, and built a nearest-neighbor graph from the PCA space. So in this analysis, PCA was less about making the final figure and more about building a cleaner graph for downstream modeling.

### t-SNE: keeping nearby cells together

t-SNE is a visualization method. It is especially popular in single-cell atlases because it often separates local cell neighborhoods into visible islands.

The main idea is to turn distances into probabilities. In the original high-dimensional space, t-SNE asks: how likely is cell $$j$$ to be a neighbor of cell $$i$$?

$$
p_{j|i} =
\frac{\exp(-\lVert x_i - x_j \rVert^2 / 2\sigma_i^2)}
{\sum_{k \ne i} \exp(-\lVert x_i - x_k \rVert^2 / 2\sigma_i^2)}.
$$

The scale parameter $$\sigma_i$$ is adjusted locally, which means each cell gets a neighborhood size that works for its local density.

In the two-dimensional map, t-SNE defines another similarity:

$$
q_{ij} =
\frac{(1 + \lVert y_i - y_j \rVert^2)^{-1}}
{\sum_{k \ne l} (1 + \lVert y_k - y_l \rVert^2)^{-1}}.
$$

The low-dimensional map is optimized so that $$Q$$ resembles $$P$$. The loss function is the Kullback-Leibler divergence:

$$
KL(P \Vert Q)
= \sum_{i \ne j} p_{ij} \log \frac{p_{ij}}{q_{ij}}.
$$

This loss is asymmetric. It strongly penalizes putting true neighbors far apart. That is why t-SNE is good at preserving local neighborhoods. The tradeoff is that the distance between two distant t-SNE clusters should not be over-interpreted.

In my experiment, the GSE178341 paper-style atlas uses public t-SNE coordinates. I used these coordinates as the common visual space for checking cell populations and overlaying Scissor labels.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Public GSE178341 global t-SNE coordinates. In this project, t-SNE acted as the shared atlas map for visualizing colorectal cancer cell states and metadata.
</div>

### UMAP: preserving a neighbor graph

UMAP is also a nonlinear visualization method, but it starts from a graph. First, it builds a k-nearest-neighbor graph in the original high-dimensional space. Then it tries to build a two-dimensional graph with similar connectivity.

For nearby cells, UMAP assigns a fuzzy edge weight. A simplified form is

$$
w_{ij}
= \exp\left(-\frac{d(x_i, x_j) - \rho_i}{\sigma_i}\right),
$$

where $$d(x_i, x_j)$$ is the distance between cells, $$\rho_i$$ represents the distance to the nearest neighbor, and $$\sigma_i$$ controls local scaling.

In the low-dimensional map, UMAP defines a similar edge probability:

$$
\hat{w}_{ij}
= \frac{1}{1 + a\lVert y_i - y_j \rVert^{2b}},
$$

where $$a$$ and $$b$$ control the shape of the curve. UMAP then minimizes a cross-entropy-like objective between the high-dimensional graph and low-dimensional graph:

$$
C =
\sum_{i,j}
\left[
w_{ij}\log\frac{w_{ij}}{\hat{w}_{ij}}
+ (1-w_{ij})\log\frac{1-w_{ij}}{1-\hat{w}_{ij}}
\right].
$$

The intuition is that connected cells should stay close, while disconnected cells should have room to move apart. Compared with t-SNE, UMAP often gives a more continuous-looking map and can be faster for large data. But it is still a visualization, not a direct proof of biological distance.

In this project, UMAP was treated as an additional analysis view when generated from the public processed H5AD object. The original paper-style visual output is t-SNE based, so the blog and figures focus on t-SNE for the concrete experiment results.

| Method | Question it answers | What it preserves best | How it appeared in this project |
| --- | --- | --- | --- |
| PCA | What are the strongest linear axes of variation? | Global variance | Used before nearest-neighbor graph construction in the official Scissor workflow |
| t-SNE | Which cells are local neighbors? | Local neighborhoods | Used as the public atlas coordinate system for visualization and overlays |
| UMAP | What neighbor graph structure can be shown in 2D? | Local graph connectivity, often with more continuity than t-SNE | Treated as an additional generated visualization when public UMAP coordinates were absent |

### What the dimensionality-reduction results meant here

For **PCA**, the most important result was not a final picture. PCA created a compact coordinate system that made the official Scissor workflow computationally practical inside each detailed subgroup. Instead of building a graph directly from thousands of noisy gene dimensions, the workflow built nearest-neighbor structure from the leading principal components after normalization, variable-gene selection, and scaling. This helped stabilize the subgroup-level cell-cell networks that were later used by the survival model.

For **t-SNE**, the result was the main visual atlas. The public GSE178341 t-SNE map showed that the cells were organized into recognizable compartments, including epithelial, stromal, myeloid, T/NK/ILC, B, plasma, and mast-cell regions. Because the same coordinates could be reused across overlays, t-SNE became the common map for asking where Scissor+ and Scissor-style positive cells appeared. In the strongest official Scissor+ examples, those labels appeared prominently in endothelial, fibroblast, myofibroblast, pericyte, and selected myeloid/DC populations.

For **UMAP**, the result was more cautious. The project README records that the full workflow computes UMAP only when no public UMAP coordinates are present, and that the original paper-style views are t-SNE based. So UMAP is useful as an additional generated quality-control view of the processed public data, but I do not treat it as the main published atlas result. This distinction matters because a UMAP generated locally can support exploration, while the public t-SNE coordinates provide the shared reference for the figures shown in this post.

## Part 2: Regression and sparse regression

Dimensionality reduction helps us look at cells. Regression helps us ask whether a measurement is associated with an outcome.

In cancer biology, an outcome might be tumor type, treatment response, survival time, progression-free survival, or a risk group. In this project, the outcome was progression-free survival (PFS) from bulk data, linked back to single-cell profiles through shared genes.

### Linear regression: fitting a line in many dimensions

The basic linear regression model is

$$
y = X\beta + \epsilon.
$$

Here:

- $$y$$ is the outcome.
- $$X$$ is the feature matrix.
- $$\beta$$ is the coefficient vector.
- $$\epsilon$$ is noise that the model cannot explain.

Ordinary least squares chooses $$\beta$$ to minimize squared error:

$$
\hat{\beta}
= \arg\min_\beta \lVert y - X\beta \rVert_2^2.
$$

If $$X^\top X$$ is invertible, the closed-form solution is

$$
\hat{\beta}
= (X^\top X)^{-1}X^\top y.
$$

This equation is useful because it shows the problem clearly: when genes are highly correlated or when there are more genes than samples, $$X^\top X$$ becomes unstable or non-invertible. That is common in biology.

### Sparse regression: keeping only the strongest signals

Sparse regression adds a penalty so that many coefficients become exactly zero. The most common example is LASSO:

$$
\hat{\beta}
= \arg\min_\beta
\left(
\frac{1}{2n}\lVert y - X\beta \rVert_2^2
+ \lambda \lVert \beta \rVert_1
\right),
$$

where

$$
\lVert \beta \rVert_1 = \sum_j |\beta_j|.
$$

The tuning parameter $$\lambda$$ controls how aggressive the shrinkage is. When $$\lambda$$ is small, more coefficients remain nonzero. When $$\lambda$$ is large, more coefficients are forced to zero.

This matters in gene-expression analysis because a dense model may use thousands of weak signals, but a sparse model tries to identify a smaller set of informative features. That makes interpretation easier.

### Survival modeling: the Cox model

For survival outcomes, the response is not just a number. A patient has a follow-up time and an event indicator. Some patients may not have had the event by the end of follow-up, which is called censoring.

The Cox proportional hazards model writes the hazard as

$$
h(t \mid x)
= h_0(t)\exp(x^\top \beta).
$$

The baseline hazard $$h_0(t)$$ describes the background risk over time. The linear predictor $$x^\top\beta$$ shifts that risk up or down. If a coefficient is positive, higher expression increases the modeled hazard. If it is negative, higher expression decreases the modeled hazard.

Scissor-style methods use this survival-modeling idea to connect single-cell expression profiles to bulk clinical outcomes.

## What we did in the GSE178341 experiment

The experiment had two related branches.

The **official Scissor Cox branch** used the published Scissor workflow. For each detailed subgroup, it created a Seurat object, normalized expression, selected variable features, ran PCA, built a cell-cell network, and used Scissor with a Cox survival family. Out of 87 detailed subgroups, **84 completed**, **2 were skipped**, and **1 failed/stalled**.

The **local Scissor-style branch** was an axis-focused Cox analysis. It did not run the full official cell-cell network. Instead, it compared bulk and single-cell profiles over **84 CRC-axis genes**, computed Cox-style scores, applied FDR correction, and labeled cells as positive, negative, or neutral. This branch is useful for comparison, but it should not be described as the published Scissor algorithm.

The PFS input contained **1,719 samples**, **383 events**, and follow-up times ranging from **0.3** to **201.0** months, with a median around **41.4** months.

In plain language, the regression question was: if a cell looks more like the expression pattern associated with shorter or longer PFS in bulk data, can we identify that cell or subgroup as clinically relevant? Sparse modeling is important here because the gene list is still large enough to contain correlated signals, and because a smaller set of genes is easier to interpret biologically.

### Official Scissor results

The strongest official Scissor+ signals were concentrated in stromal and vascular-associated populations. The top detailed subgroups by Scissor+ percentage were:

| Subgroup | Cell type label | Scissor+ cells | Scissor+ % |
| --- | --- | --- | --- |
| cS07 | Endo capillary-like | 51 / 100 | 51.00 |
| cS14 | Endo | 74 / 146 | 50.68 |
| cS23 | Fibro BMP-producing | 108 / 233 | 46.35 |
| cS26 | Myofibro | 114 / 246 | 46.34 |
| cS11 | Endo proif | 90 / 203 | 44.34 |
| cS30 | CAF CCL8 Fibro-like | 116 / 262 | 44.27 |
| cM03 | DC1 | 135 / 309 | 43.69 |
| cS16 | Pericyte | 85 / 198 | 42.93 |
| cS04 | Endo | 84 / 199 | 42.21 |
| cS29 | MMP3+ CAF | 162 / 384 | 42.19 |

At the broad-lineage level, stromal cells had the strongest official Scissor enrichment: **36.49% Scissor+** and **34.85% Scissor-** among tested stromal cells. Myeloid cells had lower overall percentages, but the DC1 subgroup cM03 still appeared in the top detailed-subgroup list.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/cs07-official-scissor-tsne-status.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/cs07-scissor-style-tsne-status.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Example t-SNE overlays for cS07. The official Scissor branch and the Scissor-style branch are related but not identical, so their labels should be interpreted separately.
</div>

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-ranked-percent-positive.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Official Scissor+ percentage ranked across detailed subgroups. The top results are dominated by endothelial, fibroblast, myofibroblast, pericyte, and related stromal compartments.
</div>

### Scissor-style results

The Scissor-style branch produced much stronger positive labeling in some stromal/pericyte populations. Several subgroups had nearly all cells labeled positive, including:

| Subgroup | Cell type label | Positive cells | Positive % |
| --- | --- | --- | --- |
| cS18 | Pericyte | 234 / 234 | 100.00 |
| cS04 | Endo | 199 / 199 | 100.00 |
| cS20 | Pericyte prolif | 44 / 44 | 100.00 |
| cS19 | Pericyte | 374 / 374 | 100.00 |
| cS21 | Fibro stem cell niche | 541 / 542 | 99.82 |
| cS31 | CAF stem niche Fibro-like | 491 / 492 | 99.80 |
| cS32 | Smooth Muscle | 879 / 881 | 99.77 |
| cS17 | Pericyte | 382 / 383 | 99.74 |

This difference is important. A broad positive rate in the Scissor-style branch does not mean the same thing as a Scissor+ call in the official branch, because the model structure is different. The Scissor-style branch is useful as a focused comparison around CRC-axis survival association, while the official branch is the better match to the published Scissor method.

### Gene-level interpretation

After identifying the top official Scissor+ subgroups, I compared CRC-axis gene activity in Scissor+ cells versus non-Scissor+ cells within each subgroup. The top driver-gene summaries repeatedly highlighted genes such as **VEGFA**, **ENO2**, **TGFBR2**, **POSTN**, **VCAN**, **COL5A1**, **CCL2**, **RGS5**, and **ACTA2**.

For example:

- cS07, the top official Scissor+ subgroup, had high-ranking genes including **VIM**, **VEGFA**, **ENO2**, **POSTN**, **DCN**, **SYP**, **COL1A1**, and **CCL2**.
- cM03, the DC1 subgroup, highlighted **TGFBR2**, **EPHB4**, **ENO2**, **VEGFA**, **CCL2**, **ACTA2**, **SPARC**, and **VCAN**.
- cS26, the myofibroblast subgroup, highlighted **EFNB2**, **ENO2**, **TGFBR2**, **VEGFA**, **VCAN**, **CCL2**, **MMP14**, and **RGS5**.

These genes should not be read as simply "most expressed genes." They were ranked by how much higher their activity was in official Scissor+ cells compared with non-Scissor+ cells inside the same subgroup.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-full-atlas-highlight-page-01.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-top10-driver-gene-heatmap.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Full-atlas highlighting shows where Scissor+ cells sit in the broader t-SNE map. The gene heatmap connects those labels back to CRC-axis gene activity.
</div>

## How I interpret these methods together

PCA, t-SNE, and UMAP help answer a structural question: how are cells arranged relative to one another? PCA gives a stable linear summary. t-SNE gives a local atlas view. UMAP gives a graph-based alternative that can be useful for continuous structures.

Regression asks a different question: how do molecular features relate to an outcome? Sparse regression adds interpretability by shrinking weak signals and emphasizing a smaller set of features. Cox modeling adapts this idea to survival data.

In this colorectal cancer analysis, dimensionality reduction helped organize and visualize the single-cell atlas, while Scissor and Scissor-style survival modeling linked cell states and CRC-axis genes to PFS-associated signals. The strongest official Scissor+ patterns appeared in stromal, endothelial, fibroblast, pericyte, myofibroblast, and selected myeloid/DC populations, with driver-gene summaries pointing toward angiogenesis, extracellular matrix, inflammatory signaling, and TGF-beta-related biology.
