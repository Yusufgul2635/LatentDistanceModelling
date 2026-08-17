# Graph-Based Latent Distance Models for Data Visualization

This repository contains the implementation and experiments from our project **Graph-Based Latent Distance Models for Data Visualization**.

The project investigates whether **Latent Distance Models (LDMs)**, traditionally used in network analysis, can be adapted for **dimensionality reduction (DR)**. The proposed method converts high-dimensional data into a weighted graph and uses an LDM to learn a two-dimensional representation.

The method is evaluated against **UMAP**, **PaCMAP**, and **Random Projection** on five benchmark datasets.

## Overview

Dimensionality reduction aims to represent high-dimensional data in a lower-dimensional space while preserving important relationships between observations.

Our approach follows the pipeline:

```text
High-dimensional data
        ↓
Pairwise distances
        ↓
Weighted k-NN graph
        ↓
Positive + negative edge sampling
        ↓
Latent Distance Model
        ↓
Optimization
        ↓
2D embedding
```

A central question of the project is whether constructing a graph from the original feature space provides enough information for an LDM to recover a useful low-dimensional representation.

## Method

### 1. Graph Construction

For each observation, the `k` nearest neighbors are identified using Euclidean distance.

The graph contains:

* **Positive edges** between nearest neighbors.
* **Negative edges** sampled randomly from non-neighbors.
* **Weighted edges** based on distances in the original feature space.

Edge probabilities are calculated as

[
p_{ij} =
\operatorname{clip}
\left(
e^{-D_{ij}/\sigma},
\epsilon,
1-\epsilon
\right)
]

where:

* (D_{ij}) is the distance between observations (i) and (j),
* (\sigma) determines the distance scale,
* (\epsilon) prevents probabilities from reaching exactly 0 or 1.

### 2. Latent Distance Model

Each observation is represented by a latent position

[
z_i \in \mathbb{R}^2.
]

The model defines the relationship between two observations using

[
\ell_{ij}
=========

\alpha_i + \alpha_j - |z_i-z_j|_2
]

where (\alpha_i) and (\alpha_j) are learnable node-specific parameters.

The latent positions are optimized using a weighted binary cross-entropy loss.

The resulting positions form the final **2D embedding**.

## Datasets

The pipeline was evaluated on five datasets:

| Dataset   | Dimensionality | Description                         |
| --------- | -------------: | ----------------------------------- |
| Swissroll |             3D | Non-linear geometric structure      |
| S-hole    |             3D | Non-linear geometric structure      |
| Mammoth   |             3D | Complex geometric benchmark         |
| Pendigits |            16D | Handwritten digit features          |
| MNIST     |           784D | High-dimensional handwritten digits |

The combination allows the method to be evaluated on both known geometric structures and higher-dimensional real-world data.

## Benchmark Methods

The LDM pipeline is compared against:

* **UMAP** — primarily focused on preserving local neighborhood structure.
* **PaCMAP** — balances local and global relationships using near, mid-near, and distant pairs.
* **Random Projection** — included as a simple baseline.

## Evaluation Metrics

Three metrics are used to evaluate the embeddings.

**KNN Accuracy** measures local structure preservation by determining how many nearest neighbors remain neighbors after dimensionality reduction.

**Random Triplet Accuracy** evaluates global structure by checking whether relative distance relationships between randomly sampled triplets are preserved.

**Centroid Triplet Accuracy** evaluates global relationships between class centroids before and after dimensionality reduction.

## Results

### Local Structure Preservation

KNN Accuracy (`k = 5`):

| Dataset   | Random Projection |    PaCMAP |      UMAP |   LDM |
| --------- | ----------------: | --------: | --------: | ----: |
| Swissroll |             0.595 |     0.971 | **0.981** | 0.959 |
| S-hole    |             0.703 | **0.986** |     0.983 | 0.962 |
| MNIST     |             0.158 |     0.905 | **0.909** | 0.597 |
| Pendigits |             0.492 |     0.968 | **0.981** | 0.904 |

UMAP and PaCMAP generally achieve better local preservation. The LDM performs competitively on lower-dimensional datasets but experiences a substantial decrease on MNIST.

## Global Structure Preservation

