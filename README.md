# Graph-Based Latent Distance Models for Data Visualization

This repository contains the implementation and experiments from the project **Graph-Based Latent Distance Models for Data Visualization**.

The project investigates whether **Latent Distance Models (LDMs)**, traditionally used in network analysis, can be adapted for **dimensionality reduction (DR)**.

The proposed approach first transforms high-dimensional data into a weighted graph. A Latent Distance Model is then used to learn a low-dimensional representation of that graph.

The method is evaluated against **UMAP**, **PaCMAP**, and **Random Projection** on five benchmark datasets.

---

## Overview

Dimensionality reduction aims to represent high-dimensional data in a lower-dimensional space while preserving important relationships between observations.

For visualization, the goal is typically to create a two-dimensional representation that preserves as much useful structure as possible.

Our approach follows the pipeline:

```text
High-dimensional data
        |
        v
Pairwise distances
        |
        v
Weighted k-NN graph
        |
        v
Positive + negative edge sampling
        |
        v
Latent Distance Model
        |
        v
Optimization
        |
        v
2D embedding
```

A central question of the project is whether a graph constructed from the original feature space contains enough information for an LDM to recover a useful low-dimensional representation.

---

## Method

### 1. Graph Construction

Given a dataset

$$
X \in \mathbb{R}^{N \times d}
$$

we first compute the Euclidean distance between observations:

$$
D_{ij} = \lVert x_i - x_j \rVert_2
$$

For each observation, the `k` nearest neighbors are identified.

The graph contains:

* **Positive edges** between each observation and its nearest neighbors.
* **Negative edges** sampled randomly from non-neighbors.
* **Weighted edges** based on distances in the original feature space.

A global distance scale, $\sigma$, is determined from a chosen percentile $q$ of the pairwise distances.

The edge weight before clipping is calculated as

$$
p_{ij} = e^{-D_{ij}/\sigma}
$$

where:

* $D_{ij}$ is the distance between observations $i$ and $j$.
* $\sigma$ controls how quickly the edge weight decreases with distance.
* $p_{ij}$ represents the target relationship strength between observations $i$ and $j$.

The probabilities are clipped to

$$
[\epsilon,\ 1-\epsilon]
$$

with

$$
\epsilon = 10^{-4}
$$

to prevent probabilities from becoming exactly 0 or 1.

### 2. Negative Sampling

Using only nearest-neighbor edges would primarily describe the local structure of the dataset.

To provide information about larger-scale relationships, the pipeline also samples random non-neighbors.

For every observation:

* `k` nearest neighbors are used as positive edges.
* `k * r` non-neighbors are randomly sampled as negative edges.

Here, $r$ is the **negative sampling ratio**.

Because randomly selected observations in large datasets are usually relatively far apart, these negative samples provide information about the global structure of the original space.

---

## Latent Distance Model

After graph construction, the graph is used as input to the Latent Distance Model.

Each observation is assigned a latent position

$$
z_i \in \mathbb{R}^{m}
$$

where $m$ is the output dimension.

For visualization:

$$
m = 2
$$

The model also learns a bias parameter $\alpha_i$ for each observation.

For a pair of observations $i$ and $j$, the model calculates

$$
l_{ij} = \alpha_i + \alpha_j - \lVert z_i-z_j \rVert_2
$$

The idea is that observations with a strong relationship in the graph should be located closer together in the latent space.

Observations with weaker relationships should generally be located farther apart.

The model therefore learns both:

* latent positions $Z$,
* node-specific bias parameters $\alpha$.

These parameters are optimized using a weighted binary cross-entropy loss.

After optimization, the learned positions

$$
Z \in \mathbb{R}^{N \times 2}
$$

form the final two-dimensional embedding.

---

## Complete Pipeline

The complete dimensionality-reduction procedure can be summarized as:

```text
Input data X
    |
    v
Compute pairwise Euclidean distances
    |
    v
Find k nearest neighbors
    |
    +----------------------+
    |                      |
    v                      v
Positive edges       Random negative edges
    |                      |
    +----------+-----------+
               |
               v
      Compute edge weights
               |
               v
       Construct graph
               |
               v
    Initialize latent positions
               |
               v
      Latent Distance Model
               |
               v
     Optimize Z and alpha
               |
               v
         2D embedding
```

---

## Datasets

