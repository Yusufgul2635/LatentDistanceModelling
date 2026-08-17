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

Given a dataset

$$
X \in \mathbb{R}^{N \times d},
$$

we first compute pairwise Euclidean distances between observations:

$$
D_{ij} = \lVert x_i - x_j \rVert_2.
$$

For each observation, the `k` nearest neighbors are identified.

The graph contains:

* **Positive edges** between each observation and its nearest neighbors.
* **Negative edges** sampled randomly from non-neighbors.
* **Weighted edges** based on distances in the original feature space.

A global distance scale is defined using a percentile of the pairwise distances:

$$
\sigma = \operatorname{Percentile}({D_{ij} \mid i < j}, q).
$$

The target edge probability is then calculated as

$$
p_{ij}
======

\operatorname{clip}
\left(
e^{-D_{ij}/\sigma},
\epsilon,
1-\epsilon
\right).
$$

Here:

* $D_{ij}$ is the distance between observations $i$ and $j$,
* $\sigma$ determines the distance scale,
* $\epsilon = 10^{-4}$ prevents probabilities from reaching exactly 0 or 1.

### 2. Latent Distance Model

Each observation is represented by a latent position

$$
z_i \in \mathbb{R}^{m}.
$$

For visualization, the output dimension is set to $m=2$.

The model assigns each pair $(i,j)$ the logit

$$
\ell_{ij}
=========

\alpha_i + \alpha_j - \lVert z_i-z_j\rVert_2,
$$

where $\alpha_i$ and $\alpha_j$ are learnable node-specific bias parameters.

The corresponding relationship strength is determined by the distance between the observations in latent space. Smaller distances result in a higher probability of a relationship.

The latent positions $Z$ and node biases $\alpha$ are optimized using a weighted binary cross-entropy loss:

$$
L =
\frac{1}{\sum_{(i,j)} w_{ij}}
\sum_{(i,j)}
w_{ij}
\left[
-p_{ij}\log \sigma(\ell_{ij})
-----------------------------

(1-p_{ij})\log(1-\sigma(\ell_{ij}))
\right].
$$

The resulting latent positions form the final **2D embedding**.

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

* **UMAP** — focuses strongly on preserving local neighborhood structure.
* **PaCMAP** — balances local and global relationships using near, mid-near, and distant pairs.
* **Random Projection** — included as a simple baseline.

## Evaluation Metrics

Three metrics are used to evaluate the embeddings.

### KNN Accuracy

Measures **local structure preservation** by determining how many nearest neighbors from the original space remain neighbors after dimensionality reduction.

For the experiments, the neighborhood size was set to $k=5$.

### Random Triplet Accuracy

Measures **global structure preservation** by checking whether relative distance relationships between randomly sampled triplets are maintained after dimensionality reduction.

### Centroid Triplet Accuracy

Measures global structure preservation using triplets formed from class centroids.

## Results

### Local Structure Preservation

KNN Accuracy with $k=5$:

| Dataset   | Random Projection |    PaCMAP |      UMAP |   LDM |
| --------- | ----------------: | --------: | --------: | ----: |
| Swissroll |             0.595 |     0.971 | **0.981** | 0.959 |
| S-hole    |             0.703 | **0.986** |     0.983 | 0.962 |
| MNIST     |             0.158 |     0.905 | **0.909** | 0.597 |
| Pendigits |             0.492 |     0.968 | **0.981** | 0.904 |

UMAP and PaCMAP generally achieve the strongest local preservation.

The LDM performs close to the two methods on the lower-dimensional datasets but experiences a significant decrease in local preservation on the 784-dimensional MNIST dataset.

## Global Structure Preservation

### Random Triplet Accuracy

| Dataset   | Random Projection | PaCMAP |  UMAP |       LDM |
| --------- | ----------------: | -----: | ----: | --------: |
| Swissroll |             0.734 |  0.755 | 0.623 | **0.835** |
| S-hole    |             0.824 |  0.870 | 0.775 | **0.900** |
| MNIST     |             0.555 |  0.601 | 0.623 | **0.715** |
| Pendigits |             0.630 |  0.742 | 0.747 | **0.800** |
| Mammoth   |             0.858 |  0.864 | 0.801 | **0.907** |

The LDM achieves the highest Random Triplet Accuracy on all five datasets.

### Centroid Triplet Accuracy

| Dataset   | Random Projection | PaCMAP |  UMAP |       LDM |
| --------- | ----------------: | -----: | ----: | --------: |
| Swissroll |             0.746 |  0.820 | 0.514 | **0.928** |
| S-hole    |             0.824 |  0.894 | 0.727 | **0.929** |
| MNIST     |             0.592 |  0.767 | 0.781 | **0.804** |
| Pendigits |             0.540 |  0.719 | 0.746 | **0.876** |