### Random Triplet Accuracy

| Dataset   | Random Projection | PaCMAP |  UMAP |       LDM |
| --------- | ----------------: | -----: | ----: | --------: |
| Swissroll |             0.734 |  0.755 | 0.623 | **0.835** |
| S-hole    |             0.824 |  0.870 | 0.775 | **0.900** |
| MNIST     |             0.555 |  0.601 | 0.623 | **0.715** |
| Pendigits |             0.630 |  0.742 | 0.747 | **0.800** |
| Mammoth   |             0.858 |  0.864 | 0.801 | **0.907** |

### Centroid Triplet Accuracy

| Dataset   | Random Projection | PaCMAP |  UMAP |       LDM |
| --------- | ----------------: | -----: | ----: | --------: |
| Swissroll |             0.746 |  0.820 | 0.514 | **0.928** |
| S-hole    |             0.824 |  0.894 | 0.727 | **0.929** |
| MNIST     |             0.592 |  0.767 | 0.781 | **0.804** |
| Pendigits |             0.540 |  0.719 | 0.746 | **0.876** |

The LDM achieves the highest score on all tested datasets for both global preservation metrics.

## Key Findings

* Latent Distance Models can successfully be adapted for dimensionality reduction.
* **Graph construction is a crucial component** of the proposed pipeline.
* The LDM shows particularly strong **global structure preservation**.
* UMAP and PaCMAP generally perform better at preserving **local neighborhoods**.
* The LDM performs well on low- and medium-dimensional datasets.
* Performance deteriorates on very high-dimensional data such as MNIST.
* Strong quantitative global preservation does not necessarily result in visually separated clusters.
* The current LDM implementation is considerably slower than optimized UMAP and PaCMAP implementations.

## Default Hyperparameters

The following default parameters were used:

| Parameter         |  Value |
| ----------------- | -----: |
| `k`               |     20 |
| `sigma`           |     15 |
| `positive_weight` |      2 |
| `epochs`          |    150 |
| `negative_ratio`  |     10 |
| `weight_decay`    | 0.0001 |
| `learning_rate`   |   0.05 |

These parameters are intended as a general starting point rather than an optimal configuration for every dataset.

## Runtime

For approximately 5,500 Pendigits samples:

| Method | Runtime |
| ------ | ------: |
| PaCMAP |   1.8 s |
| UMAP   |  52.2 s |
| LDM    |   192 s |

Runtime optimization was not a primary focus of the project and remains an important direction for future work.

## Future Work

Possible extensions include:

* improving runtime and memory efficiency;
* systematic hyperparameter optimization;
* investigating interactions between graph-construction parameters;
* improving local preservation for high-dimensional datasets;
* evaluating the method on additional datasets between 100 and 600 dimensions;
* exploring alternative graph construction strategies;
* exploring alternative LDM formulations and loss functions.

## Conclusion

The project demonstrates that **Latent Distance Models provide a viable approach to dimensionality reduction**.

The proposed pipeline performs especially well at preserving the **global structure** of datasets and, according to the evaluated global metrics, outperforms UMAP and PaCMAP across the tested datasets.

Its main limitation is high-dimensional data, where local preservation and visual cluster separation deteriorate. The method is also currently significantly slower than established dimensionality-reduction implementations.

Overall, the results suggest that graph-based Latent Distance Models are a promising direction when preservation of the **macro-structure of the original data** is particularly important.

## Authors

**Søren Jacobsen**
**Carl V. C. Nielsen**
**Yusuf Gül**

Technical University of Denmark (DTU), 2026.

## References

1. Hoff, P. D., Raftery, A. E., & Handcock, M. S. (2002). *Latent Space Approaches to Social Network Analysis*. Journal of the American Statistical Association.

2. Beyer, K. et al. (1999). *When Is "Nearest Neighbor" Meaningful?*

3. Wang, Y. et al. (2021). *Understanding How Dimension Reduction Tools Work: An Empirical Approach to Deciphering t-SNE, UMAP, TriMAP, and PaCMAP for Data Visualization*.

## Project Report

For the complete methodology, experiments, results, and discussion, see the accompanying project report:

**Graph-Based Latent Distance Models for Data Visualization**
