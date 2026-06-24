---
layout: post
title: Dimensionality reduction and sparse regression in single-cell cancer analysis
date: 2026-06-24 15:30:00-0400
description: PCA, t-SNE, UMAP, and sparse regression through a colorectal cancer single-cell example.
tags: single-cell bioinformatics dimensionality-reduction regression cancer-biology
categories: methods
related_posts: false
thumbnail: assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png
---

Single-cell RNA-seq data is high-dimensional: each cell is measured across thousands of genes, but the biological patterns we want to interpret are usually lower dimensional. Cells may separate by lineage, state, tissue context, tumor program, or clinical association. The practical question is how to turn a large gene-by-cell matrix into a view that can be inspected and a model that can be tested.

In my colorectal cancer single-cell analysis using public data from `GSE178341`, I used dimensionality reduction to inspect cell-state structure and regression-style models to connect cell profiles with progression-free survival (PFS). This post summarizes the intuition behind PCA, t-SNE, UMAP, linear regression, and sparse regression, then links those ideas to the analysis workflow.

## Part 1: PCA, t-SNE, and UMAP

The input can be written as a matrix

$$
X \in \mathbb{R}^{n \times p},
$$

where rows are cells and columns are genes. In single-cell data, $$p$$ is often much larger than the number of biological programs we expect to interpret. Dimensionality reduction tries to build a smaller representation

$$
Z \in \mathbb{R}^{n \times d}, \quad d \ll p,
$$

that keeps the important structure while removing noise and redundancy.

### PCA: a linear view of strongest variation

Principal component analysis (PCA) is the most direct starting point. After centering and usually scaling the data, PCA finds orthogonal directions that explain the largest variance in the dataset.

If $$X_c$$ is the centered expression matrix, the covariance matrix is

$$
\Sigma = \frac{1}{n - 1} X_c^\top X_c.
$$

PCA solves the eigenvalue problem

$$
\Sigma v_j = \lambda_j v_j,
$$

where each eigenvector $$v_j$$ is a principal direction and each eigenvalue $$\lambda_j$$ measures how much variance is explained along that direction. The projected cell coordinates are

$$
z_{ij} = x_i^\top v_j.
$$

The intuition is geometric: PCA rotates the gene-expression coordinate system so that the first axis captures the strongest linear variation, the second axis captures the next strongest variation orthogonal to the first, and so on.

In the `GSE178341_crc_immune_hubs` workflow, PCA appears in the official Scissor detailed-subgroup branch through Seurat preprocessing. For each detailed subgroup, the script normalizes cells, selects variable genes, scales them, runs `RunPCA`, and then builds a shared nearest-neighbor graph from the PCA space. In that setting, PCA is not mainly the final figure. It is an intermediate representation that makes graph construction more stable and computationally manageable.

### t-SNE: preserving local neighborhoods

t-SNE is a nonlinear visualization method designed to preserve local neighborhoods. Instead of trying to preserve all pairwise distances, it asks: if two cells are close in the high-dimensional space, can they remain close in the two-dimensional map?

In the high-dimensional space, t-SNE converts distances into conditional probabilities. A simplified form is

$$
p_{j|i} =
\frac{\exp(-\lVert x_i - x_j \rVert^2 / 2\sigma_i^2)}
{\sum_{k \ne i} \exp(-\lVert x_i - x_k \rVert^2 / 2\sigma_i^2)}.
$$

In the low-dimensional map, t-SNE defines similarities using a heavy-tailed Student t distribution:

$$
q_{ij} =
\frac{(1 + \lVert y_i - y_j \rVert^2)^{-1}}
{\sum_{k \ne l} (1 + \lVert y_k - y_l \rVert^2)^{-1}}.
$$

The embedding is found by minimizing the Kullback-Leibler divergence:

$$
KL(P \Vert Q) = \sum_{i \ne j} p_{ij} \log \frac{p_{ij}}{q_{ij}}.
$$

This objective penalizes cases where points that should be close in the original space are placed far apart in the map. That is why t-SNE often produces visually separated islands of cell states. The tradeoff is that distances between far-apart clusters are harder to interpret.

In the GSE178341 reproduction, the published atlas-style views use public t-SNE coordinates. I used those coordinates to reproduce paper-style visualizations and to overlay Scissor/Scissor-style labels on cell populations.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/gse178341-global-tsne-mmr.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Public GSE178341 global t-SNE coordinates, used as the atlas-style coordinate system for the colorectal cancer single-cell analysis.
</div>

### UMAP: preserving a neighbor graph

UMAP is also nonlinear, but its intuition is graph based. It first builds a k-nearest-neighbor graph in the high-dimensional space, then optimizes a low-dimensional layout whose fuzzy graph resembles the original graph.

At a high level, UMAP assigns edge strengths between nearby cells:

$$
w_{ij} \approx \exp\left(-\frac{d(x_i, x_j) - \rho_i}{\sigma_i}\right),
$$