The LDM also achieves the highest Centroid Triplet Accuracy across all evaluated datasets.

## Key Findings

The experiments indicate that the LDM pipeline has a tendency toward **global structure preservation**.

### Strengths

* Strong global structure preservation.
* Competitive local preservation on low- and medium-dimensional datasets.
* Captures macro-global structures such as the Swissroll and the overall S-shape of S-hole.
* Demonstrates that Latent Distance Models can be adapted for dimensionality reduction.
* Outperforms the benchmark methods on the evaluated global preservation metrics.

### Limitations

* Local neighborhood preservation decreases as dimensionality increases.
* Visual cluster separation on MNIST is weaker than UMAP and PaCMAP.
* The quality of the final embedding depends heavily on graph construction.
* The current implementation is considerably slower than the optimized benchmark methods.

An important observation is that **strong global preservation does not necessarily imply visually well-separated clusters**.

This is particularly visible for MNIST, where the LDM achieves the strongest global quantitative scores while producing less visually separated digit clusters than UMAP and PaCMAP.

## Default Hyperparameters

The following default parameters were used for the LDM:

| Parameter           |     Value |
| ------------------- | --------: |
| $k$                 |        20 |
| $\sigma$ percentile |        15 |
| Positive weight     |         2 |
| Epochs              |       150 |
| Negative ratio      |        10 |
| Weight decay        | $10^{-4}$ |
| Learning rate       |      0.05 |

These values were selected as reasonable defaults across the tested datasets rather than optimal parameters for every individual dataset.

## Hyperparameter Observations

The experiments resulted in several useful observations:

* Smaller $k$ values place more emphasis on local neighborhoods.
* Larger $k$ values introduce more global information into the graph.
* Values of $k$ between approximately 10 and 30 were relatively stable.
* $\sigma$ was also relatively stable in approximately the 10–30 range.
* Increasing the positive-edge weight can improve local preservation for noisier datasets such as MNIST.
* Around 150 epochs were generally sufficient for the tested datasets.
* A weight decay around $10^{-4}$ often provided a useful balance between preserving structure and flattening the representation into 2D.

## Runtime

The current implementation prioritizes experimentation rather than computational optimization.

For approximately 5,500 Pendigits samples, the measured runtimes were:

| Method | Runtime |
| ------ | ------: |
| PaCMAP |   1.8 s |
| UMAP   |  52.2 s |
| LDM    |   192 s |

The LDM pipeline is therefore currently significantly slower than the benchmark implementations.

## Future Work

Possible directions for future development include:

* Improving runtime and memory efficiency.
* More systematic hyperparameter optimization.
* Investigating interactions between graph-construction parameters.
* Improving local preservation for high-dimensional datasets.
* Evaluating the method on additional datasets between approximately 100 and 600 dimensions.
* Exploring alternative graph-construction strategies.
* Exploring alternative LDM formulations and loss functions.

## Conclusion

This project demonstrates that **Latent Distance Models can be used as the foundation of a dimensionality-reduction pipeline**.

The graph-construction stage plays a central role because it determines which information from the original high-dimensional space is passed to the LDM.

Across the tested datasets, the proposed method showed particularly strong **global structure preservation**, achieving the highest scores on the evaluated global metrics.

Its main limitation appears when dimensionality becomes very high. On MNIST, local neighborhood preservation and visual cluster separation deteriorate compared with UMAP and PaCMAP.

Overall, the results suggest that graph-based Latent Distance Models are a promising direction for dimensionality reduction, particularly when preserving the **macro-global structure of the original data** is an important objective.

## Authors

* **Søren Jacobsen**
* **Carl V. C. Nielsen**
* **Yusuf Gül**

Technical University of Denmark (DTU), 2026.

## References

1. Hoff, P. D., Raftery, A. E., & Handcock, M. S. (2002). *Latent Space Approaches to Social Network Analysis*. Journal of the American Statistical Association.

2. Beyer, K. et al. (1999). *When Is "Nearest Neighbor" Meaningful?*

3. Wang, Y. et al. (2021). *Understanding How Dimension Reduction Tools Work: An Empirical Approach to Deciphering t-SNE, UMAP, TriMAP, and PaCMAP for Data Visualization*.

## Project Report

For the complete methodology, experiments, results, visualizations, and discussion, see:

**Graph-Based Latent Distance Models for Data Visualization**