The pipeline was evaluated on five datasets with different dimensionalities and structural properties.

| Dataset   | Dimension | Description                         |
| --------- | --------: | ----------------------------------- |
| Swissroll |        3D | Non-linear geometric benchmark      |
| S-hole    |        3D | Non-linear geometric benchmark      |
| Mammoth   |        3D | Complex geometric benchmark         |
| Pendigits |       16D | Handwritten digit features          |
| MNIST     |      784D | High-dimensional handwritten digits |

Swissroll, S-hole, and Mammoth provide geometric structures that can be visually inspected after dimensionality reduction.

Pendigits and MNIST are used to investigate how the methods behave as the dimensionality of the input data increases.

---

## Benchmark Methods

The proposed LDM pipeline was compared against three methods.

### UMAP

**Uniform Manifold Approximation and Projection (UMAP)** constructs a nearest-neighbor graph in the high-dimensional space and optimizes a low-dimensional representation.

UMAP generally places a strong emphasis on preserving local neighborhood structure.

### PaCMAP

**Pairwise Controlled Manifold Approximation and Projection (PaCMAP)** uses three types of point pairs:

* Near pairs
* Mid-near pairs
* Further pairs

The importance of these pairs changes throughout optimization, allowing PaCMAP to balance local and global structure.

### Random Projection

Random Projection was included as a baseline.

It projects the original high-dimensional data onto a randomly selected two-dimensional subspace.

---

## Evaluation Metrics

Three quantitative metrics were used to evaluate the resulting embeddings.

### KNN Accuracy

KNN Accuracy measures **local structure preservation**.

It evaluates how many nearest neighbors in the original space remain nearest neighbors after dimensionality reduction.

The experiments use

$$
k = 5
$$

for this metric.

### Random Triplet Accuracy

Random Triplet Accuracy measures **global structure preservation**.

For three randomly selected observations, the metric checks whether their relative distance ordering is preserved after dimensionality reduction.

Higher values indicate better preservation of global relationships.

### Centroid Triplet Accuracy

Centroid Triplet Accuracy also measures **global structure preservation**.

Instead of individual observations, it compares relative relationships between class centroids before and after dimensionality reduction.

---

## Results

### Local Structure Preservation

KNN Accuracy with `k = 5`:

| Dataset   | Random Projection |    PaCMAP |      UMAP |   LDM |
| --------- | ----------------: | --------: | --------: | ----: |
| Swissroll |             0.595 |     0.971 | **0.981** | 0.959 |
| S-hole    |             0.703 | **0.986** |     0.983 | 0.962 |
| MNIST     |             0.158 |     0.905 | **0.909** | 0.597 |
| Pendigits |             0.492 |     0.968 | **0.981** | 0.904 |

UMAP and PaCMAP generally achieve the strongest local preservation.

The LDM remains relatively close on the lower-dimensional datasets but experiences a significant reduction in local preservation on MNIST.

---

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

---

## Key Findings

The experiments suggest that the LDM pipeline has a stronger tendency toward **global structure preservation** than local structure preservation.

### Strengths

* Strong global structure preservation.
* Highest scores on all evaluated global preservation metrics.
* Competitive local preservation on low- and medium-dimensional datasets.
* Captures macro-global structures such as the Swissroll and the overall S-shape of S-hole.
* Demonstrates that Latent Distance Models can be adapted for dimensionality reduction.

### Limitations

* Local neighborhood preservation decreases as dimensionality increases.
* Visual cluster separation on MNIST is weaker than with UMAP and PaCMAP.
* Embedding quality depends strongly on the graph construction.
* The current implementation is considerably slower than the benchmark implementations.

One important observation is that **strong quantitative global preservation does not necessarily produce visually well-separated clusters**.

MNIST illustrates this particularly well. The LDM achieves the highest global preservation scores while producing less visually separated digit clusters than UMAP and PaCMAP.

---

## Default Hyperparameters

The following default values were used for the LDM experiments:

| Parameter         |  Value |
| ----------------- | -----: |
| `k`               |     20 |
| `sigma`           |     15 |
| `positive_weight` |      2 |
| `epochs`          |    150 |
| `negative_ratio`  |     10 |
| `weight_decay`    | 0.0001 |
| `learning_rate`   |   0.05 |

These parameters were chosen as reasonable default values across the tested datasets rather than optimal values for every individual dataset.