where $$\rho_i$$ adjusts for the nearest local neighbor and $$\sigma_i$$ adapts to local density. UMAP then finds low-dimensional points $$y_i$$ whose edge probabilities match the high-dimensional graph as closely as possible.

Compared with t-SNE, UMAP is often faster on large datasets and can preserve more global structure, although the exact interpretation of large-scale distances still requires caution. In the GSE178341 pipeline, UMAP is treated as an additional visualization when generated from the public processed H5AD data. The original paper-style panels are t-SNE based, so I keep that distinction clear in the analysis notes.

| Method | Core idea | Strength | Main caution |
| --- | --- | --- | --- |
| PCA | Linear projection maximizing variance | Fast, stable, useful for preprocessing | Only captures linear structure |
| t-SNE | Match local similarity probabilities | Strong local cluster visualization | Global distances can be misleading |
| UMAP | Match high-dimensional and low-dimensional neighbor graphs | Scalable, often preserves more continuum structure | Sensitive to graph and parameter choices |

## Part 2: Linear regression and sparse regression

Dimensionality reduction helps us see structure. Regression helps us ask whether a measured feature is associated with an outcome.

### Linear regression: a baseline model

The standard linear regression model is

$$
y = X\beta + \epsilon,
$$

where $$y$$ is the response, $$X$$ is a feature matrix, $$\beta$$ is a vector of coefficients, and $$\epsilon$$ is noise. The ordinary least squares estimate minimizes squared error:

$$
\hat{\beta}
= \arg\min_\beta \lVert y - X\beta \rVert_2^2.
$$

The coefficient $$\beta_j$$ measures how much the response changes with feature $$j$$ after accounting for the other features in the model. This is useful, but it becomes unstable when the number of features is large, features are correlated, or the sample size is limited. All three problems are common in gene-expression data.

### Sparse regression: selecting a smaller signal

Sparse regression adds a penalty that encourages many coefficients to become exactly zero. The most common example is the LASSO:

$$
\hat{\beta}
= \arg\min_\beta
\left(
\lVert y - X\beta \rVert_2^2
+ \lambda \lVert \beta \rVert_1
\right).
$$

The $$L_1$$ penalty

$$
\lVert \beta \rVert_1 = \sum_j |\beta_j|
$$

shrinks weak coefficients to zero. This is valuable in biology because we often want an interpretable subset of genes, pathways, or cells rather than a dense model involving every possible feature.

For survival data, the model is usually not ordinary linear regression, but the same linear-predictor idea appears inside a Cox proportional hazards model:

$$
h(t \mid x) = h_0(t)\exp(x^\top \beta).
$$

Here, $$x^\top \beta$$ is the risk score. A positive coefficient increases the estimated hazard, while a negative coefficient decreases it.

### How this appears in my single-cell analysis

In the `GSE178341_crc_immune_hubs` project, the regression question is: which single cells or cell states are associated with worse or better progression-free survival?

The workflow connects bulk survival data to single-cell profiles through shared CRC-axis genes. The local Scissor-style branch uses 84 CRC-axis genes, 1,719 bulk PFS samples, and 383 PFS events. For each detailed subgroup, it computes a Cox-style score for cells and classifies them as positive, negative, or neutral after multiple-testing correction.

The official Scissor branch is separate. It uses the published Scissor Cox workflow, builds a cell-cell network using Seurat PCA and nearest-neighbor steps, and then identifies `Scissor+`, `Scissor-`, and neutral cells. In the completed official detailed-subgroup run, 84 subgroups completed successfully. The top Scissor+ percentages included endothelial, fibroblast, pericyte, and myeloid/DC populations.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/cs07-official-scissor-tsne-status.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/blog/single-cell-methods/cs07-scissor-style-tsne-status.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Example t-SNE overlays for subgroup cS07. The official Scissor Cox branch and the local Scissor-style branch answer related questions but should be interpreted separately.
</div>

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-ranked-percent-positive.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Official Scissor+ percentage ranked across detailed subgroups. This summarizes where the survival-associated signal is concentrated.
</div>

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-full-atlas-highlight-page-01.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/blog/single-cell-methods/official-scissor-top10-driver-gene-heatmap.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
<div class="caption">
    Full-atlas Scissor+ highlighting and CRC-axis driver-gene activity help move from statistical labels back to biological interpretation.
</div>

## What I learned

PCA, t-SNE, and UMAP are not interchangeable plotting tools. PCA gives a linear and stable summary of dominant variation. t-SNE is strong for local atlas-style visualization. UMAP builds a graph-based layout that can be useful for additional exploratory views.

Regression answers a different question. It does not only ask where cells are located in an embedding; it asks how features relate to an outcome. In high-dimensional biological data, sparse regression ideas are especially useful because they force the model toward a smaller and more interpretable signal.

For my single-cell work, these ideas fit together naturally: dimensionality reduction helps organize and inspect the cell atlas, while sparse survival-associated modeling helps connect cell states and CRC-axis genes to clinical outcomes.
