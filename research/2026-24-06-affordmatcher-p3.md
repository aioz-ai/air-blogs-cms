---
last_modified_on: "2026-05-04"
title: AffordMatcher: Affordance Learning in 3D Scenes from Visual Signifiers
description: Semantic Matching, Affordance
series_position: 11
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: advance", "guides: smart_caching"]
---

import CodeExplanation from '@site/src/components/CodeExplanation';
import Highlight from '@site/src/components/Highlight';


In the previous part, we explored how the AffordBridge dataset is constructed. We now shift our focus to the details of our method, AffordMatcher, for detecting affordance regions in 3D scenes using image cues.

## Methodology
### Problem Formulation.
We formulate affordance grounding as an alignment problem between visual signifiers and 3D affordance regions. Our objective is to minimize the cross-modal feature discrepancy:
$$
        \min_{\phi, \psi} \sum_{i=1}^{n} \sum_{j=1}^{m} A_{ij} \; \big| \phi(b_i) - \psi(a_j, P_j) \big|^{2}, \\
        \text{ s. t. } \sum_{j=1}^{m} A_{ij} = 1, \quad \forall i \in \{1, \dots, n\}, (1)
$$
where $\mathcal{P} = \{P_{j}\}_{j=1}^{m}$ denotes the point cloud composed of local regions $P_{j}$, and $I$ is the corresponding RGB image containing $n$ interaction cues $\{b_{i}\}_{i=1}^{n}$. The reasoning extractor $\phi: \mathbb{R}^{d_b} \rightarrow \mathbb{R}^d$ encodes each $b_{i}$ that represents human-part, object, and interaction features. Meanwhile, the affordance extractor $\psi: \mathbb{R}^{d_{a}} \times \mathbb{R}^{3} \rightarrow \mathbb{R}^{d}$ is mapping each 3D region $P_{j}$ and its action descriptor $a_{j}$ into the same embedding space. The weight $A_{ij}$ measures the confidence that $b_{i}$ aligns with the affordance instance $(a_{j}, P_{j})$. Figure 4 describes our AffordMatcher method in detail.