---

## Hyperparameter Observations

Several empirical observations were made during experimentation.

### Number of Neighbors

Smaller values of `k` generally place more emphasis on local neighborhoods.

Larger values introduce more global information into the graph.

The model was relatively stable for approximately:

```text
k = 10-30
```

### Sigma

The `sigma` parameter determines how quickly graph edge weights decrease with distance.

The model was relatively stable around:

```text
sigma = 10-30
```

Smaller values generally produce more stretched-out embeddings, while larger values tend to pull observations into more tightly clustered structures.

### Positive Weight

The positive weight increases the importance of nearest-neighbor edges during optimization.

For simpler datasets, a value close to `1.0` was often sufficient.

For noisier and higher-dimensional datasets such as MNIST, larger values could help keep known neighbors closer together.

### Epochs

Approximately

```text
epochs = 150
```

was generally sufficient for the tested datasets.

Increasing the number of epochs beyond this point often resulted in relatively small changes to the embedding.

### Weight Decay

A value around

```text
weight_decay = 0.0001
```

often provided a useful balance between preserving dataset structure and forcing the embedding to conform to the two-dimensional output space.

---

## Runtime

The current implementation focuses on investigating the method rather than computational optimization.

For approximately 5,500 Pendigits samples, the reported runtimes were:

| Method | Runtime |
| ------ | ------: |
| PaCMAP |   1.8 s |
| UMAP   |  52.2 s |
| LDM    |   192 s |

The current LDM implementation is therefore significantly slower than the benchmark implementations.

Runtime optimization remains an important area for future development.

---

## Discussion

The results suggest that the LDM can provide useful low-dimensional representations, particularly when preserving the larger-scale structure of a dataset is important.

For low- and medium-dimensional datasets, the method achieves both strong global preservation and relatively competitive local preservation.

The situation changes for very high-dimensional datasets.

As dimensionality increases, nearest-neighbor relationships become more difficult to identify reliably. Because the LDM pipeline depends heavily on its k-nearest-neighbor graph, inaccuracies in the graph directly affect the information available to the model.

This is particularly visible on MNIST, where the original data has 784 dimensions.

Although the LDM still achieves strong global preservation scores, both its local KNN Accuracy and visual cluster separation deteriorate compared with UMAP and PaCMAP.

---

## Future Work

Possible directions for future development include:

* Improving runtime and memory efficiency.
* More systematic hyperparameter optimization.
* Investigating interactions between graph-construction parameters.
* Improving local preservation for high-dimensional datasets.
* Testing additional datasets between approximately 100 and 600 dimensions.
* Exploring alternative graph-construction methods.
* Investigating alternative sampling strategies.
* Exploring alternative Latent Distance Model formulations and loss functions.

---

## Conclusion

This project demonstrates that **Latent Distance Models can be used as the foundation of a dimensionality-reduction pipeline**.

A crucial component of the method is the graph-construction stage, since the graph determines which information from the original high-dimensional space is passed to the LDM.

Across the tested datasets, the proposed method shows particularly strong **global structure preservation**, achieving the highest scores on the evaluated global metrics.

Its main limitation appears as dimensionality increases. On high-dimensional datasets such as MNIST, local neighborhood preservation and visual cluster separation deteriorate compared with UMAP and PaCMAP.

Overall, the results suggest that graph-based Latent Distance Models are a promising direction for dimensionality reduction, particularly when preserving the **macro-global structure of the original data** is an important objective.

---

## Authors

* **Søren Jacobsen**
* **Carl V. C. Nielsen**
* **Yusuf Gül**

Technical University of Denmark (DTU), 2026.

---

## References

1. Hoff, P. D., Raftery, A. E., & Handcock, M. S. (2002). *Latent Space Approaches to Social Network Analysis*. Journal of the American Statistical Association.

2. Beyer, K. et al. (1999). *When Is "Nearest Neighbor" Meaningful?*

3. Wang, Y. et al. (2021). *Understanding How Dimension Reduction Tools Work: An Empirical Approach to Deciphering t-SNE, UMAP, TriMAP, and PaCMAP for Data Visualization*.

---

## Project Report

For the complete methodology, experiments, visualizations, quantitative results, and discussion, see the accompanying report:

**Graph-Based Latent Distance Models for Data Visualization**