![](https://res.cloudinary.com/dxtsa1xbl/image/upload/v1777953233/model_cfacgg.png)
*<center>**Figure 4**:  Design architecture of **AffordMatcher**: Given a high-resolution voxelized scene point cloud and a visual signifier, **AffordMatcher** reasons over these inputs for zero-shot affordance segmentation. The **affordance extractor** identifies 3D interactable regions, while the **reasoning extractor** encodes 2D human-object cues. Cross-modal alignment is achieved via instance matching through a dissimilarity matrix. The features from the dissimilarity matrix are thus optimized through match-to-match attention, followed by a zero-shot affordance optimization to localize actionable spatial regions that align with the given signifier.</center>*

### Instance Matching \& 3D Reasoning
Let $F_{P} \in \mathbb{R}^{m \times N_{P}}$ and $F_{I} \in \mathbb{R}^{n \times N_{I}}$ denote the feature representations of 3D candidate regions $\{M_{j}\}_{j=1}^{m}$ and visual signifiers $\{b_{i}\}_{i=1}^{n}$, where $N_{P}$ and $N_{I}$ represent the number of point features and the dimensionality of each visual signifier, respectively. We first project them into a shared space of dimension $N_{D}$ to obtain queries, keys, and values as: 
$$
  Q^{(I)} = F_I W_q^{(I)},\text{ }K^{(P)} = F_P W_k^{(P)},\text{ }V^{(P)} = F_P W_v^{(P)} \ \ (2a)\\ 
  Q^{(P)} = F_P W_q^{(P)},\text{ }K^{(I)} = F_I W_k^{(I)},\text{ }V^{(I)} = F_I W_v^{(I)} \ \ (2b)
$$
Based on Equation 2, cross-modal attention is then applied bidirectionally to align 2D and 3D representations as follows:
$$
        W^{(M)} = \texttt{softmax}\left(Q^{(I)} {K^{(P)}}^{\top}\right) V^{(P)}, \ \ (3a) \\
        W^{(R)} = \texttt{softmax}\left(Q^{(P)} {K^{(I)}}^{\top}\right) V^{(I)}, \ \ (3b)
$$
where $W^{(M)} \in \mathbb{R}^{n \times N_{D}}$ localizes spatial keypoint features guided by visual signifiers, and $W^{(R)} \in \mathbb{R}^{m \times N_D}$ captures reasoning feedback propagated from the 3D context.


### Dissimilarity Quantification
From Equation 3, we compute the dissimilarity matrix $D \in \mathbb{R}^{n \times m}$ to quantify the cross-modal correspondence between visual and spatial features. Each entry $D_{ij} \in [0, 1]$ measures the cosine dissimilarity between the $i$-th and the $j$-th spatial features:
$$
    D_{ij} = 1 - \max \left\{0, \, \frac{W^{(M)}_{i} \cdot W^{(R)}_{j}}{\|W^{(M)}_{i}\|_2 \, \|W^{(R)}_{j}\|_2}\right\}, (4)
$$
where $\cdot$ is the inner product and $\|\cdot\|_{2}$ is the Euclidean norm.

The dissimilarity matrix $D$, with each entry as in Equation 4, is flattened into a length $L = nm$ and projected into an embedding of dimension $N_{X}$, yielding $X = DW_{X} \in \mathbb{R}^{L \times N_{X}}$. We then apply additive *FastFormer-style self-attention* over $(Q, K, V)$, defined to be $(XW_{q}, XW_{k}, XW_{v})$, as:
$$
    Z = K \odot \left( \sigma(Q w_{q})^{\top} Q \right), \; M = V \odot \left( \sigma(Z w_{k})^{\top} Z \right), (5)
$$

where $\sigma(\cdot)$ denotes the element-wise sigmoid, $\odot$ represents the Hadamard product, and $w_{q}, w_{k} \in \mathbb{R}^{N_{X}}$ are learnable parameter vectors. Thus, a multi-head projection produces the final match matrix. With $M$ from Equation 5, we obtain:
$$
    \mathcal{M} = \texttt{MultiHead}(M) \; W_{h} + b_{h}, (6)
$$
where $\mathcal{M}$ is the \texttt{Match2Match} attention map, which is processed by the bounding-box and mask prediction heads. To address one-to-many correspondences as described, we apply a soft-thresholding mask on $D$. High-similarity pairs, $D_{ij} < 0.2$, are allowed to propagate multiple times through $\mathcal{M}$ to seek robust and stable matches in cluttered scenes.

### Cross-modality Affordance Learning
To enable cross-modality affordance learning that solves Equation 1, we align **(i)** embedding normalization for global consistency, followed by **(ii)**semantic and geometric embeddings, **(iii)** bidirectional mapping between modalities (Equation 2), and **(iv)** cross-modal attention dissimilarity. 

First, let $\phi_{i}: b_{i} \mapsto \mathbb{R}^{d}$ and $\psi_{j}: (a_{j}, P_{j}) \mapsto \mathbb{R}^{d}$ denote the projection heads for visual signifiers and spatial regions, respectively. Both embeddings are constrained to lie on the unit hypersphere, $\|\phi(b_{i})\|_{2} = 1$ and $\|\psi(a_{j}, P_{j})\|_{2} = 1$, to maintain feature and geometric consistency. The weighted sum of a normalization term and a regularization term results in the embedding regularization loss $\mathcal{L}_{\text{embed}}$:

$$
    \alpha \left[ \sum_{i=1}^{n}(\|\phi_i\|_2 - 1)^2 + \sum_{j=1}^{m}(\|\psi_j\|_2 - 1)^2 \right] + \beta \sum_{\theta} \|\theta\|_F^2,
$$

where $\theta \in \Theta_\phi \cup \Theta_\psi$, $\Theta_{\phi}$ and $\Theta_{\psi}$ represent the trainable parameters of $\phi$ and $\psi$, respectively. Then, let $M_{ij}$ denote the FastFormer attention output (Equation 2) and $T_{ij}$ its pseudo-target from S-CLIP, computed as a convex combination of CLIP text embeddings. The alignment loss $\mathcal{L}_{\mathrm{align}}$ is:
$$
    \mathcal{L}_{\mathrm{align}} = \sum_{i=1}^{n}\sum_{j=1}^{m} A_{ij}\,\|M_{ij} - T_{ij}\|_2^2.
$$
Next, with two linear projection heads, $g_{\text{ins}}, g_{\text{r}}: \mathbb{R}^d \rightarrow \mathbb{R}^d$, we further enforce bidirectional consistency $\mathcal{L}_{\text{bidir}}$:
$$
    \mathcal{L}_{\text{bidir}} = \sum_{i,j} A_{ij} \left( \|g_{\text{ins}}(\phi_i) - \psi_j\|_2^2 + \|g_{\text{r}}(\psi_j) - \phi_i\|_2^2 \right)
$$
Lastly, we penalize the ReLU-clipped cosine dissimilarity between $W^{(M)}$ and $W^{(R)}$, yielding the dissimilarity loss that explicitly reduces the cross-modal attention $\mathcal{L}_{\text{dissim}}$:
$$
    \mathcal{L}_{\text{dissim}} = \sum_{i,j} A_{ij} \left[ 1 - \frac{W^{(M)}_i \!\cdot\! W^{(R)}_j}{\|W^{(M)}_i\|_2 \|W^{(R)}_j\|_2} \right].
$$

Overall, the training objective combines all losses with the empirically chosen set of weights $\{\alpha, \beta, \lambda, \gamma, \eta \}$:
$$
    \mathcal{L}_{\text{total}} = \mathcal{L}_{\text{embed}} + \lambda\,\mathcal{L}_{\text{align}} + \gamma \mathcal{L}_{\text{bidir}} + \eta\,\mathcal{L}_{\text{dissim}}.
$$
To train Equation 3, we employ the reasoning extractor $\phi$ and the affordance extractor $\psi$ with **ViT-B/16** and **PointNet++**, respectively, each followed by two-layer MLP projection heads, with pose augmentation applied to $\phi$.



In the next part, we will evaluate the effectiveness of our approach on the AffordBridge dataset, comparing it with baseline methods.
</CodeExplanation>